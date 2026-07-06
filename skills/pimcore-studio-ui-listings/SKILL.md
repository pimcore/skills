---
name: pimcore-studio-ui-listings
description: Building listings in Pimcore Studio UI using the ListingBuilder decorator pattern - sorting, paging, filtering, inline editing, and custom decorators
metadata:
  audience: pimcore-developers
  focus: ui-extension-points
---

## What This Skill Covers

How to build and customize listings in Pimcore Studio UI:
- **ListingBuilder** decorator-based builder pattern
- **Built-in decorators** for sorting, paging, filtering, selection, and more
- **Building and rendering** listings with `BaseListing`
- **Custom decorators** with context, data, and view layers
- **Toolbar customization** via ComponentRegistry slots
- **Connecting listings** to navigation and widgets

## When to Use This Skill

Use this when:
- Creating a new listing view for data objects or custom entities
- Customizing an existing listing (adding/removing features)
- Adding sorting, paging, filtering, or inline editing to a data view
- Building a selectable list with row actions
- Registering a listing as a navigable widget
- Creating custom decorator behavior for listings

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## ListingBuilder Pattern

The listing system uses a **decorator-based builder pattern**. Each decorator wraps the listing props to add functionality (sorting, paging, filtering, row selection, etc.). The `ListingBuilder` manages the chain of decorators and produces final props for `BaseListing`.

Retrieve the builder from the DI container, then **always `copy()` before modifying** — the instance from the container is shared and mutating it breaks every other listing.

```typescript
import { container } from '@pimcore/studio-ui-bundle/app'

const listingBuilder = container.get<ObjectListingBuilder>('DataObject/Listing/Builder')
const customBuilder = listingBuilder.copy()

// Add a decorator (lower priority = outermost wrapper, higher = closer to base)
customBuilder.addDecorator({
  name: 'myDecorator',
  decorator: MyDecorator,
  priority: 50
})

// Override an existing decorator by name
customBuilder.overrideDecorator({
  name: 'sorting',
  decorator: CustomSortingDecorator
})

// Remove a decorator entirely
customBuilder.removeDecorator('tagFilter')
```

Priority guideline: 10-30 infrastructure, 30-50 core (sort/page), 50-70 features (filter/select), 70-90 UI (actions/menus), 90+ custom overrides. Use distinct values — equal priorities have undefined order.

## Available Decorators

The listing system ships with these built-in decorators:

### SortingDecorator

Adds column sorting functionality. Clicking column headers toggles sort direction, and the sort state is sent to the API.

### PagingDecorator

Adds pagination controls (page size selector, page navigation). Manages page state and injects paging parameters into API queries.

### RowSelectionDecorator

Enables single or multi-row selection. Configure the mode via the builder config:

```typescript
// Single selection - only one row at a time
config: {
  rowSelection: {
    config: { rowSelectionMode: 'single' }
  }
}

// Multiple selection - checkboxes, select many rows
config: {
  rowSelection: {
    config: { rowSelectionMode: 'multiple' }
  }
}
```

### InlineEditDecorator

Enables inline cell editing. Cells become editable on interaction, and changes are tracked for batch saving.

### GeneralFiltersDecorator

Registered as `generalFilters`. Adds the search + filter sidebar for the listing — text search, PQL, and per-column **field filters**.

It is built on the **declarative filter framework** (`defineFilter` / `createFiltersStore` / `useFilterQuery`): each filter is a descriptor, an *applied* store drives the query while a *draft* store backs the sidebar, and the decorator's data layer folds the composed filters into the listing query args. To add your own filter to this sidebar, or register a new per-column filter type (e.g. a currency or geo filter), see [`pimcore-studio-ui-filters`](../pimcore-studio-ui-filters/SKILL.md) — don't hand-roll it.

### ActionColumnDecorator

Appends an actions column with configurable action buttons per row (edit, delete, open, etc.).

### ContextMenuDecorator

Adds right-click context menus on rows. Context menu items can be registered and configured per listing.

### ColumnConfigurationDecorator

Adds column visibility toggling. Users can show/hide columns via a configuration popover.

### TagFilterDecorator

Adds tag-based filtering. Users can filter listing rows by assigned tags.

### DynamicTypeDecorator

Enables dynamic type cell rendering. Cells render differently based on the data type of each field (text, number, date, select, etc.).

## Building and Rendering a Listing

Call `build()` on the builder to produce props, then spread them onto `BaseListing`. The whole tree must be wrapped in `DataObjectProvider` (see next section). Pass a `config` object to `build()` to configure individual decorators at build time.

```typescript
import React from 'react'
import { container } from '@pimcore/studio-ui-bundle/app'
import { BaseListing, DataObjectProvider } from '@pimcore/studio-ui-bundle/modules/data-object'

export const MyListing = (): React.JSX.Element => {
  const listingBuilder = container.get<ObjectListingBuilder>('DataObject/Listing/Builder')
  const customBuilder = listingBuilder.copy()

  // Customize the decorator chain
  customBuilder.addDecorator({
    name: 'statusHighlight',
    decorator: StatusHighlightDecorator,
    priority: 90
  })
  customBuilder.removeDecorator('tagFilter')

  return (
    <DataObjectProvider id={1}>
      <BaseListing {
        ...customBuilder.build({
          props: { ...listingDefaultProps },
          config: {
            // Disable a decorator without removing it from the chain
            inlineEdit: { enabled: false },
            // Configure a decorator
            rowSelection: {
              config: { rowSelectionMode: 'multiple' }
            }
          }
        })
      } />
    </DataObjectProvider>
  )
}
```

**`removeDecorator` vs `config: { enabled: false }`:**
- `removeDecorator`: permanently removes the decorator from the builder.
- `config: { enabled: false }`: leaves the decorator in the chain but skips it for this build call — toggleable per render.

## The DataObjectProvider Wrapper

Every data object listing **must** be wrapped in a `DataObjectProvider`. This provider supplies the element context (ID, type, permissions) that the listing and its decorators depend on. The `id` prop specifies the parent folder ID for the listing.

```typescript
import { DataObjectProvider } from '@pimcore/studio-ui-bundle/modules/data-object'

<DataObjectProvider id={1}>
  <BaseListing { ...props } />
</DataObjectProvider>
```

Without `DataObjectProvider`, decorators that depend on element context (permissions, tag filters, etc.) will fail silently or throw errors.

## Toolbar Customization

Listings expose toolbar slots where extra components (buttons, filters, etc.) can be registered via the `ComponentRegistry`. Slot naming convention:

```
{listingName}.toolbar.left     - Left-aligned
{listingName}.toolbar.center   - Center
{listingName}.toolbar.right    - Right-aligned
```

```typescript
import { container, serviceIds } from '@pimcore/studio-ui-bundle/app'
import { type ComponentRegistry } from '@pimcore/studio-ui-bundle/modules/app'

const componentRegistry = container.get<ComponentRegistry>(serviceIds.componentRegistry)

componentRegistry.registerToSlot('carsListing.toolbar.right', {
  name: 'exportButton',
  component: ExportButton
})
```

The registered `component` is a regular React component — use standard hooks (e.g. `useTranslation`) and Studio UI components (e.g. `IconTextButton`) inside it.

## Connecting Listings to Navigation and Widgets

To make a listing reachable from the main navigation, register it as a widget and add a navigation entry in your module's `onInit`. See [`pimcore-studio-ui-widgets`](../pimcore-studio-ui-widgets/SKILL.md) and [`pimcore-studio-ui-navigation`](../pimcore-studio-ui-navigation/SKILL.md) for details.

```typescript
import { type AbstractModule } from '@pimcore/studio-ui-bundle'
import { container, serviceIds } from '@pimcore/studio-ui-bundle/app'
import { type MainNavRegistry } from '@pimcore/studio-ui-bundle/modules/app'
import { type WidgetRegistry } from '@pimcore/studio-ui-bundle/modules/widget-manager'

export const CarsListingModule: AbstractModule = {
  onInit: (): void => {
    const widgetRegistry = container.get<WidgetRegistry>(serviceIds.widgetManager)
    const mainNavRegistry = container.get<MainNavRegistry>(serviceIds.mainNavRegistry)

    widgetRegistry.registerWidget({
      name: 'cars-listing',
      component: CarsListing
    })

    mainNavRegistry.registerMainNavItem({
      path: 'Tools/Cars Listing',
      label: 'cars-listing.navigation.title',
      widgetConfig: {
        name: 'Cars Listing',
        id: 'cars-listing',
        component: 'cars-listing',
        config: {
          translationKey: 'cars-listing.navigation.title',
          icon: { type: 'name', value: 'car' }
        }
      }
    })
  }
}
```

## Custom Decorator Pattern

A decorator is a function `(props, config?) => props` — it receives the composed listing props and returns modified props. Override only the fields you need and pass the rest through via `...baseProps`. The overridable fields fall into three layers:

| Field | Layer | What it controls |
|---|---|---|
| `ContextComponent` | Context | React provider wrapping the listing; shares state across the other layers |
| `useDataQueryHelper` | Data | The backend query — wrap its `getArgs()` to add filters/sort/paging |
| `useGridOptions` | View | Grid/table UI — column definitions, cell renderers, `getGridProps()` |
| `useSidebarOptions` | View | Sidebar tabs/entries (this is where the built-in filter tab lives) |
| `ViewComponent` / `ConfigurationComponent` / `DataComponent` | — | Whole container components; rarely overridden |

There is **no `ToolbarComponent` and no `queryArg`** in the contract — a toolbar is rendered from a `toolbarSlotName` slot (see [Toolbar Customization](#toolbar-customization)), and query args live under `body.filters`. Every hook layer follows the same pattern: grab the base hook, call it, override one method, spread the rest through.

Every layer wraps the same way: grab the base hook/component, override the one piece you need, spread the rest through. The data layer is the most common — wrap `useDataQueryHelper` and override `getArgs()`. Filters, sort, and paging all live under `body.filters` (`columnFilters` as an array of Pimcore's internal `ColumnFilter` type, `sortFilter`, and `page`/`pageSize`):

```typescript
import { type AbstractDecorator, type AbstractDecoratorProps } from '@pimcore/studio-ui-bundle/modules/element'

const withArchivedScope = (useBase: AbstractDecoratorProps['useDataQueryHelper']): AbstractDecoratorProps['useDataQueryHelper'] => {
  return () => {
    const { getArgs: baseGetArgs, ...rest } = useBase()
    const getArgs: typeof baseGetArgs = () => {
      const args = baseGetArgs()
      return {
        ...args,
        body: {
          ...args.body,
          filters: { ...args.body.filters, columnFilters: [...(args.body.filters?.columnFilters ?? []), /* your ColumnFilter */] }
        }
      }
    }
    return { ...rest, getArgs }
  }
}

export const ArchivedScopeDecorator: AbstractDecorator = (props) => {
  const { useDataQueryHelper, ...baseProps } = props
  return { ...baseProps, useDataQueryHelper: withArchivedScope(useDataQueryHelper) }
}

const customBuilder = listingBuilder.copy()
customBuilder.addDecorator({ name: 'archivedScope', decorator: ArchivedScopeDecorator, priority: 60 })
```

The other layers follow the same shape: wrap `ContextComponent` (an `ElementType`) with a React provider to share state, `useGridOptions` to reshape columns / `getGridProps`, or `useSidebarOptions` to add a sidebar tab.

> **For user-facing filters, don't assemble `columnFilters` by hand.** The built-in `generalFilters` decorator already runs the declarative filter framework in this same `useDataQueryHelper` layer — reuse it (see [`pimcore-studio-ui-filters`](../pimcore-studio-ui-filters/SKILL.md)). Hand-wrapping the data layer is for fixed, non-user constraints (scoping a view) or custom sort/paging.

## Common Mistakes

### Forgetting the DataObjectProvider Wrapper

```typescript
// WRONG - Missing DataObjectProvider
export const BrokenListing = (): React.JSX.Element => {
  return (
    <BaseListing { ...listingBuilder.build({ props: { ...listingDefaultProps } }) } />
  )
}

// CORRECT - Always wrap with DataObjectProvider
export const WorkingListing = (): React.JSX.Element => {
  return (
    <DataObjectProvider id={1}>
      <BaseListing { ...listingBuilder.build({ props: { ...listingDefaultProps } }) } />
    </DataObjectProvider>
  )
}
```

### Using the Wrong Builder Service ID

```typescript
// WRONG - incorrect service ID
const listingBuilder = container.get<ObjectListingBuilder>('Listing/Builder')

// CORRECT - use the full service ID
const listingBuilder = container.get<ObjectListingBuilder>('DataObject/Listing/Builder')
```

### Not Copying the Builder Before Modifying

```typescript
// WRONG - mutates the shared global builder, breaks other listings
const listingBuilder = container.get<ObjectListingBuilder>('DataObject/Listing/Builder')
listingBuilder.removeDecorator('tagFilter')

// CORRECT - copy first
const customBuilder = listingBuilder.copy()
customBuilder.removeDecorator('tagFilter')
```

### Decorator Priority Conflicts

```typescript
// WRONG - same priority causes undefined order
customBuilder.addDecorator({ name: 'decoratorA', decorator: DecA, priority: 50 })
customBuilder.addDecorator({ name: 'decoratorB', decorator: DecB, priority: 50 })

// CORRECT - distinct priorities for deterministic order
customBuilder.addDecorator({ name: 'decoratorA', decorator: DecA, priority: 50 })
customBuilder.addDecorator({ name: 'decoratorB', decorator: DecB, priority: 55 })
```

### Overriding a Non-Existent Decorator

```typescript
// WRONG - decorator name does not match any built-in
customBuilder.overrideDecorator({
  name: 'sort',  // Wrong name! The built-in is 'sorting'
  decorator: CustomSortingDecorator
})

// CORRECT - use the exact built-in decorator name
customBuilder.overrideDecorator({
  name: 'sorting',
  decorator: CustomSortingDecorator
})
```

### Spreading Props Incorrectly

```typescript
// WRONG - missing spread operator
<BaseListing
  listingBuilder.build({ props: { ...listingDefaultProps } })
/>

// CORRECT - spread the build result
<BaseListing {
  ...listingBuilder.build({
    props: { ...listingDefaultProps }
  })
} />
```

## Next Steps

- [**pimcore-studio-ui-filters**](../pimcore-studio-ui-filters/SKILL.md) - The filter framework behind `generalFilters`, plus custom per-column filter types
- [**pimcore-studio-ui-tables-grids**](../pimcore-studio-ui-tables-grids/SKILL.md) - Grid component and TanStack Table for custom table views
- [**pimcore-studio-ui-context-menus**](../pimcore-studio-ui-context-menus/SKILL.md) - Context menu registration and configuration
- [**pimcore-studio-ui-widgets**](../pimcore-studio-ui-widgets/SKILL.md) - Widget system for registering listing views
- [**pimcore-studio-ui-navigation**](../pimcore-studio-ui-navigation/SKILL.md) - Navigation registration with perspective permissions
