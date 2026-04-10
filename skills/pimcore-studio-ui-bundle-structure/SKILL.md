---
name: pimcore-studio-ui-bundle-structure
description: Organization and structure of Pimcore Studio bundles - modules, components, file layout
metadata:
  audience: pimcore-developers
  focus: architecture
---

## 🚨 CRITICAL: Import Paths in Bundles

**This skill covers BUNDLE structure (outside `studio-ui-bundle`)**

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).** All bundle imports use `@pimcore/studio-ui-bundle/*`.

---

## What This Skill Covers

Understanding the structure and organization of Pimcore Studio bundles:
- Directory layout and file organization
- Module structure and naming conventions
- Component hierarchy
- API file organization
- Translation file structure
- Build output location

## When to Use This Skill

Use this when:
- Creating a new module in a bundle
- Understanding where files should be placed
- Navigating an unfamiliar bundle
- Setting up a new Pimcore Studio bundle
- Understanding the relationship between frontend and backend code

## Bundle Directory Structure

```
{bundle-name}/                           # e.g., personalization-bundle
│
├── assets/studio/                       # Frontend code (TypeScript/React)
│   ├── js/
│   │   ├── src/
│   │   │   ├── modules/                # All Studio modules
│   │   │   │   ├── {module-name}/     # e.g., targeting-rules, personalization
│   │   │   │   │   ├── components/    # React components
│   │   │   │   │   │   ├── editor/
│   │   │   │   │   │   ├── detail/
│   │   │   │   │   │   └── list/
│   │   │   │   │   ├── hooks/         # Custom React hooks
│   │   │   │   │   ├── api/           # API utilities (if complex)
│   │   │   │   │   └── {module-name}-api.tsx  # RTK Query endpoints
│   │   │   │   └── index.tsx          # Re-exports all modules
│   │   │   └── index.tsx               # Main entry point
│   │   └── tsconfig.json
│   ├── package.json                     # Frontend dependencies
│   └── webpack.config.js                # Build configuration
│
├── public/studio/                       # Build output (committed to Git)
│   └── build/
│       └── {hash}/                      # Unique build hash
│           ├── remoteEntry.js           # Module federation entry
│           └── *.js                     # Compiled bundles
│
├── src/                                 # PHP backend code
│   ├── Controller/                      # API controllers
│   │   └── Studio/
│   │       └── {ModuleName}Controller.php
│   ├── EventSubscriber/                 # Event subscribers
│   ├── Service/                         # Business logic
│   └── DependencyInjection/
│
├── translations/                        # Translation files
│   ├── studio.en.yaml                  # English (Studio UI)
│   └── admin.en.yaml                   # Legacy admin (if any)
│
├── config/                              # Configuration
│   └── services.yaml
│
└── composer.json                        # PHP dependencies
```

## Module Structure

### Typical Module Organization

```
modules/{module-name}/
├── components/                          # React components
│   ├── editor/                         # Edit/create views
│   │   ├── {module}-editor.tsx
│   │   └── tabs/                       # If editor has tabs
│   ├── detail/                         # Detail/view components
│   │   └── {module}-detail.tsx
│   ├── list/                           # List/table views
│   │   └── {module}-list.tsx
│   └── shared/                         # Shared components
│
├── hooks/                               # Custom hooks
│   ├── use-{module}-helper.tsx
│   └── use-{module}-form.tsx
│
└── {module}-api.tsx                     # RTK Query API definition
```

### Real Example: Targeting Rules Module

```
modules/targeting-rules/
├── components/
│   ├── editor/
│   │   └── targeting-rule-editor.tsx   # Main editor component
│   ├── list/
│   │   └── targeting-rules-list.tsx    # List view
│   └── shared/
│       └── rule-status-badge.tsx       # Shared UI component
│
├── hooks/
│   └── use-targeting-rules-helper.tsx  # Helper hook
│
└── targeting-rules-api.tsx             # API endpoints
```

## API File Pattern

### Standard API File Structure

```typescript
// modules/{module-name}/{module-name}-api.tsx

import { api as baseApi } from '@Pimcore/modules/app/app-api'

// Type definitions
export interface EntityDetail {
  id: number
  settings: {
    name: string
    active: boolean
    // ... other fields
  }
}

export interface EntityListItem {
  id: number
  name: string
  // ... abbreviated fields
}

// API endpoints
export const api = baseApi.injectEndpoints({
  endpoints: (build) => ({
    // List endpoint
    getEntityList: build.query<EntityListItem[], void>({
      query: () => ({
        url: '/pimcore-bundle-{bundle}/{entity}/list',
      }),
    }),
    
    // Detail endpoint
    getEntityById: build.query<EntityDetail, { id: number }>({
      query: ({ id }) => ({
        url: `/pimcore-bundle-{bundle}/{entity}/${id}`,
      }),
    }),
    
    // Update mutation
    updateEntity: build.mutation<void, { id: number; body: any }>({
      query: ({ id, body }) => ({
        url: `/pimcore-bundle-{bundle}/{entity}/${id}`,
        method: 'PUT',
        body,
      }),
    }),
    
    // Delete mutation
    deleteEntity: build.mutation<void, { id: number }>({
      query: ({ id }) => ({
        url: `/pimcore-bundle-{bundle}/{entity}/${id}`,
        method: 'DELETE',
      }),
    }),
  }),
})

// Export hooks
export const {
  useGetEntityListQuery,
  useGetEntityByIdQuery,
  useUpdateEntityMutation,
  useDeleteEntityMutation,
} = api
```

### Real Example: Targeting Rules API

```typescript
// modules/targeting-rules/targeting-rules-api.tsx

export const api = baseApi.injectEndpoints({
  endpoints: (build) => ({
    bundlePersonalizationTargetingRuleGetById: build.query<
      TargetingRuleDetail,
      { ruleId: number }
    >({
      query: ({ ruleId }) => ({
        url: `/pimcore-bundle-personalization/targeting-rule/${ruleId}`,
      }),
    }),
    
    bundlePersonalizationTargetingRuleUpdate: build.mutation<
      void,
      { ruleId: number; body: UpdateBody }
    >({
      query: ({ ruleId, body }) => ({
        url: `/pimcore-bundle-personalization/targeting-rule/${ruleId}`,
        method: 'PUT',
        body,
      }),
    }),
  }),
})

export const {
  useBundlePersonalizationTargetingRuleGetByIdQuery,
  useBundlePersonalizationTargetingRuleUpdateMutation,
} = api
```

## Component Naming Conventions

### File Naming
- **Kebab case**: `targeting-rule-editor.tsx`
- **Component export**: PascalCase `TargetingRuleEditor`

### Component Types

```typescript
// Editor component
export const TargetingRuleEditor: FC<Props> = ({ rule }) => {
  // Edit existing or create new
}

// Detail component
export const TargetingRuleDetail: FC<Props> = ({ ruleId }) => {
  // View-only display
}

// List component
export const TargetingRulesList: FC = () => {
  // Table/grid of items
}
```

## Translation Structure

### File Location
```
translations/studio.en.yaml
```

### Organization Pattern

```yaml
# Module name (top level)
personalization:
  
  # Entity/feature (second level)
  targeting-rules:
    
    # Context (third level)
    list:
      title: 'Targeting Rules'
      empty: 'No targeting rules found'
      
    editor:
      title: 'Edit Targeting Rule'
      save: 'Save'
      cancel: 'Cancel'
      
    # Actions (third level)
    create:
      success: 'Targeting rule created'
      error: 'Failed to create targeting rule'
      
    update:
      success: 'Saved successfully'
      error: 'Failed to save targeting rule'
      
    delete:
      success: 'Targeting rule deleted'
      error: 'Failed to delete targeting rule'
      confirm: 'Are you sure you want to delete this rule?'
  
  # Another entity
  target-groups:
    list:
      title: 'Target Groups'
    update:
      success: 'Saved successfully'
```

## Backend Structure (PHP)

### Controller Pattern

```php
// src/Controller/Studio/{Module}Controller.php

namespace Pimcore\Bundle\{BundleName}\Controller\Studio;

use Pimcore\Bundle\StudioBackendBundle\Controller\AbstractApiController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Annotation\Route;

#[Route('/pimcore-bundle-{bundle}/{entity}', name: 'pimcore_bundle_{bundle}_{entity}_')]
class EntityController extends AbstractApiController
{
    #[Route('/{id}', name: 'get', methods: ['GET'])]
    public function getById(int $id): JsonResponse
    {
        // Implementation
    }
    
    #[Route('/{id}', name: 'update', methods: ['PUT'])]
    public function update(int $id): JsonResponse
    {
        // Implementation
    }
}
```

### Service Pattern

```php
// src/Service/{Module}Service.php

namespace Pimcore\Bundle\{BundleName}\Service;

class EntityService
{
    public function getById(int $id): array
    {
        // Business logic
    }
    
    public function update(int $id, array $data): void
    {
        // Business logic
    }
}
```

## Module Registration

### Main Entry Point

```typescript
// assets/studio/js/src/index.tsx

export * from './modules/targeting-rules/targeting-rules-api'
export * from './modules/personalization/personalization-api'
// Export other modules
```

### Module Index

```typescript
// assets/studio/js/src/modules/index.tsx

export * from './targeting-rules/targeting-rules-api'
export * from './personalization/personalization-api'
```

## Build Output Structure

### Output Location
```
public/studio/build/{hash}/
```

### Key Files
- **`remoteEntry.js`**: Module federation entry point (loaded by Studio)
- **`*.js`**: Compiled JavaScript bundles
- **`*.js.map`**: Source maps for debugging

### Important Notes
- Build output **is committed to Git** in Pimcore bundles
- Hash changes on each build
- Old hashes should be removed (one active build at a time)

## Finding Files

### To Find a Component
1. Know the module name (e.g., `targeting-rules`)
2. Navigate to `assets/studio/js/src/modules/{module}/components/`
3. Look in subdirectories: `editor/`, `list/`, `detail/`

### To Find an API Endpoint
1. Look in `assets/studio/js/src/modules/{module}/{module}-api.tsx`
2. Search for the endpoint name or URL pattern

### To Find Translations
1. All in `translations/studio.en.yaml`
2. Search for the module name (e.g., `targeting-rules`)

### To Find Backend Controller
1. Navigate to `src/Controller/Studio/`
2. Look for `{Module}Controller.php`
3. Match URL pattern to frontend API calls

## Common Patterns

### Module with Tabs

```
modules/data-importer/
├── components/
│   ├── editor/
│   │   ├── data-importer-editor.tsx
│   │   └── tabs/
│   │       ├── general-tab.tsx
│   │       ├── mapping-tab.tsx
│   │       └── schedule-tab.tsx
│   └── list/
│       └── data-importer-list.tsx
```

### Module with Complex Forms

```
modules/workflow/
├── components/
│   ├── editor/
│   │   ├── workflow-editor.tsx
│   │   └── forms/
│   │       ├── basic-info-form.tsx
│   │       ├── transitions-form.tsx
│   │       └── places-form.tsx
```

### Module with Nested Views

```
modules/reports/
├── components/
│   ├── dashboard/
│   │   └── reports-dashboard.tsx
│   ├── viewer/
│   │   └── report-viewer.tsx
│   └── builder/
│       └── report-builder.tsx
```

## Key Principles

1. **Modules are self-contained** - Each module has its own components, hooks, and API
2. **API files use RTK Query** - Inject endpoints into base API
3. **Components use functional style** - React.FC with TypeScript
4. **Translations are centralized** - All in `studio.en.yaml`
5. **Build output is committed** - Unlike typical JS projects
6. **Backend follows Studio patterns** - Extend AbstractApiController
7. **URL patterns are consistent** - `/pimcore-bundle-{name}/{entity}/{id}`

## When Creating a New Module

1. Create directory: `assets/studio/js/src/modules/{module-name}/`
2. Create API file: `{module-name}-api.tsx`
3. Create components directory: `components/`
4. Add translations: `translations/studio.en.yaml`
5. Create backend controller: `src/Controller/Studio/{Module}Controller.php`
6. Export from module index: `modules/index.tsx`
7. **Validation/Build**: Ask user if dev server running
   - If running: `npm run check-types && npm run lint-fix`
   - If not: `npm run dev` (bundles) or `npm run dev-app` (SDK)

## Troubleshooting

### Can't find a component
- Check module name matches (e.g., `targeting-rules` not `targetingRules`)
- Look in subdirectories: `editor/`, `list/`, `detail/`
- Check `index.tsx` files for re-exports

### API endpoint not found
- Check API file: `modules/{module}/{module}-api.tsx`
- Verify URL pattern matches backend route
- Check if endpoint is injected correctly

### Translation not working
- Verify key exists in `translations/studio.en.yaml`
- Check key pattern: `module.entity.action.result`
- Ensure no typos in translation key

### Changes not appearing

**First: Check dev server status**

**If dev server IS running:**
- Changes should appear automatically via HMR
- Check browser console for HMR errors
- Try hard refresh (Ctrl+Shift+R)
- If structure changed, user may need to restart dev servers

**If dev server NOT running:**
- Run appropriate build command (after asking user):
  - SDK: `npm run dev-app`
  - Bundle: `npm run dev`
- Verify build succeeded (check output for errors)
- Verify files in `public/studio/build/{hash}/`
- Clear browser cache
- Commit build output to Git

### Type errors

**Always safe to run:**
```bash
npm run check-types
```

**Fix common issues:**
- Missing imports: Check CRITICAL-IMPORT-PATHS.md
- Type mismatches: Use TypeScript best practices (isNil, ??, etc.)
- Any types: Replace with specific types or unknown + guards

## Development Workflow

### 🚨 CRITICAL: Understanding Dev Servers vs Builds

**AI/Assistants: NEVER start or stop dev servers! They are managed by the user.**

#### Dev Server vs Build Mode

The user operates in ONE of two modes:

**Mode 1: Dev Server Running (Development)**
- User manually starts dev servers
- Changes reflect immediately via HMR
- **DO NOT run builds** - this will destroy the dev server
- ✅ Can run: `npm run check-types`, `npm run lint-fix`
- ❌ Cannot run: `npm run build`, `npm run dev`, `npm run dev-app`

**Mode 2: Build Mode (Production/Testing)**
- Dev servers are NOT running
- Changes require explicit builds
- ✅ Can run: builds, type checks, linting
- After build, commit changes to Git

#### Before Any Build Command

**ALWAYS ask the user first:**

```
Is your dev server currently running?
- If YES → Run only npm run check-types and npm run lint-fix
- If NO → Proceed with build commands
```

### Build Commands Reference

#### Studio UI Bundle (Core SDK)

**Location**: `vendor/pimcore/studio-ui-bundle/assets/studio/`

```bash
# Development build (when dev server NOT running)
npm run dev-app

# Type checking (safe to run anytime)
npm run check-types

# Linting with auto-fix (safe to run anytime)
npm run lint-fix

# Production build
npm run build
```

**When to build studio-ui-bundle:**
- Making changes to core SDK components
- Adding new SDK features that bundles will use
- Modifying shared utilities or hooks

#### Custom Bundle

**Location**: `{your-bundle}/assets/studio/`

```bash
# Development build (when dev server NOT running)
npm run dev

# Type checking (safe to run anytime)
npm run check-types

# Linting with auto-fix (safe to run anytime)
npm run lint-fix

# Production build
npm run build
```

### Bundle Development Workflow

When developing a bundle that uses SDK changes:

**Step 1: Check Dev Server Status**
```
AI: "Is your dev server currently running?"
```

**Step 2a: If Dev Server Running**
```bash
# Only validate, don't build
cd {your-bundle}/assets/studio
npm run check-types
npm run lint-fix
```

**Step 2b: If Dev Server NOT Running**
```bash
# Build SDK changes first
cd vendor/pimcore/studio-ui-bundle/assets/studio
npm run dev-app

# Then build your bundle
cd ../../../../{your-bundle}/assets/studio
npm run dev
```

### Dev Server Information (For User Reference)

**Users start dev servers manually in this order:**

**Terminal 1: SDK Server**
```bash
cd vendor/pimcore/studio-ui-bundle/assets/studio
npm run dev-server-sdk  # Runs on port 3030
```

**Terminal 2: Bundle Server**
```bash
cd {your-bundle}/assets/studio
npm run dev-server  # Runs on port 3031
```

**Why this order:**
1. Bundle server depends on SDK server
2. Module Federation requires SDK remote entries
3. Starting in wrong order causes module loading errors

### Development Ports

- **SDK Server**: http://localhost:3030
- **Bundle Dev Server**: http://localhost:3031
- **Backend API**: http://localhost (your Pimcore installation)

### Hot Module Replacement (HMR)

When dev servers are running:
- ✅ Component changes reload instantly
- ✅ Style changes apply without refresh
- ✅ React state preserved during HMR
- ⚠️ API/type changes may require server restart

### Validation Commands (Always Safe)

These can run **regardless of dev server status:**

```bash
# Type checking
npm run check-types

# Linting with auto-fix
npm run lint-fix

# Both together
npm run check-types && npm run lint-fix
```

### When Changes Don't Appear

**If dev server is running:**
- Check browser console for HMR errors
- Try hard refresh (Ctrl+Shift+R)
- Restart dev servers if structure changed

**If dev server is NOT running:**
- Run appropriate build command
- Verify build succeeded
- Check `public/studio/build/{hash}/` for output
- Clear browser cache

### 🚨 WARNING: Never Run Builds With Dev Server Active

Running builds while dev server is active will:
- ❌ Destroy the running dev server
- ❌ Break hot module replacement
- ❌ Force user to restart dev servers
- ❌ Lose current development state

**Always ask first!**

## Next Steps

- [**pimcore-studio-ui-react-components**](../pimcore-studio-ui-react-components/SKILL.md) - Component structure and patterns
- [**pimcore-studio-ui-using-sdk-in-bundles**](../pimcore-studio-ui-using-sdk-in-bundles/SKILL.md) - Using SDK components in bundles
- [**pimcore-studio-ui-rtk-query-fundamentals**](../pimcore-studio-ui-rtk-query-fundamentals/SKILL.md) - API integration patterns
- [**pimcore-studio-ui-navigation**](../pimcore-studio-ui-navigation/SKILL.md) - Adding navigation items
