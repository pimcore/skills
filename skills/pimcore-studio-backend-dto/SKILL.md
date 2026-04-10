---
name: pimcore-studio-backend-dto
description: Pimcore Studio DTO, Schema, Hydrator, and Event patterns — response DTOs with AdditionalAttributes, parameter DTOs, query DTOs, OpenAPI schema annotations, hydrator structure, and pre-response events
metadata:
  audience: pimcore-developers
  focus: backend
---

# Architecture Context

The StudioBackendBundle uses a layered architecture: Controllers (HTTP only) -> Services (business logic) -> Hydrators (DTO creation). Events provide extension points. All APIs are OpenAPI-documented.

DTOs carry data between layers. Hydrators construct DTOs from raw data. Events are dispatched before returning response DTOs, providing extension points for third-party code.

---

# Response DTOs

```php
#[Schema(
    schema: 'BundleDataImporterDataPreviewResponse',
    title: 'Bundle Data Importer Data Preview Response',
    required: ['dataPreview', 'previewRecordIndex'],
    type: 'object'
)]
final class DataPreviewResponse implements AdditionalAttributesInterface
{
    use AdditionalAttributesTrait;

    public function __construct(
        #[Property(description: '...', type: 'array', items: new Items(...))]
        private readonly array $dataPreview,
        #[Property(description: '...', type: 'integer', example: 0)]
        private readonly int $previewRecordIndex,
    ) {
    }

    // getters...
}
```

## Key Rules

- **All response DTOs** implement `AdditionalAttributesInterface` and use `AdditionalAttributesTrait`.
- **Schema name prefix**: `BundleDataImporter{Name}` (matches the bundle name).
- **`required` array** in `#[Schema]` lists all non-nullable, non-defaulted properties.
- **OpenAPI `type: 'array'` must always have `items`**:
  ```php
  // GOOD
  #[Property(type: 'array', items: new Items(type: 'string'))]
  #[Property(type: 'array', items: new Items(type: 'object'))]
  #[Property(type: 'array', items: new Items(properties: [...], type: 'object'))]

  // BAD -- missing items
  #[Property(type: 'array')]
  ```
- **Import `Items`** from `OpenApi\Attributes\Items` when using array types.
- Use `readonly` on all constructor-promoted properties.
- Use `final class` (not `final readonly class`) because `AdditionalAttributesTrait` has a mutable `$additionalAttributes` array.
- Trailing comma after the last constructor parameter.
- `@internal` PHPDoc on all DTO classes.
- Namespace: `Schema\`

## Naming

- Response DTOs: `{Name}Response` (e.g., `DataPreviewResponse`)
- Non-response detail DTOs: just the name (e.g., `ConfigurationDetail`)

---

# Parameter DTOs (Request Bodies)

```php
#[Schema(
    schema: 'BundleDataImporterLoadPreviewParameters',
    title: 'Bundle Data Importer Load Preview Parameters',
    type: 'object'
)]
final readonly class LoadPreviewParameters
{
    public function __construct(
        #[Property(description: '...', type: 'object', nullable: true)]
        private ?array $currentConfig = null,
        #[Property(description: '...', type: 'integer', example: 0)]
        private int $recordNumber = 0,
    ) {
    }
    // getters...
}
```

Parameter DTOs don't need `AdditionalAttributesInterface` and can be `final readonly class`.
- Namespace: `Schema\`
- Name pattern: `{Name}Parameters` (e.g., `LoadPreviewParameters`)

---

# Query String Parameter DTOs

No OpenAPI attributes needed on the class -- the controller adds `#[TextFieldParameter]`, `#[BoolParameter]`, `#[IntParameter]`, etc. individually per query param:

```php
final readonly class ClassAttributeParameters
{
    public function __construct(
        private ?string $classId = null,
        private bool $loadAdvancedRelations = false,
    ) {
    }
    // getters...
}
```

### Paginated/Sortable Query DTOs

For endpoints with pagination, sorting, and filtering:
```php
final readonly class ClassificationStoreKeyParameters
{
    public function __construct(
        private ?string $classId = null,
        private ?string $fieldName = null,
        private ?string $transformationResultType = null,
        private ?string $sort = null,       // JSON-encoded sort config
        private int $start = 0,             // offset, not page number
        private int $limit = 15,
        private ?string $searchfilter = null,
        private ?string $filter = null,
    ) {
    }
    // getters...
}
```

Key conventions:
- **Pagination** uses `$start` (offset) and `$limit` (count), not `$page`/`$perPage`.
- **Sort** is passed as `?string` (JSON-encoded), not a structured type.
- **All properties have defaults**, making the entire query string optional.
- **Validation logic goes in getters**, not the constructor (e.g., `max(1, $this->page)`).
- **The controller signature MUST default to a new instance**: `$parameters = new SomeParameters()`.
- **Shared DTOs** are encouraged: if multiple endpoints share the same parameters, create a single reusable DTO.

Controller per-param attributes:
```php
#[TextFieldParameter(name: 'classId', description: '...', required: false, example: 'Car')]
#[IntParameter(name: 'start', description: '...', required: false, example: 0)]
#[IntParameter(name: 'limit', description: '...', required: false, example: 15)]
#[TextFieldParameter(name: 'sort', description: '...', required: false, example: null)]
```

---

# Hydrator Patterns

## Structure

```php
final readonly class PreviewHydrator implements PreviewHydratorInterface
{
    public function __construct(
        private SecurityServiceInterface $securityService,
        private PreviewService $previewService,
        private InterpreterFactory $interpreterFactory
    ) {
    }

    public function hydrateDataPreview(array $dataPreview, int $recordNumber): DataPreviewResponse
    {
        return new DataPreviewResponse($dataPreview, $recordNumber);
    }
}
```

## Key Rules

- `final readonly class` with `@internal`.
- Hydrators create DTOs. They can also encapsulate shared utility logic (e.g., `loadAvailableColumnHeaders`, `isValidJson`) when that logic is needed by both services and hydrators.
- **Grouped by domain** (one hydrator per domain, not one per DTO).
- Name pattern: `{Domain}HydratorInterface` / `{Domain}Hydrator`
- Namespace: `Hydrator\`
- Method naming: `hydrate{DtoName}` (e.g., `hydrateDataPreview`, `hydrateImportStart`).
- One interface + one implementation per domain (same as services).

---

# Event Patterns

## Structure

```php
final class DataPreviewEvent extends AbstractPreResponseEvent
{
    public const string EVENT_NAME = 'pre_response.data_importer.data_preview';

    public function __construct(
        private readonly DataPreviewResponse $dataPreview
    ) {
        parent::__construct($dataPreview);
    }

    public function getDataPreview(): DataPreviewResponse
    {
        return $this->dataPreview;
    }
}
```

## Key Rules

- Extend `AbstractPreResponseEvent` from StudioBackendBundle.
- `EVENT_NAME` constant follows pattern: `pre_response.{bundle_name}.{snake_case_dto_name}`.
- Typed `const string` for the event name.
- Constructor takes the response DTO, passes it to parent, and stores it in a readonly promoted property.
- Getter returns the specific DTO type (not the generic `AdditionalAttributesInterface`).
- **No `@internal`** -- events are public extension points (public API).
- **No redundant PHPDoc** on the getter if the return type is self-evident.
- **One event class per response DTO.**
- Namespace: `Event\Studio\PreResponse\`
- Name pattern: `{ResponseName}Event` (e.g., `ConfigurationDetailEvent` for `ConfigurationDetail` DTO).

## Dispatching Events in Services

Events MUST be dispatched before returning every response DTO:
```php
$response = $this->hydrator->hydrateSomething(...);
$this->eventDispatcher->dispatch(new SomeEvent($response), SomeEvent::EVENT_NAME);
return $response;
```
