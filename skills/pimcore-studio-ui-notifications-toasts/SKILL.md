---
name: pimcore-studio-ui-notifications-toasts
description: Displaying notification messages and toasts in Pimcore Studio UI using the useMessage hook
metadata:
  audience: pimcore-developers
  focus: ui-components-notifications
---

## What This Skill Covers

How to display notification messages (toasts) in Pimcore Studio UI:
- **useMessage** hook for displaying notifications
- Success messages for completed operations
- Error messages for failures
- Info and warning messages
- Message types and configuration
- Duration and auto-dismiss behavior
- Custom icons and content

## When to Use This Skill

Use this when:
- Showing feedback after save/update operations
- Displaying error messages to users
- Confirming successful actions (copy, delete, create)
- Showing warnings or informational messages
- Providing user feedback for async operations

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## useMessage Hook

The `useMessage` hook provides a message API for displaying toast notifications.

### Basic Usage

```typescript
import { useMessage } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from 'react-i18next'

export const MyComponent = (): React.JSX.Element => {
  const messageApi = useMessage()
  const { t } = useTranslation()

  const handleSave = async (): Promise<void> => {
    try {
      await saveData()
      await messageApi.success(t('save-success'))
    } catch (error) {
      await messageApi.error({
        content: error.message
      })
    }
  }

  return (
    <Button onClick={ handleSave }>
      {t('save')}
    </Button>
  )
}
```

### Message API Methods

```typescript
interface MessageInstance {
  success: (content: string | ArgsProps, duration?, onClose?) => MessageType
  error: (content: string | ArgsProps, duration?, onClose?) => MessageType
  warning: (content: string | ArgsProps, duration?, onClose?) => MessageType
  info: (content: string | ArgsProps, duration?, onClose?) => MessageType
  loading: (content: string | ArgsProps, duration?, onClose?) => MessageType
  open: (config: ArgsProps) => MessageType
}
```

## Success Messages

Show success messages after successful operations.

### Simple Success Message

```typescript
// Simple text message
await messageApi.success(t('save-success'))

// With custom duration (seconds)
await messageApi.success(t('save-success'), 5)

// With callback after close
await messageApi.success(t('save-success'), 3, () => {
  console.log('Message closed')
})
```

### Success Message with Config

```typescript
messageApi.success({
  content: t('operation-completed'),
  duration: 3,  // seconds (0 = never auto-close)
  onClose: () => {
    // Callback when message closes
  }
})
```

### Real-World Example: Save Success

```typescript
// File: asset/editor/toolbar/save-button/save-button.tsx
import { useMessage } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from 'react-i18next'

export const EditorToolbarSaveButton = (): React.JSX.Element => {
  const { t } = useTranslation()
  const messageApi = useMessage()
  const [saveAsset, { isSuccess }] = useAssetUpdateByIdMutation()

  useEffect(() => {
    const handleSuccessEvent = async (): Promise<void> => {
      if (isSuccess) {
        await messageApi.success(t('save-success'))
      }
    }

    handleSuccessEvent().catch((error) => {
      console.error(error)
    })
  }, [isSuccess])

  return (
    <Button
      onClick={ handleSave }
      type="primary"
    >
      {t('save')}
    </Button>
  )
}
```

## Error Messages

Display error messages when operations fail.

### Simple Error Message

```typescript
// Simple text error
await messageApi.error(t('save-failed'))

// With error details
await messageApi.error({
  content: t('save-failed')
})
```

### Error with Exception Details

```typescript
try {
  await saveData()
} catch (error) {
  await messageApi.error({
    content: error.message || t('unknown-error')
  })
}
```

### Real-World Example: Login Error

```typescript
// File: auth/components/login-form/login-form.tsx
import { useMessage } from '@pimcore/studio-ui-bundle/components'

export const LoginForm = (): React.JSX.Element => {
  const messageApi = useMessage()

  const handleAuthentication = async (event: React.FormEvent): Promise<void> => {
    try {
      event.preventDefault()
      const response = await login({ credentials: formState })

      if (response.error !== undefined) {
        await messageApi.error({
          content: t('login-failed')
        })
      } else {
        dispatch(setAuthState(true))
      }
    } catch (e: any) {
      await messageApi.error({
        content: e.message
      })
    }
  }

  return (
    <form onSubmit={ handleAuthentication }>
      {/* form fields */}
    </form>
  )
}
```

### Real-World Example: Auto-Save Failure

```typescript
// File: data-object/editor/providers/edit-form-provider.tsx
useEffect(() => {
  if (isError) {
    messageApi.error(t('auto-save-failed'))
  }
}, [isError])
```

## Warning Messages

Display warning messages for non-critical issues.

### Simple Warning

```typescript
messageApi.warning(t('unsaved-changes'))

messageApi.warning({
  content: t('operation-may-take-time'),
  duration: 5
})
```

### Warning with Action Needed

```typescript
const handleDelete = (): void => {
  if (hasRelatedItems) {
    messageApi.warning({
      content: t('item-has-dependencies'),
      duration: 0  // Don't auto-close
    })
    return
  }
  
  performDelete()
}
```

## Info Messages

Display informational messages.

### Simple Info Message

```typescript
messageApi.info(t('loading-data'))

messageApi.info({
  content: t('background-task-started'),
  duration: 3
})
```

### Info with Custom Icon

The `useMessage` hook automatically adds custom icons for info messages:

```typescript
// Automatically includes info-circle icon
messageApi.info(t('processing'))
```

## Loading Messages

Display loading messages for ongoing operations.

### Basic Loading Message

```typescript
const hideLoading = messageApi.loading(t('saving'))

// Later, hide the message manually
hideLoading()
```

### Loading with Duration

```typescript
// Auto-hide after 3 seconds
messageApi.loading(t('processing'), 3)
```

### Real-World Pattern: Long Operation

```typescript
const handleExport = async (): Promise<void> => {
  const hide = messageApi.loading(t('exporting-data'), 0)
  
  try {
    await performExport()
    hide()
    await messageApi.success(t('export-complete'))
  } catch (error) {
    hide()
    await messageApi.error(t('export-failed'))
  }
}
```

## Advanced Configuration

### Message Configuration Options

```typescript
interface ArgsProps {
  content: React.ReactNode         // Message content
  duration?: number                // Duration in seconds (0 = manual close)
  onClose?: VoidFunction          // Callback when closed
  icon?: React.ReactNode          // Custom icon element
  key?: string | number           // Unique key for the message
  className?: string              // Custom CSS class
  style?: React.CSSProperties     // Custom inline styles
  onClick?: (e: React.MouseEvent) => void  // Click handler
}
```

### Custom Duration

```typescript
// Show for 5 seconds (default is 3)
messageApi.success(t('saved'), 5)

// Never auto-close (must be closed manually)
messageApi.success({
  content: t('important-message'),
  duration: 0
})

// Close immediately
messageApi.success(t('quick-message'), 0.5)
```

### Manual Message Control

```typescript
// Store reference to close manually
const hide = messageApi.success({
  content: t('uploading'),
  duration: 0
})

// Later, close the message
setTimeout(() => {
  hide()
}, 5000)
```

### Sequential Messages

```typescript
const handleMultiStepOperation = async (): Promise<void> => {
  // Step 1
  const loading = messageApi.loading(t('step-1'))
  await performStep1()
  loading()
  
  // Step 2
  const loading2 = messageApi.loading(t('step-2'))
  await performStep2()
  loading2()
  
  // Final success
  await messageApi.success(t('all-steps-complete'))
}
```

## Common Patterns

### Pattern 1: Save Operation Feedback

```typescript
const [saveData, { isLoading, isSuccess, isError, error }] = useSaveMutation()
const messageApi = useMessage()
const { t } = useTranslation()

useEffect(() => {
  if (isSuccess) {
    void messageApi.success(t('save-success'))
  }
}, [isSuccess])

useEffect(() => {
  if (isError) {
    void messageApi.error(t('save-failed'))
  }
}, [isError])
```

### Pattern 2: Copy/Paste Confirmation

```typescript
// File: element/actions/copy-paste/tree-copy-paste-context.tsx
const handleCopy = (element: Element): void => {
  setCopiedElement({ element, mode: 'copy' })
  
  void messageApi.success(t('element.tree.copy-success-description', {
    type: element.type,
    path: element.path
  }))
}

const handleCut = (element: Element): void => {
  setCopiedElement({ element, mode: 'cut' })
  
  void messageApi.success(t('element.tree.cut-success-description', {
    type: element.type,
    path: element.path
  }))
}
```

### Pattern 3: Data Cleared Confirmation

```typescript
// File: dynamic-types/.../hotspot-image/footer.tsx
const handleClearData = async (): Promise<void> => {
  clearHotspots()
  await messageApi.success(t('hotspots.data-cleared'))
}
```

### Pattern 4: Batch Operation Results

```typescript
const handleBatchDelete = async (items: Item[]): Promise<void> => {
  const loading = messageApi.loading(t('deleting-items', { count: items.length }), 0)
  
  try {
    const results = await Promise.allSettled(
      items.map(item => deleteItem(item.id))
    )
    
    loading()
    
    const succeeded = results.filter(r => r.status === 'fulfilled').length
    const failed = results.filter(r => r.status === 'rejected').length
    
    if (failed === 0) {
      await messageApi.success(t('batch-delete-success', { count: succeeded }))
    } else {
      await messageApi.warning(t('batch-delete-partial', { 
        succeeded, 
        failed 
      }))
    }
  } catch (error) {
    loading()
    await messageApi.error(t('batch-delete-failed'))
  }
}
```

### Pattern 5: Tag Configuration Actions

```typescript
// File: tags/hooks/use-tag-config.tsx
const handleCreateTag = async (): Promise<void> => {
  const result = await createTag(tagData)
  
  if (result.success) {
    messageApi.success({
      content: t('tag.created', { name: tagData.name }),
      duration: 3
    })
  }
}

const handleUpdateTag = async (): Promise<void> => {
  const result = await updateTag(tagData)
  
  if (result.success) {
    messageApi.success({
      content: t('tag.updated'),
      duration: 2
    })
  }
}

const handleDeleteTag = async (): Promise<void> => {
  const result = await deleteTag(tagId)
  
  if (result.success) {
    messageApi.success({
      content: t('tag.deleted'),
      duration: 2
    })
  }
}
```

## Message Types Comparison

| Type | Use Case | Default Duration | Auto Icon |
|------|----------|------------------|-----------|
| `success` | Successful operations | 3s | ✓ (checkmark) |
| `error` | Failed operations | 3s | ✓ (error) |
| `warning` | Non-critical issues | 3s | ✓ (warning) |
| `info` | Information | 3s | ✓ (info-circle, custom) |
| `loading` | Ongoing operations | Manual | ✓ (spinner) |

## Common Mistakes

### ❌ Not Using Translation Keys

```typescript
// DON'T - Hardcoded text
messageApi.success('Saved successfully')

// DO - Use i18n
messageApi.success(t('save-success'))
```

### ❌ Forgetting Error Handling

```typescript
// DON'T - No error feedback
const handleSave = async (): Promise<void> => {
  await saveData()
  messageApi.success(t('saved'))
}

// DO - Handle errors
const handleSave = async (): Promise<void> => {
  try {
    await saveData()
    await messageApi.success(t('saved'))
  } catch (error) {
    await messageApi.error(t('save-failed'))
  }
}
```

### ❌ Not Awaiting Messages

```typescript
// DON'T - Not awaiting (may cause timing issues)
messageApi.success(t('saved'))
doNextThing()

// DO - Await message
await messageApi.success(t('saved'))
doNextThing()
```

### ❌ Using console.log Instead of Messages

```typescript
// DON'T - User can't see console
console.log('Data saved')

// DO - Show user-visible message
messageApi.success(t('save-success'))
```

### ❌ Too Many Messages

```typescript
// DON'T - Spam the user
items.forEach(item => {
  messageApi.success(t('item-saved', { name: item.name }))
})

// DO - Single summary message
messageApi.success(t('items-saved', { count: items.length }))
```

### ❌ Wrong Message Type

```typescript
// DON'T - Using error for non-errors
if (items.length === 0) {
  messageApi.error(t('no-items'))  // Not an error!
}

// DO - Use info or warning
if (items.length === 0) {
  messageApi.info(t('no-items'))
}
```

## Best Practices

### 1. Always Provide User Feedback

```typescript
// For any user action, provide feedback
const handleAction = async (): Promise<void> => {
  try {
    await performAction()
    await messageApi.success(t('action-complete'))  // Always confirm
  } catch (error) {
    await messageApi.error(t('action-failed'))      // Always report errors
  }
}
```

### 2. Use Appropriate Message Types

```typescript
// Success: Completed operations
messageApi.success(t('saved'))

// Error: Failed operations
messageApi.error(t('save-failed'))

// Warning: Potential issues
messageApi.warning(t('unsaved-changes'))

// Info: Informational updates
messageApi.info(t('data-loading'))

// Loading: Ongoing operations
messageApi.loading(t('processing'))
```

### 3. Keep Messages Concise

```typescript
// DO - Short and clear
messageApi.success(t('saved'))

// AVOID - Too verbose
messageApi.success(t('your-changes-have-been-successfully-saved-to-the-database'))
```

### 4. Use Translation Interpolation

```typescript
// Include dynamic data in messages
messageApi.success(t('item-created', { name: item.name }))
messageApi.warning(t('items-remaining', { count: remaining }))
```

### 5. Handle Async Operations Properly

```typescript
// Show loading, then result
const handleExport = async (): Promise<void> => {
  const hide = messageApi.loading(t('exporting'), 0)
  
  try {
    const result = await exportData()
    hide()
    await messageApi.success(t('export-complete'))
  } catch (error) {
    hide()
    await messageApi.error(t('export-failed'))
  }
}
```

## Quick Reference

### Common Use Cases

```typescript
// Simple success
await messageApi.success(t('saved'))

// Simple error
await messageApi.error(t('failed'))

// With duration
messageApi.success(t('saved'), 5)

// Never auto-close
messageApi.warning({
  content: t('warning'),
  duration: 0
})

// Loading (manual close)
const hide = messageApi.loading(t('loading'), 0)
hide()  // Close it later

// With interpolation
messageApi.success(t('created', { name: item.name }))
```

## Next Steps

- [**pimcore-studio-ui-i18n**](../pimcore-studio-ui-i18n/SKILL.md) - Translation system for message content
- [**pimcore-studio-ui-modals**](../pimcore-studio-ui-modals/SKILL.md) - Using messages with modal dialogs
- [**pimcore-studio-ui-rtk-query-fundamentals**](../pimcore-studio-ui-rtk-query-fundamentals/SKILL.md) - Handling API errors with messages
- [**pimcore-studio-ui-buttons**](../pimcore-studio-ui-buttons/SKILL.md) - Triggering messages from button actions
