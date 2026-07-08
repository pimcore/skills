---
name: pimcore-studio-ui-token-sets
description: Use when creating a new color theme or token set for Pimcore Studio UI (a DynamicTypeThemeAbstract subclass registered in the ThemeRegistry), including choosing an extends chain, filling AntD color tokens, seeding the boot palette, and picking colors that meet WCAG AAA contrast
metadata:
  audience: pimcore-developers
  focus: theming-accessibility
---

## What This Skill Covers

How to create a new **color token set** (a theme) in Pimcore Studio UI:
- **What a token set is** — a theme dynamic type (`DynamicTypeThemeAbstract`)
- **The recipe** — define the theme class, choose an `extends` chain, fill the color `token` map, register it
- **Boot palette** — which tokens seed the loading/login screen
- **AAA color guidance** — targets and how to verify contrast (see [`aaa-color-checklist.md`](aaa-color-checklist.md))
- **Token catalog** — what the common semantic color tokens mean (see [`token-reference.md`](token-reference.md))

**Scope:** colors only for now. The same class also carries `components` and typography/spacing tokens — those are out of scope here but slot into the same `getThemeConfig()`.

> **A token set is tokens only. Never change a component to make a theme work.** If a look can't be achieved by overriding tokens, pick a token-safe alternative — do not patch component styles/markup. See [What tokens can't do](#what-tokens-cant-do).

## When to Use This Skill

Use this when:
- Adding a new selectable theme to Studio (brand theme, high-contrast theme, dark variant)
- Overriding brand/interactive colors of the built-in light or dark theme
- You need the color choices to meet WCAG AA/AAA contrast
- Making a token set selectable in the Theme Manager (Backend Power Tools)

Not for: registering the picker UI itself (that ships with Backend Power Tools), or non-color design tokens.

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`) and are ready to follow as-is for an external bundle. For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## What a Token Set Is

A token set is a **theme dynamic type** — a subclass of `DynamicTypeThemeAbstract` registered in `DynamicTypes/ThemeRegistry`. It has:

- `id: string` — the token set id (used as the theme id everywhere)
- `extends?: string[]` — inheritance chain, base → leaf, **leaf wins**. Inherit a base so you only override deltas. `studioThemeIds` only defines `light` (`studio-default-light`); the dark base has no id constant — use the bare string `'pimcore-dark'` (shipped by Backend Power Tools).
- `getThemeConfig(): PimcoreThemeConfig` — returns `{ token?, components?, algorithm? }`, where `token` is the Ant Design semantic color map.

The abstract has **no `label` field** — it defines the token config only, not a display name. See [Label & selectability](#label--selectability) for how the theme shows up (and is named) in the picker.

Token sets are a [dynamic type](pimcore-studio-ui-dynamic-types) — same register-a-type pattern used for grid cells, field definitions, etc.

## Recipe (external bundle)

### 1. Define the theme class

```typescript
import { injectable } from '@pimcore/studio-ui-bundle/app'
import {
  DynamicTypeThemeAbstract,
  type PimcoreThemeConfig,
  studioThemeIds
} from '@pimcore/studio-ui-bundle/modules/app'

export const oceanThemeId = 'my-bundle-ocean'

@injectable()
export class DynamicTypeThemeOcean extends DynamicTypeThemeAbstract {
  id: string = oceanThemeId
  extends: string[] = [studioThemeIds.light] // inherit the light base; override deltas only

  getThemeConfig (): PimcoreThemeConfig {
    return {
      token: {
        // Brand ramp — see step 3 + the AAA checklist for the values
        colorPrimary: '#00595E',
        colorPrimaryHover: '#004E52',
        colorPrimaryActive: '#003C40',
        colorLink: '#00595E',
        // Boot palette (see step 4)
        colorBgCanvas: '#F2FBFB',
        colorBgLogoOrbit: 'rgba(0, 89, 94, 0.12)'
      }
    }
  }
}
```

For a **dark** theme, do exactly what the built-in `pimcore-dark` does: keep `extends: [studioThemeIds.light]` (it inherits the light *structure*) **and** add `algorithm: darkThemeAlgorithm` (imported from the same path), then override the dark-specific colors. This is intentional, not a typo — the dark base inherits light and flips the algorithm. To merely tweak the existing dark theme instead, use `extends: ['pimcore-dark']`.

### 2. Fill the color tokens

Override only what differs from the base. See [`token-reference.md`](token-reference.md) for what each semantic token means and which are text vs. UI vs. boot tokens.

**Global tokens are not enough — component tokens win.** The base theme also sets colors **per component** in a `components` map (e.g. `Table.controlItemBgActive`, `Tabs.itemHoverColor`/`inkBarColor`, `Tag.colorPrimaryHover`, `Colors.Neutral.Fill.colorFill`), and an AntD component token overrides the matching global `token`. So if you retint `controlItemBgActive` globally but a component hardcodes its own, that component keeps the old color — the classic symptom is **stray hovers/active states still showing the base's hue** after you thought you'd recolored everything. Add a `components` block overriding those keys too:

```typescript
getThemeConfig (): PimcoreThemeConfig {
  return {
    token: { /* global colors */ },
    components: {
      Table: { controlItemBgActive: '#7FC7C7' },        // grid selected-row highlight
      Tabs:  { inkBarColor: '#00595E', itemHoverColor: 'rgba(127,199,199,.45)' },
      Tag:   { colorPrimaryHover: '#004E52' }
    }
  }
}
```

The `extends` chain **deep-merges** (lodash `mergeWith`, recursive on plain objects), so overriding a single key like `components.Tabs.inkBarColor` keeps every other base `Tabs` setting intact — you don't restate the whole component. To find what to override, grep the base theme's `components` block for the hue you're replacing.

### 3. Verify AAA contrast

Colors are the point of this skill — **verify, don't eyeball**. Run every text/background and brand pairing through the checklist in [`aaa-color-checklist.md`](aaa-color-checklist.md) before moving on.

### 4. Wire the boot palette

These tokens seed the loading/login screen (via `resolveTokenDefaults`), so give them real values:
`colorPrimary` → brand, `colorBgCanvas` → page background, `colorBgLogoOrbit` → glow behind logo, `colorBgContainer` → neutral figure. `colorBgContainer` is commonly inherited from the base — set it only if your surface differs.

### 5. Register the token set

Three parts, mirroring every other example plugin:

```typescript
// index.ts — the plugin: bind the type, register the module
import { type IAbstractPlugin } from '@pimcore/studio-ui-bundle'
import { ThemeModule } from './modules/theme-module'
import { DynamicTypeThemeOcean } from './dynamic-types/definitions/dynamic-type-theme-ocean'

export const OceanThemePlugin: IAbstractPlugin = {
  name: 'OceanThemePlugin',
  onInit ({ container }) {
    container.bind('DynamicTypes/Theme/Ocean').to(DynamicTypeThemeOcean)
  },
  onStartup ({ moduleSystem }) {
    moduleSystem.registerModule(ThemeModule)
  }
}
```

```typescript
// modules/theme-module.ts — register into the ThemeRegistry
import { type AbstractModule, container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import { type DynamicTypeThemeRegistry } from '@pimcore/studio-ui-bundle/modules/app'

export const ThemeModule: AbstractModule = {
  onInit: (): void => {
    const themeRegistry = container.get<DynamicTypeThemeRegistry>(serviceIds['DynamicTypes/ThemeRegistry'])
    themeRegistry.registerDynamicType(container.get('DynamicTypes/Theme/Ocean'))
  }
}
```

The binding key (`'DynamicTypes/Theme/Ocean'`) is a **free-form Inversify string id** — any unique string works, but the `container.bind(...)` in the plugin and the `container.get(...)` in the module must use the *identical* string. (This is a plain DI key, unlike the framework-owned `serviceIds[...]` lookups.)

Finally **export the plugin from your module-federation entry** (`plugins.ts`) — a token set that isn't exported never loads. `plugins.ts` is the file wired as the federation `exposes` entry; add your plugin to whatever export shape it already uses (the other example plugins show the convention).

### Label & selectability

Registering the dynamic type makes the *token config* available, but the Theme Manager builds its picker from its own backend theme records (`getThemeManagerSettings().themes`), labelling each entry `theme.label ?? id`. So:

- With no matching Theme Manager record, the theme still resolves and can be applied programmatically (`store.dispatch(setThemeId(id))`) — but it shows its raw `id` rather than a friendly name.
- To appear as a properly-labelled, user-selectable entry, it needs a corresponding Theme Manager record (keyed by the same `id`) carrying the `label`. That backend/record setup is owned by Backend Power Tools and is out of scope for this skill — keep your `id` stable and matching.

## What tokens can't do

A token set may **only** supply token values — it must never require a change to a component's styles or markup to look right. Some looks are therefore off-limits, and the fix is to choose a token-safe alternative, not to patch the component.

The common trap: **a component paints one text token across all states.** The element tree label uses `colorTextTreeElement` for both selected and unselected rows (no per-state override), while only the selected *background* comes from `controlItemBgActive`. So a bold "solid dark selected row with white text" is impossible through tokens — a dark `controlItemBgActive` leaves the dark label on top (≈1.2:1, unreadable), and darkening the shared label token would wreck every unselected row.

Token-safe rule of thumb: **when a background token sits under a text token you don't control, keep that background light/dark enough that the inherited text stays legible** (≥ 4.5:1, ideally 7:1). For a bold selection, that means the strongest tint the shared label survives on — plus brand accents you *do* control (`colorBorderActive`, `colorPrimary`) — not an inverted-text bar.

Before committing a bold interaction color, check what text token renders on top of it and whether that token is state-aware. If it isn't, the bold version needs a component change — so don't do it.

## Verify

- Build the bundle assets (`npm ci && npm run build`, or `npm run check-types`) — source edits are not live until the federation assets are rebuilt.
- The token set appears in the Theme Manager (requires `backend-power-tools-bundle`) and can be selected per user / as system default.
- The `extends` chain resolves (unknown ids fall back to light).

## Common Mistakes

| Mistake | Fix |
|---|---|
| Lightening the brand on hover in a **light** theme | White button label drops below 7:1. Darken on hover/active to keep AAA. |
| Reusing a bright accent (e.g. `#13a8a8`) for borders/icons | Often below the 3:1 UI floor on white. Verify accents too. |
| Redefining the whole token map | Set `extends` and override only deltas. |
| Stray hovers/active states keep the base hue after retinting | Component tokens beat global ones — also override the base's `components.*` keys (`Table`, `Tabs`, `Tag`, `Colors.Neutral.Fill`, …). See [step 2](#2-fill-the-color-tokens). |
| Forgetting the federation export | Export the plugin from `plugins.ts` or it never registers. |
| Expecting changes live without a rebuild | Rebuild the bundle's assets; the running Studio serves pre-built federation output. |
| Leaving boot tokens empty | The loading screen flashes the default look. Set `colorPrimary`, `colorBgCanvas`, `colorBgLogoOrbit` (and `colorBgContainer` if it differs from the base). |
| Expecting the theme to show a friendly name automatically | The dynamic type has no `label`; the picker name comes from a Theme Manager record. See [Label & selectability](#label--selectability). |
| Patching a component to make a bold look work | A token set is tokens only. Pick a token-safe alternative. See [What tokens can't do](#what-tokens-cant-do). |
| Setting a dark selected/active background under a text token you don't control | Dark-on-dark. Keep the background legible for the inherited text (≥ 4.5:1), or add presence via `colorBorderActive`/`colorPrimary` instead. |
