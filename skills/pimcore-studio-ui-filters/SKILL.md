---
name: pimcore-studio-ui-filters
description: Building filters in Pimcore Studio UI - the declarative defineFilter framework for filter panels and sidebars (createFiltersStore, FiltersRenderer, useFilterQuery) and custom field-filter dynamic types for per-column listing filters (DynamicTypeFieldFilterAbstract, DynamicTypeFieldFilterRegistry). Use this whenever building, customizing, or extending any kind of filtering, filter panel, filter sidebar, or search/filter UI, or adding a new filter type to a listing or data view - even when the user only says "add filtering", "let users filter by X", "filter this list", or "build a filter sidebar".
metadata:
  audience: pimcore-developers
  focus: ui-extension-points
---

## What This Skill Covers

Filtering in Pimcore Studio UI splits into **two distinct subsystems**. This skill explains both and, more importantly, helps you pick the right one:

1. **The declarative filter framework** (`defineFilter` + `createFiltersStore` + `FiltersRenderer` + `useFilterQuery`). A host-agnostic toolkit for assembling a filter panel or sidebar for *any* data view — recycle bin, notifications, reports, translations, custom listings. You describe each filter as a `FilterDescriptor`; the framework handles value storage, rendering, and turning values into a backend query.

2. **Field-filter dynamic types** (`DynamicTypeFieldFilterAbstract` + `DynamicTypeFieldFilterRegistry`). The registry-backed type system for per-column filter editors inside element listings' "field filters" UI. You add a new *kind* of column filter (a currency filter, a geo filter, …) by registering a type, exactly like other Studio dynamic types.

These are related but solve different problems — see [Which subsystem do I need?](#which-subsystem-do-i-need) before writing code.

## When to Use This Skill

Use this when:

- Building a **filter panel or sidebar** for a listing, widget, or custom view
- Adding **search-term, date-range, or advanced (PQL) filtering** to a data view
- Wiring filter state into an **RTK Query** call so the list re-fetches when filters change
- Adding a **new type of per-column filter** to element listings (data objects, assets, documents)
- **Customizing or overriding** an existing built-in filter type
- Understanding how the two filter systems fit together

For the listing itself (columns, paging, the filter *decorator* that hosts this UI), see [`pimcore-studio-ui-listings`](../pimcore-studio-ui-listings/SKILL.md). Field-filter types are a specialization of the pattern in [`pimcore-studio-ui-dynamic-types`](../pimcore-studio-ui-dynamic-types/SKILL.md) — read that first if the registry/abstract pattern is unfamiliar.

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file. Both subsystems are exported through the SDK, so external bundles get the full API:

```typescript
// Declarative framework
import { defineFilter, createFiltersStore, FiltersRenderer, useFilterQuery, useDraftSync } from '@pimcore/studio-ui-bundle/components'
import type { FilterDescriptor, FilterHostAdapter, AnyFilterDescriptor } from '@pimcore/studio-ui-bundle/components'

// Field-filter dynamic types
import { DynamicTypeFieldFilterAbstract, DynamicTypeFieldFilterRegistry } from '@pimcore/studio-ui-bundle/modules/element'
import { container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
```

**One caveat:** the `FieldFilterFrontendType` enum and the `FieldFilter` value type are internal (`@Pimcore/*`) and are **not** on the SDK surface. External bundles supply their own filter-type string from `getFieldFilterType()` rather than importing the enum. Details in [`field-filter-types.md`](field-filter-types.md).

## Which subsystem do I need?

| You want to… | Use | Deep dive |
|---|---|---|
| Build a filter sidebar/panel for a view (search box, date range, checkboxes, "Apply"/"Reset") | **Declarative framework** | [`declarative-filters.md`](declarative-filters.md) |
| Turn a set of filter values into a backend query and re-fetch a list | **Declarative framework** (`useFilterQuery`) | [`declarative-filters.md`](declarative-filters.md) |
| Add a *new type* of column filter to an element listing's field filters | **Field-filter dynamic type** | [`field-filter-types.md`](field-filter-types.md) |
| Replace/override a built-in column filter editor | **Field-filter dynamic type** (`overrideDynamicType`) | [`field-filter-types.md`](field-filter-types.md) |

They compose: element listings use the **declarative framework** for the overall filter panel, and one of its descriptors (`fieldFilters`) routes each column's value through the **field-filter registry** to produce the query. So a fully custom listing filter experience can touch both — most tasks touch only one.

---

## Quick start A — a filter panel with the declarative framework

The core idea: describe each filter as a `FilterDescriptor` (via `defineFilter`), then let the framework store values, render controls, and build the query. A view keeps **two stores** — an *applied* store that drives the query and a *draft* store the sidebar edits ("Apply" copies draft → applied, which re-fetches the list).

```tsx
import { defineFilter, createFiltersStore } from '@pimcore/studio-ui-bundle/components'
import type { FilterControlProps } from '@pimcore/studio-ui-bundle/components'

// 1. Describe a filter. Generics: <TValue, TContribution, TContext>.
const searchTermFilter = defineFilter<string, MyContribution, MyContext>({
  key: 'searchTerm',
  defaultValue: '',
  section: 'search',
  order: 0,
  isEnabled: () => true,
  Control: SearchTermControl,                    // a FC<FilterControlProps<string>>
  toQuery: (value) => value !== ''               // undefined = contribute nothing
    ? { filters: [{ key: 'path', type: 'like', filterValue: value }] }
    : undefined
})

// 2. Create the two isolated stores (each call is a fresh, independent store).
export const { FiltersStoreProvider: AppliedProvider, useFiltersStore: useApplied } = createFiltersStore()
export const { FiltersStoreProvider: DraftProvider, useFiltersStore: useDraft } = createFiltersStore()
```

`Control` receives `{ value, onChange }`. Use `renderSection` instead when the control needs the `context`. The full store/renderer/adapter wiring — including how `useFilterQuery` folds contributions into your RTK Query args — is in [`declarative-filters.md`](declarative-filters.md). **Read it before building a panel**; the applied-vs-draft split and the adapter are easy to get wrong.

---

## Quick start B — a custom field-filter type

Field filters follow the standard Studio dynamic-type pattern: an `@injectable()` class with a unique `id`, extending an abstract base, bound in the container, and registered into a registry at init. Adding one means: *this column type should offer this filter editor.*

```tsx
import { injectable } from 'inversify'
import { DynamicTypeFieldFilterAbstract } from '@pimcore/studio-ui-bundle/modules/element'
import type { AbstractFieldFilterDefinition } from '@pimcore/studio-ui-bundle/modules/element'
import { useDynamicFilter } from '@pimcore/studio-ui-bundle/components' // provides { data, setData }
import React, { type ReactElement } from 'react'

@injectable()
export class DynamicTypeFieldFilterCurrency extends DynamicTypeFieldFilterAbstract {
  id = 'my-bundle:currency'                       // namespace to avoid collisions

  getFieldFilterType (): string {
    return 'my_bundle.currency'                    // the backend filter-type string
  }

  getFieldFilterComponent (props: AbstractFieldFilterDefinition): ReactElement {
    return <CurrencyFilterComponent { ...props } />
  }
}
```

The editor component reads and writes its value through `useDynamicFilter()` (not props). Registration (`container.bind(...)` + `registry.registerDynamicType(...)` in a module `onInit`), the overridable hooks (`shouldApply`, `isFilterAvailable`, `transformFilterToApiRequest`), and how a value flows into the listing query are all in [`field-filter-types.md`](field-filter-types.md).

---

## Common mistakes to avoid

- **Using one store for both editing and querying.** Editing directly mutates the live query with every keystroke and there's nothing to "Apply". Keep a *draft* store for the sidebar and an *applied* store for the query. See [`declarative-filters.md`](declarative-filters.md).
- **Calling `composeQuery` by hand in the host.** It's exported, but hosts almost always define a `FilterHostAdapter` and use `useFilterQuery`, which calls `composeQuery` internally and folds the result into your query shape.
- **Registering a manually constructed field-filter instance** (`registry.registerDynamicType(new MyType())`). This bypasses DI. Bind the class, then register `container.get(serviceId)`.
- **Reusing a built-in `id`.** `registerDynamicType` with a duplicate id logs an error and keeps the original. To *replace* a built-in filter, use `overrideDynamicType`; for a new one, namespace the id (`my-bundle:…`).
- **Importing `FieldFilterFrontendType` in a bundle.** It's internal. Return your own filter-type string from `getFieldFilterType()`.
- **Reaching into `antd` directly** for filter controls. Always use SDK components — see [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).

## Next steps

- [`declarative-filters.md`](declarative-filters.md) — the `defineFilter` framework end to end (descriptors, stores, renderer, adapter, a full recycle-bin walkthrough)
- [`field-filter-types.md`](field-filter-types.md) — registering, overriding, and querying custom field-filter types
- [`pimcore-studio-ui-listings`](../pimcore-studio-ui-listings/SKILL.md) — the listing and its filter decorator that host these filters
- [`pimcore-studio-ui-dynamic-types`](../pimcore-studio-ui-dynamic-types/SKILL.md) — the general registry/abstract pattern field filters build on
- [`pimcore-studio-ui-rtk-query-fundamentals`](../pimcore-studio-ui-rtk-query-fundamentals/SKILL.md) — the queries that filters feed into
- [`pimcore-studio-ui-forms-antd`](../pimcore-studio-ui-forms-antd/SKILL.md) — building the input controls inside filters
