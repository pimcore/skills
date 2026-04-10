# Pimcore Skills

A collection of skills for AI coding assistants (Claude Code, OpenCode) to help build Pimcore Studio bundles and extensions.

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

### Updating

Pull the latest skills with:

```bash
cd ~/pimcore-skills && git pull
```

If new skills were added, re-run the symlink command from the installation step to link them. Existing symlinks pick up content updates automatically.

### Windows

Symlinks only work in **WSL**. On native Windows, manually copy the `skills/*` folders into the agent's skills directory after each `git pull`.

## Skills

### Pimcore Studio

#### UI — Bundle Setup & Architecture

- [pimcore-studio-bundle-setup](skills/pimcore-studio-bundle-setup/SKILL.md)
- [pimcore-studio-ui-bundle-structure](skills/pimcore-studio-ui-bundle-structure/SKILL.md)
- [pimcore-studio-ui-using-sdk-in-bundles](skills/pimcore-studio-ui-using-sdk-in-bundles/SKILL.md)

#### UI — Components

- [pimcore-studio-ui-buttons](skills/pimcore-studio-ui-buttons/SKILL.md)
- [pimcore-studio-ui-forms-antd](skills/pimcore-studio-ui-forms-antd/SKILL.md)
- [pimcore-studio-ui-icons](skills/pimcore-studio-ui-icons/SKILL.md)
- [pimcore-studio-ui-layout-components](skills/pimcore-studio-ui-layout-components/SKILL.md)
- [pimcore-studio-ui-modals](skills/pimcore-studio-ui-modals/SKILL.md)
- [pimcore-studio-ui-notifications-toasts](skills/pimcore-studio-ui-notifications-toasts/SKILL.md)
- [pimcore-studio-ui-react-components](skills/pimcore-studio-ui-react-components/SKILL.md)
- [pimcore-studio-ui-tables-grids](skills/pimcore-studio-ui-tables-grids/SKILL.md)

#### UI — Extension Points

- [pimcore-studio-ui-context-menus](skills/pimcore-studio-ui-context-menus/SKILL.md)
- [pimcore-studio-ui-listings](skills/pimcore-studio-ui-listings/SKILL.md)
- [pimcore-studio-ui-navigation](skills/pimcore-studio-ui-navigation/SKILL.md)
- [pimcore-studio-ui-tabs-editors](skills/pimcore-studio-ui-tabs-editors/SKILL.md)
- [pimcore-studio-ui-widgets](skills/pimcore-studio-ui-widgets/SKILL.md)

#### UI — Patterns & Fundamentals

- [pimcore-studio-ui-dynamic-types](skills/pimcore-studio-ui-dynamic-types/SKILL.md)
- [pimcore-studio-ui-error-handling](skills/pimcore-studio-ui-error-handling/SKILL.md)
- [pimcore-studio-ui-i18n](skills/pimcore-studio-ui-i18n/SKILL.md)
- [pimcore-studio-ui-permissions](skills/pimcore-studio-ui-permissions/SKILL.md)
- [pimcore-studio-ui-rtk-query-fundamentals](skills/pimcore-studio-ui-rtk-query-fundamentals/SKILL.md)
- [pimcore-studio-ui-typescript-best-practices](skills/pimcore-studio-ui-typescript-best-practices/SKILL.md)

#### UI — Reference

- [CRITICAL-IMPORT-PATHS.md](skills/CRITICAL-IMPORT-PATHS.md)

#### Backend — Architecture & Code Style

- [pimcore-studio-backend-checklist](skills/pimcore-studio-backend-checklist/SKILL.md)
- [pimcore-studio-backend-code-style](skills/pimcore-studio-backend-code-style/SKILL.md)
- [pimcore-studio-backend-config](skills/pimcore-studio-backend-config/SKILL.md)

#### Backend — Layers

- [pimcore-studio-backend-controller](skills/pimcore-studio-backend-controller/SKILL.md)
- [pimcore-studio-backend-service](skills/pimcore-studio-backend-service/SKILL.md)
- [pimcore-studio-backend-dto](skills/pimcore-studio-backend-dto/SKILL.md)

#### Backend — Patterns

- [pimcore-studio-backend-exception](skills/pimcore-studio-backend-exception/SKILL.md)
- [pimcore-studio-backend-openapi-docs](skills/pimcore-studio-backend-openapi-docs/SKILL.md)
