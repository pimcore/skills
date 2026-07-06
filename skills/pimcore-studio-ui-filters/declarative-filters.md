# Declarative Filter Framework (`defineFilter`)

The host-agnostic toolkit for building a filter panel or sidebar for any data view. You describe each filter once; the framework stores values, renders controls, and composes the backend query.

> Read [`SKILL.md`](SKILL.md) first for the two-subsystem overview and import rules. This file is the deep dive on the `defineFilter` framework. For imports, see [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md) — examples here use bundle paths (`@pimcore/studio-ui-bundle/components`).

## Table of contents

- [The five pieces](#the-five-pieces)
- [1. FilterDescriptor — describe a filter](#1-filterdescriptor--describe-a-filter)
- [2. FiltersStore — hold values (applied + draft)](#2-filtersstore--hold-values-applied--draft)
- [3. FiltersRenderer — render the controls](#3-filtersrenderer--render-the-controls)
- [4. Adapter + useFilterQuery — build the backend query](#4-adapter--usefilterquery--build-the-backend-query)
- [5. useDraftSync — keep the draft in sync](#5-usedraftsync--keep-the-draft-in-sync)
- [Full walkthrough: a recycle-bin-style filter](#full-walkthrough-a-recycle-bin-style-filter)
- [Where to look in the codebase](#where-to-look-in-the-codebase)

## The five pieces

```
defineFilter(...)  →  descriptors: AnyFilterDescriptor[]
                          │
   ┌──────────────────────┼───────────────────────────────┐
   ▼                       ▼                                ▼
FiltersStore          FiltersRenderer                  useFilterQuery(adapter, values)
 · values              · reads store.values             · runs composeQuery(descriptors, values, ctx)
 · setValue/setValues  · renders each Control /          · adapter.composeIntoQuery(...) → TQuery
 · reset                 renderSection                   · returns (baseQuery) => TQuery
```

Everything is generic over three type parameters, consistently:

- **`TValue`** — the value of *one* filter (e.g. `string`, `FieldFilter[]`).
- **`TContribution`** — the query fragment one filter contributes (returned by `toQuery`).
- **`TContext`** — a host-built object passed to every callback (available columns, a resolver, the current user, …).

## 1. FilterDescriptor — describe a filter

`defineFilter` is a typed identity function — it exists purely to infer/lock the three generics on a descriptor literal:

```typescript
export const defineFilter = <TValue, TContribution = unknown, TContext = unknown>(
  descriptor: FilterDescriptor<TValue, TContribution, TContext>
): FilterDescriptor<TValue, TContribution, TContext> => descriptor
```

The descriptor shape:

```typescript
interface FilterDescriptor<TValue, TContribution, TContext> {
  key: string                                        // unique; store key + React key
  defaultValue: TValue                               // seed + fallback when key is absent
  order?: number                                     // sort order in the panel (default 0)
  section?: string                                   // grouping; matched by FiltersRenderer's `section`
  isEnabled: (context: TContext) => boolean          // gate: only enabled filters reach composeQuery
  isVisible?: (context: TContext) => boolean         // render gate (default true)
  Control?: FC<FilterControlProps<TValue>>           // control that needs only { value, onChange }
  renderSection?: (props: FilterSectionProps<TValue, TContext>) => ReactNode  // control that needs context
  toQuery?: (value: TValue, context: TContext) => TContribution | undefined   // value → query fragment
}
```

with

```typescript
interface FilterControlProps<TValue> { value: TValue, onChange: (value: TValue) => void }
interface FilterSectionProps<TValue, TContext> extends FilterControlProps<TValue> { context: TContext }
```

Key points:

- **`Control` vs `renderSection`.** Use `Control` (a component receiving `{ value, onChange }`) for the common case. Use `renderSection` when the control needs `context` (e.g. to render options from available columns). The renderer prefers `renderSection` when both are present. A descriptor may have **neither** — that's a query-only filter whose value is edited by some other UI and only participates via `toQuery`.
- **`isEnabled` is required and gates the query.** A disabled filter contributes nothing, regardless of its value. `isVisible` gates only rendering.
- **`toQuery` returning `undefined` means "add nothing"** (e.g. empty search box). This is how you keep empty filters out of the query.

Example with a plain `Control`:

```tsx
const searchTermFilter = defineFilter<string, MyContribution, MyContext>({
  key: 'searchTerm',
  defaultValue: '',
  section: 'search',
  order: 0,
  isEnabled: () => true,
  Control: ({ value, onChange }) => (
    <Input allowClear onChange={ (e) => { onChange(e.target.value) } } value={ value } />
  ),
  toQuery: (value) => value !== ''
    ? { kind: 'columnFilters', filters: [{ key: 'path', type: 'like', filterValue: value }] }
    : undefined
})
```

Example with `renderSection` (needs a stateful control / context):

```tsx
const pqlFilter = defineFilter<string, MyContribution, MyContext>({
  key: 'pql',
  defaultValue: '',
  section: 'advanced',
  order: 10,
  isEnabled: () => true,
  renderSection: ({ value, onChange }) => (
    <PqlFilterControl onChange={ onChange } value={ value } />
  ),
  toQuery: (value) => value === '' ? undefined : { kind: 'columnFilters', filters: [{ type: 'pql', filterValue: value }] }
})
```

Collect descriptors into a typed array, which both the store provider and the renderer consume:

```typescript
export const myFilterDescriptors: ReadonlyArray<AnyFilterDescriptor<MyContribution, MyContext>> = [
  searchTermFilter,
  pqlFilter
]
```

## 2. FiltersStore — hold values (applied + draft)

`createFiltersStore()` is a **factory**: each call builds a fresh, isolated React context with its own Provider and hooks. This isolation is why you can — and should — create two stores per view.

```typescript
const createFiltersStore = (): FiltersStoreInstance

interface FiltersStoreInstance {
  FiltersStoreProvider: FC<FiltersStoreProviderProps>
  useFiltersStore: () => FiltersStore              // throws outside its provider
  useFiltersStoreOptional: () => FiltersStore | undefined
}

interface FiltersStore {
  values: FilterValues                             // Record<string, unknown>, keyed by descriptor.key
  setValue: (key: string, value: unknown) => void  // merge one key
  setValues: (values: FilterValues) => void         // shallow-merge a partial map
  reset: () => void                                 // restore defaults
}

interface FiltersStoreProviderProps {
  children: ReactNode
  descriptors: readonly FilterValueSeed[]          // FilterDescriptor is structurally compatible
  initialValues?: FilterValues                     // merged over defaults; read only at mount
}
```

Destructure and **rename** each store's Provider/hook so the roles are explicit:

```typescript
export const {
  FiltersStoreProvider: MyAppliedFiltersProvider,
  useFiltersStore: useMyAppliedFilters
} = createFiltersStore()

export const {
  FiltersStoreProvider: MyDraftFiltersProvider,
  useFiltersStore: useMyDraftFilters
} = createFiltersStore()
```

### Why two stores

- **Applied store** — the source of truth for the query. Mounted high in the view tree. Only changes when the user clicks "Apply".
- **Draft store** — the sidebar's working copy. The renderer edits this. "Apply" copies draft → applied (`appliedStore.setValues(draftStore.values)`); "Reset" calls `draftStore.reset()`.

Using a single store means every keystroke re-queries and there's no "Apply" semantics. Keep them separate.

**Notes:** the same descriptor array satisfies `descriptors` because `FilterDescriptor` has both `key` and `defaultValue` (a superset of `FilterValueSeed`). `initialValues` is read only at mount (lazy initializer) — to push newly-applied values into an already-mounted draft store, use [`useDraftSync`](#5-usedraftsync--keep-the-draft-in-sync). `setValues` **merges**, it does not replace.

## 3. FiltersRenderer — render the controls

```typescript
interface FiltersRendererProps<TContext> {
  descriptors: ReadonlyArray<AnyFilterDescriptor<unknown, TContext>>
  context: TContext
  store: FiltersStore          // normally the DRAFT store
  section?: string
}
```

The renderer, for each descriptor: filters by `isVisible?.(context) ?? true`, then by `section`, sorts ascending by `order ?? 0`, reads the value from `store.values` (falling back to `defaultValue`), and wires `onChange` to `store.setValue(key, next)`. It prefers `renderSection` over `Control`; renders nothing if neither is set.

**Section matching is inclusive:** with `section='search'`, the renderer shows descriptors whose `section` is `'search'` **and** descriptors with no `section` at all. Use sections to render one filter group per tab/area.

```tsx
<FiltersRenderer
  context={ filterContext }
  descriptors={ myFilterDescriptors }
  section='search'
  store={ draftStore }
/>
```

## 4. Adapter + useFilterQuery — build the backend query

`composeQuery(descriptors, values, context)` walks the descriptors, skips disabled/`toQuery`-less ones, and returns a flat `TContribution[]`. You rarely call it directly. Instead define a **host adapter** that also knows how to build the context and fold contributions into *your* query shape:

```typescript
interface FilterHostAdapter<TContribution, TContext, TQuery> {
  descriptors: ReadonlyArray<AnyFilterDescriptor<TContribution, TContext>>
  useBuildContext: () => TContext                                                   // a hook
  composeIntoQuery: (contributions: TContribution[], baseQuery: TQuery, context: TContext) => TQuery
}

// useFilterQuery is itself a hook (it calls useBuildContext). It returns a builder.
const useFilterQuery: <TContribution, TContext, TQuery>(
  adapter: FilterHostAdapter<TContribution, TContext, TQuery>,
  appliedValues: FilterValues
) => (baseQuery: TQuery) => TQuery
```

Connect the **applied** store (not the draft) to the query:

```tsx
const myFilterAdapter: FilterHostAdapter<MyContribution, MyContext, MyQuery> = {
  descriptors: myFilterDescriptors,
  useBuildContext: useMyFilterContext,                        // returns { columns, ... }
  composeIntoQuery: (contributions, baseQuery) => {
    const next = { ...baseQuery }
    const columnFilters = contributions.flatMap((c) => c.filters)
    if (columnFilters.length > 0) { next.columnFilters = columnFilters }
    return next
  }
}

// In the container that owns the list:
const { values: appliedValues } = useMyAppliedFilters()
const buildFilterQuery = useFilterQuery(myFilterAdapter, appliedValues)
const { columnFilters } = buildFilterQuery({})               // baseQuery = {}

const { data } = useMyListGetCollectionQuery({
  body: { filters: { page, pageSize, columnFilters } }
})
```

When `appliedValues` changes, `buildFilterQuery` produces new args and RTK Query re-fetches automatically.

## 5. useDraftSync — keep the draft in sync

```typescript
const useDraftSync = (appliedValues: FilterValues, draftStore: Pick<FiltersStore, 'setValues'>): void
```

Because a draft store reads `initialValues` only at mount, this hook pushes later applied-value changes (external reset, programmatic apply) into an already-mounted draft store so the sidebar reflects the live filters:

```tsx
const DraftSync = ({ children }) => {
  const { values } = useMyAppliedFilters()
  const draftStore = useMyDraftFilters()
  useDraftSync(values, draftStore)
  return <>{children}</>
}
```

## Full walkthrough: a recycle-bin-style filter

This is the complete cycle. (a) define stores/descriptors/adapter, (b) mount the applied provider over the view, (c) build the query, (d) mount the draft provider in the sidebar seeded from applied values, (e) render + apply/reset.

**(a) `filters.tsx` — stores, descriptors, adapter**

```tsx
import { createFiltersStore, defineFilter } from '@pimcore/studio-ui-bundle/components'
import type { AnyFilterDescriptor, FilterHostAdapter } from '@pimcore/studio-ui-bundle/components'

export const { FiltersStoreProvider: AppliedProvider, useFiltersStore: useApplied } = createFiltersStore()
export const { FiltersStoreProvider: DraftProvider,   useFiltersStore: useDraft }   = createFiltersStore()

export const useFilterContext = (): MyContext => {
  const { getType } = useDynamicTypeResolver()
  return { columns: FILTERABLE_FIELDS, getType }
}

export const descriptors: ReadonlyArray<AnyFilterDescriptor<MyContribution, MyContext>> = [searchTermFilter, fieldFiltersFilter]

export const adapter: FilterHostAdapter<MyContribution, MyContext, MyQuery> = {
  descriptors,
  useBuildContext: useFilterContext,
  composeIntoQuery: (contributions, baseQuery) => {
    const next = { ...baseQuery }
    const columnFilters = contributions.flatMap((c) => c.filters)
    if (columnFilters.length > 0) { next.columnFilters = columnFilters }
    return next
  }
}
```

**(b) mount the applied provider high in the tree**

```tsx
<AppliedProvider descriptors={ descriptors }>
  <MyContainerInner />
</AppliedProvider>
```

**(c) compose the query & fetch (inside the container)**

```tsx
import { useFilterQuery } from '@pimcore/studio-ui-bundle/components'

const { values: appliedValues } = useApplied()
const buildFilterQuery = useFilterQuery(adapter, appliedValues)
const { columnFilters } = buildFilterQuery({})

const { data } = useMyGetCollectionQuery({ body: { filters: { page, pageSize, columnFilters } } })
```

**(d) draft provider in the sidebar, seeded from applied values**

```tsx
const { values: appliedValues } = useApplied()

<DraftProvider descriptors={ descriptors } initialValues={ appliedValues }>
  <Sidebar entries={ entries } />
</DraftProvider>
```

**(e) render + apply/reset (inside the sidebar tab)**

```tsx
import { FiltersRenderer } from '@pimcore/studio-ui-bundle/components'

const draftStore   = useDraft()
const appliedStore = useApplied()
const context      = useFilterContext()

const handleApply = (): void => { appliedStore.setValues(draftStore.values) }
const handleClear = (): void => { draftStore.reset() }

<FiltersRenderer context={ context } descriptors={ descriptors } section='search' store={ draftStore } />
<Button onClick={ handleClear }>Reset</Button>
<Button onClick={ handleApply } type='primary'>Apply</Button>
```

Flow: edits mutate the **draft** store via the renderer → "Apply" copies draft → **applied** → the applied store feeds `useFilterQuery` → RTK Query re-fetches.

## Where to look in the codebase

Study these live consumers when you need a pattern to copy (paths under `studio-ui-bundle/assets/js/src/core/modules/`):

- **Recycle bin** (`recycle-bin/`) — the cleanest complete example: `filters/filters.tsx`, `recycle-bin-container.tsx`, `recycle-bin-container-inner.tsx`, `recycle-bin-sidebar/…/filter-tab/filter-tab.tsx`.
- **Reports** (`reports/reports-view/…/report-sidebar/components/`) — minimal, and a `toQuery`-only descriptor (`columns-filters/columns-filter-descriptor.ts`) whose value is edited by a separate UI.
- **Element listing general filters** (`element/listing/decorators/general-filters/`) — the most advanced: uses `useDraftSync`, `renderSection`, and routes into the field-filter registry (see [`field-filter-types.md`](field-filter-types.md)).
- Also: `notifications/filters/`, `translations/filters/`, `notes-and-events/filters/`.
