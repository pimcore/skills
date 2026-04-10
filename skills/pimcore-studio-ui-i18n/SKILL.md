---
name: pimcore-studio-ui-i18n
description: Internationalization and translation system in Pimcore Studio UI - useTranslation hook, translation keys, and localization
metadata:
  audience: pimcore-developers
  focus: i18n-translations
---

## What This Skill Covers

Internationalization (i18n) and translation system in Pimcore Studio UI:
- useTranslation hook for accessing translations
- Translation key structure and naming conventions
- Translation file locations and format
- Interpolation (dynamic values in translations)
- Pluralization
- Real-world examples from the codebase

## When to Use This Skill

Use this when:
- Adding any user-facing text to components
- Creating new features that need localization
- Adding error messages, labels, or UI text
- Working with forms and validation messages
- Building navigation items or menus

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

> **Note:** `useTranslation` is re-exported from `@pimcore/studio-ui-bundle/app`. Core code can also import directly from `react-i18next`. The hook at `@Pimcore/modules/translations/hooks/use-translation` is a different hook for CRUD operations on translation entities.

## useTranslation Hook

The `useTranslation` hook provides access to the translation function and current language.

### Basic Usage

```typescript
import { useTranslation } from '@pimcore/studio-ui-bundle/app'

export const MyComponent = () => {
  const { t } = useTranslation()
  
  return (
    <div>
      <h1>{t('my-feature.title')}</h1>
      <Button>{t('save')}</Button>
    </div>
  )
}
```

### Hook Return Value

```typescript
const { t, i18n } = useTranslation()

// t - Translation function
// i18n - i18next instance (rarely needed directly)
```

## Translation Key Structure

### 🚨 CRITICAL: Flat Dot Notation Required!

**ALWAYS use flat dot notation in YAML files. NEVER use nested structures!**

```yaml
# ✅ DO - Flat dot notation (CORRECT)
personalization.target-groups: Target Groups
personalization.target-group.add: Add Target Group
personalization.target-group.update.success: Target group updated successfully
personalization.target-group.update.error: Failed to update target group
personalization.target-group.configuration.name: Name
personalization.target-group.configuration.description: Description

# ❌ DON'T - Nested structure (WRONG!)
personalization:
  target-groups: Target Groups
  target-group:
    add: Add Target Group
    update:
      success: Target group updated successfully
```

### 🚨 CRITICAL: Always Maintain English Translations!

**Every translation key MUST have an English translation in `studio.en.yaml`!**

1. **English is the base language** - Always maintain `studio.en.yaml` in your bundle
2. **Never hardcode labels** - Always use `t('translation.key')` 
3. **Other languages are optional** - But English is required
4. **English acts as fallback** - If a translation is missing in another language, English is used

```typescript
// ❌ DON'T - Hardcoded text
<Button>Save</Button>
<h1>Target Groups</h1>

// ✅ DO - Always use translation keys
<Button>{t('save')}</Button>
<h1>{t('personalization.target-groups')}</h1>
```

### Naming Conventions

Translation keys follow a hierarchical structure using dot notation:

```
{bundle}.{module}.{component}.{specific-key}
```

This is the **recommended convention** for new bundles. However, some existing bundles use different conventions (e.g., `snake_case`, `camelCase`). When working with an existing bundle, **follow its existing convention** rather than imposing this pattern.

### Examples from Core

```yaml
# File: studio-ui-bundle/translations/studio.en.yaml

# Common actions
save: Save
delete: Delete
cancel: Cancel
refresh: Refresh
new: New
search: Search

# Form labels
form.label.new-item: New Item
form.validation.required: This field is required

# Navigation
navigation.quick-access: Quick Access
navigation.data-management: Data Management

# Element operations
element.delete.confirmation.title: Delete Element
element.delete.confirmation.text: Are you sure you want to delete this element?
element.tree.copy-success-description: '{{elementType}} "{{name}}" copied to clipboard'

# Toolbar
toolbar.save: Save
toolbar.publish: Publish
```

### Examples from Personalization Bundle

```yaml
# File: personalization-bundle/translations/studio.en.yaml

# Bundle-specific keys (FLAT DOT NOTATION)
personalization: Personalisation / Targeting
personalization.target-groups: Target Groups
personalization.target-group.add: Add Target Group
personalization.target-group.update.success: Target group updated successfully
personalization.target-group.update.error: Failed to update target group
personalization.target-group.delete.success: Target group deleted successfully
personalization.target-group.validation.message: Only alphanumeric characters, hyphens and underscores allowed
personalization.target-group.configuration.name: Name
personalization.target-group.configuration.description: Description
personalization.target-group.configuration.threshold: Threshold
personalization.target-group.configuration.active: Active
personalization.target-group.general-settings: General Settings

# Targeting rules
personalization.targeting-rules.navigation.title: Global Targeting Rules
```

## Translation Files Location

### Core Translations
```
studio-ui-bundle/translations/studio.{locale}.yaml
```

**Example:** `studio-ui-bundle/translations/studio.en.yaml`

### Bundle Translations
```
your-bundle/translations/studio.{locale}.yaml
```

**Example:** `personalization-bundle/translations/studio.en.yaml`

### 🚨 IMPORTANT: Work with English Only (Unless Instructed)

**By default, ONLY work with English translations (`studio.en.yaml`)!**

- English is the **base language** and **always required**
- Only add other language files if explicitly requested by the user
- Other locales use the same format: `studio.{locale}.yaml` (e.g., `studio.de.yaml`, `studio.fr.yaml`)
- When adding other languages, use the **same keys** with flat dot notation

## Translation File Format

Translation files use YAML format with **flat dot notation**:

```yaml
# ✅ CORRECT - Flat dot notation
save: Save
delete: Delete
personalization.target-groups: Target Groups
personalization.target-group.add: Add Target Group
personalization.target-group.update.success: Target group updated successfully
personalization.target-group.update.error: Failed to update target group
personalization.target-group.configuration.name: Name
personalization.target-group.configuration.description: Description

# With interpolation placeholders
element.tree.copy-success-description: '{{elementType}} "{{name}}" copied to clipboard'
  
# Pluralization (i18next v23 uses _one/_other suffixes)
notification.items-selected_one: '{{count}} item selected'
notification.items-selected_other: '{{count}} items selected'

# Multi-line text (use pipe for multi-line)
help.description: |
  This is a longer description
  that spans multiple lines.

# ❌ WRONG - Never use nested structure!
# personalization:
#   target-groups: Target Groups
```

## Interpolation (Dynamic Values)

### Basic Interpolation

```typescript
// Translation key:
// element.tree.copy-success-description: '{{elementType}} "{{name}}" copied to clipboard'

const { t } = useTranslation()

const message = t('element.tree.copy-success-description', {
  elementType: 'Document',
  name: 'Homepage'
})
// Result: 'Document "Homepage" copied to clipboard'
```

### Real-World Example from Copy/Paste

```typescript
// File: element/actions/copy-paste/tree-copy-paste-context.tsx
const messageApi = useMessage()
const { t } = useTranslation()

const copyNode = useCallback((node: TreeNodeProps, elementType: ElementType): void => {
  void messageApi.success(t('element.tree.copy-success-description', {
    elementType: t(elementType),  // Translate element type
    name: getNodeName(node),
    interpolation: { escapeValue: false }  // Don't escape HTML
  }))
}, [messageApi, t])
```

### With HTML Content

```typescript
// When translation contains HTML, disable escaping
const { t } = useTranslation()

const description = t('my-feature.description', {
  link: '<a href="#">Click here</a>',
  interpolation: { escapeValue: false }
})
```

## Pluralization

### Translation Key Format

```yaml
# Singular form (i18next v23 uses _one/_other suffixes)
item-selected_one: '{{count}} item selected'

# Plural form (append _other, NOT _plural)
item-selected_other: '{{count}} items selected'
```

### Using Pluralization

```typescript
const { t } = useTranslation()

const selectedCount = 5

const message = t('item-selected', { count: selectedCount })
// count = 1: "1 item selected"
// count > 1: "5 items selected"
```

### Real-World Example

```typescript
// Pagination total display
<Pagination
  showTotal={(total) => t('pagination.show-total', { total })}
/>

// Translation keys:
// pagination.show-total_one: '{{total}} item'
// pagination.show-total_other: '{{total}} items'
```

## Common Translation Patterns

### Pattern 1: Form Labels and Validation

```typescript
import { Form, Input } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from '@pimcore/studio-ui-bundle/app'

export const MyForm = () => {
  const { t } = useTranslation()
  
  return (
    <Form>
      <Form.Item
        label={t('form.label.name')}
        name="name"
        rules={[
          { required: true, message: t('form.validation.required') }
        ]}
      >
        <Input placeholder={t('form.placeholder.enter-name')} />
      </Form.Item>
    </Form>
  )
}
```

### Pattern 2: Button Labels

```typescript
import { Button } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from '@pimcore/studio-ui-bundle/app'

export const Toolbar = () => {
  const { t } = useTranslation()
  
  return (
    <>
      <Button onClick={handleCancel}>{t('cancel')}</Button>
      <Button type="primary" onClick={handleSave}>{t('save')}</Button>
    </>
  )
}
```

### Pattern 3: Modal Titles and Content

```typescript
import { useFormModal } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from '@pimcore/studio-ui-bundle/app'

export const useAddItem = () => {
  const modal = useFormModal()
  const { t } = useTranslation()
  
  const handleAdd = () => {
    modal.input({
      title: t('my-feature.add-item.title'),
      label: t('my-feature.add-item.label'),
      rule: {
        required: true,
        message: t('form.validation.required')
      },
      onOk: async (value) => {
        // Handle add
      }
    })
  }
  
  return { handleAdd }
}
```

### Pattern 4: Confirmation Dialogs

```typescript
import { useFormModal } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from '@pimcore/studio-ui-bundle/app'

export const useDeleteItem = () => {
  const modal = useFormModal()
  const { t } = useTranslation()
  
  const handleDelete = (itemName: string) => {
    modal.confirm({
      title: t('my-feature.delete.confirmation.title'),
      content: (
        <>
          <span>{t('my-feature.delete.confirmation.text')}</span>
          <br />
          <b>{itemName}</b>
        </>
      ),
      okText: t('delete'),
      cancelText: t('cancel'),
      onOk: async () => {
        // Handle delete
      }
    })
  }
  
  return { handleDelete }
}
```

### Pattern 5: Success/Error Messages

```typescript
import { useMessage } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from '@pimcore/studio-ui-bundle/app'

export const useSaveItem = () => {
  const messageApi = useMessage()
  const { t } = useTranslation()
  
  const handleSave = async () => {
    try {
      await saveItem()
      void messageApi.success(t('my-feature.save.success'))
    } catch (error) {
      void messageApi.error(t('my-feature.save.error'))
    }
  }
  
  return { handleSave }
}
```

### Pattern 6: Table/Grid Column Headers

```typescript
import { createColumnHelper } from '@tanstack/react-table'
import { useTranslation } from '@pimcore/studio-ui-bundle/app'

export const useColumns = () => {
  const { t } = useTranslation()
  const columnHelper = createColumnHelper<DataType>()
  
  return [
    columnHelper.accessor('name', {
      header: t('my-feature.columns.name')
    }),
    columnHelper.accessor('status', {
      header: t('my-feature.columns.status')
    })
  ]
}
```

### Pattern 7: Navigation Items

```typescript
// In module initialization
const mainNavRegistry = container.get<MainNavRegistry>(serviceIds.mainNavRegistry)

mainNavRegistry.registerMainNavItem({
  path: 'MyGroup/MyFeature',
  label: 'my-feature.navigation.title',  // Translation key
  order: 5,
  widgetConfig: {
    name: 'My Feature',
    id: 'my-feature',
    component: 'my-feature',
    config: {
      translationKey: 'my-feature.navigation.title',  // Translation key
      icon: {
        type: 'name',
        value: 'my-icon'
      }
    }
  }
})
```

## Real-World Example: Target Group Form

```typescript
// File: personalization-bundle/.../target-group-detail.tsx
import { Form, Input, TextArea, InputNumber, Switch } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from '@pimcore/studio-ui-bundle/app'

export const TargetGroupDetail = ({ id }: { id: number }) => {
  const { t } = useTranslation()
  const [form] = Form.useForm()
  
  return (
    <Form form={form} layout="vertical">
      <Panel title={t('personalization.target-group.general-settings')}>
        <Form.Item
          label={t('personalization.target-group.configuration.name')}
          name="name"
        >
          <Input 
            disabled 
            placeholder={t('personalization.target-group.configuration.name')} 
          />
        </Form.Item>
        
        <Form.Item
          label={t('personalization.target-group.configuration.description')}
          name="description"
        >
          <TextArea
            placeholder={t('personalization.target-group.configuration.description')}
            rows={4}
          />
        </Form.Item>
        
        <Form.Item
          label={t('personalization.target-group.configuration.threshold')}
          name="threshold"
        >
          <InputNumber />
        </Form.Item>
        
        <Form.Item
          name="active"
          valuePropName="checked"
        >
          <Switch 
            labelRight={t('personalization.target-group.configuration.active')} 
          />
        </Form.Item>
      </Panel>
    </Form>
  )
}
```

## Common Mistakes

### ❌ Hardcoded Text (Most Common Mistake!)

**NEVER hardcode user-facing text. ALWAYS use translation keys!**

```typescript
// ❌ DON'T - Hardcoded text (NEVER DO THIS!)
<Button>Save</Button>
<h1>Settings</h1>
<p>Are you sure you want to delete this item?</p>
```

```typescript
// ✅ DO - Always use translation keys
const { t } = useTranslation()

<Button>{t('save')}</Button>
<h1>{t('settings.title')}</h1>
<p>{t('delete.confirmation.text')}</p>
```

### ❌ Using Nested YAML Structure

```yaml
# ❌ DON'T - Nested structure
personalization:
  target-groups: Target Groups
  target-group:
    add: Add Target Group

# ✅ DO - Flat dot notation
personalization.target-groups: Target Groups
personalization.target-group.add: Add Target Group
```

### ❌ Missing English Translations

```yaml
# ❌ DON'T - Only German translations, no English base
# File: studio.de.yaml
mein-feature.titel: Mein Feature

# ✅ DO - Always have English in studio.en.yaml first
# File: studio.en.yaml
my-feature.title: My Feature

# File: studio.de.yaml
my-feature.title: Mein Feature
```

### ❌ Not Using Interpolation

```typescript
// DON'T DO THIS
const message = t('saved') + ' ' + itemName

// DO THIS
const message = t('saved-item', { name: itemName })
// Translation: 'saved-item': 'Saved {{name}} successfully'
```

### ❌ Missing Translation Keys

```typescript
// DON'T - Key doesn't exist in translation file
t('non-existent-key')  // Shows the key itself

// DO - Always add keys to translation files first
// Then use them in code
t('my-feature.existing-key')
```

### ❌ Incorrect Pluralization

```typescript
// DON'T - Manual plural handling
const message = count === 1 ? t('item') : t('items')

// DO - Use count parameter (i18next v23 uses _one/_other suffixes)
const message = t('item', { count })
// Translation keys:
// item_one: '{{count}} item'
// item_other: '{{count}} items'
```

## Adding New Translations

### 🚨 CRITICAL: Workflow for Adding Translations

**Step 1: ALWAYS Add English First (REQUIRED)**

Add translations to `studio.en.yaml` in your bundle using **flat dot notation**:

```yaml
# File: your-bundle/translations/studio.en.yaml

# ✅ CORRECT - Flat dot notation
my-feature.title: My Feature
my-feature.description: This is my feature description
my-feature.add-item.title: Add Item
my-feature.add-item.label: Item Name
my-feature.save.success: Item saved successfully
my-feature.save.error: Failed to save item

# ❌ WRONG - Never use nested structure!
# my-feature:
#   title: My Feature
```

**Step 2: Use in Components (Never Hardcode!)**

```typescript
import { useTranslation } from '@pimcore/studio-ui-bundle/app'

export const MyFeature = () => {
  const { t } = useTranslation()
  
  return (
    <div>
      {/* ✅ DO - Always use translation keys */}
      <h1>{t('my-feature.title')}</h1>
      <p>{t('my-feature.description')}</p>
      
      {/* ❌ DON'T - Never hardcode */}
      {/* <h1>My Feature</h1> */}
    </div>
  )
}
```

**Step 3: Add Other Languages (Optional)**

**German:** `studio.de.yaml`
```yaml
# ✅ CORRECT - Flat dot notation, same keys as English
my-feature.title: Meine Funktion
my-feature.description: Dies ist meine Funktionsbeschreibung
my-feature.add-item.title: Element hinzufügen
my-feature.add-item.label: Elementname

# ❌ WRONG - Never use nested structure!
```

**French:** `studio.fr.yaml`
```yaml
# ✅ CORRECT - Flat dot notation
my-feature.title: Ma Fonctionnalité
my-feature.description: Ceci est ma description de fonctionnalité
```

### Important Notes on Translation Files

1. **English (`studio.en.yaml`) is MANDATORY** - Must contain all translation keys
2. **Other languages are OPTIONAL** - But use same keys and flat dot notation
3. **Flat dot notation ALWAYS** - Never use nested YAML structures
4. **Fallback to English** - If a translation is missing, English is used automatically
5. **Never hardcode text** - Always use `t('translation.key')` in components

## Next Steps

- [**pimcore-studio-ui-forms-antd**](../pimcore-studio-ui-forms-antd/SKILL.md) - Form components and validation with translations
- [**pimcore-studio-ui-notifications-toasts**](../pimcore-studio-ui-notifications-toasts/SKILL.md) - Success/error messages with translations
- [**pimcore-studio-ui-modals**](../pimcore-studio-ui-modals/SKILL.md) - Modals and confirmations with translations
- [**pimcore-studio-ui-navigation**](../pimcore-studio-ui-navigation/SKILL.md) - Adding navigation items with translation keys
