# Color Token Reference

The semantic color tokens a Studio token set commonly overrides, grouped by role. These are Ant Design theme tokens plus Pimcore custom tokens (`colorBg*`, `colorCoding*`, inverse tokens). Override **only what differs** from your `extends` base.

The authoritative, complete list is the built-in light theme:
`studio-ui-bundle/.../theme/dynamic-types/definitions/studio-default-light/dynamic-type-theme-studio-default-light.ts`
and the built-in dark theme `pimcore-dark` (Backend Power Tools).

Legend: **[T]** text (target 7:1) · **[UI]** border/icon/graphic (≥ 3:1) · **[BG]** surface · **[boot]** seeds the loading/login screen.

## Brand / primary

| Token | Role |
|---|---|
| `colorPrimary` | Brand color: button bg, primary accents. **[T/UI, boot]** — check both link-on-surface and label-on-brand |
| `colorPrimaryHover`, `colorPrimaryActive` | Interactive states of the brand. In a light theme, **darken** these |
| `colorPrimaryText`, `colorPrimaryTextHover`, `colorPrimaryTextActive` | Brand used as text **[T]** |
| `colorPrimaryBg`, `colorPrimaryBorder`, `colorPrimaryBorderHover` | Tinted brand surfaces/borders **[UI]** |
| `colorLogo` | Logo tint **[UI]** |

## Links

| Token | Role |
|---|---|
| `colorLink`, `colorLinkHover`, `colorLinkActive` | Hyperlink text **[T]** — 7:1 on the surface it sits on |

## Text

| Token | Role |
|---|---|
| `colorText` | Primary body text **[T]** |
| `colorTextTertiary`, `colorTextDescription` | Muted text **[T]** — still aim 4.5:1+, ideally 7:1 |

## Surfaces

| Token | Role |
|---|---|
| `colorBgContainer` | Default component/panel background **[BG, boot: soft figure]** |
| `colorBgLayout` | App layout background **[BG]** |
| `colorBgCanvas` | Outermost page background (html/body) **[BG, boot: page bg]** |
| `colorBgLogoOrbit` | Glow bubble behind the logo on the loading screen **[boot]** |
| `colorBgFieldset`, `colorBgToolbar`, `colorBgToolstrip`, `colorBgMainNavColumn` | Region-specific surfaces **[BG]** |

## Borders / accents (UI, ≥ 3:1)

| Token | Role |
|---|---|
| `colorAccent`, `colorAccentSecondary` | Accent color for highlights **[UI]** |
| `colorBorderActive`, `colorBorderActiveTab` | Active/selected outlines **[UI]** |
| `colorBorderFieldset`, `colorBorderContainer`, `colorBorderTertiary` | Structural borders **[UI]** |

## Navigation / tree / tabs

| Token | Role |
|---|---|
| `colorIconSidebar`, `colorIconSecondary`, `colorFillNav`, `colorBgSidebarOptions` | Sidebar/nav icons & fills |
| `colorTextTreeElement`, `colorIconTree`, `colorIconTreeUnpublished` | Element tree text/icons **[T/UI]** |
| `colorBgSelectedTab`, `colorBgUnselectedTab`, `itemActiveColor`, `itemColor` | Tab strip colors |

## Inverse (light-on-dark chips, tooltips)

`colorTextInverse`, `colorFillInverse`, `colorBorderInverse`, `colorButtonInverse` — check inverse text against the inverse fill (dark), not the light surface.

## Coding colors (icon / status palette)

`colorCodingRed*`, `colorCodingGreen*`, `colorCodingBlue*`, `colorCodingPurple*`, … plus the `colorCodingBg*/Content*/Border*` tag triples. These drive element-type icon colors and status tags. When used as text/icons on white, hold them to the text/UI targets in [`aaa-color-checklist.md`](aaa-color-checklist.md); the `Bg/Content/Border` triples are designed to be checked as a set (content text on the bg tint).

**Rebranding note:** when you're sweeping the base to replace one brand hue with another (e.g. purple → teal), **leave `colorCoding*` alone** — those hues are semantic *categories* (a purple tag means "purple category"), not brand chrome. Retinting them corrupts the type/status palette. The one grey area is a coding slot that reuses the old *brand* color (e.g. `colorCodingContentWhite` set to the brand) — decide per case.
