---
name: pimcore-studio-ui-tables-grids
description: Using Grid component and TanStack Table in Pimcore Studio UI for data tables and grids
metadata:
  audience: pimcore-developers
  focus: ui-components-tables
---

## What This Skill Covers

How to use tables and grids in Pimcore Studio UI:
- **Grid** component for data tables
- **TanStack Table** (React Table v8) for table functionality
- Column definitions with `createColumnHelper`
- Sorting, selection, and row actions
- Loading states and empty states
- Real-world patterns

## When to Use This Skill

Use this when:
- Displaying tabular data (lists, collections)
- Creating data grids with sorting/filtering
- Building list views with selectable rows
- Showing metadata, properties, or configurations in tables
- Creating editable data tables

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## Grid Component

The `Grid` component is Pimcore's wrapper around TanStack Table with Studio UI styling and features.

### Basic Grid Usage

```typescript
import { Grid } from '@pimcore/studio-ui-bundle/components'
import { createColumnHelper } from '@tanstack/react-table'
import { useTranslation } from 'react-i18next'

interface DataRow {
  id: number
  name: string
  type: string
  modified: string
}

export const MyDataGrid = () => {
  const { t } = useTranslation()
  const columnHelper = createColumnHelper<DataRow>()

  const columns = [
    columnHelper.accessor('name', {
      header: t('columns.name'),
      size: 200
    }),
    columnHelper.accessor('type', {
      header: t('columns.type'),
      size: 150
    }),
    columnHelper.accessor('modified', {
      header: t('columns.modified'),
      size: 180
    })
  ]

  const data: DataRow[] = [
    { id: 1, name: 'Item 1', type: 'Type A', modified: '2024-01-15' },
    { id: 2, name: 'Item 2', type: 'Type B', modified: '2024-01-16' }
  ]

  return (
    <Grid
      columns={ columns }
      data={ data }
    />
  )
}
```

### Grid Props

```typescript
interface GridProps {
  columns: ColumnDef<any>[]           // Column definitions (required)
  data: any[]                          // Row data (required)
  
  // Selection
  enableRowSelection?: boolean         // Enable single row selection
  enableMultipleRowSelection?: boolean // Enable multiple row selection
  selectedRows?: Record<string, boolean> // Controlled selection state
  onRowSelectionChange?: (selection: Record<string, boolean>) => void
  
  // Sorting
  enableSorting?: boolean              // Enable column sorting
  manualSorting?: boolean              // Control sorting externally
  sorting?: SortingState               // Controlled sorting state
  onSortingChange?: (sorting: SortingState) => void
  
  // Loading & Empty States
  isLoading?: boolean                  // Show loading skeleton
  
  // Appearance
  size?: 'small' | 'normal' | 'large' // Row size
  hideColumnHeaders?: boolean          // Hide header row
  autoWidth?: boolean                  // Auto-size table width
  
  // Advanced
  enableRowDrag?: boolean              // Enable row drag-and-drop
  enableRowVirtualizer?: boolean       // Virtual scrolling for rows
  enableColumnVirtualizer?: boolean    // Virtual scrolling for columns
}
```

## Defining Columns with createColumnHelper

Use `createColumnHelper` to define type-safe columns.

### Basic Column Definition

```typescript
import { createColumnHelper } from '@tanstack/react-table'

interface MyData {
  id: number
  name: string
  status: string
  count: number
}

const columnHelper = createColumnHelper<MyData>()

const columns = [
  // Accessor column - directly access property
  columnHelper.accessor('name', {
    header: t('columns.name'),
    size: 200
  }),
  
  // Accessor column with custom cell
  columnHelper.accessor('status', {
    header: t('columns.status'),
    size: 120,
    cell: (info) => {
      const status = info.getValue()
      return <Badge status={ status }>{status}</Badge>
    }
  }),
  
  // Display column - computed value
  columnHelper.display({
    id: 'actions',
    header: t('columns.actions'),
    size: 100,
    cell: (info) => (
      <IconButton
        icon={ { value: 'edit' } }
        onClick={ () => handleEdit(info.row.original.id) }
        tooltip={ { title: t('edit') } }
      />
    )
  })
]
```

### Column Properties

```typescript
{
  // Identification
  id?: string                          // Unique column ID
  header: string | (() => JSX.Element) // Header content
  
  // Sizing
  size?: number                        // Default width in pixels
  minSize?: number                     // Minimum width
  maxSize?: number                     // Maximum width
  
  // Cell Rendering
  cell?: (info: CellContext) => JSX.Element  // Custom cell renderer
  
  // Sorting
  enableSorting?: boolean              // Enable sorting for this column
  sortingFn?: string | SortingFn       // Custom sort function
  
  // Metadata
  meta?: ColumnMetaType                // Custom metadata (Pimcore-specific)
}
```

### Column Types

#### 1. Accessor Column (Direct Property Access)

```typescript
columnHelper.accessor('propertyName', {
  header: t('header'),
  size: 200
})
```

#### 2. Accessor Column with Transform

```typescript
columnHelper.accessor((row) => row.firstName + ' ' + row.lastName, {
  id: 'fullName',
  header: t('full-name'),
  size: 250
})
```

#### 3. Display Column (Custom Content)

```typescript
columnHelper.display({
  id: 'actions',
  header: t('actions'),
  cell: (info) => (
    <ButtonGroup items={ [
      <IconButton
        icon={ { value: 'edit' } }
        key="edit"
        onClick={ () => handleEdit(info.row.original.id) }
        tooltip={ { title: t('edit') } }
      />,
      <IconButton
        icon={ { value: 'delete' } }
        key="delete"
        onClick={ () => handleDelete(info.row.original.id) }
        tooltip={ { title: t('delete') } }
      />
    ] }/>
  )
})
```

## Row Selection

Enable row selection for interactive grids.

### Single Row Selection

```typescript
const [selectedRows, setSelectedRows] = useState<Record<string, boolean>>({})

return (
  <Grid
    columns={ columns }
    data={ data }
    enableRowSelection
    selectedRows={ selectedRows }
    onRowSelectionChange={ setSelectedRows }
  />
)
```

### Multiple Row Selection

```typescript
const [selectedRows, setSelectedRows] = useState<Record<string, boolean>>({})

return (
  <Grid
    columns={ columns }
    data={ data }
    enableMultipleRowSelection  // Adds checkboxes
    selectedRows={ selectedRows }
    onRowSelectionChange={ setSelectedRows }
  />
)
```

### Getting Selected Row Data

```typescript
const [selectedRows, setSelectedRows] = useState<Record<string, boolean>>({})

const getSelectedData = () => {
  return data.filter((row, index) => selectedRows[index])
}

const handleBulkAction = () => {
  const selected = getSelectedData()
  console.log('Selected items:', selected)
}
```

## Sorting

Enable column sorting functionality.

### Basic Sorting

```typescript
const [sorting, setSorting] = useState<SortingState>([])

return (
  <Grid
    columns={ columns }
    data={ data }
    enableSorting
    sorting={ sorting }
    onSortingChange={ setSorting }
  />
)
```

### Manual Sorting (Server-Side)

```typescript
const [sorting, setSorting] = useState<SortingState>([])

// Fetch data when sorting changes
useEffect(() => {
  if (sorting.length > 0) {
    const { id, desc } = sorting[0]
    fetchData({
      sortBy: id,
      sortOrder: desc ? 'DESC' : 'ASC'
    })
  }
}, [sorting])

return (
  <Grid
    columns={ columns }
    data={ data }
    enableSorting
    manualSorting  // Don't sort locally
    sorting={ sorting }
    onSortingChange={ setSorting }
  />
)
```

## Loading States

Show skeleton loading while data is fetching.

### Basic Loading State

```typescript
const { data, isLoading } = useQuery()

return (
  <Grid
    columns={ columns }
    data={ data ?? [] }
    isLoading={ isLoading }  // Shows skeleton rows
  />
)
```

## Real-World Pattern: Metadata Table

```typescript
import { Grid, IconButton } from '@pimcore/studio-ui-bundle/components'
import { createColumnHelper } from '@tanstack/react-table'
import { useTranslation } from 'react-i18next'

interface Metadata {
  id: number
  name: string
  type: string
  value: string
  language?: string
}

export const MetadataTable = ({ metadata }: { metadata: Metadata[] }) => {
  const { t } = useTranslation()
  const columnHelper = createColumnHelper<Metadata>()

  const columns = [
    columnHelper.accessor('name', {
      header: t('metadata.columns.name'),
      size: 200
    }),
    columnHelper.accessor('type', {
      header: t('metadata.columns.type'),
      size: 150
    }),
    columnHelper.accessor('language', {
      header: t('metadata.columns.language'),
      size: 100,
      cell: (info) => info.getValue() || '-'
    }),
    columnHelper.accessor('value', {
      header: t('metadata.columns.value'),
      size: 300
    }),
    columnHelper.display({
      id: 'actions',
      header: t('metadata.columns.actions'),
      size: 100,
      cell: (info) => (
        <IconButton
          icon={ { value: 'edit' } }
          onClick={ () => handleEdit(info.row.original.id) }
          tooltip={ { title: t('edit') } }
        />
      )
    })
  ]

  return (
    <Grid
      columns={ columns }
      data={ metadata }
    />
  )
}
```

## Real-World Pattern: Selectable List with Actions

```typescript
import { Grid, Button, IconButton } from '@pimcore/studio-ui-bundle/components'
import { createColumnHelper } from '@tanstack/react-table'
import { useTranslation } from 'react-i18next'

interface Item {
  id: number
  name: string
  status: 'active' | 'inactive'
  modified: string
}

export const SelectableItemList = () => {
  const { t } = useTranslation()
  const [selectedRows, setSelectedRows] = useState<Record<string, boolean>>({})
  const columnHelper = createColumnHelper<Item>()

  const { data, isLoading } = useGetItemsQuery()

  const columns = [
    columnHelper.accessor('name', {
      header: t('columns.name'),
      size: 250
    }),
    columnHelper.accessor('status', {
      header: t('columns.status'),
      size: 120,
      cell: (info) => {
        const status = info.getValue()
        return (
          <Badge color={ status === 'active' ? 'green' : 'gray' }>
            {t(`status.${status}`)}
          </Badge>
        )
      }
    }),
    columnHelper.accessor('modified', {
      header: t('columns.modified'),
      size: 180
    }),
    columnHelper.display({
      id: 'actions',
      header: t('columns.actions'),
      size: 100,
      cell: (info) => (
        <IconButton
          icon={ { value: 'edit' } }
          onClick={ () => handleEdit(info.row.original.id) }
          tooltip={ { title: t('edit') } }
        />
      )
    })
  ]

  const selectedCount = Object.values(selectedRows).filter(Boolean).length

  const handleBulkDelete = () => {
    const selectedItems = data?.filter((_, index) => selectedRows[index])
    // Handle bulk delete
  }

  return (
    <div>
      {selectedCount > 0 && (
        <div style={ { marginBottom: 16 } }>
          <span>{t('items-selected', { count: selectedCount })}</span>
          <Button
            color="danger"
            onClick={ handleBulkDelete }
            style={ { marginLeft: 16 } }
          >
            {t('delete-selected')}
          </Button>
        </div>
      )}
      
      <Grid
        columns={ columns }
        data={ data ?? [] }
        enableMultipleRowSelection
        isLoading={ isLoading }
        selectedRows={ selectedRows }
        onRowSelectionChange={ setSelectedRows }
      />
    </div>
  )
}
```

## Common Patterns

### Pattern 1: Simple Data Table

```typescript
const columnHelper = createColumnHelper<DataType>()

const columns = [
  columnHelper.accessor('field1', {
    header: t('field1'),
    size: 200
  }),
  columnHelper.accessor('field2', {
    header: t('field2'),
    size: 150
  })
]

return <Grid columns={ columns } data={ data } />
```

### Pattern 2: Table with Actions Column

```typescript
const columns = [
  // ... data columns
  columnHelper.display({
    id: 'actions',
    header: t('actions'),
    size: 80,
    cell: (info) => (
      <IconButton
        icon={ { value: 'edit' } }
        onClick={ () => handleEdit(info.row.original.id) }
        tooltip={ { title: t('edit') } }
      />
    )
  })
]
```

### Pattern 3: Custom Cell Rendering

```typescript
columnHelper.accessor('status', {
  header: t('status'),
  size: 120,
  cell: (info) => {
    const value = info.getValue()
    return <Badge status={ value }>{t(`status.${value}`)}</Badge>
  }
})
```

### Pattern 4: Sortable Table

```typescript
const [sorting, setSorting] = useState<SortingState>([])

return (
  <Grid
    columns={ columns }
    data={ data }
    enableSorting
    sorting={ sorting }
    onSortingChange={ setSorting }
  />
)
```

## Common Mistakes

### ❌ Not Using Translation Keys

```typescript
// DON'T - Hardcoded headers
columnHelper.accessor('name', {
  header: 'Name'
})

// DO - Always translate
columnHelper.accessor('name', {
  header: t('columns.name')
})
```

### ❌ Forgetting Column IDs for Display Columns

```typescript
// DON'T - No ID
columnHelper.display({
  cell: () => <button>Action</button>
})

// DO - Always provide ID
columnHelper.display({
  id: 'actions',
  cell: () => <button>Action</button>
})
```

### ❌ Not Handling Loading States

```typescript
// DON'T - No loading state
const { data } = useQuery()
return <Grid columns={ columns } data={ data } />

// DO - Handle loading
const { data, isLoading } = useQuery()
return <Grid columns={ columns } data={ data ?? [] } isLoading={ isLoading } />
```

### ❌ Missing Tooltips on Action Buttons

```typescript
// DON'T - No tooltip
<IconButton
  icon={ { value: 'edit' } }
  onClick={ handleEdit }
/>

// DO - Always include tooltip
<IconButton
  icon={ { value: 'edit' } }
  onClick={ handleEdit }
  tooltip={ { title: t('edit') } }
/>
```

## Quick Reference

### Basic Grid Setup
```typescript
import { Grid } from '@pimcore/studio-ui-bundle/components'
import { createColumnHelper } from '@tanstack/react-table'

const columnHelper = createColumnHelper<DataType>()
const columns = [
  columnHelper.accessor('field', {
    header: t('header'),
    size: 200
  })
]

<Grid columns={ columns } data={ data } />
```

### Common Column Patterns
```typescript
// Simple accessor
columnHelper.accessor('name', { header: t('name') })

// Custom cell
columnHelper.accessor('status', {
  header: t('status'),
  cell: (info) => <Badge>{info.getValue()}</Badge>
})

// Actions column
columnHelper.display({
  id: 'actions',
  cell: (info) => <IconButton icon={ { value: 'edit' } } />
})
```

## Next Steps

- [**pimcore-studio-ui-buttons**](../pimcore-studio-ui-buttons/SKILL.md) - Action buttons for table actions
- [**pimcore-studio-ui-i18n**](../pimcore-studio-ui-i18n/SKILL.md) - Translation keys for column headers
- [**pimcore-studio-ui-layout-components**](../pimcore-studio-ui-layout-components/SKILL.md) - Layout context for tables
- [**pimcore-studio-ui-react-components**](../pimcore-studio-ui-react-components/SKILL.md) - Component patterns for cells
