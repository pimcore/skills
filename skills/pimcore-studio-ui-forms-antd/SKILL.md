---
name: pimcore-studio-ui-forms-antd
description: Building forms with FormKit and Ant Design in Pimcore Studio - FormKit usage, validation, panels, field types
metadata:
  audience: pimcore-developers
  focus: ui-forms
---

## What This Skill Covers

Building forms in Pimcore Studio using FormKit (the standard form wrapper):
- Why and when to use FormKit (always!)
- FormKit vs plain Ant Design forms
- FormKit.Panel for organized form sections
- Form.Item for fields and validation
- Creating custom form components (controlled mode with value/onChange)
- Common field types and validation patterns
- Form state management basics
- Integration with RTK Query mutations

## When to Use This Skill

Use this when:
- Creating any form in Pimcore Studio
- Building edit/create dialogs
- Implementing form validation
- Organizing complex forms with panels
- Integrating forms with RTK Query

**Note**: This skill focuses on forms themselves. For user feedback (success/error messages), see the [`pimcore-studio-ui-notifications-toasts`](../pimcore-studio-ui-notifications-toasts/SKILL.md) skill.

## FormKit - The Standard Form Wrapper

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

**Never import from Ant Design directly!** Always import form components from `@pimcore/studio-ui-bundle/components`.

### Why FormKit?

**ALWAYS use FormKit for Pimcore Studio forms.** Do not use plain Ant Design `<Form>` directly.

FormKit provides:
1. **Consistent styling** - Automatic vertical layout, proper spacing, unified theming
2. **Built-in panels** - Easy form organization with `FormKit.Panel`
3. **Theme integration** - Wraps forms with ConfigProvider for consistent design
4. **Field width management** - Automatic consistent field widths (small: 200px, medium: 300px, large: 900px)
5. **Required for dynamic types** - Form definitions and dynamic type forms must use FormKit

## Basic FormKit Usage

### Simple Form

```typescript
import { FormKit, Form, Input, Button } from '@pimcore/studio-ui-bundle/components'

const MyForm = () => {
  const [form] = Form.useForm()
  
  const handleSubmit = (values) => {
    console.log('Form values:', values)
  }
  
  return (
    <FormKit
      formProps={{
        form,
        onFinish: handleSubmit
      }}
    >
      <Form.Item name="username" label="Username">
        <Input />
      </Form.Item>
      
      <Form.Item name="email" label="Email">
        <Input />
      </Form.Item>
      
      <Form.Item>
        <Button type="primary" htmlType="submit">
          Submit
        </Button>
      </Form.Item>
    </FormKit>
  )
}
```

**Key differences from plain Form:**
- Wrap everything with `<FormKit>`
- Pass Ant Design Form props via `formProps` prop
- FormKit automatically applies vertical layout
- No need to configure theme/styling manually

### FormKit Props

```typescript
interface FormKitProps {
  formProps?: Omit<FormProps, 'children'>  // Ant Design Form props
  children?: React.ReactNode
}
```

**Common formProps:**
- `form` - Form instance from `Form.useForm()`
- `initialValues` - Initial form values
- `onFinish` - Submit handler (called when validation passes)
- `onValuesChange` - Callback when any value changes
- `disabled` - Disable all form fields
- `component` - Set to `false` to remove form wrapper element

## FormKit.Panel - Organizing Forms

Use `FormKit.Panel` to group related fields into sections with proper theming and styling.

### Basic Panel

```typescript
import { FormKit, Form, Input, Select } from '@pimcore/studio-ui-bundle/components'

<FormKit formProps={{ form }}>
  <FormKit.Panel title="Basic Information">
    <Form.Item name="name" label="Name">
      <Input />
    </Form.Item>
    
    <Form.Item name="description" label="Description">
      <Input.TextArea rows={3} />
    </Form.Item>
  </FormKit.Panel>
  
  <FormKit.Panel title="Advanced Settings">
    <Form.Item name="status" label="Status">
      <Select>
        <Select.Option value="active">Active</Select.Option>
        <Select.Option value="inactive">Inactive</Select.Option>
      </Select>
    </Form.Item>
  </FormKit.Panel>
</FormKit>
```

### Panel Themes

FormKit.Panel supports multiple themes:

```typescript
// Default theme (card with highlight)
<FormKit.Panel title="Settings">

// Fieldset theme (border with inset title)
<FormKit.Panel title="Options" theme="fieldset">

// Border highlight theme
<FormKit.Panel title="Info" theme="border-highlight">
```

**Common pattern**: Use `theme="fieldset"` for nested panels inside a main panel.

### Panel Props

```typescript
interface FormKitPanelProps {
  title?: string              // Panel title
  tooltip?: ReactNode         // Tooltip for the title
  border?: boolean            // Show border
  collapsible?: boolean       // Make panel collapsible
  collapsed?: boolean         // Initial collapsed state
  theme?: 'default' | 'fieldset' | 'card-with-highlight' | 'border-highlight'
  extra?: ReactNode           // Extra content in header
  extraPosition?: 'start' | 'end'  // Position of extra content
  contentPadding?: BoxProps['padding']  // Custom padding
}
```

### Nested Panels Example

```typescript
import { FormKit, Form, Input, InputNumber, Switch } from '@pimcore/studio-ui-bundle/components'

<FormKit formProps={{ form }}>
  <FormKit.Panel title="Specific Settings">
    <Form.Item name="width" label="Width">
      <Input />
    </Form.Item>
    
    {/* Nested panel with fieldset theme */}
    <FormKit.Panel
      border
      theme="fieldset"
      title="Default Values"
    >
      <Form.Item name="defaultValue" label="Default Value">
        <InputNumber />
      </Form.Item>
      
      <Form.Item name="defaultValueGenerator" label="Generator">
        <Input />
      </Form.Item>
    </FormKit.Panel>
    
    <Form.Item name="enforceValidation">
      <Switch labelRight="Enforce Validation" />
    </Form.Item>
  </FormKit.Panel>
</FormKit>
```

## Form.Item - Fields and Validation

All form fields must be wrapped in `Form.Item` which handles:
- Field value binding (via `name` prop)
- Label display
- Validation
- Error messages
- Layout

### Basic Form.Item

```typescript
<Form.Item
  name="email"           // Field name in form values
  label="Email"          // Label text
  required               // Shows required indicator
  tooltip="Help text"    // Help tooltip
  rules={[/* validation */]}
>
  <Input />
</Form.Item>
```

### Nested Field Names

```typescript
// Nested object: { user: { email: "..." } }
<Form.Item name={['user', 'email']} label="Email">
  <Input />
</Form.Item>

// Array item: { items: [{ name: "..." }] }
<Form.Item name={['items', 0, 'name']} label="Item Name">
  <Input />
</Form.Item>
```

## Validation Rules

### Built-in Rules

```typescript
<Form.Item
  name="email"
  label="Email"
  rules={[
    { required: true, message: 'Please enter email' },
    { type: 'email', message: 'Please enter valid email' },
    { min: 5, message: 'Minimum 5 characters' },
    { max: 50, message: 'Maximum 50 characters' },
    { pattern: /^[a-z]+$/, message: 'Lowercase letters only' },
  ]}
>
  <Input />
</Form.Item>
```

**Available rule types:**
- `required` - Field must have value
- `type` - Validates type ('email', 'url', 'number', etc.)
- `min`/`max` - Length or number range
- `pattern` - Regex pattern
- `whitespace` - Disallow only whitespace

### Custom Validation

```typescript
<Form.Item
  name="password"
  rules={[
    { required: true },
    {
      validator: (_, value) => {
        if (!value || value.length >= 8) {
          return Promise.resolve()
        }
        return Promise.reject('Password must be at least 8 characters')
      }
    }
  ]}
>
  <Input.Password />
</Form.Item>
```

### Dependent Field Validation

```typescript
<Form.Item
  name="confirmPassword"
  label="Confirm Password"
  dependencies={['password']}  // Re-validate when password changes
  rules={[
    { required: true },
    ({ getFieldValue }) => ({
      validator(_, value) {
        if (!value || getFieldValue('password') === value) {
          return Promise.resolve()
        }
        return Promise.reject('Passwords do not match')
      },
    }),
  ]}
>
  <Input.Password />
</Form.Item>
```

## Common Field Types

### Text Input

```typescript
<Form.Item name="username" label="Username">
  <Input placeholder="Enter username" />
</Form.Item>
```

### Text Area

```typescript
<Form.Item name="description" label="Description">
  <Input.TextArea rows={4} />
</Form.Item>
```

### Number Input

```typescript
<Form.Item name="age" label="Age">
  <InputNumber min={0} max={120} />
</Form.Item>
```

### Select

```typescript
<Form.Item name="country" label="Country">
  <Select
    placeholder="Select country"
    options={[
      { label: 'United States', value: 'us' },
      { label: 'United Kingdom', value: 'uk' },
      { label: 'Germany', value: 'de' }
    ]}
  />
</Form.Item>
```

### Switch (Boolean)

```typescript
<Form.Item name="active" valuePropName="checked">
  <Switch labelRight="Active" />
</Form.Item>
```

**IMPORTANT**: Always use `valuePropName="checked"` for boolean fields (Switch, Checkbox)!

### Checkbox

```typescript
<Form.Item name="agree" valuePropName="checked">
  <Checkbox>I agree to the terms</Checkbox>
</Form.Item>
```

### Radio Group

```typescript
<Form.Item name="gender" label="Gender">
  <Radio.Group>
    <Radio value="male">Male</Radio>
    <Radio value="female">Female</Radio>
    <Radio value="other">Other</Radio>
  </Radio.Group>
</Form.Item>
```

### Date Picker

```typescript
<Form.Item name="birthdate" label="Birth Date">
  <DatePicker />
</Form.Item>
```

## Creating Custom Form Components

Form components in Ant Design/FormKit work in **controlled mode** using `value` and `onChange` props.

### How Form.Item Works

When you wrap a component in `Form.Item`, the form automatically:
1. Passes the current field `value` as a prop
2. Passes an `onChange` handler to update the value
3. Handles validation and error display

```typescript
// Form.Item automatically manages:
<Form.Item name="myField">
  <MyComponent 
    value={currentValue}        // Injected by Form.Item
    onChange={handleChange}     // Injected by Form.Item
  />
</Form.Item>
```

### Basic Custom Component Pattern

```typescript
interface MyCustomInputProps {
  value?: string           // Provided by Form.Item
  onChange?: (value: string) => void  // Provided by Form.Item
  disabled?: boolean       // Your custom props
  placeholder?: string
}

export const MyCustomInput = ({ 
  value = '', 
  onChange, 
  disabled,
  placeholder 
}: MyCustomInputProps) => {
  const handleChange = (newValue: string) => {
    // Call onChange with the new value
    onChange?.(newValue)
  }
  
  return (
    <div>
      <input
        value={value}
        onChange={(e) => handleChange(e.target.value)}
        disabled={disabled}
        placeholder={placeholder}
      />
    </div>
  )
}

// Usage in form
<Form.Item name="customField" label="Custom Field">
  <MyCustomInput placeholder="Enter value" />
</Form.Item>
```

### Complex Value Types

For non-string values (objects, arrays, numbers), just use the appropriate type:

```typescript
interface ColorPickerProps {
  value?: { r: number; g: number; b: number }
  onChange?: (value: { r: number; g: number; b: number }) => void
}

export const ColorPicker = ({ value, onChange }: ColorPickerProps) => {
  const handleColorChange = (channel: 'r' | 'g' | 'b', newValue: number) => {
    onChange?.({
      ...value,
      [channel]: newValue
    })
  }
  
  return (
    <div>
      <input 
        type="range" 
        value={value?.r ?? 0} 
        onChange={(e) => handleColorChange('r', Number(e.target.value))} 
      />
      {/* ... more sliders */}
    </div>
  )
}

// Usage
<Form.Item name="brandColor" label="Brand Color">
  <ColorPicker />
</Form.Item>
```

### Using valuePropName for Non-Standard Props

Some components use a different prop name instead of `value` (e.g., `checked` for checkboxes):

```typescript
// For components that use 'checked' instead of 'value'
<Form.Item name="enabled" valuePropName="checked">
  <Switch />
</Form.Item>

// Custom component with 'checked'
interface ToggleProps {
  checked?: boolean
  onChange?: (checked: boolean) => void
}

export const Toggle = ({ checked, onChange }: ToggleProps) => {
  return (
    <button onClick={() => onChange?.(!checked)}>
      {checked ? 'ON' : 'OFF'}
    </button>
  )
}

// Usage
<Form.Item name="isActive" valuePropName="checked">
  <Toggle />
</Form.Item>
```

### Real-World Example: Element Selector

```typescript
// Custom element selector component
interface ElementSelectorProps {
  value?: { id: number; type: string; path: string }
  onChange?: (value: { id: number; type: string; path: string }) => void
  allowedTypes?: string[]
  disabled?: boolean
}

export const ElementSelector = ({ 
  value, 
  onChange, 
  allowedTypes = ['document', 'asset'],
  disabled 
}: ElementSelectorProps) => {
  const [modalOpen, setModalOpen] = useState(false)
  
  const handleSelect = (element: any) => {
    onChange?.({
      id: element.id,
      type: element.type,
      path: element.path
    })
    setModalOpen(false)
  }
  
  const handleClear = () => {
    onChange?.(undefined)
  }
  
  return (
    <div>
      {value ? (
        <div>
          <span>{value.path}</span>
          <Button onClick={handleClear} disabled={disabled}>
            Clear
          </Button>
        </div>
      ) : (
        <Button onClick={() => setModalOpen(true)} disabled={disabled}>
          Select Element
        </Button>
      )}
      
      <ElementPickerModal
        open={modalOpen}
        onClose={() => setModalOpen(false)}
        onSelect={handleSelect}
        allowedTypes={allowedTypes}
      />
    </div>
  )
}

// Usage in form
<Form.Item 
  name="linkedDocument" 
  label="Linked Document"
  rules={[{ required: true, message: 'Please select a document' }]}
>
  <ElementSelector allowedTypes={['document']} />
</Form.Item>
```

### Wrapping Existing Components

You can create controlled wrappers around any component:

```typescript
// Wrap a third-party library component
import SomeLibraryComponent from 'some-library'

interface WrappedComponentProps {
  value?: any
  onChange?: (value: any) => void
}

export const WrappedComponent = ({ value, onChange }: WrappedComponentProps) => {
  return (
    <SomeLibraryComponent
      currentValue={value}  // Library uses different prop name
      onValueChange={onChange}  // Library uses different callback name
    />
  )
}

// Usage
<Form.Item name="wrappedField">
  <WrappedComponent />
</Form.Item>
```

### Key Points

1. **Always accept `value` and `onChange` props** - Form.Item will inject these
2. **Make them optional with defaults** - Component should work standalone too
3. **Call `onChange` with the new value** - Not an event object, just the value
4. **Use proper TypeScript types** - Define interfaces for your props
5. **Handle undefined/null values** - Forms may initialize with no value
6. **Use `valuePropName`** - When your component uses a different prop (e.g., `checked`)

## Form State Management

### Setting Initial Values

**Method 1: initialValues in formProps**
```typescript
<FormKit
  formProps={{
    form,
    initialValues: {
      username: 'john',
      active: true
    }
  }}
>
```

**Method 2: form.setFieldsValue() in useEffect**
```typescript
const [form] = Form.useForm()
const { data } = useGetEntityQuery({ id })

useEffect(() => {
  if (data) {
    form.setFieldsValue({
      username: data.username,
      email: data.email
    })
  }
}, [data, form])
```

**Use Method 2 when loading async data!**

### Getting Form Values

```typescript
const [form] = Form.useForm()

// Get all values
const values = form.getFieldsValue()

// Get specific field
const username = form.getFieldValue('username')
```

### Resetting Form

```typescript
// Reset to initial values
form.resetFields()

// Reset specific fields
form.resetFields(['username', 'email'])
```

## Form Submission with RTK Query

### Edit Form Pattern

```typescript
import { useAssetUpdateMutation, useAssetGetByIdQuery } from '@pimcore/studio-ui-bundle/api/asset'
import { FormKit, Form, Input, Button, Skeleton, message } from '@pimcore/studio-ui-bundle/components'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
import { useEffect } from 'react'
import { isNil } from 'lodash'

const AssetEditForm = ({ assetId }: { assetId: number }) => {
  const [form] = Form.useForm()
  const { data, isLoading, error: queryError } = useAssetGetByIdQuery({ id: assetId })
  const [updateAsset, { data: updateData, error: updateError, isLoading: isSaving }] = useAssetUpdateMutation()
  
  // Track query error
  useEffect(() => {
    if (!isNil(queryError)) {
      trackError(new ApiError(queryError))
    }
  }, [queryError])
  
  // Track mutation error
  useEffect(() => {
    if (!isNil(updateError)) {
      trackError(new ApiError(updateError))
    }
  }, [updateError])
  
  // Handle success
  useEffect(() => {
    if (!isNil(updateData)) {
      message.success('Saved successfully')
    }
  }, [updateData])
  
  // Load data into form
  useEffect(() => {
    if (data) {
      form.setFieldsValue({
        filename: data.filename,
        description: data.description
      })
    }
  }, [data, form])
  
  const handleSubmit = (values) => {
    // No try/catch needed - errors tracked via useEffect
    updateAsset({
      id: assetId,
      body: values
    })
  }
  
  if (isLoading) return <Skeleton />
  
  return (
    <FormKit formProps={{ form, onFinish: handleSubmit }}>
      <FormKit.Panel title="Basic Information">
        <Form.Item name="filename" label="Filename">
          <Input />
        </Form.Item>
        
        <Form.Item name="description" label="Description">
          <Input.TextArea rows={3} />
        </Form.Item>
      </FormKit.Panel>
      
      <Form.Item>
        <Button 
          type="primary" 
          htmlType="submit"
          loading={isSaving}
        >
          Save
        </Button>
      </Form.Item>
    </FormKit>
  )
}
```

### Create Form Pattern

```typescript
import { useCreateEntityMutation } from '@pimcore/studio-ui-bundle/api/entity'
import { FormKit, Form, Button, message } from '@pimcore/studio-ui-bundle/components'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
import { useEffect } from 'react'
import { isNil } from 'lodash'

const CreateForm = () => {
  const [createEntity, { data, error, isLoading }] = useCreateEntityMutation()
  
  // Track errors
  useEffect(() => {
    if (!isNil(error)) {
      trackError(new ApiError(error))
    }
  }, [error])
  
  // Handle success
  useEffect(() => {
    if (!isNil(data)) {
      message.success('Created successfully')
      // Navigate, close dialog, etc.
    }
  }, [data])
  
  const handleSubmit = (values) => {
    createEntity({ body: values })
  }
  
  return (
    <FormKit formProps={{ onFinish: handleSubmit }}>
      <FormKit.Panel title="Create New Entity">
        {/* fields */}
      </FormKit.Panel>
      
      <Button type="primary" htmlType="submit" loading={isLoading}>
        Create
      </Button>
    </FormKit>
  )
}
```

## Advanced Patterns

### Dynamic Fields (Form.List)

Add/remove fields dynamically:

```typescript
<FormKit formProps={{ form }}>
  <Form.List name="emails">
    {(fields, { add, remove }) => (
      <>
        {fields.map((field) => (
          <Space key={field.key}>
            <Form.Item
              {...field}
              name={[field.name, 'email']}
              rules={[{ required: true, type: 'email' }]}
            >
              <Input placeholder="Email address" />
            </Form.Item>
            
            <Button onClick={() => remove(field.name)}>
              Remove
            </Button>
          </Space>
        ))}
        
        <Button onClick={() => add()}>
          Add Email
        </Button>
      </>
    )}
  </Form.List>
</FormKit>
```

### Conditional Fields

Show fields based on other values:

```typescript
const MyForm = () => {
  const [form] = Form.useForm()
  const type = Form.useWatch('type', form)
  
  return (
    <FormKit formProps={{ form }}>
      <Form.Item name="type" label="Type">
        <Select>
          <Select.Option value="basic">Basic</Select.Option>
          <Select.Option value="advanced">Advanced</Select.Option>
        </Select>
      </Form.Item>
      
      {type === 'advanced' && (
        <Form.Item name="advancedSettings" label="Advanced Settings">
          <Input />
        </Form.Item>
      )}
    </FormKit>
  )
}
```

### Watching Form Values

Monitor field changes:

```typescript
const [form] = Form.useForm()

// Watch single field
const username = Form.useWatch('username', form)

// Watch nested field
const email = Form.useWatch(['user', 'email'], form)

// Watch all values
const values = Form.useWatch([], form)
```

### Handling Value Changes

```typescript
<FormKit
  formProps={{
    form,
    onValuesChange: (changedValues, allValues) => {
      console.log('Changed:', changedValues)
      console.log('All:', allValues)
    }
  }}
>
```

## Common Mistakes to Avoid

❌ **Using plain Form instead of FormKit**
```typescript
// BAD - don't use Ant Design Form directly
<Form form={form}>
```

✅ **Always use FormKit**
```typescript
// GOOD
<FormKit formProps={{ form }}>
```

---

❌ **Forgetting valuePropName="checked" for boolean fields**
```typescript
// BAD - won't work correctly
<Form.Item name="active">
  <Switch />
</Form.Item>
```

✅ **Use valuePropName for non-value props**
```typescript
// GOOD
<Form.Item name="active" valuePropName="checked">
  <Switch />
</Form.Item>
```

---

❌ **Not handling undefined data**
```typescript
// BAD - crashes if data is undefined
form.setFieldsValue({
  name: data.name // Error if data is undefined!
})
```

✅ **Check data exists first**
```typescript
// GOOD
if (data) {
  form.setFieldsValue({
    name: data.name
  })
}
```

---

❌ **Not disabling submit during save**
```typescript
// BAD - can submit multiple times
<Button htmlType="submit">Save</Button>
```

✅ **Disable while saving**
```typescript
// GOOD
<Button htmlType="submit" loading={isSaving} disabled={isSaving}>
  Save
</Button>
```

## TypeScript Tips

Type your form values:

```typescript
interface FormValues {
  username: string
  email: string
  active: boolean
}

const MyForm = () => {
  const [form] = Form.useForm<FormValues>()
  
  const handleSubmit = (values: FormValues) => {
    // values is typed!
  }
  
  return (
    <FormKit formProps={{ form, onFinish: handleSubmit }}>
      {/* fields */}
    </FormKit>
  )
}
```

## Common Mistakes

### ❌ Mistake 1: Using Plain Ant Design Form
```typescript
// ❌ WRONG - Don't use Ant Design Form directly
import { Form } from 'antd'
const [form] = Form.useForm()

// ✅ CORRECT - Always use FormKit
import { FormKit, Form } from '@pimcore/studio-ui-bundle/components'
const [form] = Form.useForm()
return <FormKit formProps={{ form }}>{/* fields */}</FormKit>
```

### ❌ Mistake 2: Wrong Import Paths
```typescript
// ❌ WRONG - Importing from antd
import { Form, Input } from 'antd'

// ✅ CORRECT - Import from SDK
import { FormKit, Form, Input } from '@pimcore/studio-ui-bundle/components'
```

### ❌ Mistake 3: Not Using FormKit.Panel
```typescript
// ❌ WRONG - Manual grouping
<div className="form-section">
  <h3>User Details</h3>
  <Form.Item name="name">...</Form.Item>
</div>

// ✅ CORRECT - Use FormKit.Panel
<FormKit.Panel title="User details">
  <Form.Item name="name">...</Form.Item>
</FormKit.Panel>
```

### ❌ Mistake 4: Missing value/onChange in Custom Components
```typescript
// ❌ WRONG - Missing controlled props
export const MyField = (props: Props): React.JSX.Element => {
  const [value, setValue] = useState(props.initialValue)
  return <input value={value} onChange={(e) => setValue(e.target.value)} />
}

// ✅ CORRECT - Controlled with value/onChange
interface Props {
  value?: string
  onChange?: (value: string) => void
}
export const MyField = ({ value, onChange }: Props): React.JSX.Element => {
  return <input value={value ?? ''} onChange={(e) => onChange?.(e.target.value)} />
}
```

### ❌ Mistake 5: Not Integrating with RTK Query Mutations
```typescript
// ❌ WRONG - Manual API calls
const handleSubmit = async (values) => {
  await fetch('/api/save', { body: JSON.stringify(values) })
}

// ✅ CORRECT - Use RTK Query mutation
const [updateUser] = useUserUpdateMutation()
const handleSubmit = (values: FormValues): void => {
  updateUser({ id: userId, body: values })
}
```

## Next Steps

- [**pimcore-studio-ui-typescript-best-practices**](../pimcore-studio-ui-typescript-best-practices/SKILL.md) - Critical type safety patterns (essential for form logic)
- [**pimcore-studio-ui-notifications-toasts**](../pimcore-studio-ui-notifications-toasts/SKILL.md) - Success/error messages and toasts
- [**pimcore-studio-ui-rtk-query-fundamentals**](../pimcore-studio-ui-rtk-query-fundamentals/SKILL.md) - API integration patterns
- [**pimcore-studio-ui-dynamic-types**](../pimcore-studio-ui-dynamic-types/SKILL.md) - Form definitions and dynamic type forms
- [**pimcore-studio-ui-i18n**](../pimcore-studio-ui-i18n/SKILL.md) - Translation keys for form labels and validation
