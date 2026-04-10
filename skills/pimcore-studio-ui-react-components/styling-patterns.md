# CSS-in-JS Styling Patterns with antd-style

Comprehensive guide to styling patterns in Pimcore Studio UI using `antd-style`.

## Table of Contents

- [Design Tokens](#design-tokens)
- [BEM-Style Modifiers](#bem-style-modifiers)
- [Nested Selectors](#nested-selectors)
- [Style Priority Control](#style-priority-control)
- [Real-World Styling Examples](#real-world-styling-examples)

## Design Tokens

Ant Design provides many design tokens for consistent styling:

### Complete Token Reference

```typescript
import { createStyles } from 'antd-style'

export const useStyles = createStyles(({ token, css }) => {
  return {
    component: css`
      // ==================
      // COLORS
      // ==================
      
      // Primary colors
      color: ${token.colorPrimary};
      background: ${token.colorPrimaryBg};
      border-color: ${token.colorPrimaryBorder};
      
      // Success colors
      color: ${token.colorSuccess};
      background: ${token.colorSuccessBg};
      border-color: ${token.colorSuccessBorder};
      
      // Error colors
      color: ${token.colorError};
      background: ${token.colorErrorBg};
      border-color: ${token.colorErrorBorder};
      
      // Warning colors
      color: ${token.colorWarning};
      background: ${token.colorWarningBg};
      border-color: ${token.colorWarningBorder};
      
      // Info colors
      color: ${token.colorInfo};
      background: ${token.colorInfoBg};
      border-color: ${token.colorInfoBorder};
      
      // Background colors
      background: ${token.colorBgContainer};      // Container background
      background: ${token.colorBgLayout};         // Layout background
      background: ${token.colorBgElevated};       // Elevated (popover) background
      background: ${token.colorBgSpotlight};      // Spotlight background
      background: ${token.colorBgMask};           // Modal mask background
      
      // Text colors
      color: ${token.colorText};                  // Primary text
      color: ${token.colorTextSecondary};         // Secondary text
      color: ${token.colorTextTertiary};          // Tertiary text
      color: ${token.colorTextQuaternary};        // Quaternary text
      color: ${token.colorTextDisabled};          // Disabled text
      
      // Border colors
      border-color: ${token.colorBorder};         // Default border
      border-color: ${token.colorBorderSecondary}; // Secondary border
      
      // Hover/Active states
      background: ${token.colorBgTextHover};
      background: ${token.colorBgTextActive};
      border-color: ${token.colorPrimaryHover};
      border-color: ${token.colorPrimaryActive};
      
      // Disabled state
      background: ${token.colorBgContainerDisabled};
      
      // ==================
      // SPACING
      // ==================
      
      // Padding (standard: 16px)
      padding: ${token.padding}px;
      padding: ${token.paddingXXS}px;   // 4px
      padding: ${token.paddingXS}px;    // 8px
      padding: ${token.paddingSM}px;    // 12px
      padding: ${token.paddingMD}px;    // 16px (same as padding)
      padding: ${token.paddingLG}px;    // 24px
      padding: ${token.paddingXL}px;    // 32px
      
      // Margin (standard: 16px)
      margin: ${token.margin}px;
      margin: ${token.marginXXS}px;     // 4px
      margin: ${token.marginXS}px;      // 8px
      margin: ${token.marginSM}px;      // 12px
      margin: ${token.marginMD}px;      // 16px (same as margin)
      margin: ${token.marginLG}px;      // 24px
      margin: ${token.marginXL}px;      // 32px
      margin: ${token.marginXXL}px;     // 48px
      
      // ==================
      // TYPOGRAPHY
      // ==================
      
      // Font sizes
      font-size: ${token.fontSize}px;           // 14px (default)
      font-size: ${token.fontSizeSM}px;         // 12px
      font-size: ${token.fontSizeLG}px;         // 16px
      font-size: ${token.fontSizeXL}px;         // 20px
      font-size: ${token.fontSizeHeading1}px;   // 38px
      font-size: ${token.fontSizeHeading2}px;   // 30px
      font-size: ${token.fontSizeHeading3}px;   // 24px
      font-size: ${token.fontSizeHeading4}px;   // 20px
      font-size: ${token.fontSizeHeading5}px;   // 16px
      
      // Font weights
      font-weight: ${token.fontWeightStrong};   // 600
      
      // Line heights
      line-height: ${token.lineHeight};         // 1.5714
      line-height: ${token.lineHeightLG};       // 1.5
      line-height: ${token.lineHeightSM};       // 1.66
      line-height: ${token.lineHeightHeading1}; // 1.21
      line-height: ${token.lineHeightHeading2}; // 1.27
      
      // ==================
      // BORDERS & RADIUS
      // ==================
      
      // Border radius
      border-radius: ${token.borderRadius}px;   // 6px (default)
      border-radius: ${token.borderRadiusXS}px; // 2px
      border-radius: ${token.borderRadiusSM}px; // 4px
      border-radius: ${token.borderRadiusLG}px; // 8px
      
      // Border width
      border-width: ${token.lineWidth}px;       // 1px
      border-width: ${token.lineWidthBold}px;   // 2px
      
      // Border styles (full border declarations)
      border: ${token.lineWidth}px solid ${token.colorBorder};
      
      // ==================
      // SHADOWS & EFFECTS
      // ==================
      
      // Shadows
      box-shadow: ${token.boxShadow};
      box-shadow: ${token.boxShadowSecondary};
      box-shadow: ${token.boxShadowTertiary};
      
      // ==================
      // SIZING
      // ==================
      
      // Control heights
      height: ${token.controlHeight}px;         // 32px (default)
      height: ${token.controlHeightSM}px;       // 24px
      height: ${token.controlHeightLG}px;       // 40px
      height: ${token.controlHeightXS}px;       // 16px
      
      // ==================
      // Z-INDEX
      // ==================
      
      z-index: ${token.zIndexBase};             // 0
      z-index: ${token.zIndexPopupBase};        // 1000
    `
  }
})
```

### Practical Token Usage Example

```typescript
export const useStyles = createStyles(({ token, css }) => {
  return {
    button: css`
      // Use tokens for consistent styling
      padding: ${token.paddingSM}px;
      font-size: ${token.fontSize}px;
      border-radius: ${token.borderRadius}px;
      color: ${token.colorPrimary};
      background: ${token.colorBgContainer};
      border: ${token.lineWidth}px solid ${token.colorBorder};
      
      &:hover {
        background: ${token.colorBgTextHover};
        border-color: ${token.colorPrimaryHover};
      }
      
      &:active {
        background: ${token.colorBgTextActive};
      }
      
      &:disabled {
        color: ${token.colorTextDisabled};
        background: ${token.colorBgContainerDisabled};
        cursor: not-allowed;
      }
    `
  }
})
```

## BEM-Style Modifiers

Use BEM naming for component variants and states:

### Basic BEM Pattern

```typescript
export const useStyles = createStyles(({ token, css }) => {
  return {
    button: css`
      padding: 6px;
      height: 32px;
      border-radius: ${token.borderRadius}px;
      
      // Theme modifiers
      &.button--theme-primary {
        color: ${token.colorPrimary};
        background: ${token.colorPrimaryBg};
      }
      
      &.button--theme-secondary {
        color: ${token.colorTextSecondary};
        background: ${token.colorBgLayout};
      }
      
      &.button--theme-danger {
        color: ${token.colorError};
        background: ${token.colorErrorBg};
      }
      
      // Size modifiers
      &.button--size-small {
        height: 24px;
        padding: 4px;
        font-size: ${token.fontSizeSM}px;
      }
      
      &.button--size-large {
        height: 40px;
        padding: 8px;
        font-size: ${token.fontSizeLG}px;
      }
      
      // State modifiers
      &.button--disabled {
        opacity: 0.5;
        cursor: not-allowed;
      }
      
      &.button--loading {
        pointer-events: none;
      }
      
      &.button--active {
        background: ${token.colorBgTextActive};
      }
    `
  }
})
```

### Applying BEM Classes

```typescript
import cn from 'classnames'
import { useStyles } from './button.styles'

interface ButtonProps {
  theme?: 'primary' | 'secondary' | 'danger'
  size?: 'small' | 'medium' | 'large'
  disabled?: boolean
  loading?: boolean
  active?: boolean
}

export const Button = (props: ButtonProps): React.JSX.Element => {
  const { theme = 'primary', size = 'medium', disabled, loading, active } = props
  const { styles } = useStyles()
  
  const className = cn(
    styles.button,
    `button--theme-${theme}`,
    `button--size-${size}`,
    {
      'button--disabled': disabled,
      'button--loading': loading,
      'button--active': active
    }
  )
  
  return <button className={className}>Click me</button>
}
```

### Complex BEM Example

```typescript
export const useStyles = createStyles(({ token, css }) => {
  return {
    card: css`
      padding: ${token.padding}px;
      background: ${token.colorBgContainer};
      border: ${token.lineWidth}px solid ${token.colorBorder};
      border-radius: ${token.borderRadius}px;
      
      // Element modifiers
      &.card--bordered {
        border-width: ${token.lineWidthBold}px;
      }
      
      &.card--shadow {
        box-shadow: ${token.boxShadow};
      }
      
      &.card--hoverable {
        cursor: pointer;
        transition: all 0.3s;
        
        &:hover {
          box-shadow: ${token.boxShadowSecondary};
          border-color: ${token.colorPrimaryBorder};
        }
      }
      
      // Status modifiers
      &.card--status-success {
        border-color: ${token.colorSuccessBorder};
        background: ${token.colorSuccessBg};
      }
      
      &.card--status-error {
        border-color: ${token.colorErrorBorder};
        background: ${token.colorErrorBg};
      }
      
      &.card--status-warning {
        border-color: ${token.colorWarningBorder};
        background: ${token.colorWarningBg};
      }
    `
  }
})
```

## Nested Selectors

Target child elements and pseudo-selectors:

### Child Element Selectors

```typescript
export const useStyles = createStyles(({ token, css }) => {
  return {
    container: css`
      display: flex;
      padding: ${token.padding}px;
      
      // Target specific child classes
      .header {
        font-weight: ${token.fontWeightStrong};
        font-size: ${token.fontSizeLG}px;
        margin-bottom: ${token.marginSM}px;
      }
      
      .content {
        flex: 1;
        padding: ${token.paddingSM}px;
      }
      
      .footer {
        border-top: ${token.lineWidth}px solid ${token.colorBorder};
        padding-top: ${token.paddingSM}px;
        margin-top: ${token.marginSM}px;
      }
    `
  }
})
```

### Pseudo-Selectors

```typescript
export const useStyles = createStyles(({ token, css }) => {
  return {
    list: css`
      // First/last child
      &:first-child {
        margin-top: 0;
      }
      
      &:last-child {
        margin-bottom: 0;
      }
      
      // Nth child patterns
      &:nth-child(odd) {
        background: ${token.colorBgLayout};
      }
      
      &:nth-child(even) {
        background: ${token.colorBgContainer};
      }
      
      // Hover/focus/active states
      &:hover {
        background: ${token.colorBgTextHover};
      }
      
      &:focus {
        outline: ${token.lineWidth}px solid ${token.colorPrimary};
        outline-offset: 2px;
      }
      
      &:active {
        background: ${token.colorBgTextActive};
      }
      
      // Before/after pseudo-elements
      &::before {
        content: '';
        display: block;
        width: 4px;
        height: 100%;
        background: ${token.colorPrimary};
        position: absolute;
        left: 0;
      }
    `
  }
})
```

### Targeting Ant Design Components

```typescript
export const useStyles = createStyles(({ token, css }) => {
  return {
    form: css`
      // Override Ant Design input styles
      .ant-input {
        border-radius: ${token.borderRadiusSM}px;
        
        &:focus {
          border-color: ${token.colorPrimary};
          box-shadow: 0 0 0 2px ${token.colorPrimaryBg};
        }
      }
      
      // Override Ant Design button styles
      .ant-btn {
        &.ant-btn-primary {
          &:hover {
            opacity: 0.9;
          }
        }
      }
      
      // Target form items
      .ant-form-item {
        margin-bottom: ${token.marginLG}px;
      }
      
      // Target labels
      .ant-form-item-label {
        font-weight: ${token.fontWeightStrong};
      }
    `
  }
})
```

### Complex Nested Example

```typescript
export const useStyles = createStyles(({ token, css }) => {
  return {
    sidebar: css`
      background: ${token.colorBgContainer};
      
      // Navigation menu
      .nav-menu {
        .nav-item {
          padding: ${token.paddingSM}px;
          cursor: pointer;
          
          &:hover {
            background: ${token.colorBgTextHover};
          }
          
          &.nav-item--active {
            background: ${token.colorPrimaryBg};
            color: ${token.colorPrimary};
            font-weight: ${token.fontWeightStrong};
            
            &::before {
              content: '';
              position: absolute;
              left: 0;
              width: 3px;
              height: 100%;
              background: ${token.colorPrimary};
            }
          }
          
          .nav-icon {
            margin-right: ${token.marginXS}px;
            color: ${token.colorTextSecondary};
          }
        }
      }
    `
  }
})
```

## Style Priority Control

Control CSS specificity with `hashPriority`:

### Basic Priority Control

```typescript
// Default (high priority)
export const useStyles = createStyles(({ token, css }) => {
  return {
    button: css`
      background: ${token.colorPrimary};
    `
  }
})

// Low priority (easier to override)
export const useStyles = createStyles(({ token, css }) => {
  return {
    button: css`
      background: ${token.colorPrimary};
    `
  }
}, { hashPriority: 'low' })

// High priority (harder to override)
export const useStyles = createStyles(({ token, css }) => {
  return {
    button: css`
      background: ${token.colorPrimary};
    `
  }
}, { hashPriority: 'high' })
```

### When to Use Low Priority

Use `hashPriority: 'low'` when:
- Creating base/default styles that should be easily overridden
- Building component libraries
- Providing fallback styles

```typescript
// Base button styles (low priority - easy to override)
export const useBaseButtonStyles = createStyles(({ token, css }) => {
  return {
    button: css`
      padding: ${token.paddingSM}px;
      border-radius: ${token.borderRadius}px;
      cursor: pointer;
    `
  }
}, { hashPriority: 'low' })
```

### When to Use High Priority

Use `hashPriority: 'high'` when:
- Styles must not be overridden
- Critical styling requirements
- Enforcing design system rules

```typescript
// Critical styles (high priority - hard to override)
export const useCriticalStyles = createStyles(({ token, css }) => {
  return {
    container: css`
      max-width: 1200px;
      margin: 0 auto;
    `
  }
}, { hashPriority: 'high' })
```

## Real-World Styling Examples

### Example 1: Icon Button with States

```typescript
export const useStyles = createStyles(({ token, css }) => {
  return {
    iconButton: css`
      display: inline-flex;
      align-items: center;
      justify-content: center;
      width: 32px;
      height: 32px;
      padding: 0;
      border: none;
      background: transparent;
      border-radius: ${token.borderRadius}px;
      cursor: pointer;
      transition: all 0.2s;
      
      // Theme variants
      &.icon-button--theme-primary {
        color: ${token.colorPrimary};
        
        &:hover {
          background: ${token.colorPrimaryBg};
        }
      }
      
      &.icon-button--theme-secondary {
        color: ${token.colorTextSecondary};
        
        &:hover {
          background: ${token.colorBgTextHover};
        }
      }
      
      &.icon-button--theme-danger {
        color: ${token.colorError};
        
        &:hover {
          background: ${token.colorErrorBg};
        }
      }
      
      // States
      &:disabled {
        color: ${token.colorTextDisabled};
        cursor: not-allowed;
        
        &:hover {
          background: transparent;
        }
      }
      
      &:focus-visible {
        outline: 2px solid ${token.colorPrimary};
        outline-offset: 2px;
      }
    `
  }
})
```

### Example 2: Card with Status Border

```typescript
export const useStyles = createStyles(({ token, css }) => {
  return {
    statusCard: css`
      position: relative;
      padding: ${token.padding}px;
      background: ${token.colorBgContainer};
      border: ${token.lineWidth}px solid ${token.colorBorder};
      border-radius: ${token.borderRadius}px;
      
      // Left border indicator
      &::before {
        content: '';
        position: absolute;
        left: 0;
        top: 0;
        bottom: 0;
        width: 4px;
        border-radius: ${token.borderRadius}px 0 0 ${token.borderRadius}px;
      }
      
      // Status variants
      &.status-card--success::before {
        background: ${token.colorSuccess};
      }
      
      &.status-card--error::before {
        background: ${token.colorError};
      }
      
      &.status-card--warning::before {
        background: ${token.colorWarning};
      }
      
      &.status-card--info::before {
        background: ${token.colorInfo};
      }
      
      // Hover state
      &.status-card--hoverable {
        cursor: pointer;
        transition: all 0.3s;
        
        &:hover {
          box-shadow: ${token.boxShadow};
          transform: translateY(-2px);
        }
      }
    `
  }
})
```

### Example 3: Form Field with Validation

```typescript
export const useStyles = createStyles(({ token, css }) => {
  return {
    formField: css`
      margin-bottom: ${token.marginLG}px;
      
      .field-label {
        display: block;
        margin-bottom: ${token.marginXS}px;
        font-weight: ${token.fontWeightStrong};
        color: ${token.colorText};
      }
      
      .field-input {
        width: 100%;
        padding: ${token.paddingSM}px;
        border: ${token.lineWidth}px solid ${token.colorBorder};
        border-radius: ${token.borderRadius}px;
        font-size: ${token.fontSize}px;
        transition: all 0.2s;
        
        &:focus {
          border-color: ${token.colorPrimary};
          outline: none;
          box-shadow: 0 0 0 2px ${token.colorPrimaryBg};
        }
        
        &:disabled {
          background: ${token.colorBgContainerDisabled};
          color: ${token.colorTextDisabled};
          cursor: not-allowed;
        }
      }
      
      // Validation states
      &.form-field--error {
        .field-input {
          border-color: ${token.colorError};
          
          &:focus {
            box-shadow: 0 0 0 2px ${token.colorErrorBg};
          }
        }
        
        .field-error {
          display: block;
          margin-top: ${token.marginXS}px;
          color: ${token.colorError};
          font-size: ${token.fontSizeSM}px;
        }
      }
      
      &.form-field--success {
        .field-input {
          border-color: ${token.colorSuccess};
        }
      }
    `
  }
})
```

### Example 4: Dropdown Menu

```typescript
export const useStyles = createStyles(({ token, css }) => {
  return {
    dropdown: css`
      background: ${token.colorBgElevated};
      border-radius: ${token.borderRadius}px;
      box-shadow: ${token.boxShadowSecondary};
      padding: ${token.paddingXS}px 0;
      
      .dropdown-item {
        display: flex;
        align-items: center;
        gap: ${token.marginXS}px;
        padding: ${token.paddingSM}px ${token.padding}px;
        cursor: pointer;
        transition: all 0.2s;
        
        &:hover {
          background: ${token.colorBgTextHover};
        }
        
        &:active {
          background: ${token.colorBgTextActive};
        }
        
        &.dropdown-item--active {
          background: ${token.colorPrimaryBg};
          color: ${token.colorPrimary};
        }
        
        &.dropdown-item--disabled {
          color: ${token.colorTextDisabled};
          cursor: not-allowed;
          
          &:hover {
            background: transparent;
          }
        }
        
        &.dropdown-item--danger {
          color: ${token.colorError};
          
          &:hover {
            background: ${token.colorErrorBg};
          }
        }
        
        .item-icon {
          font-size: ${token.fontSizeLG}px;
        }
        
        .item-text {
          flex: 1;
        }
        
        .item-shortcut {
          color: ${token.colorTextSecondary};
          font-size: ${token.fontSizeSM}px;
        }
      }
      
      .dropdown-divider {
        height: ${token.lineWidth}px;
        background: ${token.colorBorder};
        margin: ${token.marginXS}px 0;
      }
    `
  }
})
```

## Best Practices

### ✅ DO: Use Design Tokens

```typescript
// GOOD - Consistent with design system
css`
  padding: ${token.paddingSM}px;
  color: ${token.colorPrimary};
`
```

### ❌ DON'T: Hardcode Values

```typescript
// BAD - Not consistent with design system
css`
  padding: 12px;
  color: #1890ff;
`
```

### ✅ DO: Use BEM for Modifiers

```typescript
// GOOD - Clear modifier naming
css`
  &.button--size-large {
    height: 40px;
  }
`
```

### ❌ DON'T: Create Unclear Class Names

```typescript
// BAD - Unclear naming
css`
  &.big {
    height: 40px;
  }
`
```

### ✅ DO: Organize Styles Logically

```typescript
// GOOD - Grouped by purpose
css`
  // Layout
  display: flex;
  padding: ${token.padding}px;
  
  // Colors
  color: ${token.colorText};
  background: ${token.colorBgContainer};
  
  // States
  &:hover {
    background: ${token.colorBgTextHover};
  }
`
```

## Back to Main Skill

👉 **Return to main component guide:** [pimcore-studio-ui-react-components SKILL.md](./SKILL.md)
