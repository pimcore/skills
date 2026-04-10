---
name: pimcore-studio-backend-checklist
description: Pimcore Studio migration and review checklist — 34-point verification checklist, anti-patterns to avoid, and reference base classes for validating Studio code completeness
metadata:
  audience: pimcore-developers
  focus: backend
---

# Architecture Context

The StudioBackendBundle uses a layered architecture: Controllers (HTTP only) -> Services (business logic) -> Hydrators (DTO creation). Events provide extension points. All APIs are OpenAPI-documented.

Use this checklist before considering any migration or new Studio feature complete.

---

# Anti-Patterns to Avoid

1. **Passing nullable values to PHP filesystem functions** -- in PHP 8.x with `declare(strict_types=1)`, functions like `is_file()`, `filesize()`, `unlink()` throw `TypeError` when passed `null`. Always null-check before calling:
    ```php
    // BAD -- crashes with TypeError when $path is null
    if (is_file($path)) { ... }

    // GOOD -- guard against null
    if ($path === null || !is_file($path)) { return []; }

    // GOOD -- positive check
    if ($path !== null && is_file($path)) { ... }
    ```

2. **Assuming non-standard endpoints should be migrated** -- when encountering endpoints that break Studio conventions (e.g., public-facing webhooks with Bearer token auth, different route prefixes, non-admin endpoints), **always ask the user for a decision** before proceeding rather than assuming they should be migrated.

3. **Redundant runtime type checks after typed constructor params** -- if a constructor already enforces a type (e.g., `array $doctrineConnections`), don't add runtime `is_array()` or similar checks. The type system handles it.

4. **Static calls to Pimcore classes outside the bundle** -- never use static calls like `Pimcore\Tool::getValidLanguages()`, `Pimcore\Model\Asset::getById()`, or any other static call to a Pimcore class that lives outside the current bundle. Always inject the corresponding resolver interface from `Pimcore\Bundle\StaticResolverBundle` instead. Third-party library static calls are not affected by this rule.

5. **Using raw `Request` to extract query or body parameters** -- never use `$request->query->get()`, `$request->request->get()`, or `$request->get()` in controllers. Always define a DTO class and use `#[MapQueryString]` (for GET) or `#[MapRequestPayload]` (for POST/PUT). The only exception is file uploads (`$request->files->get('file')`). If you encounter a situation where a DTO seems impossible, **ask the user before proceeding with raw `Request` access**.

---

# Testing Checklist (34 Points)

Before considering any migration or new feature complete, verify:

## Controllers
1. Controllers: `final class`, `@internal`, extends `AbstractApiController`.
2. Controllers: no business logic, permission checks, or vendor calls.
3. Monolithic controllers: split into one-action-per-controller in subdirectories.
4. Route priorities: `priority: 10` on static routes conflicting with wildcards.

## Services & Hydrators
5. Services/hydrators: `final readonly class` with `@internal`.
6. Permission checks: present in every service method that takes a config name.
7. File cleanup: uses `try/finally` pattern.
8. Non-HTTP exceptions: caught and converted to Studio API exceptions in services.
9. Null sort params: null-checked before building sort arrays.
10. Static calls: no static calls to Pimcore classes outside the bundle -- replaced with injected `StaticResolverBundle` resolver interfaces (third-party library static calls are exempt).
11. Container params: converted from `$this->getParameter()` to constructor injection with DI wiring.

## DTOs & Events
12. Response DTOs: implements `AdditionalAttributesInterface` with `AdditionalAttributesTrait`.
13. Response DTOs: each has a corresponding event class.
14. Events: dispatched before returning every response DTO.
15. OpenAPI arrays: every `type: 'array'` has `items`.

## Parameter Binding
16. Parameter binding: all GET query params use `#[MapQueryString]` with a DTO -- no raw `$request->query->get()`.
17. Parameter binding: all POST/PUT body params use `#[MapRequestPayload]` with a DTO -- no raw `$request->request->get()`.
18. Query DTOs: `final readonly class`, all properties have defaults, validation in getters not constructors.
19. Query DTOs: controller signature defaults to `new SomeParameters()` for optional query strings.

## Code Style
20. `@throws`: on interfaces only (exception: trait methods).
21. `@throws`: short class names with proper `use` imports.
22. `@throws`: documents all throwable exceptions including `Exception`.
23. `@throws` types: reference Studio API exceptions, not generic PHP exceptions.
24. Imports: no unused imports.
25. Named arguments: only when skipping positional parameters.
26. FQCNs: none in code or PHPDoc -- everything via `use` imports.
27. Line length: max 120 chars. No exceptions.
28. Traits: `@internal`, `@property` for using-class properties, `@throws` on methods directly.
29. Nullable paths: guarded before `is_file()`, `filesize()`, etc.

## Endpoints
30. Void endpoints: return `new Response()`, `#[SuccessResponse]` omits `content`.
31. Upload endpoints: validate `instanceof UploadedFile`, check size, include `MAX_FILE_SIZE_EXCEEDED`.

## Configuration
32. DI config: `studio_backend.yaml` has bindings for all new interfaces.
33. Routing config: `studio_routing.yaml` exists and points to correct controller directory.
34. Translation strings: added for all new controller OpenAPI keys.

---

# Reference: StudioBackendBundle Base Classes

| Class | Purpose |
|-------|---------|
| `AbstractApiController` | Base controller with `jsonResponse()` helper. Injects `SerializerInterface`. |
| `AbstractPreResponseEvent` | Base event for pre-response extension. Takes `AdditionalAttributesInterface`. |
| `AdditionalAttributesInterface` | Contract for DTOs that support runtime extension via key-value attributes. |
| `AdditionalAttributesTrait` | Default implementation of `AdditionalAttributesInterface`. |
| `SecurityServiceInterface` | `getCurrentUser()` to resolve the authenticated user. |

## Studio API Exception Classes

All in namespace `Pimcore\Bundle\StudioBackendBundle\Exception\Api\` unless noted.

| Exception Class | HTTP Code | Use Case | Constructor |
|---|---|---|---|
| `NotFoundException` | 404 | Entity not found | `new NotFoundException(type: 'entity', id: $id, parameter: 'id', previous: $e)` |
| `ForbiddenException` | 403 | Permission denied | `new ForbiddenException(sprintf('Access denied to "%s"', $name))` |
| `ConflictException` | 409 | Optimistic locking, resource state conflicts | `new ConflictException($message)` |
| `EnvironmentException` | 500 | Server/environment errors (intentional 500) | `new EnvironmentException($message)` |
| `InvalidArgumentException` | 422 | Invalid input, validation failures | `new InvalidArgumentException(message: $msg, previous: $e)` |
| `MaxFileSizeExceededException` | 413 | File too large | `new MaxFileSizeExceededException($maxSize)` |
| `NotFoundHttpException` (Symfony) | 404 | Alternative to `NotFoundException` | `new NotFoundHttpException($message)` |
