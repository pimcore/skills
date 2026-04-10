---
name: pimcore-studio-ui-tabs-editors
description: Adding and customizing editor tabs in Pimcore Studio UI - tab managers, registration, override, permissions, and detachable tabs
metadata:
  audience: pimcore-developers
  focus: ui-extension-points
---

## What This Skill Covers

How to add, customize, and manage editor tabs in Pimcore Studio UI:
- **Tab Manager Architecture** - How each element type and subtype has its own tab manager
- **IEditorTab Interface** - The shape of a tab registration object
- **Available Tab Managers** - Complete list of tab managers for assets, documents, and data objects
- **Tab Registration & Override** - Adding or replacing tabs, with permissions and detachable support
- **Tab Component Pattern** - Building tab content with standard layout components
- **Plugin Wiring** - Connecting tab extensions to the module system

## When to Use This Skill

Use this when:
- Adding a new tab to an asset, document, or data object editor
- Replacing or extending an existing editor tab (e.g., custom preview)
- Controlling tab visibility based on permissions or element state
- Creating tabs that can be detached into their own window
- Need to know which tab manager to target for a specific element subtype

## Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## Tab Manager Architecture

Every element editor in Pimcore Studio is composed of tabs managed by **Tab Managers** -- injectable services in the DI container. Each element type (asset, document, data-object) has multiple tab managers, one per subtype.

### Key Concepts

1. **One tab manager per subtype** -- e.g., `ImageTabManager` for image assets, `PageTabManager` for page documents.
2. **Injectable via DI container** -- accessed through `container.get<T>(serviceIds[...])`.
3. **Registration order matters** -- Built-in tabs are registered first; custom tabs appear after.
4. **Tabs are plain objects** -- described by an `IEditorTab` object, not a class.

## IEditorTab Interface

Every tab registration uses the `IEditorTab` interface:

```typescript
interface IEditorTab {
  /** Unique key identifying the tab. Must not collide with existing tabs. */
  key: string

  /** Display label shown in the tab bar. Can be a string or JSX for custom rendering. */
  label: string | React.JSX.Element

  /** The React component rendered as tab content. */
  children: React.JSX.Element

  /** Icon shown next to the label in the tab bar. */
  icon: React.JSX.Element

  /**
   * Element-level permission required to see this tab.
   * Checked against the element's permissions object.
   * Example: 'properties', 'versions', 'settings'
   */
  workspacePermission?: string

  /**
   * User-level permission required to see this tab.
   * Checked against the current user's global permissions.
   * Example: 'notes_events', 'tags_configuration'
   */
  userPermission?: string

  /**
   * Dynamic visibility function. Return true to hide the tab.
   * Receives the current element as argument.
   */
  hidden?: (element: any) => boolean

  /**
   * Whether this tab can be popped out into a separate window.
   * Defaults to false.
   */
  isDetachable?: boolean
}
```

### Permission Evaluation Order

When a tab has multiple permission fields, all must pass for the tab to be visible:

1. `userPermission` -- Does the user have this global permission?
2. `workspacePermission` -- Does the user have this permission on the element?
3. `hidden()` -- Does the dynamic function return true?

## Available Tab Managers

Use the service ID that matches the element subtype you want to extend. Service IDs always follow the pattern `'{ElementType}/Editor/{SubType}TabManager'`.

### Asset Tab Managers

| Subtype | Service ID | Type |
|---------|-----------|------|
| Image | `serviceIds['Asset/Editor/ImageTabManager']` | `ImageTabManager` |
| Video | `serviceIds['Asset/Editor/VideoTabManager']` | `VideoTabManager` |
| Audio | `serviceIds['Asset/Editor/AudioTabManager']` | `AudioTabManager` |
| Text | `serviceIds['Asset/Editor/TextTabManager']` | `TextTabManager` |
| Document (PDF, etc.) | `serviceIds['Asset/Editor/DocumentTabManager']` | `DocumentTabManager` |
| Archive (ZIP, etc.) | `serviceIds['Asset/Editor/ArchiveTabManager']` | `ArchiveTabManager` |
| Folder | `serviceIds['Asset/Editor/FolderTabManager']` | `FolderTabManager` |
| Unknown | `serviceIds['Asset/Editor/UnknownTabManager']` | `UnknownTabManager` |

Asset types are imported from `@pimcore/studio-ui-bundle/modules/asset`.

### Document Tab Managers

| Subtype | Service ID | Type |
|---------|-----------|------|
| Page | `serviceIds['Document/Editor/PageTabManager']` | `PageTabManager` |
| Snippet | `serviceIds['Document/Editor/SnippetTabManager']` | `SnippetTabManager` |
| Email | `serviceIds['Document/Editor/EmailTabManager']` | `EmailTabManager` |
| Link | `serviceIds['Document/Editor/LinkTabManager']` | `LinkTabManager` |
| Hardlink | `serviceIds['Document/Editor/HardlinkTabManager']` | `HardlinkTabManager` |
| Folder | `serviceIds['Document/Editor/FolderTabManager']` | `FolderTabManager` |

Document types are imported from `@pimcore/studio-ui-bundle/modules/document`.

### Data Object Tab Managers

| Subtype | Service ID | Type |
|---------|-----------|------|
| Object | `serviceIds['DataObject/Editor/ObjectTabManager']` | `ObjectTabManager` |
| Variant | `serviceIds['DataObject/Editor/VariantTabManager']` | `VariantTabManager` |
| Folder | `serviceIds['DataObject/Editor/FolderTabManager']` | `FolderTabManager` |

Data object types are imported from `@pimcore/studio-ui-bundle/modules/data-object`.

**IMPORTANT:** The `FolderTabManager` type exists in all three modules (asset, document, data-object). Always import it from the module that matches your target element type — mismatching will cause type errors.

## Registering a New Tab

The standard pattern uses `AbstractModule.onInit()` to get the tab manager from the container and call `register()`.

```typescript
// File: your-bundle/assets/studio/js/src/modules/image-metadata-tab-extension.tsx
import React from 'react'
import { type AbstractModule, container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import { Icon } from '@pimcore/studio-ui-bundle/components'
import { type ImageTabManager } from '@pimcore/studio-ui-bundle/modules/asset'
import { ImageMetadataTab } from '../components/image-metadata-tab/image-metadata-tab'

export const ImageMetadataTabExtension: AbstractModule = {
  onInit (): void {
    const imageTabManager = container.get<ImageTabManager>(
      serviceIds['Asset/Editor/ImageTabManager']
    )

    imageTabManager.register({
      key: 'my-bundle.image-metadata',
      label: 'Metadata',
      icon: <Icon value="info-circle" />,
      children: <ImageMetadataTab />,

      // Element-level: user must have 'settings' permission on this asset
      workspacePermission: 'settings',

      // User-level: user must have global 'assets' permission
      userPermission: 'assets',

      // Dynamic visibility: hide for unpublished elements
      hidden: (element) => element?.published !== true,

      // Allow popping out into a separate window
      isDetachable: true
    })
  }
}
```

**To target a different element type or subtype**, change three things:
1. The type import (`ImageTabManager` -> your target)
2. The service ID (`'Asset/Editor/ImageTabManager'` -> your target from the tables above)
3. The import module path (`modules/asset` -> `modules/document` or `modules/data-object`)

### Registering Tabs for Multiple Subtypes

If your tab should appear across multiple subtypes, define it once and register with each tab manager:

```typescript
export const UniversalAssetTabExtension: AbstractModule = {
  onInit (): void {
    const customTab = {
      key: 'my-bundle.universal-tab',
      label: 'Universal Tab',
      icon: <Icon value="star" />,
      children: <UniversalTabContent />
    }

    container.get<ImageTabManager>(serviceIds['Asset/Editor/ImageTabManager']).register(customTab)
    container.get<VideoTabManager>(serviceIds['Asset/Editor/VideoTabManager']).register(customTab)
    container.get<DocumentTabManager>(serviceIds['Asset/Editor/DocumentTabManager']).register(customTab)
  }
}
```

## Overriding an Existing Tab

Replace an existing tab's content while preserving its key, label, icon, and permissions.

```typescript
// File: your-bundle/assets/studio/js/src/modules/custom-preview-tab.tsx
import { type AbstractModule, container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import { type ObjectTabManager } from '@pimcore/studio-ui-bundle/modules/data-object'
import { CustomPreviewTab } from '../components/custom-preview-tab/custom-preview-tab'

export const CustomPreviewOverride: AbstractModule = {
  onInit (): void {
    const objectTabManager = container.get<ObjectTabManager>(
      serviceIds['DataObject/Editor/ObjectTabManager']
    )

    // Always check that the tab exists — core may rename or remove it
    const previewTab = objectTabManager.getTab('preview')

    if (previewTab === undefined) {
      console.warn('[my-bundle] Tab "preview" not found. Override skipped.')
      return
    }

    // Re-register with the same key but new children (and optionally more)
    objectTabManager.register({
      ...previewTab,
      children: <CustomPreviewTab />,
      // Optionally override more fields:
      // label: 'Enhanced Preview',
      // isDetachable: true
    })
  }
}
```

**How it works:**
1. `getTab('preview')` retrieves the existing tab registration.
2. Spread `...previewTab` copies all existing properties.
3. `children` replaces only the rendered content.
4. `register()` with an existing key overwrites the previous registration.

## Tab Component Pattern

Tab content should use the standard layout pattern with `Content` and `Header` components. Access the current element via `useElementContext()` and element-specific draft hooks.

```typescript
// File: your-bundle/assets/studio/js/src/components/asset-info-tab/asset-info-tab.tsx
import React from 'react'
import { Content, Header } from '@pimcore/studio-ui-bundle/components'
import { useElementContext } from '@pimcore/studio-ui-bundle/modules/element'
import { useAssetDraft } from '@pimcore/studio-ui-bundle/modules/asset'
import { useTranslation } from 'react-i18next'

export const AssetInfoTab = (): React.JSX.Element => {
  const { t } = useTranslation()
  const { id } = useElementContext()
  const { asset } = useAssetDraft(id)

  if (asset === undefined) {
    return <Content padded>Loading...</Content>
  }

  return (
    <Content padded>
      <Header title={t('my-bundle.asset-info.title')} />
      <p>Type: {asset.type}</p>
      <p>Size: {asset.fileSize}</p>
    </Content>
  )
}
```

For tabs with toolbars or forms, wrap in `<Flex vertical className="absolute-stretch">` and add a `<Toolbar>` alongside `<Content>`. For form-based tabs, use `FormKit` — see [`pimcore-studio-ui-forms-antd`](../pimcore-studio-ui-forms-antd/SKILL.md).

## Plugin Wiring

Tab extensions must be wired into the plugin system to be loaded.

```typescript
// File: your-bundle/assets/studio/js/src/plugin.ts
import { type IAbstractPlugin } from '@pimcore/studio-ui-bundle'
import { ImageMetadataTabExtension } from './modules/image-metadata-tab-extension'
import { CustomPreviewOverride } from './modules/custom-preview-tab'

export const MyBundlePlugin: IAbstractPlugin = {
  name: 'MyBundlePlugin',

  onStartup ({ moduleSystem }) {
    moduleSystem.registerModule(ImageMetadataTabExtension)
    moduleSystem.registerModule(CustomPreviewOverride)
  }
}
```

### Lifecycle: Plugin vs Module

```
1. Plugin.onStartup()  -> Register modules via moduleSystem.registerModule()
                          (DO NOT register tabs here -- too early!)
2. Module.onInit()     -> Register tabs via tabManager.register()
                          (This is the correct place)
3. Application renders -> Tabs visible in element editors
```

**Rule:** Always register tabs in `Module.onInit()`, never in `Plugin.onStartup()`.

## Common Mistakes

### Wrong Service ID Format

```typescript
// WRONG
serviceIds['AssetFolderTabManager']  // Does not exist

// CORRECT — always '{ElementType}/Editor/{SubType}TabManager'
serviceIds['Asset/Editor/FolderTabManager']
```

### Registering Tabs in Plugin onStartup (Too Early)

```typescript
// WRONG — tab managers may not be ready, registration may fail or be overwritten
export const MyPlugin: IAbstractPlugin = {
  name: 'MyPlugin',
  onStartup ({ moduleSystem }) {
    const tabManager = container.get<ImageTabManager>(serviceIds['Asset/Editor/ImageTabManager'])
    tabManager.register({ ... })
  }
}

// CORRECT — register the module in the plugin, register tabs in Module.onInit()
export const MyPlugin: IAbstractPlugin = {
  name: 'MyPlugin',
  onStartup ({ moduleSystem }) {
    moduleSystem.registerModule(MyTabModule)
  }
}
```

### Mismatched Tab Manager Type and Element Module

```typescript
// WRONG — FolderTabManager imported from asset but service ID is for document
import { type FolderTabManager } from '@pimcore/studio-ui-bundle/modules/asset'
container.get<FolderTabManager>(serviceIds['Document/Editor/FolderTabManager'])

// CORRECT — match the import module to the element type
import { type FolderTabManager } from '@pimcore/studio-ui-bundle/modules/document'
container.get<FolderTabManager>(serviceIds['Document/Editor/FolderTabManager'])
```

### Tab Key Collision

```typescript
// WRONG — generic keys may collide with built-in tabs
key: 'settings'

// CORRECT — namespace your keys
key: 'my-bundle.settings'
```

### Not Handling Missing Tab During Override

```typescript
// WRONG — getTab() may return undefined
const previewTab = objectTabManager.getTab('preview')
objectTabManager.register({ ...previewTab, children: <CustomPreview /> })

// CORRECT — check before spreading
const previewTab = objectTabManager.getTab('preview')
if (previewTab === undefined) {
  console.warn('[my-bundle] Tab not found. Override skipped.')
  return
}
objectTabManager.register({ ...previewTab, children: <CustomPreview /> })
```

### Forgetting Icon or React Imports

```typescript
// CORRECT — both imports required when using JSX
import React from 'react'
import { Icon } from '@pimcore/studio-ui-bundle/components'
```

## Next Steps

- [**pimcore-studio-ui-react-components**](../pimcore-studio-ui-react-components/SKILL.md) - Component structure, props patterns, and styling for tab content
- [**pimcore-studio-ui-widgets**](../pimcore-studio-ui-widgets/SKILL.md) - Widget system for standalone panels and tools
- [**pimcore-studio-ui-permissions**](../pimcore-studio-ui-permissions/SKILL.md) - Detailed permission checking patterns
- [**pimcore-studio-ui-layout-components**](../pimcore-studio-ui-layout-components/SKILL.md) - Content, Box, Flex, and other layout components
- [**pimcore-studio-ui-forms-antd**](../pimcore-studio-ui-forms-antd/SKILL.md) - Building forms with FormKit for settings tabs
