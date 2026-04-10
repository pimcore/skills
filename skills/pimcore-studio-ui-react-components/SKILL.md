---
name: pimcore-studio-ui-react-components
description: Creating React components in Pimcore Studio UI - component structure, props patterns, styling with CSS-in-JS, file organization, component vs module placement, Storybook requirements
metadata:
  audience: pimcore-developers
  focus: ui-components
---

## What This Skill Covers

Best practices for creating React components in Pimcore Studio UI:
- Component file structure and folder organization
- Function component format with proper return types
- Props type definitions with interfaces
- CSS-in-JS styling with `antd-style`
- Component patterns (simple, forwardRef, complex)
- Export patterns and code organization
- When to split up heavy components

## When to Use This Skill

Use this when:
- Creating a new React component
- Refactoring existing components
- Need to add styling to components
- Understanding component organization
- Splitting complex components into smaller pieces
- Following Pimcore Studio UI conventions

## 🚨 CRITICAL: Component vs Module Components

**Where you place your component determines if it needs Storybook stories:**

### `/core/components/` - Reusable UI Components
- **Location**: `assets/studio/js/src/core/components/`
- **Purpose**: Generic, reusable UI components used across the application
- **Examples**: Button, Input, Modal, Card, Icon, Badge, Tooltip
- **Storybook**: ✅ **REQUIRED** - Must have `.stories.tsx` file
- **Exports**: Exported from `@sdk/components` or `@pimcore/studio-ui-bundle/components`

```
core/components/
├── button/
│   ├── button.tsx
│   ├── button.styles.tsx
│   └── button.stories.tsx        ✅ Required!
└── icon/
    ├── icon.tsx
    └── icon.stories.tsx            ✅ Required!
```

### `/core/modules/` - Business Logic Components
- **Location**: `assets/studio/js/src/core/modules/[module-name]/components/`
- **Purpose**: Module-specific, business logic components
- **Examples**: AssetEditor, DataObjectGrid, ElementTree, SaveButton
- **Storybook**: ❌ **NOT REQUIRED** - No stories needed
- **Exports**: Not exported from main components barrel

```
core/modules/
├── asset/
│   └── components/
│       └── asset-editor/
│           ├── asset-editor.tsx
│           └── asset-editor.styles.tsx    ❌ No stories needed
└── element/
    └── components/
        └── element-tree/
            ├── element-tree.tsx
            └── element-tree.styles.tsx    ❌ No stories needed
```

### Decision Tree: Where Should My Component Go?

```
Is the component reusable across multiple modules?
│
├─ YES → Is it generic UI (Button, Input, Card, etc.)?
│   │
│   ├─ YES → `/core/components/` + Storybook stories ✅
│   │
│   └─ NO → Is it business-logic specific (AssetEditor, etc.)?
│       └─ YES → `/core/modules/[module]/components/` (no stories)
│
└─ NO → Only used in one module?
    └─ `/core/modules/[module]/components/` (no stories)
```

### Examples of Each Type

**Reusable UI Components** (`/core/components/`):
- Button, IconButton, ButtonGroup
- Input, Textarea, Select, Checkbox
- Modal, Drawer, Popover, Tooltip
- Card, Panel, Collapse, Accordion
- Icon, Badge, Tag, Avatar
- Spin, Skeleton, Progress

**Module Components** (`/core/modules/*/components/`):
- AssetEditor, AssetGrid, AssetDetailPanel
- DataObjectEditor, DataObjectTree
- ElementTree, ElementToolbar
- SaveButton, PublishButton (in asset/document modules)
- FilterPanel, SearchResults (in search module)

## 🚨 CRITICAL: TypeScript Type Safety

**Before writing any code, read:** [**pimcore-studio-ui-typescript-best-practices**](../pimcore-studio-ui-typescript-best-practices/SKILL.md)

All code must follow strict type safety rules:
- ✅ Use `isNil()`, `isEmpty()`, `isUndefined()` from lodash (never `!value` or `value == null`)
- ✅ Use `??` for default values (never `||`)
- ✅ Annotate all function return types
- ✅ Avoid `any`, use `unknown` with type guards

```typescript
// ❌ DON'T
if (!data) return null
const value = props.value || 0

// ✅ DO
import { isNil } from 'lodash'
if (isNil(data)) return null
const value = props.value ?? 0
```

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## Component File Structure

### Rule: Always Use Folder Structure

**❌ DON'T create single files:**
```
components/
└── my-component.tsx
```

**✅ DO create component folders:**
```
components/
└── my-component/
    ├── my-component.tsx
    ├── my-component.styles.tsx
    └── my-component.stories.tsx (optional)
```

**Why:**
- Components often need related files (styles, tests, sub-components)
- Easier to find all related files
- Consistent structure across codebase
- Room to grow without refactoring

### This Applies to ALL Component Levels

Even in subdirectories, use folders:

**❌ BAD:**
```
components/
└── toolbar/
    ├── toolbar.tsx
    ├── toolbar.styles.tsx
    └── save-button.tsx              ← Single file
```

**✅ GOOD:**
```
components/
└── toolbar/
    ├── toolbar.tsx
    ├── toolbar.styles.tsx
    └── save-button/                 ← Folder
        ├── save-button.tsx
        └── save-button.styles.tsx
```

### Complex Components with Sub-Components

For components with multiple related files:

```
my-complex-component/
├── my-complex-component.tsx          # Main component
├── my-complex-component.styles.tsx   # Styles
├── my-complex-component.stories.tsx  # Storybook stories
├── index.ts                          # Re-exports
├── sub-component-a/                  # Sub-component
│   ├── sub-component-a.tsx
│   └── sub-component-a.styles.tsx
├── sub-component-b/                  # Another sub-component
│   ├── sub-component-b.tsx
│   └── sub-component-b.styles.tsx
└── hooks/                            # Related hooks
    └── use-my-complex-logic.tsx
```

**Example from codebase (Modal):**
```
modal/
├── modal.tsx
├── modal.styles.tsx
├── modal.stories.tsx
├── alert-modal/
│   └── alert-modal.tsx
├── form-modal/
│   └── form-modal.tsx
├── footer/
│   └── footer.tsx
└── hooks/
    └── useModal/
        └── use-modal.tsx
```

## Component Format Pattern

### Basic Component Template

```typescript
import React from 'react'
import { useStyles } from './component-name.styles'

export interface ComponentNameProps {
  // Props definition
}

export const ComponentName = (props: ComponentNameProps): React.JSX.Element => {
  const { styles } = useStyles()
  
  // Component logic
  
  return (
    <div className={styles.container}>
      {/* Component JSX */}
    </div>
  )
}
```

### Key Rules

1. **Return Type:** Always specify `React.JSX.Element` (or `React.JSX.Element | null` if component can return null)
2. **Props Interface:** Always define and export the props interface
3. **Naming:** Use PascalCase for component and `ComponentNameProps` for props interface
4. **Export:** Use named exports, not default exports
5. **Styles:** Import from separate `.styles.tsx` file

## Props Type Definitions

### Simple Props Interface

```typescript
export interface ButtonProps {
  label: string
  onClick: () => void
  disabled?: boolean
  type?: 'primary' | 'secondary'
  icon?: React.JSX.Element
}

export const Button = (props: ButtonProps): React.JSX.Element => {
  const { label, onClick, disabled = false, type = 'primary', icon } = props
  
  return (
    <button onClick={onClick} disabled={disabled}>
      {icon}
      {label}
    </button>
  )
}
```

**Best practices:**
- Use `?` for optional props
- Provide default values in destructuring
- Export the interface for external use

### Extending Ant Design Props

```typescript
import { type ButtonProps as AntButtonProps } from 'antd'

export interface CustomButtonProps extends AntButtonProps {
  customProp?: string
  theme?: 'primary' | 'secondary'
}

export const CustomButton = ({ theme = 'primary', ...props }: CustomButtonProps): React.JSX.Element => {
  return <AntButton {...props} className={`button--${theme}`} />
}
```

### Omitting Props When Extending

```typescript
import { type ButtonProps as AntButtonProps } from 'antd'

// Omit certain props and replace with custom ones
export interface IconButtonProps extends Omit<AntButtonProps, 'icon'> {
  icon: IconProps  // Custom icon prop
  tooltip?: string
  theme?: 'primary' | 'secondary'
}

export const IconButton = (props: IconButtonProps): React.JSX.Element => {
  const { icon, tooltip, theme = 'primary', ...buttonProps } = props
  // Implementation
}
```

### Complex Props with Union Types

```typescript
export interface DataValue {
  id: number
  type: 'asset' | 'document' | 'object'
  path: string
}

export interface TextValue {
  textInput: true
  path: string
}

export type ComponentValue = DataValue | TextValue | null

export interface ManyToOneProps {
  value?: ComponentValue
  onChange?: (value: ComponentValue) => void
  disabled?: boolean
  readOnly?: boolean
}
```

## CSS-in-JS with antd-style

### 🚨 MANDATORY: Separate .styles.tsx Files

**ALL component styles MUST be in a separate `.styles.tsx` file. NEVER use inline styles!**

```typescript
// ❌ WRONG - Inline styles forbidden
export const Button = (): React.JSX.Element => {
  return (
    <div style={{ padding: '16px', background: '#fff' }}>
      Content
    </div>
  )
}

// ✅ CORRECT - Separate .styles.tsx file
// button.styles.tsx
import { createStyles } from 'antd-style'

export const useStyles = createStyles(({ token, css }) => ({
  container: css`padding: ${token.padding}px`
}))

// button.tsx
import { useStyles } from './button.styles'

export const Button = (): React.JSX.Element => {
  const { styles } = useStyles()
  return <div className={styles.container}>Content</div>
}
```

**Why separate files are mandatory:**
- **Consistency** - Same pattern across entire codebase
- **Maintainability** - Easy to find and update styles
- **Reusability** - Styles can be imported by sub-components
- **Code organization** - Clear separation of concerns

**Exception**: Small utility components (< 10 lines) with no custom styling can omit .styles.tsx file.

### Basic Styles File Pattern

```typescript
// component-name.styles.tsx
import { createStyles } from 'antd-style'

export const useStyles = createStyles(({ token, css }) => {
  return {
    container: css`
      display: flex;
      padding: ${token.padding}px;
      background-color: ${token.colorBgContainer};
      border-radius: ${token.borderRadius}px;
    `
  }
})

// component-name.tsx
import { useStyles } from './component-name.styles'

export const ComponentName = (): React.JSX.Element => {
  const { styles } = useStyles()
  return <div className={styles.container}>Content</div>
}
```

### Common Design Tokens

```typescript
export const useStyles = createStyles(({ token, css }) => ({
  button: css`
    // Colors
    color: ${token.colorPrimary};
    background: ${token.colorBgContainer};
    
    // Spacing
    padding: ${token.paddingSM}px;
    margin: ${token.marginXS}px;
    
    // Typography
    font-size: ${token.fontSize}px;
    border-radius: ${token.borderRadius}px;
    
    // States
    &:hover {
      background: ${token.colorBgTextHover};
    }
  `
}))
```

**🎨 For complete styling patterns, see:** [styling-patterns.md](./styling-patterns.md)

This covers:
- Complete design token reference
- BEM-style modifiers
- Nested selectors and pseudo-classes
- Style priority control with `hashPriority`
- Real-world styling examples

## Component Patterns

### Pattern 1: Simple Wrapper Component

Wrap Ant Design components with custom defaults:

```typescript
import React from 'react'
import { Badge as AntBadge, type BadgeProps } from 'antd'

export const Badge = ({ color, ...props }: BadgeProps): React.JSX.Element => {
  return (
    <AntBadge
      color={color}
      styles={{
        indicator: { outline: `1px solid ${color}` },
        root: { marginRight: '5px' }
      }}
      {...props}
    />
  )
}
```

### Pattern 2: Component with Custom Props

```typescript
import React from 'react'
import cn from 'classnames'
import { Title } from '@pimcore/studio-ui-bundle/components'
import { useStyles } from './header.styles'

export interface HeaderProps {
  title: string
  icon?: React.JSX.Element
  className?: string
  fullWidth?: boolean
  children?: React.ReactNode
}

export const Header = (props: HeaderProps): React.JSX.Element => {
  const { styles } = useStyles()
  const { icon, title, children, className, fullWidth } = props
  
  return (
    <div className={cn(styles.header, className)}>
      {title !== '' && (
        <span className="header__text">
          <Title icon={icon}>
            {title}
          </Title>
        </span>
      )}
      
      <div className={cn('header__content', { 'w-full': fullWidth })}>
        {children}
      </div>
    </div>
  )
}
```

### Pattern 3: forwardRef Component

When component needs ref access:

```typescript
import React, { forwardRef } from 'react'
import { Button, type ButtonProps } from 'antd'
import { Icon, type IconProps } from '../icon/icon'
import { useStyles } from './icon-button.styles'

export interface IconButtonProps extends Omit<ButtonProps, 'icon'> {
  icon: IconProps
  theme?: 'primary' | 'secondary'
}

const Component = (props: IconButtonProps, ref): React.JSX.Element => {
  const { icon, theme = 'primary', ...buttonProps } = props
  const { styles } = useStyles()
  
  return (
    <Button
      ref={ref}
      className={styles.button}
      {...buttonProps}
    >
      <Icon {...icon} />
    </Button>
  )
}

export const IconButton = forwardRef(Component)
```

### Pattern 4: forwardRef with Typed Ref

For better type safety:

```typescript
import React, { forwardRef } from 'react'
import { Input, type InputRef } from 'antd'
import { type SearchProps as AntSearchProps } from 'antd/es/input/Search'
import { useStyles } from './search-input.styles'

export interface SearchInputProps extends AntSearchProps {
  withPrefix?: boolean
  withClear?: boolean
}

export const SearchInput = forwardRef<InputRef, SearchInputProps>(
  ({ withPrefix = false, withClear = true, ...props }, ref): React.JSX.Element => {
    const { styles } = useStyles()
    
    return (
      <Input.Search
        ref={ref}
        allowClear={withClear}
        className={styles.search}
        {...props}
      />
    )
  }
)

SearchInput.displayName = 'SearchInput'
```

**Key points:**
- Type `forwardRef<RefType, PropsType>`
- Add `displayName` for React DevTools

### Pattern 5: Component with Hooks

```typescript
import React, { useState, useEffect, useRef } from 'react'
import { useStyles } from './component.styles'

export interface ComponentProps {
  initialValue?: string
  onChange?: (value: string) => void
}

export const Component = ({ initialValue = '', onChange }: ComponentProps): React.JSX.Element => {
  const [value, setValue] = useState(initialValue)
  const inputRef = useRef<HTMLInputElement>(null)
  const { styles } = useStyles()
  
  useEffect(() => {
    if (inputRef.current) {
      inputRef.current.focus()
    }
  }, [])
  
  const handleChange = (newValue: string): void => {
    setValue(newValue)
    onChange?.(newValue)
  }
  
  return (
    <input
      ref={inputRef}
      value={value}
      onChange={(e) => handleChange(e.target.value)}
      className={styles.input}
    />
  )
}
```

### Pattern 6: Controlled Component

Support both controlled and uncontrolled usage:

```typescript
import React from 'react'
import { useControlledState } from '@pimcore/studio-ui-bundle/utils'

export interface ControlledComponentProps {
  value?: string
  defaultValue?: string
  onChange?: (value: string) => void
}

export const ControlledComponent = (props: ControlledComponentProps): React.JSX.Element => {
  const { value, handleChange } = useControlledState<string>(
    props.value ?? props.defaultValue ?? '',
    props.onChange
  )
  
  return (
    <input
      value={value}
      onChange={(e) => handleChange(e.target.value)}
    />
  )
}
```

### Pattern 7: Component with Context

```typescript
import React, { useContext } from 'react'
import { MyContext } from './context'
import { useStyles } from './component.styles'

export interface ComponentProps {
  // props
}

export const Component = (props: ComponentProps): React.JSX.Element | null => {
  const context = useContext(MyContext)
  const { styles } = useStyles()
  
  if (!context) {
    return null
  }
  
  return (
    <div className={styles.container}>
      {context.data}
    </div>
  )
}
```

## Splitting Heavy Components

### When to Split

Split a component when:
- **Lines of code:** Component exceeds ~200-300 lines
- **Multiple responsibilities:** Component does several unrelated things
- **Reusable parts:** Sections could be used elsewhere
- **Complex logic:** Hard to understand what component does
- **Deep nesting:** JSX is nested more than 3-4 levels

### Strategy 1: Extract Sub-Components

**Before (Heavy Component):**
```typescript
export const UserProfile = ({ user }: UserProfileProps): React.JSX.Element => {
  return (
    <div>
      {/* Header section - 50 lines */}
      <div className="header">
        <img src={user.avatar} />
        <div>
          <h1>{user.name}</h1>
          <p>{user.email}</p>
          <div>{user.badges.map(...)}</div>
        </div>
      </div>
      
      {/* Stats section - 50 lines */}
      <div className="stats">
        <div>Posts: {user.posts}</div>
        <div>Followers: {user.followers}</div>
        {/* More stats */}
      </div>
      
      {/* Activity section - 100 lines */}
      <div className="activity">
        {user.activities.map(activity => (
          <div key={activity.id}>
            {/* Complex activity rendering */}
          </div>
        ))}
      </div>
    </div>
  )
}
```

**After (Split into Sub-Components):**

```
user-profile/
├── user-profile.tsx
├── user-profile.styles.tsx
├── user-header/
│   ├── user-header.tsx
│   └── user-header.styles.tsx
├── user-stats/
│   ├── user-stats.tsx
│   └── user-stats.styles.tsx
└── user-activity/
    ├── user-activity.tsx
    └── user-activity.styles.tsx
```

```typescript
// user-profile.tsx
export const UserProfile = ({ user }: UserProfileProps): React.JSX.Element => {
  return (
    <div>
      <UserHeader user={user} />
      <UserStats user={user} />
      <UserActivity activities={user.activities} />
    </div>
  )
}

// user-header/user-header.tsx
export const UserHeader = ({ user }: UserHeaderProps): React.JSX.Element => {
  // Header implementation
}

// user-stats/user-stats.tsx
export const UserStats = ({ user }: UserStatsProps): React.JSX.Element => {
  // Stats implementation
}

// user-activity/user-activity.tsx
export const UserActivity = ({ activities }: UserActivityProps): React.JSX.Element => {
  // Activity implementation
}
```

### Strategy 2: Extract Custom Hooks

Move complex logic to custom hooks:

**Before:**
```typescript
export const DataTable = ({ dataSource }: DataTableProps): React.JSX.Element => {
  const [filteredData, setFilteredData] = useState(dataSource)
  const [sortColumn, setSortColumn] = useState<string | null>(null)
  const [sortDirection, setSortDirection] = useState<'asc' | 'desc'>('asc')
  const [selectedRows, setSelectedRows] = useState<number[]>([])
  
  // 50 lines of sorting logic
  const handleSort = (column: string) => {
    // complex sorting
  }
  
  // 30 lines of filtering logic
  const handleFilter = (filters: Filters) => {
    // complex filtering
  }
  
  // 20 lines of selection logic
  const handleSelect = (rowId: number) => {
    // complex selection
  }
  
  return (
    <table>
      {/* table implementation */}
    </table>
  )
}
```

**After:**

```
data-table/
├── data-table.tsx
├── data-table.styles.tsx
└── hooks/
    ├── use-table-sorting.tsx
    ├── use-table-filtering.tsx
    └── use-table-selection.tsx
```

```typescript
// hooks/use-table-sorting.tsx
export const useTableSorting = (data: DataItem[]) => {
  const [sortColumn, setSortColumn] = useState<string | null>(null)
  const [sortDirection, setSortDirection] = useState<'asc' | 'desc'>('asc')
  
  const sortedData = useMemo(() => {
    // sorting logic
  }, [data, sortColumn, sortDirection])
  
  const handleSort = (column: string) => {
    // sorting implementation
  }
  
  return { sortedData, sortColumn, sortDirection, handleSort }
}

// data-table.tsx
export const DataTable = ({ dataSource }: DataTableProps): React.JSX.Element => {
  const { sortedData, handleSort } = useTableSorting(dataSource)
  const { filteredData, handleFilter } = useTableFiltering(sortedData)
  const { selectedRows, handleSelect } = useTableSelection()
  
  return (
    <table>
      {/* Clean table implementation */}
    </table>
  )
}
```

### Strategy 3: Extract Render Functions

For complex conditional rendering:

```typescript
export const Dashboard = ({ user, data }: DashboardProps): React.JSX.Element => {
  const renderHeader = (): React.JSX.Element => {
    return (
      <div>
        {/* Header JSX */}
      </div>
    )
  }
  
  const renderContent = (): React.JSX.Element => {
    if (!data) {
      return <EmptyState />
    }
    
    if (data.error) {
      return <ErrorState error={data.error} />
    }
    
    return (
      <div>
        {/* Content JSX */}
      </div>
    )
  }
  
  const renderFooter = (): React.JSX.Element => {
    return (
      <div>
        {/* Footer JSX */}
      </div>
    )
  }
  
  return (
    <div>
      {renderHeader()}
      {renderContent()}
      {renderFooter()}
    </div>
  )
}
```

### Strategy 4: Use Composition

Build complex components from simpler ones:

```typescript
// Simple building blocks
export const Card = ({ children }: CardProps): React.JSX.Element => (
  <div className="card">{children}</div>
)

export const CardHeader = ({ title }: CardHeaderProps): React.JSX.Element => (
  <div className="card-header">{title}</div>
)

export const CardBody = ({ children }: CardBodyProps): React.JSX.Element => (
  <div className="card-body">{children}</div>
)

export const CardFooter = ({ children }: CardFooterProps): React.JSX.Element => (
  <div className="card-footer">{children}</div>
)

// Compose them
export const UserCard = ({ user }: UserCardProps): React.JSX.Element => {
  return (
    <Card>
      <CardHeader title={user.name} />
      <CardBody>
        <UserInfo user={user} />
      </CardBody>
      <CardFooter>
        <UserActions user={user} />
      </CardFooter>
    </Card>
  )
}
```

## Export Patterns

### Pattern 1: Direct Named Export

```typescript
// component-name.tsx
export const ComponentName = (props: ComponentNameProps): React.JSX.Element => {
  // implementation
}
```

### Pattern 2: Export Props with Component

```typescript
// component-name.tsx
export interface ComponentNameProps {
  // props
}

export const ComponentName = (props: ComponentNameProps): React.JSX.Element => {
  // implementation
}
```

### Pattern 3: Index File Re-exports

For complex components with sub-components:

```typescript
// my-component/index.ts
export { MyComponent } from './my-component'
export type { MyComponentProps } from './my-component'
export { SubComponentA } from './sub-component-a/sub-component-a'
export type { SubComponentAProps } from './sub-component-a/sub-component-a'
export { SubComponentB } from './sub-component-b/sub-component-b'
export type { SubComponentBProps } from './sub-component-b/sub-component-b'
```

**Usage:**
```typescript
import { MyComponent, SubComponentA, type MyComponentProps } from '@pimcore/studio-ui-bundle/components'
```

## Common Patterns

### Pattern: Conditional Rendering with Early Return

```typescript
export const Component = ({ data }: ComponentProps): React.JSX.Element | null => {
  if (!data) {
    return null
  }
  
  if (data.error) {
    return <ErrorState error={data.error} />
  }
  
  return (
    <div>
      {/* Main content */}
    </div>
  )
}
```

### Pattern: Loading State

```typescript
export const Component = ({ id }: ComponentProps): React.JSX.Element => {
  const { data, isLoading } = useGetDataQuery({ id })
  const { styles } = useStyles()
  
  if (isLoading) {
    return <Skeleton />
  }
  
  return (
    <div className={styles.container}>
      {data && <DataDisplay data={data} />}
    </div>
  )
}
```

### Pattern: Memoized Component

For expensive renders:

```typescript
import React, { memo } from 'react'

export interface ExpensiveComponentProps {
  data: ComplexData
}

const Component = ({ data }: ExpensiveComponentProps): React.JSX.Element => {
  // Expensive rendering logic
  return <div>{/* Complex JSX */}</div>
}

export const ExpensiveComponent = memo(Component)
```

### Pattern: Event Handlers

```typescript
export const Component = ({ onSubmit }: ComponentProps): React.JSX.Element => {
  const handleClick = (): void => {
    // Handle click
  }
  
  const handleChange = (value: string): void => {
    // Handle change
  }
  
  const handleSubmit = (event: React.FormEvent): void => {
    event.preventDefault()
    onSubmit?.()
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input onChange={(e) => handleChange(e.target.value)} />
      <button onClick={handleClick}>Submit</button>
    </form>
  )
}
```

## Common Mistakes to Avoid

### ❌ Don't Create Single Component Files

```typescript
// BAD
components/
└── my-button.tsx
```

✅ **Always use folders:**
```typescript
// GOOD
components/
└── my-button/
    ├── my-button.tsx
    └── my-button.styles.tsx
```

---

### ❌ Don't Forget Return Type

```typescript
// BAD
export const Component = (props: Props) => {
  return <div>Content</div>
}
```

✅ **Always specify return type:**
```typescript
// GOOD
export const Component = (props: Props): React.JSX.Element => {
  return <div>Content</div>
}
```

---

### ❌ Don't Define Props Inline

```typescript
// BAD
export const Component = (props: {
  title: string
  onClick: () => void
}): React.JSX.Element => {
  return <div>{props.title}</div>
}
```

✅ **Define props interface separately:**
```typescript
// GOOD
export interface ComponentProps {
  title: string
  onClick: () => void
}

export const Component = (props: ComponentProps): React.JSX.Element => {
  return <div>{props.title}</div>
}
```

---

### ❌ Don't Use Inline Styles

```typescript
// BAD
export const Component = (): React.JSX.Element => {
  return (
    <div style={{ padding: '16px', background: '#fff' }}>
      Content
    </div>
  )
}
```

✅ **Use CSS-in-JS with styles file:**
```typescript
// GOOD
// component.styles.tsx
export const useStyles = createStyles(({ token, css }) => ({
  container: css`
    padding: ${token.padding}px;
    background: ${token.colorBgContainer};
  `
}))

// component.tsx
export const Component = (): React.JSX.Element => {
  const { styles } = useStyles()
  return <div className={styles.container}>Content</div>
}
```

---

### ❌ Don't Forget displayName for forwardRef

```typescript
// BAD
export const Component = forwardRef<InputRef, ComponentProps>(
  (props, ref) => {
    return <input ref={ref} />
  }
)
```

✅ **Add displayName:**
```typescript
// GOOD
export const Component = forwardRef<InputRef, ComponentProps>(
  (props, ref) => {
    return <input ref={ref} />
  }
)

Component.displayName = 'Component'
```

---

### ❌ Don't Destructure Props in Parameters

```typescript
// BAD - hard to add/modify props
export const Component = ({
  prop1,
  prop2,
  prop3,
  prop4,
  prop5
}: ComponentProps): React.JSX.Element => {
  return <div>{prop1}</div>
}
```

✅ **Destructure in function body:**
```typescript
// GOOD - easier to manage
export const Component = (props: ComponentProps): React.JSX.Element => {
  const { prop1, prop2, prop3, prop4, prop5 } = props
  return <div>{prop1}</div>
}
```

**Exception:** Simple components with 1-3 props can destructure in parameters.

## Quick Reference

### File Naming
- Component: `component-name.tsx`
- Styles: `component-name.styles.tsx`
- Types: `component-name.types.ts` (if needed)
- Index: `index.ts` (for re-exports)

### Component Template
```typescript
import React from 'react'
import { useStyles } from './component-name.styles'

export interface ComponentNameProps {
  // Props
}

export const ComponentName = (props: ComponentNameProps): React.JSX.Element => {
  const { styles } = useStyles()
  
  return (
    <div className={styles.container}>
      {/* JSX */}
    </div>
  )
}
```

### Styles Template
```typescript
import { createStyles } from 'antd-style'

export const useStyles = createStyles(({ token, css }) => {
  return {
    container: css`
      /* styles */
    `
  }
})
```

## Common Mistakes

### ❌ Mistake 1: Wrong Import Paths
```typescript
// ❌ WRONG - Using antd directly
import { Button } from 'antd'

// ❌ WRONG - Using @sdk in bundle
import { Button } from '@sdk/components' // outside studio-ui-bundle

// ✅ CORRECT - Check your context!
// In studio-ui-bundle:
import { Button } from '@sdk/components'
// In bundles:
import { Button } from '@pimcore/studio-ui-bundle/components'
```

### ❌ Mistake 2: Styles in Component File
```typescript
// ❌ WRONG - Styles inline in component
const useStyles = createStyles(...)
export const MyComponent = () => { ... }

// ✅ CORRECT - Separate files
// my-component.styles.tsx
export const useStyles = createStyles(...)
// my-component.tsx
import { useStyles } from './my-component.styles'
```

### ❌ Mistake 3: Missing Return Types
```typescript
// ❌ WRONG
export const MyComponent = (props: Props) => {
  return <div>...</div>
}

// ✅ CORRECT
export const MyComponent = (props: Props): React.JSX.Element => {
  return <div>...</div>
}
```

### ❌ Mistake 4: Using Falsy Checks
```typescript
// ❌ WRONG
if (!data) return null
const name = user.name || 'Unknown'

// ✅ CORRECT
import { isNil } from 'lodash'
if (isNil(data)) return null
const name = user.name ?? 'Unknown'
```

## Next Steps

After mastering React components:
- [**pimcore-studio-ui-layout-components**](../pimcore-studio-ui-layout-components/SKILL.md) - Using layout components
- [**pimcore-studio-ui-forms-antd**](../pimcore-studio-ui-forms-antd/SKILL.md) - Building forms with FormKit
- **Testing patterns** - Component testing strategies
