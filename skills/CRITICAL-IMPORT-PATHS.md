# CRITICAL: Import Paths - Core vs Bundles

**THIS IS THE MOST FUNDAMENTAL RULE IN PIMCORE STUDIO DEVELOPMENT**

## The Rule

### Inside studio-ui-bundle (Core Development)

Core has three valid import styles:

```typescript
// ✅ CORRECT - Use @sdk/* alias (for SDK-exported modules)
import { container } from '@sdk'
import { serviceIds } from '@sdk/app'
import { FormKit, Form, Input } from '@sdk/components'
import { useAssetGetByIdQuery } from '@sdk/api/asset'
import { trackError, ApiError } from '@sdk/modules/app'

// ✅ CORRECT - Use @Pimcore/* alias (for internal files NOT exported via SDK)
import { SomeInternalHelper } from '@Pimcore/modules/some-internal/helper'
import { InternalComponent } from '@Pimcore/components/internal-component/internal-component'

// ✅ ALSO CORRECT - Use relative paths
import { container } from '../../../container'
import { FormKit } from '../../components/form/form-kit'
```

**When to use `@sdk/*` vs `@Pimcore/*` in core:**
- `@sdk/*` — for anything that is exported via the SDK public API
- `@Pimcore/*` — for internal files that are NOT part of the SDK exports (internal components, helpers, etc.)
- Both aliases only work inside `studio-ui-bundle` — never use either in external bundles

### Outside studio-ui-bundle (Bundle/Plugin Development)
```typescript
// ✅ CORRECT - Use @pimcore/studio-ui-bundle/*
import { container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import { FormKit, Form, Input } from '@pimcore/studio-ui-bundle/components'
import { useAssetGetByIdQuery } from '@pimcore/studio-ui-bundle/api/asset'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
```

### NEVER
```typescript
// ❌ WRONG - Never import from antd directly
import { Form, Input } from 'antd'

// ❌ WRONG - Don't use @sdk or @Pimcore in a bundle
import { container } from '@sdk' // in a bundle — won't resolve
import { SomeHelper } from '@Pimcore/modules/helper' // in a bundle — won't resolve

// ❌ WRONG - Don't use @pimcore/studio-ui-bundle in core
import { FormKit } from '@pimcore/studio-ui-bundle/components' // inside studio-ui-bundle

// ❌ WRONG - Don't mix @sdk and @pimcore/studio-ui-bundle in the same file
import { container } from '@sdk'
import { FormKit } from '@pimcore/studio-ui-bundle/components'
```

## Why This Matters

1. **@sdk/* is an alias that only exists in core** - It maps to the sdk folder during core development
2. **@pimcore/studio-ui-bundle/* is the published package** - Available to external bundles after build
3. **They point to the same code** but are different namespaces depending on context
4. **Mixing them breaks the build** and causes import resolution errors

## How to Know Which Context You're In

### You're in CORE if:
- Working in: `/studio-ui-bundle/assets/js/src/`
- File path contains: `studio-ui-bundle`
- Use: `@sdk/*` (SDK-exported), `@Pimcore/*` (internal), or relative paths

### You're in a BUNDLE if:
- Working in: `/your-bundle-name/assets/studio/`
- File path contains: any bundle except `studio-ui-bundle`
- Use: `@pimcore/studio-ui-bundle/*`

## Quick Reference Table

| Context | Imports | Example Path |
|---------|---------|--------------|
| Core (SDK-exported) | `@sdk/*` | `/studio-ui-bundle/assets/js/src/core/` |
| Core (internal) | `@Pimcore/*` | `/studio-ui-bundle/assets/js/src/core/` |
| Bundle | `@pimcore/studio-ui-bundle/*` | `/personalization-bundle/assets/studio/js/` |

## Import Checklist

Before writing any import statement, ask:

1. ✅ Am I inside `studio-ui-bundle/`? → Use `@sdk/*` for SDK-exported modules, `@Pimcore/*` for internal files
2. ✅ Am I in another bundle? → Use `@pimcore/studio-ui-bundle/*`
3. ✅ Am I importing UI components? → **NEVER from `antd`**, always from SDK
4. ✅ Are all imports from the correct namespace for my context? → Don't use `@sdk` or `@Pimcore` in bundles

## Common Import Patterns

### Core Development
```typescript
// File: studio-ui-bundle/assets/js/src/core/modules/my-feature/component.tsx

// SDK-exported modules → @sdk/*
import { container } from '@sdk'
import { serviceIds, trackError, ApiError } from '@sdk/modules/app'
import { FormKit, Form, Input, Button } from '@sdk/components'
import { useAssetGetByIdQuery } from '@sdk/api/asset'

// Internal files not in SDK → @Pimcore/*
import { SomeInternalHelper } from '@Pimcore/modules/my-feature/helper'

import { isNil } from 'lodash'
import React, { useEffect } from 'react'
```

### Bundle Development
```typescript
// File: my-bundle/assets/studio/js/src/components/my-component.tsx

import { container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
import { FormKit, Form, Input, Button } from '@pimcore/studio-ui-bundle/components'
import { useAssetGetByIdQuery } from '@pimcore/studio-ui-bundle/api/asset'
import { isNil } from 'lodash'
import React, { useEffect } from 'react'
```

**Notice:** The code is identical except for the import paths!

## Enforcement Strategy

When reviewing code or writing code:

1. **Check file location first** - Core or bundle?
2. **Verify ALL imports** use correct namespace
3. **Search for `from 'antd'`** - Should be ZERO occurrences
4. **Search for mixed imports** - All should use same base path
5. **Test imports resolve** - Run build to verify

## What Happens If You Break This Rule

### Using wrong namespace:
```typescript
// In core using @pimcore/studio-ui-bundle
import { FormKit } from '@pimcore/studio-ui-bundle/components'
// ❌ BUILD ERROR: Cannot find module '@pimcore/studio-ui-bundle/components'
```

### Importing from antd:
```typescript
import { Form } from 'antd'
// ❌ RUNTIME ERROR: Form won't have Pimcore customizations
// ❌ THEME ERROR: Styling won't match
// ❌ FUNCTIONALITY ERROR: Missing Pimcore-specific features
```

### Mixing namespaces:
```typescript
import { container } from '@sdk'
import { FormKit } from '@pimcore/studio-ui-bundle/components'
// ❌ BUILD ERROR: Inconsistent module resolution
```

## Remember

**ALWAYS CHECK YOUR CONTEXT BEFORE IMPORTING!**

Are you in `studio-ui-bundle/`? → `@sdk/*` (SDK-exported) or `@Pimcore/*` (internal)  
Are you in another bundle? → `@pimcore/studio-ui-bundle/*`  
Never ever ever → `antd`
