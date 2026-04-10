---
name: pimcore-studio-ui-icons
description: Using icons in Pimcore Studio UI - Icon component, custom icon registration, icon color groups, and SVG best practices
metadata:
  audience: pimcore-developers
  focus: ui-components
---

## What This Skill Covers

How to use icons in Pimcore Studio UI:
- **Icon component** - Rendering icons by name or path
- **Built-in icon library** - Common icon names by category
- **Registering custom icons** - Adding new icons via modules
- **Creating SVG icon components** - Proper SVG format for the icon library
- **Icon color groups** - Applying element and field definition colors

## When to Use This Skill

Use this when:
- Displaying icons in the UI (toolbars, trees, tabs, etc.)
- Creating or registering custom SVG icons
- Adding colored element-type icons (asset, data-object, document)
- Using sub-icon badges or sphere wrappers
- Customizing icon sizes, colors, or styles

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## Icon Component

The `Icon` component is the standard way to render icons in Pimcore Studio UI. It looks up icons by name from the icon library and renders the corresponding SVG.

```typescript
import { Icon } from '@pimcore/studio-ui-bundle/components'

// Render an icon by name
<Icon value="asset" />
```

For a direct URL or image path (e.g., from an API response), use `type="path"`:

```typescript
<Icon value="/admin/asset/get-image-thumbnail?id=42" type="path" />
```

## IconProps Reference

```typescript
interface IconProps {
  value: string              // Icon name from library (or URL if type='path')
  type?: 'name' | 'path'    // 'name' for registered icons, 'path' for image URL
  options?: React.SVGProps<SVGSVGElement>  // width, height, style, className, etc.
  className?: string         // Additional CSS class
  subIconName?: string       // Badge icon overlay (renders a smaller icon on top)
  subIconVariant?: 'default' | 'green'  // Sub-icon color variant
  sphere?: boolean           // Wrap icon in a circular background
  iconColorGroup?: string | string[]  // 'element', 'fieldDefinition', or custom
  colorToken?: string        // Direct Ant Design color token override
}
```

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `value` | `string` | (required) | Icon name from library, or image URL if `type='path'` |
| `type` | `'name' \| 'path'` | `'name'` | Whether `value` is a registered icon name or an image URL |
| `options` | `React.SVGProps<SVGSVGElement>` | `undefined` | SVG attributes: `width`, `height`, `style`, etc. |
| `className` | `string` | `undefined` | Additional CSS class name |
| `subIconName` | `string` | `undefined` | Name of a smaller badge icon overlaid on the main icon |
| `subIconVariant` | `'default' \| 'green'` | `'default'` | Color variant for the sub-icon badge |
| `sphere` | `boolean` | `false` | Wraps the icon in a circular background |
| `iconColorGroup` | `string \| string[]` | `undefined` | Maps icon name to a color via a registered color group |
| `colorToken` | `string` | `undefined` | Direct Ant Design design token for color override |

**Sub-icons** overlay a small badge icon on the main icon (e.g., a lock on an asset). **Sphere mode** wraps the icon in a circular background for visual emphasis in lists/cards.

## Common Built-In Icons

The icon library ships with a large set of built-in icons. Names are strings with no TypeScript autocomplete — verify exact names against the library source.

### Elements

| Icon Name | Description |
|-----------|-------------|
| `asset` | Asset element |
| `data-object` | Data object element |
| `document` | Document element |
| `folder` | Folder (used across all element types) |

### Actions

| Icon Name | Description |
|-----------|-------------|
| `edit` | Edit/modify |
| `delete` | Delete/remove |
| `download` | Download file |
| `upload` | Upload file |
| `refresh` | Refresh/reload |
| `save` | Save changes |
| `plus` | Add/create new |
| `close` | Close/dismiss |
| `copy` | Copy to clipboard |
| `paste` | Paste from clipboard |
| `cut` | Cut (move to clipboard) |

### Navigation

| Icon Name | Description |
|-----------|-------------|
| `chevron-down` | Expand/dropdown indicator |
| `chevron-left` | Navigate back/collapse |
| `chevron-right` | Navigate forward/expand |
| `chevron-up` | Collapse indicator |
| `arrow-left` | Navigate back |
| `arrow-right` | Navigate forward |

### Status

| Icon Name | Description |
|-----------|-------------|
| `check-circle` | Success/completed |
| `alert` | Warning/attention |
| `info` | Information |
| `lock` | Locked/protected |
| `unlock` | Unlocked/editable |
| `eye` | Visible/preview |
| `eye-off` | Hidden/not visible |

### Media

| Icon Name | Description |
|-----------|-------------|
| `image` | Image file |
| `video` | Video file |
| `audio` | Audio file |
| `pdf` | PDF document |

### UI

| Icon Name | Description |
|-----------|-------------|
| `settings` | Settings/configuration |
| `search` | Search/find |
| `filter` | Filter/refine |
| `drag-handle` | Drag and drop handle |
| `more` | More options (ellipsis) |
| `trash` | Trash/recycle bin |

## Registering Custom Icons

To add your own icons to the icon library, register them in a module's `onInit` method using the icon library service.

```typescript
// File: modules/custom-icon-extension.ts
import { type AbstractModule, container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import { type IconLibrary } from '@pimcore/studio-ui-bundle/modules/icon-library'
import { Rocket } from '../icons/rocket'

export const CustomIconExtension: AbstractModule = {
  onInit (): void {
    const iconLibrary = container.get<IconLibrary>(serviceIds.iconLibrary)
    iconLibrary.register({ name: 'rocket-example', component: Rocket })
  }
}
```

Once registered, use it like any built-in icon: `<Icon value="rocket-example" />`. Call `register` multiple times in the same `onInit` to register several icons.

## Creating SVG Icon Components

When creating SVG components for the icon library, follow this exact pattern:

```typescript
import React from 'react'

export interface RocketProps extends React.SVGProps<SVGSVGElement> {}

export const Rocket = (props: RocketProps): React.JSX.Element => {
  return (
    <svg viewBox="0 0 512 512" xmlns="http://www.w3.org/2000/svg" {...props}>
      <path d="M156.6 384.9L125.7 354..." fill='currentColor' />
    </svg>
  )
}
```

### Required Rules

1. **Extend `React.SVGProps<SVGSVGElement>`** - ensures TypeScript compatibility with all SVG attributes
2. **Spread `{...props}` on the `<svg>` element** - without this, `width`/`height`/`style`/`className` from `options` will not work
3. **Use `fill='currentColor'`** on all paths - allows the icon to inherit color from parent, themes, and color groups
4. **Include a `viewBox`** - required for proper scaling
5. **No hardcoded `width`/`height`** on the `<svg>` tag - let the consumer control size via props

For multi-path SVGs, apply `fill='currentColor'` to each path/circle/rect that should inherit color. When converting external SVGs, replace hardcoded `fill`/`stroke` colors with `currentColor` and remove fixed dimensions.

## Icon Color Groups

Icon color groups map icon names to specific colors. This is used to give element-type icons their characteristic colors (e.g., assets are green, data objects are mint).

```typescript
// Element color group - assigns colors to element type icons
<Icon value="asset" iconColorGroup="element" />
<Icon value="data-object" iconColorGroup="element" />
<Icon value="document" iconColorGroup="element" />

// Field definition color group - for field type icons in class editors
<Icon value="input" iconColorGroup="fieldDefinition" />

// Direct Ant Design color token override
<Icon value="alert" colorToken="colorWarning" />
<Icon value="check-circle" colorToken="colorSuccess" />
<Icon value="close" colorToken="colorError" />
<Icon value="info" colorToken="colorInfo" />

// Array fallback - first group with a mapping wins
<Icon value="asset" iconColorGroup={['customGroup', 'element']} />
```

### Color Group Reference

| Group | Purpose |
|-------|---------|
| `element` | Element type icons (asset, data-object, document, folder) |
| `fieldDefinition` | Field type icons in class editors |
| `colorToken` prop | Direct Ant Design design token override (`colorWarning`, `colorSuccess`, `colorError`, `colorInfo`, etc.) |

## Common Mistakes

### Wrong Icon Name (No Autocomplete)

The icon library does not provide TypeScript autocomplete. A typo simply renders nothing with no error:

```typescript
// WRONG - typo, renders nothing
<Icon value="asett" />

// CORRECT
<Icon value="asset" />
```

### Forgetting `fill='currentColor'` in Custom SVGs

```typescript
// WRONG - hardcoded color, won't respond to themes or color groups
<path d="M12 2..." fill="#333333" />

// CORRECT - inherits color from parent/theme
<path d="M12 2..." fill='currentColor' />
```

### Importing Icon from Ant Design Instead of SDK

```typescript
// WRONG - Ant Design Icon is a different component
import { Icon } from 'antd'

// CORRECT - Always import from Pimcore SDK
import { Icon } from '@pimcore/studio-ui-bundle/components'
```

The Pimcore `Icon` supports the icon library, color groups, sub-icons, and sphere mode. The Ant Design icon component does not.

### Using Inline SVG Instead of Registering

```typescript
// WRONG - not reusable, no icon library integration
<svg width="16" height="16" viewBox="0 0 24 24">
  <path d="M12 2..." fill="currentColor" />
</svg>

// CORRECT - register the icon, then use it
<Icon value="my-custom-icon" />
```

### Forgetting to Spread Props in Custom SVGs

```typescript
// WRONG - options.width/height will have no effect
<svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">

// CORRECT - props spread on svg element
<svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" {...props}>
```

### Hardcoding Icon Size on the SVG Element

```typescript
// WRONG - hardcoded width/height prevents resize via options
<svg width="24" height="24" viewBox="0 0 24 24" {...props}>

// CORRECT - only viewBox, consumer controls size
<svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" {...props}>
```

## Next Steps

- [**pimcore-studio-ui-buttons**](../pimcore-studio-ui-buttons/SKILL.md) - Using icons with Button, IconButton, and IconTextButton components
- [**pimcore-studio-ui-react-components**](../pimcore-studio-ui-react-components/SKILL.md) - Component structure and patterns for creating icon wrappers
