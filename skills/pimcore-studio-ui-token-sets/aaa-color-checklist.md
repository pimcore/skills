# AAA Color Checklist

WCAG contrast targets for a Pimcore Studio token set. **AAA is a guideline here, not a hard gate** — but every color you set should be *measured*, not eyeballed, and any pairing that misses its target should be a deliberate, documented choice.

## Targets

| Pairing | AAA | AA (fallback) | Applies to |
|---|---|---|---|
| Normal text on its background | **7:1** | 4.5:1 | `colorText*`, `colorLink*`, `colorPrimaryText*`, label/menu text |
| Large text (≥ 24px, or ≥ 19px bold) | **4.5:1** | 3:1 | headings, large numbers |
| UI components & graphical objects | — (no AAA level) | **3:1** | borders, icons, active/focus outlines, `colorAccent`, `colorBorderActive` |
| Decorative / disabled | none | none | pure decoration, disabled states |

Note: WCAG defines **no AAA level for non-text contrast** — UI elements top out at the 3:1 AA requirement. Aim text at 7:1; hold UI at ≥ 3:1.

## Pairings to check for a token set

- **Brand as text:** `colorLink` / `colorPrimaryText` on the surface (`colorBgContainer`/white) → 7:1
- **Brand as button:** the button label color on `colorPrimary` **and** on `colorPrimaryHover` **and** `colorPrimaryActive` → 7:1 for each state. AntD auto-derives the solid primary-button label (white by default), so you usually check white-on-brand; if your base theme changes that label token, check its actual value instead.
- **Body text:** `colorText` on `colorBgContainer` and on `colorBgLayout` → 7:1
- **Accents / borders:** `colorAccent`, `colorBorderActive`, active-tab border on their background → ≥ 3:1
- **Boot palette:** any text shown on `colorBgCanvas` → 7:1

### The light-theme trap

In a **light** theme the brand background must **darken** on hover/active — a lighter hover raises luminance and drops the white label below 7:1. (Dark themes lighten instead.) Always check the hover/active states, not just the resting color.

## How to measure

WCAG contrast = `(L_light + 0.05) / (L_dark + 0.05)`, where `L` is relative luminance. Use any checker, or this snippet:

```python
def _lin(c):
    c /= 255
    return c / 12.92 if c <= 0.03928 else ((c + 0.055) / 1.055) ** 2.4

def luminance(hex_color):
    h = hex_color.lstrip('#')
    r, g, b = (int(h[i:i+2], 16) for i in (0, 2, 4))
    return 0.2126 * _lin(r) + 0.7152 * _lin(g) + 0.0722 * _lin(b)

def contrast(a, b):
    la, lb = luminance(a), luminance(b)
    hi, lo = max(la, lb), min(la, lb)
    return (hi + 0.05) / (lo + 0.05)

# Text needs >= 7 (AAA) / >= 4.5 (AA); UI needs >= 3.
print(round(contrast('#00595E', '#ffffff'), 2))  # 8.11  -> AAA
```

Semi-transparent tokens (`rgba(...)`) must be measured against the color they actually composite over, not against white.

## Worked example (deep-teal light theme)

All measured against `#ffffff`:

| Token | Value | Ratio | Verdict |
|---|---|---|---|
| `colorPrimary` (link + button bg) | `#00595E` | 8.11:1 | ✅ AAA (text & white-on-brand) |
| `colorPrimaryHover` | `#004E52` | 9.50:1 | ✅ AAA (darker than base) |
| `colorPrimaryActive` | `#003C40` | 12.21:1 | ✅ AAA |
| `colorAccent` / borders | `#087881` | 5.23:1 | ✅ UI (≥ 3:1) |
| default AntD accent | `#13a8a8` | 2.92:1 | ❌ fails even the 3:1 UI floor on white |
