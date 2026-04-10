---
name: pimcore-studio-ui-navigation
description: Adding main navigation entries and perspective permissions in Pimcore Studio UI - frontend registration and backend permission management
metadata:
  audience: pimcore-developers
  focus: navigation-perspectives
---

## What This Skill Covers

How to add navigation entries to Pimcore Studio UI with perspective permissions:
- Registering main navigation items (frontend)
- perspectivePermission field and relation to perspectives
- Backend perspective permission registration (PHP)
- StudioContextPermissionsSubscriber pattern
- Translation keys for navigation and perspective editor
- Permission groups and naming conventions

## When to Use This Skill

Use this when:
- Adding new features that need main navigation entries
- Creating bundle navigation items
- Setting up perspective permissions for features
- Controlling feature visibility per perspective
- Adding menu items to the sidebar

## Writing Style for Navigation Labels

**Navigation items use Title Case** - each principal word is capitalized.

This applies to:
- Main navigation menu items
- Tab titles when menu items are opened
- Context menu labels
- Section headings
- Widget titles

### Title Case Examples

```typescript
// ✅ CORRECT - Title Case for navigation
mainNavRegistry.registerMainNavItem({
  path: 'Data Management/Custom Reports',  // "Custom Reports"
  label: 'custom-reports.title'
})

// Translation in studio.en.yaml:
// custom-reports.title: Custom Reports
// output-channels.title: Output Channels
// data-objects.title: Data Objects
// user-settings.title: User Settings
```

### Navigation vs Buttons

| Element Type | Case Style | Example |
|--------------|------------|---------|
| Navigation items | Title Case | "Custom Reports" |
| Menu items | Title Case | "Output Channels" |
| Tab titles | Title Case | "Data Objects" |
| Buttons | Sentence case | "Export CSV" |
| Actions | Sentence case | "Save draft" |

**Remember:** Navigation = Title Case, Buttons/Actions = Sentence case

For button and action labels, see [**pimcore-studio-ui-buttons**](../pimcore-studio-ui-buttons/SKILL.md) skill.

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## Navigation Registration (Frontend)

Navigation items are registered in your module's initialization file using the **MainNavRegistry**.

### Basic Pattern

```typescript
// File: your-bundle/assets/studio/js/src/modules/your-feature/index.ts
import { type AbstractModule } from '@pimcore/studio-ui-bundle'
import { container, serviceIds } from '@pimcore/studio-ui-bundle/app'
import { type MainNavRegistry } from '@pimcore/studio-ui-bundle/modules/app'

export const YourFeatureModule: AbstractModule = {
  onInit: (): void => {
    const mainNavRegistry = container.get<MainNavRegistry>(serviceIds.mainNavRegistry)

    mainNavRegistry.registerMainNavItem({
      path: 'ParentGroup/SubGroup/Your Feature',
      label: 'your-feature.navigation.title',
      order: 5,
      perspectivePermission: 'dataManagement.yourFeature',
      widgetConfig: {
        name: 'Your Feature',
        id: 'your-feature',
        component: 'your-feature',
        config: {
          translationKey: 'your-feature.navigation.title',
          icon: {
            type: 'name',
            value: 'your-icon'
          }
        }
      }
    })
  }
}
```

### Navigation Item Properties

```typescript
interface IMainNavItem {
  path: string                        // Hierarchical path 'Group/SubGroup/Item'
  label?: string                      // Translation key for display text
  order?: number                      // Display order (default: 1000)
  perspectivePermission?: string      // Permission key 'group.permission'
  widgetConfig?: WidgetManagerTabConfig  // Widget configuration for clickable items
  
  // Optional properties
  id?: string                         // Unique identifier
  icon?: string                       // Icon name
  groupIcon?: string                  // Group-level icon
  group?: string                      // Group name
  dividerBottom?: boolean             // Show divider after item
  children?: IMainNavItem[]           // Nested items
  permission?: string                 // User permission required
  perspectivePermissionHide?: string  // Permission to hide item
  className?: string                  // Custom CSS class
  hidden?: () => boolean              // Dynamic visibility function
}
```

### Real-World Example: Target Groups

```typescript
// File: personalization-bundle/.../target-groups/index.ts
export const TargetGroupsModule: AbstractModule = {
  onInit: (): void => {
    const mainNavRegistry = container.get<MainNavRegistry>(serviceIds.mainNavRegistry)

    mainNavRegistry.registerMainNavItem({
      path: 'ExperienceEcommerce/PersonalisationTargeting/Target Groups',
      label: 'personalization.target-groups',
      order: 10,
      permission: 'targeting',
      perspectivePermission: 'experienceEcommerce.personalizationTargetGroups',
      widgetConfig: {
        name: 'Target Groups',
        id: 'target-groups',
        component: 'target-groups-container',
        config: {
          translationKey: 'personalization.target-groups',
          icon: {
            type: 'name',
            value: 'target-group'
          }
        }
      }
    })
  }
}
```

## perspectivePermission Field

The `perspectivePermission` field controls **which perspectives can see** the navigation item.

### Format: Dot Notation

```
perspectivePermission: '{group}.{permissionKey}'
```

- **First part:** Group name (must match `ContextPermissionGroups` enum)
- **Second part:** Permission key (registered in PHP)

### Available Permission Groups

```typescript
// From: studio-backend-bundle/.../ContextPermissionGroups.php
enum ContextPermissionGroups: string
{
    case QUICK_ACCESS = 'quickAccess';
    case DATA_MANAGEMENT = 'dataManagement';
    case EXPERIENCE_ECOMMERCE = 'experienceEcommerce';
    case ASSET_MANAGEMENT = 'assetManagement';
    case TRANSLATIONS = 'translations';
    case REPORTING = 'reporting';
    case SYSTEM = 'system';
    case SEARCH = 'search';
}
```

### Permission Examples

```typescript
// Experience & E-commerce group
perspectivePermission: 'experienceEcommerce.personalizationTargetingRules'
perspectivePermission: 'experienceEcommerce.personalizationTargetGroups'
perspectivePermission: 'experienceEcommerce.portalEngine'
perspectivePermission: 'experienceEcommerce.emails'

// Data Management group
perspectivePermission: 'dataManagement.portalEngineCollections'
perspectivePermission: 'dataManagement.bookmarkLists'
perspectivePermission: 'dataManagement.tagConfiguration'

// Reporting group
perspectivePermission: 'reporting.dashboards'

// Quick Access group
perspectivePermission: 'quickAccess.open_asset'
perspectivePermission: 'quickAccess.open_document'

// System group
perspectivePermission: 'system.users'
perspectivePermission: 'system.roles'
```

### How It Works

1. **Frontend** checks `perspectivePermission` against active perspective
2. **Backend** provides list of enabled permissions per perspective
3. **Navigation** filters items based on permission check
4. **Perspective Editor** uses these permissions to configure perspectives

```typescript
// Permission check (happens automatically)
const isAllowedInPerspective = (permission: string): boolean => {
  const activePerspective = selectActivePerspective(store.getState())
  if (!activePerspective) return false
  
  // Walks nested object: activePerspective.contextPermissions.experienceEcommerce.personalizationTargetGroups
  return isPathTrue(activePerspective.contextPermissions, permission)
}
```

## Backend Permission Registration (PHP)

### 🚨 IMPORTANT: Where to Register Permissions

**The approach differs based on which bundle you're working in:**

#### studio-ui-bundle (Core)
If you're adding permissions for **studio-ui-bundle** features, register them **directly in studio-backend-bundle**:

**Primary Location (Most Permissions):**
- File: `studio-backend-bundle/src/Perspective/Service/ContextPermissionService.php`
- Most core perspective permissions are defined directly in this service class
- See: https://github.com/pimcore/studio-backend-bundle/blob/1.x/src/Perspective/Service/ContextPermissionService.php

**Additional Permissions via Subscribers:**
- Additional permissions can be registered via event subscribers in studio-backend-bundle
- Multiple context-specific subscribers add their permissions via `ContextPermissionsServiceInterface`

#### Custom Bundles (e.g., personalization-bundle, your-bundle)
If you're creating a **custom bundle** (first-party or third-party), register permissions in your **own bundle** using a StudioContextPermissionsSubscriber as shown below.

---

### For studio-ui-bundle: Add to ContextPermissionService

Most core studio-ui-bundle permissions are defined directly in the `ContextPermissionService`:

```php
<?php
// File: studio-backend-bundle/src/Perspective/Service/ContextPermissionService.php

namespace Pimcore\Bundle\StudioBackendBundle\Perspective\Service;

final readonly class ContextPermissionService implements ContextPermissionsServiceInterface
{
    // ... existing code ...
    
    private function getDefaultPermissions(): array
    {
        return [
            // Quick Access permissions
            new ContextPermissionData(
                'open_asset',
                ContextPermissionGroups::QUICK_ACCESS->value,
                true
            ),
            new ContextPermissionData(
                'open_document',
                ContextPermissionGroups::QUICK_ACCESS->value,
                true
            ),
            
            // Add your new studio-ui-bundle permission here
            new ContextPermissionData(
                'yourNewFeature',
                ContextPermissionGroups::DATA_MANAGEMENT->value,
                true
            ),
            
            // ... more permissions ...
        ];
    }
}
```

### For Custom Bundles: Create a StudioContextPermissionsSubscriber

Permissions for custom bundles must be registered using a **StudioContextPermissionsSubscriber** in your bundle.

#### Step 1: Create the Subscriber

```php
<?php
// File: your-bundle/src/EventSubscriber/StudioContextPermissionsSubscriber.php

namespace YourVendor\YourBundle\EventSubscriber;

use Pimcore\Bundle\StudioBackendBundle\Perspective\Model\ContextPermissionData;
use Pimcore\Bundle\StudioBackendBundle\Perspective\Service\ContextPermissionsServiceInterface;
use Pimcore\Bundle\StudioBackendBundle\Perspective\Util\Constant\ContextPermissionGroups;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\KernelEvents;

final readonly class StudioContextPermissionsSubscriber implements EventSubscriberInterface
{
    public function __construct(
        private ContextPermissionsServiceInterface $permissionsService,
    ) {
    }

    public static function getSubscribedEvents(): array
    {
        return [
            KernelEvents::CONTROLLER => 'addContextPermissions',
        ];
    }

    public function addContextPermissions(): void
    {
        // Register your permissions here
        $this->permissionsService->add(
            new ContextPermissionData(
                'yourFeature',                                    // Permission key
                ContextPermissionGroups::DATA_MANAGEMENT->value,  // Group
                true                                               // Default enabled
            )
        );
    }
}
```

#### Step 2: Register as Service (Auto-configured)

Most Symfony bundles auto-configure event subscribers. If not, add to `services.yaml`:

```yaml
services:
  YourVendor\YourBundle\EventSubscriber\StudioContextPermissionsSubscriber:
    tags:
      - { name: kernel.event_subscriber }
```

### ContextPermissionData Parameters

```php
new ContextPermissionData(
    string $key,              // Permission key (e.g., 'yourFeature')
    string $group,            // Group from ContextPermissionGroups enum
    bool $defaultValue = true // Default enabled state in new perspectives
)
```

### Real-World Example: Personalization Bundle

```php
<?php
// File: personalization-bundle/src/EventSubscriber/StudioContextPermissionsSubscriber.php

namespace Pimcore\Bundle\PersonalizationBundle\EventSubscriber;

use Pimcore\Bundle\StudioBackendBundle\Perspective\Model\ContextPermissionData;
use Pimcore\Bundle\StudioBackendBundle\Perspective\Service\ContextPermissionsServiceInterface;
use Pimcore\Bundle\StudioBackendBundle\Perspective\Util\Constant\ContextPermissionGroups;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\KernelEvents;

final readonly class StudioContextPermissionsSubscriber implements EventSubscriberInterface
{
    public function __construct(
        private ContextPermissionsServiceInterface $permissionsService,
    ) {
    }

    public static function getSubscribedEvents(): array
    {
        return [
            KernelEvents::CONTROLLER => 'addContextPermissions',
        ];
    }

    public function addContextPermissions(): void
    {
        // Register targeting rules permission
        $this->permissionsService->add(
            new ContextPermissionData(
                'personalizationTargetingRules',
                ContextPermissionGroups::EXPERIENCE_ECOMMERCE->value
            )
        );
        
        // Register target groups permission
        $this->permissionsService->add(
            new ContextPermissionData(
                'personalizationTargetGroups',
                ContextPermissionGroups::EXPERIENCE_ECOMMERCE->value
            )
        );
    }
}
```

## Translation Keys

You need **two sets** of translation keys:

### 1. Navigation Item Label

Used in the main navigation menu.

```yaml
# File: your-bundle/translations/studio.en.yaml
your-feature:
  navigation:
    title: Your Feature Name
```

### 2. Perspective Editor Label

Used in the Perspective Editor for enabling/disabling the feature.

```yaml
# Pattern: perspective-editor.form.main-nav-permission.{group}.{permissionKey}
perspective-editor:
  form:
    main-nav-permission:
      dataManagement:
        yourFeature: Data Management > Your Feature Name
```

### Real-World Example: Target Groups

```yaml
# Navigation label
personalization:
  target-groups: Target Groups

# Perspective editor label
perspective-editor:
  form:
    main-nav-permission:
      experienceEcommerce:
        personalizationTargetGroups: Personalisation / Targeting > Target Groups
```

### Group Category Labels

If adding a new group category label:

```yaml
perspective-editor:
  form:
    main-nav-permission:
      category:
        experienceEcommerce: Experience & E-commerce
        dataManagement: Data Management
        reporting: Reporting
```

## Complete Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│ 1. FRONTEND: Register Navigation Item                        │
│                                                               │
│   mainNavRegistry.registerMainNavItem({                      │
│     path: 'Group/SubGroup/Item',                            │
│     perspectivePermission: 'group.permissionKey'            │
│   })                                                          │
└────────────────────────────┬──────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. BACKEND: Register Permission (PHP)                        │
│                                                               │
│   StudioContextPermissionsSubscriber::addContextPermissions()│
│     → permissionsService->add(                               │
│         new ContextPermissionData('permissionKey', 'group')  │
│       )                                                       │
└────────────────────────────┬──────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. TRANSLATIONS: Add Translation Keys                        │
│                                                               │
│   - your-feature.navigation.title                            │
│   - perspective-editor.form.main-nav-permission.group.key    │
└────────────────────────────┬──────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. PERSPECTIVE CONFIGURATION                                  │
│                                                               │
│   User enables/disables in Perspective Editor                │
│   Settings stored per perspective                            │
└────────────────────────────┬──────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. RUNTIME: Navigation Filtering                             │
│                                                               │
│   - Frontend checks perspectivePermission                    │
│   - Only shows nav items with enabled permissions            │
└─────────────────────────────────────────────────────────────┘
```

## Testing Your Navigation Item

### 1. Verify Registration

Check browser console for your module initialization:
```javascript
console.log('YourFeatureModule initialized')
```

### 2. Check Perspective Editor

1. Navigate to **System > Perspective Editor**
2. Select or create a perspective
3. Expand the appropriate group (e.g., "Experience & E-commerce")
4. Find your permission checkbox
5. Verify translation label is correct

### 3. Toggle Perspective Permission

1. Enable your permission in the perspective
2. Save perspective
3. Reload Studio
4. Verify navigation item appears
5. Disable permission and verify item disappears

### 4. Check Translation

1. Change language in user settings
2. Verify navigation label updates correctly
3. Check perspective editor label in different language

## Common Patterns

### Pattern 1: Simple Feature Navigation

```typescript
// Single navigation item under existing group
mainNavRegistry.registerMainNavItem({
  path: 'DataManagement/Your Feature',
  label: 'your-feature.title',
  perspectivePermission: 'dataManagement.yourFeature',
  widgetConfig: {
    name: 'Your Feature',
    id: 'your-feature',
    component: 'your-feature',
    config: {
      translationKey: 'your-feature.title',
      icon: { type: 'name', value: 'your-icon' }
    }
  }
})
```

### Pattern 2: Multiple Related Items

```typescript
// Register multiple items in same group
mainNavRegistry.registerMainNavItem({
  path: 'ExperienceEcommerce/YourBundle/Feature 1',
  label: 'your-bundle.feature1',
  order: 10,
  perspectivePermission: 'experienceEcommerce.feature1',
  widgetConfig: { /* ... */ }
})

mainNavRegistry.registerMainNavItem({
  path: 'ExperienceEcommerce/YourBundle/Feature 2',
  label: 'your-bundle.feature2',
  order: 20,
  perspectivePermission: 'experienceEcommerce.feature2',
  widgetConfig: { /* ... */ }
})
```

### Pattern 3: With User Permission Check

```typescript
// Combine perspectivePermission with user permission
mainNavRegistry.registerMainNavItem({
  path: 'System/Your Admin Feature',
  label: 'your-feature.admin',
  permission: 'admin',  // Requires admin user permission
  perspectivePermission: 'system.yourAdminFeature',  // AND perspective permission
  widgetConfig: { /* ... */ }
})
```

## Common Mistakes

### ❌ Missing Backend Registration

```typescript
// DON'T - Only register frontend
mainNavRegistry.registerMainNavItem({
  perspectivePermission: 'dataManagement.yourFeature'  // Won't work!
})

// DO - Register backend first
// Then frontend will work
```

### ❌ Wrong Group Name

```php
// DON'T - Typo or wrong group
new ContextPermissionData(
    'yourFeature',
    'dataManagment'  // Wrong! Typo
)

// DO - Use enum
new ContextPermissionData(
    'yourFeature',
    ContextPermissionGroups::DATA_MANAGEMENT->value  // Correct
)
```

### ❌ Mismatched Permission Keys

```typescript
// Frontend
perspectivePermission: 'dataManagement.yourFeature'  // 'yourFeature'

// Backend  
new ContextPermissionData('your-feature', ...)  // 'your-feature' - MISMATCH!

// DO - Keep them identical
// Frontend: 'dataManagement.yourFeature'
// Backend: 'yourFeature'
```

### ❌ Missing Translation Keys

```yaml
# DON'T - Forget perspective editor translation
your-feature:
  navigation:
    title: Your Feature

# DO - Add both
your-feature:
  navigation:
    title: Your Feature

perspective-editor:
  form:
    main-nav-permission:
      dataManagement:
        yourFeature: Data Management > Your Feature
```

## Quick Reference Checklist

To add a navigation item with perspective permissions:

### Frontend (TypeScript):
- [ ] Create module with `onInit` function
- [ ] Get `MainNavRegistry` from container
- [ ] Register item with `perspectivePermission: 'group.key'`
- [ ] Add widget configuration with icon and translation key
- [ ] Add navigation translation: `your-feature.navigation.title`

### Backend (PHP):
- [ ] Create `StudioContextPermissionsSubscriber` class
- [ ] Implement `EventSubscriberInterface`
- [ ] Subscribe to `KernelEvents::CONTROLLER`
- [ ] Call `$permissionsService->add()` in `addContextPermissions()`
- [ ] Use appropriate `ContextPermissionGroups` enum value
- [ ] Ensure service is registered (usually auto-configured)

### Translations:
- [ ] Add navigation label: `your-feature.navigation.title`
- [ ] Add perspective editor label: `perspective-editor.form.main-nav-permission.{group}.{key}`
- [ ] Add labels for all supported languages

### Testing:
- [ ] Navigate to System > Perspective Editor
- [ ] Verify permission appears in correct category
- [ ] Toggle permission on/off
- [ ] Verify navigation item appears/disappears
- [ ] Check translation labels

## Next Steps

- [**pimcore-studio-ui-i18n**](../pimcore-studio-ui-i18n/SKILL.md) - Translation system for labels
- [**pimcore-studio-ui-layout-components**](../pimcore-studio-ui-layout-components/SKILL.md) - Building the widget content
- [**pimcore-studio-ui-react-components**](../pimcore-studio-ui-react-components/SKILL.md) - Creating widget components
- [**pimcore-studio-ui-bundle-structure**](../pimcore-studio-ui-bundle-structure/SKILL.md) - Understanding bundle organization
