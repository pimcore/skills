---
name: pimcore-studio-bundle-setup
description: Setting up a new Pimcore Studio bundle from scratch - boilerplate, frontend build, module federation, PHP configuration
metadata:
  audience: pimcore-developers
  focus: architecture
---

## What This Skill Covers

How to create a new Pimcore Studio bundle from scratch:
- Complete directory structure and boilerplate files
- PHP bundle class and service configuration
- Frontend setup with rsbuild and module federation
- Plugin and module system entry points
- Build commands (dev, dev-server, production)
- Connecting the bundle to Pimcore Studio

## When to Use This Skill

Use this when:
- Creating a new Pimcore Studio extension bundle
- Setting up the frontend build for a bundle
- Need the boilerplate structure for a Studio plugin
- Connecting a PHP bundle to the Studio frontend
- Understanding how module federation works in Studio bundles

## References

- **Getting Started Guide**: https://docs.pimcore.com/platform/Studio_UI/Plugins_Getting_Started
- **Example Bundle** (blueprint): https://github.com/pimcore/studio-example-bundle

## Complete Bundle Structure

```
your-bundle/
├── assets/                           # Frontend assets
│   ├── js/
│   │   └── src/
│   │       ├── main.ts               # Intentionally empty — webpack entry stub; plugins.ts is exposed directly by module federation
│   │       ├── plugins.ts            # Plugin exports (real module federation entry point)
│   │       └── modules/              # Your feature modules
│   │           └── your-feature/
│   │               ├── index.ts      # Plugin definition
│   │               └── modules/
│   │                   └── your-feature-module.tsx
│   ├── package.json                  # NPM dependencies and scripts
│   ├── rsbuild.config.ts             # Build configuration with module federation
│   └── tsconfig.json                 # TypeScript configuration
├── config/
│   └── services.yaml                 # Symfony service definitions
├── public/
│   └── build/                        # Build output (git-ignored in dev)
├── src/
│   ├── PimcoreYourBundle.php         # Bundle class
│   └── Webpack/
│       └── WebpackEntryPointProvider.php  # Registers frontend entry points
└── composer.json                     # PHP dependencies
```

## Step 1: PHP Bundle Setup

### Bundle Class

```php
<?php
// src/PimcoreYourBundle.php

namespace Pimcore\Bundle\YourBundle;

use Pimcore\Extension\Bundle\AbstractPimcoreBundle;

class PimcoreYourBundle extends AbstractPimcoreBundle
{
    public function getPath(): string
    {
        return \dirname(__DIR__);
    }
}
```

### Composer Configuration

```json
{
  "name": "pimcore/your-bundle",
  "type": "pimcore-bundle",
  "require": {
    "pimcore/studio-ui-bundle": "^1.0",
    "pimcore/studio-backend-bundle": "^1.0"
  },
  "autoload": {
    "psr-4": {
      "Pimcore\\Bundle\\YourBundle\\": "src/"
    }
  },
  "extra": {
    "pimcore": {
      "bundles": [
        "Pimcore\\Bundle\\YourBundle\\PimcoreYourBundle"
      ]
    }
  }
}
```

### WebpackEntryPointProvider

This PHP class tells Studio where to find the frontend build output:

```php
<?php
// src/Webpack/WebpackEntryPointProvider.php

namespace Pimcore\Bundle\YourBundle\Webpack;

use Pimcore\Bundle\StudioUiBundle\Webpack\WebpackEntryPointProviderInterface;

final class WebpackEntryPointProvider implements WebpackEntryPointProviderInterface
{
    public function getEntryPointsJsonLocations(): array
    {
        return glob(__DIR__ . '/../../public/build/*/entrypoints.json');
    }

    public function getEntryPoints(): array
    {
        return ['exposeRemote'];
    }

    public function getOptionalEntryPoints(): array
    {
        return [];
    }
}
```

### Service Configuration

```yaml
# config/services.yaml
services:
  _defaults:
    autowire: true
    autoconfigure: true
    public: false

  Pimcore\Bundle\YourBundle\Webpack\WebpackEntryPointProvider:
    tags:
      - { name: pimcore_studio_ui.webpack_entry_point_provider }
      # Optional: also add this tag to support document editor iframe loading
      # - { name: pimcore_studio_ui.webpack_entry_point_provider.document_editor_iframe }
```

**The `pimcore_studio_ui.webpack_entry_point_provider` tag is required** — it registers the frontend entry points with Studio core.

## Step 2: Frontend Setup

### package.json

```json
{
  "name": "pimcore-your-bundle",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "dev": "NODE_ENV=development rsbuild build",
    "dev-server": "NODE_ENV=dev-server rsbuild dev",
    "build": "rsbuild build",
    "lint": "eslint --ext .js,.jsx,.ts,.tsx ./js",
    "lint-fix": "eslint --ext .js,.jsx,.ts,.tsx ./js --fix",
    "check-types": "tsc --noEmit"
  },
  "dependencies": {
    "@pimcore/studio-ui-bundle": "<pin exact canary version — see Install Dependencies below>",
    "react": "18.3.x",
    "react-dom": "18.3.x",
    "uuid": "^10.0.0"
  },
  "devDependencies": {
    "@module-federation/rsbuild-plugin": "^0.13.1",
    "@rsbuild/core": "^1.3.16",
    "@rsbuild/plugin-react": "^1.3.1",
    "typescript": "^5.3.3"
  }
}
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "module": "ESNext",
    "target": "es2017",
    "moduleResolution": "bundler",
    "types": ["@types/webpack-env"],
    "baseUrl": "./js/",
    "strictNullChecks": true,
    "jsx": "react",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "resolveJsonModule": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "skipLibCheck": true,
    "paths": {
      "*": ["./@mf-types/*"]
    }
  },
  "include": [
    "./js/**/*",
    "./@mf-types/*"
  ]
}
```

### main.ts

`assets/js/src/main.ts` is **intentionally empty**. rsbuild requires an entry file in `source.entry`; this stub satisfies that requirement. Module federation exposes `plugins.ts` directly via the `exposes` config — `main.ts` is never imported or executed.

```typescript
// assets/js/src/main.ts
// Intentionally empty.
// rsbuild requires a source entry file; this stub satisfies that requirement.
// Module federation exposes plugins.ts directly via rsbuild.config.ts `exposes`.
// Do NOT add imports here.
```

### rsbuild.config.ts (Module Federation)

This is the build configuration that enables your bundle to integrate with Studio via module federation:

```typescript
// assets/rsbuild.config.ts
import { defineConfig } from '@rsbuild/core'
import { pluginReact } from '@rsbuild/plugin-react'
import { pluginModuleFederation } from '@module-federation/rsbuild-plugin'
import { pluginGenerateEntrypoints } from '@pimcore/studio-ui-bundle/rsbuild/plugins'
import { createDynamicRemote } from '@pimcore/studio-ui-bundle/rsbuild/utils'
import path from 'path'
import fs from 'fs'
import { v4 } from 'uuid'
import packages from './package.json'

const buildId = v4()
const buildPath = path.resolve(__dirname, '..', 'public', 'build', buildId)

// Clean previous builds
if (fs.existsSync(path.resolve(__dirname, '..', 'public', 'build'))) {
  fs.readdirSync(path.resolve(__dirname, '..', 'public', 'build')).forEach((file) => {
    if (file !== 'studio-npm-package.tgz') {
      fs.rmSync(path.resolve(__dirname, '..', 'public', 'build', file), { recursive: true })
    }
  })
}

if (!fs.existsSync(buildPath)) {
  fs.mkdirSync(buildPath, { recursive: true })
}

let nodeEnv = process.env.NODE_ENV
let env: 'development' | 'production' = 'production'
const isDevServer = nodeEnv === 'dev-server'
if (nodeEnv !== env) {
  env = 'development'
}

export default defineConfig({
  mode: env,
  server: {
    port: 3032,  // Pick a unique port per bundle
  },
  dev: {
    ...(!isDevServer ? { assetPrefix: '/bundles/pimcoreyour/build/' + buildId } : {}),
    client: {
      host: 'localhost',
      port: 3032,
      protocol: 'ws'
    }
  },
  source: {
    entry: {
      main: './js/src/main.ts'
    },
    decorators: {
      version: 'legacy'
    }
  },
  output: {
    manifest: true,
    assetPrefix: '/bundles/pimcoreyour/build/' + buildId,
    distPath: {
      root: buildPath
    },
  },
  tools: {
    bundlerChain: (chain, { env }) => {
      chain.output.uniqueName('pimcore_your_bundle')
    },
  },
  plugins: [
    pluginGenerateEntrypoints(),
    pluginReact(),
    pluginModuleFederation({
      name: 'pimcore_your_bundle',
      filename: 'static/js/remoteEntry.js',
      exposes: {
        '.': './js/src/plugins.ts',
      },
      dts: false,
      remotes: {
        '@pimcore/studio-ui-bundle': createDynamicRemote('pimcore_studio_ui_bundle'),
      },
      shared: {
        ...packages.dependencies,
        react: {
          singleton: true,
          eager: true,
          requiredVersion: false,
        },
        'react-dom': {
          singleton: true,
          eager: true,
          requiredVersion: false,
        }
      },
    })
  ]
})
```

**Deriving the right values from the bundle class name** (e.g. `PimcoreInspireCocktailDemoBundle`):

| Value | Rule | Example |
|-------|------|---------|
| `assetPrefix` | Strip `Pimcore` + `Bundle`, lowercase everything, prefix `/bundles/`, suffix `/build/` | `/bundles/inspirecocktaildemo/build/` |
| `uniqueName` / MF `name` | Full class name without `Pimcore` prefix, snake_case | `pimcore_inspire_cocktail_demo_bundle` |
| `server.port` | Pick any free port above 3030 (studio core uses 3030) | `3032` |

## Step 3: Plugin Entry Point

### plugins.ts

The main entry point that Studio discovers and loads:

```typescript
// assets/js/src/plugins.ts
import { type IAbstractPlugin } from '@pimcore/studio-ui-bundle'

export const YourPlugin: IAbstractPlugin = {
  name: 'YourPlugin',

  onInit ({ container }) {
    // Register or override services in DI container
    // container.bind('some-service', SomeService)
  },

  onStartup ({ moduleSystem }) {
    // Register modules that configure the app
    // moduleSystem.registerModule(YourModule)
  }
}
```

**Lifecycle:**
1. `onInit` runs first — register NEW services or OVERRIDE existing ones
2. `onStartup` runs after all services are ready — register modules that CONFIGURE services

### Adding a Module

```typescript
// assets/js/src/modules/your-feature/modules/your-feature-module.tsx
import { type AbstractModule, container } from '@pimcore/studio-ui-bundle'
import { serviceIds } from '@pimcore/studio-ui-bundle/app'
import { type MainNavRegistry } from '@pimcore/studio-ui-bundle/modules/app'

export const YourFeatureModule: AbstractModule = {
  onInit: () => {
    const mainNavRegistry = container.get<MainNavRegistry>(serviceIds.mainNavRegistry)

    mainNavRegistry.registerMainNavItem({
      path: 'Tools/Your Feature',
      label: 'your-feature.title',
      widgetConfig: {
        name: 'Your Feature',
        id: 'your-feature',
        component: 'your-feature-widget',
        config: {
          translationKey: 'your-feature.title',
          icon: { type: 'name', value: 'tool' }
        }
      }
    })
  }
}
```

### Wiring Plugin to Module

```typescript
// assets/js/src/modules/your-feature/index.ts
import { type IAbstractPlugin } from '@pimcore/studio-ui-bundle'
import { YourFeatureModule } from './modules/your-feature-module'

export const YourFeaturePlugin: IAbstractPlugin = {
  name: 'YourFeaturePlugin',

  onStartup ({ moduleSystem }) {
    moduleSystem.registerModule(YourFeatureModule)
  }
}
```

Then export from plugins.ts:

```typescript
// assets/js/src/plugins.ts
import { YourFeaturePlugin } from './modules/your-feature'

export { YourFeaturePlugin }
```

## Step 4: Build and Run

### Install Dependencies

`@pimcore/studio-ui-bundle` has no stable release yet — only canary versions. Before running `npm install`, check the latest canary and pin it exactly:

```bash
npm view @pimcore/studio-ui-bundle dist-tags
```

Update `package.json` with the exact version string from the `canary` tag (e.g. `1.0.0-canary.20260402-132451-02a210f`), then:

```bash
# In the assets/ directory
npm install
```

### Build Commands

```bash
# Development build (one-time, outputs to public/build/)
npm run dev

# Development server (hot reload, for active development)
npm run dev-server

# Production build
npm run build

# Type checking
npm run check-types

# Lint
npm run lint-fix
```

### Dev Server vs Dev Build

| Command | Use When | Output |
|---------|----------|--------|
| `npm run dev` | Quick build for testing | `public/build/` |
| `npm run dev-server` | Active frontend development | Served from memory (port 3032) |
| `npm run build` | Production/deployment | `public/build/` (optimized) |

**When the dev server is running**, Studio core automatically discovers it and loads your bundle's frontend from the dev server instead of the static build.

## Common Mistakes to Avoid

### Missing WebpackEntryPointProvider Tag

```yaml
# DON'T - Missing tag
Pimcore\Bundle\YourBundle\Webpack\WebpackEntryPointProvider: ~

# DO - Include the tag
Pimcore\Bundle\YourBundle\Webpack\WebpackEntryPointProvider:
  tags:
    - { name: pimcore_studio_ui.webpack_entry_point_provider }
```

Without the tag, Studio won't discover your frontend bundle.

---

### Wrong assetPrefix Path

```typescript
// DON'T - Wrong path format
assetPrefix: '/bundles/YourBundle/build/' + buildId

// DO - Lowercase, no separators
assetPrefix: '/bundles/pimcoreyour/build/' + buildId
```

Symfony serves bundle assets at `/bundles/{lowercasebundlename}/`. The prefix is the bundle name without "Bundle", all lowercase.

---

### Port Conflicts

```typescript
// DON'T - Use same port as another bundle or studio core (3030)
server: { port: 3030 }

// DO - Use unique port
server: { port: 3032 }
```

Studio core uses port 3030. Each bundle dev server needs its own port.

---

### Missing Module Federation Remote

```typescript
// DON'T - Forget the remote
pluginModuleFederation({
  name: 'pimcore_your_bundle',
  // Missing remotes!
})

// DO - Always include the studio-ui-bundle remote
pluginModuleFederation({
  name: 'pimcore_your_bundle',
  remotes: {
    '@pimcore/studio-ui-bundle': createDynamicRemote('pimcore_studio_ui_bundle'),
  },
})
```

---

### Exporting Plugins Incorrectly

```typescript
// DON'T - Default export
export default YourPlugin

// DO - Named export
export { YourPlugin }
```

Module federation expects named exports from `plugins.ts`.

## Quick Reference Checklist

To set up a new Studio bundle:

### PHP Side:
- [ ] Create bundle class extending `AbstractPimcoreBundle`
- [ ] Add `composer.json` with `pimcore/studio-ui-bundle` dependency
- [ ] Create `WebpackEntryPointProvider` implementing `WebpackEntryPointProviderInterface`
- [ ] Register provider in `services.yaml` with `pimcore_studio_ui.webpack_entry_point_provider` tag

### Frontend Side:
- [ ] Create `assets/` directory with `package.json`, `tsconfig.json`, `rsbuild.config.ts`
- [ ] Set unique `name`, `port`, and `assetPrefix` in rsbuild config
- [ ] Create `plugins.ts` with named plugin exports
- [ ] Create at least one module with `onInit`
- [ ] Run `npm install` and `npm run dev`

### Verify:
- [ ] Bundle is registered in Pimcore (`bin/console pimcore:bundle:list`)
- [ ] Frontend build exists in `public/build/`
- [ ] Studio loads without errors
- [ ] Your plugin appears in the browser console (check for registration logs)

## Next Steps

After setting up the bundle:
- [**pimcore-studio-ui-using-sdk-in-bundles**](../pimcore-studio-ui-using-sdk-in-bundles/SKILL.md) - SDK imports, DI, registries
- [**pimcore-studio-ui-navigation**](../pimcore-studio-ui-navigation/SKILL.md) - Adding navigation entries
- [**pimcore-studio-ui-react-components**](../pimcore-studio-ui-react-components/SKILL.md) - Creating UI components
- [**pimcore-studio-ui-bundle-structure**](../pimcore-studio-ui-bundle-structure/SKILL.md) - Module organization patterns
