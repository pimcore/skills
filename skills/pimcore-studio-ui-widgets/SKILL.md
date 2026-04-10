---
name: pimcore-studio-ui-widgets
description: Widget system in Pimcore Studio UI - registering widgets, opening them in layout areas, WidgetManagerTabConfig, and connecting widgets to navigation
metadata:
  audience: pimcore-developers
  focus: ui-extension-points
---

## What This Skill Covers

Working with the Pimcore Studio UI widget system: registering widgets, opening them in layout areas (main/left/right/bottom), configuring tabs via `WidgetManagerTabConfig`, and connecting widgets to main navigation.

## When to Use This Skill

- Creating a feature that needs its own panel/tab in Studio UI
- Registering a custom widget component
- Opening widgets programmatically
- Connecting a widget to a main navigation entry
- Building singleton or multi-instance widget tabs

## Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

Examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## Widget Registry

Widgets must be registered before they can be opened. Registration happens during module initialization via the `WidgetRegistry` service.

```typescript
import { type WidgetRegistry } from '@pimcore/studio-ui-bundle/modules/widget-manager'
import { container, serviceIds } from '@pimcore/studio-ui-bundle/app'
import { MyWidgetComponent } from './my-widget-component'

const widgetRegistryService = container.get<WidgetRegistry>(serviceIds.widgetManager)

widgetRegistryService.registerWidget({
  name: 'my-widget',
  component: MyWidgetComponent
})
```

- `name` is the identifier used later in `WidgetManagerTabConfig.component`
- `component` must be a React component (**named export**, not default)
- Register in your module's `onInit` before any code tries to open the widget

## Widget Interface

```typescript
interface Widget {
  name: string
  component: ComponentType
  titleComponent?: ComponentType
  contentTitleComponent?: ComponentType
  isModified?: (tabNode: TabNode) => boolean
  getContextProvider?: (context, children) => React.JSX.Element
  defaultGlobalContext?: boolean
  transformConfig?: (config) => config
  isVisible?: (widget: WidgetConfig) => boolean
}
```

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | `string` | Yes | Unique identifier for the widget |
| `component` | `ComponentType` | Yes | React component rendered as the widget body |
| `titleComponent` | `ComponentType` | No | Custom component for the tab title |
| `contentTitleComponent` | `ComponentType` | No | Custom component for the content area title bar |
| `isModified` | `(tabNode) => boolean` | No | Whether the widget has unsaved changes (shows tab indicator) |
| `getContextProvider` | `(context, children) => JSX.Element` | No | Wraps the widget in a context provider |
| `defaultGlobalContext` | `boolean` | No | Whether to use global context by default |
| `transformConfig` | `(config) => config` | No | Transform the widget config before rendering |
| `isVisible` | `(widget) => boolean` | No | Dynamic visibility of the widget |

Most widgets only need `name` and `component`.

## Widget Areas

Pimcore Studio UI uses a flexible layout with four widget areas:

```
┌──────────────────────────────────────────────────────┐
│                    Toolbar / Navigation              │
├──────────┬───────────────────────────┬───────────────┤
│          │                           │               │
│   LEFT   │          MAIN             │    RIGHT      │
│          │        (center)           │               │
│          ├───────────────────────────┤               │
│          │         BOTTOM            │               │
├──────────┴───────────────────────────┴───────────────┤
│                      Status Bar                      │
└──────────────────────────────────────────────────────┘
```

| Area | Purpose | Typical Use |
|------|---------|-------------|
| **main** | Center content area | Editors, listings, dashboards |
| **left** | Left sidebar | Tree navigation, filters |
| **right** | Right sidebar | Properties, details panels |
| **bottom** | Bottom panel | Logs, console output, search results |

Feature widgets usually open in **main**. Tree views and navigation panels typically go in **left**.

## useWidgetManager Hook

```typescript
import { useWidgetManager, type WidgetManagerTabConfig } from '@pimcore/studio-ui-bundle/modules/widget-manager'

const widgetManager = useWidgetManager()
```

| Method | Description |
|--------|-------------|
| `openMainWidget(config)` | Open widget in center area |
| `openLeftWidget(config)` | Open widget in left sidebar |
| `openRightWidget(config)` | Open widget in right sidebar |
| `openBottomWidget(config)` | Open widget in bottom panel |
| `closeWidget(id)` | Close a widget by its ID |
| `switchToWidget(id)` | Focus an already-open widget |

All four `open*Widget` methods take the same `WidgetManagerTabConfig` shape — pick the one matching the target area.

### Usage Example

```typescript
export const MyComponent = (): React.JSX.Element => {
  const widgetManager = useWidgetManager()

  const openMyFeature = (): void => {
    const tabConfig: WidgetManagerTabConfig = {
      name: 'My Feature',
      id: 'my-feature-singleton',
      component: 'my-feature',
      config: {
        translationKey: 'my-feature.title',
        icon: { type: 'name', value: 'widget-default' }
      }
    }

    widgetManager.openMainWidget(tabConfig)
  }

  return <Button onClick={ openMyFeature }>Open Feature</Button>
}
```

## WidgetManagerTabConfig

```typescript
interface WidgetManagerTabConfig {
  name: string
  id?: string
  component: string
  config: {
    translationKey?: string
    label?: string
    icon?: { type: 'name', value: string }
    [key: string]: any
  }
}
```

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | `string` | Yes | Display name for the tab |
| `id` | `string` | No | Unique ID. Auto-generated if omitted. Set a fixed value for singletons. |
| `component` | `string` | Yes | Must match a registered widget's `name` |
| `config` | `object` | Yes | Configuration passed to the widget component |
| `config.translationKey` | `string` | No | Translation key for the tab label |
| `config.label` | `string` | No | Static label (prefer `translationKey`) |
| `config.icon` | `object` | No | Icon displayed on the tab |
| `config.[key]` | `any` | No | Any additional data the widget needs (e.g., `assetId`) |

The `config` object is flexible — add any properties the widget component needs (e.g., `assetId`, `mode`, `parentFolderId`). Access them inside the widget via the widget context or props.

## Full Example: Widget + Registration + Navigation

End-to-end example showing a widget wired into main navigation.

### 1. Widget Component

```typescript
// your-bundle/assets/studio/js/src/modules/my-feature/components/my-feature-widget.tsx
import React from 'react'
import { Content, Header, Select, Button } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from 'react-i18next'

export const MyFeatureWidget = (): React.JSX.Element => {
  const { t } = useTranslation()

  return (
    <Content padded>
      <Header title={ t('my-feature.title') }>
        <Button type="primary" onClick={ () => {} }>
          { t('my-feature.action-button') }
        </Button>
      </Header>
      <p>{ t('my-feature.description') }</p>
    </Content>
  )
}
```

### 2. Module: Register Widget + Nav Item

```typescript
// your-bundle/assets/studio/js/src/modules/my-feature/index.ts
import { type AbstractModule } from '@pimcore/studio-ui-bundle'
import { container, serviceIds } from '@pimcore/studio-ui-bundle/app'
import { type WidgetRegistry } from '@pimcore/studio-ui-bundle/modules/widget-manager'
import { type MainNavRegistry } from '@pimcore/studio-ui-bundle/modules/app'
import { MyFeatureWidget } from './components/my-feature-widget'

export const MyFeatureModule: AbstractModule = {
  onInit: (): void => {
    const widgetRegistryService = container.get<WidgetRegistry>(serviceIds.widgetManager)
    widgetRegistryService.registerWidget({
      name: 'my-feature',
      component: MyFeatureWidget
    })

    const mainNavRegistry = container.get<MainNavRegistry>(serviceIds.mainNavRegistry)
    mainNavRegistry.registerMainNavItem({
      path: 'DataManagement/My Feature',
      label: 'my-feature.navigation.title',
      order: 10,
      perspectivePermission: 'dataManagement.myFeature',
      widgetConfig: {
        name: 'My Feature',
        id: 'my-feature',
        component: 'my-feature',  // Must match registered widget name
        config: {
          translationKey: 'my-feature.navigation.title',
          icon: { type: 'name', value: 'widget-default' }
        }
      }
    })
  }
}
```

### 3. Register Module in Plugin

```typescript
// your-bundle/assets/studio/js/src/plugin.ts
import { type AbstractPlugin } from '@pimcore/studio-ui-bundle'
import { MyFeatureModule } from './modules/my-feature'

export const YourBundlePlugin: AbstractPlugin = {
  modules: [MyFeatureModule]
}
```

### 4. Translations

```yaml
# your-bundle/translations/studio.en.yaml
my-feature:
  navigation:
    title: My Feature
  title: My Feature
  description: This is my custom feature widget.
  action-button: Run action

perspective-editor:
  form:
    main-nav-permission:
      dataManagement:
        myFeature: Data Management > My Feature
```

## Connecting a Widget to Navigation

The `component` field in `widgetConfig` is a **string** that must exactly match the `name` passed to `registerWidget()`. This is how the widget manager resolves which React component to render when the nav item is clicked.

```typescript
// Register with name 'my-feature'
widgetRegistryService.registerWidget({
  name: 'my-feature',        // <-- This name...
  component: MyFeatureWidget
})

// Reference in navigation's widgetConfig
mainNavRegistry.registerMainNavItem({
  path: 'DataManagement/My Feature',
  label: 'my-feature.navigation.title',
  widgetConfig: {
    component: 'my-feature',  // <-- ...must match here
    name: 'My Feature',
    id: 'my-feature',
    config: {
      translationKey: 'my-feature.navigation.title',
      icon: { type: 'name', value: 'widget-default' }
    }
  }
})
```

Widgets don't require a navigation entry — you can open them from any component by calling `widgetManager.openMainWidget(config)` (or any other `open*Widget` variant).

## Singleton vs Multi-Instance Widgets

| Pattern | ID strategy | Behavior | Use for |
|---------|-------------|----------|---------|
| **Singleton** | Fixed `id` | Re-opening switches to existing tab | Settings, listings, dashboards |
| **Multi-instance** | Omit `id` or use dynamic `id` (e.g. `asset-editor-${assetId}`) | Each open creates a new tab | Element editors, detail views, comparisons |

```typescript
// Singleton: always the same tab
{ name: 'Settings', id: 'settings', component: 'settings', config: { /* ... */ } }

// Multi-instance: new tab per asset
{ name: `Asset ${assetId}`, id: `asset-editor-${assetId}`, component: 'asset-editor',
  config: { assetId, /* ... */ } }
```

## Common Mistakes

### Referencing an unregistered widget in navigation

```typescript
// ❌ WRONG - Widget was never registered, fails silently
mainNavRegistry.registerMainNavItem({
  widgetConfig: { component: 'my-feature', /* ... */ }
})

// ✅ CORRECT - Register first, then reference
widgetRegistryService.registerWidget({ name: 'my-feature', component: MyFeatureWidget })
mainNavRegistry.registerMainNavItem({
  widgetConfig: { component: 'my-feature', /* ... */ }
})
```

### Default export instead of named export

```typescript
// ❌ WRONG
const MyWidget = () => <div>Hello</div>
export default MyWidget

// ✅ CORRECT
export const MyWidget = (): React.JSX.Element => <div>Hello</div>
```

### Case mismatch between registered name and widgetConfig.component

```typescript
// ❌ WRONG - Case-sensitive mismatch
registerWidget({ name: 'my-feature', component: MyFeatureWidget })
widgetConfig: { component: 'My-Feature' }

// ✅ CORRECT - Exact string match
widgetConfig: { component: 'my-feature' }
```

### Missing icon in config

```typescript
// ❌ Tab looks incomplete without an icon
config: { translationKey: 'my-feature.title' }

// ✅ Always provide an icon
config: {
  translationKey: 'my-feature.title',
  icon: { type: 'name', value: 'widget-default' }
}
```

### Not setting ID for singleton widgets

Omitting `id` on a nav-linked widget opens a new tab on every click. Set a fixed `id` for singletons.

## Next Steps

- [**pimcore-studio-ui-navigation**](../pimcore-studio-ui-navigation/SKILL.md) - Main navigation entries and perspective permissions
- [**pimcore-studio-ui-tabs-editors**](../pimcore-studio-ui-tabs-editors/SKILL.md) - Tabbed editor interfaces within widgets
- [**pimcore-studio-ui-context-menus**](../pimcore-studio-ui-context-menus/SKILL.md) - Context menu actions that open widgets
