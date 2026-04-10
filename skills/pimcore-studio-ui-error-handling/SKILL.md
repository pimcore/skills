---
name: pimcore-studio-ui-error-handling
description: Error handling patterns in Pimcore Studio UI - trackError, ApiError, GeneralError, ErrorBoundary, and proper error flow
metadata:
  audience: pimcore-developers
  focus: error-handling
---

## What This Skill Covers

Error handling in Pimcore Studio UI: `trackError` for centralized reporting, `ApiError`/`GeneralError` wrappers, `ErrorBoundary` for render errors, and the standard reactive pattern for RTK Query.

## When to Use This Skill

- Handling errors from RTK Query mutations or queries
- Wrapping non-API (general/sync) errors for display
- Converting Axios/fetch errors to the trackError system
- Wrapping components with error boundaries

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## Core Imports

```typescript
import { trackError, ApiError, GeneralError, isApiErrorData, ErrorBoundary } from '@pimcore/studio-ui-bundle/modules/app'
```

## API Reference

| Symbol | Purpose |
|--------|---------|
| `trackError(provider, handler?)` | Centralized error display. Deduplicates errors within the same execution cycle. |
| `ApiError(error)` | Wraps RTK Query / structured API errors. Extracts `errorKey` and renders localized messages via `error.${errorKey}` translation keys. |
| `GeneralError(message)` | Wraps simple string errors. **Throws after display** — stops execution. |
| `isApiErrorData(unknown)` | Type guard for unknown error shapes. Returns true if object matches API error structure. |
| `ErrorBoundary` | React component that catches render errors. Optional `fallback` prop. |

### trackError Behavior

1. Accepts `ApiError` or `GeneralError`
2. Default: shows error modal via `ErrorModalService`
3. Optional second arg: custom handler callback (overrides modal)
4. **Deduplicates** identical errors within the same execution cycle
5. **GeneralError throws** after display — code after `trackError(new GeneralError(...))` does not execute

### When to Use Which

| Scenario | Use |
|----------|-----|
| RTK Query error from a query or mutation | `ApiError` |
| Axios/fetch error with structured response data | `ApiError({ data: error.response.data })` |
| Network failure / no response | `GeneralError` |
| Validation failure in client code | `GeneralError` |
| Missing required configuration / unexpected state | `GeneralError` |

## The Standard Pattern: RTK Query + useEffect + trackError

This is **the only correct way** to handle RTK Query errors in Pimcore Studio. Use the `error` state from the hook with `useEffect` and `trackError`.

```typescript
import { useAssetUpdateMutation } from '@pimcore/studio-ui-bundle/api/asset'
import { trackError, ApiError } from '@pimcore/studio-ui-bundle/modules/app'
import { useMessage } from '@pimcore/studio-ui-bundle/components'
import { useTranslation } from 'react-i18next'
import { useEffect } from 'react'
import { isNil } from 'lodash'

const AssetEditor = ({ asset }: { asset: Asset }): React.JSX.Element => {
  const { t } = useTranslation()
  const messageApi = useMessage()
  const [updateAsset, { data, error }] = useAssetUpdateMutation()

  // Track errors
  useEffect(() => {
    if (!isNil(error)) {
      trackError(new ApiError(error))
    }
  }, [error])

  // Handle success separately
  useEffect(() => {
    if (!isNil(data)) {
      void messageApi.success(t('save-success'))
    }
  }, [data])

  const handleSave = (values: FormValues): void => {
    updateAsset({ id: asset.id, body: values })
  }

  return <Form onFinish={ handleSave }>{/* fields */}</Form>
}
```

**Why this pattern:**
- Decouples error handling from the trigger action
- `trackError` deduplicates and provides consistent UX
- No try/catch needed — RTK Query exposes the error reactively

### Multiple Mutations

Use a **separate `useEffect` per mutation error**. Combining them into one effect causes stale closures and missed updates.

```typescript
const [createAsset, { error: createError }] = useAssetCreateMutation()
const [updateAsset, { error: updateError }] = useAssetUpdateMutation()

useEffect(() => {
  if (!isNil(createError)) trackError(new ApiError(createError))
}, [createError])

useEffect(() => {
  if (!isNil(updateError)) trackError(new ApiError(updateError))
}, [updateError])
```

## How ApiError Extracts Error Keys

When the API returns:

```json
{
  "errorKey": "asset.not-found",
  "message": "Asset with ID 123 was not found"
}
```

`ApiError` looks up the translation key `error.asset.not-found`. If the translation exists, it displays the localized message. Otherwise, it falls back to the raw `message` from the response.

## GeneralError for Sync / Validation Errors

Use `GeneralError` when there is no API response — validation failures, missing config, unexpected state. Remember it **throws after display**.

```typescript
import { trackError, GeneralError } from '@pimcore/studio-ui-bundle/modules/app'
import { isNil } from 'lodash'

const processElement = (element: Element | undefined): void => {
  if (isNil(element)) {
    trackError(new GeneralError('Cannot process: element is undefined'))
    // throws — code below is unreachable
  }

  element.update()
}
```

## Axios / Fetch Errors

Outside RTK Query, convert errors manually. Use `ApiError` when you have a structured response, `GeneralError` for network failures.

```typescript
import { trackError, ApiError, GeneralError } from '@pimcore/studio-ui-bundle/modules/app'
import { isNil } from 'lodash'

const uploadFile = async (url: string, data: FormData): Promise<void> => {
  try {
    await axios.post(url, data)
  } catch (error: any) {
    if (!isNil(error.response)) {
      trackError(new ApiError({ data: error.response.data }))
    } else {
      trackError(new GeneralError('Network error'))
    }
  }
}
```

## ErrorBoundary

Catches React render errors and prevents the entire app from crashing.

```typescript
import { ErrorBoundary } from '@pimcore/studio-ui-bundle/modules/app'

<ErrorBoundary fallback={ <div>Something went wrong</div> }>
  <MyComponent />
</ErrorBoundary>
```

**Important:** The widget container wraps every widget in an `ErrorBoundary` automatically. **Do not** add an `ErrorBoundary` around your widget root. Add your own only for isolated component trees that should fail independently (e.g., a third-party chart, a complex feature panel).

## isApiErrorData Type Guard

Use when you receive an error from a non-RTK-Query source and need to determine its shape.

```typescript
import { trackError, ApiError, GeneralError, isApiErrorData } from '@pimcore/studio-ui-bundle/modules/app'

const handleExternalError = (error: unknown): void => {
  if (isApiErrorData(error)) {
    trackError(new ApiError(error))
  } else if (error instanceof Error) {
    trackError(new GeneralError(error.message))
  } else {
    trackError(new GeneralError('An unknown error occurred'))
  }
}
```

## CRITICAL: Do NOT Use .unwrap() with try/catch

The `.unwrap()` pattern from generic Redux Toolkit usage is **wrong** in Pimcore Studio.

```typescript
// ❌ WRONG
const handleSave = async (): Promise<void> => {
  try {
    await updateAsset({ id, body }).unwrap()
    message.success('Saved')
  } catch (error) {
    message.error('Save failed')
  }
}
```

Problems:
1. Bypasses `trackError` — no centralized error handling, no error modal
2. No deduplication — errors can show multiple times
3. Inconsistent UX — error format differs from rest of app
4. Mixes imperative and reactive patterns

**Always use the `useEffect` + `error` state pattern shown above.**

## CRITICAL: Do NOT Double-Display Errors

`trackError` already shows the error modal. Do not also show a toast for the same error.

```typescript
// ❌ WRONG — duplicate display
useEffect(() => {
  if (!isNil(error)) {
    trackError(new ApiError(error))
    messageApi.error('Operation failed')  // redundant!
  }
}, [error])

// ✅ CORRECT
useEffect(() => {
  if (!isNil(error)) {
    trackError(new ApiError(error))
  }
}, [error])
```

## Common Mistakes to Avoid

### Forgetting isNil Check

```typescript
// ❌ WRONG — error could be undefined on mount
useEffect(() => {
  trackError(new ApiError(error))
}, [error])

// ✅ CORRECT
useEffect(() => {
  if (!isNil(error)) {
    trackError(new ApiError(error))
  }
}, [error])
```

### console.error Instead of trackError

```typescript
// ❌ WRONG — user never sees this
console.error('API call failed:', error)

// ✅ CORRECT
trackError(new ApiError(error))
```

### Combining Multiple Mutations in One Effect

```typescript
// ❌ WRONG — stale closures, missed updates
useEffect(() => {
  if (!isNil(createError)) trackError(new ApiError(createError))
  if (!isNil(updateError)) trackError(new ApiError(updateError))
}, [createError, updateError])

// ✅ CORRECT — separate useEffect per error
useEffect(() => {
  if (!isNil(createError)) trackError(new ApiError(createError))
}, [createError])

useEffect(() => {
  if (!isNil(updateError)) trackError(new ApiError(updateError))
}, [updateError])
```

### Wrapping Widgets in ErrorBoundary Manually

```typescript
// ❌ WRONG — widget container already provides ErrorBoundary
<ErrorBoundary fallback={ <div>Error</div> }>
  <MyWidget />
</ErrorBoundary>

// ✅ CORRECT — just render the widget
<MyWidget />
```

## Next Steps

- [**pimcore-studio-ui-rtk-query-fundamentals**](../pimcore-studio-ui-rtk-query-fundamentals/SKILL.md) - RTK Query basics
- [**pimcore-studio-ui-notifications-toasts**](../pimcore-studio-ui-notifications-toasts/SKILL.md) - Success messages alongside trackError
</content>
</invoke>