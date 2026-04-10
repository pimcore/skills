# Pimcore Skills

A collection of skills for AI coding assistants (Claude Code, OpenCode, etc.) to help build Pimcore Studio UI bundles and extensions.

## What Are Skills?

Skills are domain-specific instruction sets that give AI assistants the context they need to write code that follows Pimcore Studio conventions. They cover component patterns, extension points, data fetching, forms, permissions, and more.

## Installation

### Claude Code

```bash
git clone https://github.com/pimcore/skills.git ~/pimcore-skills

mkdir -p ~/.claude/skills
ln -s ~/pimcore-skills/skills/* ~/.claude/skills/
```

### OpenCode

```bash
git clone https://github.com/pimcore/skills.git ~/pimcore-skills

mkdir -p ~/.config/opencode/skills
ln -s ~/pimcore-skills/skills/* ~/.config/opencode/skills/
```

Update later with `cd ~/pimcore-skills && git pull`.

## What's Included

### Bundle Setup & Architecture
- **pimcore-studio-bundle-setup** — Create a Studio bundle from scratch (PHP + frontend boilerplate)
- **pimcore-studio-ui-bundle-structure** — Bundle directory layout and organization
- **pimcore-studio-ui-using-sdk-in-bundles** — Plugins, modules, dependency injection, registries

### Components
- **pimcore-studio-ui-buttons** — Button, IconButton, IconTextButton, DropdownButton, ButtonGroup
- **pimcore-studio-ui-forms-antd** — FormKit + Ant Design forms
- **pimcore-studio-ui-icons** — Icon component, custom SVG registration, color groups
- **pimcore-studio-ui-layout-components** — Content, Box, Flex, Space, ConfigLayout
- **pimcore-studio-ui-modals** — Modal dialogs (declarative + imperative)
- **pimcore-studio-ui-notifications-toasts** — Toast messages via useMessage
- **pimcore-studio-ui-react-components** — Component patterns, structure, styling
- **pimcore-studio-ui-tables-grids** — Grid component and TanStack Table

### Extension Points
- **pimcore-studio-ui-context-menus** — Adding items to tree/grid/toolbar context menus
- **pimcore-studio-ui-listings** — Custom listings via the ListingBuilder decorator pattern
- **pimcore-studio-ui-navigation** — Main navigation entries and perspective permissions
- **pimcore-studio-ui-tabs-editors** — Custom editor tabs (asset/document/data-object)
- **pimcore-studio-ui-widgets** — Custom widgets and widget areas

### Patterns & Fundamentals
- **pimcore-studio-ui-dynamic-types** — Extensible type system (grid cells, field definitions, data types)
- **pimcore-studio-ui-error-handling** — trackError, ApiError, GeneralError, ErrorBoundary
- **pimcore-studio-ui-i18n** — useTranslation, key conventions, interpolation
- **pimcore-studio-ui-permissions** — Element, user, and perspective permission checks
- **pimcore-studio-ui-rtk-query-fundamentals** — Data fetching, mutations, caching, error handling
- **pimcore-studio-ui-typescript-best-practices** — Type safety rules, lodash utils, return types

### Reference
- **CRITICAL-IMPORT-PATHS.md** — Import path rules (`@sdk/*` vs `@Pimcore/*` vs `@pimcore/studio-ui-bundle/*`)

## Resources

- [Pimcore Studio UI Documentation](https://docs.pimcore.com/platform/Studio_UI/)
- [Studio Plugins Getting Started](https://docs.pimcore.com/platform/Studio_UI/Plugins_Getting_Started)
- [Studio Example Bundle](https://github.com/pimcore/studio-example-bundle)

## License

See LICENSE file.
