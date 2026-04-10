---
name: pimcore-studio-ui-context-menus
description: Adding and customizing context menu items in Pimcore Studio UI - ContextMenuRegistry, slots, providers, and priority system
metadata:
  audience: pimcore-developers
  focus: ui-extension-points
---

## What This Skill Covers

Adding and customizing context menus in Pimcore Studio UI: registering new items, modifying built-in ones, controlling priority/position, and hiding items conditionally.

## When to Use This Skill

- Adding custom actions to tree, grid, or toolbar context menus
- Modifying, replacing, or hiding existing context menu items
- Positioning custom items relative to built-in entries

## Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## Core Concepts

### ContextMenuRegistry

Central registry that manages all context menu items. Each menu location is a **slot**, and you register **providers** into slots.

### Slots

A slot is a named attachment point. Each tree, grid, or toolbar has one or more slots, available as constants on `contextMenuConfig`.

### Providers

A provider implements `ContextMenuItemProvider`: a unique `name`, optional `priority`, and a `useMenuItem` React hook that returns the menu item (or `null` to hide it).

### Priority

**Lower number = higher position.** Default is `999`. Use `contextMenuConfig` priority constants to position relative to built-in entries.

---

## Accessing the ContextMenuRegistry

```typescript
import { container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import {
  contextMenuConfig,
  type ContextMenuRegistryInterface
} from '@pimcore/studio-ui-bundle/modules/app'

const contextMenuRegistry = container.get<ContextMenuRegistryInterface>(
  serviceIds['App/ContextMenuRegistry/ContextMenuRegistry']
)
```

---

## ContextMenuRegistry API

### registerToSlot — Add a New Item

```typescript
contextMenuRegistry.registerToSlot(slotName: string, provider: ContextMenuItemProvider): void
```

Example:

```typescript
import React from 'react'
import { type AbstractModule, container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import { useAlertModal, Icon } from '@pimcore/studio-ui-bundle/components'
import {
  contextMenuConfig,
  type ContextMenuRegistryInterface,
  type DataObjectTreeContextMenuProps
} from '@pimcore/studio-ui-bundle/modules/app'
import { useTranslation } from 'react-i18next'

export const DataObjectContextMenuExtension: AbstractModule = {
  onInit () {
    const contextMenuRegistry = container.get<ContextMenuRegistryInterface>(
      serviceIds['App/ContextMenuRegistry/ContextMenuRegistry']
    )
    const config = contextMenuConfig.dataObjectTree

    contextMenuRegistry.registerToSlot(config.name, {
      name: 'custom-item',
      priority: config.priority.rename - 1,
      useMenuItem: (context: DataObjectTreeContextMenuProps) => {
        const { t } = useTranslation()
        const alertModal = useAlertModal()

        // Conditional hiding: return null to hide for this context
        if (context.target.type === 'folder') {
          return null
        }

        return {
          key: 'custom-item',
          label: t('my-bundle.custom-item.label'),
          icon: <Icon value="pimcore" />,
          onClick: () => {
            alertModal.info({
              title: `Clicked for id: ${context.target.id}`,
              content: 'Custom action.'
            })
          }
        }
      }
    })
  }
}
```

### overrideSlotProvider — Replace an Existing Item

```typescript
contextMenuRegistry.overrideSlotProvider(slotName: string, provider: ContextMenuItemProvider): void
```

Completely replaces an existing provider (matched by `name`).

```typescript
const config = contextMenuConfig.dataObjectTree

contextMenuRegistry.overrideSlotProvider(config.name, {
  name: 'rename', // must match existing provider name
  priority: config.priority.rename,
  useMenuItem: (context: DataObjectTreeContextMenuProps) => {
    const { t } = useTranslation()
    return {
      key: 'rename',
      label: t('data-object.custom-rename'),
      icon: <Icon value="edit" />,
      onClick: () => { /* custom rename */ }
    }
  }
})
```

### updateSlotProvider — Modify an Existing Item

```typescript
contextMenuRegistry.updateSlotProvider(
  slotName: string,
  name: string,
  updater: (existing: ContextMenuItemProvider) => ContextMenuItemProvider
): void
```

Wraps an existing provider. **Always call the existing hook first and respect a `null` return** so permission checks and built-in hide logic are preserved.

```typescript
const config = contextMenuConfig.assetTree

contextMenuRegistry.updateSlotProvider(config.name, 'delete', (existingItem) => ({
  ...existingItem,
  useMenuItem: (context: AssetTreeContextMenuProps) => {
    const existingMenuItem = existingItem.useMenuItem(context)
    const { confirm } = useFormModal()
    const { t } = useTranslation()

    if (existingMenuItem === null) {
      return null
    }

    const originalOnClick = existingMenuItem.onClick

    return {
      ...existingMenuItem,
      onClick: () => {
        confirm({
          title: t('delete.confirmation.title'),
          content: t('delete.confirmation.extra-warning'),
          okText: t('delete'),
          okButtonProps: { color: 'danger' },
          onOk: () => { originalOnClick?.() }
        })
      }
    }
  }
}))
```

---

## ContextMenuItemProvider Interface

```typescript
interface ContextMenuItemProvider {
  name: string            // Unique identifier within the slot
  priority?: number       // Lower = higher in menu (default: 999)
  useMenuItem: (context: ContextProps) => ItemType | null  // Return null to hide
}
```

### Key Rules

- **`name`** must be unique within a slot. Duplicate names cause the second registration to silently fail.
- **`priority`** defaults to `999`. Use `contextMenuConfig` constants to position relative to built-in items.
- **`useMenuItem`** is a React hook — you can call other hooks inside it (`useTranslation`, `useAlertModal`, etc.). Return `null` to hide.

### ItemType (Ant Design `MenuItemType`)

```typescript
interface ItemType {
  key: string                    // Unique key for React rendering
  label: string | ReactNode      // Display text
  icon?: ReactNode               // Icon component
  onClick?: () => void           // Click handler
  disabled?: boolean             // Greyed-out state
  danger?: boolean               // Red/danger styling
  children?: ItemType[]          // Submenu items
}
```

---

## Available Slots

### Document Slots

| Slot Constant | Slot Name | Context Type | Used In |
|---------------|-----------|--------------|---------|
| `contextMenuConfig.documentTree.name` | `document.tree` | `DocumentTreeContextMenuProps` | Document tree |
| `contextMenuConfig.documentTreeAdvanced.name` | `document.tree.advanced` | `DocumentTreeContextMenuProps` | Document tree (advanced submenu) |
| `contextMenuConfig.documentEditorToolbar.name` | `document.editor.toolbar` | `DocumentEditorToolbarContextMenuProps` | Document editor toolbar |

### Data Object Slots

| Slot Constant | Slot Name | Context Type | Used In |
|---------------|-----------|--------------|---------|
| `contextMenuConfig.dataObjectTree.name` | `data-object.tree` | `DataObjectTreeContextMenuProps` | Data object tree |
| `contextMenuConfig.dataObjectTreeAdvanced.name` | `data-object.tree.advanced` | `DataObjectTreeContextMenuProps` | Data object tree (advanced submenu) |
| `contextMenuConfig.dataObjectListGrid.name` | `data-object.list-grid` | `DataObjectListGridContextMenuProps` | Data object list/grid view |

### Asset Slots

| Slot Constant | Slot Name | Context Type | Used In |
|---------------|-----------|--------------|---------|
| `contextMenuConfig.assetTree.name` | `asset.tree` | `AssetTreeContextMenuProps` | Asset tree |
| `contextMenuConfig.assetListGrid.name` | `asset.list-grid` | `AssetListGridContextMenuProps` | Asset grid view |
| `contextMenuConfig.assetPreviewCard.name` | `asset.preview-card` | `AssetPreviewCardContextMenuProps` | Asset preview cards |
| `contextMenuConfig.assetEditorToolbar.name` | `asset.editor.toolbar` | `AssetEditorToolbarContextMenuProps` | Asset editor toolbar |

---

## Context Types (Props)

### Tree Context Props

All tree context menus share a similar shape:

```typescript
interface TreeContextMenuProps {
  target: {
    id: number          // Element ID
    type: string        // Element type (e.g., 'folder', 'page', 'image')
    key: string         // Tree node key
    parentId?: number   // Parent element ID
  }
}
```

Specific types:

```typescript
import {
  type DocumentTreeContextMenuProps,
  type DataObjectTreeContextMenuProps,
  type AssetTreeContextMenuProps
} from '@pimcore/studio-ui-bundle/modules/app'
```

### Grid/List Context Props

```typescript
import {
  type AssetListGridContextMenuProps,
  type DataObjectListGridContextMenuProps
} from '@pimcore/studio-ui-bundle/modules/app'
```

Grid context props include the row data for the right-clicked item.

### Editor Toolbar Context Props

```typescript
import {
  type DocumentEditorToolbarContextMenuProps,
  type AssetEditorToolbarContextMenuProps
} from '@pimcore/studio-ui-bundle/modules/app'
```

Toolbar context props include information about the currently open element.

---

## Priority System

- **Lower number = higher position** in the menu
- Default priority is `999` (bottom)
- Each slot config exposes a `priority` object with constants for built-in items:

```typescript
const config = contextMenuConfig.dataObjectTree

config.priority.open        // Priority of built-in "Open"
config.priority.rename      // Priority of built-in "Rename"
config.priority.delete      // Priority of built-in "Delete"
config.priority.copy        // ...
config.priority.paste
```

### Positioning Strategies

```typescript
// Above rename
priority: config.priority.rename - 1

// Below rename
priority: config.priority.rename + 1

// At the very top
priority: 1

// At the bottom (default)
// omit priority
```

---

## Grid/List Context Menus

Grids work the same way — just use the grid slot constant and its context type:

```typescript
const config = contextMenuConfig.assetListGrid

contextMenuRegistry.registerToSlot(config.name, {
  name: 'grid-download-asset',
  useMenuItem: (context: AssetListGridContextMenuProps) => {
    const { t } = useTranslation()
    return {
      key: 'grid-download-asset',
      label: t('asset.grid.download'),
      icon: <Icon value="download" />
    }
  }
})
```

---

## Common Mistakes to Avoid

### Forgetting to Return null for Conditional Hiding

```typescript
// DON'T - item still renders, just disabled
useMenuItem: (context) => ({
  key: 'image-resize',
  label: 'Resize Image',
  disabled: context.target.type !== 'image'
})

// DO - return null to hide entirely
useMenuItem: (context) => {
  if (context.target.type !== 'image') return null
  return { key: 'image-resize', label: 'Resize Image' }
}
```

### Duplicate Provider Names

Duplicate `name` values in the same slot cause the second registration to **silently fail**. Always use unique, bundle-prefixed names (e.g., `my-bundle-action-1`).

### Using Wrong Slot Name

```typescript
// DON'T - hardcoded string
contextMenuRegistry.registerToSlot('dataObject.tree', provider)

// DO - use contextMenuConfig constants
contextMenuRegistry.registerToSlot(contextMenuConfig.dataObjectTree.name, provider)
```

### Magic Priority Numbers

```typescript
// DON'T - magic number, may collide or drift
priority: 42

// DO - position relative to a known item
priority: config.priority.rename - 1
```

### Ignoring Existing Logic in updateSlotProvider

Always call `existingItem.useMenuItem(context)` and honor a `null` return. Skipping it breaks built-in permission checks and hide conditions.

### Hardcoded Labels

Use `useTranslation()` inside `useMenuItem`; never hardcode menu labels.

---

## Next Steps

- [**pimcore-studio-ui-widgets**](../pimcore-studio-ui-widgets/SKILL.md) - Building widget panels that context menu actions can open
- [**pimcore-studio-ui-tabs-editors**](../pimcore-studio-ui-tabs-editors/SKILL.md) - Editor tabs that context menu actions can interact with
- [**pimcore-studio-ui-using-sdk-in-bundles**](../pimcore-studio-ui-using-sdk-in-bundles/SKILL.md) - How bundles consume the SDK, register modules, and use DI
- [**pimcore-studio-ui-modals**](../pimcore-studio-ui-modals/SKILL.md) - Modal dialogs to trigger from context menu actions
- [**pimcore-studio-ui-i18n**](../pimcore-studio-ui-i18n/SKILL.md) - Translation system for menu item labels
