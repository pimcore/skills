---
name: pimcore-studio-ui-dynamic-types
description: Dynamic type system fundamentals in Pimcore Studio - extensible type pattern for polymorphic behavior
metadata:
  audience: pimcore-developers
  focus: architecture-patterns
---

## What This Skill Covers

The fundamental concept of Dynamic Types in Pimcore Studio:
- What dynamic types are and the problem they solve
- Core architecture pattern
- How to create and register custom dynamic types
- When to use dynamic types
- Available dynamic type systems

**Note**: For specific dynamic type implementations, see:
- [`grid-cells.md`](grid-cells.md) - Grid cell type examples
- [`field-definitions.md`](field-definitions.md) - Data object field definition types
- [`data-types.md`](data-types.md) - Data object data types

## When to Use This Skill

Use this when:
- Understanding the dynamic type pattern
- Creating extensible type systems
- Adding new types to existing registries
- Building polymorphic UI components
- Need type-specific behavior across contexts

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## What are Dynamic Types?

**Dynamic Types** are an architectural pattern for creating extensible, polymorphic type systems in Pimcore Studio.

### The Problem

In complex UIs, you often have many **types** of things that need **different behavior**:

- A "text" field renders differently than a "number" field
- An "image" cell displays differently than a "date" cell
- A "panel" layout arranges differently than a "tab" layout
- Different types need different components, logic, validation, formatting

**Traditional approach problems:**
- Hardcoded switch statements (`if type === 'text' ... else if type === 'number'`)
- Difficult to extend from bundles
- Tight coupling between type definitions and UI
- Code duplication across contexts

### The Solution

**Dynamic Types** provide a plugin architecture for types:

1. **Define an abstract base class** for the type system
2. **Each type extends the base** with its specific behavior
3. **Types register in a registry** at startup
4. **Runtime lookup** finds the right type implementation
5. **Bundles add new types** without modifying core

```
┌─────────────────────────────────────────┐
│  DynamicTypeAbstract (base class)       │
│  - id: string                            │
│  - getComponent(): ReactElement          │
└─────────────────────────────────────────┘
              ▲
              │ extends
              │
    ┌─────────┴──────────┬──────────┐
    │                    │          │
┌───┴────┐        ┌──────┴───┐  ┌──┴─────┐
│TextType│        │NumberType│  │DateType│
│id='text'│       │id='number'│ │id='date'│
└────────┘        └──────────┘  └────────┘
    │                  │            │
    └──────────────────┴────────────┘
              │
    ┌─────────▼──────────┐
    │   TypeRegistry     │
    │  .get('text')      │
    │  .get('number')    │
    │  .get('date')      │
    └────────────────────┘
```

## Core Architecture Pattern

### 1. Abstract Base Class

Defines the interface all types must implement:

```typescript
// Core defines the base
export abstract class DynamicTypeAbstract {
  abstract id: string
  abstract getComponent(props: any): ReactElement
}
```

### 2. Concrete Type Implementation

Each type implements the interface:

```typescript
import { injectable } from '@pimcore/studio-ui-bundle/app'
import { DynamicTypeAbstract } from './base'

@injectable()
export class TextType extends DynamicTypeAbstract {
  id = 'text'
  
  getComponent(props) {
    return <TextComponent {...props} />
  }
}
```

**Key elements:**
- `@injectable()` - Registers as DI service
- `id` - Unique identifier for type lookup
- Implements required methods from base class
- Returns type-specific React component

### 3. Registry

Manages all registered types:

```typescript
export class DynamicTypeRegistry {
  private types = new Map<string, DynamicTypeAbstract>()
  
  registerDynamicType(type: DynamicTypeAbstract): void {
    this.types.set(type.id, type)
  }
  
  getDynamicType(id: string): DynamicTypeAbstract | undefined {
    return this.types.get(id)
  }
}
```

### 4. Runtime Lookup

UI components look up types at runtime:

```typescript
// Component needs to render based on type
const SomeComponent = ({ typeId, ...props }) => {
  const registry = useInjection<TypeRegistry>(serviceIds.typeRegistry)
  const type = registry.getDynamicType(typeId)
  
  if (!type) return <DefaultComponent {...props} />
  
  // Render type-specific component
  return type.getComponent(props)
}
```

## Available Dynamic Type Systems

Pimcore Studio uses dynamic types in multiple contexts:

### 1. **Grid Cell Types**
Custom cell renderers for data grids.

**Base**: `DynamicTypeGridCellAbstract`  
**Registry**: `DynamicTypeGridCellRegistry`  
**Use**: Custom data visualization in grids

See: [`grid-cells.md`](grid-cells.md)

---

### 2. **Field Definition Types**
Data object field definitions (input, textarea, numeric, etc.).

**Base**: `DynamicTypeFieldDefinitionAbstract`  
**Registry**: `DynamicTypeFieldDefinitionRegistry`  
**Use**: Field configuration forms

See: [`field-definitions.md`](field-definitions.md)

---

### 3. **Data Object Data Types**
Data type implementations for data objects.

**Base**: `DynamicTypeDataObjectAbstract`  
**Registry**: `DynamicTypeDataObjectRegistry`  
**Use**: Field rendering and editing in data object editors

See: [`data-types.md`](data-types.md)

---

### 4. **Layout Types**
Custom layout components for editors.

**Base**: `DynamicTypeLayoutAbstract`  
**Registry**: `DynamicTypeLayoutRegistry`  
**Use**: Specialized editor layouts

---

### 5. **Metadata Types**
Custom metadata field types.

**Base**: `DynamicTypeMetadataAbstract`  
**Registry**: `DynamicTypeMetadataRegistry`  
**Use**: Asset/document metadata fields

---

### 6. **Filter Types**
Custom filter types for data filtering.

**Base**: `DynamicTypeFilterAbstract`  
**Registry**: `DynamicTypeFilterRegistry`  
**Use**: Complex filter UIs

---

## Creating a Custom Dynamic Type

### 🚨 CRITICAL: Check Import Paths First!

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

---

## File Naming and Folder Structure Conventions

### Standard Folder Hierarchy

```
[module-name]/
├── dynamic-types/
│   ├── definitions/              # Type definition classes (OR use types/)
│   │   ├── [type-name]/         # Optional: subfolder for complex types
│   │   │   ├── dynamic-type-[category]-[type-name].tsx
│   │   │   └── [component-files].tsx
│   │   └── dynamic-type-[category]-abstract.tsx  # Abstract base
│   ├── components/               # Shared React components (optional)
│   │   └── [component-name]/
│   │       ├── [component-name].tsx
│   │       └── [component-name].styles.tsx
│   ├── registry/                 # Registry class
│   │   └── dynamic-type-[category]-registry.ts
│   ├── hooks/                    # Custom React hooks (optional)
│   └── index.ts                  # Module registration
```

### Naming Conventions

#### Type Definition Files
**Pattern**: `dynamic-type-[category]-[type-name].tsx`

**Examples:**
- `dynamic-type-grid-cell-status-badge.tsx`
- `dynamic-type-field-definition-select.tsx`
- `dynamic-type-theme-studio-default-light.ts`
- `dynamic-type-filter-text.tsx`

#### Abstract Base Classes
**Pattern**: `dynamic-type-[category]-abstract.tsx`

**Examples:**
- `dynamic-type-grid-cell-abstract.tsx`
- `dynamic-type-field-definition-abstract.tsx`
- `dynamic-type-theme-abstract.ts`

#### Registry Files
**Pattern**: `dynamic-type-[category]-registry.ts`

**Examples:**
- `dynamic-type-grid-cell-registry.ts`
- `dynamic-type-field-definition-registry.tsx`
- `dynamic-type-theme-registry.ts`

#### Component Files
**Pattern**: `[component-name]-[purpose].tsx` or `[component-name].tsx`

**Examples:**
- `status-badge-cell.tsx` (grid cell component)
- `field-definition-select-form-fields.tsx` (companion form)
- `asset-preview-cell.tsx`

#### Style Files
**Pattern**: `[component-name].styles.tsx`

**Examples:**
- `status-badge-cell.styles.tsx`
- `checkbox-cell.styles.tsx`

### Real-World Example: Grid Cell Type

```
my-bundle/
└── assets/studio/js/src/
    └── modules/
        └── custom-grid-cells/
            ├── dynamic-types/
            │   ├── definitions/
            │   │   ├── status-badge/
            │   │   │   └── dynamic-type-grid-cell-status-badge.tsx
            │   │   └── progress-bar/
            │   │       └── dynamic-type-grid-cell-progress-bar.tsx
            │   ├── components/
            │   │   ├── status-badge/
            │   │   │   ├── status-badge-cell.tsx
            │   │   │   └── status-badge-cell.styles.tsx
            │   │   └── progress-bar/
            │   │       └── progress-bar-cell.tsx
            │   └── index.ts
            └── modules/
                └── grid-cell-extension.tsx
```

### Folder Organization Patterns

#### Simple Types (Single file)
```
dynamic-types/
└── definitions/
    └── dynamic-type-grid-cell-text.tsx
```

#### Complex Types (Multiple files)
```
dynamic-types/
├── definitions/
│   └── select/
│       ├── dynamic-type-field-definition-select.tsx
│       └── field-definition-select-form-fields.tsx
└── components/
    └── select-options-grid/
        └── select-options-grid.tsx
```

#### With Shared Components
```
dynamic-types/
├── definitions/
│   ├── type-a/
│   │   └── dynamic-type-grid-cell-type-a.tsx
│   └── type-b/
│       └── dynamic-type-grid-cell-type-b.tsx
└── components/              # Shared between types
    └── common-formatter/
        └── common-formatter.tsx
```

### Alternative: `types/` Instead of `definitions/`

Some modules use `types/` instead of `definitions/`:

```
dynamic-types/
├── types/                   # Instead of definitions/
│   ├── select/
│   │   └── dynamic-type-field-definition-select.tsx
│   └── image/
│       └── dynamic-type-field-definition-image.tsx
└── registry/
    └── dynamic-type-field-definition-registry.tsx
```

**Note**: Use EITHER `definitions/` OR `types/`, not both.

### Special Folders

#### Abstract Base Classes
```
dynamic-types/
└── types/
    └── _abstracts/          # Private/internal abstracts
        ├── data/
        │   └── dynamic-type-field-definition-data-abstract.tsx
        └── layout/
            └── dynamic-type-field-definition-layout-abstract.tsx
```

#### Supporting Files
```
dynamic-types/
├── definitions/
├── components/
├── hooks/                   # Custom React hooks
│   └── use-type-options.ts
└── utils/                   # Utility functions
    └── format-helpers.ts
```

### Registration File Pattern

**File**: `dynamic-types/index.ts` or `modules/[feature]-extension.tsx`

```typescript
// dynamic-types/index.ts
import { type AbstractModule, container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'

export const DynamicTypeExtension: AbstractModule = {
  onInit: () => {
    const registry = container.get<DynamicTypeGridCellRegistry>(
      serviceIds['DynamicTypes/GridCellRegistry']
    )
    
    // Register each type
    registry.registerDynamicType(
      container.get('DynamicTypes/GridCell/StatusBadge')
    )
    registry.registerDynamicType(
      container.get('DynamicTypes/GridCell/ProgressBar')
    )
  }
}
```

### Best Practices

1. **Consistent naming**: Always use `dynamic-type-[category]-[name]` pattern
2. **Group by feature**: Put related types in same folder
3. **Separate concerns**: Type definitions separate from components
4. **Shared components**: Use `components/` for reusable pieces
5. **Co-located files**: Keep type + component together in subfolder for complex types
6. **Clear hierarchy**: Use subfolders for organization, not deep nesting
7. **Index files**: Export types from `index.ts` for easier imports

### Common Patterns by Category

#### Grid Cells
```
dynamic-types/
├── definitions/
│   └── [cell-type]/
│       └── dynamic-type-grid-cell-[cell-type].tsx
└── components/
    └── [cell-type]/
        ├── [cell-type]-cell.tsx
        └── [cell-type]-cell.styles.tsx
```

#### Field Definitions
```
dynamic-types/
├── types/
│   └── [field-type]/
│       ├── dynamic-type-field-definition-[field-type].tsx
│       └── field-definition-[field-type]-form-fields.tsx
└── components/              # Shared across field types
```

#### Themes
```
dynamic-types/
└── definitions/
    ├── dynamic-type-theme-abstract.ts
    ├── [theme-name]/
    │   └── dynamic-type-theme-[theme-name].ts
    └── registry/
        └── dynamic-type-theme-registry.ts
```

---

## Creating a Dynamic Type - Step by Step

### Step 1: Define the Type Class

```typescript
import { injectable } from '@pimcore/studio-ui-bundle/app'
import { DynamicTypeGridCellAbstract } from '@pimcore/studio-ui-bundle/modules/element'
import type { AbstractGridCellDefinition } from '@pimcore/studio-ui-bundle/modules/element'
import React, { type ReactElement } from 'react'

@injectable()
export class MyCustomType extends DynamicTypeGridCellAbstract {
  // Unique identifier
  id = 'my-custom-type'
  
  // Return the component for this type
  getGridCellComponent(props: AbstractGridCellDefinition): ReactElement {
    return <MyCustomComponent {...props} />
  }
}
```

**Requirements:**
1. Extend appropriate abstract base class
2. Add `@injectable()` decorator
3. Set unique `id` property
4. Implement required methods

### Step 2: Create the Component

```typescript
import { type AbstractGridCellDefinition } from '@pimcore/studio-ui-bundle/modules/element'
import { Badge } from '@pimcore/studio-ui-bundle/components'
import React from 'react'

export const MyCustomComponent = ({ 
  getValue,
  row,
  column 
}: AbstractGridCellDefinition): React.JSX.Element => {
  const value = getValue()
  
  return (
    <div className="default-cell__content">
      <Badge color="blue">{value}</Badge>
    </div>
  )
}
```

### Step 3: Register in DI Container

Update your bundle's service configuration:

```typescript
// In your plugin or module setup
import { container } from '@pimcore/studio-ui-bundle'
import { MyCustomType } from './dynamic-types/my-custom-type'

// Register as DI service with unique ID
container.bind('DynamicTypes/GridCell/MyCustomType')
  .to(MyCustomType)
  .inSingletonScope()
```

### Step 4: Register in Registry

```typescript
import { type AbstractModule, container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import type { DynamicTypeGridCellRegistry } from '@pimcore/studio-ui-bundle/modules/element'

export const MyModule: AbstractModule = {
  onInit: () => {
    // Get the registry
    const registry = container.get<DynamicTypeGridCellRegistry>(
      serviceIds['DynamicTypes/GridCellRegistry']
    )
    
    // Register your type (get from DI container)
    registry.registerDynamicType(
      container.get('DynamicTypes/GridCell/MyCustomType')
    )
  }
}
```

## When to Use Dynamic Types

### ✅ Use Dynamic Types When:

1. **Multiple types need different behavior**
   - Each type has unique UI
   - Different rendering/editing logic per type
   - Type-specific validation/formatting

2. **Types need to be extensible**
   - Bundles should add new types
   - No core modifications required
   - Plugin architecture needed

3. **Type behavior is reusable**
   - Same type used in multiple contexts
   - Consistent behavior across UI
   - Avoid code duplication

4. **Runtime type resolution**
   - Type determined from data
   - Dynamic type switching
   - Configuration-driven types

### ❌ Don't Use Dynamic Types When:

1. **Simple conditional rendering**
   - Just 2-3 variants
   - Simple if/else sufficient
   - No extensibility needed

2. **Single-use components**
   - Type only used once
   - No reusability benefit
   - Over-engineering

3. **Fixed, unchanging types**
   - Types never extended
   - Core-only, no bundle extensions
   - Static component selection

## Benefits of Dynamic Types

### 1. **Extensibility**
Bundles add types without core changes:

```typescript
// Core provides: text, number, date
// Bundle adds: currency, percentage, status

// All registered in same registry
// All work seamlessly together
```

### 2. **Consistency**
Same type renders consistently everywhere:

```typescript
// Type registered once
registry.registerDynamicType(StatusBadgeType)

// Used automatically across:
// - Data object grids
// - Asset listings
// - Custom tables
// - Search results
```

### 3. **Type Safety**
TypeScript enforces correct implementation:

```typescript
// Must extend base class
class MyType extends DynamicTypeAbstract {
  // Must implement required properties/methods
  id: string = 'my-type'
  getComponent(props: Props): ReactElement {
    // TypeScript checks return type
  }
}
```

### 4. **Decoupling**
Type logic separated from UI:

```typescript
// UI doesn't know about specific types
const Cell = ({ typeId, ...props }) => {
  const type = registry.getDynamicType(typeId)
  return type.getComponent(props)
}

// Types added independently
registry.registerDynamicType(newType)
```

### 5. **Testability**
Types can be tested in isolation:

```typescript
describe('MyCustomType', () => {
  it('renders correctly', () => {
    const type = new MyCustomType()
    const component = type.getComponent({ value: 'test' })
    // Test component
  })
})
```

## Common Pattern: Type with Options

Types often need configuration:

```typescript
@injectable()
export class ConfigurableType extends DynamicTypeAbstract {
  id = 'configurable'
  
  getComponent(props: Props & { options?: Options }): ReactElement {
    const { options = defaultOptions } = props
    
    return <Component {...props} options={options} />
  }
}

// Usage with options
const typeInstance = registry.getDynamicType('configurable')
return typeInstance.getComponent({
  value: 'data',
  options: { format: 'detailed', color: 'blue' }
})
```

## Common Pattern: Type with Context

Access additional context in types:

```typescript
@injectable()
export class ContextAwareType extends DynamicTypeAbstract {
  id = 'context-aware'
  
  getComponent(props: Props): ReactElement {
    return <ContextAwareComponent {...props} />
  }
}

const ContextAwareComponent = ({ value, context }) => {
  // Access context data
  const { userId, permissions } = context
  
  // Conditional rendering based on context
  if (permissions.includes('admin')) {
    return <AdminView value={value} />
  }
  
  return <UserView value={value} />
}
```

## Best Practices

1. **Unique IDs**: Use namespaced IDs (`bundle:type-name`) to avoid conflicts
2. **Injectable**: Always use `@injectable()` decorator
3. **Single Responsibility**: One type = one behavior
4. **Performance**: Keep components lightweight, use memoization
5. **Error Handling**: Handle missing/invalid data gracefully
6. **Documentation**: Document type purpose and props clearly

## Common Mistakes to Avoid

❌ **Forgetting @injectable()**
```typescript
// BAD - won't be available in DI container
export class MyType extends DynamicTypeAbstract {
```

✅ **Always decorate**
```typescript
// GOOD
@injectable()
export class MyType extends DynamicTypeAbstract {
```

---

❌ **Registering manually created instances**
```typescript
// BAD - bypasses DI container
registry.registerDynamicType(new MyType())
```

✅ **Get from container**
```typescript
// GOOD - uses DI container
registry.registerDynamicType(
  container.get('DynamicTypes/MyType')
)
```

---

❌ **Non-unique IDs**
```typescript
// BAD - conflicts with other bundles
id = 'text'
```

✅ **Namespace your IDs**
```typescript
// GOOD
id = 'my-bundle:custom-text'
```

## Next Steps

- [**pimcore-studio-ui-typescript-best-practices**](../pimcore-studio-ui-typescript-best-practices/SKILL.md) - Critical type safety patterns (essential for type definitions)
- **grid-cells.md** - Grid cell type implementations (in this skill directory)
- [**pimcore-studio-ui-using-sdk-in-bundles**](../pimcore-studio-ui-using-sdk-in-bundles/SKILL.md) - Plugin and module system
- [**pimcore-studio-ui-forms-antd**](../pimcore-studio-ui-forms-antd/SKILL.md) - Form components for dynamic types
- [**pimcore-studio-ui-react-components**](../pimcore-studio-ui-react-components/SKILL.md) - Component patterns for types

## Reference

- **Example Bundle**: `/examples/dynamic-types/` in studio-example-bundle
- **Base Classes**: `@pimcore/studio-ui-bundle/modules/element`
- **Service IDs**: `serviceIds['DynamicTypes/*']`
