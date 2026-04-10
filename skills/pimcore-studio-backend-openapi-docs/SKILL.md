---
name: pimcore-studio-backend-openapi-docs
description: Pimcore Studio OpenAPI documentation — translation keys for API docs, Prefix and Tags config classes, PermissionConstants, and mandatory translation string creation for new controllers
metadata:
  audience: pimcore-developers
  focus: backend
---

# Architecture Context

The StudioBackendBundle uses a layered architecture: Controllers (HTTP only) -> Services (business logic) -> Hydrators (DTO creation). Events provide extension points. All APIs are OpenAPI-documented.

All OpenAPI description, summary, and tag values use translation keys resolved from `translations/studio_api_docs.en.yaml` in the bundle. Every new controller MUST have corresponding translation strings.

---

# OpenAPI Config Classes

Namespace: `OpenApi\Config\`

## Prefix Class

A `final class` (not an enum) with `@internal` and a single typed string constant:

```php
/**
 * @internal
 */
final class Prefix
{
    public const string BUNDLE = AbstractApiController::PREFIX . '/bundle/data-importer';
}
```

## Tags Enum

A **backed string enum** with `@internal` and a `#[Tag]` OpenAPI attribute placed **before** the PHPDoc block:

```php
#[Tag(
    name: Tags::DataImporter->value,
    description: 'bundle_tag_data_importer_description',
)]
/**
 * @internal
 */
enum Tags: string
{
    case DataImporter = 'Bundle Data Importer';
}
```

- The `#[Tag]` attribute self-references (`Tags::DataImporter->value`) in its `name` parameter.
- Tag description uses a translation key: `bundle_tag_{bundle_name}_description`.

---

# Permission Constants

Namespace: `Utils\Constants\`

A `final class` with `@internal` and typed string constants:

```php
/**
 * @internal
 */
final class PermissionConstants
{
    public const string PLUGIN_DATA_IMPORTER_CONFIG = 'plugin_datahub_config';
    public const string PLUGIN_DATA_IMPORTER_PERMISSION_READ = 'plugin_datahub_permission_read';
    public const string PLUGIN_DATA_IMPORTER_PERMISSION_UPDATE = 'plugin_datahub_permission_update';
    public const string PLUGIN_DATA_IMPORTER_PERMISSION_DELETE = 'plugin_datahub_permission_delete';
    public const string PLUGIN_DATA_IMPORTER_ADMIN = 'plugin_datahub_admin';
}
```

**Two-tier permission model**:
- **Gate-level** (`PLUGIN_DATA_IMPORTER_CONFIG`): used in controller `#[IsGranted(...)]` -- broad "can access the data importer at all" check.
- **Entity-level** (`_PERMISSION_READ`, `_PERMISSION_UPDATE`, `_PERMISSION_DELETE`): used in service `loadConfigurationWithPermission()` calls -- fine-grained per-configuration checks.

---

# Translation Keys for OpenAPI Documentation

All OpenAPI description, summary, and tag values use translation keys resolved from `translations/studio_api_docs.en.yaml` in the bundle.

## Key Naming Convention

Keys follow the pattern: `bundle_{bundle_name}_{domain}_{action}_{suffix}`

| Suffix | Usage |
|--------|-------|
| `_summary` | `#[Get]` / `#[Post]` / `#[Put]` / `#[Delete]` `summary` parameter |
| `_description` | `#[Get]` / `#[Post]` etc. `description` parameter |
| `_success_description` | `#[SuccessResponse]` `description` parameter |

Examples:
```yaml
# translations/studio_api_docs.en.yaml
bundle_copilot_configuration_get_summary: 'Get configuration by name'
bundle_copilot_configuration_get_description: 'Returns a single copilot configuration by name'
bundle_copilot_configuration_get_success_description: 'Configuration data'
bundle_copilot_configuration_list_summary: 'List all configurations'
bundle_copilot_job_run_list_summary: 'List all job runs'
bundle_tag_copilot_description: 'Copilot bundle endpoints'
```

Tag descriptions use: `bundle_tag_{bundle_name}_description`.

---

# Mandatory Translation String Creation

**When adding a new controller, you MUST also add the corresponding translation strings to `translations/studio_api_docs.en.yaml`.** Every `operationId`, `description`, `summary`, and `#[SuccessResponse] description` value used in OpenAPI attributes is a translation key that must have a matching entry in the YAML file. Forgetting to add these results in raw translation keys being displayed in the OpenAPI documentation instead of human-readable text.

Checklist for each new controller:
1. `{operationId}_description` -- describes what the endpoint does (use YAML `|` for multi-line)
2. `{operationId}_summary` -- short one-line summary
3. `{success_response_key}` -- the key used in `#[SuccessResponse(description: '...')]`

Example for a new `TreeController`:
```yaml
class_field_collection_get_tree_description: |
    Get the field collection tree with grouped folders.
class_field_collection_get_tree_summary: Get field collection tree
class_field_collection_get_tree_success_response: Field collection tree with nodes and folders
```
