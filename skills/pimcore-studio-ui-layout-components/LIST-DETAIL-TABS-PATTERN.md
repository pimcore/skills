# ConfigLayout: List-Detail with Tabs Pattern

## Overview

This is a **standard UI pattern** for managing collections of entities in Pimcore Studio UI. It provides a complete interface for browsing, creating, editing, and deleting items.

**Core concept:** ConfigLayout with a searchable list on the left and tabbed detail views on the right.

## Visual Structure

```
┌─────────────────────────────────────────────────────────┐
│ ConfigLayout                                            │
├─────────────┬───────────────────────────────────────────┤
│ LEFT SIDE   │ RIGHT SIDE                                │
│             │                                           │
│ ┌─────────┐ │ ┌─────────────────────────────────────┐  │
│ │ Search  │ │ │ Tabs (one per open item)            │  │
│ └─────────┘ │ ├─────────────────────────────────────┤  │
│             │ │ ┌─────────────────────────────────┐ │  │
│ Item 1      │ │ │ Form fields                     │ │  │
│ Item 2 *    │ │ │ - Name                          │ │  │
│ Item 3      │ │ │ - Description (textarea)        │ │  │
│             │ │ │ - Other fields                  │ │  │
│ ┌─────────┐ │ │ │ - Switch/Toggle                 │ │  │
│ │🔄  +New │ │ │ └─────────────────────────────────┘ │  │
│ └─────────┘ │ │                                     │  │
│             │ ├─────────────────────────────────────┤  │
│             │ │ [🔄] [🗑️]          [Save Button] │  │
│             │ └─────────────────────────────────────┘  │
└─────────────┴───────────────────────────────────────────┘
```

## Key Components

### 1. Left Side - List View
- **Search input** at the top (filter items)
- **Scrollable list** of items with icons
- **Context menu** on right-click (delete, etc.)
- **Toolbar at bottom**:
  - Refresh button (left)
  - Add/New button (right)

### 2. Right Side - Detail View
- **Tabs** - One tab per open item
- **Tab labels** - Show item name + `*` if modified
- **Tab content**:
  - Form fields wrapped in `<Panel>` or sections
  - FormKit for data binding (NOT plain Ant Design Form)
- **Toolbar at bottom**:
  - Refresh button (left)
  - Delete button (center-left)
  - Save button (right, primary, disabled when pristine)

## Implementation Pattern

### Main Container Component

```typescript
import { ConfigLayout, Tabs, Content } from '@pimcore/studio-ui-bundle/components'
import { useApiListQuery } from './api/your-api-slice'

export const YourEntityContent = () => {
  // Fetch data
  const { data, isLoading, isFetching, refetch } = useApiListQuery()
  
  // Track which items are open as tabs
  const [openIds, setOpenIds] = useState<number[]>([])
  const [activeTabKey, setActiveTabKey] = useState<string | undefined>()
  const [modifiedItems, setModifiedItems] = useState<string[]>([])

  // Open a detail view in a new tab
  const openDetail = useCallback((id: number) => {
    setOpenIds((prev) => {
      if (prev.includes(id)) return prev
      return [...prev, id]
    })
    setActiveTabKey(String(id))
  }, [])

  // Close a tab
  const closeDetail = useCallback((key: number | string) => {
    const numKey = Number(key)
    
    setOpenIds((prev) => {
      const targetIndex = prev.indexOf(numKey)
      const updated = prev.filter((id) => id !== numKey)
      
      // Update active tab to adjacent tab if closing active
      setActiveTabKey((currentActiveKey) => {
        if (String(key) === currentActiveKey) {
          const nextId = prev[targetIndex - 1] ?? prev[targetIndex + 1]
          return nextId !== undefined ? String(nextId) : undefined
        }
        return currentActiveKey
      })
      
      return updated
    })
  }, [])

  // Create tab items
  const tabItems = useMemo(() => {
    return openIds.map((id) => {
      const item = data?.items?.find((i) => i.id === id)
      
      return {
        key: String(id),
        label: `${item?.name ?? id}${modifiedItems.includes(String(id)) ? ' *' : ''}`,
        children: (
          <YourEntityDetail
            id={id}
            modifiedItems={modifiedItems}
            setModifiedItems={setModifiedItems}
            onDelete={() => { handleDelete(id) }}
            onSave={() => { void refetch() }}
          />
        )
      }
    })
  }, [data?.items, modifiedItems, openIds])

  // Main content area
  const mainContent = () => {
    if (activeTabKey === undefined) {
      return <Content none />
    }
    
    return (
      <Tabs
        activeKey={activeTabKey}
        items={tabItems}
        onChange={setActiveTabKey}
        onClose={closeDetail}
      />
    )
  }

  return (
    <ConfigLayout
      leftItem={{
        children: (
          <YourEntityTree
            isFetching={isFetching}
            isLoading={isLoading}
            items={data?.items ?? []}
            onCloseDetail={closeDetail}
            onOpenDetail={openDetail}
            onRefetch={refetch}
          />
        )
      }}
      rightItem={{ children: mainContent() }}
    />
  )
}
```

### Left Side - List Component

```typescript
import { 
  ContentLayout, 
  Content, 
  Flex, 
  SearchInput, 
  Dropdown, 
  Icon, 
  Text,
  Toolbar,
  IconButton,
  IconTextButton
} from '@pimcore/studio-ui-bundle/components'

interface TreeProps {
  isLoading: boolean
  isFetching: boolean
  onRefetch: () => Promise<any>
  items: YourItemType[]
  onOpenDetail: (id: number) => void
  onCloseDetail: (id: number) => void
}

export const YourEntityTree = ({
  items,
  isLoading,
  isFetching,
  onRefetch,
  onOpenDetail,
  onCloseDetail
}: TreeProps) => {
  const { t } = useTranslation()
  const [searchTerm, setSearchTerm] = useState('')
  
  // Filter items by search
  const filteredItems = useMemo(() => {
    if (searchTerm === '') return items
    return items.filter(item => 
      item.name.toLowerCase().includes(searchTerm.toLowerCase())
    )
  }, [items, searchTerm])
  
  const handleAdd = async () => {
    // Show modal/form to create new item
    // Then refetch and open detail
  }
  
  const handleRemove = (id: number) => {
    // Show confirmation modal
    // Delete via API
    // Close detail view
    // Refetch list
  }
  
  return (
    <ContentLayout
      renderToolbar={
        <Toolbar>
          <IconButton
            disabled={isFetching}
            icon={{ value: 'refresh' }}
            onClick={() => { onRefetch() }}
          />
          
          <IconTextButton
            icon={{ value: 'new' }}
            onClick={handleAdd}
            type="link"
          >
            {t('new')}
          </IconTextButton>
        </Toolbar>
      }
    >
      <Content loading={isLoading} padded>
        <SearchInput
          onChange={(e) => setSearchTerm(e.target.value)}
          placeholder={t('search')}
          value={searchTerm}
          withoutAddon
        />
        
        <Flex gap="mini" vertical>
          {filteredItems.map((item) => (
            <Dropdown
              key={item.id}
              menu={{
                items: [
                  {
                    icon: <Icon value="trash" />,
                    key: 'delete',
                    label: t('delete'),
                    onClick: () => { handleRemove(item.id) }
                  }
                ]
              }}
              trigger={['contextMenu']}
            >
              <Flex
                align="center"
                gap="mini"
                onClick={() => { onOpenDetail(item.id) }}
                style={{ cursor: 'pointer', padding: '8px' }}
              >
                <Icon value="your-icon-name" />
                <Text>{item.name}</Text>
              </Flex>
            </Dropdown>
          ))}
        </Flex>
      </Content>
    </ContentLayout>
  )
}
```

### Right Side - Detail Component

```typescript
import { 
  ContentLayout, 
  Content, 
  Panel,
  Form,
  FormKit,
  Input,
  TextArea,
  InputNumber,
  Switch,
  Toolbar,
  IconButton,
  Button,
  Space,
  Tooltip
} from '@pimcore/studio-ui-bundle/components'

interface DetailProps {
  id: number
  modifiedItems: string[]
  setModifiedItems: (updater: (prev: string[]) => string[]) => void
  onSave: () => void
  onDelete: () => void
}

interface FormValues {
  name: string
  description: string
  threshold: number
  active: boolean
}

export const YourEntityDetail = ({
  id,
  modifiedItems,
  setModifiedItems,
  onSave,
  onDelete
}: DetailProps) => {
  const { t } = useTranslation()
  const [form] = Form.useForm<FormValues>()
  const initializedId = useRef<number | null>(null)
  const initialValuesRef = useRef<string>('')
  
  const isDirty = useMemo(
    () => modifiedItems.includes(String(id)),
    [modifiedItems, id]
  )
  
  // Fetch item data
  const { data, isLoading, isFetching, refetch } = useGetByIdQuery(
    { id },
    { refetchOnMountOrArgChange: true }
  )
  
  const [updateMutation, { isLoading: isSaving }] = useUpdateMutation()
  
  // Initialize form when data loads
  useEffect(() => {
    if (data && initializedId.current !== id) {
      const values: FormValues = {
        name: data.name,
        description: data.description,
        threshold: data.threshold,
        active: data.active
      }
      form.setFieldsValue(values)
      initialValuesRef.current = JSON.stringify(values)
      setModifiedItems((prev) => prev.filter(gid => gid !== String(id)))
      initializedId.current = id
    }
  }, [id, form, data, setModifiedItems])
  
  // Mark as dirty when form changes
  const onValuesChange = useCallback(() => {
    setModifiedItems((prev) => {
      const gid = String(id)
      return prev.includes(gid) ? prev : [...prev, gid]
    })
  }, [id, setModifiedItems])
  
  // Save handler
  const handleSave = useCallback(() => {
    if (!data) return
    
    form.validateFields()
      .then(async () => {
        const values = form.getFieldsValue(true) as FormValues
        
        await updateMutation({
          id,
          updateRequest: {
            description: values.description,
            threshold: values.threshold,
            active: values.active
          }
        }).unwrap()
        
        setModifiedItems((prev) => prev.filter(gid => gid !== String(id)))
        onSave()
      })
      .catch(() => {
        // Handle validation errors
      })
  }, [id, updateMutation, form, onSave, setModifiedItems, data])
  
  return (
    <ContentLayout
      className="h-full"
      renderToolbar={
        <Toolbar>
          <Space size="extra-small">
            <Tooltip title={t('refresh')}>
              <IconButton
                disabled={isFetching}
                icon={{ value: 'refresh' }}
                onClick={refetch}
              />
            </Tooltip>
            <Tooltip title={t('delete')}>
              <IconButton
                icon={{ value: 'trash' }}
                onClick={onDelete}
              />
            </Tooltip>
          </Space>
          
          <Button
            disabled={!isDirty}
            loading={isSaving}
            onClick={handleSave}
            type="primary"
          >
            {t('toolbar.save')}
          </Button>
        </Toolbar>
      }
    >
      <Content
        className="h-full"
        loading={isLoading || isFetching}
      >
        {!isLoading && !isFetching && data && (
          <Content gap="none" padded>
            <FormKit
              formProps={{
                form,
                layout: 'vertical',
                onValuesChange
              }}
            >
              <Panel title={t('general-settings')}>
                <Form.Item
                  label={t('name')}
                  name="name"
                >
                  <Input disabled placeholder={t('name')} />
                </Form.Item>
                
                <Form.Item
                  label={t('description')}
                  name="description"
                >
                  <TextArea
                    placeholder={t('description')}
                    rows={4}
                  />
                </Form.Item>
                
                <Form.Item
                  label={t('threshold')}
                  name="threshold"
                >
                  <InputNumber />
                </Form.Item>
                
                <Form.Item
                  name="active"
                  valuePropName="checked"
                >
                  <Switch labelRight={t('active')} />
                </Form.Item>
              </Panel>
            </FormKit>
          </Content>
        )}
      </Content>
    </ContentLayout>
  )
}
```

## Key Implementation Details

### State Management

You need to track three key pieces of state:

1. **openIds: number[]** - Which items have open tabs
2. **activeTabKey: string** - Which tab is currently active
3. **modifiedItems: string[]** - Which items have unsaved changes (shows `*` in tab label)

### Tab Management

**Opening tabs:**
- Check if ID already exists to prevent duplicates
- Add to `openIds` array
- Set as `activeTabKey`

**Closing tabs:**
- Remove from `openIds`
- Find next adjacent tab to activate
- If closing last tab, set `activeTabKey` to `undefined`

**Tab labels:**
- Show item name
- Add ` *` suffix if item has unsaved changes
- Use `modifiedItems` array to track dirty state

### Form Dirty Tracking

Track modifications at the **container level** (not just form instance):

1. Initialize `initialValuesRef` when data loads
2. Mark item as modified in `onValuesChange`
3. Remove from modified list after successful save
4. Disable save button when `!isDirty`

**Why track at container level?**
- Tab labels need to show `*` for unsaved changes
- Multiple tabs open simultaneously
- User can switch between tabs without losing dirty state

### Error Handling

- Show loading state while fetching
- Validate form before save
- Show success/error messages
- Handle network errors gracefully

## Common Mistakes

### ❌ Using Plain Form Instead of FormKit

```typescript
// DON'T DO THIS
<Form form={form} onFinish={handleSave}>
  <Form.Item name="name">
    <Input />
  </Form.Item>
</Form>
```

```typescript
// DO THIS
<FormKit
  formProps={{
    form,
    onValuesChange: handleChange
  }}
>
  <Form.Item name="name">
    <Input />
  </Form.Item>
</FormKit>
```

### ❌ Not Tracking Modified State

```typescript
// DON'T - No way to show * in tab label
const tabLabel = item.name

// DO - Track and display modified state
const tabLabel = `${item.name}${modifiedItems.includes(String(id)) ? ' *' : ''}`
```

### ❌ Forgetting to Handle Tab Closure

```typescript
// DON'T - Orphaned tabs when item deleted
const handleDelete = async (id: number) => {
  await deleteItem(id)
  refetch()
}

// DO - Close tab first
const handleDelete = async (id: number) => {
  await deleteItem(id)
  closeDetail(id)
  refetch()
}
```

### ❌ Not Using ContentLayout for Nested Areas

```typescript
// DON'T - Missing toolbar structure
<div>
  <div>Toolbar buttons</div>
  <div>Form content</div>
</div>

// DO - Use ContentLayout with renderToolbar
<ContentLayout
  renderToolbar={<Toolbar>...</Toolbar>}
>
  <Content padded>
    Form content
  </Content>
</ContentLayout>
```

## When to Use This Pattern

This pattern is perfect for:

- **Settings/Configuration pages** - User roles, API keys, webhooks
- **Entity management** - Categories, tags, target groups
- **List-based CRUD** - Any collection where users browse and edit individual items
- **Multi-tab editing** - Users need to compare or work on multiple items simultaneously

**Not suitable for:**
- Single-item editors (use simple form instead)
- Table-based bulk editing (use data grid)
- Wizard/multi-step flows (use stepper)

## Real-World Examples in Codebase

### Target Groups (Personalization Bundle)
- **Container:** `personalization-bundle/.../target-group-content.tsx`
- **List:** `personalization-bundle/.../target-group-tree/target-group-tree.tsx`
- **Detail:** `personalization-bundle/.../target-group-detail.tsx`
- **Toolbar:** `personalization-bundle/.../target-group-toolbar.tsx`

This is the canonical implementation - study it when implementing this pattern!

## Next Steps

- **pimcore-studio-ui-forms-antd** - Form fields and validation
- **pimcore-studio-ui-rtk-query-fundamentals** - API integration
- **pimcore-studio-ui-react-components** - Component structure
- **pimcore-studio-ui-layout-components** - Parent skill with all layout components
