---
name: pimcore-studio-backend-config
description: Pimcore Studio DI and routing configuration — studio_backend.yaml service bindings, controller auto-discovery, container parameters, explicit argument wiring, and studio_routing.yaml setup
metadata:
  audience: pimcore-developers
  focus: backend
---

# Architecture Context

The StudioBackendBundle uses a layered architecture: Controllers (HTTP only) -> Services (business logic) -> Hydrators (DTO creation). Events provide extension points. All APIs are OpenAPI-documented.

Configuration files wire up all Studio components: `studio_backend.yaml` for DI bindings, `studio_routing.yaml` for route discovery.

---

# DI Configuration (studio_backend.yaml)

```yaml
services:
    _defaults:
        autowire: true
        autoconfigure: true
        public: false

    Pimcore\Bundle\DataImporterBundle\Controller\Studio\:
        resource: '../../Controller/Studio'
        public: true
        tags: [ 'controller.service_arguments' ]

    # Service interface -> implementation bindings
    Pimcore\Bundle\DataImporterBundle\Service\Studio\ConfigurationServiceInterface:
        class: Pimcore\Bundle\DataImporterBundle\Service\Studio\ConfigurationService

    # Hydrator interface -> implementation bindings
    Pimcore\Bundle\DataImporterBundle\Hydrator\ConfigurationDetailHydratorInterface:
        class: Pimcore\Bundle\DataImporterBundle\Hydrator\ConfigurationDetailHydrator
```

## Key Rules

- Controllers are auto-discovered via resource glob (recursive -- covers all subdirectories).
- Controllers must be `public: true` with `controller.service_arguments` tag.
- All service and hydrator bindings are interface -> class mappings.
- Everything else uses autowiring defaults.

---

# Container Parameters in Services

When a legacy controller uses `$this->getParameter('some.param')` to access a container parameter, the Studio service should receive it via **constructor injection with explicit DI wiring** in `studio_backend.yaml`:

```yaml
Pimcore\Bundle\DataImporterBundle\Service\Studio\ConnectionServiceInterface:
    class: Pimcore\Bundle\DataImporterBundle\Service\Studio\ConnectionService
    arguments:
        $doctrineConnections: '%doctrine.connections%'
```

The service constructor declares a typed parameter:
```php
public function __construct(
    private readonly array $doctrineConnections,
    // ... other autowired dependencies
) {
}
```

This keeps services decoupled from the container and makes dependencies explicit.

---

# Explicit Argument Wiring for Non-Autowirable Dependencies

When a service depends on multiple implementations of the same interface (e.g., different repository implementations), Symfony's autowiring cannot resolve the ambiguity. Use explicit `arguments` wiring in `studio_backend.yaml`:

```yaml
Pimcore\Bundle\CopilotBundle\Service\Studio\Configuration\ConfigurationServiceInterface:
    class: Pimcore\Bundle\CopilotBundle\Service\Studio\Configuration\ConfigurationService
    arguments:
        $automationActionConfigurationRepository: '@pimcore.copilot.automation_action_configuration_repository'
        $interactionActionConfigurationRepository: '@pimcore.copilot.interaction_action_configuration_repository'
```

Only wire the arguments that cannot be autowired -- all other dependencies are resolved automatically.

---

# Routing Configuration (studio_routing.yaml)

Routes are defined in `Resources/config/pimcore/studio_routing.yaml`:

```yaml
pimcore_data_importer_studio:
    resource: "@PimcoreDataImporterBundle/Controller/Studio"
    type: attribute
    prefix: '%pimcore_studio_backend.url_prefix%/bundle/data-importer'
    options:
        expose: true
```

## Key Rules

- **`type: attribute`** -- uses PHP attribute-based routing (not YAML/annotation).
- **`prefix`** -- uses the Symfony parameter `%pimcore_studio_backend.url_prefix%` concatenated with `/bundle/{bundle-name}`. This corresponds to `Prefix::BUNDLE`.
- **`resource`** -- points to the `Controller/Studio` directory using `@BundleName` shortcut. Covers all subdirectories recursively.
- **`options.expose: true`** -- exposes routes to JavaScript (FOSJsRoutingBundle).
- One routing entry per bundle; all Studio controllers are discovered from the single resource directory.
