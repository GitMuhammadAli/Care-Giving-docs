# Monorepo Structure

> Understanding CareCircle's code organization philosophy.

---

## 1. What Is a Monorepo?

### Plain English Explanation

A monorepo is a **single repository containing multiple projects** that may or may not be related.

Think of it like a **shopping mall**:
- One building (repository)
- Multiple stores (apps/packages)
- Shared facilities (utilities, configs)
- One landlord (single Git history)

### The Core Problem It Solves

```
POLYREPO (Multiple Repositories):
─────────────────────────────────
  carecircle-api/       (separate repo)
  carecircle-web/       (separate repo)
  carecircle-workers/   (separate repo)
  carecircle-shared/    (separate repo)

Problems:
• Version mismatch between API and Web
• Publishing shared types as npm packages
• Coordinating releases across repos
• Running full-stack locally is complex


MONOREPO (Single Repository):
─────────────────────────────
  carecircle/
  ├── apps/api/
  ├── apps/web/
  ├── apps/workers/
  └── packages/shared/

Benefits:
• One PR updates API + Web atomically
• Shared types without publishing
• Single Git history, single CI/CD
• Easy to run everything locally
```

---

## 2. CareCircle's Structure

```
Care-Giving/
├── apps/
│   ├── api/                  # NestJS Backend
│   │   ├── src/
│   │   │   ├── auth/         # Authentication module
│   │   │   ├── family/       # Family management
│   │   │   ├── medications/  # Medication tracking
│   │   │   ├── ...           # Other feature modules
│   │   │   └── main.ts       # Entry point
│   │   ├── Dockerfile
│   │   └── package.json
│   │
│   ├── web/                  # Next.js Frontend
│   │   ├── src/
│   │   │   ├── app/          # App Router pages
│   │   │   ├── components/   # React components
│   │   │   ├── hooks/        # Custom hooks
│   │   │   └── lib/          # Utilities
│   │   ├── Dockerfile
│   │   └── package.json
│   │
│   └── workers/              # Background Jobs
│       ├── src/
│       │   ├── queues.ts     # Queue definitions
│       │   ├── scheduler.ts  # Cron jobs
│       │   └── index.ts      # Entry point
│       ├── Dockerfile
│       └── package.json
│
├── packages/
│   ├── database/             # Prisma schema & client
│   │   ├── prisma/
│   │   │   └── schema.prisma
│   │   └── package.json
│   │
│   ├── config/               # Shared configuration
│   │   └── package.json
│   │
│   └── logger/               # Shared logging
│       └── package.json
│
├── env/                      # Environment profiles
│   ├── base.env
│   ├── local.env
│   └── cloud.env
│
├── scripts/                  # Utility scripts
│   ├── use-local.ps1
│   └── use-cloud.ps1
│
├── docs/                     # Documentation
│   └── tech-stack/           # You are here!
│
├── pnpm-workspace.yaml       # Workspace definition
├── docker-compose.yml        # Local infrastructure
└── package.json              # Root scripts
```

---

## 3. Key Concepts

### Apps vs Packages

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         APPS vs PACKAGES                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  APPS (apps/)                         │  PACKAGES (packages/)               │
│  ────────────                         │  ──────────────────                 │
│                                       │                                     │
│  Deployable units                     │  Shared libraries                   │
│  Have entry points (main.ts)          │  No entry point                     │
│  Run independently                    │  Used by apps                       │
│  Build to Docker images               │  Built for consumption              │
│                                       │                                     │
│  Examples:                            │  Examples:                          │
│  • api (NestJS server)                │  • database (Prisma client)         │
│  • web (Next.js app)                  │  • config (env validation)          │
│  • workers (job processor)            │  • logger (shared logging)          │
│                                       │                                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Dependency Rules

```
                    ┌─────────────┐
                    │   apps/api  │ ─────────┐
                    └──────┬──────┘          │
                           │                 │
                           ▼                 │
                    ┌─────────────┐          │
                    │  apps/web   │ ─────────┤
                    └──────┬──────┘          │
                           │                 │
                           ▼                 │
                    ┌─────────────┐          │
                    │apps/workers │ ─────────┤
                    └─────────────┘          │
                                             │
                           ▼                 ▼
                    ┌─────────────────────────────┐
                    │        packages/*           │
                    │  (database, config, logger) │
                    └─────────────────────────────┘

ALLOWED:
  • Apps can import from packages
  • Packages can import from other packages

NOT ALLOWED:
  • Apps cannot import from other apps
  • Packages cannot import from apps
```

---

## 4. Why This Structure

| Benefit | Explanation |
|---------|-------------|
| **Atomic changes** | One PR updates API + Web together |
| **Shared types** | Prisma types used everywhere |
| **Consistent tooling** | Same ESLint, TypeScript config |
| **Simplified testing** | Integration tests across apps |
| **Single CI/CD** | One pipeline, one deployment |

### Trade-offs

| Pro | Con |
|-----|-----|
| Atomic commits | Larger repo size |
| Shared types | All changes in one place (noise) |
| Single CI/CD | Longer CI runs (mitigated with caching) |
| Easy refactoring | Requires discipline (no app cross-imports) |

---

## 5. pnpm Workspaces

### How It Works

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

This tells pnpm:
- `apps/api`, `apps/web`, `apps/workers` are workspace packages
- `packages/database`, `packages/config` are workspace packages
- They can reference each other using `workspace:*`

### Package References

```json
// apps/api/package.json
{
  "dependencies": {
    "@carecircle/database": "workspace:*",
    "@carecircle/config": "workspace:*"
  }
}
```

The `workspace:*` means "use the local version from this monorepo, don't download from npm."

---

## 6. Common Commands

### Running Apps

```bash
# Run specific app
pnpm --filter @carecircle/api dev
pnpm --filter @carecircle/web dev
pnpm --filter @carecircle/workers dev

# Run all apps
pnpm dev
```

### Building

```bash
# Build specific app
pnpm --filter @carecircle/api build

# Build all
pnpm build
```

### Adding Dependencies

```bash
# Add to specific app
pnpm --filter @carecircle/api add lodash

# Add workspace dependency
pnpm --filter @carecircle/api add @carecircle/database
```

---

## 7. Best Practices

### Keep Apps Independent

Apps should not import from each other:

```typescript
// ❌ BAD: apps/web importing from apps/api
import { CreateUserDto } from '../../api/src/users/dto';

// ✅ GOOD: apps/web imports from packages
import { UserSchema } from '@carecircle/database';
```

### Shared Code Goes in Packages

If code is used by multiple apps, extract to a package:

```
Before:
  apps/api/src/utils/date-utils.ts
  apps/web/src/lib/date-utils.ts    # Duplicate!

After:
  packages/utils/src/date-utils.ts  # Single source
```

### Each Package Has Clear Purpose

```
packages/
├── database/   # Prisma schema, client, types
├── config/     # Environment validation, config loading
└── logger/     # Shared logging configuration

NOT:
├── shared/     # Dumping ground for everything
├── utils/      # Vague, will grow forever
```

---

## 8. Quick Reference

### File Locations

| What | Where |
|------|-------|
| API controllers | `apps/api/src/*/` |
| Frontend pages | `apps/web/src/app/` |
| Worker jobs | `apps/workers/src/` |
| Database schema | `packages/database/prisma/schema.prisma` |
| Shared config | `packages/config/src/` |
| Environment files | `env/*.env` |

### Package Names

| Directory | Package Name |
|-----------|--------------|
| `apps/api` | `@carecircle/api` |
| `apps/web` | `@carecircle/web` |
| `apps/workers` | `@carecircle/workers` |
| `packages/database` | `@carecircle/database` |
| `packages/config` | `@carecircle/config` |

---

*Next: [Architecture Principles](principles.md) | [Backend Overview](../backend/_BACKEND_OVERVIEW.md)*

