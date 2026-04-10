---
name: pimcore-studio-backend-controller
description: Pimcore Studio controller patterns — one-action-per-controller, OpenAPI attributes, route configuration, parameter binding with DTOs, void/upload endpoints, and route priority rules
metadata:
  audience: pimcore-developers
  focus: backend
---

# Architecture Context

The StudioBackendBundle uses a layered architecture: Controllers (HTTP only) -> Services (business logic) -> Hydrators (DTO creation). Events provide extension points. All APIs are OpenAPI-documented.

Controllers handle HTTP concerns ONLY: route definition, OpenAPI docs, parameter binding, delegate to a service, return a response. No business logic, no permission checks, no vendor calls.

---

# Controller Structure

```php
final class GetController extends AbstractApiController
{
    private const string ROUTE = '/config/{name}';

    public function __construct(
        SerializerInterface $serializer,
        private readonly ConfigurationServiceInterface $configurationService
    ) {
        parent::__construct($serializer);
    }

    #[Route(path: self::ROUTE, name: 'pimcore_studio_api_...', methods: ['GET'])]
    #[Get(
        path: Prefix::BUNDLE . self::ROUTE,
        operationId: 'bundle_data_importer_...',
        description: 'bundle_data_importer_..._description',
        summary: 'bundle_data_importer_..._summary',
        tags: [Tags::DataImporter->value]
    )]
    // ... OpenAPI parameter/response attributes ...
    #[IsGranted(PermissionConstants::PLUGIN_DATA_IMPORTER_CONFIG)]
    #[DefaultResponses([HttpResponseCodes::UNAUTHORIZED, ...])]
    public function actionMethod(string $name): JsonResponse
    {
        return $this->jsonResponse($this->configurationService->doSomething($name));
    }
}
```

## Key Rules

- `final class` (not `final readonly class`) — extends `AbstractApiController` with mutable state.
- `@internal` PHPDoc on every controller class.
- **Route constant**: `private const string ROUTE` — always typed.
- **OpenAPI path**: `Prefix::BUNDLE . self::ROUTE` (bundle defines its own prefix).
- **Tags**: Use the bundle's `Tags` enum (`Tags::DataImporter->value`).
- **operationId / description / summary**: Translation keys following `bundle_{bundle_name}_{domain}_{action}[_suffix]`.
- **`#[IsGranted]`**: Always present on every action.
- **`#[DefaultResponses]`**: Always includes at least `HttpResponseCodes::UNAUTHORIZED`.
- **One service dependency per controller.** If a controller needs two services, reconsider the domain split.
- **No business logic in controllers.** No permission checks, no data transformation, no vendor calls.
- **Return types**: `JsonResponse` via `$this->jsonResponse(...)` for data responses; `Response` via `new Response()` for void/upload operations.

---

# File & Class Naming

- **One action per controller class.** No multi-action controllers.
- **Namespace mirrors URL structure:** `Controller\Studio\{Domain}\{Action}Controller`
    - `Config/GetController`, `Config/SaveController`, `DataType/LoadClassAttributesController`
- **Shorter names** when the namespace provides context (e.g., `GetController` not `GetConfigurationController`).
- Controllers go in subdirectories: `Config/`, `ClassificationStore/`, `DataType/`, `Utility/`.

## Splitting Monolithic Legacy Controllers

When migrating a legacy controller with multiple actions, split each action into its own controller class under a subdirectory:

```
# Legacy (one controller, many actions):
Admin/JobRunController.php  ->  list(), getRunningJobs(), rerun(), cancel()

# Studio (one controller per action, organized in subdirectory):
Controller/Studio/JobRun/JobRunsController.php          ->  GET /job-runs
Controller/Studio/JobRun/JobRunsRunningController.php   ->  GET /job-runs/running
Controller/Studio/JobRun/RerunController.php            ->  POST /job-runs/{id}/rerun
Controller/Studio/JobRun/CancelController.php           ->  POST /job-runs/{id}/cancel
```

Name each controller after the action it performs (or the resource it returns), not the legacy method name. Group related controllers in a subdirectory named after the domain.

---

# Route Priority for Wildcard Conflicts

When a parent route uses a wildcard parameter (e.g., `/{name}`) and sibling routes have literal path segments (e.g., `/types`, `/environments`), the wildcard can capture the literal segment before the specific route is matched. Fix this by adding `priority: 10` to the `#[Route]` attribute on static-path controllers:

```php
// This route would be captured by /{name} without priority
#[Route(path: self::ROUTE, name: 'pimcore_studio_api_copilot_configuration_types', methods: ['GET'], priority: 10)]
```

Apply this to every controller whose literal path segment could conflict with a wildcard in a sibling controller.

---

# Parameter Binding

| Source | Attribute | Controller Param |
|--------|----------|-----------------|
| Path params | `#[IdParameter(...)]` | Native typed param: `string $name` |
| JSON body | `#[ReferenceRequestBody(...)]` | `#[MapRequestPayload] SomeParameters $params` |
| Query string | Per-param attributes + `#[MapQueryString]` | `#[MapQueryString] SomeParameters $params = new SomeParameters()` |
| File upload | `#[MultipartFormDataRequestBody(...)]` | `Request $request` + manual `$request->files->get('file')` |

---

# Mandatory DTO Usage for Parameter Binding (STRICT RULE)

**Controllers MUST NEVER access raw `Request` objects to extract query parameters or request body data.** Always use dedicated DTO classes with Symfony's `#[MapQueryString]` or `#[MapRequestPayload]` attributes instead.

### Why

- Raw `$request->query->get()` / `$request->request->get()` bypasses type safety, has no validation, and scatters parameter parsing logic across controllers.
- DTOs centralize parameter definitions, enforce types via PHP's type system, and keep controllers thin.
- DTOs are reusable across controllers that share the same parameter sets.

### GET Endpoints -- `#[MapQueryString]`

Every GET endpoint that accepts query parameters MUST define a `final readonly class` DTO and bind it via `#[MapQueryString]`:

```php
// DTO
final readonly class PaginationParameters
{
    public function __construct(
        private int $page = 1,
        private int $pageSize = 100,
    ) {
    }

    public function getPage(): int
    {
        return max(1, $this->page);
    }

    public function getPageSize(): int
    {
        return max(1, $this->pageSize);
    }
}

// Controller
public function listItems(
    #[MapQueryString] PaginationParameters $parameters = new PaginationParameters(),
): JsonResponse {
    return $this->jsonResponse(
        $this->service->listItems(
            $parameters->getPage(),
            $parameters->getPageSize(),
        )
    );
}
```

Key rules for query string DTOs:
- **`final readonly class`** with `@internal`.
- **All properties MUST have default values** -- query params are optional by nature for collection endpoints.
- **Validation logic goes in getters**, not the constructor (e.g., `max(1, $this->page)` to enforce minimum values).
- **The controller signature MUST default to a new instance**: `$parameters = new SomeParameters()`.
- **OpenAPI per-param attributes** (`#[IntParameter]`, `#[TextFieldParameter]`, `#[BoolParameter]`) stay on the controller method for documentation, but actual value extraction comes from the DTO.
- **Shared DTOs** are encouraged: if multiple endpoints share the same parameters, create a single reusable DTO like `PaginationParameters`.

### POST/PUT Endpoints -- `#[MapRequestPayload]`

Every POST/PUT endpoint that accepts a JSON body MUST define a DTO and bind it via `#[MapRequestPayload]`:

```php
public function createItem(
    #[MapRequestPayload] CreateItemParameters $parameters,
): JsonResponse {
    return $this->jsonResponse(
        $this->service->createItem($parameters)
    );
}
```

### Exceptions -- When Raw `Request` Is Acceptable

Raw `Request $request` is ONLY acceptable for **file uploads** (`$request->files->get('file')`), because Symfony's MapRequestPayload does not handle multipart file data. If you encounter any other situation where you believe a DTO cannot be used, STOP and ask the user for a decision before implementing with raw `Request` access.

### Anti-Pattern: Raw Request Access

```php
// BAD -- raw Request access for query params
public function listItems(Request $request): JsonResponse
{
    $page = (int) $request->query->get('page', 1);
    $pageSize = (int) $request->query->get('pageSize', 100);
    return $this->jsonResponse($this->service->listItems($page, $pageSize));
}

// GOOD -- DTO with MapQueryString
public function listItems(
    #[MapQueryString] PaginationParameters $parameters = new PaginationParameters(),
): JsonResponse {
    return $this->jsonResponse(
        $this->service->listItems($parameters->getPage(), $parameters->getPageSize())
    );
}
```

---

# Void Response Pattern

Endpoints that perform an action but return no data (uploads, copy-preview, cancel-execution):

```php
public function upload(Request $request): Response
{
    $this->service->uploadSomething(...);

    return new Response();
}
```

- Return `new Response()` (Symfony `HttpFoundation\Response`), not `JsonResponse`.
- The service method returns `void`.
- OpenAPI uses `#[SuccessResponse(description: '...')]` with **no `content` parameter**:
  ```php
  #[SuccessResponse(description: 'bundle_data_importer_upload_preview_success_description')]
  ```
  Data endpoints in contrast include `content: new JsonContent(ref: SomeResponse::class)`.

---

# File Upload Pattern

```php
// Controller
public function upload(Request $request): Response
{
    $file = $request->files->get('file');

    if (!$file instanceof UploadedFile) {
        throw new EnvironmentException('Invalid file found in the request');
    }

    $this->previewDataService->uploadPreviewData($name, $file);

    return new Response();
}
```

OpenAPI for file uploads uses `MultipartFormDataRequestBody` with a second argument for required fields:
```php
#[MultipartFormDataRequestBody(
    [
        new Property(
            property: 'file',
            description: 'Preview data file to upload',
            type: 'string',
            format: 'binary'
        ),
    ],
    ['file']
)]
```

Service-side file upload validation:
```php
private const int MAX_FILE_SIZE = 10485760; // 10MB

public function uploadPreviewData(string $name, UploadedFile $file): void
{
    try {
        // ... permission check ...
        if ($file->getSize() === 0) {
            throw new EnvironmentException('Uploaded file is empty');
        }
        if ($file->getSize() > self::MAX_FILE_SIZE) {
            throw new MaxFileSizeExceededException(self::MAX_FILE_SIZE);
        }
        // ... process file ...
    } finally {
        @unlink($file->getPathname());
    }
}
```

- `DefaultResponses` for upload endpoints includes `HttpResponseCodes::MAX_FILE_SIZE_EXCEEDED`.
