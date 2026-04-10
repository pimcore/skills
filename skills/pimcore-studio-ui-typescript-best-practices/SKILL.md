---
name: pimcore-studio-ui-typescript-best-practices
description: TypeScript coding standards and best practices for Pimcore Studio UI - type safety, null checks, and code quality
metadata:
  audience: pimcore-developers
  focus: typescript-code-quality
---

## What This Skill Covers

Critical TypeScript coding standards for Pimcore Studio UI:
- **Type Safety Rules** - Mandatory lodash utility usage for null/undefined checks
- **Nullish Coalescing** - Using `??` instead of `||` for default values
- **Type Annotations** - Explicit types for functions and public APIs
- **Avoiding `any`** - Proper use of `unknown` and generics
- **Type Guards** - Runtime type checking patterns
- **Common TypeScript Patterns** - Best practices used throughout the codebase

## When to Use This Skill

**Read this FIRST before writing any TypeScript code in Pimcore Studio UI.**

Use this when:
- Writing any TypeScript code (components, hooks, services, utilities)
- Checking for null/undefined values
- Setting default values
- Reviewing code for type safety
- Creating new functions or classes
- Working with dynamic data

## 📋 Import Context Note

These TypeScript best practices apply regardless of context (core or bundle). For SDK-specific imports like `trackError` or components, see [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).

## 🚨 CRITICAL: Type Safety is Mandatory

**Never use truthy/falsy checks or direct property access.**

These rules apply to **ALL** TypeScript code in the project - components, hooks, services, utilities, everywhere.

## Type Safety Rules

### Always Use Lodash Utilities for Checks

Import from lodash and use typesafe utility functions:

```typescript
import { isNull, isUndefined, isNil, isEmpty, isArray, isString, isNumber, isBoolean } from 'lodash'
```

### Null and Undefined Checks

```typescript
// ❌ DON'T - Unsafe, can fail with falsy values
if (!value) {
  // This triggers for: null, undefined, 0, false, '', NaN
}

if (value) {
  // This triggers for any truthy value
}

if (value == null) {
  // Loose equality is discouraged
}

if (value === null || value === undefined) {
  // Verbose and repetitive
}

// ✅ DO - Explicit and typesafe
if (isNil(value)) {
  // Only true for null or undefined
}

if (isUndefined(value)) {
  // Only true for undefined
}

if (isNull(value)) {
  // Only true for null
}
```

### Array and Collection Checks

```typescript
// ❌ DON'T - Unsafe
if (array.length === 0) {
  // Crashes if array is null/undefined
}

if (!array || array.length === 0) {
  // Works but verbose
}

// ✅ DO - Typesafe
if (isEmpty(array)) {
  // Handles null, undefined, empty arrays, empty objects, empty strings
}

// For arrays specifically
if (isArray(value) && isEmpty(value)) {
  // Checks both that it's an array AND that it's empty
}
```

### Type Checks

```typescript
// ❌ DON'T - Less readable
if (typeof value === 'string') { }
if (typeof value === 'number') { }
if (Array.isArray(value)) { }

// ✅ DO - Use lodash utilities
if (isString(value)) { }
if (isNumber(value)) { }
if (isBoolean(value)) { }
if (isArray(value)) { }
```

### Real-World Examples

```typescript
import { isNil, isEmpty, isUndefined } from 'lodash'

// Example 1: Checking element permissions
export const checkElementPermission = (
  permissions: ElementPermissions | undefined,
  permission: ElementPermissionKeys
): boolean => {
  // ✅ CORRECT - Using isUndefined
  if (isUndefined(permissions)) {
    return false
  }
  return permissions[permission] === true
}

// Example 2: Checking asset draft
export const useAssetDraft = (id: number) => {
  const { asset } = useAsset(id)
  
  // ✅ CORRECT - Using isNil
  if (isNil(asset)) {
    return null
  }
  
  // ✅ CORRECT - Using isEmpty for arrays
  if (isEmpty(asset.properties)) {
    return defaultProperties
  }
  
  return asset
}

// Example 3: Validating form data
export const validateForm = (values: FormValues) => {
  const errors: Record<string, string> = {}
  
  // ✅ CORRECT - Using isNil and isEmpty
  if (isNil(values.username) || isEmpty(values.username)) {
    errors.username = 'Username is required'
  }
  
  return errors
}

// Example 4: API response handling
export const MyComponent = () => {
  const { data, isLoading, error } = useQuery()
  
  // ✅ CORRECT - Explicit checks
  if (isLoading) {
    return <Loader />
  }
  
  if (!isUndefined(error)) {
    return <Error message={error.message} />
  }
  
  if (isNil(data)) {
    return <EmptyState />
  }
  
  return <DataView data={data} />
}
```

## Nullish Coalescing Operator (??)

### Always Use ?? Instead of || for Default Values

The nullish coalescing operator (`??`) only checks for `null` and `undefined`, not all falsy values.

```typescript
// ❌ DON'T - Logical OR fails with falsy values
const count = userInput || 0
// If userInput is 0, result is 0 (unexpected!)
// If userInput is false, result is 0 (unexpected!)
// If userInput is '', result is 0 (unexpected!)

const enabled = settings.enabled || true
// If settings.enabled is false, result is true (WRONG!)

// ✅ DO - Nullish coalescing only checks null/undefined
const count = userInput ?? 0
// If userInput is 0, result is 0 (correct!)
// If userInput is false, result is false (correct!)
// If userInput is null/undefined, result is 0 (correct!)

const enabled = settings.enabled ?? true
// If settings.enabled is false, result is false (correct!)
// If settings.enabled is null/undefined, result is true (correct!)
```

### Real-World Examples

```typescript
import { isNil } from 'lodash'

// Example 1: Component props with defaults
interface ButtonProps {
  loading?: boolean
  disabled?: boolean
  size?: 'small' | 'medium' | 'large'
}

export const Button = ({ loading, disabled, size }: ButtonProps) => {
  // ✅ CORRECT - false is a valid value
  const isLoading = loading ?? false
  const isDisabled = disabled ?? false
  const buttonSize = size ?? 'medium'
  
  // ❌ WRONG - This would fail if loading is false
  // const isLoading = loading || false  // Always false!
}

// Example 2: Widget configuration
export const WidgetForm = () => {
  const { widget } = useWidget(id)
  
  // ✅ CORRECT - false is a valid value for isWriteable
  const isWriteable = widget.isWriteable ?? true
  
  // ❌ WRONG - If isWriteable is false, this would return true
  // const isWriteable = widget.isWriteable || true
}

// Example 3: Pagination defaults
export const usePagination = (initialPage?: number) => {
  // ✅ CORRECT - Page 0 is valid
  const [page, setPage] = useState(initialPage ?? 0)
  
  // ❌ WRONG - Page 0 would become 1
  // const [page, setPage] = useState(initialPage || 1)
}

// Example 4: Number inputs
export const NumberInput = ({ value, defaultValue }: Props) => {
  // ✅ CORRECT - 0 is a valid number
  const currentValue = value ?? defaultValue ?? 0
  
  // ❌ WRONG - 0 would be replaced with defaultValue
  // const currentValue = value || defaultValue || 0
}
```

### When to Use || vs ??

```typescript
// Use ?? for default values (checks only null/undefined)
const count = value ?? 0
const name = user.name ?? 'Anonymous'
const enabled = settings.enabled ?? true

// Use || only when you explicitly want to check all falsy values
const trimmedName = name.trim() || 'Untitled'  // Empty string → 'Untitled'
const hasContent = content.length > 0 || hasPlaceholder  // Boolean logic
```

## Type Annotations

### Always Annotate Function Return Types

```typescript
// ❌ DON'T - Inferred return type
export const calculateTotal = (items: Item[]) => {
  return items.reduce((sum, item) => sum + item.price, 0)
}

// ✅ DO - Explicit return type
export const calculateTotal = (items: Item[]): number => {
  return items.reduce((sum, item) => sum + item.price, 0)
}

// ✅ DO - React components
export const MyComponent = (props: Props): React.JSX.Element => {
  return <div>{props.children}</div>
}

// ✅ DO - Async functions
export const fetchData = async (id: number): Promise<Data> => {
  const response = await api.getData(id)
  return response.data
}

// ✅ DO - Functions returning nothing
export const logMessage = (message: string): void => {
  console.log(message)
}
```

### Always Annotate Public API Parameters

```typescript
// ❌ DON'T - Implicit any
export const processData = (data) => {
  return data.map(item => item.value)
}

// ✅ DO - Explicit types
export const processData = (data: DataItem[]): number[] => {
  return data.map(item => item.value)
}

// ✅ DO - Interface for complex parameters
interface ProcessOptions {
  sort?: boolean
  filter?: (item: DataItem) => boolean
  transform?: (value: number) => number
}

export const processDataAdvanced = (
  data: DataItem[],
  options: ProcessOptions
): number[] => {
  let result = data.map(item => item.value)
  
  if (options.filter !== undefined) {
    result = data.filter(options.filter).map(item => item.value)
  }
  
  if (options.transform !== undefined) {
    result = result.map(options.transform)
  }
  
  return result
}
```

## Avoiding `any`

### Use `unknown` for Truly Dynamic Data

```typescript
// ❌ DON'T - any defeats type checking
const parseJSON = (input: string): any => {
  return JSON.parse(input)
}

// ✅ DO - unknown requires type checking
const parseJSON = (input: string): unknown => {
  return JSON.parse(input)
}

// Usage requires type guard
const result = parseJSON(jsonString)
if (isString(result)) {
  console.log(result.toUpperCase())
}
```

### Use Generic Constraints

```typescript
// ❌ DON'T - Too permissive
function getProperty<T>(obj: any, key: string): any {
  return obj[key]
}

// ✅ DO - Proper generic constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}

// Usage
interface User {
  name: string
  age: number
}

const user: User = { name: 'John', age: 30 }
const name = getProperty(user, 'name')  // Type: string
const age = getProperty(user, 'age')    // Type: number
// const invalid = getProperty(user, 'invalid')  // Error: Argument not assignable
```

### Proper Type Guards

```typescript
import { isString, isNumber, isArray } from 'lodash'

// Type guard for custom types
interface User {
  id: number
  name: string
}

function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value &&
    isNumber((value as User).id) &&
    isString((value as User).name)
  )
}

// Usage
const data: unknown = await fetchData()

if (isUser(data)) {
  // TypeScript knows data is User here
  console.log(data.name)
}

// Array type guard
function isStringArray(value: unknown): value is string[] {
  return isArray(value) && value.every(isString)
}

// Usage
if (isStringArray(data)) {
  // TypeScript knows data is string[] here
  data.forEach(str => console.log(str.toUpperCase()))
}
```

## Common TypeScript Patterns

### Optional Chaining with Nullish Coalescing

```typescript
// ✅ Combining optional chaining (?.) with nullish coalescing (??)
const userName = user?.profile?.name ?? 'Anonymous'
const itemCount = data?.items?.length ?? 0
const isEnabled = settings?.features?.newUI?.enabled ?? false

// This is safe and concise - checks all levels for null/undefined
```

### Discriminated Unions

```typescript
// Define discriminated union types
type LoadingState = 
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: Data }
  | { status: 'error'; error: Error }

// TypeScript can narrow types based on discriminator
function handleState(state: LoadingState): React.JSX.Element {
  switch (state.status) {
    case 'idle':
      return <div>Ready to load</div>
    case 'loading':
      return <Spinner />
    case 'success':
      // TypeScript knows state.data exists here
      return <DataView data={state.data} />
    case 'error':
      // TypeScript knows state.error exists here
      return <Error message={state.error.message} />
  }
}
```

### Readonly and Immutability

```typescript
// Use readonly for props that shouldn't be modified
interface Props {
  readonly data: ReadonlyArray<Item>
  readonly config: Readonly<Config>
}

// Use const assertions for literal types
const ACTIONS = {
  ADD: 'add',
  REMOVE: 'remove',
  UPDATE: 'update'
} as const

type ActionType = typeof ACTIONS[keyof typeof ACTIONS]
// Type: 'add' | 'remove' | 'update'
```

### Type Narrowing with lodash

```typescript
import { isNil, isEmpty, isString, isNumber } from 'lodash'

function processValue(value: string | number | null | undefined): string {
  // Narrow out null/undefined
  if (isNil(value)) {
    return 'No value'
  }
  
  // Now value is string | number
  if (isString(value)) {
    return value.toUpperCase()
  }
  
  // TypeScript knows value is number here
  return value.toString()
}
```

## Common Mistakes

### ❌ Using Truthy/Falsy Checks

```typescript
// DON'T
if (!value) { }
if (value) { }
if (array.length) { }

// DO
if (isNil(value)) { }
if (!isNil(value)) { }
if (!isEmpty(array)) { }
```

### ❌ Using || for Default Values

```typescript
// DON'T - Fails with 0, false, ''
const count = props.count || 0
const enabled = config.enabled || true

// DO - Only checks null/undefined
const count = props.count ?? 0
const enabled = config.enabled ?? true
```

### ❌ Missing Return Type Annotations

```typescript
// DON'T - Implicit return type
export const getData = async (id: number) => {
  const response = await api.get(id)
  return response.data
}

// DO - Explicit return type
export const getData = async (id: number): Promise<Data> => {
  const response = await api.get(id)
  return response.data
}
```

### ❌ Using `any` Instead of `unknown`

```typescript
// DON'T - Defeats type safety
const parseJSON = (str: string): any => JSON.parse(str)

// DO - Requires type checking
const parseJSON = (str: string): unknown => JSON.parse(str)
```

### ❌ Not Using Type Guards

```typescript
// DON'T - Unsafe cast
const data: unknown = await fetchData()
const user = data as User  // No runtime check!
console.log(user.name)

// DO - Type guard with runtime check
const data: unknown = await fetchData()
if (isUser(data)) {
  console.log(data.name)  // Safe
}
```

### ❌ Direct Property Access Without Checks

```typescript
// DON'T - Can crash
const name = user.profile.name

// DON'T - Verbose
const name = user && user.profile && user.profile.name

// DO - Optional chaining + nullish coalescing
const name = user?.profile?.name ?? 'Anonymous'

// DO - Explicit check with lodash
if (!isNil(user?.profile)) {
  const name = user.profile.name
}
```

## Quick Reference

### Lodash Utilities (Must Use)

| Check | Use | Instead of |
|-------|-----|------------|
| Null or undefined | `isNil(value)` | `!value`, `value == null` |
| Undefined only | `isUndefined(value)` | `value === undefined` |
| Null only | `isNull(value)` | `value === null` |
| Empty (array/object/string) | `isEmpty(value)` | `array.length === 0` |
| Is array | `isArray(value)` | `Array.isArray(value)` |
| Is string | `isString(value)` | `typeof value === 'string'` |
| Is number | `isNumber(value)` | `typeof value === 'number'` |
| Is boolean | `isBoolean(value)` | `typeof value === 'boolean'` |

### Operators

| Operator | Use For | Example |
|----------|---------|---------|
| `??` | Default values (null/undefined only) | `value ?? defaultValue` |
| `?.` | Safe property access | `user?.profile?.name` |
| `||` | Boolean logic or explicit falsy checks | `hasA || hasB` |

### Type Annotations

| Context | Pattern |
|---------|---------|
| Function return | `(): ReturnType => { }` |
| React component | `(): React.JSX.Element => { }` |
| Async function | `async (): Promise<Type> => { }` |
| No return value | `(): void => { }` |
| Parameters | `(param: Type): ReturnType => { }` |

### Type Safety Checklist

- [ ] All null/undefined checks use lodash utilities (isNil, isEmpty, etc.)
- [ ] All default values use `??` not `||`
- [ ] All functions have explicit return types
- [ ] No usage of `any` (use `unknown` instead)
- [ ] Type guards implemented for dynamic data
- [ ] Optional chaining (`?.`) used for nested property access
- [ ] Public APIs have full type annotations

## Next Steps

- [**pimcore-studio-ui-react-components**](../pimcore-studio-ui-react-components/SKILL.md) - Apply these rules to React components
- [**pimcore-studio-ui-rtk-query-fundamentals**](../pimcore-studio-ui-rtk-query-fundamentals/SKILL.md) - Type safety in API calls
- [**pimcore-studio-ui-forms-antd**](../pimcore-studio-ui-forms-antd/SKILL.md) - Form type safety and validation
- [**pimcore-studio-ui-permissions**](../pimcore-studio-ui-permissions/SKILL.md) - Type-safe permission checks
