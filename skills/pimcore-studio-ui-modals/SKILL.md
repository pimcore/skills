---
name: pimcore-studio-ui-modals
description: Using modal dialogs in Pimcore Studio UI - declarative Modal and WindowModal components, plus imperative useFormModal, useAlertModal, useStudioModal hooks
metadata:
  audience: pimcore-developers
  focus: ui-components-modals
---

## What This Skill Covers

How to use modal dialogs in Pimcore Studio UI:
- **Modal Component** - Declarative modal with open/onCancel props (preferred for complex UIs)
- **WindowModal Component** - Window-style modals with draggable, resizable behavior
- **useFormModal** - Imperative confirmation modals, input forms, textarea forms, upload forms
- **useAlertModal** - Imperative info, error, warning, and success alerts
- **useStudioModal** - Base modal functionality (works across iframes)
- When to use declarative vs imperative modals
- Modal types and patterns

## When to Use This Skill

Use this when:
- Creating modals with complex state management (use `<Modal>`)
- Building draggable/resizable window-style modals (use `<WindowModal>`)
- Confirming destructive actions (use `useFormModal.confirm()`)
- Collecting user input via modals
- Showing alerts or informational dialogs
- Creating forms in modal dialogs
- Uploading files via modal

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## Choosing the Right Modal Approach

### Declarative Components (`<Modal>`, `<WindowModal>`)

**Use declarative components when:**
- Modal has complex state management (forms with validation, multi-step workflows)
- Need full control over modal lifecycle
- Modal content changes based on component state
- Building custom modal layouts or behaviors
- Need to integrate with React hooks (useForm, useState, etc.)

**Example use cases:**
- Edit forms with RTK Query data
- Multi-step wizards
- Complex configuration dialogs
- Draggable/resizable windows

### Imperative Hooks (`useFormModal`, `useAlertModal`)

**Use imperative hooks when:**
- Simple confirmations (delete, yes/no)
- Quick user input (single field, textarea)
- Alert messages (success, error, info)
- Fire-and-forget interactions

**Example use cases:**
- "Are you sure you want to delete?"
- "Enter a name for this item"
- "Success! Item saved"

---

## Declarative Modal Component

The `<Modal>` component provides full control over modal behavior using React state.

### Basic Modal Usage

```typescript
import { useState } from 'react'
import { Modal, Button } from '@pimcore/studio-ui-bundle/components'

export const MyComponent = (): React.JSX.Element => {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <>
      <Button onClick={() => setIsOpen(true)}>
        Open modal
      </Button>

      <Modal
        open={isOpen}
        onCancel={() => setIsOpen(false)}
        title="My Modal"
        footer={[
          <Button key="cancel" onClick={() => setIsOpen(false)}>
            Cancel
          </Button>,
          <Button
            key="submit"
            type="primary"
            onClick={() => {
              // Handle submit
              setIsOpen(false)
            }}
          >
            Submit
          </Button>
        ]}
      >
        <p>Modal content goes here</p>
      </Modal>
    </>
  )
}
```

### Modal with Form and RTK Query

**Common pattern: Edit modal with data fetching and mutation**

```typescript
import { useState, useEffect } from 'react'
import { Modal, Button, FormKit, Form, Input, Skeleton } from '@pimcore/studio-ui-bundle/components'
import { useAssetGetByIdQuery, useAssetUpdateMutation } from '@pimcore/studio-ui-bundle/api/asset'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
import { isNil } from 'lodash'

export const EditAssetModal = ({ assetId, onClose }: Props): React.JSX.Element => {
  const [isOpen, setIsOpen] = useState(true)
  const [form] = Form.useForm()
  
  // Fetch data
  const { data, isLoading, error: fetchError } = useAssetGetByIdQuery({ id: assetId })
  
  // Mutation
  const [updateAsset, { isLoading: isUpdating, data: updateData, error: updateError }] = useAssetUpdateMutation()

  // Track errors
  useEffect(() => {
    if (!isNil(fetchError)) {
      trackError(new ApiError(fetchError))
    }
  }, [fetchError])

  useEffect(() => {
    if (!isNil(updateError)) {
      trackError(new ApiError(updateError))
    }
  }, [updateError])

  // Close modal on successful update
  useEffect(() => {
    if (!isNil(updateData)) {
      setIsOpen(false)
      onClose()
    }
  }, [updateData])

  // Set form values when data loads
  useEffect(() => {
    if (!isNil(data)) {
      form.setFieldsValue({
        filename: data.filename,
        title: data.metadata?.title ?? ''
      })
    }
  }, [data, form])

  const handleSubmit = (values: FormValues): void => {
    updateAsset({
      id: assetId,
      body: {
        filename: values.filename,
        metadata: { title: values.title }
      }
    })
  }

  const handleCancel = (): void => {
    setIsOpen(false)
    onClose()
  }

  return (
    <Modal
      open={isOpen}
      onCancel={handleCancel}
      title="Edit asset"
      footer={[
        <Button key="cancel" onClick={handleCancel}>
          Cancel
        </Button>,
        <Button
          key="submit"
          type="primary"
          loading={isUpdating}
          onClick={() => form.submit()}
        >
          Save
        </Button>
      ]}
    >
      {isLoading ? (
        <Skeleton />
      ) : (
        <FormKit formProps={{ form, onFinish: handleSubmit }}>
          <Form.Item name="filename" label="Filename" rules={[{ required: true }]}>
            <Input />
          </Form.Item>
          <Form.Item name="title" label="Title">
            <Input />
          </Form.Item>
        </FormKit>
      )}
    </Modal>
  )
}
```

### Modal Props

```typescript
interface ModalProps {
  open: boolean                    // Controls modal visibility
  onCancel?: () => void           // Called when user closes modal (X button, Esc, backdrop click)
  onOk?: () => void               // Called when OK button clicked (if using default footer)
  title?: React.ReactNode         // Modal title
  width?: number | string         // Modal width (default: 520)
  footer?: React.ReactNode        // Custom footer (null = no footer)
  closable?: boolean              // Show close X button (default: true)
  maskClosable?: boolean          // Click backdrop to close (default: true)
  destroyOnClose?: boolean        // Destroy children when closed (default: false)
  centered?: boolean              // Vertically center modal (default: false)
}
```

---

## WindowModal Component

The `<WindowModal>` component provides draggable, resizable window-style modals.

### Basic WindowModal Usage

```typescript
import { useState } from 'react'
import { WindowModal, Button } from '@pimcore/studio-ui-bundle/components'

export const MyComponent = (): React.JSX.Element => {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <>
      <Button onClick={() => setIsOpen(true)}>
        Open window
      </Button>

      <WindowModal
        open={isOpen}
        onClose={() => setIsOpen(false)}
        title="Draggable Window"
        width={800}
        height={600}
      >
        <div>
          <h3>Window content</h3>
          <p>This modal can be dragged and resized!</p>
        </div>
      </WindowModal>
    </>
  )
}
```

### WindowModal with Tabs

**Common pattern: Multi-section window**

```typescript
import { useState } from 'react'
import { WindowModal, Tabs } from '@pimcore/studio-ui-bundle/components'
import type { TabsProps } from '@pimcore/studio-ui-bundle/components'

export const SettingsWindow = ({ onClose }: Props): React.JSX.Element => {
  const [isOpen, setIsOpen] = useState(true)

  const items: TabsProps['items'] = [
    {
      key: 'general',
      label: 'General',
      children: <GeneralSettings />
    },
    {
      key: 'advanced',
      label: 'Advanced',
      children: <AdvancedSettings />
    }
  ]

  return (
    <WindowModal
      open={isOpen}
      onClose={() => {
        setIsOpen(false)
        onClose()
      }}
      title="Settings"
      width={900}
      height={700}
      resizable
      draggable
    >
      <Tabs items={items} />
    </WindowModal>
  )
}
```

### WindowModal Props

```typescript
interface WindowModalProps {
  open: boolean                   // Controls visibility
  onClose?: () => void           // Called when window closes
  title?: React.ReactNode        // Window title
  width?: number                 // Window width (default: 520)
  height?: number                // Window height (default: auto)
  resizable?: boolean            // Enable resize handles (default: false)
  draggable?: boolean            // Enable dragging by title bar (default: true)
  centered?: boolean             // Center window initially (default: true)
  destroyOnClose?: boolean       // Destroy children when closed (default: false)
}
```

---

## Imperative Modals (Hooks)

For simple interactions, use imperative hooks instead of declarative components.

## useFormModal Hook

The `useFormModal` hook provides modal dialogs with form functionality. The **most commonly used** is the **confirm** modal for user confirmations.

### 🚨 MOST IMPORTANT: Confirmation Modal

**The `confirm()` method is the most frequently used modal type!** Use it for:
- Confirming destructive actions (delete, remove, discard changes)
- Yes/No questions requiring explicit user confirmation
- Any action that needs user acknowledgment before proceeding

### Modal Types Available

```typescript
interface UseFormModalHookResponse {
  confirm: (props) => { destroy: () => void, update: (config) => void }  // ⭐ MOST COMMON
  input: (props) => { destroy: () => void, update: (config) => void }
  textarea: (props) => { destroy: () => void, update: (config) => void }
  upload: (props) => { destroy: () => void, update: (config) => void }
}
```

## Confirmation Modal (Most Common Use Case)

### Basic Confirmation Modal

**This is the most common pattern in Pimcore Studio UI:**

```typescript
import { useFormModal } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from 'react-i18next'

export const MyComponent = () => {
  const modal = useFormModal()
  const { t } = useTranslation()

  const handleDelete = () => {
    // ⭐ Most common modal pattern - confirmation dialog
    modal.confirm({
      title: t('delete.confirmation.title'),
      content: t('delete.confirmation.text'),
      okText: t('delete.confirmation.ok'),
      cancelText: t('cancel'),
      onOk: async () => {
        await deleteItem()
      }
    })
  }

  return (
    <Button
      color="danger"
      onClick={ handleDelete }
    >
      {t('delete')}
    </Button>
  )
}
```

### Confirmation Modal Props

```typescript
interface ConfirmFormModalProps {
  title?: string | React.ReactNode      // Modal title
  content?: string | React.ReactNode    // Modal content/message
  okText?: string                        // OK button text (default: "Yes")
  cancelText?: string                    // Cancel button text (default: "No")
  onOk?: () => void | Promise<void>     // OK callback (⭐ REQUIRED)
  onCancel?: () => void                  // Cancel callback (optional)
  dontAskAgainKey?: string              // Enable "Don't ask again" checkbox
  type?: 'info' | 'success' | 'error' | 'warning' | 'confirm'
  icon?: React.ReactNode                 // Custom icon
  okButtonProps?: ButtonProps            // OK button properties (e.g., { color: 'danger' })
  cancelButtonProps?: ButtonProps        // Cancel button properties
  width?: number | string                // Modal width
}
```

### Real-World Example: Delete Confirmation

```typescript
// File: widget-editor/hooks/use-widget-editor.tsx
import { useFormModal } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from 'react-i18next'

export const useWidgetEditor = () => {
  const modal = useFormModal()
  const { t } = useTranslation()
  const [deleteWidget] = useWidgetDeleteMutation()

  // ⭐ Standard delete confirmation pattern
  const removeWithConfirmation = (widgetId: string, onFinish?: () => void) => {
    modal.confirm({
      title: t('element.delete.confirmation.title'),
      content: <span>{t('element.delete.confirmation.text')}</span>,
      okText: t('element.delete.confirmation.ok'),
      onOk: async () => {
        await deleteWidget({ widgetId })
        onFinish?.()
      }
    })
  }

  return { removeWithConfirmation }
}
```

### Confirmation with "Don't Ask Again"

Use `dontAskAgainKey` to add a "Don't ask again" checkbox that stores the user's preference:

```typescript
modal.confirm({
  title: t('close.confirmation.title'),
  content: t('close.confirmation.text'),
  okText: t('yes'),
  cancelText: t('no'),
  dontAskAgainKey: 'close-without-saving-confirmation',  // Unique key for localStorage
  onOk: async () => {
    await closeWithoutSaving()
  }
})

// On subsequent calls, if user checked "Don't ask again", 
// the modal won't show and onOk() is called immediately
```

### Common Confirmation Modal Patterns

#### Pattern: Delete Action
```typescript
// ⭐ Most common pattern - delete with danger styling
modal.confirm({
  title: t('delete.confirmation.title'),
  content: t('delete.confirmation.text'),
  okText: t('delete'),
  okButtonProps: { color: 'danger' },  // Red button for destructive action
  onOk: async () => {
    await deleteItem()
    await messageApi.success(t('delete.success'))
  }
})
```

#### Pattern: Unsaved Changes
```typescript
// ⭐ Common pattern - warn about losing changes
modal.confirm({
  title: t('unsaved-changes.title'),
  content: t('unsaved-changes.text'),
  okText: t('discard-changes'),
  cancelText: t('keep-editing'),
  dontAskAgainKey: 'close-without-saving',  // Optional: remember choice
  onOk: () => {
    closeWithoutSaving()
  }
})
```

#### Pattern: Publish/Unpublish
```typescript
// ⭐ Common pattern - state change confirmation
modal.confirm({
  title: t('publish.confirmation.title'),
  content: t('publish.confirmation.text'),
  okText: t('publish'),
  onOk: async () => {
    await publishItem()
    await messageApi.success(t('publish.success'))
  }
})
```

#### Pattern: Generic Confirmation
```typescript
// ⭐ Simple yes/no confirmation
modal.confirm({
  title: t('confirm.title'),
  content: t('confirm.text'),
  okText: t('yes'),     // Default
  cancelText: t('no'),  // Default
  onOk: async () => {
    await performAction()
  }
})
```

## Input Modal

Displays a modal with a single text input field.

### Basic Input Modal

```typescript
const modal = useFormModal()
const { t } = useTranslation()

const handleRename = () => {
  modal.input({
    title: t('rename.title'),
    label: t('rename.label'),
    initialValue: currentName,
    okText: t('save'),
    cancelText: t('cancel'),
    onOk: async (newName: string) => {
      await updateName(newName)
    }
  })
}
```

### Input Modal Props

```typescript
interface InputFormModalProps {
  title?: string                         // Modal title
  label?: string                         // Input field label
  initialValue?: string                  // Pre-filled value
  rule?: Rule                           // Validation rule
  okText?: string
  cancelText?: string
  onOk?: (value: string) => void | Promise<void>
}
```

### Input Modal with Validation

```typescript
modal.input({
  title: t('create-folder.title'),
  label: t('create-folder.label'),
  rule: {
    required: true,
    message: t('create-folder.validation.required')
  },
  okText: t('create'),
  onOk: async (folderName: string) => {
    await createFolder(folderName)
  }
})
```

### Real-World Example: Rename Widget

```typescript
// File: open-element/context/open-element-data-context.tsx
const { input } = useFormModal()
const { t } = useTranslation()

const handleRenameWidget = (widgetId: string, currentName: string) => {
  input({
    title: t('widget.rename.title'),
    label: t('widget.rename.label'),
    initialValue: currentName,
    rule: {
      required: true,
      message: t('widget.rename.validation.required')
    },
    onOk: async (newName: string) => {
      await updateWidget(widgetId, { name: newName })
    }
  })
}
```

## Textarea Modal

Displays a modal with a multi-line textarea input.

### Basic Textarea Modal

```typescript
const modal = useFormModal()
const { t } = useTranslation()

const handleEditDescription = () => {
  modal.textarea({
    title: t('edit-description.title'),
    label: t('edit-description.label'),
    initialValue: currentDescription,
    placeholder: t('edit-description.placeholder'),
    okText: t('save'),
    onOk: async (newDescription: string) => {
      await updateDescription(newDescription)
    }
  })
}
```

### Textarea Modal Props

```typescript
interface TextareaFormModalProps {
  title?: string                         // Modal title
  label?: string                         // Textarea label
  initialValue?: string                  // Pre-filled value
  placeholder?: string                   // Placeholder text
  okText?: string
  cancelText?: string
  onOk?: (value: string) => void | Promise<void>
}
```

### Real-World Example: Edit Notes

```typescript
const modal = useFormModal()
const { t } = useTranslation()

const handleEditNotes = (currentNotes: string) => {
  modal.textarea({
    title: t('notes.edit.title'),
    label: t('notes.edit.label'),
    initialValue: currentNotes,
    placeholder: t('notes.edit.placeholder'),
    onOk: async (newNotes: string) => {
      await saveNotes(newNotes)
      await messageApi.success(t('notes.save.success'))
    }
  })
}
```

## Upload Modal

Displays a modal with a file upload input.

### Basic Upload Modal

```typescript
const modal = useFormModal()
const { t } = useTranslation()

const handleUploadFile = () => {
  modal.upload({
    title: t('upload.title'),
    label: t('upload.label'),
    accept: '.csv,.xlsx',  // Accepted file types
    okText: t('upload'),
    onOk: async (files: FileList) => {
      await uploadFiles(files)
    }
  })
}
```

### Upload Modal Props

```typescript
interface UploadFormModalProps {
  title?: string                         // Modal title
  label?: string                         // Upload field label
  accept?: string                        // Accepted file types (e.g., '.csv,.pdf')
  rule?: Rule                           // Validation rule
  okText?: string
  cancelText?: string
  onOk?: (files: FileList) => void | Promise<void>
}
```

### Upload with Validation

```typescript
modal.upload({
  title: t('import.title'),
  label: t('import.file-label'),
  accept: '.csv',
  rule: {
    required: true,
    message: t('import.validation.file-required')
  },
  onOk: async (files: FileList) => {
    if (files.length > 0) {
      await importData(files[0])
      await messageApi.success(t('import.success'))
    }
  }
})
```

## useAlertModal Hook

The `useAlertModal` hook provides simple alert-style modals for info, success, error, and warning messages.

### Alert Types

```typescript
interface UseAlertModalResponse {
  info: (props) => { destroy: () => void, update: (config) => void }
  success: (props) => { destroy: () => void, update: (config) => void }
  error: (props) => { destroy: () => void, update: (config) => void }
  warn: (props) => { destroy: () => void, update: (config) => void }
}
```

### Basic Alert Usage

```typescript
import { useAlertModal } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from 'react-i18next'

export const MyComponent = () => {
  const modal = useAlertModal()
  const { t } = useTranslation()

  const showInfo = () => {
    modal.info({
      title: 'information',  // Translation key (auto-translated)
      content: t('app.loading-info')
    })
  }

  const showError = () => {
    modal.error({
      title: 'error',  // Translation key (auto-translated)
      content: t('app.initialization-failed')
    })
  }

  const showSuccess = () => {
    modal.success({
      content: t('operation.completed')
      // title defaults to t('success')
    })
  }

  const showWarning = () => {
    modal.warn({
      content: t('unsaved-changes.warning')
      // title defaults to t('warning')
    })
  }

  return (
    // Your component
  )
}
```

### Alert Modal Props

```typescript
interface IAlertModalProps {
  title?: string                         // Title (translation key, auto-translated)
  content: string | React.ReactNode      // Content message
  okText?: string                        // OK button text
  onOk?: () => void                      // OK callback
  width?: number | string                // Modal width
}
```

### Real-World Example: App Initialization Error

```typescript
// File: app/app-loader/app-loader.tsx
import { useAlertModal } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from 'react-i18next'

export const AppLoader = () => {
  const modal = useAlertModal()
  const { t } = useTranslation()

  useEffect(() => {
    const initializeApp = async () => {
      try {
        await loadAppData()
      } catch (error) {
        modal.error({
          title: 'error',  // Auto-translated to t('error')
          content: t('app.initialization-failed'),
          onOk: () => {
            window.location.reload()
          }
        })
      }
    }

    initializeApp()
  }, [])

  return <div>Loading...</div>
}
```

## useStudioModal Hook

The `useStudioModal` hook provides the base modal functionality. It works seamlessly across iframe boundaries.

### Basic Usage

```typescript
import { useStudioModal } from '@pimcore/studio-ui-bundle/components'

export const MyComponent = () => {
  const { modal, localModal } = useStudioModal()

  const showConfirmation = () => {
    // `modal` works across iframes (uses parent window's modal if in iframe)
    modal.confirm({
      title: 'Confirm Action',
      content: 'Are you sure?',
      onOk: () => {
        // Handle confirmation
      }
    })
  }

  return <Button onClick={ showConfirmation }>Show Modal</Button>
}
```

### StudioModal Response

```typescript
interface StudioModalResponse {
  modal: ModalStaticFunctions        // Modal instance (parent if in iframe)
  localModal: ModalStaticFunctions   // Always local modal instance
}
```

### When to Use

- Use `modal` for most cases (works across iframes automatically)
- Use `localModal` when you specifically need the modal in the current window only

## Modal Control Methods

All modal hooks return methods to control the modal:

### Destroy Modal

```typescript
const modalInstance = modal.confirm({
  title: t('confirm.title'),
  content: t('confirm.text'),
  onOk: handleConfirm
})

// Later, close the modal programmatically
modalInstance.destroy()
```

### Update Modal

```typescript
const modalInstance = modal.confirm({
  title: t('processing.title'),
  content: t('processing.text'),
  okButtonProps: { loading: false }
})

// Update the modal (e.g., show loading state)
modalInstance.update({
  okButtonProps: { loading: true }
})

// Update again
setTimeout(() => {
  modalInstance.update({
    content: t('processing.complete'),
    okButtonProps: { loading: false }
  })
}, 2000)
```

## Common Patterns

### Pattern 1: Delete Confirmation

```typescript
const modal = useFormModal()
const { t } = useTranslation()
const [deleteItem, { isLoading }] = useDeleteMutation()

const handleDelete = (itemId: number) => {
  modal.confirm({
    title: t('delete.confirmation.title'),
    content: t('delete.confirmation.text'),
    okText: t('delete'),
    okButtonProps: { color: 'danger' },
    onOk: async () => {
      await deleteItem({ id: itemId })
    }
  })
}
```

### Pattern 2: Unsaved Changes Warning

```typescript
const modal = useFormModal()
const { t } = useTranslation()

const handleClose = (hasUnsavedChanges: boolean) => {
  if (!hasUnsavedChanges) {
    closeEditor()
    return
  }

  modal.confirm({
    title: t('unsaved-changes.title'),
    content: t('unsaved-changes.text'),
    okText: t('discard-changes'),
    cancelText: t('keep-editing'),
    dontAskAgainKey: 'close-without-saving',
    onOk: () => {
      closeEditor()
    }
  })
}
```

### Pattern 3: Create with Name Input

```typescript
const modal = useFormModal()
const { t } = useTranslation()
const [createItem] = useCreateMutation()

const handleCreate = () => {
  modal.input({
    title: t('create-item.title'),
    label: t('create-item.name-label'),
    rule: {
      required: true,
      message: t('create-item.validation.name-required'),
      pattern: /^[a-zA-Z0-9-_]+$/,
      message: t('create-item.validation.name-pattern')
    },
    onOk: async (name: string) => {
      await createItem({ name })
      await messageApi.success(t('create-item.success'))
    }
  })
}
```

### Pattern 4: Multi-Step Modal Updates

```typescript
const modal = useFormModal()
const { t } = useTranslation()

const handleComplexOperation = async () => {
  const modalInstance = modal.confirm({
    title: t('operation.title'),
    content: t('operation.step-1'),
    okText: t('continue'),
    cancelText: t('cancel'),
    onOk: async () => {
      // Show loading
      modalInstance.update({
        content: t('operation.processing'),
        okButtonProps: { loading: true },
        cancelButtonProps: { disabled: true }
      })

      try {
        await performStep1()
        
        // Update to step 2
        modalInstance.update({
          content: t('operation.step-2'),
          okButtonProps: { loading: false },
          cancelButtonProps: { disabled: false }
        })

        await performStep2()
        
        // Close and show success
        modalInstance.destroy()
        await messageApi.success(t('operation.success'))
      } catch (error) {
        modalInstance.destroy()
        await messageApi.error(t('operation.failed'))
      }
    }
  })
}
```

### Pattern 5: Error Alert with Retry

```typescript
const modal = useAlertModal()
const { t } = useTranslation()

const handleLoadDataWithRetry = async () => {
  try {
    await loadData()
  } catch (error) {
    modal.error({
      title: 'error',
      content: t('data.load-failed'),
      okText: t('retry'),
      onOk: () => {
        handleLoadDataWithRetry()  // Retry
      }
    })
  }
}
```

## When to Use Each Modal Type

### ⭐ useFormModal.confirm() - MOST COMMON!
**This is the most frequently used modal in Pimcore Studio UI!**

Use for:
- **Confirming destructive actions** (delete, remove, discard) - Most common!
- **Yes/No questions** requiring explicit user confirmation
- **State changes** that need confirmation (publish, unpublish)
- **Unsaved changes warnings** before closing/navigating
- Any action that needs explicit user acknowledgment before proceeding

```typescript
// ⭐ The pattern you'll use most often:
modal.confirm({
  title: t('action.confirmation.title'),
  content: t('action.confirmation.text'),
  okText: t('confirm'),
  onOk: async () => {
    await performAction()
  }
})
```

### useFormModal.input()
Use for:
- Collecting single-line text input (names, titles)
- Rename operations
- Creating items with a name

### useFormModal.textarea()
Use for:
- Collecting multi-line text (descriptions, notes, comments)
- Editing longer text content

### useFormModal.upload()
Use for:
- File upload operations
- Import/export with file selection
- Attachment uploads

### useAlertModal.info()
Use for:
- Informational messages
- Help or guidance

### useAlertModal.success()
Use for:
- Success confirmations (less common, prefer toast messages)
- Operation completed notifications

### useAlertModal.error()
Use for:
- Critical errors that block workflow
- Initialization failures
- Errors requiring user acknowledgment

### useAlertModal.warn()
Use for:
- Non-blocking warnings
- Important notices

## Common Mistakes

### ❌ Not Using Translations

```typescript
// DON'T - Hardcoded text
modal.confirm({
  title: 'Delete Item',
  content: 'Are you sure?'
})

// DO - Always use translations
modal.confirm({
  title: t('delete.confirmation.title'),
  content: t('delete.confirmation.text')
})
```

### ❌ Not Handling Async Operations

```typescript
// DON'T - Not awaiting
modal.confirm({
  onOk: () => {
    deleteItem()  // Fire and forget
  }
})

// DO - Await async operations
modal.confirm({
  onOk: async () => {
    await deleteItem()
    await messageApi.success(t('deleted'))
  }
})
```

### ❌ Using Modal for Simple Notifications

```typescript
// DON'T - Modal for simple success message
modal.success({
  content: t('saved')
})

// DO - Use toast message instead
messageApi.success(t('saved'))
```

### ❌ Not Providing Feedback

```typescript
// DON'T - No feedback after action
modal.confirm({
  title: t('delete.title'),
  content: t('delete.text'),
  onOk: async () => {
    await deleteItem()
    // No feedback!
  }
})

// DO - Always provide feedback
modal.confirm({
  title: t('delete.title'),
  content: t('delete.text'),
  onOk: async () => {
    await deleteItem()
    await messageApi.success(t('delete.success'))  // Feedback!
  }
})
```

### ❌ Missing Validation in Input Modals

```typescript
// DON'T - No validation
modal.input({
  title: t('create.title'),
  onOk: async (name: string) => {
    await create(name)  // Could be empty or invalid!
  }
})

// DO - Add validation rules
modal.input({
  title: t('create.title'),
  rule: {
    required: true,
    message: t('validation.name-required')
  },
  onOk: async (name: string) => {
    await create(name)
  }
})
```

## Quick Reference

### Modal Type Selection

| Need | Use |
|------|-----|
| Confirm action | `useFormModal().confirm()` |
| Get single-line input | `useFormModal().input()` |
| Get multi-line input | `useFormModal().textarea()` |
| Upload file | `useFormModal().upload()` |
| Show info | `useAlertModal().info()` or toast |
| Show error | `useAlertModal().error()` for critical |
| Show success | Toast (not modal) |
| Show warning | `useAlertModal().warn()` or toast |

### Common Modal Configurations

```typescript
// Delete confirmation
modal.confirm({
  title: t('delete.title'),
  content: t('delete.text'),
  okText: t('delete'),
  okButtonProps: { color: 'danger' },
  onOk: async () => await delete()
})

// Create with input
modal.input({
  title: t('create.title'),
  label: t('name.label'),
  rule: { required: true, message: t('validation.required') },
  onOk: async (name) => await create(name)
})

// Don't ask again
modal.confirm({
  title: t('close.title'),
  content: t('close.text'),
  dontAskAgainKey: 'close-confirmation',
  onOk: () => close()
})
```

## Next Steps

- [**pimcore-studio-ui-notifications-toasts**](../pimcore-studio-ui-notifications-toasts/SKILL.md) - Toast messages for simple feedback
- [**pimcore-studio-ui-forms-antd**](../pimcore-studio-ui-forms-antd/SKILL.md) - Form components and validation
- [**pimcore-studio-ui-buttons**](../pimcore-studio-ui-buttons/SKILL.md) - Buttons that trigger modals
- [**pimcore-studio-ui-i18n**](../pimcore-studio-ui-i18n/SKILL.md) - Translation keys for modal content
