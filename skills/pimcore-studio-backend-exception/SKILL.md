---
name: pimcore-studio-backend-exception
description: Pimcore Studio exception handling — exception types, non-HTTP exception conversion, ModelNotFoundException to API NotFoundException, aliasing patterns, and the ApiExceptionSubscriber behavior
metadata:
  audience: pimcore-developers
  focus: backend
---

# Architecture Context

The StudioBackendBundle uses a layered architecture: Controllers (HTTP only) -> Services (business logic) -> Hydrators (DTO creation). Events provide extension points. All APIs are OpenAPI-documented.

Exception handling is critical because the `ApiExceptionSubscriber` in StudioBackendBundle **only handles exceptions implementing `HttpExceptionInterface`**. All other exceptions bypass it and produce raw 500 HTML errors. Services MUST catch vendor/legacy exceptions and convert them to Studio API exceptions.

---

# Studio API Exception Classes

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

**Key insight**: `Pimcore\Model\Exception\NotFoundException` (from the Pimcore model layer) extends `RuntimeException`, NOT `HttpExceptionInterface`. It will always produce a 500 if not caught. Always convert it to the Studio API `NotFoundException` in service methods.

---

# Non-HTTP Exception Conversion

## `ModelNotFoundException` -> API `NotFoundException`

`Pimcore\Model\Exception\NotFoundException` extends `RuntimeException` (NOT `HttpExceptionInterface`), so it always results in a 500 if uncaught. Catch it in services and rethrow as the Studio API `NotFoundException`:

```php
use Pimcore\Bundle\StudioBackendBundle\Exception\Api\NotFoundException;
use Pimcore\Model\Exception\NotFoundException as ModelNotFoundException;

try {
    $entity = $this->repository->getById($id);
} catch (ModelNotFoundException $e) {
    throw new NotFoundException(
        type: 'entity name',
        id: $id,
        parameter: 'id',
        previous: $e,
    );
}
```

Note: `NotFoundException` uses named parameters: `new NotFoundException(type: ..., id: ..., parameter: ..., previous: ...)`.

## `\InvalidArgumentException` / `\JsonException` -> API `InvalidArgumentException`

When vendor/legacy code throws `\InvalidArgumentException` or `\JsonException` for validation errors, convert them to the Studio API `InvalidArgumentException` (HTTP 422):

```php
use Pimcore\Bundle\StudioBackendBundle\Exception\Api\InvalidArgumentException as ApiInvalidArgumentException;

try {
    $result = $this->legacyService->doSomething($input);
} catch (\InvalidArgumentException | \JsonException $e) {
    throw new ApiInvalidArgumentException(
        message: $e->getMessage(),
        previous: $e,
    );
}
```

---

# Exception Aliasing Pattern

When both a vendor and Studio API have a class named `NotFoundException` (or `InvalidArgumentException`), use aliasing to avoid conflicts:

```php
use Pimcore\Bundle\StudioBackendBundle\Exception\Api\NotFoundException;
use Pimcore\Model\Exception\NotFoundException as ModelNotFoundException;

use Pimcore\Bundle\StudioBackendBundle\Exception\Api\InvalidArgumentException as ApiInvalidArgumentException;
```

---

# Reference: StudioBackendBundle Base Classes

| Class | Purpose |
|-------|---------|
| `AbstractApiController` | Base controller with `jsonResponse()` helper. Injects `SerializerInterface`. |
| `AbstractPreResponseEvent` | Base event for pre-response extension. Takes `AdditionalAttributesInterface`. |
| `AdditionalAttributesInterface` | Contract for DTOs that support runtime extension via key-value attributes. |
| `AdditionalAttributesTrait` | Default implementation of `AdditionalAttributesInterface`. |
| `SecurityServiceInterface` | `getCurrentUser()` to resolve the authenticated user. |
