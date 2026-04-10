---
name: pimcore-studio-ui-using-sdk-in-bundles
description: How bundles consume the Pimcore Studio UI SDK - plugins, modules, DI, registries, and imports
metadata:
  audience: pimcore-developers
  focus: sdk-architecture
---

## What This Skill Covers

Core principles for consuming the Pimcore Studio UI SDK in bundles:
- Plugin and module system
- Dependency injection with Inversify
- SDK imports and available APIs
- Extension registries (components, widgets, menus, types)
- Module federation architecture

## When to Use This Skill

Use this when:
- Creating a new Pimcore Studio bundle
- Understanding how to extend Studio functionality
- Working with plugins and modules
- Accessing SDK services and components
- Registering UI elements (widgets, tabs, menus)
- Need to understand the overall architecture

## Core Architecture Principles

### Module Federation

Pimcore Studio uses **Webpack Module Federation** to enable dynamic loading of bundles:

- **Studio Core** = Host application (loads at startup)
- **Bundles** = Remote modules (loaded dynamically at runtime)
- **Shared Dependencies** = React, Ant Design, Inversify (singletons, one instance across all bundles)

**Key Benefit**: Bundles are independently built and loaded, but share common dependencies.

### The Extension Flow

```
Bundle Build → Entrypoint Registration → Runtime Loading → Plugin Execution → Modules Register → UI Renders
```

1. Bundle compiles assets with Module Federation
2. PHP service registers entrypoint location
3. Studio core discovers and loads the remote module
4. Bundle's `plugins.ts` exports are discovered
5. Plugins execute lifecycle hooks (`onInit`, `onStartup`)
6. Modules register configurations
7. React renders with extended functionality

## Plugins vs Modules

### Plugins - The Lifecycle Integration

Plugins are **entry points** that hook into the application lifecycle:

```typescript
import { type IAbstractPlugin } from '@pimcore/studio-ui-bundle'

export const MyPlugin: IAbstractPlugin = {
  name: 'MyPlugin',
  
  // Phase 1: Service registration (early)
  onInit: ({ container }) => {
    // Register or override services in DI container
    container.rebind(serviceIds['Some/Service'])
      .to(MyCustomService)
      .inSingletonScope()
  },
  
  // Phase 2: Module registration (after services ready)
  onStartup: ({ moduleSystem }) => {
    // Register modules that will configure the app
    moduleSystem.registerModule(MyModule)
  }
}
```

**When to use each hook:**
- **`onInit`**: Register NEW services or OVERRIDE existing ones
- **`onStartup`**: Register modules that CONFIGURE existing services

### Modules - The Configuration Executors

Modules are **configuration snippets** that run after services are initialized but before React renders:

```typescript
import { type AbstractModule, container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'

export const MyModule: AbstractModule = {
  onInit: () => {
    // Access services from container
    const widgetRegistry = container.get<WidgetRegistry>(serviceIds.widgetManager)
    
    // Configure the service
    widgetRegistry.registerWidget({
      name: 'my-widget',
      component: MyWidgetComponent
    })
  }
}
```

**Common module tasks:**
- Register widgets in WidgetManager
- Register tabs in TabManagers
- Add components to ComponentRegistry
- Register context menu items
- Configure dynamic types

### Mental Model

```
Plugin = "When and how to integrate with Studio lifecycle"
Module = "What to configure once services are ready"
```

**One plugin can register multiple modules** - organize by feature or concern.

## Dependency Injection (Inversify)

All services are managed by an IoC (Inversion of Control) container.

### Accessing Services in Modules/Plugins

```typescript
import { container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'

// Get a service
const widgetRegistry = container.get<WidgetRegistry>(
  serviceIds.widgetManager
)
```

### Accessing Services in React Components

```typescript
import { useInjection } from '@pimcore/studio-ui-bundle/app'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'

const MyComponent = () => {
  const widgetManager = useInjection<WidgetManager>(
    serviceIds['Widget/WidgetManager']
  )
  
  // Use the service
}
```

### Overriding Services (Advanced)

Replace core services with custom implementations:

```typescript
// In plugin's onInit
onInit: ({ container }) => {
  container.rebind(serviceIds['Asset/Editor/FolderTabManager'])
    .to(CustomFolderTabManager)
    .inSingletonScope()
}
```

**Key Principle**: All services are registered with string IDs (`serviceIds`), enabling type-safe access and easy replacement.

## SDK Imports - The Public API

### 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). The `@sdk/*` alias maps to the same code but is only available inside studio-ui-bundle core.

---

### Core Imports

```typescript
// Base SDK and container
import { 
  type IAbstractPlugin,
  type AbstractModule,
  container 
} from '@pimcore/studio-ui-bundle'

// DI container, service IDs, hooks, store, router
import { 
  serviceIds,
  useInjection,
  store,
  router 
} from '@pimcore/studio-ui-bundle/app'
```

### Component Imports

```typescript
// Reusable UI components (based on Ant Design)
import { 
  Content,
  Header,
  Sidebar 
} from '@pimcore/studio-ui-bundle/components'
```

### API Imports (RTK Query)

```typescript
// Auto-generated API hooks from OpenAPI
import { 
  useAssetGetByIdQuery,
  useAssetUpdateMutation 
} from '@pimcore/studio-ui-bundle/api/asset'

import {
  useDataObjectGetByIdQuery
} from '@pimcore/studio-ui-bundle/api/data-object'
```

### Module-Specific Imports

```typescript
// Application modules (asset editor, data object editor, etc.)
import { type AssetEditorTabManager } from '@pimcore/studio-ui-bundle/modules/asset'
import { type WidgetRegistry } from '@pimcore/studio-ui-bundle/modules/widget-manager'
```

### Utility Imports

```typescript
// Helper utilities
import { formatters } from '@pimcore/studio-ui-bundle/utils'
import { generateUuid } from '@pimcore/studio-ui-bundle/utils'
```

**Import Principle**: Folder structure mirrors import paths
- SDK code location: `studio-ui-bundle/assets/js/src/core/app/sdk/api/asset/index.ts`
- Bundle import: `@pimcore/studio-ui-bundle/api/asset`
- Core import: `@sdk/api/asset`

The `@sdk/*` alias (core only) and `@pimcore/studio-ui-bundle/*` (bundles) both point to the same SDK folder.

## Extension Registries

### ComponentRegistry

Manages React components with two patterns:

**1. Single Components** (unique, one per slot):
```typescript
componentRegistry.override('asset-toolbar-actions', MyToolbarComponent)
```

**2. Slots** (multiple components with priority):
```typescript
componentRegistry.registerToSlot({
  slot: 'data-object-context-menu',
  component: MyMenuItemComponent,
  priority: 10
})
```

**Extension points** are defined in `component-config.ts` - these are the available slots.

### WidgetManager

Manages widgets in 4 areas: **main**, **left**, **bottom**, **right**

```typescript
// Register widget
widgetRegistry.registerWidget({
  name: 'my-widget',
  component: MyWidgetComponent
})

// Open widget programmatically
const widgetManager = useInjection<WidgetManager>(
  serviceIds['Widget/WidgetManager']
)

widgetManager.open({
  name: 'my-widget',
  config: { /* widget config */ }
})
```

### ContextMenuRegistry

Centralized context menu management:

```typescript
contextMenuRegistry.registerToSlot({
  slot: 'tree-node-context-menu',
  component: MyMenuItem,
  priority: 100
})

// Or override entire slot provider
contextMenuRegistry.overrideSlotProvider('grid-row-context-menu', MyMenuProvider)
```

**Priority system** controls menu item order (higher = appears first).

### Dynamic Types

Abstract base classes for extensible type systems (grid cells, layouts, metadata, filters, etc.).

```typescript
// Example: Custom grid cell type
@injectable()
class CustomCellType extends DynamicTypeGridCellAbstract {
  id = 'custom-cell'
  getGridCellComponent(props) {
    return <CustomCellComponent {...props} />
  }
}

// Register in module
gridCellRegistry.registerDynamicType(container.get('DynamicTypes/GridCell/CustomCell'))
```

**For details**, see the `pimcore-studio-dynamic-types` skill.

## Bundle Structure Example

```typescript
// assets/js/src/plugins.ts
import { type IAbstractPlugin } from '@pimcore/studio-ui-bundle'
import { MyModule } from './modules/my-module'

export const MyPlugin: IAbstractPlugin = {
  name: 'MyPlugin',
  
  onStartup({ moduleSystem }) {
    moduleSystem.registerModule(MyModule)
  }
}

// assets/js/src/modules/my-module.tsx
import { type AbstractModule, container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import { type WidgetRegistry } from '@pimcore/studio-ui-bundle/modules/widget-manager'
import { MyWidget } from '../components/my-widget'

export const MyModule: AbstractModule = {
  onInit: () => {
    const widgetRegistry = container.get<WidgetRegistry>(
      serviceIds.widgetManager
    )
    
    widgetRegistry.registerWidget({
      name: 'my-widget',
      component: MyWidget
    })
  }
}

// assets/js/src/components/my-widget.tsx
import React from 'react'
import { useInjection } from '@pimcore/studio-ui-bundle/app'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'

export const MyWidget = () => {
  const someService = useInjection<SomeService>(serviceIds.someService)
  
  return <div>My Widget</div>
}
```

## Common Patterns

### Registering a New Tab

```typescript
export const TabExtension: AbstractModule = {
  onInit: () => {
    const tabManager = container.get<AssetEditorTabManager>(
      serviceIds['Asset/Editor/AssetEditorTabManager']
    )
    
    tabManager.register({
      key: 'my-tab',
      label: 'My Tab',
      component: MyTabComponent
    })
  }
}
```

### Adding a Toolbar Button

```typescript
export const ToolbarExtension: AbstractModule = {
  onInit: () => {
    const componentRegistry = container.get<ComponentRegistry>(
      serviceIds.componentRegistry
    )
    
    componentRegistry.registerToSlot({
      slot: 'asset-editor-toolbar',
      component: MyButton,
      priority: 50
    })
  }
}
```

### Creating a Navigation Entry

```typescript
export const NavExtension: AbstractModule = {
  onInit: () => {
    const mainNavRegistry = container.get<MainNavRegistry>(
      serviceIds.mainNavRegistry
    )
    
    mainNavRegistry.registerMainNavItem({
      path: 'My Tool/Sub Section',
      widgetConfig: {
        name: 'My Tool',
        id: 'my-tool',
        component: 'my-tool-widget'
      }
    })
  }
}
```

### Using RTK Query API

```typescript
import { useAssetGetByIdQuery } from '@pimcore/studio-ui-bundle/api/asset'

const MyComponent = ({ assetId }: { assetId: number }) => {
  const { data, isLoading, error } = useAssetGetByIdQuery({ id: assetId })
  
  if (isLoading) return <div>Loading...</div>
  if (error) return <div>Error</div>
  if (!data) return null
  
  return <div>{data.filename}</div>
}
```

## Key Principles

1. **Separation of Concerns**: Plugins (lifecycle) → Modules (configuration) → Components (render)
2. **Inversion of Control**: DI container manages all services, enabling easy replacement
3. **Open/Closed**: Core is closed for modification, open for extension via registries
4. **Type Safety**: Full TypeScript support with auto-generated types from OpenAPI
5. **Configuration over Code**: Declarative registration patterns instead of imperative code
6. **Progressive Enhancement**: Bundles extend without modifying core code

## Common Mistakes to Avoid

❌ **Don't import from internal paths**
```typescript
// BAD
import { Something } from '@pimcore/studio-ui-bundle/src/internal/path'
```

✅ **Only import from public API paths**
```typescript
// GOOD
import { Something } from '@pimcore/studio-ui-bundle/components'
```

❌ **Don't access services outside DI container**
```typescript
// BAD - direct instantiation
const service = new SomeService()
```

✅ **Always get services from container**
```typescript
// GOOD
const service = container.get<SomeService>(serviceIds.someService)
```

❌ **Don't configure in onInit of plugin**
```typescript
// BAD - too early, services might not be ready
onInit: ({ container }) => {
  const registry = container.get<Registry>(...)
  registry.register(...) // Service might not be fully initialized
}
```

✅ **Configure in modules (via onStartup)**
```typescript
// GOOD - services are ready
onStartup: ({ moduleSystem }) => {
  moduleSystem.registerModule(MyConfigModule)
}
```

## Reference

- **Docs**: https://docs.pimcore.com/platform/Studio_UI/
- **Example Bundle**: https://github.com/pimcore/studio-example-bundle
- **SDK Package**: `@pimcore/studio-ui-bundle` (npm)

## Next Steps

- [**pimcore-studio-ui-rtk-query-fundamentals**](../pimcore-studio-ui-rtk-query-fundamentals/SKILL.md) - Fetching data from APIs
- [**pimcore-studio-ui-forms-antd**](../pimcore-studio-ui-forms-antd/SKILL.md) - Building form UIs with FormKit
- [**pimcore-studio-ui-bundle-structure**](../pimcore-studio-ui-bundle-structure/SKILL.md) - Bundle organization and structure
- [**pimcore-studio-ui-navigation**](../pimcore-studio-ui-navigation/SKILL.md) - Adding navigation items to bundles
