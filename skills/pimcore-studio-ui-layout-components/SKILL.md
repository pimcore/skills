---
name: pimcore-studio-ui-layout-components
description: Fundamental layout components in Pimcore Studio UI - Content, Box, Flex, Space, ConfigLayout with real-world patterns
metadata:
  audience: pimcore-developers
  focus: ui-components
---

## What This Skill Covers

Core layout components for building UIs in Pimcore Studio:
- Content - Main container with padding and loading states
- Box - Utility wrapper for spacing (padding/margin)
- Flex - Flexible layouts with alignment and gaps
- Space - Consistent spacing between elements
- ConfigLayout - Two-column layouts with resizable panels
- **List-Detail with Tabs Pattern** - Standard pattern for entity management (see [`LIST-DETAIL-TABS-PATTERN.md`](LIST-DETAIL-TABS-PATTERN.md))
- Common layout patterns and real-world usage

## When to Use This Skill

Use this when:
- Building any UI component or page
- Need to add spacing or padding
- Creating layouts with sidebars
- Organizing form content
- Structuring editor interfaces
- Need responsive, flexible layouts
- **Building entity management UIs** (see [`LIST-DETAIL-TABS-PATTERN.md`](LIST-DETAIL-TABS-PATTERN.md) for the complete pattern)

## 🚨 Import Paths

**BEFORE writing any import, read [`CRITICAL-IMPORT-PATHS.md`](../CRITICAL-IMPORT-PATHS.md).**

All examples below use bundle imports (`@pimcore/studio-ui-bundle/*`). For core development (`@sdk/*`, `@Pimcore/*`), see the referenced file.

## Spacing Size System

All layout components use a consistent spacing system:

- `mini` - Smallest spacing (4px)
- `extra-small` - Very small spacing (8px)
- `small` - Small spacing (12px)
- `medium` - Medium spacing (16px) - default
- `large` - Large spacing (24px)
- `extra-large` - Extra large spacing (32px)

You can also use numbers for pixel-perfect spacing.

## Content Component

The `Content` component is the **primary container** for page content sections. Use it as a wrapper for main content areas.

### Basic Usage

```typescript
import { Content } from '@pimcore/studio-ui-bundle/components'

const MyPage = () => {
  return (
    <Content padded>
      <h1>My Page Title</h1>
      <p>Content goes here</p>
    </Content>
  )
}
```

### With Loading State

```typescript
import { Content } from '@pimcore/studio-ui-bundle/components'
import { useGetDataQuery } from '@pimcore/studio-ui-bundle/api/data'

const DataView = () => {
  const { data, isLoading } = useGetDataQuery()
  
  return (
    <Content loading={isLoading} padded>
      {data && <DataDisplay data={data} />}
    </Content>
  )
}
```

**Why use loading on Content:**
- Shows built-in loading skeleton
- Prevents layout shift
- Consistent loading UX across app

### Custom Padding

```typescript
// Fine-grained padding control
<Content
  padded
  padding={{
    x: 'extra-small',  // Horizontal padding
    y: 'small'          // Vertical padding
  }}
>
  {/* Content */}
</Content>
```

### With Gap Between Children

```typescript
// Automatic spacing between direct children
<Content
  padded
  gap="small"
>
  <SearchForm />
  <ResultsTable />
  <Pagination />
</Content>
```

### Empty State

```typescript
// No content, no padding
<Content none />
```

### Content Props

```typescript
interface ContentProps {
  padded?: boolean                    // Apply default padding
  padding?: {                         // Custom padding control
    x?: SpacingSize                  // Horizontal
    y?: SpacingSize                  // Vertical
    top?: SpacingSize
    bottom?: SpacingSize
    left?: SpacingSize
    right?: SpacingSize
  }
  loading?: boolean                   // Show loading skeleton
  gap?: SpacingSize                   // Gap between children
  none?: boolean                      // Empty state
  children?: ReactNode
}
```

### Real-World Example: Widget Content

```typescript
import { Content, Header } from '@pimcore/studio-ui-bundle/components'

export const ExampleWidget = () => {
  const { data, isLoading } = useWidgetDataQuery()
  
  return (
    <Content
      loading={isLoading}
      padded
      padding={{
        x: 'extra-small',
        y: 'extra-small'
      }}
      gap="small"
    >
      <Header title="Example Widget" />
      
      <div>Select a widget:</div>
      
      {data && <WidgetList items={data} />}
    </Content>
  )
}
```

## Box Component

The `Box` component is a **utility wrapper** for adding spacing (margin/padding) around elements. Use it for fine-grained spacing control.

### Basic Usage

```typescript
import { Box } from '@pimcore/studio-ui-bundle/components'

const MyComponent = () => {
  return (
    <Box padding="small">
      <SomeContent />
    </Box>
  )
}
```

### Padding Control

```typescript
// String value (all sides)
<Box padding="small">
  <Panel />
</Box>

// Object for specific sides
<Box padding={{ x: 'extra-small', y: 'small' }}>
  <Panel />
</Box>

// Individual sides
<Box padding={{ top: 'small', bottom: 'large', left: 'extra-small', right: 'extra-small' }}>
  <Panel />
</Box>
```

### Margin Control

```typescript
// Add margin to separate sections
<Box margin={{ top: 'small', bottom: 'small' }}>
  <Section />
</Box>

// Specific side margins
<Box margin={{ bottom: 'large' }}>
  <Section />
</Box>
```

### Combining Padding and Margin

```typescript
<Box
  padding="extra-small"
  margin={{ top: 'extra-small', bottom: 'extra-small' }}
>
  <Flex align="center" gap="extra-small">
    <Icon value="info" />
    <Text>Important information</Text>
  </Flex>
</Box>
```

### Box Props

```typescript
interface BoxProps {
  padding?: SpacingSize | {
    x?: SpacingSize
    y?: SpacingSize
    top?: SpacingSize
    bottom?: SpacingSize
    left?: SpacingSize
    right?: SpacingSize
  }
  margin?: SpacingSize | {
    x?: SpacingSize
    y?: SpacingSize
    top?: SpacingSize
    bottom?: SpacingSize
    left?: SpacingSize
    right?: SpacingSize
  }
  children?: ReactNode
}
```

### Real-World Example: Form Section Spacing

```typescript
import { Box, Flex, Space } from '@pimcore/studio-ui-bundle/components'

export const ScheduleToolbar = () => {
  return (
    <Box padding={{ y: 'small' }}>
      <Space>
        <Text strong>Archived Schedules</Text>
        <IconTextButton
          icon={{ value: 'trash' }}
          onClick={handleDelete}
        >
          Delete All
        </IconTextButton>
      </Space>
    </Box>
  )
}
```

### When to Use Box vs Content

**Use Content when:**
- It's a main content area/section
- Need loading state
- Container for multiple sections

**Use Box when:**
- Need spacing around a single element
- Fine-tuning margin/padding
- Wrapping for layout purposes

## Flex Component

The `Flex` component provides **flexible layout control** with CSS Flexbox. Use it for aligning, distributing, and organizing elements.

### Basic Horizontal Layout

```typescript
import { Flex } from '@pimcore/studio-ui-bundle/components'

const Toolbar = () => {
  return (
    <Flex align="center" gap="small">
      <Icon value="search" />
      <Input placeholder="Search..." />
      <Button>Search</Button>
    </Flex>
  )
}
```

### Vertical Layout

```typescript
// Stack elements vertically
<Flex vertical gap="small">
  <Title>Settings</Title>
  <SettingsPanel />
  <SaveButton />
</Flex>
```

### Alignment

```typescript
// Center items horizontally and vertically
<Flex align="center" justify="center">
  <LoadingSpinner />
</Flex>

// Align items to start/end
<Flex align="start" justify="space-between">
  <Title>Header</Title>
  <IconButton icon={{ value: 'close' }} />
</Flex>
```

### Gap Control

```typescript
// Use spacing system
<Flex gap="large">
  <Panel />
  <Panel />
  <Panel />
</Flex>

// Or specific pixel value
<Flex gap={24}>
  <Item />
  <Item />
</Flex>
```

### Nested Flex Layouts

```typescript
// Complex toolbar with multiple sections
<Flex justify="space-between" align="center">
  <Flex gap="small">
    <IconButton icon={{ value: 'arrow-left' }} />
    <Title>Document Editor</Title>
  </Flex>
  
  <Flex gap="mini">
    <Button>Cancel</Button>
    <Button type="primary">Save</Button>
  </Flex>
</Flex>
```

### Full-Height Layouts

```typescript
// Stretch to fill parent height
<Flex
  className="absolute-stretch"
  vertical
  justify="space-between"
>
  <Content padded>
    {/* Main content */}
  </Content>
  
  <Toolbar justify="flex-end">
    <Button>Save</Button>
  </Toolbar>
</Flex>
```

### Flex Props

```typescript
interface FlexProps {
  vertical?: boolean                  // Stack vertically
  align?: 'start' | 'center' | 'end' | 'stretch' | 'baseline'
  justify?: 'start' | 'center' | 'end' | 'space-between' | 'space-around' | 'space-evenly'
  gap?: SpacingSize | number         // Gap between children
  wrap?: boolean                      // Allow wrapping
  className?: string
  style?: CSSProperties
  children?: ReactNode
}
```

### Real-World Example: Email Card Layout

```typescript
import { Flex, Icon } from '@pimcore/studio-ui-bundle/components'

export const EmailCard = ({ email }) => {
  return (
    <Flex align="center" gap="extra-small">
      <Icon value="send-03" />
      <span>{email.subject}</span>
    </Flex>
  )
}
```

### Real-World Example: Form with Actions

```typescript
import { Flex, Content, Toolbar, Button } from '@pimcore/studio-ui-bundle/components'

export const AppearanceForm = () => {
  return (
    <Flex
      className="appearance-branding-form absolute-stretch"
      justify="space-between"
      vertical
    >
      <Content padded>
        <FormKit formProps={{ form }}>
          {/* Form fields */}
        </FormKit>
      </Content>
      
      <Toolbar justify="flex-end">
        <Button onClick={handleCancel}>Cancel</Button>
        <Button type="primary" onClick={handleSave}>Save</Button>
      </Toolbar>
    </Flex>
  )
}
```

## Space Component

The `Space` component adds **consistent spacing between children**. It's simpler than Flex when you just need spacing without alignment control.

### Basic Usage

```typescript
import { Space } from '@pimcore/studio-ui-bundle/components'

const ButtonGroup = () => {
  return (
    <Space size="small">
      <Button>Cancel</Button>
      <Button type="primary">Save</Button>
    </Space>
  )
}
```

### Vertical Spacing

```typescript
// Stack with spacing
<Space direction="vertical" size="large">
  <ColorPanel />
  <ImagePanel />
  <LogoPanel />
</Space>
```

### With Title and Tooltip

```typescript
import { Space, TooltipIcon } from '@pimcore/studio-ui-bundle/components'

const PanelHeader = ({ title, tooltip }) => {
  return (
    <Space size="extra-small">
      {title}
      <TooltipIcon tooltip={tooltip} />
    </Space>
  )
}
```

### In Forms

```typescript
// Switch with label
<Space
  className="pimcore-schedule-toolbar__filters__active-switch"
  size="extra-small"
>
  <Switch
    labelLeft="Show active only"
    onChange={setActiveOnly}
    value={activeOnly}
  />
</Space>
```

### Full Width

```typescript
// Fill available width
<Space
  className="w-full"
  direction="vertical"
  size="extra-small"
>
  <FormItem1 />
  <FormItem2 />
  <FormItem3 />
</Space>
```

### Space Props

```typescript
interface SpaceProps {
  size?: SpacingSize                  // Spacing size
  direction?: 'horizontal' | 'vertical'  // Layout direction (default: horizontal)
  className?: string
  children?: ReactNode
}
```

### Real-World Example: Icon Toolbar

```typescript
import { Space, IconButton } from '@pimcore/studio-ui-bundle/components'

export const ColumnsConfiguration = () => {
  return (
    <Space size="mini">
      <IconButton
        icon={{ value: 'trash' }}
        onClick={handleRemove}
        theme="secondary"
      />
      <IconButton
        icon={{ value: 'drag-handle' }}
        theme="secondary"
      />
    </Space>
  )
}
```

### When to Use Space vs Flex

**Use Space when:**
- Only need spacing between elements
- Simple horizontal or vertical stacking
- Don't need alignment control
- Want automatic wrapping

**Use Flex when:**
- Need alignment control (align, justify)
- Complex layouts
- Need stretch or specific sizing
- Want more control over layout behavior

## ConfigLayout Component

The `ConfigLayout` component creates a **two-column layout** with an optional resizable divider. Perfect for sidebar + content layouts.

---

### 🎯 **IMPORTANT: List-Detail with Tabs Pattern**

**ConfigLayout has a standard pattern for managing collections of entities** (settings, configurations, CRUD interfaces).

👉 **See [`LIST-DETAIL-TABS-PATTERN.md`](LIST-DETAIL-TABS-PATTERN.md) for the complete implementation guide!**

**This pattern provides:**
- Searchable list on the left with context menus
- Tabbed detail views on the right (multiple items open simultaneously)
- Toolbar with refresh/add buttons (left side)
- Form with FormKit + toolbar with refresh/delete/save buttons (right side)
- Dirty state tracking with `*` in tab labels
- Automatic tab management (open, close, switch)

**Perfect for:** Settings pages, entity management, configuration UIs, list-based CRUD

**Examples in codebase:** Target Groups (personalization-bundle), Email Log, Reports Editor, Field Definitions

---

### Basic Two-Column Layout

```typescript
import { ConfigLayout } from '@pimcore/studio-ui-bundle/components'

const EditorLayout = () => {
  return (
    <ConfigLayout
      leftItem={{
        children: <Sidebar />
      }}
      rightItem={{
        children: <MainContent />
      }}
    />
  )
}
```

### Resizable Sidebar

```typescript
// Allow user to resize the sidebar
// Only specify size, minSize, maxSize when you need resizeAble
<ConfigLayout
  leftItem={{
    minSize: 250,
    maxSize: 350,
    children: <DetailSidebar />
  }}
  resizeAble
  rightItem={{
    children: <DetailContent />
  }}
/>
```

### ConfigLayout Props

```typescript
interface ConfigLayoutProps {
  leftItem: {
    children: ReactNode
    minSize?: number              // Minimum width in pixels (only when resizeAble)
    maxSize?: number              // Maximum width in pixels (only when resizeAble)
    size?: number                 // Default width in pixels (rarely needed, use default)
  }
  rightItem: {
    children: ReactNode
  }
  resizeAble?: boolean            // Enable resizing
}
```

**Note:** Usually you don't need to specify `size`, `minSize`, or `maxSize`. The default width is fine for most cases. Only use these when you specifically need a resizable sidebar.

### Real-World Example: Reports Editor

```typescript
import { ConfigLayout, Content } from '@pimcore/studio-ui-bundle/components'

export const ReportsEditor = () => {
  return (
    <Content loading={isLoading}>
      <ConfigLayout
        leftItem={{
          children: (
            <ReportsSidebar
              handleOpenReport={handleOpenReport}
              isFetching={isFetching}
              isLoading={isLoading}
            />
          )
        }}
        rightItem={{
          children: <ReportContent />
        }}
      />
    </Content>
  )
}
```

### Real-World Example: Field Definition Editor

```typescript
import { ConfigLayout, ContentLayout, Toolbar, Flex, IconButton } from '@pimcore/studio-ui-bundle/components'

export const FieldDefinitionDetail = () => {
  return (
    <ContentLayout
      className="absolute-stretch"
      renderToolbar={
        <Toolbar>
          <Flex gap="mini">
            <IconButton icon={{ value: 'refresh' }} onClick={handleRefresh} />
          </Flex>
          <DetailSave />
        </Toolbar>
      }
    >
      <Content loading={isLoading}>
        <ConfigLayout
          leftItem={{
            minSize: 250,
            maxSize: 350,
            children: <DetailSidebar />
          }}
          resizeAble
          rightItem={{
            children: (
              <Content padded>
                <FormKit formProps={{ form }}>
                  {/* Field configuration form */}
                </FormKit>
              </Content>
            )
          }}
        />
      </Content>
    </ContentLayout>
  )
}
```

## Common Layout Patterns

### Pattern 1: Form with Toolbar

**Use case:** Edit forms with save/cancel actions

```typescript
import { Flex, Content, Toolbar, Button } from '@pimcore/studio-ui-bundle/components'

export const EntityEditForm = ({ entity, onSave, onCancel }) => {
  return (
    <Flex
      className="absolute-stretch"
      vertical
      justify="space-between"
    >
      <Content padded>
        <FormKit formProps={{ form, onFinish: onSave }}>
          {/* Form fields */}
        </FormKit>
      </Content>
      
      <Toolbar justify="flex-end">
        <Button onClick={onCancel}>Cancel</Button>
        <Button type="primary" htmlType="submit">Save</Button>
      </Toolbar>
    </Flex>
  )
}
```

### Pattern 2: Sidebar with Main Content

**Use case:** Settings pages, editors with navigation

```typescript
import { ConfigLayout, Content } from '@pimcore/studio-ui-bundle/components'

export const SettingsPage = () => {
  return (
    <ConfigLayout
      leftItem={{
        children: <SettingsNavigation />
      }}
      rightItem={{
        children: (
          <Content padded>
            <SettingsContent />
          </Content>
        )
      }}
    />
  )
}
```

### Pattern 3: Stacked Content Sections

**Use case:** Multi-section forms, complex toolbars, data views

```typescript
import { Content, Space, Box, Flex } from '@pimcore/studio-ui-bundle/components'

// Multi-section form
export const MultiSectionForm = () => {
  return (
    <Content padded padding={{ x: 'extra-small', y: 'extra-small' }}>
      <Space direction="vertical" size="large">
        <Box><Title level={2}>Basic Info</Title><BasicInfoForm /></Box>
        <Box><Title level={2}>Advanced</Title><AdvancedForm /></Box>
      </Space>
    </Content>
  )
}

// Complex toolbar with grouped actions
export const ComplexToolbar = () => {
  return (
    <Flex justify="space-between" align="center">
      <Flex gap="small"><IconButton icon={{ value: 'arrow-left' }} /><Title level={3}>Editor</Title></Flex>
      <Space size="mini"><IconButton icon={{ value: 'undo' }} /><IconButton icon={{ value: 'redo' }} /></Space>
      <Flex gap="mini"><Button>Cancel</Button><Button type="primary">Save</Button></Flex>
    </Flex>
  )
}

// Data view with loading/empty states
export const DataView = () => {
  const { data, isLoading } = useGetDataQuery()
  
  return (
    <Content loading={isLoading} padded>
      {data?.length ? (
        <Space direction="vertical" size="small">
          {data.map(item => <DataCard key={item.id} item={item} />)}
        </Space>
      ) : (
        <EmptyState message="No data" />
      )}
    </Content>
  )
}
```

## Best Practices

### 1. Use Content for Main Sections

✅ **Do this:**
```typescript
<Content loading={isLoading} padded>
  <MyForm />
</Content>
```

❌ **Not this:**
```typescript
{isLoading ? <Spinner /> : (
  <div style={{ padding: '16px' }}>
    <MyForm />
  </div>
)}
```

**Why:** Content provides consistent loading UI and spacing.

---

### 2. Prefer Semantic Components Over Inline Styles

✅ **Do this:**
```typescript
<Box padding="small" margin={{ bottom: 'large' }}>
  <Section />
</Box>
```

❌ **Not this:**
```typescript
<div style={{ padding: '12px', marginBottom: '24px' }}>
  <Section />
</div>
```

**Why:** Uses design system values, consistent across app.

---

### 3. Use Space for Simple Spacing

✅ **Do this:**
```typescript
<Space size="small">
  <Button>Cancel</Button>
  <Button type="primary">Save</Button>
</Space>
```

❌ **Not this:**
```typescript
<Flex gap="small">
  <Button>Cancel</Button>
  <Button type="primary">Save</Button>
</Flex>
```

**Why:** Space is simpler when you don't need alignment.

---

### 4. Use Flex for Alignment Control

✅ **Do this:**
```typescript
<Flex align="center" justify="space-between">
  <Title>Header</Title>
  <IconButton icon={{ value: 'close' }} />
</Flex>
```

✅ **Also good:**
```typescript
<Space size="small">
  <Button>Action 1</Button>
  <Button>Action 2</Button>
</Space>
```

**Why:** Choose the right tool - Flex when you need alignment, Space when you don't.

---

### 5. Use ConfigLayout for Two-Column Layouts

✅ **Do this:**
```typescript
// Basic usage - no size specs needed
<ConfigLayout
  leftItem={{
    children: <Sidebar />
  }}
  rightItem={{
    children: <Content padded><MainView /></Content>
  }}
/>

// Only add size props when you need resizeAble
<ConfigLayout
  leftItem={{
    minSize: 250,
    maxSize: 400,
    children: <Sidebar />
  }}
  resizeAble
  rightItem={{
    children: <Content padded><MainView /></Content>
  }}
/>
```

**Why:** Built-in two-column layout with optional resize behavior, consistent UX. Use default width unless you specifically need custom sizing.

---

### 6. Combine Components for Complex Layouts

✅ **Do this:**
```typescript
<Flex vertical className="absolute-stretch">
  <Content padded>
    <Space direction="vertical" size="large">
      <Section1 />
      <Section2 />
    </Space>
  </Content>
  
  <Toolbar justify="flex-end">
    <Button>Save</Button>
  </Toolbar>
</Flex>
```

**Why:** Composable components create flexible, maintainable layouts.

---

### 7. Use Loading Props on Content

✅ **Do this:**
```typescript
<Content loading={isLoading} padded>
  {data && <DataDisplay data={data} />}
</Content>
```

❌ **Not this:**
```typescript
<Content padded>
  {isLoading ? <Skeleton /> : data && <DataDisplay data={data} />}
</Content>
```

**Why:** Content has built-in loading skeleton that prevents layout shift.

## Common Mistakes to Avoid

### ❌ Don't Nest Content Inside Content

```typescript
// BAD - nested Content components
<Content padded>
  <Content padded>
    <Form />
  </Content>
</Content>
```

✅ **Use Box or Flex for inner sections:**
```typescript
// GOOD
<Content padded>
  <Box padding="small">
    <Form />
  </Box>
</Content>
```

---

### ❌ Don't Use Inline Styles for Spacing

```typescript
// BAD
<div style={{ margin: '8px 16px' }}>
  <Component />
</div>
```

✅ **Use Box or spacing props:**
```typescript
// GOOD
<Box margin={{ y: 'extra-small', x: 'small' }}>
  <Component />
</Box>
```

---

### ❌ Don't Mix Spacing Systems

```typescript
// BAD - mixing px values with spacing system
<Flex gap={8}>
  <Box padding="small">
    <Component />
  </Box>
</Flex>
```

✅ **Be consistent:**
```typescript
// GOOD
<Flex gap="extra-small">
  <Box padding="small">
    <Component />
  </Box>
</Flex>
```

---

### ❌ Don't Specify Sizes Unless You Need Resizable

```typescript
// BAD - unnecessary size specification without resizeAble
<ConfigLayout
  leftItem={{ 
    size: 300,
    minSize: 250,
    maxSize: 400,
    children: <Sidebar /> 
  }}
  rightItem={{ children: <Main /> }}
/>
```

✅ **Use default size, only specify when resizeAble:**
```typescript
// GOOD - use default width (most cases)
<ConfigLayout
  leftItem={{ children: <Sidebar /> }}
  rightItem={{ children: <Main /> }}
/>

// GOOD - only specify sizes when you need resizeAble
<ConfigLayout
  leftItem={{
    minSize: 250,
    maxSize: 400,
    children: <Sidebar />
  }}
  resizeAble
  rightItem={{ children: <Main /> }}
/>
```

---

### ❌ Don't Overuse Flex When Space Is Enough

```typescript
// BAD - overkill for simple spacing
<Flex direction="horizontal" gap="small" align="start">
  <Button>Cancel</Button>
  <Button>Save</Button>
</Flex>
```

✅ **Use Space for simple cases:**
```typescript
// GOOD
<Space size="small">
  <Button>Cancel</Button>
  <Button>Save</Button>
</Space>
```

## Quick Reference

### Component Selection Guide

| Need | Use | Props |
|------|-----|-------|
| Main content wrapper | `Content` | `padded`, `loading`, `gap` |
| Add spacing around element | `Box` | `padding`, `margin` |
| Align/distribute elements | `Flex` | `align`, `justify`, `gap`, `vertical` |
| Simple spacing | `Space` | `size`, `direction` |
| Sidebar + content | `ConfigLayout` | `leftItem`, `rightItem`, `resizeAble` |

### Spacing Values Quick Ref

- `mini` = 4px
- `extra-small` = 8px
- `small` = 12px
- `medium` = 16px (default)
- `large` = 24px
- `extra-large` = 32px

## Next Steps

After mastering layout components:
- [**pimcore-studio-ui-forms-antd**](../pimcore-studio-ui-forms-antd/SKILL.md) - Form layout patterns with FormKit
- [**pimcore-studio-ui-bundle-structure**](../pimcore-studio-ui-bundle-structure/SKILL.md) - Understanding overall component organization
- **Ant Design components** - Buttons, inputs, and other UI elements
