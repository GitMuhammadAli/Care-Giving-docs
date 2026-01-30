# 📦 Monorepo Management Complete Guide

> A comprehensive guide to managing monorepos with Turborepo, Nx, workspace management, build optimization, and scaling strategies.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "A monorepo is a single repository containing multiple projects with shared dependencies and tooling, enabling code reuse, atomic changes, and unified CI/CD - managed by tools like Turborepo or Nx for efficient builds."

### The 6 Core Concepts (Memorize!)
```
1. WORKSPACE           → A package/project within the monorepo
2. TASK PIPELINE       → Define order of build tasks (build → test → deploy)
3. REMOTE CACHING      → Share build cache across team/CI (10x faster builds)
4. AFFECTED COMMANDS   → Only build/test what changed (not entire repo)
5. DEPENDENCY GRAPH    → Understand relationships between packages
6. TASK PARALLELIZATION → Run independent tasks simultaneously
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Affected"** | "We only run tests on affected packages - saves 80% CI time" |
| **"Remote caching"** | "Turborepo remote cache means CI hits cache 90% of the time" |
| **"Task graph"** | "Nx builds a task graph to parallelize independent builds" |
| **"Topological order"** | "Dependencies build first in topological order" |
| **"Workspace protocol"** | "We use workspace:* to link internal packages" |
| **"Colocated"** | "Shared code is colocated, reducing duplication" |
| **"Atomic commits"** | "Monorepo enables atomic commits across packages" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Cache hit speedup | **10-100x** | Skip work that's already done |
| Affected-only builds | **80-90%** | time saved in CI |
| Parallel task limit | **CPU cores** | Default parallelization |
| Recommended packages | **<500** | Beyond this, consider splitting |

### Turborepo vs Nx (Quick Comparison)
| Feature | Turborepo | Nx |
|---------|-----------|-----|
| **Setup** | Simple, minimal config | Feature-rich, more config |
| **Caching** | Excellent | Excellent |
| **Learning curve** | Low | Medium |
| **Code generation** | None built-in | Powerful generators |
| **Plugins** | Limited | Extensive |
| **Best for** | Simple monorepos | Large, complex monorepos |
| **Owned by** | Vercel | Nrwl |

### The "Wow" Statement (Memorize This!)
> "Before monorepo tooling, our CI took 45 minutes - building everything on every commit. With Turborepo's remote caching and affected commands, we're down to 3 minutes on average. The first developer builds, the cache is shared, and everyone else gets instant builds. We have 50 packages but only rebuild what's actually affected by each PR. Plus, atomic commits across packages mean we never have version mismatches - when we update the shared UI library, all apps using it are updated in the same commit."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                        MONOREPO STRUCTURE                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  monorepo/                                                      │
│  ├── apps/                    ◄── Deployable applications       │
│  │   ├── web/                     (Next.js, Express, etc.)     │
│  │   ├── mobile/                                                │
│  │   └── api/                                                   │
│  │                                                              │
│  ├── packages/                ◄── Shared libraries              │
│  │   ├── ui/                      (Reusable across apps)       │
│  │   ├── utils/                                                 │
│  │   ├── config/                                                │
│  │   └── types/                                                 │
│  │                                                              │
│  ├── turbo.json               ◄── Task pipeline config         │
│  ├── package.json             ◄── Root package + workspaces    │
│  └── pnpm-workspace.yaml      ◄── Workspace definition         │
│                                                                  │
│  DEPENDENCY GRAPH:                                              │
│                                                                  │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐                  │
│  │   web   │────►│   ui    │────►│  utils  │                  │
│  └─────────┘     └─────────┘     └─────────┘                  │
│       │                               ▲                        │
│       │          ┌─────────┐          │                        │
│       └─────────►│  types  │──────────┘                        │
│                  └─────────┘                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is a monorepo?"**
> "Single repository with multiple projects sharing dependencies and tooling. Enables code reuse, atomic changes across packages, and unified CI/CD."

**Q: "Monorepo vs polyrepo?"**
> "Monorepo: easier code sharing, atomic commits, single CI. Polyrepo: independent deployments, clearer ownership, simpler per-repo. Choose monorepo when teams share code frequently."

**Q: "How do you optimize monorepo builds?"**
> "Three strategies: caching (local + remote), affected commands (only build changed packages), and parallelization (run independent tasks simultaneously)."

**Q: "What is remote caching?"**
> "Shared build cache across team and CI. First person builds, cache is uploaded. Others download cache instead of rebuilding. 10-100x faster."

**Q: "Turborepo vs Nx?"**
> "Turborepo: simpler, faster setup, great caching. Nx: more features, code generators, better for large complex repos. Both excellent."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "Why use a monorepo?"

**Junior Answer:**
> "It's when you put all your code in one repository."

**Senior Answer:**
> "A monorepo solves several problems in multi-project development:

**1. Code Sharing**
- Shared libraries are colocated - no publishing to npm needed
- Import directly: `import { Button } from '@myorg/ui'`
- Changes to shared code are immediately available

**2. Atomic Changes**
- Update a shared library AND all consumers in one commit
- No version mismatches between packages
- Refactoring across packages is straightforward

**3. Unified Tooling**
- One ESLint config, one TypeScript config, one CI pipeline
- Consistent developer experience across all projects
- Single source of truth for dependencies

**4. Build Optimization**
- **Caching**: Don't rebuild what hasn't changed
- **Affected**: Only test/build packages affected by changes
- **Parallelization**: Run independent tasks simultaneously

Trade-offs:
- Requires tooling (Turborepo, Nx) to manage at scale
- Repository can grow large (use sparse checkout)
- CI complexity (need affected commands)
- Access control is all-or-nothing (can be mitigated with CODEOWNERS)"

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "How does caching work?" | "Hash inputs (source, deps, config). If hash matches, use cached output. Remote cache shares across team - first build uploads, others download." |
| "How do affected commands work?" | "Compare git diff to dependency graph. If `utils` changed, rebuild `utils` and everything that depends on it, skip the rest." |
| "How do you handle versioning?" | "Two approaches: fixed (all packages same version, like Angular) or independent (Lerna/changesets). Fixed is simpler, independent more flexible." |
| "What about large repos?" | "Sparse checkout (clone only what you need), remote caching, CI parallelization, and consider splitting if >500 packages." |

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Tools Comparison](#2-tools-comparison)
3. [Workspace Management](#3-workspace-management)
4. [Build Optimization](#4-build-optimization)
5. [CI/CD Strategies](#5-cicd-strategies)
6. [Common Patterns](#6-common-patterns)
7. [Scaling & Performance](#7-scaling--performance)
8. [When to Use / Not Use](#8-when-to-use--not-use)
9. [Interview Questions](#9-interview-questions)

---

## 1. Core Concepts

### Monorepo vs Polyrepo

```
┌─────────────────────────────────────────────────────────────────┐
│                    MONOREPO vs POLYREPO                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  POLYREPO (Multiple Repositories):                              │
│  ─────────────────────────────────                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                      │
│  │ repo-web │  │ repo-api │  │ repo-ui  │                      │
│  └──────────┘  └──────────┘  └──────────┘                      │
│       │              │              │                           │
│       └──────────────┼──────────────┘                           │
│                      │                                          │
│              npm publish @myorg/ui                              │
│                                                                  │
│  ✓ Independent deployments                                     │
│  ✓ Clear ownership boundaries                                  │
│  ✓ Simple per-repo CI/CD                                       │
│  ✗ Sharing code requires npm publish                           │
│  ✗ Cross-repo changes need multiple PRs                        │
│  ✗ Version mismatches between repos                            │
│                                                                  │
│  MONOREPO (Single Repository):                                  │
│  ─────────────────────────────                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │ monorepo                                                │    │
│  │ ├── apps/web     (imports @myorg/ui directly)         │    │
│  │ ├── apps/api     (imports @myorg/utils directly)      │    │
│  │ └── packages/ui  (no npm publish needed)              │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ✓ Code sharing without npm publish                            │
│  ✓ Atomic changes across packages                              │
│  ✓ Unified tooling and dependencies                            │
│  ✓ Single source of truth                                      │
│  ✗ Requires monorepo tooling                                   │
│  ✗ CI complexity (need affected builds)                        │
│  ✗ Repository can grow large                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### How Monorepo Builds Work

```
BUILD OPTIMIZATION FLOW:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. DETECT CHANGES                                              │
│     └── git diff main...HEAD                                   │
│     └── Identify changed files                                 │
│                                                                  │
│  2. BUILD DEPENDENCY GRAPH                                      │
│     └── Parse package.json dependencies                        │
│     └── Create DAG (Directed Acyclic Graph)                   │
│                                                                  │
│  3. DETERMINE AFFECTED PACKAGES                                 │
│     └── Changed package + all dependents                       │
│     └── Example: utils changed → ui, web, api affected        │
│                                                                  │
│  4. CHECK CACHE                                                 │
│     └── Hash: source files + dependencies + config             │
│     └── Cache hit? → Return cached output                      │
│     └── Cache miss? → Build and cache result                   │
│                                                                  │
│  5. EXECUTE IN TOPOLOGICAL ORDER                               │
│     └── Dependencies build before dependents                   │
│     └── Parallelize independent tasks                          │
│                                                                  │
│  EXAMPLE:                                                       │
│  ─────────────────────────────────────────────────────────────  │
│  Changed: packages/utils/index.ts                               │
│                                                                  │
│  Dependency graph:                                              │
│  utils ← ui ← web                                              │
│  utils ← api                                                    │
│                                                                  │
│  Build order:                                                   │
│  1. utils (changed, must rebuild)                              │
│  2. ui + api (parallel, both depend on utils)                  │
│  3. web (depends on ui)                                        │
│                                                                  │
│  Skipped: packages/config (not affected)                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Package Linking

```typescript
// ════════════════════════════════════════════════════════════════
// HOW PACKAGES REFERENCE EACH OTHER
// ════════════════════════════════════════════════════════════════

// packages/utils/package.json
{
  "name": "@myorg/utils",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts"
}

// packages/ui/package.json
{
  "name": "@myorg/ui",
  "version": "1.0.0",
  "dependencies": {
    "@myorg/utils": "workspace:*"  // ← Links to local package
  }
}

// apps/web/package.json
{
  "name": "@myorg/web",
  "dependencies": {
    "@myorg/ui": "workspace:*",
    "@myorg/utils": "workspace:*"
  }
}

// ════════════════════════════════════════════════════════════════
// WORKSPACE PROTOCOL VERSIONS
// ════════════════════════════════════════════════════════════════

"dependencies": {
  // workspace:* → Any version (most common)
  "@myorg/utils": "workspace:*",
  
  // workspace:^ → Caret range when publishing
  "@myorg/ui": "workspace:^",
  
  // workspace:~ → Tilde range when publishing
  "@myorg/config": "workspace:~"
}

// When published to npm, workspace:* becomes actual version:
// "workspace:*" → "1.0.0"
// "workspace:^" → "^1.0.0"
```

---

## 2. Tools Comparison

### Turborepo

Fast, simple monorepo build tool by Vercel.

```json
// turbo.json - Turborepo configuration
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    // Build task - depends on other packages' builds
    "build": {
      "dependsOn": ["^build"],  // ^ means dependencies first
      "outputs": ["dist/**", ".next/**"]
    },
    
    // Test depends on build being done first
    "test": {
      "dependsOn": ["build"],
      "outputs": []
    },
    
    // Lint has no dependencies, can run in parallel
    "lint": {
      "outputs": []
    },
    
    // Dev runs in persistent mode (watch)
    "dev": {
      "cache": false,
      "persistent": true
    },
    
    // Type check
    "typecheck": {
      "dependsOn": ["^build"],
      "outputs": []
    }
  }
}
```

```bash
# Turborepo Commands
# ══════════════════════════════════════════════════════════════════

# Run build in all packages (respects dependency order)
turbo run build

# Run build only in affected packages
turbo run build --filter=...[origin/main]

# Run build only in specific package and its dependencies
turbo run build --filter=@myorg/web...

# Run multiple tasks
turbo run build test lint

# Run in specific package only
turbo run build --filter=@myorg/ui

# Force no cache (fresh build)
turbo run build --force

# Dry run (show what would execute)
turbo run build --dry-run

# With remote caching (Vercel)
turbo run build --token=$TURBO_TOKEN --team=myteam
```

### Nx

Feature-rich monorepo tool with generators and plugins.

```json
// nx.json - Nx configuration
{
  "$schema": "./node_modules/nx/schemas/nx-schema.json",
  "namedInputs": {
    "default": ["{projectRoot}/**/*", "sharedGlobals"],
    "sharedGlobals": [],
    "production": [
      "default",
      "!{projectRoot}/**/*.spec.ts",
      "!{projectRoot}/tsconfig.spec.json"
    ]
  },
  "targetDefaults": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["production", "^production"],
      "cache": true
    },
    "test": {
      "inputs": ["default", "^production"],
      "cache": true
    },
    "lint": {
      "inputs": ["default"],
      "cache": true
    }
  },
  "defaultBase": "main"
}
```

```typescript
// project.json - Nx project configuration
{
  "name": "web",
  "$schema": "../../node_modules/nx/schemas/project-schema.json",
  "projectType": "application",
  "sourceRoot": "apps/web/src",
  "targets": {
    "build": {
      "executor": "@nx/next:build",
      "outputs": ["{options.outputPath}"],
      "options": {
        "outputPath": "dist/apps/web"
      }
    },
    "serve": {
      "executor": "@nx/next:server",
      "options": {
        "buildTarget": "web:build",
        "dev": true
      }
    },
    "test": {
      "executor": "@nx/jest:jest",
      "options": {
        "jestConfig": "apps/web/jest.config.ts"
      }
    }
  }
}
```

```bash
# Nx Commands
# ══════════════════════════════════════════════════════════════════

# Run build in all projects
nx run-many -t build

# Run build only in affected projects
nx affected -t build

# Run for specific project
nx build web

# Run multiple targets
nx run-many -t build,test,lint

# Generate new application
nx generate @nx/next:application my-app

# Generate new library
nx generate @nx/js:library my-lib

# View dependency graph
nx graph

# Show affected projects
nx affected:graph

# Reset Nx cache
nx reset
```

### pnpm Workspaces (No Build Tool)

Just workspace management, no caching or affected commands.

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

```json
// Root package.json
{
  "name": "monorepo",
  "private": true,
  "scripts": {
    "build": "pnpm -r run build",
    "test": "pnpm -r run test",
    "lint": "pnpm -r run lint"
  }
}
```

```bash
# pnpm Workspace Commands
# ══════════════════════════════════════════════════════════════════

# Install all dependencies
pnpm install

# Run script in all packages
pnpm -r run build

# Run script in specific package
pnpm --filter @myorg/web run build

# Run script in package and dependencies
pnpm --filter @myorg/web... run build

# Add dependency to specific package
pnpm --filter @myorg/web add react

# Add dependency to root
pnpm add -w typescript

# Add workspace dependency
pnpm --filter @myorg/web add @myorg/ui --workspace
```

### Full Comparison Table

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    MONOREPO TOOLS COMPARISON                              │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  FEATURE            │ Turborepo │ Nx        │ pnpm      │ Lerna       │
│  ─────────────────────────────────────────────────────────────────────── │
│  Local caching      │ ✓ Fast    │ ✓ Fast    │ ✗         │ ✓ Basic     │
│  Remote caching     │ ✓ Vercel  │ ✓ Nx Cloud│ ✗         │ ✗           │
│  Affected commands  │ ✓         │ ✓         │ ✗         │ ✓           │
│  Parallelization    │ ✓         │ ✓         │ ✗         │ ✓           │
│  Task pipelines     │ ✓         │ ✓         │ ✗         │ ✗           │
│  Code generation    │ ✗         │ ✓ (Great) │ ✗         │ ✗           │
│  Plugins            │ Limited   │ Extensive │ ✗         │ ✗           │
│  Setup complexity   │ Simple    │ Medium    │ Minimal   │ Simple      │
│  Learning curve     │ Low       │ Medium    │ Low       │ Low         │
│  Best for           │ Simple    │ Complex   │ Basic     │ Publishing  │
│                     │ monorepos │ monorepos │ linking   │ packages    │
│                                                                           │
│  WHEN TO CHOOSE:                                                         │
│  ─────────────────────────────────────────────────────────────────────── │
│  Turborepo → You want fast setup, great caching, Vercel deployment      │
│  Nx        → You need generators, plugins, complex project structures    │
│  pnpm      → You just need workspace linking, no build orchestration    │
│  Lerna     → You're publishing multiple npm packages (use with Nx)      │
│                                                                           │
└──────────────────────────────────────────────────────────────────────────┘
```

### Setting Up Each Tool

```bash
# ══════════════════════════════════════════════════════════════════
# TURBOREPO SETUP
# ══════════════════════════════════════════════════════════════════

# Create new monorepo
npx create-turbo@latest

# Or add to existing repo
npm install turbo --save-dev
# Then create turbo.json

# ══════════════════════════════════════════════════════════════════
# NX SETUP
# ══════════════════════════════════════════════════════════════════

# Create new Nx workspace
npx create-nx-workspace@latest

# Add Nx to existing monorepo
npx nx@latest init

# ══════════════════════════════════════════════════════════════════
# PNPM WORKSPACES SETUP
# ══════════════════════════════════════════════════════════════════

# Initialize
pnpm init

# Create pnpm-workspace.yaml
echo "packages:\n  - 'apps/*'\n  - 'packages/*'" > pnpm-workspace.yaml

# Install dependencies
pnpm install
```

---

## 3. Workspace Management

### Recommended Folder Structure

```
monorepo/
├── apps/                          # Deployable applications
│   ├── web/                       # Next.js frontend
│   │   ├── src/
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── next.config.js
│   │
│   ├── api/                       # Express/Fastify backend
│   │   ├── src/
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── mobile/                    # React Native app
│   │   └── ...
│   │
│   └── docs/                      # Documentation site
│       └── ...
│
├── packages/                      # Shared libraries
│   ├── ui/                        # Shared UI components
│   │   ├── src/
│   │   │   ├── Button/
│   │   │   ├── Input/
│   │   │   └── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── utils/                     # Shared utilities
│   │   ├── src/
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── config/                    # Shared configs
│   │   ├── eslint/
│   │   ├── typescript/
│   │   └── tailwind/
│   │
│   ├── types/                     # Shared TypeScript types
│   │   ├── src/
│   │   └── package.json
│   │
│   └── database/                  # Database client & schemas
│       ├── src/
│       ├── prisma/
│       └── package.json
│
├── tooling/                       # Build & dev tooling
│   ├── scripts/
│   └── generators/
│
├── turbo.json                     # Turborepo config
├── package.json                   # Root package.json
├── pnpm-workspace.yaml            # Workspace definition
├── tsconfig.base.json             # Base TypeScript config
└── .eslintrc.js                   # Root ESLint config
```

### Package Configuration

```json
// packages/ui/package.json
{
  "name": "@myorg/ui",
  "version": "0.0.0",
  "private": true,
  "main": "./dist/index.js",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.mjs",
      "require": "./dist/index.js"
    },
    "./Button": {
      "types": "./dist/Button/index.d.ts",
      "import": "./dist/Button/index.mjs",
      "require": "./dist/Button/index.js"
    }
  },
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts",
    "dev": "tsup src/index.ts --format cjs,esm --dts --watch",
    "lint": "eslint src/",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "@myorg/utils": "workspace:*"
  },
  "peerDependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@myorg/config": "workspace:*",
    "tsup": "^8.0.0",
    "typescript": "^5.0.0"
  }
}
```

```json
// apps/web/package.json
{
  "name": "@myorg/web",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "@myorg/ui": "workspace:*",
    "@myorg/utils": "workspace:*",
    "@myorg/types": "workspace:*",
    "next": "^14.0.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@myorg/config": "workspace:*",
    "typescript": "^5.0.0"
  }
}
```

### Shared Configuration Packages

```typescript
// packages/config/eslint/index.js
module.exports = {
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'prettier'
  ],
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint'],
  rules: {
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/no-explicit-any': 'warn'
  }
};

// packages/config/eslint/next.js
module.exports = {
  extends: [
    './index.js',
    'next/core-web-vitals'
  ],
  rules: {
    '@next/next/no-html-link-for-pages': 'off'
  }
};

// packages/config/eslint/react.js
module.exports = {
  extends: [
    './index.js',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended'
  ],
  settings: {
    react: {
      version: 'detect'
    }
  }
};
```

```json
// packages/config/typescript/base.json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "declaration": true,
    "declarationMap": true
  }
}

// packages/config/typescript/nextjs.json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "./base.json",
  "compilerOptions": {
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "noEmit": true,
    "module": "esnext",
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }]
  }
}

// packages/config/typescript/library.json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "./base.json",
  "compilerOptions": {
    "lib": ["es2022"],
    "module": "esnext",
    "target": "es2022",
    "outDir": "./dist"
  }
}
```

```json
// apps/web/tsconfig.json - Using shared config
{
  "extends": "@myorg/config/typescript/nextjs.json",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}

// packages/ui/tsconfig.json
{
  "extends": "@myorg/config/typescript/library.json",
  "compilerOptions": {
    "rootDir": "./src",
    "outDir": "./dist"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Internal Package Imports

```typescript
// ════════════════════════════════════════════════════════════════
// HOW TO USE INTERNAL PACKAGES
// ════════════════════════════════════════════════════════════════

// In apps/web/src/app/page.tsx
import { Button, Card, Input } from '@myorg/ui';
import { formatDate, debounce } from '@myorg/utils';
import type { User, Order } from '@myorg/types';

export default function Home() {
  return (
    <Card>
      <h1>Welcome</h1>
      <Button onClick={() => console.log('clicked')}>
        Click me
      </Button>
    </Card>
  );
}

// ════════════════════════════════════════════════════════════════
// PACKAGE EXPORTS SETUP (for tree-shaking)
// ════════════════════════════════════════════════════════════════

// packages/ui/src/index.ts
export { Button } from './Button';
export { Card } from './Card';
export { Input } from './Input';
export type { ButtonProps, CardProps, InputProps } from './types';

// Or with barrel exports per component
export * from './Button';
export * from './Card';

// ════════════════════════════════════════════════════════════════
// HANDLING STYLES IN SHARED PACKAGES
// ════════════════════════════════════════════════════════════════

// Option 1: CSS-in-JS (no extra setup)
// packages/ui/src/Button/Button.tsx
import styled from 'styled-components';

export const Button = styled.button`
  padding: 8px 16px;
  background: blue;
`;

// Option 2: Tailwind (consumer must include config)
// packages/ui/src/Button/Button.tsx
export function Button({ children }) {
  return (
    <button className="px-4 py-2 bg-blue-500 text-white rounded">
      {children}
    </button>
  );
}

// Consumer's tailwind.config.js must include package path:
// content: [
//   './src/**/*.{js,ts,jsx,tsx}',
//   '../../packages/ui/src/**/*.{js,ts,jsx,tsx}'  // Include UI package
// ]
```

---

## 4. Build Optimization

### How Caching Works

```
TURBOREPO CACHING:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  CACHE KEY CALCULATION:                                         │
│  ─────────────────────                                          │
│  Hash = hash(                                                   │
│    source files          (packages/ui/src/**)                  │
│    + dependencies        (package.json, lock file)             │
│    + environment vars    (NODE_ENV, etc.)                      │
│    + task config         (turbo.json pipeline)                 │
│    + global deps         (globalDependencies in turbo.json)    │
│  )                                                              │
│                                                                  │
│  CACHE HIT FLOW:                                                │
│  ──────────────                                                 │
│  1. Calculate hash for task                                    │
│  2. Check local cache (~/.turbo)                               │
│  3. If miss, check remote cache (Vercel/custom)               │
│  4. If hit, download and replay outputs                        │
│  5. If miss, execute task and cache result                     │
│                                                                  │
│  WHAT GETS CACHED:                                              │
│  ─────────────────                                              │
│  - Output files (dist/**, .next/**)                            │
│  - Terminal output (console logs)                              │
│  - Exit code                                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```json
// turbo.json - Optimized caching configuration
{
  "$schema": "https://turbo.build/schema.json",
  
  // Files that affect ALL tasks (changes invalidate everything)
  "globalDependencies": [
    ".env",
    "tsconfig.base.json"
  ],
  
  // Environment variables that affect all tasks
  "globalEnv": ["NODE_ENV", "CI"],
  
  "pipeline": {
    "build": {
      // ^build means: build dependencies first
      "dependsOn": ["^build"],
      
      // What files to cache
      "outputs": ["dist/**", ".next/**", "build/**"],
      
      // Environment variables that affect this task's cache
      "env": ["NEXT_PUBLIC_API_URL"],
      
      // Files that affect this task (beyond source)
      "inputs": ["$TURBO_DEFAULT$", ".env.production"]
    },
    
    "test": {
      // Depends on build being done first
      "dependsOn": ["build"],
      
      // Tests don't produce cacheable outputs
      "outputs": [],
      
      // But we can still cache the execution
      "cache": true
    },
    
    "lint": {
      // No dependencies, can run in parallel with everything
      "outputs": [],
      "cache": true
    },
    
    "dev": {
      // Never cache dev server
      "cache": false,
      "persistent": true
    }
  }
}
```

### Remote Caching Setup

```bash
# ══════════════════════════════════════════════════════════════════
# TURBOREPO REMOTE CACHING (Vercel)
# ══════════════════════════════════════════════════════════════════

# Login to Vercel (one-time)
npx turbo login

# Link to your Vercel team
npx turbo link

# Now all builds will use remote cache
turbo run build

# In CI, use environment variables
# TURBO_TOKEN: Your Vercel token
# TURBO_TEAM: Your team slug

# ══════════════════════════════════════════════════════════════════
# SELF-HOSTED REMOTE CACHE
# ══════════════════════════════════════════════════════════════════

# turbo.json
{
  "remoteCache": {
    "signature": true  // Verify cache integrity
  }
}

# Set cache server
# TURBO_API=https://your-cache-server.com
# TURBO_TOKEN=your-token
# TURBO_TEAM=your-team
```

```yaml
# .github/workflows/ci.yml - CI with remote caching
name: CI

on:
  push:
    branches: [main]
  pull_request:

env:
  TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
  TURBO_TEAM: ${{ vars.TURBO_TEAM }}

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
      
      - run: pnpm install
      
      # This will use remote cache!
      - run: pnpm turbo run build test lint
```

### Affected Commands

```bash
# ══════════════════════════════════════════════════════════════════
# RUN ONLY AFFECTED PACKAGES
# ══════════════════════════════════════════════════════════════════

# TURBOREPO: Filter by git changes
# Build packages affected since main
turbo run build --filter=...[origin/main]

# Build packages affected since last commit
turbo run build --filter=...[HEAD^1]

# Build specific package and everything that depends on it
turbo run build --filter=@myorg/ui...

# Build package and its dependencies
turbo run build --filter=...@myorg/web

# ══════════════════════════════════════════════════════════════════
# NX: Affected commands
# ══════════════════════════════════════════════════════════════════

# Run build on affected projects
nx affected -t build

# Run multiple targets on affected
nx affected -t build,test,lint

# Affected since specific commit
nx affected -t build --base=origin/main

# Show affected projects
nx show projects --affected

# View affected graph
nx affected:graph
```

### Parallelization

```
TASK EXECUTION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  DEPENDENCY GRAPH:                                              │
│  ─────────────────                                              │
│                                                                  │
│         @myorg/types                                            │
│              │                                                   │
│         @myorg/utils                                            │
│           /     \                                                │
│     @myorg/ui   @myorg/api                                      │
│          \       /                                               │
│         @myorg/web                                              │
│                                                                  │
│  EXECUTION (with dependsOn: ["^build"]):                       │
│  ─────────────────────────────────────────                      │
│                                                                  │
│  Time  │ Parallel Execution                                     │
│  ──────┼─────────────────────────────────────────────          │
│  T1    │ [types] ──────────────────────────────────►           │
│  T2    │          [utils] ─────────────────────────►           │
│  T3    │                   [ui] ───►  [api] ───────►           │
│  T4    │                                    [web] ──►          │
│                                                                  │
│  Without parallelization: 5 steps                               │
│  With parallelization: 4 steps (ui and api run together)       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```json
// turbo.json - Controlling parallelization
{
  "pipeline": {
    // Maximum concurrency (default: 10 or CPU cores)
    "build": {
      "dependsOn": ["^build"]
    },
    
    // Tasks with no dependencies run fully parallel
    "lint": {},
    "typecheck": {},
    
    // Force sequential (useful for resource-heavy tasks)
    "e2e": {
      "dependsOn": ["build"],
      // Can use concurrency: 1 via CLI
    }
  }
}
```

```bash
# Control parallelization
turbo run build --concurrency=4        # Max 4 parallel tasks
turbo run build --concurrency=50%      # 50% of CPU cores
turbo run build --concurrency=1        # Sequential (debugging)
```

### Incremental Builds

```typescript
// ════════════════════════════════════════════════════════════════
// TYPESCRIPT INCREMENTAL BUILDS
// ════════════════════════════════════════════════════════════════

// tsconfig.json
{
  "compilerOptions": {
    "incremental": true,           // Enable incremental compilation
    "tsBuildInfoFile": "./.tsbuildinfo"  // Cache file location
  }
}

// For project references (larger monorepos)
// tsconfig.json (root)
{
  "files": [],
  "references": [
    { "path": "./packages/utils" },
    { "path": "./packages/ui" },
    { "path": "./apps/web" }
  ]
}

// Build with references
// tsc --build --incremental

// ════════════════════════════════════════════════════════════════
// NEXT.JS INCREMENTAL BUILDS
// ════════════════════════════════════════════════════════════════

// next.config.js
module.exports = {
  // Faster builds in development
  swcMinify: true,
  
  // Transpile workspace packages
  transpilePackages: ['@myorg/ui', '@myorg/utils'],
  
  // Experimental features
  experimental: {
    // Turbopack for faster dev builds
    turbo: {
      rules: {
        '*.svg': ['@svgr/webpack']
      }
    }
  }
};
```

---

## 5. CI/CD Strategies

### GitHub Actions Setup

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

env:
  TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
  TURBO_TEAM: ${{ vars.TURBO_TEAM }}

jobs:
  build:
    name: Build and Test
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 2  # Need history for affected commands
      
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Build
        run: pnpm turbo run build --filter=...[origin/main]
      
      - name: Test
        run: pnpm turbo run test --filter=...[origin/main]
      
      - name: Lint
        run: pnpm turbo run lint --filter=...[origin/main]
      
      - name: Type check
        run: pnpm turbo run typecheck --filter=...[origin/main]

  # Separate job for E2E tests (only affected apps)
  e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: build
    
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
        with:
          version: 8
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
      
      - run: pnpm install
      - run: pnpm turbo run build --filter=@myorg/web
      
      - name: Run E2E
        run: pnpm --filter @myorg/web run e2e
```

### Deploy Only Changed Apps

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      web: ${{ steps.changes.outputs.web }}
      api: ${{ steps.changes.outputs.api }}
      docs: ${{ steps.changes.outputs.docs }}
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 2
      
      - name: Check for changes
        id: changes
        run: |
          # Get changed files
          CHANGED=$(git diff --name-only HEAD~1 HEAD)
          
          # Check if web app or its dependencies changed
          if echo "$CHANGED" | grep -qE "^(apps/web|packages/)"; then
            echo "web=true" >> $GITHUB_OUTPUT
          else
            echo "web=false" >> $GITHUB_OUTPUT
          fi
          
          # Check if api changed
          if echo "$CHANGED" | grep -qE "^(apps/api|packages/)"; then
            echo "api=true" >> $GITHUB_OUTPUT
          else
            echo "api=false" >> $GITHUB_OUTPUT
          fi

  deploy-web:
    needs: detect-changes
    if: needs.detect-changes.outputs.web == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pnpm install
      - run: pnpm turbo run build --filter=@myorg/web
      - name: Deploy to Vercel
        run: vercel deploy --prod

  deploy-api:
    needs: detect-changes
    if: needs.detect-changes.outputs.api == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pnpm install
      - run: pnpm turbo run build --filter=@myorg/api
      - name: Deploy to AWS
        run: ./scripts/deploy-api.sh
```

### Versioning Strategies

```
VERSIONING APPROACHES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. FIXED/UNIFIED VERSIONING                                    │
│     └── All packages have the same version                     │
│     └── Simple, like Angular's approach                        │
│     └── One changelog for entire monorepo                      │
│     └── Good for: tightly coupled packages                     │
│                                                                  │
│  2. INDEPENDENT VERSIONING                                      │
│     └── Each package versioned independently                   │
│     └── More flexible, like Babel's approach                   │
│     └── Per-package changelogs                                 │
│     └── Good for: published npm packages                       │
│     └── Tools: Changesets, Lerna                               │
│                                                                  │
│  3. NO VERSIONING (Internal Only)                              │
│     └── Packages never published to npm                        │
│     └── All packages at version "0.0.0"                        │
│     └── Good for: private monorepos                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# ══════════════════════════════════════════════════════════════════
# CHANGESETS (for independent versioning)
# ══════════════════════════════════════════════════════════════════

# Install
pnpm add -Dw @changesets/cli

# Initialize
pnpm changeset init

# Create a changeset (interactive)
pnpm changeset

# Version packages (bumps versions, updates changelogs)
pnpm changeset version

# Publish to npm
pnpm changeset publish
```

---

## 6. Common Patterns

### Shared Database Package

```typescript
// packages/database/src/client.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}

// packages/database/src/index.ts
export * from '@prisma/client';
export { prisma } from './client';

// Usage in apps/api
import { prisma, User } from '@myorg/database';
const users = await prisma.user.findMany();
```

### Shared API Types

```typescript
// packages/types/src/api.ts
export interface ApiResponse<T> {
  data: T;
  error?: string;
}

export interface User {
  id: string;
  email: string;
  name: string;
}

// apps/api and apps/web both import from @myorg/types
// Ensures frontend and backend stay in sync
```

### Feature Flags Package

```typescript
// packages/feature-flags/src/index.ts
type FeatureFlags = {
  newDashboard: boolean;
  darkMode: boolean;
};

class FeatureFlagService {
  private flags: FeatureFlags = { newDashboard: false, darkMode: true };
  
  isEnabled(flag: keyof FeatureFlags): boolean {
    return this.flags[flag] ?? false;
  }
}

export const featureFlags = new FeatureFlagService();
```

---

## 7. Scaling & Performance

### Large Monorepo Challenges

```
SCALING CHALLENGES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  PROBLEM                    │ SOLUTION                          │
│  ─────────────────────────────────────────────────────────────  │
│  Slow git clone             │ Shallow clone, sparse checkout   │
│  Long install times         │ pnpm (hardlinks), remote cache   │
│  CI taking forever          │ Affected commands, parallelization│
│  IDE performance            │ Focus on specific packages        │
│  Too many packages          │ Consider splitting monorepo       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# ══════════════════════════════════════════════════════════════════
# SPARSE CHECKOUT (Clone only what you need)
# ══════════════════════════════════════════════════════════════════

# Clone without checking out files
git clone --filter=blob:none --sparse https://github.com/org/monorepo.git
cd monorepo

# Checkout only specific directories
git sparse-checkout set apps/web packages/ui packages/utils

# Add more directories as needed
git sparse-checkout add apps/api

# ══════════════════════════════════════════════════════════════════
# SHALLOW CLONE (Limited history)
# ══════════════════════════════════════════════════════════════════

# Clone only recent history (for CI)
git clone --depth=1 https://github.com/org/monorepo.git

# For affected commands, need a bit more history
git clone --depth=10 https://github.com/org/monorepo.git
```

### When to Split a Monorepo

```
SIGNS YOU NEED TO SPLIT:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. BUILD TIMES UNMANAGEABLE                                    │
│     └── Even with caching, builds take >30 minutes             │
│     └── Affected commands still touch too much                 │
│                                                                  │
│  2. TEAM INDEPENDENCE NEEDED                                    │
│     └── Teams want different release cycles                    │
│     └── Different security/compliance requirements             │
│     └── Organizational boundaries are clear                    │
│                                                                  │
│  3. TOOLING CONFLICTS                                           │
│     └── Different Node versions needed                         │
│     └── Conflicting dependency versions                        │
│     └── Different CI/CD pipelines                              │
│                                                                  │
│  4. SCALE LIMITS                                                │
│     └── 500+ packages becoming unmanageable                   │
│     └── IDE/editor can't handle codebase                       │
│     └── Git operations too slow                                │
│                                                                  │
│  SPLITTING STRATEGIES:                                         │
│  └── By domain (billing repo, user repo)                      │
│  └── By team (team-a-repo, team-b-repo)                       │
│  └── By deployment (frontend repo, backend repo)              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. When to Use / Not Use

### When TO Use a Monorepo

```
✅ USE MONOREPO WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. SHARED CODE BETWEEN PROJECTS                               │
│     └── Common UI components                                   │
│     └── Shared utilities/types                                 │
│     └── Shared configuration                                   │
│                                                                  │
│  2. ATOMIC CHANGES NEEDED                                       │
│     └── API change + frontend update in one PR                │
│     └── Library update + all consumers                        │
│     └── Refactoring across packages                           │
│                                                                  │
│  3. UNIFIED TOOLING DESIRED                                     │
│     └── Same ESLint/TypeScript config everywhere              │
│     └── Consistent CI/CD                                       │
│     └── Single dependency management                          │
│                                                                  │
│  4. SAME TEAM/ORG OWNS EVERYTHING                              │
│     └── Full-stack team                                        │
│     └── Same release cycle                                     │
│     └── Shared ownership                                       │
│                                                                  │
│  GOOD EXAMPLES:                                                 │
│  └── Full-stack app (web + api + shared)                      │
│  └── Design system + consuming apps                           │
│  └── Microservices with shared types                          │
│  └── SDK + examples + docs                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When NOT to Use a Monorepo

```
❌ DON'T USE MONOREPO WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. INDEPENDENT TEAMS/ORGS                                      │
│     └── Different release cycles                               │
│     └── Different ownership                                    │
│     └── Different security requirements                        │
│                                                                  │
│  2. NO SHARED CODE                                              │
│     └── Completely independent projects                        │
│     └── Different tech stacks                                  │
│     └── No code reuse benefit                                  │
│                                                                  │
│  3. OPEN SOURCE LIBRARIES                                       │
│     └── External contributors                                  │
│     └── Separate versioning important                          │
│     └── Independent discoverability                           │
│                                                                  │
│  4. LEGACY INTEGRATION                                          │
│     └── Existing repos with complex history                   │
│     └── Different VCS (git + svn)                             │
│     └── Compliance/audit requirements                         │
│                                                                  │
│  5. SCALE CONCERNS                                              │
│     └── Thousands of developers                               │
│     └── Cannot invest in tooling                              │
│     └── CI infrastructure limitations                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Interview Questions & Answers

### Basic Questions

**Q1: What is a monorepo?**
> **A:** A single repository containing multiple projects (apps, packages) that share dependencies and tooling. Enables code reuse, atomic changes across packages, and unified CI/CD. Tools like Turborepo or Nx make it efficient with caching and affected commands.

**Q2: Monorepo vs polyrepo - when to choose which?**
> **A:** 
> **Monorepo**: When teams share code frequently, need atomic changes across packages, want unified tooling. Example: full-stack app with shared types.
>
> **Polyrepo**: When teams are independent, different release cycles, no shared code, or external contributors. Example: separate microservices owned by different teams.

**Q3: What is the workspace protocol?**
> **A:** The `workspace:*` syntax in package.json that links to local packages instead of npm versions. When published, it's replaced with actual version. Enables seamless local development without npm publish.

**Q4: What is remote caching?**
> **A:** Shared build cache across team and CI. First person builds, cache is uploaded. Others download cached outputs instead of rebuilding. Can make builds 10-100x faster. Provided by Vercel (Turborepo) or Nx Cloud.

### Intermediate Questions

**Q5: How do affected commands work?**
> **A:** 
> 1. Compare git diff to find changed files
> 2. Map files to packages they belong to
> 3. Use dependency graph to find all packages that depend on changed ones
> 4. Only build/test those packages
>
> Example: If `utils` changed, rebuild `utils`, `ui` (depends on utils), and `web` (depends on ui). Skip `docs` (not affected).

**Q6: Turborepo vs Nx - how do you choose?**
> **A:**
> **Turborepo**: Simple setup, excellent caching, minimal config. Best for: smaller monorepos, Vercel users, teams wanting simplicity.
>
> **Nx**: More features (generators, plugins, visualization), steeper learning curve. Best for: large complex monorepos, teams needing code generation, enterprise features.

**Q7: How do you handle versioning in a monorepo?**
> **A:** Three approaches:
> 1. **Fixed**: All packages same version (Angular style) - simple
> 2. **Independent**: Each package versioned separately (Changesets) - flexible
> 3. **None**: Internal only, all at "0.0.0" - private monorepos
>
> Use Changesets for independent versioning with automated changelogs.

**Q8: How does caching work in Turborepo?**
> **A:** 
> 1. Hash inputs: source files + dependencies + config + env vars
> 2. Check local cache (~/.turbo) for matching hash
> 3. If miss, check remote cache (Vercel)
> 4. If hit, download and replay outputs
> 5. If miss, execute task, cache outputs
>
> Outputs include files (dist/**) and terminal output.

### Advanced Questions

**Q9: How do you optimize CI for a monorepo?**
> **A:**
> 1. **Affected commands**: Only build/test changed packages
> 2. **Remote caching**: Share cache between CI runs
> 3. **Parallelization**: Run independent tasks simultaneously
> 4. **Shallow clone**: `--depth=10` for faster checkout
> 5. **Selective deployment**: Only deploy apps that changed
>
> Combined, these can reduce CI from 45 minutes to under 5 minutes.

**Q10: How do you handle shared dependencies?**
> **A:**
> - Common deps in root package.json (dev tools: TypeScript, ESLint)
> - Package-specific deps in package's package.json
> - Use `pnpm` for efficient deduplication (hardlinks)
> - Shared config packages for ESLint, TypeScript, Tailwind configs
> - `workspace:*` protocol for internal dependencies

**Q11: What are the scaling limits of monorepos?**
> **A:**
> - **Recommended**: <500 packages
> - **Challenges at scale**: Git operations slow, IDE struggles, CI complexity
> - **Solutions**: Sparse checkout (clone subset), remote caching, affected commands
> - **When to split**: Different teams, different release cycles, >500 packages, unmanageable build times

**Q12: How do you manage releases in a monorepo?**
> **A:**
> Using Changesets:
> 1. Developers add changeset files describing changes
> 2. CI collects changesets on merge to main
> 3. Changesets action creates Release PR with version bumps
> 4. Merging Release PR publishes to npm
>
> Supports independent versioning with auto-generated changelogs.

### Scenario Questions

**Q13: You have a monorepo with web, api, and shared packages. How do you set it up?**
> **A:**
> ```
> monorepo/
> ├── apps/
> │   ├── web/        # Next.js
> │   └── api/        # Express
> ├── packages/
> │   ├── ui/         # Shared components
> │   ├── utils/      # Shared utilities
> │   └── types/      # Shared TypeScript types
> ├── turbo.json      # Task pipeline
> └── pnpm-workspace.yaml
> ```
>
> - Apps import packages via `@myorg/ui`, `@myorg/utils`
> - `turbo.json` defines build order (packages before apps)
> - Shared TypeScript config in `packages/config`
> - Remote caching enabled for faster CI

**Q14: CI is taking 30 minutes. How do you optimize?**
> **A:**
> 1. **Enable remote caching**: First build caches, subsequent runs hit cache
> 2. **Use affected commands**: `turbo run build --filter=...[origin/main]`
> 3. **Parallelize**: Ensure turbo.json has proper `dependsOn` for parallelization
> 4. **Shallow clone**: `--depth=10` in CI checkout
> 5. **Split large tests**: E2E in separate job, only for affected apps
>
> Expected result: 30 min → 3-5 min average.

---

## 🎓 Key Takeaways

1. **Monorepo = code sharing + atomic changes + unified tooling**
2. **Caching** is the #1 optimization - 10-100x speedup
3. **Affected commands** - only build what changed
4. **Turborepo** for simplicity, **Nx** for features
5. **Remote caching** shares builds across team and CI
6. **workspace:*** protocol links local packages
7. **Shared config packages** for consistency
8. **CI optimization**: affected + caching + parallelization
9. **Changesets** for independent versioning
10. **Know when to split** - >500 packages, independent teams

---

## 📚 Resources

### Documentation
- [Turborepo Docs](https://turbo.build/repo/docs)
- [Nx Docs](https://nx.dev/getting-started/intro)
- [pnpm Workspaces](https://pnpm.io/workspaces)
- [Changesets](https://github.com/changesets/changesets)

### Tools
- [Turborepo](https://turbo.build/)
- [Nx](https://nx.dev/)
- [pnpm](https://pnpm.io/)
- [Lerna](https://lerna.js.org/) (with Nx)

### Examples
- [Turborepo Examples](https://github.com/vercel/turbo/tree/main/examples)
- [Nx Examples](https://github.com/nrwl/nx-examples)


