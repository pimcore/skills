# Grid Cell Dynamic Types

Specific implementation guide for grid cell dynamic types in Pimcore Studio.

## 🚨 CRITICAL: Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).** All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`).

---

## Overview

Grid cells are dynamic types used to render custom cell content in data grids (data object listings, asset grids, search results, etc.).

**Base Class**: `DynamicTypeGridCellAbstract`  
**Registry**: `DynamicTypeGridCellRegistry`  
**Service ID**: `serviceIds['DynamicTypes/GridCellRegistry']`

## Use Cases

- Custom data visualization (charts, badges, progress bars)
- Live-updating cells (polling, SSE, WebSocket)
- Interactive cells (switches, buttons, actions)
- Complex formatting (currency, dates, percentages)
- Conditional rendering based on row data
- Image/media previews

## Complete Example

### 1. Type Definition

```typescript
// dynamic-types/definitions/status-badge-cell.tsx
import { injectable } from '@pimcore/studio-ui-bundle/app'
import { 
  DynamicTypeGridCellAbstract,
  type AbstractGridCellDefinition 
} from '@pimcore/studio-ui-bundle/modules/element'
import React, { type ReactElement } from 'react'
import { StatusBadgeCellComponent } from '../components/status-badge-cell-component'

@injectable()
export class StatusBadgeCell extends DynamicTypeGridCellAbstract {
  id = 'status-badge'
  
  getGridCellComponent(props: AbstractGridCellDefinition): ReactElement {
    return <StatusBadgeCellComponent {...props} />
  }
}
```

### 2. Component Implementation

```typescript
// dynamic-types/components/status-badge-cell-component.tsx
import { type AbstractGridCellDefinition } from '@pimcore/studio-ui-bundle/modules/element'
import { Badge } from '@pimcore/studio-ui-bundle/components'
import React from 'react'

export const StatusBadgeCellComponent = ({ 
  getValue 
}: AbstractGridCellDefinition): React.JSX.Element => {
  const status = getValue() as string
  
  const colorMap: Record<string, string> = {
    active: 'green',
    inactive: 'red',
    pending: 'orange'
  }
  
  return (
    <div className="default-cell__content">
      <Badge color={colorMap[status] ?? 'gray'}>
        {status}
      </Badge>
    </div>
  )
}
```

### 3. Register in Module

```typescript
// modules/grid-cell-extension.tsx
import { type AbstractModule, container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import { type DynamicTypeGridCellRegistry } from '@pimcore/studio-ui-bundle/modules/element'

export const GridCellExtension: AbstractModule = {
  onInit: () => {
    const registry = container.get<DynamicTypeGridCellRegistry>(
      serviceIds['DynamicTypes/GridCellRegistry']
    )
    
    registry.registerDynamicType(
      container.get('DynamicTypes/GridCell/StatusBadgeCell')
    )
  }
}
```

## Available Props

`AbstractGridCellDefinition` provides:

```typescript
interface AbstractGridCellDefinition {
  getValue: () => any           // Get cell value
  row: Row<any>                 // Row data (row.original for full data)
  column: Column<any>           // Column configuration
  table: Table<any>             // Table instance
}
```

## Common Patterns

### Pattern: Using Row Context

Access other columns in the row:

```typescript
export const ContextAwareCellComponent = ({ 
  getValue,
  row 
}: AbstractGridCellDefinition): React.JSX.Element => {
  const value = getValue()
  const rowData = row.original
  
  // Access other fields
  const status = rowData.status
  const priority = rowData.priority
  
  return (
    <div className="default-cell__content">
      {status === 'urgent' && <Icon name="warning" />}
      {value} (Priority: {priority})
    </div>
  )
}
```

### Pattern: Interactive Cell

Cell with user interactions:

```typescript
import { useAssetUpdateMutation } from '@pimcore/studio-ui-bundle/api/asset'
import { Switch } from '@pimcore/studio-ui-bundle/components'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
import { isNil } from 'lodash'

export const ToggleCellComponent = ({ 
  getValue,
  row 
}: AbstractGridCellDefinition): React.JSX.Element => {
  const value = getValue() as boolean
  const [updateAsset, { error }] = useAssetUpdateMutation()
  
  useEffect(() => {
    if (!isNil(error)) {
      trackError(new ApiError(error))
    }
  }, [error])
  
  const handleToggle = (checked: boolean) => {
    updateAsset({
      id: row.original.id,
      body: { published: checked }
    })
  }
  
  return (
    <div className="default-cell__content">
      <Switch checked={value} onChange={handleToggle} />
    </div>
  )
}
```

### Pattern: Formatted Cell

Cell with formatting logic:

```typescript
import { useMemo } from 'react'

export const CurrencyCellComponent = ({ 
  getValue 
}: AbstractGridCellDefinition): React.JSX.Element => {
  const value = getValue() as number
  
  const formatted = useMemo(() => {
    return new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: 'USD'
    }).format(value)
  }, [value])
  
  return (
    <div className="default-cell__content">
      {formatted}
    </div>
  )
}
```

### Pattern: Image Preview Cell

```typescript
export const ImagePreviewCellComponent = ({ 
  getValue,
  row 
}: AbstractGridCellDefinition): React.JSX.Element => {
  const imageId = getValue() as number
  const imageUrl = `/admin/asset/get-image/${imageId}`
  
  return (
    <div className="default-cell__content">
      <img 
        src={imageUrl} 
        alt={row.original.filename} 
        style={{ height: 40, objectFit: 'cover' }} 
      />
    </div>
  )
}
```

### Pattern: Progress Bar Cell

```typescript
import { Progress } from '@pimcore/studio-ui-bundle/components'

export const ProgressCellComponent = ({ 
  getValue 
}: AbstractGridCellDefinition): React.JSX.Element => {
  const percentage = getValue() as number
  
  return (
    <div className="default-cell__content">
      <Progress 
        percent={percentage} 
        size="small" 
        status={percentage === 100 ? 'success' : 'active'}
      />
    </div>
  )
}
```

### Pattern: Live-Updating Cell

Cell that updates automatically:

```typescript
import { useState, useEffect } from 'react'
import { useAssetGetByIdQuery } from '@pimcore/studio-ui-bundle/api/asset'

export const LiveUpdatingCellComponent = ({ 
  row 
}: AbstractGridCellDefinition): React.JSX.Element => {
  const { data } = useAssetGetByIdQuery(
    { id: row.original.id },
    { pollingInterval: 5000 } // Refetch every 5 seconds
  )
  
  return (
    <div className="default-cell__content">
      {data?.status ?? 'Loading...'}
    </div>
  )
}
```

## Best Practices

1. **Always use `default-cell__content` class** for consistent spacing
2. **Handle null/undefined values** gracefully
3. **Use memoization** for expensive calculations
4. **Keep components lightweight** - grids render many cells
5. **Avoid heavy API calls** in every cell (use polling on parent instead)
6. **Test with different data types** - handle edge cases

## Performance Tips

```typescript
// ❌ BAD - Recalculates on every render
export const SlowCell = ({ getValue }) => {
  const value = getValue()
  const result = expensiveCalculation(value) // Runs every render!
  return <div>{result}</div>
}

// ✅ GOOD - Memoizes calculation
export const FastCell = ({ getValue }) => {
  const value = getValue()
  const result = useMemo(() => expensiveCalculation(value), [value])
  return <div>{result}</div>
}
```

## Styling

Use consistent cell styling:

```typescript
// Standard cell content wrapper
<div className="default-cell__content">
  {content}
</div>

// Center content
<div className="default-cell__content" style={{ textAlign: 'center' }}>
  {content}
</div>

// Right-align (e.g., numbers)
<div className="default-cell__content" style={{ textAlign: 'right' }}>
  {content}
</div>
```

## Testing

Test grid cells in isolation:

```typescript
import { render } from '@testing-library/react'
import { StatusBadgeCellComponent } from './status-badge-cell-component'

describe('StatusBadgeCellComponent', () => {
  it('renders active status correctly', () => {
    const props = {
      getValue: () => 'active',
      row: { original: { id: 1 } },
      column: {},
      table: {}
    }
    
    const { getByText } = render(<StatusBadgeCellComponent {...props} />)
    expect(getByText('active')).toBeInTheDocument()
  })
})
```

## Common Use Cases

### Use Case: Action Buttons Cell

```typescript
import { Button, Space } from '@pimcore/studio-ui-bundle/components'

export const ActionsCellComponent = ({ 
  row 
}: AbstractGridCellDefinition): React.JSX.Element => {
  const handleEdit = () => {
    // Open edit dialog
  }
  
  const handleDelete = () => {
    // Open delete confirmation
  }
  
  return (
    <div className="default-cell__content">
      <Space>
        <Button size="small" onClick={handleEdit}>Edit</Button>
        <Button size="small" danger onClick={handleDelete}>Delete</Button>
      </Space>
    </div>
  )
}
```

### Use Case: Conditional Icon Cell

```typescript
import { Icon } from '@pimcore/studio-ui-bundle/components'

export const IconCellComponent = ({ 
  getValue,
  row 
}: AbstractGridCellDefinition): React.JSX.Element => {
  const value = getValue()
  const status = row.original.status
  
  return (
    <div className="default-cell__content">
      {status === 'published' && <Icon name="check-circle" color="green" />}
      {status === 'draft' && <Icon name="edit" color="orange" />}
      {value}
    </div>
  )
}
```

## Reference

- **Base Class**: `DynamicTypeGridCellAbstract` from `@pimcore/studio-ui-bundle/modules/element`
- **Registry**: Access via `serviceIds['DynamicTypes/GridCellRegistry']`
- **Example**: studio-example-bundle `/examples/dynamic-types/grid-cells/`
