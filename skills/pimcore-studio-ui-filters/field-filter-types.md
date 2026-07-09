# Field-Filter Dynamic Types

The registry-backed type system for **per-column filter editors** inside element listings' "field filters" UI. Adding one means: *columns of this type should offer this filter editor, and its value maps to the listing query this way.*

> Read [`SKILL.md`](SKILL.md) first for the two-subsystem overview. Field filters are a specialization of the general dynamic-type pattern — if the registry/abstract/`@injectable` pattern is unfamiliar, read [`pimcore-studio-ui-dynamic-types`](../pimcore-studio-ui-dynamic-types/SKILL.md) before this. For imports, see [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).

## Table of contents

- [Anatomy of a field filter](#anatomy-of-a-field-filter)
- [1. The abstract base class](#1-the-abstract-base-class)
- [2. The definition class](#2-the-definition-class)
- [3. The editor component](#3-the-editor-component)
- [4. The registry](#4-the-registry)
- [5. Registration via DI](#5-registration-via-di)
- [6. How a value reaches the listing query](#6-how-a-value-reaches-the-listing-query)
- [Overriding a built-in filter](#overriding-a-built-in-filter)
- [SDK surface & caveats](#sdk-surface--caveats)

## Anatomy of a field filter

A field filter is **two collaborating parts** plus registration:

1. A **definition class** (`@injectable()`, extends `DynamicTypeFieldFilterAbstract`, unique `id`) — declares the backend filter type, decides when the filter applies, and returns the editor component.
2. An **editor component** — the React UI; reads/writes its value through the `useDynamicFilter()` context (not props).
3. **Registration** — bind the class to a service id, then register the instance into `DynamicTypeFieldFilterRegistry` in a module `onInit`.

At runtime, the shared dynamic-type **resolver** maps the target `FIELD_FILTER` to the `getFieldFilterComponent` callback and looks the type up in the field-filter registry by id.

## 1. The abstract base class

`DynamicTypeFieldFilterAbstract` implements `DynamicTypeAbstract` (the base every dynamic type shares). Only `id` and `getFieldFilterComponent` are truly abstract; the rest have sensible defaults you override as needed.

```typescript
interface AbstractFieldFilterDefinition {}   // empty marker; the props base

abstract class DynamicTypeFieldFilterAbstract implements DynamicTypeAbstract {
  abstract readonly id: string
  abstract getFieldFilterComponent (props: AbstractFieldFilterDefinition): ReactElement

  // --- overridable, with defaults ---
  getFieldFilterType (): string { return '' }                 // backend filter-type string
  shouldApply (filter: FieldFilter): boolean { /* reject null/''/[] */ }
  isFilterAvailable (subtype: string | null): boolean { return true }
  shouldOverrideFilterType (): boolean { return false }
  transformFilterToApiRequest (filter: FieldFilter): FieldFilter {
    return { ...filter, key: filter.meta?.filters?.key ?? filter.key, type: this.getFieldFilterType() }
  }
}
```

What each member is for:

| Member | Purpose |
|---|---|
| `id` (abstract) | Unique registry key. Namespace it in bundles (`my-bundle:currency`). Built-ins use plain ids (`'string'`, `'number'`, `'relation'`, `'dataobject.adapter'`). |
| `getFieldFilterComponent(props)` (abstract) | Returns the editor UI React element. |
| `getFieldFilterType()` | The backend/frontend filter-type string written onto the API request by `transformFilterToApiRequest`. Built-ins return a `FieldFilterFrontendType` enum value (internal). |
| `shouldApply(filter)` | Whether the current value is "non-empty enough" to send. Default rejects `null`/`''`/empty array. Override for structured values (e.g. a range with `from`/`to`). |
| `isFilterAvailable(subtype)` | Whether this filter should be offered for a given column subtype. Default `true`. |
| `shouldOverrideFilterType()` | `true` for *adapter* filters that delegate to another type's filter (the data-object adapter, object bricks, classification store). Default `false` — you rarely change this. |
| `transformFilterToApiRequest(filter)` | Maps the in-UI `FieldFilter` to the API shape (rewrites `key` from `meta.filters.key`, sets `type`). Override only for non-standard request mapping. |

`AbstractFieldFilterDefinition` is an empty marker interface. Concrete components extend it (`interface MyProps extends AbstractFieldFilterDefinition {}`), but in practice the component takes no meaningful props and reads state from context.

## 2. The definition class

The minimal custom filter overrides `id`, `getFieldFilterType`, and `getFieldFilterComponent`. Override `shouldApply` when the value isn't a plain scalar.

```tsx
import { injectable } from 'inversify'
import { DynamicTypeFieldFilterAbstract } from '@pimcore/studio-ui-bundle/modules/element'
import type { AbstractFieldFilterDefinition } from '@pimcore/studio-ui-bundle/modules/element'
import React, { type ReactElement } from 'react'
import { CurrencyFilterComponent, type CurrencyFilterProps } from './currency-filter-component'

@injectable()
export class DynamicTypeFieldFilterCurrency extends DynamicTypeFieldFilterAbstract {
  id = 'my-bundle:currency'

  getFieldFilterType (): string {
    return 'my_bundle.currency'                  // your backend filter-type string
  }

  getFieldFilterComponent (props: CurrencyFilterProps): ReactElement<CurrencyFilterProps> {
    return <CurrencyFilterComponent { ...props } />
  }

  // Value is { amount, currency }; only apply when an amount is set.
  shouldApply (filter): boolean {
    const value = filter.filterValue
    return value != null && typeof value === 'object' && value.amount != null
  }
}
```

Built-ins show two idiomatic shapes worth copying:

- **`DynamicTypeFieldFilterString`** extends an intermediate abstract (`DynamicTypeFieldFilterAbstractText`) that supplies the component, so the leaf only sets `id` + `getFieldFilterType()`. Use an intermediate abstract when several filters share one editor component.
- **`DynamicTypeFieldFilterNumber`** extends the base directly, brings its own component, and overrides `shouldApply` to accept structured `{ is, from, to }` values.

## 3. The editor component

The component reads and writes its value through `useDynamicFilter()`, which exposes `{ id, translationKey, type, data, setData, frontendType, config }`. The standard pattern keeps a local copy for editing and commits on blur:

```tsx
import { useDynamicFilter } from '@pimcore/studio-ui-bundle/components'
import { Input } from '@pimcore/studio-ui-bundle/components'
import React, { useEffect, useState } from 'react'
import type { AbstractFieldFilterDefinition } from '@pimcore/studio-ui-bundle/modules/element'

export interface CurrencyFilterProps extends AbstractFieldFilterDefinition {}

export const CurrencyFilterComponent = (): React.JSX.Element => {
  const { data, setData } = useDynamicFilter()
  const [value, setValue] = useState(data)

  useEffect(() => { setValue(data) }, [data])   // resync if external value changes

  return (
    <Input
      onBlur={ () => { setData(value) } }        // commit to the filter value
      onChange={ (e) => { setValue(e.target.value) } }
      value={ value }
    />
  )
}
```

Build the actual inputs with SDK form components — see [`pimcore-studio-ui-forms-antd`](../pimcore-studio-ui-forms-antd/SKILL.md). Never import from `antd` directly.

## 4. The registry

`DynamicTypeFieldFilterRegistry` extends the shared `DynamicTypeRegistryAbstract<DynamicTypeFieldFilterAbstract>` and adds one method:

```typescript
@injectable()
class DynamicTypeFieldFilterRegistry extends DynamicTypeRegistryAbstract<DynamicTypeFieldFilterAbstract> {
  getComponent (id: string, props: AbstractFieldFilterDefinition): ReactElement {
    return this.getDynamicType(id).getFieldFilterComponent(props)
  }
}
```

Inherited API (from the base registry):

- `registerDynamicType(type)` — add a type. **Duplicate id → logs an error via `trackError` and keeps the original** (does not throw, does not replace).
- `overrideDynamicType(type)` — replace an existing id (requires it to already exist). This is the supported way to swap out a built-in.
- `getDynamicType(id, throwException = true)`, `getDynamicTypes()`, `hasDynamicType(id)`.

## 5. Registration via DI

Two steps, exactly like every other dynamic type.

**(a) Bind the class to a service id** in your bundle's container setup:

```typescript
import { container } from '@pimcore/studio-ui-bundle'
import { DynamicTypeFieldFilterCurrency } from './dynamic-types/field-filters/dynamic-type-field-filter-currency'

container.bind('MyBundle/FieldFilter/Currency').to(DynamicTypeFieldFilterCurrency).inSingletonScope()
```

**(b) Register the instance into the registry** in a module `onInit`, fetching both by service id:

```typescript
import { container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import type { DynamicTypeFieldFilterRegistry } from '@pimcore/studio-ui-bundle/modules/element'

export const MyFilterModule = {
  onInit () {
    const registry = container.get<DynamicTypeFieldFilterRegistry>(
      serviceIds['DynamicTypes/FieldFilterRegistry']
    )
    registry.registerDynamicType(container.get('MyBundle/FieldFilter/Currency'))
  }
}
```

The registry is a shared singleton reached by the stable token `serviceIds['DynamicTypes/FieldFilterRegistry']` — the same instance core populates with the built-in filters, so your type joins the same pool. **Always `container.get(serviceId)`** rather than `new` — registering a hand-built instance bypasses DI. See [`pimcore-studio-ui-using-sdk-in-bundles`](../pimcore-studio-ui-using-sdk-in-bundles/SKILL.md) for the module/`onInit` mechanics.

## 6. How a value reaches the listing query

This ties the two subsystems together. Element listings host their filter panel with the [declarative framework](declarative-filters.md); one descriptor, `fieldFilters`, holds a `FieldFilter[]` and its `toQuery` routes each entry through the field-filter registry:

```typescript
// element-filters/definitions/field-filters-filter.ts (core)
export const fieldFiltersFilterDescriptor = defineFilter<FieldFilter[], ElementFilterQueryPart, ElementFilterContext>({
  key: 'fieldFilters',
  defaultValue: [],
  section: 'fields',
  order: 40,
  isEnabled: () => true,
  toQuery: (value, context) => value.length === 0
    ? undefined
    : { kind: 'columnFilters', filters: prepareFieldFilters(value, context) }
})
```

`prepareFieldFilters` is the value → query bridge. For each `FieldFilter`:

1. find the matching column, then resolve its filter type: `getType({ target: 'FIELD_FILTER', dynamicTypeIds: [column.type, column.frontendType] })`;
2. call `shouldApply(filter)` — skip if it returns false;
3. call `transformFilterToApiRequest(filter)` to produce the API `ColumnFilter`.

The resulting `columnFilters` are merged into the listing query args (`buildElementFilterQuery`) and sent to the API. So your two overrides do the heavy lifting: **`shouldApply` gates inclusion, `transformFilterToApiRequest` shapes the request.**

The `FieldFilter` value shape (internal type):

```typescript
interface FieldFilter {
  key: string
  type: string
  filterValue: any
  locale: string | null | undefined
  filterType?: string
  meta: { translationKey: string, [key: string]: any }
}
```

**Cross-family delegation (advanced):** data-object columns don't map directly to a field filter. Object-data dynamic types expose a `dynamicTypeFieldFilterType` property (a field-filter instance pulled from the container). Consumers unwrap it when `getType` returns an object-data type instead of a field-filter type. The `dataobject.adapter` / `dataobject.object-brick` / `dataobject.classificationstore` filters set `shouldOverrideFilterType() = true` and delegate `isFilterAvailable`/`transformFilterToApiRequest` to the wrapped field's own filter type. You only touch this when adding a new *data-object data type* — a plain custom field filter does not.

## Overriding a built-in filter

To change how an existing column filter behaves, register a type with the **same id** using `overrideDynamicType`:

```typescript
onInit () {
  const registry = container.get<DynamicTypeFieldFilterRegistry>(serviceIds['DynamicTypes/FieldFilterRegistry'])
  registry.overrideDynamicType(container.get('MyBundle/FieldFilter/MyString'))   // MyString has id = 'string'
}
```

`registerDynamicType` with a duplicate id would be rejected (logged, original kept) — `overrideDynamicType` is the intentional replace.

## SDK surface & caveats

Exported through the SDK (available to bundles via `@pimcore/studio-ui-bundle/modules/element` and `@pimcore/studio-ui-bundle/components`):

- `DynamicTypeFieldFilterAbstract`, `AbstractFieldFilterDefinition`, `DynamicTypeFieldFilterRegistry`
- The built-in editor components for text / number / date / checkbox
- `useDynamicFilter` and the `FieldFilters` component

**Not on the SDK surface (internal `@Pimcore/*`) — do not import from a bundle:**

- `FieldFilterFrontendType` enum — return your own filter-type string from `getFieldFilterType()`.
- The `FieldFilter` provider type and most individual `types/*` definitions (only date / number / multiselect definitions are re-exported).

Built-in filter ids for reference (registered by core): `string`, `number`, `boolean`, `boolean-select`, `date`, `datetime`, `time`, `color`, `consent`, `id`, `multiselect`, `quantity-value`, `input-quantity-value`, `relation`, `none`, plus the delegating adapters `dataobject.adapter`, `dataobject.object-brick`, `dataobject.classificationstore`. Namespace your own ids to avoid clashing with these.
