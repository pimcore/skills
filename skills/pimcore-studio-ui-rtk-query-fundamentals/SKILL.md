---
name: pimcore-studio-ui-rtk-query-fundamentals
description: RTK Query basics for data fetching in Pimcore Studio - queries, mutations, caching, error handling with trackError
metadata:
  audience: pimcore-developers
  focus: data-fetching
---

## What This Skill Covers

Fundamental patterns for using RTK Query (Redux Toolkit Query) in Pimcore Studio:
- Queries vs mutations
- Using auto-generated API hooks
- Proper error handling with `trackError`
- Loading and error states
- Cache management
- Common patterns

## When to Use This Skill

Use this when:
- Fetching data from the Pimcore backend
- Implementing create/update/delete operations
- Managing loading states
- Handling API errors properly
- Understanding caching behavior
- Need to refetch data

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## What is RTK Query?

**RTK Query** is Redux Toolkit's data fetching layer:
- Auto-generates React hooks from API endpoints
- Manages caching automatically
- Handles loading/error states
- Provides refetch capabilities
- Built-in request deduplication

**In Pimcore Studio**: API hooks are auto-generated from OpenAPI specifications, ensuring type safety and backend/frontend consistency.

## Queries vs Mutations

### Queries - Reading Data

**Use queries for**: GET requests, fetching data

```typescript
import { useAssetGetByIdQuery } from '@pimcore/studio-ui-bundle/api/asset'

const MyComponent = ({ assetId }: { assetId: number }) => {
  const { data, isLoading, error, refetch } = useAssetGetByIdQuery({
    id: assetId
  })
  
  // data: response from API
  // isLoading: true during initial fetch
  // error: contains error if request failed
  // refetch: function to manually refetch
}
```

### Mutations - Writing Data

**Use mutations for**: POST, PUT, PATCH, DELETE requests

```typescript
import { useAssetUpdateMutation } from '@pimcore/studio-ui-bundle/api/asset'

const MyComponent = () => {
  const [updateAsset, { isLoading, error }] = useAssetUpdateMutation()
  
  const handleSave = () => {
    updateAsset({
      id: 123,
      body: { filename: 'new-name.jpg' }
    })
    // Don't use .unwrap() - handle errors via error state
  }
}
```

**Key Difference**:
- **Queries** trigger automatically when component mounts
- **Mutations** only execute when you call them

## Error Handling - The Correct Way

### IMPORTANT: Use trackError, Not try/catch

**DON'T use `.unwrap()` and `try/catch` blocks!** Instead, handle errors via the `error` state with `trackError` in `useEffect`.

### Query Error Handling Pattern

```typescript
import { useAssetGetByIdQuery } from '@pimcore/studio-ui-bundle/api/asset'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
import { useEffect } from 'react'
import { isNil } from 'lodash'

const AssetDetail = ({ assetId }: { assetId: number }) => {
  const { data, isLoading, error } = useAssetGetByIdQuery({ id: assetId })
  
  // Track errors at component top level
  useEffect(() => {
    if (!isNil(error)) {
      trackError(new ApiError(error))
    }
  }, [error])
  
  if (isLoading) return <Skeleton />
  if (!data) return null
  
  return <div>{data.filename}</div>
}
```

### Mutation Error Handling Pattern

```typescript
import { useAssetUpdateMutation } from '@pimcore/studio-ui-bundle/api/asset'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
import { useEffect } from 'react'
import { isNil } from 'lodash'

const AssetEditor = ({ asset }: { asset: Asset }) => {
  const [updateAsset, { isLoading, error }] = useAssetUpdateMutation()
  
  // Track mutation errors at component top level
  useEffect(() => {
    if (!isNil(error)) {
      trackError(new ApiError(error))
    }
  }, [error])
  
  const handleSave = (values: FormValues) => {
    // No try/catch needed - errors tracked via useEffect
    updateAsset({
      id: asset.id,
      body: values
    })
  }
  
  return (
    <Form onFinish={handleSave}>
      {/* fields */}
      <Button htmlType="submit" loading={isLoading}>
        Save
      </Button>
    </Form>
  )
}
```

### Multiple Mutations - Track Each Error Separately

When you have multiple mutations, track each error separately:

```typescript
import { isNil } from 'lodash'

const MyComponent = () => {
  const [createMutation, { error: createError }] = useCreateMutation()
  const [updateMutation, { error: updateError }] = useUpdateMutation()
  const [deleteMutation, { error: deleteError }] = useDeleteMutation()
  
  // Track each error separately
  useEffect(() => {
    if (!isNil(createError)) {
      trackError(new ApiError(createError))
    }
  }, [createError])
  
  useEffect(() => {
    if (!isNil(updateError)) {
      trackError(new ApiError(updateError))
    }
  }, [updateError])
  
  useEffect(() => {
    if (!isNil(deleteError)) {
      trackError(new ApiError(deleteError))
    }
  }, [deleteError])
}
```

### Checking for Success vs Error

Instead of try/catch, check the error state:

```typescript
import { isNil } from 'lodash'

const MyForm = () => {
  const [createEntity, { data, error, isLoading }] = useCreateEntityMutation()
  
  useEffect(() => {
    if (!isNil(error)) {
      trackError(new ApiError(error))
    }
  }, [error])
  
  // Check for success
  useEffect(() => {
    if (!isNil(data)) {
      // Success! Navigate, close dialog, show message, etc.
      message.success('Created successfully')
    }
  }, [data])
  
  const handleSubmit = (values: FormValues) => {
    createEntity({ body: values })
  }
}
```

### How trackError Works

`trackError` automatically:
1. **Shows error modal** with formatted error message
2. **Prevents duplicates** - won't show same error multiple times
3. **Extracts error details** from API response (message, errorKey, etc.)
4. **Displays formatted UI** for API errors

You don't need to manually show error messages - `trackError` handles it!

## Basic Query Patterns

### Simple Data Fetch

```typescript
import { useDataObjectGetByIdQuery } from '@pimcore/studio-ui-bundle/api/data-object'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
import { isNil } from 'lodash'

const DataObjectDetail = ({ id }: { id: number }) => {
  const { data, isLoading, error } = useDataObjectGetByIdQuery({ id })
  
  useEffect(() => {
    if (!isNil(error)) {
      trackError(new ApiError(error))
    }
  }, [error])
  
  if (isLoading) return <Spin />
  if (!data) return null
  
  return <div>{data.key}</div>
}
```

### Conditional Query (Skip Pattern)

Don't fetch until condition is met:

```typescript
import { isNil } from 'lodash'

const DetailPanel = ({ selectedId }: { selectedId: number | null }) => {
  const { data, error } = useAssetGetByIdQuery(
    { id: selectedId! },
    { skip: selectedId === null } // Don't fetch if no selection
  )
  
  useEffect(() => {
    if (!isNil(error)) {
      trackError(new ApiError(error))
    }
  }, [error])
  
  if (!selectedId) return <div>Select an asset</div>
  if (!data) return <Spin />
  
  return <div>{data.filename}</div>
}
```

### Manual Refetch

```typescript
const AssetView = ({ id }: { id: number }) => {
  const { data, isFetching, refetch } = useAssetGetByIdQuery({ id })
  
  return (
    <div>
      <Button 
        onClick={() => void refetch()}
        loading={isFetching}
      >
        Refresh
      </Button>
      
      <div>{data?.filename}</div>
    </div>
  )
}
```

## Basic Mutation Patterns

### Create Operation

```typescript
import { useDataObjectCreateMutation } from '@pimcore/studio-ui-bundle/api/data-object'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
import { isNil } from 'lodash'

const CreateDialog = () => {
  const [create, { data, error, isLoading }] = useDataObjectCreateMutation()
  
  useEffect(() => {
    if (!isNil(error)) {
      trackError(new ApiError(error))
    }
  }, [error])
  
  useEffect(() => {
    if (!isNil(data)) {
      message.success('Created successfully')
      // Close dialog, navigate, etc.
    }
  }, [data])
  
  const handleCreate = (values: FormValues) => {
    create({
      body: {
        parentId: values.parentId,
        key: values.key,
        className: values.className
      }
    })
  }
  
  return (
    <FormKit type="form" onSubmit={handleCreate}>
      {/* form fields */}
      <FormKit type="submit" loading={isLoading}>
        Create
      </FormKit>
    </FormKit>
  )
}
```

### Update Operation

```typescript
import { useAssetUpdateMutation } from '@pimcore/studio-ui-bundle/api/asset'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
import { isNil } from 'lodash'

const AssetEditor = ({ asset }: { asset: Asset }) => {
  const [updateAsset, { data, error, isLoading }] = useAssetUpdateMutation()
  
  useEffect(() => {
    if (!isNil(error)) {
      trackError(new ApiError(error))
    }
  }, [error])
  
  useEffect(() => {
    if (!isNil(data)) {
      message.success('Saved successfully')
    }
  }, [data])
  
  const handleSave = (values: FormValues) => {
    updateAsset({
      id: asset.id,
      body: values
    })
  }
  
  return (
    <FormKit type="form" onSubmit={handleSave}>
      {/* fields */}
      <FormKit type="submit" loading={isLoading}>
        Save
      </FormKit>
    </FormKit>
  )
}
```

### Delete Operation

```typescript
import { useAssetDeleteMutation } from '@pimcore/studio-ui-bundle/api/asset'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
import { isNil } from 'lodash'

const DeleteButton = ({ assetId }: { assetId: number }) => {
  const [deleteAsset, { data, error, isLoading }] = useAssetDeleteMutation()
  
  useEffect(() => {
    if (!isNil(error)) {
      trackError(new ApiError(error))
    }
  }, [error])
  
  useEffect(() => {
    if (!isNil(data)) {
      message.success('Deleted successfully')
      // Navigate away, close dialog, etc.
    }
  }, [data])
  
  const handleDelete = () => {
    deleteAsset({ id: assetId })
  }
  
  return (
    <Button 
      danger 
      onClick={handleDelete}
      loading={isLoading}
    >
      Delete
    </Button>
  )
}
```

## Understanding Loading States

### isLoading vs isFetching

```typescript
const { data, isLoading, isFetching } = useAssetGetByIdQuery({ id })
```

- **`isLoading`**: `true` only on **first fetch** (no cached data exists)
- **`isFetching`**: `true` on **any fetch** (including refetch with cached data)

**Use cases:**
- **`isLoading`**: Show skeleton/spinner for initial load
- **`isFetching`**: Show refresh indicator when refetching

```typescript
// Initial load
if (isLoading) {
  return <Skeleton active />
}

// Refetch indicator
return (
  <div>
    {isFetching && <Spin />}
    <Button onClick={() => void refetch()}>
      Refresh
    </Button>
    <Content data={data} />
  </div>
)
```

### Mutation Loading State

```typescript
const [updateAsset, { isLoading: isSaving }] = useAssetUpdateMutation()

return (
  <FormKit type="form" onSubmit={handleSave}>
    {/* form fields */}
    <FormKit 
      type="submit"
      loading={isSaving}
      disabled={isSaving}
    >
      {isSaving ? 'Saving...' : 'Save'}
    </FormKit>
  </FormKit>
)
```

## Caching Behavior

### Automatic Caching

RTK Query caches responses automatically:

```typescript
// First component
const ComponentA = () => {
  const { data } = useAssetGetByIdQuery({ id: 123 })
  // Fetches from server
}

// Second component (same query)
const ComponentB = () => {
  const { data } = useAssetGetByIdQuery({ id: 123 })
  // Uses cached data, no network request!
}
```

### Cache Invalidation

Cache is automatically invalidated when:
- Mutation that affects the data is executed
- Cache timeout expires
- Manual invalidation is triggered

### Manual Refetch

Force refetch even with cached data:

```typescript
const { refetch } = useAssetGetByIdQuery({ id: 123 })

// Refetch from server
refetch()
```

## Common Patterns

### List + Detail Pattern

```typescript
import { isNil } from 'lodash'

const AssetBrowser = () => {
  const [selectedId, setSelectedId] = useState<number | null>(null)
  
  // List query (always active)
  const { data: list, error: listError } = useAssetListQuery()
  
  // Detail query (only when selected)
  const { data: detail, error: detailError } = useAssetGetByIdQuery(
    { id: selectedId! },
    { skip: !selectedId }
  )
  
  // Track errors
  useEffect(() => {
    if (!isNil(listError)) {
      trackError(new ApiError(listError))
    }
  }, [listError])
  
  useEffect(() => {
    if (!isNil(detailError)) {
      trackError(new ApiError(detailError))
    }
  }, [detailError])
  
  return (
    <div>
      <AssetList 
        assets={list} 
        onSelect={setSelectedId}
      />
      {detail && <AssetDetail asset={detail} />}
    </div>
  )
}
```

### Dependent Queries

Load data based on previous query result:

```typescript
import { isNil } from 'lodash'

const AssetWithParent = ({ id }: { id: number }) => {
  // First query
  const { data: asset, error: assetError } = useAssetGetByIdQuery({ id })
  
  // Second query (depends on first)
  const { data: parent, error: parentError } = useAssetGetByIdQuery(
    { id: asset?.parentId! },
    { skip: !asset?.parentId } // Don't fetch until we have parentId
  )
  
  useEffect(() => {
    if (!isNil(assetError)) {
      trackError(new ApiError(assetError))
    }
  }, [assetError])
  
  useEffect(() => {
    if (!isNil(parentError)) {
      trackError(new ApiError(parentError))
    }
  }, [parentError])
  
  return (
    <div>
      <div>Asset: {asset?.filename}</div>
      {parent && <div>Parent: {parent.filename}</div>}
    </div>
  )
}
```

### Polling (Auto-refresh)

Automatically refetch at intervals:

```typescript
const { data } = useAssetGetByIdQuery(
  { id: 123 },
  {
    pollingInterval: 5000 // Refetch every 5 seconds
  }
)
```

### Optimistic Updates with Error Recovery

Show UI change immediately, revert if mutation fails:

```typescript
const ToggleButton = ({ asset }: { asset: Asset }) => {
  const [localState, setLocalState] = useState(asset.published)
  const [updateAsset, { error }] = useAssetUpdateMutation()
  
  useEffect(() => {
    if (error !== undefined) {
      // Revert on error
      setLocalState(asset.published)
      trackError(new ApiError(error))
    }
  }, [error, asset.published])
  
  const handleToggle = () => {
    const newState = !localState
    setLocalState(newState) // Optimistic update
    
    updateAsset({
      id: asset.id,
      body: { published: newState }
    })
  }
  
  return (
    <Switch checked={localState} onChange={handleToggle} />
  )
}
```

## API Hook Naming Convention

Pimcore Studio follows a consistent naming pattern:

```
use{Entity}{Action}{Query|Mutation}
```

**Examples:**
- `useAssetGetByIdQuery` - Get single asset
- `useAssetListQuery` - Get asset list
- `useAssetCreateMutation` - Create asset
- `useAssetUpdateMutation` - Update asset
- `useAssetDeleteMutation` - Delete asset
- `useDataObjectGetByIdQuery` - Get data object
- `useDataObjectUpdateMutation` - Update data object

**Import pattern:**
```typescript
import { 
  useAssetGetByIdQuery,
  useAssetUpdateMutation 
} from '@pimcore/studio-ui-bundle/api/asset'
```

## TypeScript Types

Hooks are fully typed:

```typescript
// Query hook returns typed response
const { data } = useAssetGetByIdQuery({ id: 123 })
// data: Asset | undefined

// Mutation accepts typed parameters
const [update] = useAssetUpdateMutation()
update({
  id: number,
  body: AssetUpdateBody
})
```

Types are auto-generated from OpenAPI spec - always up to date with backend.

## Common Mistakes to Avoid

❌ **Don't use .unwrap() and try/catch**
```typescript
// BAD - Don't do this!
try {
  await updateAsset({ id, body }).unwrap()
  message.success('Saved')
} catch (error) {
  message.error('Failed')
}
```

✅ **Use error state with trackError**
```typescript
// GOOD
const [updateAsset, { data, error }] = useAssetUpdateMutation()

useEffect(() => {
  if (error !== undefined) {
    trackError(new ApiError(error))
  }
}, [error])

useEffect(() => {
  if (data !== undefined) {
    message.success('Saved')
  }
}, [data])

const handleSave = (values: FormValues) => {
  updateAsset({ id, body: values })
}

return (
  <FormKit type="form" onSubmit={handleSave}>
    {/* form fields */}
  </FormKit>
)
```

---

❌ **Don't forget to handle loading state**
```typescript
// BAD
const { data } = useAssetGetByIdQuery({ id })
return <div>{data.filename}</div> // Error if data undefined!
```

✅ **Always check data exists**
```typescript
// GOOD
const { data, isLoading } = useAssetGetByIdQuery({ id })
if (isLoading) return <Spin />
if (!data) return null
return <div>{data.filename}</div>
```

---

❌ **Don't call mutation in render**
```typescript
// BAD - causes infinite loop
const [update] = useAssetUpdateMutation()
update({ id, body }) // Called every render!
```

✅ **Call mutation in event handler**
```typescript
// GOOD
const [update] = useAssetUpdateMutation()
const handleSave = (values: FormValues) => update({ id, body: values })
return (
  <FormKit type="form" onSubmit={handleSave}>
    {/* form fields */}
    <FormKit type="submit">Save</FormKit>
  </FormKit>
)
```

---

❌ **Don't manually show error messages**
```typescript
// BAD - trackError already shows the error!
useEffect(() => {
  if (error !== undefined) {
    trackError(new ApiError(error))
    message.error('Failed') // Duplicate error display!
  }
}, [error])
```

✅ **Trust trackError to handle error display**
```typescript
// GOOD - trackError shows error modal automatically
useEffect(() => {
  if (error !== undefined) {
    trackError(new ApiError(error))
  }
}, [error])
```

## Performance Tips

1. **Use `skip` option** - Don't fetch until needed
2. **Leverage caching** - Same query parameters = cached response
3. **Conditional rendering** - Only mount components when data is ready
4. **Avoid unnecessary refetches** - Trust the cache unless data must be fresh
5. **Track errors at component top level** - One useEffect per error

## Common Mistakes

### ❌ Mistake 1: Using .unwrap() with try/catch
```typescript
// ❌ WRONG - Don't use .unwrap()
const [updateAsset] = useAssetUpdateMutation()
const handleSave = async () => {
  try {
    await updateAsset({ id: 123, body: data }).unwrap()
  } catch (error) {
    console.error(error)
  }
}

// ✅ CORRECT - Use error state with trackError
const [updateAsset, { error }] = useAssetUpdateMutation()
useEffect(() => {
  if (error !== undefined) {
    trackError(new ApiError(error))
  }
}, [error])
```

### ❌ Mistake 2: Not Checking isNil Before Using Data
```typescript
// ❌ WRONG - Falsy check
const { data } = useAssetGetByIdQuery({ id })
if (!data) return null

// ✅ CORRECT - Use isNil
import { isNil } from 'lodash'
const { data } = useAssetGetByIdQuery({ id })
if (isNil(data)) return null
```

### ❌ Mistake 3: Double Error Display
```typescript
// ❌ WRONG - trackError already shows error modal!
useEffect(() => {
  if (error !== undefined) {
    trackError(new ApiError(error))
    message.error('Failed to load') // Duplicate!
  }
}, [error])

// ✅ CORRECT - trackError handles display
useEffect(() => {
  if (error !== undefined) {
    trackError(new ApiError(error))
  }
}, [error])
```

### ❌ Mistake 4: Not Handling Loading States
```typescript
// ❌ WRONG - Rendering before data loaded
const { data } = useAssetGetByIdQuery({ id })
return <div>{data.name}</div> // Crashes if data is undefined!

// ✅ CORRECT - Check loading and nil
import { isNil } from 'lodash'
const { data, isLoading } = useAssetGetByIdQuery({ id })
if (isLoading) return <Skeleton />
if (isNil(data)) return null
return <div>{data.name}</div>
```

### ❌ Mistake 5: Wrong Import Paths
```typescript
// ❌ WRONG - In bundle, using @sdk
import { useAssetGetByIdQuery } from '@sdk/api/asset'

// ✅ CORRECT - Use bundle imports
import { useAssetGetByIdQuery } from '@pimcore/studio-ui-bundle/api/asset'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
```

## Next Steps

- [**pimcore-studio-ui-typescript-best-practices**](../pimcore-studio-ui-typescript-best-practices/SKILL.md) - Critical type safety patterns (lodash utilities, nullish coalescing)
- [**pimcore-studio-ui-forms-antd**](../pimcore-studio-ui-forms-antd/SKILL.md) - Integrating API data with FormKit
- [**pimcore-studio-ui-notifications-toasts**](../pimcore-studio-ui-notifications-toasts/SKILL.md) - Success messages and toasts
- [**pimcore-studio-ui-permissions**](../pimcore-studio-ui-permissions/SKILL.md) - Permission checks with API data
- [**pimcore-studio-ui-dynamic-types**](../pimcore-studio-ui-dynamic-types/SKILL.md) - Dynamic type system with RTK Query
