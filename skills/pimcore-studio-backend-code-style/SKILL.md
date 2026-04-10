---
name: pimcore-studio-backend-code-style
description: Pimcore Studio PHP code style rules — strict types, class modifiers, final/readonly patterns, formatting (120 chars), named arguments, imports, PHPDoc, @throws documentation, and constructor promotion
metadata:
  audience: pimcore-developers
  focus: backend
---

# Architecture Context

The StudioBackendBundle uses a layered architecture: Controllers (HTTP only) -> Services (business logic) -> Hydrators (DTO creation). Events provide extension points. All APIs are OpenAPI-documented.

These code style rules apply to ALL PHP code written for Studio bundles.

---

# Strict Types

Every PHP file starts with:
```php
<?php
declare(strict_types=1);
```

---

# Class Modifiers

| Class Type | Modifier | Why |
|-----------|----------|-----|
| Service implementation | `final readonly class` | No inheritance, immutable after construction |
| Hydrator implementation | `final readonly class` | Same |
| Controller | `final class` | Cannot be readonly (extends AbstractApiController with mutable state) |
| Response/Parameter DTO | `final class` or `final readonly class` | `final class` when using `AdditionalAttributesTrait` (has mutable array); `final readonly class` for parameter-only DTOs |
| Event | `final class` | Extends `AbstractPreResponseEvent` (has mutable state) |
| Interface | `interface` | Standard |

---

# Internal Annotation

All classes except events get `@internal` in their PHPDoc:
```php
/**
 * @internal
 */
```
Events are public API (no `@internal`).

---

# Constructor Promotion

Always use constructor promotion for all injected dependencies and DTO properties:
```php
public function __construct(
    private readonly ConfigurationServiceInterface $configurationService
) {
}
```

---

# Readonly Properties

Use `readonly` on every property that is never reassigned. Prefer `final readonly class` when all properties are readonly and the class has no traits with mutable state.

---

# Formatting

- **Max 120 characters per line** (PSR-12). No exceptions.
- **No double blank lines** anywhere in the file.
- **Trailing comma** after the last parameter in multi-line constructor/method calls.
- **Single blank line** between methods; single blank line before `return` in multi-statement methods.

---

# Named Arguments

**Do NOT use named arguments** unless you need to skip positional parameters:

```php
// GOOD -- skipping first positional param
$this->hydrator->hydrateKeyName(groupName: $group->getName(), keyName: $key->getName());

// BAD -- unnecessary named arguments
$this->hydrator->hydrateKeyName(keyId: null, groupName: $group->getName(), keyName: $key->getName());

// BAD -- named arguments when all are positional
$this->service->doSomething(name: $name, config: $config);
```

---

# Imports

- **Always use `use` imports.** Never use FQCNs inline in code, PHPDoc, or `@throws`.
- **Sort imports** alphabetically.
- **Remove unused imports** immediately.

---

# PHPDoc

- **Remove redundant PHPDoc** -- don't document types already expressed by native type hints:
  ```php
  // BAD -- redundant
  /**
   * @param string $name The config name
   * @return ConfigurationDetail
   */
  public function getConfiguration(string $name): ConfigurationDetail

  // GOOD -- no PHPDoc needed when types are self-evident
  public function getConfiguration(string $name): ConfigurationDetail
  ```
- **Keep PHPDoc only when it adds information**: complex array shapes, `@throws`, semantic descriptions for non-obvious parameters.
- **Private helper methods** may have PHPDoc for `@throws` to document what they throw (since they have no interface).

---

# Default Values

**Remove redundant default values** at call sites. If the method signature has `$param = null` and you'd pass `null`, omit the argument (use named args if needed to skip):

```php
// BAD
$this->hydrator->hydrateKeyName(null, $group->getName(), $key->getName());

// GOOD
$this->hydrator->hydrateKeyName(groupName: $group->getName(), keyName: $key->getName());
```

---

# @throws Documentation

1. **Document `@throws` on interfaces only**, not on implementations. Implementations inherit the contract.
2. **Use short class names**, never FQCNs in `@throws`:
   ```php
   // GOOD
   use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
   /**
    * @throws NotFoundHttpException
    */

   // BAD
   /**
    * @throws \Symfony\Component\HttpKernel\Exception\NotFoundHttpException
    */
   ```
3. **Document every exception** that a method can throw, including `\Exception`. Import `Exception` via `use Exception;` and reference it as `@throws Exception` (short name, not FQCN).
4. **Add `@throws` for specific vendor exceptions** that callers should be aware of (e.g., `InvalidConfigurationException`, `QueueNotEmptyException`).
5. **Import all exception classes** used in `@throws` via `use` statements at the top of the file -- this includes `Exception` itself.
6. **Exception to the "interfaces only" rule**: Trait methods get `@throws` directly on the trait methods, since traits have no interfaces.
