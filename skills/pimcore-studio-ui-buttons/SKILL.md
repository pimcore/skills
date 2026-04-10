---
name: pimcore-studio-ui-buttons
description: Using button components in Pimcore Studio UI - Button, IconButton, IconTextButton, DropdownButton, and ButtonGroup
metadata:
  audience: pimcore-developers
  focus: ui-components-buttons
---

## What This Skill Covers

How to use button components in Pimcore Studio UI:
- **Button** - Standard text buttons (primary, default, link, text types)
- **IconButton** - Icon-only buttons with tooltips (always include tooltips!)
- **IconTextButton** - Buttons combining icon + text
- **DropdownButton** - Buttons with dropdown chevron
- **ButtonGroup** - Grouping multiple related buttons
- Button loading states and disabled states
- Color variants (primary, secondary, danger)
- When to use each button type

## When to Use This Skill

Use this when:
- Adding action buttons to toolbars, forms, or modals
- Creating icon-only toolbar buttons (always with tooltips)
- Building button groups for related actions
- Implementing save, cancel, delete, or action buttons
- Creating dropdown menus with button triggers

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## Writing Style for Button Labels

Follow these capitalization and wording guidelines for all button text:

### Sentence Case for Actions

**Buttons and interactive elements use Sentence case** - only the first word is capitalized:

```typescript
// ✅ CORRECT - Sentence case for buttons
<Button>{t('save')}</Button>                    // "Save"
<Button>{t('save-draft')}</Button>              // "Save draft"
<Button>{t('merge-version')}</Button>           // "Merge version"
<Button>{t('review-changes')}</Button>          // "Review changes"
<Button>{t('export-csv')}</Button>              // "Export CSV"
<Button>{t('confirm-delete')}</Button>          // "Confirm delete"

// ❌ WRONG - Don't use Title Case for buttons
<Button>{t('save-draft')}</Button>              // Not "Save Draft"
<Button>{t('review-changes')}</Button>          // Not "Review Changes"
```

This applies to:
- Buttons (all types)
- Modal action buttons
- Toolbar action buttons
- Form submit buttons
- Confirmation buttons

### Action-Oriented Wording

**Use verb-first phrasing** for all actionable interface elements:

```typescript
// ✅ CORRECT - Action-oriented (verb-first)
<Button>{t('export-csv')}</Button>              // "Export CSV"
<Button>{t('download-file')}</Button>           // "Download file"
<Button>{t('create-new')}</Button>              // "Create new"
<Button>{t('save-changes')}</Button>            // "Save changes"
<IconButton 
  tooltip={{ title: t('refresh-data') }}        // "Refresh data"
/>

// ❌ WRONG - Noun-based, not action-oriented
<Button>{t('csv-export')}</Button>              // Not "CSV Export"
<Button>{t('file-download')}</Button>           // Not "File Download"
<Button>{t('new-item')}</Button>                // Not "New Item"
```

### Translation Key Naming

Match your translation keys to the final output (sentence case, verb-first):

```yaml
# ✅ CORRECT - Translation keys in studio.en.yaml
toolbar.save: Save
toolbar.save-draft: Save draft
toolbar.export-csv: Export CSV
toolbar.create-new: Create new
modal.confirm-delete: Confirm delete
modal.review-changes: Review changes

# ❌ WRONG - Don't use title case or noun-first
# toolbar.save-draft: Save Draft
# toolbar.csv-export: CSV Export
```

### Writing Style Quick Reference

| Element | Case | Example | Translation Key |
|---------|------|---------|-----------------|
| Buttons | Sentence case | "Save draft" | `save-draft` |
| Toolbar actions | Sentence case | "Export CSV" | `export-csv` |
| Modal buttons | Sentence case | "Confirm delete" | `confirm-delete` |
| Icon tooltips | Sentence case | "Refresh data" | `refresh-data` |

**Remember:**
- Buttons/actions = Sentence case + verb-first
- Always translate, never hardcode

**For navigation labels (Title Case), see the pimcore-studio-ui-navigation skill.**
## Button Component

Standard button with text, the most common button type.

### Button Types

```typescript
type?: 'primary' | 'default' | 'link' | 'text' | 'action'
```

- **primary** - Main action button (blue, filled)
- **default** - Secondary action (gray border)
- **link** - Text-like button with no border
- **text** - Plain text button with hover state
- **action** - Special action style

### Color Variants

```typescript
color?: 'default' | 'primary' | 'secondary' | 'danger'
```

- **primary** - Blue emphasis
- **secondary** - Gray secondary actions
- **danger** - Red destructive actions (delete, remove)

### Basic Usage

```typescript
import { Button } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from 'react-i18next'

export const MyComponent = (): React.JSX.Element => {
  const { t } = useTranslation()

  return (
    <Button
      onClick={ handleSave }
      type="primary"
    >
      {t('toolbar.save')}
    </Button>
  )
}
```

### Button Props

```typescript
interface ButtonProps {
  type?: 'primary' | 'default' | 'link' | 'text' | 'action'
  color?: 'default' | 'primary' | 'secondary' | 'danger'
  loading?: boolean              // Shows spinner, locks width
  disabled?: boolean
  onClick?: () => void
  children: React.ReactNode      // Button text content
  className?: string
  htmlType?: 'button' | 'submit' | 'reset'
  // ... extends all Ant Design ButtonProps
}
```

### Loading State

Buttons support loading states with a spinner. The button width is locked during loading to prevent layout shift:

```typescript
const [updateAsset, { isLoading }] = useAssetUpdateByIdMutation()

return (
  <Button
    disabled={ isLoading }
    loading={ isLoading }
    onClick={ handleSave }
    type="primary"
  >
    {t('toolbar.save-and-publish')}
  </Button>
)
```

### Real-World Example: Save Button

```typescript
// File: asset/editor/toolbar/save-button/save-button.tsx
import { Button } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from 'react-i18next'
import { checkElementPermission } from '@pimcore/studio-ui-bundle/modules/element'

export const EditorToolbarSaveButton = (): React.JSX.Element => {
  const { t } = useTranslation()
  const { asset } = useAssetDraft(id)
  const [saveAsset, { isLoading }] = useAssetUpdateByIdMutation()

  return (
    <>
      {checkElementPermission(asset?.permissions, 'publish') && (
        <Button
          disabled={ isLoading }
          loading={ isLoading }
          onClick={ onSaveClick }
          type="primary"
        >
          {t(asset?.type === 'folder' ? 'toolbar.save' : 'toolbar.save-and-publish')}
        </Button>
      )}
    </>
  )
}
```

## IconButton Component

Icon-only buttons for toolbars and compact UIs.

### 🚨 CRITICAL: Always Include Tooltips!

**IconButtons MUST always have tooltips for accessibility!**

```typescript
// ❌ DON'T - No tooltip
<IconButton
  icon={ { value: 'delete' } }
  onClick={ handleDelete }
/>

// ✅ DO - Always include tooltip
<IconButton
  icon={ { value: 'delete' } }
  onClick={ handleDelete }
  tooltip={ { title: t('delete') } }
/>
```

### IconButton Props

```typescript
interface IconButtonProps {
  icon: IconProps                   // Icon configuration (required)
  tooltip?: TooltipProps            // ALWAYS provide this!
  theme?: 'primary' | 'secondary'   // Visual theme
  variant?: 'minimal' | 'static'    // Style variant
  size?: 'small' | 'middle' | 'large'
  hideShadow?: boolean              // Hide box shadow
  type?: ButtonProps['type']        // Button type (default: 'link')
  onClick?: () => void
  disabled?: boolean
  loading?: boolean
}
```

### Icon Configuration

```typescript
icon={ { 
  value: 'icon-name',        // Icon name from icon library
  options: {
    width: 16,               // Optional size override
    height: 16
  }
} }
```

Common icons: `'refresh'`, `'delete'`, `'edit'`, `'plus'`, `'close'`, `'save'`, `'download'`, `'upload'`, `'chevron-down'`, `'chevron-right'`

### Theme Variants

```typescript
// Primary theme - blue hover
<IconButton
  icon={ { value: 'edit' } }
  theme="primary"
  tooltip={ { title: t('edit') } }
/>

// Secondary theme - gray hover  
<IconButton
  icon={ { value: 'close' } }
  theme="secondary"
  tooltip={ { title: t('close') } }
/>
```

### Real-World Example: Toolbar Refresh Button

```typescript
import { IconButton } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from 'react-i18next'

export const TreeContainer = (): React.JSX.Element => {
  const { t } = useTranslation()
  const { refetch, isLoading } = useQuery()

  return (
    <Toolbar>
      <IconButton
        icon={ { value: 'refresh' } }
        loading={ isLoading }
        onClick={ async () => await refetch() }
        tooltip={ { title: t('refresh') } }
      />
    </Toolbar>
  )
}
```

### Size Variants

```typescript
// Small icon button (14px icon)
<IconButton
  icon={ { value: 'delete' } }
  size="small"
  tooltip={ { title: t('delete') } }
/>

// Medium/default (16px icon)
<IconButton
  icon={ { value: 'delete' } }
  tooltip={ { title: t('delete') } }
/>

// Large (custom size via icon options)
<IconButton
  icon={ { 
    value: 'delete',
    options: { width: 24, height: 24 }
  } }
  tooltip={ { title: t('delete') } }
/>
```

## IconTextButton Component

Buttons that combine an icon with text. Perfect for toolbar actions that need both visual and textual clarity.

### IconTextButton Props

```typescript
interface IconTextButtonProps extends Omit<ButtonProps, 'icon'> {
  icon: IconProps                      // Icon configuration (required)
  iconPlacement?: 'left' | 'right'    // Icon position (default: 'left')
  children: React.ReactNode            // Button text
}
```

### Basic Usage

```typescript
import { IconTextButton } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from 'react-i18next'

export const MyToolbar = (): React.JSX.Element => {
  const { t } = useTranslation()

  return (
    <IconTextButton
      icon={ { value: 'plus' } }
      onClick={ handleCreate }
      type="primary"
    >
      {t('toolbar.create')}
    </IconTextButton>
  )
}
```

### Icon Placement

```typescript
// Icon on the left (default)
<IconTextButton
  icon={ { value: 'save' } }
>
  {t('save')}
</IconTextButton>

// Icon on the right
<IconTextButton
  icon={ { value: 'chevron-down' } }
  iconPlacement="right"
>
  {t('more-options')}
</IconTextButton>
```

### Real-World Example: Create New Button

```typescript
// File: widget-editor/components/tree/tree-container.tsx
<Toolbar justify="space-between">
  <IconButton
    icon={ { value: 'refresh' } }
    onClick={ refetch }
    tooltip={ { title: t('refresh') } }
  />

  <IconTextButton
    icon={ { value: 'new' } }
    loading={ isLoading }
    onClick={ createWidget }
  >
    {t('toolbar.new')}
  </IconTextButton>
</Toolbar>
```

### Button Types with Icons

```typescript
// Primary action
<IconTextButton
  icon={ { value: 'save' } }
  type="primary"
>
  {t('save')}
</IconTextButton>

// Link style
<IconTextButton
  icon={ { value: 'download' } }
  type="link"
>
  {t('download')}
</IconTextButton>

// Danger action
<IconTextButton
  color="danger"
  icon={ { value: 'delete' } }
  type="primary"
>
  {t('delete')}
</IconTextButton>
```

## DropdownButton Component

Button with a dropdown chevron icon on the right. Commonly used for dropdown menus and "More" actions.

### DropdownButton Props

```typescript
interface DropdownButtonProps extends Omit<IconTextButtonProps, 'icon'> {
  icon?: IconProps  // Optional custom icon (chevron-down is default)
  children: React.ReactNode
}
```

### Basic Usage

```typescript
import { DropdownButton } from '@pimcore/studio-ui-bundle/components'
import { Dropdown } from 'antd'

export const ActionsDropdown = (): React.JSX.Element => {
  const { t } = useTranslation()

  const items = [
    { key: 'edit', label: t('edit') },
    { key: 'delete', label: t('delete'), danger: true }
  ]

  return (
    <Dropdown menu={ { items } }>
      <DropdownButton>
        {t('more-actions')}
      </DropdownButton>
    </Dropdown>
  )
}
```

### With Custom Icon

```typescript
// Override the default chevron-down
<DropdownButton
  icon={ { value: 'settings' } }
>
  {t('settings')}
</DropdownButton>
```

### Real-World Pattern: Batch Actions

```typescript
import { DropdownButton } from '@pimcore/studio-ui-bundle/components'
import { Dropdown } from 'antd'
import type { MenuProps } from 'antd'

export const BatchActionsDropdown = (): React.JSX.Element => {
  const { t } = useTranslation()
  const selectedItems = useSelectedItems()

  const menuItems: MenuProps['items'] = [
    {
      key: 'export',
      label: t('batch.export'),
      icon: <Icon value="download" />
    },
    {
      key: 'delete',
      label: t('batch.delete'),
      icon: <Icon value="delete" />,
      danger: true
    }
  ]

  const handleMenuClick: MenuProps['onClick'] = ({ key }) => {
    switch (key) {
      case 'export':
        handleExport(selectedItems)
        break
      case 'delete':
        handleDelete(selectedItems)
        break
    }
  }

  return (
    <Dropdown
      disabled={ selectedItems.length === 0 }
      menu={ { items: menuItems, onClick: handleMenuClick } }
    >
      <DropdownButton disabled={ selectedItems.length === 0 }>
        {t('batch-actions')} ({selectedItems.length})
      </DropdownButton>
    </Dropdown>
  )
}
```

## ButtonGroup Component

Groups multiple related buttons together with consistent spacing.

### ButtonGroup Props

```typescript
interface ButtonGroupProps {
  items: ReactElement[]         // Array of button elements
  withSeparator?: boolean       // Add visual separator between buttons
  noSpacing?: boolean           // Use Ant Design's compact group style
}
```

### Basic Usage

```typescript
import { ButtonGroup, Button, IconButton } from '@pimcore/studio-ui-bundle/components'

export const EditorToolbar = (): React.JSX.Element => {
  const { t } = useTranslation()

  return (
    <ButtonGroup
      items={ [
        <Button
          key="save"
          onClick={ handleSave }
          type="primary"
        >
          {t('save')}
        </Button>,
        <Button
          key="cancel"
          onClick={ handleCancel }
        >
          {t('cancel')}
        </Button>
      ] }
    />
  )
}
```

### With Separators

```typescript
// Add visual separators between button groups
<ButtonGroup
  items={ [
    <IconButton
      icon={ { value: 'undo' } }
      key="undo"
      onClick={ handleUndo }
      tooltip={ { title: t('undo') } }
    />,
    <IconButton
      icon={ { value: 'redo' } }
      key="redo"
      onClick={ handleRedo }
      tooltip={ { title: t('redo') } }
    />
  ] }
  withSeparator
/>
```

### Compact Style (No Spacing)

```typescript
// Use Ant Design's compact button group (no gaps)
<ButtonGroup
  items={ [
    <IconButton
      icon={ { value: 'align-left' } }
      key="left"
      tooltip={ { title: t('align-left') } }
    />,
    <IconButton
      icon={ { value: 'align-center' } }
      key="center"
      tooltip={ { title: t('align-center') } }
    />,
    <IconButton
      icon={ { value: 'align-right' } }
      key="right"
      tooltip={ { title: t('align-right') } }
    />
  ] }
  noSpacing
/>
```

### Real-World Pattern: Action Groups

```typescript
import { ButtonGroup, Button, IconButton } from '@pimcore/studio-ui-bundle/components'

export const FormActions = (): React.JSX.Element => {
  const { t } = useTranslation()

  // Group 1: Main actions
  const mainActions = (
    <ButtonGroup
      items={ [
        <Button
          key="save"
          loading={ isSaving }
          onClick={ handleSave }
          type="primary"
        >
          {t('save')}
        </Button>,
        <Button
          key="cancel"
          onClick={ handleCancel }
        >
          {t('cancel')}
        </Button>
      ] }
    />
  )

  // Group 2: Secondary actions
  const secondaryActions = (
    <ButtonGroup
      items={ [
        <IconButton
          icon={ { value: 'delete' } }
          key="delete"
          onClick={ handleDelete }
          tooltip={ { title: t('delete') } }
        />,
        <IconButton
          icon={ { value: 'duplicate' } }
          key="duplicate"
          onClick={ handleDuplicate }
          tooltip={ { title: t('duplicate') } }
        />
      ] }
      withSeparator
    />
  )

  return (
    <Toolbar justify="space-between">
      {mainActions}
      {secondaryActions}
    </Toolbar>
  )
}
```

## Common Button Patterns

### Pattern 1: Save/Cancel Button Pair

```typescript
<ButtonGroup
  items={ [
    <Button
      disabled={ isLoading }
      key="save"
      loading={ isLoading }
      onClick={ handleSave }
      type="primary"
    >
      {t('save')}
    </Button>,
    <Button
      disabled={ isLoading }
      key="cancel"
      onClick={ handleCancel }
    >
      {t('cancel')}
    </Button>
  ] }
/>
```

### Pattern 2: Toolbar Icon Buttons

```typescript
<Toolbar>
  <IconButton
    icon={ { value: 'refresh' } }
    loading={ isLoading }
    onClick={ refetch }
    tooltip={ { title: t('refresh') } }
  />
  
  <IconButton
    icon={ { value: 'settings' } }
    onClick={ openSettings }
    tooltip={ { title: t('settings') } }
  />
  
  <IconTextButton
    icon={ { value: 'plus' } }
    onClick={ handleCreate }
    type="primary"
  >
    {t('create-new')}
  </IconTextButton>
</Toolbar>
```

### Pattern 3: Confirmation Actions

```typescript
// Destructive action with danger color
<Button
  color="danger"
  onClick={ showDeleteConfirmation }
  type="primary"
>
  {t('delete')}
</Button>

// Inside modal confirmation
<ButtonGroup
  items={ [
    <Button
      key="confirm"
      loading={ isDeleting }
      onClick={ handleConfirmDelete }
      type="primary"
      color="danger"
    >
      {t('confirm-delete')}
    </Button>,
    <Button
      disabled={ isDeleting }
      key="cancel"
      onClick={ closeModal }
    >
      {t('cancel')}
    </Button>
  ] }
/>
```

### Pattern 4: Permission-Based Buttons

```typescript
import { checkElementPermission } from '@pimcore/studio-ui-bundle/modules/element'

// Only show button if user has permission
{checkElementPermission(element?.permissions, 'publish') && (
  <Button
    onClick={ handlePublish }
    type="primary"
  >
    {t('publish')}
  </Button>
)}

// Disable button if no permission
<Button
  disabled={ !checkElementPermission(element?.permissions, 'delete') }
  onClick={ handleDelete }
  color="danger"
>
  {t('delete')}
</Button>
```

### Pattern 5: Loading States with Multiple Operations

```typescript
const [saveAsset, { isLoading: isSaving }] = useAssetUpdateByIdMutation()
const { saveSchedules, isLoading: isSavingSchedules } = useSaveSchedules()

const isLoading = isSaving || isSavingSchedules

return (
  <Button
    disabled={ isLoading }
    loading={ isLoading }
    onClick={ handleSave }
    type="primary"
  >
    {t('save')}
  </Button>
)
```

## When to Use Each Button Type

### Use `<Button>` when:
- Primary or secondary actions need text labels
- Form submit/cancel buttons
- Modal action buttons
- Clear text description is important

### Use `<IconButton>` when:
- Space is limited (toolbars, table rows)
- Action is universally recognized by icon
- Multiple similar actions in a row
- **ALWAYS with a tooltip!**

### Use `<IconTextButton>` when:
- Action benefits from both icon and text
- Primary toolbar actions
- Creating/adding new items
- Icon reinforces the text meaning

### Use `<DropdownButton>` when:
- Multiple related actions in a menu
- "More actions" overflow menus
- Batch operations
- Settings/options menus

### Use `<ButtonGroup>` when:
- Multiple related buttons should be visually grouped
- Save/Cancel pairs
- Toolbar button sets
- Action groups with clear relationships

## Common Mistakes

### ❌ IconButton Without Tooltip

```typescript
// DON'T - No tooltip makes it inaccessible
<IconButton
  icon={ { value: 'delete' } }
  onClick={ handleDelete }
/>

// DO - Always include tooltip
<IconButton
  icon={ { value: 'delete' } }
  onClick={ handleDelete }
  tooltip={ { title: t('delete') } }
/>
```

### ❌ Wrong Button Type for Use Case

```typescript
// DON'T - Using primary for cancel
<Button
  onClick={ handleCancel }
  type="primary"
>
  {t('cancel')}
</Button>

// DO - Use default for secondary actions
<Button
  onClick={ handleCancel }
>
  {t('cancel')}
</Button>
```

### ❌ Not Handling Loading States

```typescript
// DON'T - No loading/disabled state
<Button onClick={ handleSave }>
  {t('save')}
</Button>

// DO - Show loading and disable during operation
<Button
  disabled={ isLoading }
  loading={ isLoading }
  onClick={ handleSave }
>
  {t('save')}
</Button>
```

### ❌ Forgetting Translation Keys

```typescript
// DON'T - Hardcoded text
<Button onClick={ handleSave }>
  Save
</Button>

// DO - Always use i18n
<Button onClick={ handleSave }>
  {t('save')}
</Button>
```

### ❌ Missing Keys in ButtonGroup

```typescript
// DON'T - No keys for array items
<ButtonGroup
  items={ [
    <Button onClick={ handleSave }>Save</Button>,
    <Button onClick={ handleCancel }>Cancel</Button>
  ] }
/>

// DO - Always provide keys
<ButtonGroup
  items={ [
    <Button key="save" onClick={ handleSave }>{t('save')}</Button>,
    <Button key="cancel" onClick={ handleCancel }>{t('cancel')}</Button>
  ] }
/>
```

## Quick Reference

### Button Component Comparison

| Component | Use Case | Always Include |
|-----------|----------|----------------|
| `Button` | Text actions, forms, modals | Translation |
| `IconButton` | Compact toolbar actions | **Tooltip** + Translation |
| `IconTextButton` | Primary toolbar actions | Icon + Translation |
| `DropdownButton` | Menu triggers | Translation |
| `ButtonGroup` | Related action groups | Keys for items |

### Common Button Types

| Type | Appearance | Use For |
|------|------------|---------|
| `primary` | Blue filled | Main/primary actions |
| `default` | Gray border | Secondary actions |
| `link` | Text with hover | Tertiary/inline actions |
| `text` | Plain text | Minimal emphasis actions |

### Common Button Colors

| Color | Use For |
|-------|---------|
| `primary` | Positive/main actions |
| `secondary` | Alternative actions |
| `danger` | Destructive actions (delete, remove) |

## Next Steps

- [**pimcore-studio-ui-i18n**](../pimcore-studio-ui-i18n/SKILL.md) - Translation system for button labels
- [**pimcore-studio-ui-layout-components**](../pimcore-studio-ui-layout-components/SKILL.md) - Toolbar and layout context
- [**pimcore-studio-ui-modals**](../pimcore-studio-ui-modals/SKILL.md) - Using buttons in modal dialogs
- [**pimcore-studio-ui-forms-antd**](../pimcore-studio-ui-forms-antd/SKILL.md) - Form submit buttons with FormKit
