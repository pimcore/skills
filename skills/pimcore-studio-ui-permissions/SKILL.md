---
name: pimcore-studio-ui-permissions
description: Checking and handling permissions in Pimcore Studio UI - element permissions, user permissions, perspective permissions, and writeable state for config-based features
metadata:
  audience: pimcore-developers
  focus: permissions-security
---

## What This Skill Covers

How to check and handle permissions in Pimcore Studio UI:
- **Element Permissions** - Check if user can perform actions on specific elements (assets, documents, data objects)
- **User Permissions** - Check if user has global permissions (e.g., access to certain features)
- **Perspective Permissions** - Check if features are visible/accessible in the current perspective
- **Writeable State** - Special case for config-based features (perspectives, widgets, settings) using location-aware config system
- **Conditional Rendering** - Show/hide UI elements based on permissions
- **Disabled States** - Disable buttons and actions when permissions are missing or config is not writeable
- **Real-World Patterns** - Common permission checking patterns in save buttons, toolbars, and actions

## When to Use This Skill

Use this when:
- Implementing save, publish, delete, or edit actions on elements
- Controlling access to features based on user roles
- Hiding/showing navigation items based on perspectives
- Disabling buttons when user lacks required permissions
- Building permission-aware toolbars and action menus
- Implementing element-specific actions (e.g., tree context menu)
- **Handling writeable state for config-based features (settings store, perspectives, widgets, etc.)**

## 🚨 Special Case: Config Writeable State

Some features use Pimcore's **location-aware config system** (settings-store) which determines if configurations can be modified. This is different from user permissions!

### The `isWriteable` / `writeable` Property

Configuration-based features (perspectives, widgets, thumbnails, appearance settings) include a `writeable` or `isWriteable` property that indicates if the config can be saved.

**When NOT writeable:**
- ✅ **Always disable the save button**
- ✅ **Always disable the delete button** 
- ✅ **Always show the same standard tooltip**
- ✅ **Translation key: `config_not_writeable`**
- ✅ **Tooltip text: "Not writable, this can have several reasons. For details please have a look into the docs."**

### Why Not Writeable?

Common reasons (location-aware config system):
- Config is defined in a PHP file or environment variable (higher priority than database)
- Config is locked at system level
- Config source is read-only
- Multiple config sources with priority conflicts

### Pattern: Config Forms with Writeable Check

**Standard pattern for all config-based forms:**

```typescript
import { Tooltip } from 'antd'
import { Button, IconButton, Content, Toolbar, Flex } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from 'react-i18next'

// Works for: Perspectives, Widgets, Thumbnails, Settings
export const ConfigForm = (): React.JSX.Element => {
  const { t } = useTranslation()
  const { config, isLoading } = useConfig(id) // Could be usePerspective, useWidget, etc.
  
  // Check writeable - property name varies: isWriteable or writeable
  const isWriteable = config.isWriteable ?? config.writeable ?? false
  
  return (
    <FormKit onFinish={handleSave}>
      <Flex vertical justify="space-between" className="absolute-stretch">
        <Content padded>
          {/* Form fields */}
        </Content>
        
        <Toolbar justify="space-between">
          <div>
            <IconButton
              disabled={isLoading}
              icon={{ value: 'refresh' }}
              onClick={() => form.resetFields()}
            />
            
            {/* Delete button - wrap with writeable tooltip */}
            <Tooltip title={isWriteable ? '' : t('config_not_writeable')}>
              <IconButton
                disabled={isLoading || !isWriteable}
                icon={{ value: 'trash' }}
                onClick={handleDelete}
              />
            </Tooltip>
          </div>
          
          {/* Save button - wrap with writeable tooltip */}
          <Tooltip title={isWriteable ? '' : t('config_not_writeable')}>
            <Button
              disabled={!isWriteable}
              htmlType="submit"
              loading={isLoading}
              type="primary"
            >
              {t('save')}
            </Button>
          </Tooltip>
        </Toolbar>
      </Flex>
    </FormKit>
  )
}
```

### Real-World Examples

**Perspective Editor** (`perspective-editor/components/perspective-form/perspective-form.tsx`):
```typescript
const isWriteable = perspective.isWriteable
```

**Widget Editor** (`widget-editor/components/widget-type-form/widget-form.tsx`):
```typescript
const isWriteable = widget.isWriteable !== false
```

**Appearance Settings** (`appearance-branding/components/appearance-form/appearance-form.tsx`):
```typescript
const isWriteable = adminSettings?.writeable ?? false
```

### Writeable Check Checklist

When working with config-based features:

- [ ] Check for `isWriteable` or `writeable` property in API response
- [ ] Wrap save button with `<Tooltip title={isWriteable ? '' : t('config_not_writeable')}>`
- [ ] Disable save button: `disabled={!isWriteable}`
- [ ] Wrap delete button (if applicable) with same tooltip pattern
- [ ] Disable delete button: `disabled={isLoading || !isWriteable}`
- [ ] **ALWAYS use `t('config_not_writeable')` translation key** - never create custom messages
- [ ] Empty tooltip when writeable: `title={isWriteable ? '' : t('config_not_writeable')}`

### Common Features Using Writeable Check

These features typically have writeable state:
- **Perspectives** (`isWriteable`)
- **Widgets** (`isWriteable`)
- **Image Thumbnails** (`writeable`)
- **Video Thumbnails** (`writeable`)
- **Appearance/Branding Settings** (`writeable`)
- **Custom Settings** (any settings-store based config)

### Writeable vs User Permissions

| Type | Checks | Reason for Restriction | Tooltip |
|------|--------|----------------------|---------|
| **Writeable** | Config source/location | System configuration override | `config_not_writeable` |
| **User Permission** | User role/capabilities | Access control/security | Custom per feature |
| **Element Permission** | Element-specific rights | Per-element access control | Custom per action |

**IMPORTANT:** Writeable state is **NOT** a permission check - it's a config system constraint. Always check writeable state separately from permissions!

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

> **Note:** `isAllowedInPerspective` is not exported via the SDK. It is only available in core via `@Pimcore/modules/perspectives/permission-checker`.

## Element Permissions

Check if a user can perform specific actions on an element (asset, document, or data object).

### `checkElementPermission()` Function

```typescript
checkElementPermission(
  permissions: ElementPermissions | undefined,
  permission: ElementPermissionKeys
): boolean
```

**Returns:** `true` if user has the permission, `false` otherwise

### Available Element Permissions

Common element permission keys:
- `'publish'` - Can save and publish the element
- `'delete'` - Can delete the element
- `'rename'` - Can rename the element
- `'view'` - Can view the element
- `'properties'` - Can edit properties
- `'versions'` - Can view/restore versions
- `'settings'` - Can edit settings

### Basic Usage

```typescript
import { checkElementPermission } from '@pimcore/studio-ui-bundle/modules/element'
import { useAssetDraft } from '@pimcore/studio-ui-bundle/modules/asset'

export const SaveButton = (): React.JSX.Element => {
  const { asset } = useAssetDraft(id)
  
  // Check if user has publish permission
  const canPublish = checkElementPermission(asset?.permissions, 'publish')
  
  return (
    <>
      {canPublish && (
        <Button onClick={handleSave} type="primary">
          Save and publish
        </Button>
      )}
    </>
  )
}
```

### Pattern 1: Conditional Rendering

**Show/hide UI elements based on permissions:**

```typescript
import { checkElementPermission } from '@pimcore/studio-ui-bundle/modules/element'
import { Button } from '@pimcore/studio-ui-bundle/components'

export const EditorToolbar = (): React.JSX.Element => {
  const { t } = useTranslation()
  const { asset } = useAssetDraft(id)
  
  return (
    <Toolbar>
      {/* Only show save button if user can publish */}
      {checkElementPermission(asset?.permissions, 'publish') && (
        <Button
          onClick={handleSave}
          type="primary"
        >
          {t('toolbar.save')}
        </Button>
      )}
      
      {/* Only show delete button if user can delete */}
      {checkElementPermission(asset?.permissions, 'delete') && (
        <IconButton
          color="danger"
          icon={{ value: 'delete' }}
          onClick={handleDelete}
          tooltip={{ title: t('delete') }}
        />
      )}
    </Toolbar>
  )
}
```

### Pattern 2: Disabled State

**Disable actions when permissions are missing:**

```typescript
export const ActionButtons = (): React.JSX.Element => {
  const { element } = useElement(id)
  
  const canPublish = checkElementPermission(element?.permissions, 'publish')
  const canDelete = checkElementPermission(element?.permissions, 'delete')
  
  return (
    <ButtonGroup
      items={[
        <Button
          disabled={!canPublish}
          key="save"
          onClick={handleSave}
          type="primary"
        >
          {t('save')}
        </Button>,
        <Button
          color="danger"
          disabled={!canDelete}
          key="delete"
          onClick={handleDelete}
        >
          {t('delete')}
        </Button>
      ]}
    />
  )
}
```

### Pattern 3: Conditional Behavior

**Check permissions before executing actions:**

```typescript
import { checkElementPermission } from '@pimcore/studio-ui-bundle/modules/element'
import { useMessage } from '@pimcore/studio-ui-bundle/components'

export const useElementActions = (id: number) => {
  const { element } = useElement(id)
  const messageApi = useMessage()
  
  const handleSave = (): void => {
    // Check permission before saving
    if (!checkElementPermission(element?.permissions, 'publish')) {
      messageApi.error('You do not have permission to publish this element')
      return
    }
    
    // Proceed with save
    saveElement()
  }
  
  const handleDelete = (): void => {
    // Check permission before deleting
    if (!checkElementPermission(element?.permissions, 'delete')) {
      messageApi.error('You do not have permission to delete this element')
      return
    }
    
    // Show confirmation modal
    showDeleteConfirmation()
  }
  
  return { handleSave, handleDelete }
}
```

### Real-World Example: Save Button with Permissions

```typescript
// File: asset/editor/toolbar/save-button/save-button.tsx
import { Button } from '@pimcore/studio-ui-bundle/components'
import { useAssetDraft } from '@pimcore/studio-ui-bundle/modules/asset'
import { checkElementPermission } from '@pimcore/studio-ui-bundle/modules/element'
import { useHandleKeyBindings } from '@pimcore/studio-ui-bundle/modules/app'

export const EditorToolbarSaveButton = (): React.JSX.Element => {
  const { t } = useTranslation()
  const { id } = useElementContext()
  const { asset } = useAssetDraft(id)
  const [saveAsset, { isLoading }] = useAssetUpdateByIdMutation()
  
  // Register keyboard shortcut only if user has permission
  useHandleKeyBindings(async () => {
    if (asset != null && checkElementPermission(asset.permissions, 'publish')) {
      onSaveClick()
    }
  }, 'save')
  
  return (
    <>
      {/* Only render button if user has publish permission */}
      {checkElementPermission(asset?.permissions, 'publish') && (
        <Button
          disabled={isLoading}
          loading={isLoading}
          onClick={onSaveClick}
          type="primary"
        >
          {t(asset?.type === 'folder' ? 'toolbar.save' : 'toolbar.save-and-publish')}
        </Button>
      )}
    </>
  )
  
  function onSaveClick(): void {
    if (asset?.changes === undefined) return
    
    // Save logic...
    saveAsset({ id, body: { data: {...} } })
  }
}
```

## User Permissions

Check if the current user has global permissions for features (not element-specific).

### `isAllowed()` Function

```typescript
isAllowed(permission: UserPermission | undefined): boolean
```

**Returns:** `true` if user has the permission or is admin, `false` otherwise

**Note:** Admins always return `true` for any permission check.

### UserPermission Enum

Available user permissions from `@pimcore/studio-ui-bundle/modules/auth`:

```typescript
export enum UserPermission {
  NotesAndEvents = 'notes_events',
  Translations = 'translations',
  Appearance = 'system_appearance_settings',
  Documents = 'documents',
  DocumentTypes = 'document_types',
  Objects = 'objects',
  Assets = 'assets',
  Thumbnails = 'thumbnails',
  TagsConfiguration = 'tags_configuration',
  PredefinedProperties = 'predefined_properties',
  WebsiteSettings = 'website_settings',
  Users = 'users',
  Notifications = 'notifications',
  SendNotifications = 'notifications_send',
  Emails = 'emails',
  Reports = 'reports',
  ReportsConfig = 'reports_config',
  RecycleBin = 'recyclebin',
  Redirects = 'redirects',
  ApplicationLogger = 'application_logging',
  PerspectiveEditor = 'studio_perspective_editor',
  WidgetEditor = 'studio_perspective_widget_editor',
  GDPRDataExtractor = 'gdpr_data_extractor'
}
```

### Basic Usage

```typescript
import { isAllowed, UserPermission } from '@pimcore/studio-ui-bundle/modules/auth'

export const SettingsMenu = (): React.JSX.Element => {
  const { t } = useTranslation()
  
  // Check if user can access users management
  const canManageUsers = isAllowed(UserPermission.Users)
  
  return (
    <Menu>
      {canManageUsers && (
        <MenuItem onClick={openUsersPage}>
          {t('settings.users')}
        </MenuItem>
      )}
    </Menu>
  )
}
```

### Pattern 1: Feature Access Control

```typescript
import { isAllowed, UserPermission } from '@pimcore/studio-ui-bundle/modules/auth'

export const AdminPanel = (): React.JSX.Element => {
  const canAccessUsers = isAllowed(UserPermission.Users)
  const canAccessTranslations = isAllowed(UserPermission.Translations)
  const canAccessReports = isAllowed(UserPermission.Reports)
  
  return (
    <div>
      <h1>Admin Panel</h1>
      
      {canAccessUsers && (
        <Card title="User Management">
          <Button onClick={openUsersPage}>Manage users</Button>
        </Card>
      )}
      
      {canAccessTranslations && (
        <Card title="Translations">
          <Button onClick={openTranslationsPage}>Manage translations</Button>
        </Card>
      )}
      
      {canAccessReports && (
        <Card title="Reports">
          <Button onClick={openReportsPage}>View reports</Button>
        </Card>
      )}
    </div>
  )
}
```

### Pattern 2: Conditional Navigation Items

```typescript
import { isAllowed, UserPermission } from '@pimcore/studio-ui-bundle/modules/auth'

export const useNavigationItems = () => {
  const { t } = useTranslation()
  
  const items = []
  
  // Add items based on user permissions
  if (isAllowed(UserPermission.Assets)) {
    items.push({
      key: 'assets',
      label: t('nav.assets'),
      icon: 'assets'
    })
  }
  
  if (isAllowed(UserPermission.Documents)) {
    items.push({
      key: 'documents',
      label: t('nav.documents'),
      icon: 'documents'
    })
  }
  
  if (isAllowed(UserPermission.Objects)) {
    items.push({
      key: 'objects',
      label: t('nav.objects'),
      icon: 'objects'
    })
  }
  
  return items
}
```

### Pattern 3: Editor Tab Visibility

```typescript
import { isAllowed, UserPermission } from '@pimcore/studio-ui-bundle/modules/auth'

export const editorTabs = [
  {
    key: 'properties',
    label: 'Properties',
    userPermission: UserPermission.PredefinedProperties,
    component: PropertiesTab
  },
  {
    key: 'notes',
    label: 'Notes & Events',
    userPermission: UserPermission.NotesAndEvents,
    component: NotesTab
  }
]

export const EditorTabs = (): React.JSX.Element => {
  // Filter tabs based on user permissions
  const visibleTabs = editorTabs.filter(tab => 
    !tab.userPermission || isAllowed(tab.userPermission)
  )
  
  return (
    <Tabs items={visibleTabs} />
  )
}
```

## Perspective Permissions

Check if features are visible/accessible in the current active perspective.

### `isAllowedInPerspective()` Function

```typescript
isAllowedInPerspective(permission: string): boolean
```

**Returns:** `true` if the permission is granted in the current perspective, `false` otherwise

**Permission Format:** Use dot notation like `'group.permission'` or `'feature.hidden'`

### Basic Usage

```typescript
import { isAllowedInPerspective } from '@pimcore/studio-ui-bundle/modules/perspectives'

export const SearchFeature = (): React.JSX.Element => {
  // Check if search is hidden in current perspective
  if (isAllowedInPerspective('search.hidden')) {
    return null // Don't render search
  }
  
  return <SearchComponent />
}
```

### Real-World Pattern: Navigation Item Visibility

```typescript
// File: app/base-layout/main-nav/main-nav.tsx
import { isAllowedInPerspective } from '@pimcore/studio-ui-bundle/modules/perspectives'

interface IMainNavItem {
  icon: string
  label: string
  perspectivePermissionHide?: string  // Dot notation permission key
  // ... other props
}

export const MainNav = (): React.JSX.Element => {
  const renderNavItem = (item: IMainNavItem): React.JSX.Element => {
    // Check if item should be hidden in current perspective
    const isHiddenInPerspective = 
      !isUndefined(item.perspectivePermissionHide) && 
      isAllowedInPerspective(item.perspectivePermissionHide)
    
    // Don't render if hidden
    if (isHiddenInPerspective) {
      return <></>
    }
    
    return (
      <MenuItem icon={item.icon}>
        {item.label}
      </MenuItem>
    )
  }
  
  // ... rest of component
}
```

### Pattern: Feature Toggles per Perspective

```typescript
import { isAllowedInPerspective } from '@pimcore/studio-ui-bundle/modules/perspectives'

export const EditorFeatures = (): React.JSX.Element => {
  // Check multiple perspective permissions
  const showAdvancedFeatures = !isAllowedInPerspective('editor.advanced.hidden')
  const showExportTools = !isAllowedInPerspective('editor.export.hidden')
  const showVersioning = !isAllowedInPerspective('editor.versions.hidden')
  
  return (
    <div>
      {showAdvancedFeatures && <AdvancedEditor />}
      {showExportTools && <ExportToolbar />}
      {showVersioning && <VersionHistory />}
    </div>
  )
}
```

## Combining Multiple Permission Checks

Often you need to check multiple permission types together.

### Pattern 1: Element + User Permissions

```typescript
import { checkElementPermission } from '@pimcore/studio-ui-bundle/modules/element'
import { isAllowed, UserPermission } from '@pimcore/studio-ui-bundle/modules/auth'

export const ElementActions = (): React.JSX.Element => {
  const { element } = useElement(id)
  
  // Check both element permission AND user permission
  const canAccessVersions = 
    checkElementPermission(element?.permissions, 'versions') &&
    isAllowed(UserPermission.NotesAndEvents)
  
  return (
    <>
      {canAccessVersions && (
        <Button onClick={openVersions}>
          View versions
        </Button>
      )}
    </>
  )
}
```

### Pattern 2: All Three Permission Types

```typescript
import { checkElementPermission } from '@pimcore/studio-ui-bundle/modules/element'
import { isAllowed, UserPermission } from '@pimcore/studio-ui-bundle/modules/auth'
import { isAllowedInPerspective } from '@pimcore/studio-ui-bundle/modules/perspectives'

export const AdvancedPropertiesTab = (): React.JSX.Element => {
  const { element } = useElement(id)
  
  // Check all three permission levels:
  // 1. Element permission - can edit properties
  // 2. User permission - has access to predefined properties feature
  // 3. Perspective permission - properties tab not hidden
  const canShowTab = 
    checkElementPermission(element?.permissions, 'properties') &&
    isAllowed(UserPermission.PredefinedProperties) &&
    !isAllowedInPerspective('element.properties.hidden')
  
  if (!canShowTab) {
    return null
  }
  
  return <PropertiesTabContent />
}
```

### Pattern 3: Permission-Based Action List

```typescript
import { checkElementPermission } from '@pimcore/studio-ui-bundle/modules/element'
import type { MenuProps } from 'antd'

export const useContextMenuItems = (element: Element) => {
  const { t } = useTranslation()
  
  const items: MenuProps['items'] = []
  
  // Add actions based on permissions
  if (checkElementPermission(element?.permissions, 'view')) {
    items.push({
      key: 'open',
      label: t('context-menu.open'),
      icon: <Icon value="open" />
    })
  }
  
  if (checkElementPermission(element?.permissions, 'rename')) {
    items.push({
      key: 'rename',
      label: t('context-menu.rename'),
      icon: <Icon value="edit" />
    })
  }
  
  if (checkElementPermission(element?.permissions, 'delete')) {
    items.push({
      type: 'divider'
    })
    items.push({
      key: 'delete',
      label: t('context-menu.delete'),
      icon: <Icon value="delete" />,
      danger: true
    })
  }
  
  return items
}
```

## Common Patterns

### Pattern 1: Permission Guard Hook

Create a reusable hook for permission checks:

```typescript
import { checkElementPermission, type ElementPermissionKeys } from '@pimcore/studio-ui-bundle/modules/element'

export const useElementPermissions = (element?: Element) => {
  const hasPermission = (permission: ElementPermissionKeys): boolean => {
    return checkElementPermission(element?.permissions, permission)
  }
  
  return {
    canPublish: hasPermission('publish'),
    canDelete: hasPermission('delete'),
    canRename: hasPermission('rename'),
    canView: hasPermission('view'),
    canEditProperties: hasPermission('properties'),
    hasPermission
  }
}

// Usage:
export const ElementToolbar = (): React.JSX.Element => {
  const { element } = useElement(id)
  const { canPublish, canDelete } = useElementPermissions(element)
  
  return (
    <Toolbar>
      {canPublish && <SaveButton />}
      {canDelete && <DeleteButton />}
    </Toolbar>
  )
}
```

### Pattern 2: Permission-Aware Button Component

```typescript
import { checkElementPermission, type ElementPermissionKeys } from '@pimcore/studio-ui-bundle/modules/element'
import { Button, type ButtonProps } from '@pimcore/studio-ui-bundle/components'

interface PermissionButtonProps extends ButtonProps {
  element?: Element
  requiredPermission: ElementPermissionKeys
  children: React.ReactNode
}

export const PermissionButton = ({ 
  element, 
  requiredPermission, 
  children,
  ...buttonProps 
}: PermissionButtonProps): React.JSX.Element => {
  const hasPermission = checkElementPermission(element?.permissions, requiredPermission)
  
  if (!hasPermission) {
    return null
  }
  
  return (
    <Button {...buttonProps}>
      {children}
    </Button>
  )
}

// Usage:
<PermissionButton
  element={asset}
  onClick={handleSave}
  requiredPermission="publish"
  type="primary"
>
  Save
</PermissionButton>
```

### Pattern 3: Centralized Permission Config

```typescript
// File: permissions/element-actions-config.ts
import type { ElementPermissionKeys } from '@pimcore/studio-ui-bundle/modules/element'

export interface ActionConfig {
  key: string
  label: string
  icon: string
  permission: ElementPermissionKeys
  danger?: boolean
}

export const ELEMENT_ACTIONS: ActionConfig[] = [
  {
    key: 'open',
    label: 'Open',
    icon: 'open',
    permission: 'view'
  },
  {
    key: 'rename',
    label: 'Rename',
    icon: 'edit',
    permission: 'rename'
  },
  {
    key: 'delete',
    label: 'Delete',
    icon: 'delete',
    permission: 'delete',
    danger: true
  }
]

// Usage:
import { ELEMENT_ACTIONS } from './permissions/element-actions-config'
import { checkElementPermission } from '@pimcore/studio-ui-bundle/modules/element'

export const ContextMenu = ({ element }: Props): React.JSX.Element => {
  const { t } = useTranslation()
  
  // Filter actions based on permissions
  const allowedActions = ELEMENT_ACTIONS.filter(action =>
    checkElementPermission(element?.permissions, action.permission)
  )
  
  const menuItems = allowedActions.map(action => ({
    key: action.key,
    label: t(`context-menu.${action.key}`),
    icon: <Icon value={action.icon} />,
    danger: action.danger
  }))
  
  return <Dropdown menu={{ items: menuItems }} />
}
```

## Common Mistakes

### ❌ Not Checking for Undefined Permissions

```typescript
// DON'T - May crash if permissions are undefined
const canPublish = element.permissions.publish

// DO - Use helper function that handles undefined
const canPublish = checkElementPermission(element?.permissions, 'publish')
```

### ❌ Hardcoding Permission Strings

```typescript
// DON'T - Typo-prone, not type-safe
if (element.permissions?.publsh) { // Typo!
  // ...
}

// DO - Use helper function with typed permission keys
if (checkElementPermission(element?.permissions, 'publish')) {
  // ...
}
```

### ❌ Client-Side Permission Bypass

```typescript
// DON'T - Only hiding UI, not enforcing on backend
<Button onClick={deleteElement}>Delete</Button>

// DO - Check permission before action AND on backend
{checkElementPermission(element?.permissions, 'delete') && (
  <Button onClick={deleteElement}>Delete</Button>
)}

// Backend endpoint MUST also validate permissions
```

### ❌ Forgetting Admin Check

```typescript
// DON'T - Manually checking admin status
const canAccess = user.isAdmin || user.permissions.includes('users')

// DO - Use isAllowed() which handles admin check automatically
const canAccess = isAllowed(UserPermission.Users)
```

### ❌ Using Wrong Permission Check Function

```typescript
// DON'T - Using element permission check for user permission
if (checkElementPermission(UserPermission.Assets)) { // Wrong function!
  // ...
}

// DO - Use correct function for each permission type
if (isAllowed(UserPermission.Assets)) { // User permission
  // ...
}

if (checkElementPermission(element?.permissions, 'publish')) { // Element permission
  // ...
}

if (isAllowedInPerspective('feature.hidden')) { // Perspective permission
  // ...
}
```

### ❌ Not Disabling Actions During Loading

```typescript
// DON'T - Allow clicking while loading
{canPublish && (
  <Button loading={isLoading} onClick={handleSave}>
    Save
  </Button>
)}

// DO - Disable button while loading
{canPublish && (
  <Button
    disabled={isLoading}
    loading={isLoading}
    onClick={handleSave}
  >
    Save
  </Button>
)}
```

### ❌ Forgetting Writeable Check for Config Features

```typescript
// DON'T - Not checking writeable state for config-based features
export const PerspectiveForm = (): React.JSX.Element => {
  const { perspective } = usePerspective(id)
  
  return (
    <Button htmlType="submit" type="primary">
      {t('save')}
    </Button>
  )
}

// DO - Always check writeable and show standard tooltip
export const PerspectiveForm = (): React.JSX.Element => {
  const { perspective } = usePerspective(id)
  const isWriteable = perspective.isWriteable
  
  return (
    <Tooltip title={isWriteable ? '' : t('config_not_writeable')}>
      <Button
        disabled={!isWriteable}
        htmlType="submit"
        type="primary"
      >
        {t('save')}
      </Button>
    </Tooltip>
  )
}
```

### ❌ Custom Writeable Error Messages

```typescript
// DON'T - Creating custom messages for writeable state
<Tooltip title={isWriteable ? '' : 'Cannot save this configuration'}>
  <Button disabled={!isWriteable}>Save</Button>
</Tooltip>

// DON'T - Using wrong translation key
<Tooltip title={isWriteable ? '' : t('config.readonly')}>
  <Button disabled={!isWriteable}>Save</Button>
</Tooltip>

// DO - ALWAYS use the standard translation key
<Tooltip title={isWriteable ? '' : t('config_not_writeable')}>
  <Button disabled={!isWriteable}>Save</Button>
</Tooltip>
```

### ❌ Confusing Writeable with Permissions

```typescript
// DON'T - Treating writeable as a user permission
if (isAllowed('writeable')) { // Wrong! This is not a user permission
  // ...
}

// DO - Check writeable state from the config object itself
const isWriteable = perspective.isWriteable
if (isWriteable) {
  // Allow saving
}

// DO - Check both writeable AND permissions if needed
const canSave = 
  checkElementPermission(element?.permissions, 'publish') && // Permission check
  element.isWriteable // Config writeable check
```

## Quick Reference

### Permission Check Functions

| Function | Use For | Returns |
|----------|---------|---------|
| `checkElementPermission(permissions, key)` | Element-specific actions | `true` if user has permission on element |
| `isAllowed(UserPermission.X)` | Global user permissions | `true` if user has permission (or is admin) |
| `isAllowedInPerspective('key')` | Perspective visibility | `true` if permission is set in perspective |

### Writeable State Check (Config Features)

| Property | Found In | Purpose |
|----------|----------|---------|
| `isWriteable` | Perspectives, Widgets | Indicates if config can be modified |
| `writeable` | Thumbnails, Appearance Settings | Indicates if config can be modified |
| **Tooltip** | `t('config_not_writeable')` | Standard tooltip for non-writeable configs |
| **Pattern** | `<Tooltip title={isWriteable ? '' : t('config_not_writeable')}>` | Wrap save/delete buttons |

### Common Element Permissions

| Permission | Use For |
|------------|---------|
| `'publish'` | Save/publish element |
| `'delete'` | Delete element |
| `'rename'` | Rename element |
| `'view'` | View/open element |
| `'properties'` | Edit properties |
| `'versions'` | View/restore versions |
| `'settings'` | Edit settings |

### Permission Check Decision Tree

```
Need to restrict save/delete buttons?
│
├─ Config-based feature (perspective, widget, thumbnail, settings)?
│  └─ Check: isWriteable or writeable property
│     └─ Pattern: <Tooltip title={isWriteable ? '' : t('config_not_writeable')}>
│
├─ Action on specific element (save, delete, rename)?
│  └─ Use: checkElementPermission(element?.permissions, 'permission')
│
├─ Access to global feature (users, reports, translations)?
│  └─ Use: isAllowed(UserPermission.X)
│
└─ Feature visibility in current perspective?
   └─ Use: isAllowedInPerspective('group.permission')
```

## Common Mistakes

### ❌ Mistake 1: Wrong Permission Check Type
```typescript
// ❌ WRONG - Using user permission for element
import { isAllowed, UserPermission } from '@pimcore/studio-ui-bundle/modules/auth'
if (isAllowed(UserPermission.ASSETS)) { /* edit asset */ }

// ✅ CORRECT - Use element permissions
import { checkElementPermission } from '@pimcore/studio-ui-bundle/modules/element'
if (checkElementPermission(asset.permissions, 'update')) { /* edit asset */ }
```

### ❌ Mistake 2: Not Checking isNil Before Permission Check
```typescript
// ❌ WRONG - Crashes if asset is undefined
if (checkElementPermission(asset.permissions, 'delete')) { ... }

// ✅ CORRECT - Check isNil first
import { isNil } from 'lodash'
if (!isNil(asset) && checkElementPermission(asset.permissions, 'delete')) { ... }
```

### ❌ Mistake 3: Confusing Perspective vs User Permissions
```typescript
// ❌ WRONG - Using user permission for perspective feature
import { isAllowed, UserPermission } from '@pimcore/studio-ui-bundle/modules/auth'
if (isAllowed(UserPermission.ASSETS)) { /* show nav item */ }

// ✅ CORRECT - Use perspective permission
import { isAllowedInPerspective } from '@pimcore/studio-ui-bundle/modules/perspectives'
if (isAllowedInPerspective('assets.explorer')) { /* show nav item */ }
```

### ❌ Mistake 4: Not Handling Writeable State for Config Features
```typescript
// ❌ WRONG - Using checkElementPermission for config
const asset = useAssetDraft()
if (checkElementPermission(asset.permissions, 'update')) { /* enable form */ }

// ✅ CORRECT - Check writeable state
import { isNil } from 'lodash'
const asset = useAssetDraft()
const canEdit = !isNil(asset) && (asset.writeable ?? asset.isWriteable ?? true)
return <FormKit disabled={!canEdit}>...</FormKit>
```

### ❌ Mistake 5: Forgetting to Hide UI When Permission Denied
```typescript
// ❌ WRONG - Showing disabled button
<Button disabled={!hasPermission}>Delete</Button>

// ✅ CORRECT - Hide button entirely
{hasPermission && <Button>Delete</Button>}
```

## Next Steps

- [**pimcore-studio-ui-typescript-best-practices**](../pimcore-studio-ui-typescript-best-practices/SKILL.md) - Critical type safety patterns (use with permission checks)
- [**pimcore-studio-ui-buttons**](../pimcore-studio-ui-buttons/SKILL.md) - Button components with permission-based rendering
- [**pimcore-studio-ui-modals**](../pimcore-studio-ui-modals/SKILL.md) - Confirmation modals for destructive actions
- [**pimcore-studio-ui-notifications-toasts**](../pimcore-studio-ui-notifications-toasts/SKILL.md) - Error messages for permission failures
- [**pimcore-studio-ui-navigation**](../pimcore-studio-ui-navigation/SKILL.md) - Navigation items with perspective permissions
