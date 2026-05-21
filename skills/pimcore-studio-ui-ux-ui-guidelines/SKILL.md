---
name: pimcore-studio-ui-ux-design-guidelines
description: UX and UI design conventions for Pimcore Studio - layout, spacing, action labels, writing style, and design principles for consistent extensions
metadata:
  audience: pimcore-developers
  focus: ui-components
---

## What This Skill Covers

- Core design principles: consistency, accessibility, scalability, efficiency
- Three-panel layout system and 12-column grid
- 4px baseline spacing token system and typography
- Semantic color tokens and status indicator conventions
- Action label conventions (Add, New, Delete, Remove)
- Writing style: Title case vs Sentence case, verb-first wording, i18n requirement
- Destructive action confirmation requirement
- Loading, empty, and error state conventions
- Toolbar layout convention (left/right placement)
- Modal sizing rules and panel section themes
- Theme/dark mode readiness and styling method

## When to Use This Skill

Use this when:
- Building a new widget, editor tab, or extension panel
- Adding buttons, labels, or action controls to any UI
- Choosing spacing, color, or typography values for a component
- Writing any user-facing label, title, or action text
- Positioning content within the three-panel layout
- Reviewing a component for consistency with Studio UI conventions

---

## Design Principles

These five principles govern every UI decision in Pimcore Studio.

| Principle | Core idea |
|---|---|
| **Consistency** | Shared visual vocabulary — colors, typography, spacing, iconography, and interaction patterns are uniform across all modules. |
| **Accessibility** | Keyboard navigation, semantic HTML, ARIA attributes, and high-contrast colors (exceeding WCAG thresholds) are requirements, not extras. |
| **Scalability** | The platform must support third-party extensions, custom branding, and widget integrations without breaking existing patterns. Build modular. |
| **Efficiency** | Predictable, inclusive interfaces let users work faster. Reusable patterns let developers build faster. Efficiency is the result of the other principles. |
| **Predictability** | Repeat patterns across contexts. Once a user learns an interaction in one place, it must work identically everywhere else. |

---

## Layout

Pimcore Studio uses a **three-panel layout**:

```
┌──────────────┬─────────────────────────────┬──────────────┐
│  Left Panel  │       Content Area          │ Right Panel  │
│  (widgets)   │  12-column responsive grid  │  (widgets)   │
└──────────────┴─────────────────────────────┴──────────────┘
```

**Rules:**
- **Left and right panels**: fixed default width, flex as needed; gutters are always fixed
- **Content area**: structured on a 12-column responsive grid; treated as individual boxes aligned to the grid
- **Bottom widgets / detached tabs**: can be expanded for multitasking views

**Extension placement:**
- New widgets → **left or right panels** — never inject into the content area unless it is a dedicated editor tab
- Editor tabs / content panels → **content area** using the 12-column grid
- Bottom panels → **secondary/supporting views** (logs, previews, output), not primary actions

---

## Spacing

All spacing values come from the named token set — never use arbitrary pixel values.

| Token | Value | Typical use |
|---|---|---|
| `none` | 0px | Resets |
| `mini` | 4px | Tight internal gaps (icon–label) |
| `extra-small` | 8px | Compact component padding |
| `small` | 12px | Table cells, tag groups, form item gaps |
| `normal` | 16px | Default content padding |
| `medium` | 20px | Section padding |
| `large` | 24px | Section separation |
| `extra-large` | 32px | Major layout gaps |
| `maxi` | 48px | Panel separation, page-level breathing room |

Use `Box`, `Content`, `Space`, or `Flex` components to apply these tokens — see **[pimcore-studio-ui-layout-components](../pimcore-studio-ui-layout-components/SKILL.md)**. Global utility classes (`.p-small`, `.m-y-large`, etc.) are available but prefer component props.

---

## Typography

- **Font family**: `Lato, sans-serif` — the only permitted font; never override `fontFamily` in component styles
- **Base font size**: `12px` (`token.fontSize`)
- **Heading 1**: `35px` (`token.fontSizeHeading1`)
- Always use `token.fontSize*` / `token.fontSizeHeading*` — never hardcode font sizes

---

## Semantic Color Tokens

Use semantic tokens for all color decisions. Never pick a hex value manually.

| Role | Token | Value | Use |
|---|---|---|---|
| Primary / brand | `colorPrimary` | `#722ed1` purple | Brand actions, active states |
| Accent | `colorAccent` / `colorBorderActiveTab` | `#13C2C2` / `#00bab3` teal | Active tab border, secondary highlights |
| Success | `colorSuccess` | `#52c41a` green | Confirmations, publish success |
| Warning | `colorWarning` | `#faad14` amber | Caution states |
| Error | `colorError` | `#ff4d4f` red | Destructive actions emphasis, validation errors |
| Info | `colorInfo` | maps to primary purple | Informational alerts |

For element/icon type color coding (50+ `colorCoding*` tokens — Red, Beige, Gold, Orange, Green, Mint, Blue, Purple, Violet, Magenta families), see **[pimcore-studio-ui-icons](../pimcore-studio-ui-icons/SKILL.md)**.

---

## Status Indicators

Always pair color with an icon or text label — never rely on color alone.

| Status | Tag color | Icon |
|---|---|---|
| Published | `geekblue` | — |
| Unpublished / Draft | `gold` | `eye-off` |
| Success feedback | `colorSuccess` bg | checkmark |
| Error / destructive | `colorError` | trash or close |

Use `ElementTag` for element status — it applies these conventions automatically.

---

## Action Labels

Use the exact label that matches the semantic of the action.

| Label | Meaning | Icon | Emphasis |
|---|---|---|---|
| **Add** | Insert something that **already exists** into a container | Magnifying glass | — |
| **New** | Create something **from scratch** | Plus (icon-only contexts only) | — |
| **Delete** | **Permanently erase** from the system | Trash | Red / danger |
| **Remove** | Remove from current UI/context only — data stays in backend | Close/X | — |

> "Clear," "Remove," and "Close" intentionally share the same icon — do not add a second icon to distinguish them.

> **Delete is always the last item** in context menus (highest priority number, e.g. 800).

---

## Destructive Action Confirmation

Every action that permanently deletes or irreversibly modifies data **must** show a confirmation before execution. Three mechanisms exist — choose based on context:

| Mechanism | When to use |
|---|---|
| `modal.confirm()` via `useStudioModal()` | Standard delete confirmations — works across detached tab iframes |
| `useFormModal().confirm()` | When the user may want a "don't ask again" option |
| `<Popconfirm>` | Lightweight inline confirmation close to the trigger element |

Never execute a Delete action silently. Always include translated `title`, `content` (with item name), `okText`, and `cancelText`.

---

## Writing Style

### Title Case — Navigation and Structural Labels

Use **Title Case** for: menu items, tab titles, context menu entries, section headings, panel titles.

*Examples: Custom Reports, Output Channels, Asset Management*

### Sentence case — Actions and Interactive Elements

Use **Sentence case** for: button labels, modal titles, inline actions, any element with a verb.

*Examples: Save draft, Merge version, Export CSV*

### Verb-first, Action-oriented Wording

All actionable labels must start with a **verb**:

| ❌ Don't | ✅ Do |
|---|---|
| CSV Export | Export CSV |
| Asset Upload | Upload asset |
| Template deletion | Delete template |
| Configuration reset | Reset configuration |

**Rule**: button / menu / control → Sentence case, verb-first. Heading / title → Title case.

### i18n Requirement

**All user-facing strings must go through `t()` from `useTranslation()` (`react-i18next`).** No hardcoded English strings in components — every label, placeholder, tooltip, and error message must have a translation key.

---

## Toolbar Layout Convention

Toolbars use `justify='space-between'` by default, creating a natural left/right split.

| Side | Content |
|---|---|
| **Left** | Filters, search inputs, navigation, breadcrumbs |
| **Right** | Primary actions (Save, Apply, Confirm), secondary actions |

**Position and theme rules:**
- `position='top'`: Filter bars, search bars at the top of a panel
- `position='bottom'`: Action bars with save/cancel at the bottom (default)
- `position='content'`: Toolbars between two content sections (adds borders on both sides)
- `theme='primary'`: Main toolbars (lavender `#F5F3FA` background)
- `theme='secondary'`: Neutral/white-background toolbars (secondary emphasis)

---

## Loading, Empty, and Error States

Apply these states through the `Content` component — not custom DIVs with spinners.

| State | How to apply | When |
|---|---|---|
| **Loading** | `<Content loading>` | Data is being fetched; hides children until ready |
| **Empty** | `<Content none noneOptions={{ text: t('...') }}>` | No data to display |
| **Error toast** | `useMessage().error(t('...'))` | Non-blocking operation failure |
| **Error modal** | `useAlertModal().error(...)` | Blocking error requiring user acknowledgment |

Priority order inside `Content`: loading → none → children. See **[pimcore-studio-ui-layout-components](../pimcore-studio-ui-layout-components/SKILL.md)** for the full `Content` API.

---

## Modal Sizing

Always use a predefined modal size — never set an arbitrary width.

| Size key | Width |
|---|---|
| `M` | 530px |
| `L` | 700px |
| `ML` | 872px |
| `XL` | 1000px |
| `XXL` | max 1200px / 85vw |

---

## Panel and Section Theming

Collapsible sections use `BaseView` / `CollapseItem` with named themes. Choose by visual hierarchy:

| Theme | Visual | Use for |
|---|---|---|
| `card-with-highlight` | White bg + primary-colored top separator | Default — most sections |
| `fieldset` | Light bg + left 3px accent border | Grouped form fields |
| `border-highlight` | White bg + left 3px neutral border | Sub-sections within a fieldset |
| `default` | White bg, optional border | Plain grouping without emphasis |
| `success` / `error` | Semantic green / red bg + border | Validation results, status panels |

---

## Styling and Theme Readiness

- All component styles use `createStyles` from `antd-style` — never inline styles or raw CSS files
- Access theme tokens via the `token` argument in `createStyles(({ token, css }) => ({...}))`
- Never hardcode hex colors, font sizes, or spacing pixels — always reference tokens
- Both light (`studio-default-light`) and dark (`studio-default-dark`) themes are active; dark extends light — components built with tokens adapt automatically
- See **[pimcore-studio-ui-react-components](../pimcore-studio-ui-react-components/SKILL.md)** for the full `createStyles` pattern

---

## Accessibility Checklist

Every new component must satisfy:

- [ ] Fully operable via **keyboard** (Tab, Enter, Escape, arrow keys where applicable)
- [ ] Uses **semantic HTML** elements (`<button>`, `<nav>`, `<dialog>`, etc.)
- [ ] **ARIA roles and labels** on non-semantic interactive elements
- [ ] Color contrast meets or exceeds **WCAG AA** (aim for AAA)
- [ ] No information conveyed by **color alone** — pair with icon or label
- [ ] Focus states are **visible and styled** (do not suppress `:focus-visible`)
- [ ] All strings are **translatable** via `t()`

---

## Common Mistakes to Avoid

❌ **Using "Delete" for a UI-only removal**
```
// "Delete widget" — but the widget data still exists in the backend
```
✅ Use "Remove" for non-destructive UI-only actions: `"Remove widget"`

---

❌ **Executing a Delete without confirmation**
```tsx
// onClick={() => deleteAsset(id)}  ← no confirmation
```
✅ Always confirm first via `modal.confirm()` or `<Popconfirm>`

---

❌ **Mixing Title case and Sentence case on buttons**
```
"Save Draft"     ← Title case on an action button
"merge version"  ← No capitalisation
```
✅ Sentence case on all action buttons: `"Save draft"`, `"Merge version"`

---

❌ **Noun-first action labels**
```
"CSV Export", "Asset Upload", "Property Edit"
```
✅ Verb-first: `"Export CSV"`, `"Upload asset"`, `"Edit property"`

---

❌ **Hardcoded user-facing strings**
```tsx
<Button>Save draft</Button>
```
✅ All strings through `t()`:
```tsx
<Button>{t('asset.save-draft')}</Button>
```

---

❌ **Arbitrary spacing or color values**
```tsx
style={{ padding: '14px 18px', color: '#722ed1' }}
```
✅ Use spacing tokens via `Box`/`Content` props, and color tokens via `createStyles`:
```tsx
// In createStyles: color: token.colorPrimary
// In JSX: <Box padding={{ x: 'normal', y: 'small' }}>
```

---

❌ **Custom loading spinner instead of Content state**
```tsx
{isLoading && <Spin />}
{!isLoading && data && <List />}
```
✅ Use `Content` loading state:
```tsx
<Content loading={isLoading}>
  <List data={data} />
</Content>
```

---

❌ **Placing a widget in the content area**
```
// "Quick Stats" widget rendered inside the main editor content area
```
✅ Register as a left/right panel widget via `WidgetManager`

---

## Next Steps

- **[pimcore-studio-ui-buttons](../pimcore-studio-ui-buttons/SKILL.md)** — Button API matching the action label conventions above
- **[pimcore-studio-ui-icons](../pimcore-studio-ui-icons/SKILL.md)** — Icon usage and the colorCoding token system
- **[pimcore-studio-ui-layout-components](../pimcore-studio-ui-layout-components/SKILL.md)** — Content, Box, Flex, Space implementing the spacing tokens
- **[pimcore-studio-ui-widgets](../pimcore-studio-ui-widgets/SKILL.md)** — Registering widgets into the correct panel areas
- **[pimcore-studio-ui-react-components](../pimcore-studio-ui-react-components/SKILL.md)** — Component structure and `createStyles` CSS-in-JS pattern
- **[pimcore-studio-ui-modals](../pimcore-studio-ui-modals/SKILL.md)** — Modal component API and the `useStudioModal` / `useFormModal` hooks

