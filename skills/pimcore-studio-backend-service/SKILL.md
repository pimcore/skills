---
name: pimcore-studio-backend-service
description: Pimcore Studio service layer patterns — permission checks, current user resolution, StaticResolverBundle usage, trait patterns, event dispatching, and cleanup with try/finally
metadata:
  audience: pimcore-developers
  focus: backend
---

# Architecture Context

The StudioBackendBundle uses a layered architecture: Controllers (HTTP only) -> Services (business logic) -> Hydrators (DTO creation). Events provide extension points. All APIs are OpenAPI-documented.

Services contain ALL business logic: permission checks, orchestration of vendor calls, exception conversion. Controllers delegate entirely to services.

---

# Service Structure

```php
final readonly class ConfigurationService implements ConfigurationServiceInterface
{
    public function __construct(
        private ConfigurationDetailHydratorInterface $configurationDetailHydrator
    ) {
    }

    public function getConfiguration(string $name): ConfigurationDetail
    {
        $config = $this->loadConfigurationWithPermission(
            $name,
            PermissionConstants::PLUGIN_DATA_IMPORTER_PERMISSION_READ
        );
        return $this->configurationDetailHydrator->hydrate($config);
    }
}
```

## Key Rules

- `final readonly class` with `@internal`.
- **One interface + one implementation per domain.**
- Name pattern: `{Domain}ServiceInterface` / `{Domain}Service`
- Namespace: `Service\Studio\`
- **Namespace-aware naming**: When the namespace already provides domain context, use short generic names (`ServiceInterface` / `Service`) instead of repeating the domain. For example, prefer `Pimcore\Bundle\CopilotBundle\Service\Studio\Configuration\ServiceInterface` over `ConfigurationServiceInterface` when the `Configuration` namespace already identifies the domain.
- **Max ~20 constructor dependencies.** Split into domain services if exceeded.
- **Permission checks** happen in services, not controllers. Use `loadConfigurationWithPermission()` private method.
- **Extract shared private helpers** (e.g., `loadConfigurationWithPermission`, `resolveCurrentUser`) into a trait or helper service rather than duplicating them across services.
- **Use hydrators for DTO creation**, never `new SomeResponse(...)` directly in services (except simple delegation).
- **Dispatch events** for every response DTO before returning it:
  ```php
  $response = $this->hydrator->hydrateSomething(...);
  $this->eventDispatcher->dispatch(new SomeEvent($response), SomeEvent::EVENT_NAME);
  return $response;
  ```

---

# Current User Resolution

Use `SecurityServiceInterface::getCurrentUser()` for resolving the current user. Never use `UserLoader::getUser()`.

**Always add `instanceof User` check** after `getCurrentUser()`:
```php
$user = $this->securityService->getCurrentUser();
if (!$user instanceof User) {
    throw new EnvironmentException('Could not resolve current user');
}
```

---

# Permission Check Pattern

```php
private function loadConfigurationWithPermission(string $name, string $permission): Configuration
{
    $config = Configuration::getByName($name);

    if (!$config) {
        throw new NotFoundHttpException(
            sprintf('Configuration with name "%s" not found', $name)
        );
    }

    if (!$config->isAllowed($permission)) {
        throw new ForbiddenException(
            sprintf('Access denied to configuration "%s"', $name)
        );
    }

    return $config;
}
```

Keep returning the `Configuration` object even when callers discard it -- consistency over micro-optimization.

---

# Resource Cleanup with try/finally

Use `try/finally` for cleanup of temp files/resources:
```php
try {
    // ... use file ...
} finally {
    @unlink($file->getPathname());
}
```

---

# Null Sort Parameter Guard

When building sort arrays from optional query parameters, always null-check before constructing the array. Passing `[null => null]` to Doctrine/repository methods causes unexpected behavior or errors:

```php
// BAD -- if sortBy/sortOrder are null, creates [null => null]
$sortInformation = [$parameters->getSortBy() => $parameters->getSortOrder()];

// GOOD -- guard against null
$sortInformation = [];
$sortBy = $parameters->getSortBy();
$sortOrder = $parameters->getSortOrder();
if ($sortBy !== null && $sortOrder !== null) {
    $sortInformation[$sortBy] = $sortOrder;
}
```

---

# Static Calls to Pimcore Classes Outside the Bundle (STRICT RULE)

**Never make static calls to Pimcore classes that live outside the current bundle.** This includes all static calls to Pimcore core classes (e.g., `Pimcore\Tool`, `Pimcore\Model\Asset`, `Pimcore\Logger`). Third-party library static calls (e.g., PHPUnit, Symfony helpers, utility libraries) are not affected by this rule.

Static calls to Pimcore classes couple the code to a concrete implementation, making it untestable and impossible to override. The correct replacement is **`Pimcore\Bundle\StaticResolverBundle`**, which provides injectable interface wrappers for all common Pimcore static calls.

### How to Replace a Static Call

1. Find the matching resolver interface in `Pimcore\Bundle\StaticResolverBundle` (located at `dev/pimcore/static-resolver-bundle/src/`).
2. Inject the interface via the constructor.
3. Replace the static call with the instance method call.
4. No DI wiring is needed -- all resolver interfaces are autowired automatically.

### Example: `Pimcore\Tool::getValidLanguages()`

```php
// BAD -- static call to a class outside the bundle
use Pimcore\Tool;

foreach (Tool::getValidLanguages() as $lang) { ... }

// GOOD -- inject ToolResolverInterface and call the instance method
use Pimcore\Bundle\StaticResolverBundle\Lib\ToolResolverInterface;

public function __construct(
    private ToolResolverInterface $toolResolver,
) {}

foreach ($this->toolResolver->getValidLanguages() as $lang) { ... }
```

### Finding the Right Resolver

Browse `dev/pimcore/static-resolver-bundle/src/` to find the resolver for the static class you need to replace:

| Static Class | Resolver Interface |
|---|---|
| `Pimcore\Tool` | `Pimcore\Bundle\StaticResolverBundle\Lib\ToolResolverInterface` |
| `Pimcore\Model\Asset` | `Pimcore\Bundle\StaticResolverBundle\Models\Asset\AssetServiceInterface` |
| `Pimcore\Model\DataObject\*` | `Pimcore\Bundle\StaticResolverBundle\Models\DataObject\*` |
| `Pimcore\Model\Document\*` | `Pimcore\Bundle\StaticResolverBundle\Models\Document\*` |

If no resolver exists yet for the static class you need, check with the team before creating a new one.

### Scope

This rule applies to **all layers**: services, hydrators, event listeners. It does not apply to legacy controllers (which are not to be modified during migration).

---

# Trait Patterns

## Namespace

Shared traits live in `Service\Studio\Traits\`.

## Structure

```php
/**
 * Requires the using class to have a property: SecurityServiceInterface $securityService
 *
 * @internal
 *
 * @property SecurityServiceInterface $securityService
 */
trait CurrentUserResolverTrait
{
    /**
     * @throws EnvironmentException if the current user cannot be resolved
     */
    private function resolveCurrentUser(): User
    {
        $user = $this->securityService->getCurrentUser();

        if (!$user instanceof User) {
            throw new EnvironmentException('Could not resolve current user');
        }

        return $user;
    }
}
```

## Key Rules

- **`@internal`** -- traits are internal implementation details.
- **`@property` PHPDoc** -- when a trait accesses `$this->someProperty` from the using class, document it with `@property Type $propertyName`. Also add a human-readable sentence: "Requires the using class to have a property: ...". If the trait makes no `$this->...` calls (e.g., only static calls), omit `@property`.
- **`@throws` goes on trait methods directly** -- this is the exception to the "interfaces only" rule. Traits have no interfaces, so `@throws` must be documented on the trait methods themselves.
- **Methods are `private`** -- trait methods like `loadConfigurationWithPermission()` and `resolveCurrentUser()` are private helpers, not part of any public contract.
