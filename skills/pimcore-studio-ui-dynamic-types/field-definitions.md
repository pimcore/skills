# Field Definitions in Dynamic Types

Comprehensive guide to implementing field definitions in Pimcore Studio UI's dynamic type system.

## Overview

Field definitions determine how data object fields are configured, edited, and rendered. Each field type (Input, Textarea, Select, etc.) requires:

1. **Type Registration** - Register the field type in the dynamic type registry
2. **Form Fields Component** - Configure the field settings form
3. **Grid Cell Renderer** - Display field values in data grids (see `grid-cells.md`)
4. **Detail View** - Render field in edit mode

## Field Definition Structure

```typescript
interface DynamicTypeFieldDefinition {
  type: string                          // Field type identifier (e.g., 'input', 'textarea')
  formFields: React.ComponentType       // Form configuration component
  gridCell?: React.ComponentType        // Grid cell renderer (optional)
  detailView?: React.ComponentType      // Detail view component (optional)
}
```

## Registering a Field Definition Type

```typescript
// File: field-definitions/index.tsx
import { container } from '@pimcore/studio-ui-bundle/app'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import { type DynamicTypeFieldDefinitionRegistry } from '@pimcore/studio-ui-bundle/modules/field-definitions'

// Get registry
const registry = container.get<DynamicTypeFieldDefinitionRegistry>(
  serviceIds['DynamicTypes/FieldDefinitionRegistry']
)

// Register field type
import { InputFormFields } from './types/input/input-form-fields'
import { InputGridCell } from './types/input/input-grid-cell'

registry.register({
  type: 'input',
  formFields: InputFormFields,
  gridCell: InputGridCell
})
```

## Creating Form Fields Component

Form fields component defines the configuration form for the field definition:

```typescript
// File: types/input/input-form-fields.tsx
import React from 'react'
import { FormKit, FormItem, Input, InputNumber, Checkbox } from '@pimcore/studio-ui-bundle/components'

interface InputFormFieldsProps {
  form: FormInstance
  fieldName: string[]  // Path in form (e.g., ['settings', 'input'])
}

export const InputFormFields = ({ form, fieldName }: InputFormFieldsProps): React.JSX.Element => {
  return (
    <>
      <FormItem
        label="Width"
        name={[...fieldName, 'width']}
        rules={[{ required: false }]}
      >
        <InputNumber min={0} placeholder="Width in pixels" />
      </FormItem>
      
      <FormItem
        label="Default Value"
        name={[...fieldName, 'defaultValue']}
      >
        <Input placeholder="Default text value" />
      </FormItem>
      
      <FormItem
        label="Required"
        name={[...fieldName, 'mandatory']}
        valuePropName="checked"
      >
        <Checkbox />
      </FormItem>
    </>
  )
}
```

### Field Configuration Best Practices

1. **Use consistent naming**: Match Pimcore backend field names
2. **Provide sensible defaults**: Use `initialValue` in FormItem
3. **Add validation**: Use `rules` prop for required fields
4. **Group related settings**: Use FormKit sections or fieldsets

## Real-World Example: Input Field Definition

```typescript
// File: types/input/dynamic-type-field-definition-input.tsx
import React from 'react'
import { FormItem, Input, InputNumber, Checkbox } from '@pimcore/studio-ui-bundle/components'

export const InputFieldDefinitionForm = ({ fieldName }): React.JSX.Element => {
  return (
    <>
      {/* Basic Settings */}
      <FormItem label="Name" name={[...fieldName, 'name']} rules={[{ required: true }]}>
        <Input placeholder="Field name" />
      </FormItem>
      
      <FormItem label="Title" name={[...fieldName, 'title']}>
        <Input placeholder="Display title" />
      </FormItem>
      
      {/* Input-Specific Settings */}
      <FormItem label="Width" name={[...fieldName, 'width']}>
        <InputNumber min={0} max={1000} placeholder="Width" />
      </FormItem>
      
      <FormItem label="Default Value" name={[...fieldName, 'defaultValue']}>
        <Input placeholder="Default value" />
      </FormItem>
      
      <FormItem label="Column Length" name={[...fieldName, 'columnLength']}>
        <InputNumber min={1} max={190} placeholder="190" />
      </FormItem>
      
      <FormItem label="Regex" name={[...fieldName, 'regex']}>
        <Input placeholder="^[a-z0-9]+$" />
      </FormItem>
      
      {/* Flags */}
      <FormItem name={[...fieldName, 'mandatory']} valuePropName="checked">
        <Checkbox>Mandatory</Checkbox>
      </FormItem>
      
      <FormItem name={[...fieldName, 'unique']} valuePropName="checked">
        <Checkbox>Unique</Checkbox>
      </FormItem>
    </>
  )
}
```

## Real-World Example: Select Field Definition

```typescript
// File: types/select/select-form-fields.tsx
import React from 'react'
import { FormItem, Input, Select as AntSelect, Checkbox } from '@pimcore/studio-ui-bundle/components'

export const SelectFormFields = ({ fieldName }): React.JSX.Element => {
  return (
    <>
      <FormItem label="Options Source" name={[...fieldName, 'optionsProviderType']}>
        <AntSelect>
          <AntSelect.Option value="static">Static Options</AntSelect.Option>
          <AntSelect.Option value="class">Class Method</AntSelect.Option>
          <AntSelect.Option value="sql">SQL Query</AntSelect.Option>
        </AntSelect>
      </FormItem>
      
      <FormItem label="Options" name={[...fieldName, 'options']}>
        <Input.TextArea placeholder="Enter options (one per line)" rows={5} />
      </FormItem>
      
      <FormItem name={[...fieldName, 'mandatory']} valuePropName="checked">
        <Checkbox>Mandatory</Checkbox>
      </FormItem>
      
      <FormItem name={[...fieldName, 'noteditable']} valuePropName="checked">
        <Checkbox>Not Editable</Checkbox>
      </FormItem>
    </>
  )
}
```

## Complex Field Definition: ManyToOne Relation

```typescript
// File: types/manyToOne/many-to-one-form-fields.tsx
import React from 'react'
import { FormItem, Input, InputNumber, Checkbox, Select } from '@pimcore/studio-ui-bundle/components'

export const ManyToOneFormFields = ({ fieldName }): React.JSX.Element => {
  return (
    <>
      {/* Relation Configuration */}
      <FormItem label="Classes" name={[...fieldName, 'classes']}>
        <Select mode="multiple" placeholder="Select allowed classes">
          <Select.Option value="Product">Product</Select.Option>
          <Select.Option value="Category">Category</Select.Option>
        </Select>
      </FormItem>
      
      <FormItem label="Width" name={[...fieldName, 'width']}>
        <InputNumber min={0} placeholder="Width" />
      </FormItem>
      
      <FormItem label="Asset Upload Path" name={[...fieldName, 'assetUploadPath']}>
        <Input placeholder="/uploads" />
      </FormItem>
      
      {/* Object Resolver Configuration */}
      <FormItem label="Object Resolver Class" name={[...fieldName, 'objectsAllowed']}>
        <Checkbox.Group>
          <Checkbox value="object">Objects</Checkbox>
          <Checkbox value="asset">Assets</Checkbox>
          <Checkbox value="document">Documents</Checkbox>
        </Checkbox.Group>
      </FormItem>
      
      <FormItem name={[...fieldName, 'mandatory']} valuePropName="checked">
        <Checkbox>Mandatory</Checkbox>
      </FormItem>
    </>
  )
}
```

## Field Definition Checklist

When creating a new field definition:

- [ ] Create type registration entry
- [ ] Implement form fields component with all settings
- [ ] Add validation rules where needed
- [ ] Provide sensible defaults
- [ ] Implement grid cell renderer (see `grid-cells.md`)
- [ ] Test with save/load workflow
- [ ] Add translations for all labels
- [ ] Document any special configuration requirements

## Common Form Field Patterns

### Number Input with Min/Max

```typescript
<FormItem label="Min Value" name={[...fieldName, 'minValue']}>
  <InputNumber min={0} placeholder="Minimum" />
</FormItem>

<FormItem label="Max Value" name={[...fieldName, 'maxValue']}>
  <InputNumber min={0} placeholder="Maximum" />
</FormItem>
```

### Checkbox Flags

```typescript
<FormItem name={[...fieldName, 'mandatory']} valuePropName="checked">
  <Checkbox>Mandatory Field</Checkbox>
</FormItem>

<FormItem name={[...fieldName, 'invisible']} valuePropName="checked">
  <Checkbox>Invisible</Checkbox>
</FormItem>
```

### Text Area for Long Content

```typescript
<FormItem label="Description" name={[...fieldName, 'tooltip']}>
  <Input.TextArea rows={4} placeholder="Field description or help text" />
</FormItem>
```

## Next Steps

- **Grid Cells**: See `grid-cells.md` for rendering field values in grids
- **Data Types**: See `data-types.md` for data type registry (coming soon)
- **Main Skill**: See `SKILL.md` for complete dynamic types overview
