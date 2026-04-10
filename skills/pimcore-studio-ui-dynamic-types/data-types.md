# Data Types in Dynamic Types

Comprehensive guide to implementing data types in Pimcore Studio UI's dynamic type system.

## Overview

Data types define how field values are rendered and edited in data object detail views. Each data type provides:

1. **Edit Component** - Interactive editor for the field value
2. **Display Component** - Read-only view of the field value
3. **Value Validation** - Client-side validation rules
4. **Data Transformation** - Convert between UI and API formats

## Data Type Structure

```typescript
interface DynamicTypeDataType {
  type: string                          // Data type identifier (e.g., 'input', 'select')
  component: React.ComponentType        // Edit/display component
  readOnly?: boolean                    // If true, always render as read-only
  formatter?: (value: any) => any       // Transform value for display
  parser?: (value: any) => any          // Transform value for API
}
```

## Registering a Data Type

```typescript
// File: data-types/index.tsx
import { container } from '@pimcore/studio-ui-bundle/app'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import { type DynamicTypeDataTypeRegistry } from '@pimcore/studio-ui-bundle/modules/element'

// Get registry
const registry = container.get<DynamicTypeDataTypeRegistry>(
  serviceIds['DynamicTypes/DataTypeRegistry']
)

// Register data type
import { InputDataType } from './types/input/input-data-type'

registry.register({
  type: 'input',
  component: InputDataType
})
```

## Creating a Data Type Component

Data type components receive the field value and provide editing capabilities:

```typescript
// File: types/input/input-data-type.tsx
import React from 'react'
import { Input } from '@pimcore/studio-ui-bundle/components'
import { isNil } from 'lodash'

interface InputDataTypeProps {
  value?: string
  onChange?: (value: string) => void
  disabled?: boolean
  readOnly?: boolean
  fieldConfig: FieldDefinitionConfig  // Field definition settings
}

export const InputDataType = (props: InputDataTypeProps): React.JSX.Element => {
  const { value, onChange, disabled, readOnly, fieldConfig } = props
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>): void => {
    if (isNil(onChange)) return
    onChange(e.target.value)
  }
  
  // Read-only view
  if (readOnly) {
    return <span>{value ?? '-'}</span>
  }
  
  // Editable view
  return (
    <Input
      defaultValue={fieldConfig.defaultValue}
      disabled={disabled}
      maxLength={fieldConfig.columnLength}
      onChange={handleChange}
      placeholder={fieldConfig.title}
      value={value}
    />
  )
}
```

## Data Type Component Props

All data type components receive these standard props:

```typescript
interface DataTypeProps<T = any> {
  value?: T                           // Current field value
  onChange?: (value: T) => void       // Value change handler
  disabled?: boolean                  // Disable editing
  readOnly?: boolean                  // Read-only mode
  fieldConfig: FieldDefinitionConfig  // Field definition settings
  context?: DataObjectContext         // Parent data object context
}
```

## Real-World Example: Number Data Type

```typescript
// File: types/number/number-data-type.tsx
import React from 'react'
import { InputNumber } from '@pimcore/studio-ui-bundle/components'
import { isNil } from 'lodash'

interface NumberDataTypeProps {
  value?: number
  onChange?: (value: number | null) => void
  disabled?: boolean
  readOnly?: boolean
  fieldConfig: {
    minValue?: number
    maxValue?: number
    unsigned?: boolean
    decimalPrecision?: number
  }
}

export const NumberDataType = (props: NumberDataTypeProps): React.JSX.Element => {
  const { value, onChange, disabled, readOnly, fieldConfig } = props
  
  if (readOnly) {
    return <span>{value ?? '-'}</span>
  }
  
  return (
    <InputNumber
      disabled={disabled}
      max={fieldConfig.maxValue}
      min={fieldConfig.minValue ?? (fieldConfig.unsigned ? 0 : undefined)}
      onChange={onChange}
      precision={fieldConfig.decimalPrecision ?? 0}
      value={value}
    />
  )
}
```

## Real-World Example: Select Data Type

```typescript
// File: types/select/select-data-type.tsx
import React from 'react'
import { Select } from '@pimcore/studio-ui-bundle/components'
import { isNil } from 'lodash'

interface SelectDataTypeProps {
  value?: string
  onChange?: (value: string) => void
  disabled?: boolean
  readOnly?: boolean
  fieldConfig: {
    options: Array<{ key: string; value: string }>
  }
}

export const SelectDataType = (props: SelectDataTypeProps): React.JSX.Element => {
  const { value, onChange, disabled, readOnly, fieldConfig } = props
  
  if (readOnly) {
    const option = fieldConfig.options.find(opt => opt.key === value)
    return <span>{option?.value ?? value ?? '-'}</span>
  }
  
  return (
    <Select
      disabled={disabled}
      onChange={onChange}
      value={value}
    >
      {fieldConfig.options.map(option => (
        <Select.Option key={option.key} value={option.key}>
          {option.value}
        </Select.Option>
      ))}
    </Select>
  )
}
```

## Complex Data Type: ManyToOne Relation

```typescript
// File: types/manyToOne/many-to-one-data-type.tsx
import React, { useState } from 'react'
import { Input, Button, Modal } from '@pimcore/studio-ui-bundle/components'
import { isNil } from 'lodash'

interface RelationValue {
  id: number
  type: 'asset' | 'document' | 'object'
  path: string
}

interface ManyToOneDataTypeProps {
  value?: RelationValue
  onChange?: (value: RelationValue | null) => void
  disabled?: boolean
  readOnly?: boolean
  fieldConfig: {
    classes: string[]
    objectsAllowed: string[]
  }
}

export const ManyToOneDataType = (props: ManyToOneDataTypeProps): React.JSX.Element => {
  const { value, onChange, disabled, readOnly, fieldConfig } = props
  const [selectorOpen, setSelectorOpen] = useState(false)
  
  if (readOnly) {
    return <span>{value?.path ?? '-'}</span>
  }
  
  const handleSelect = (element: RelationValue): void => {
    if (isNil(onChange)) return
    onChange(element)
    setSelectorOpen(false)
  }
  
  const handleClear = (): void => {
    if (isNil(onChange)) return
    onChange(null)
  }
  
  return (
    <>
      <Input.Group compact>
        <Input
          disabled
          style={{ width: 'calc(100% - 100px)' }}
          value={value?.path}
        />
        <Button
          disabled={disabled}
          onClick={() => setSelectorOpen(true)}
        >
          Select
        </Button>
        <Button
          disabled={disabled || isNil(value)}
          onClick={handleClear}
        >
          Clear
        </Button>
      </Input.Group>
      
      {selectorOpen && (
        <ElementSelectorModal
          allowedTypes={fieldConfig.objectsAllowed}
          classes={fieldConfig.classes}
          onCancel={() => setSelectorOpen(false)}
          onSelect={handleSelect}
        />
      )}
    </>
  )
}
```

## Data Transformation

### Formatter (API → UI)

Transform data from API format to UI format:

```typescript
const dateFormatter = (value: string): Date | null => {
  return value ? new Date(value) : null
}

registry.register({
  type: 'date',
  component: DateDataType,
  formatter: dateFormatter
})
```

### Parser (UI → API)

Transform data from UI format to API format:

```typescript
const dateParser = (value: Date | null): string | null => {
  return value ? value.toISOString() : null
}

registry.register({
  type: 'date',
  component: DateDataType,
  parser: dateParser
})
```

## Validation in Data Types

Data type components can include validation:

```typescript
export const EmailDataType = ({ value, onChange, fieldConfig }): React.JSX.Element => {
  const [error, setError] = useState<string>()
  
  const handleChange = (newValue: string): void => {
    // Validate email format
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    if (newValue && !emailRegex.test(newValue)) {
      setError('Invalid email format')
    } else {
      setError(undefined)
    }
    
    onChange?.(newValue)
  }
  
  return (
    <>
      <Input
        onChange={(e) => handleChange(e.target.value)}
        status={error ? 'error' : undefined}
        value={value}
      />
      {error && <div className="error-message">{error}</div>}
    </>
  )
}
```

## Data Type Best Practices

1. **Always check for nil**: Use `isNil()` before calling `onChange`
2. **Provide read-only view**: Always handle `readOnly` prop
3. **Use fieldConfig**: Leverage field definition settings
4. **Handle disabled state**: Respect `disabled` prop
5. **Provide feedback**: Show validation errors clearly
6. **Use SDK components**: Never import from Ant Design directly
7. **Type safety**: Always type props interfaces properly

## Common Patterns

### Conditional Rendering

```typescript
if (readOnly) {
  return <span className="readonly-value">{formatValue(value)}</span>
}

if (disabled) {
  return <Input disabled value={value} />
}

return <Input onChange={handleChange} value={value} />
```

### Value Transformation

```typescript
const handleChange = (rawValue: any): void => {
  const transformedValue = transformValue(rawValue, fieldConfig)
  onChange?.(transformedValue)
}
```

### Using Field Config

```typescript
const placeholder = fieldConfig.title ?? fieldConfig.name
const maxLength = fieldConfig.columnLength ?? 255
const required = fieldConfig.mandatory ?? false
```

## Data Type Checklist

When creating a new data type:

- [ ] Create component with standard props
- [ ] Handle `readOnly` mode
- [ ] Handle `disabled` state
- [ ] Use field configuration settings
- [ ] Add value validation if needed
- [ ] Implement formatters/parsers if needed
- [ ] Test with save/load workflow
- [ ] Add proper TypeScript types
- [ ] Use lodash utilities for null checks

## Next Steps

- **Field Definitions**: See `field-definitions.md` for field configuration
- **Grid Cells**: See `grid-cells.md` for grid rendering
- **Main Skill**: See `SKILL.md` for complete dynamic types overview
