# CI/CD Concepts

> Understanding Continuous Integration and Deployment for CareCircle.

---

## 1. What Is CI/CD?

### Plain English Explanation

CI/CD automates the process of **testing and deploying your code**.

Think of it like a **factory assembly line**:
- Code changes enter the line
- Automated checks verify quality
- Approved changes roll out to production
- No manual intervention needed

### The Core Problem CI/CD Solves

```
WITHOUT CI/CD:
──────────────
Developer: "I'll test it locally, looks good"
*Pushes directly to production*
*App crashes at 3 AM*
*Panic ensues*

WITH CI/CD:
───────────
Developer pushes code
→ Automated tests run
→ Code review required
→ Staging deployment
→ Production deployment
*Sleep peacefully*
```

---

## 2. Core Concepts & Terminology

### CI vs CD

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CI/CD PIPELINE                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  CONTINUOUS INTEGRATION (CI)                                                 │
│  ───────────────────────────                                                │
│  • Merge code frequently (multiple times/day)                               │
│  • Run automated tests on every push                                        │
│  • Catch bugs early, before they spread                                     │
│  • Build artifacts (Docker images)                                          │
│                                                                              │
│  CONTINUOUS DELIVERY (CD)                                                    │
│  ────────────────────────                                                   │
│  • Automatically deploy to staging                                          │
│  • Manual approval for production                                           │
│  • Always have deployable code                                              │
│                                                                              │
│  CONTINUOUS DEPLOYMENT (CD)                                                  │
│  ─────────────────────────                                                  │
│  • Automatically deploy to production                                       │
│  • No manual approval needed                                                │
│  • Requires strong test coverage                                            │
│                                                                              │
│  CareCircle uses: CI + Continuous DELIVERY (manual prod approval)           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Terminology

| Term | Definition |
|------|------------|
| **Pipeline** | Automated workflow of jobs |
| **Job** | Single task (test, build, deploy) |
| **Stage** | Group of jobs that run together |
| **Artifact** | Output of a job (build files, images) |
| **Runner** | Server that executes jobs |
| **Trigger** | Event that starts pipeline (push, PR) |

---

## 3. Pipeline Stages

### Typical Pipeline Flow

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│    COMMIT         BUILD          TEST           DEPLOY         MONITOR       │
│       │              │              │              │              │          │
│       ▼              ▼              ▼              ▼              ▼          │
│   ┌───────┐     ┌───────┐     ┌───────┐     ┌───────┐     ┌───────┐        │
│   │ Push  │ ──► │ Lint  │ ──► │ Unit  │ ──► │Staging│ ──► │Health │        │
│   │ Code  │     │ Build │     │ Tests │     │ Deploy│     │ Check │        │
│   └───────┘     └───────┘     └───────┘     └───────┘     └───────┘        │
│                      │              │              │                         │
│                      ▼              ▼              ▼                         │
│                 ┌───────┐     ┌───────┐     ┌───────┐                       │
│                 │Docker │     │  E2E  │     │  Prod │  (manual approval)    │
│                 │ Image │     │ Tests │     │Deploy │                       │
│                 └───────┘     └───────┘     └───────┘                       │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

### What Happens at Each Stage

| Stage | Actions | On Failure |
|-------|---------|------------|
| **Lint** | ESLint, TypeScript check | Block merge |
| **Build** | Compile, bundle, Docker build | Block merge |
| **Unit Test** | Jest tests | Block merge |
| **E2E Test** | Playwright/Cypress | Block merge |
| **Staging Deploy** | Deploy to staging env | Alert team |
| **Prod Deploy** | Deploy to production | Rollback |

---

## 4. GitHub Actions (Example)

### Basic Workflow Structure

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - run: pnpm install
      - run: pnpm lint

  test:
    runs-on: ubuntu-latest
    needs: lint  # Runs after lint passes
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v2
      - run: pnpm install
      - run: pnpm test

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t carecircle-api ./apps/api
      - run: docker build -t carecircle-web ./apps/web

  deploy-staging:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    steps:
      - run: echo "Deploy to staging"

  deploy-production:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    environment: production  # Requires approval
    steps:
      - run: echo "Deploy to production"
```

---

## 5. When to Use CI/CD Patterns ✅

### Always Run on Every Push:
- Linting (code style)
- Type checking
- Unit tests
- Security scans

### Run on Pull Requests:
- Integration tests
- E2E tests (subset)
- Preview deployments

### Run on Merge to Main:
- Full test suite
- Docker image build
- Production deployment

---

## 6. When to AVOID Patterns ❌

### DON'T Skip Tests to Deploy Faster

```yaml
# ❌ BAD: Skip tests
deploy:
  steps:
    - run: deploy-to-production.sh

# ✅ GOOD: Tests are mandatory
deploy:
  needs: [lint, test, build]  # Must pass first
  steps:
    - run: deploy-to-production.sh
```

### DON'T Store Secrets in Code

```yaml
# ❌ BAD: Hardcoded secret
env:
  DATABASE_URL: postgres://user:password@host/db

# ✅ GOOD: Use secrets
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

---

## 7. Best Practices

### Pipeline Design

```
1. FAIL FAST
   Run quick checks first (lint before build)
   
2. PARALLELIZE
   Run independent jobs simultaneously
   
3. CACHE AGGRESSIVELY
   Cache dependencies, Docker layers
   
4. USE MATRIX BUILDS
   Test multiple Node versions, OS
   
5. REQUIRE APPROVALS
   Manual gate for production
```

### Caching Strategy

```yaml
# Cache pnpm dependencies
- uses: actions/cache@v3
  with:
    path: ~/.pnpm-store
    key: ${{ runner.os }}-pnpm-${{ hashFiles('**/pnpm-lock.yaml') }}
    restore-keys: |
      ${{ runner.os }}-pnpm-
```

### Environment Strategy

```
develop branch → Staging environment (auto-deploy)
main branch → Production environment (manual approval)
feature/* branches → Preview environments (optional)
```

---

## 8. Common Mistakes & How to Avoid Them

### Mistake 1: Long-Running Pipelines

```
❌ 30+ minute pipelines
   Developers avoid pushing, batch changes

✅ Target: <10 minutes for PR checks
   Parallelize, cache, skip unnecessary steps
```

### Mistake 2: Flaky Tests

```
❌ Tests that randomly fail
   Team ignores failures, loses trust

✅ Fix or remove flaky tests immediately
   Quarantine while investigating
```

### Mistake 3: No Rollback Plan

```
❌ Deploy and pray
   
✅ Automated rollback on health check failure
   Keep previous version available
   Blue-green or canary deployments
```

---

## 9. Deployment Strategies

### Rolling Deployment

```
Old: [A] [A] [A] [A]
     [A] [A] [B] [A]  ← Replace one at a time
     [A] [B] [B] [A]
     [B] [B] [B] [B]
New: [B] [B] [B] [B]

✅ Zero downtime
❌ Mixed versions during rollout
```

### Blue-Green Deployment

```
Blue (current): [A] [A] [A] [A] ← Serving traffic
Green (new):    [B] [B] [B] [B] ← Ready, not serving

*Switch load balancer*

Blue (old):     [A] [A] [A] [A] ← Standby for rollback
Green (current):[B] [B] [B] [B] ← Now serving traffic

✅ Instant rollback
❌ Requires 2x resources
```

---

## 10. Quick Reference

### GitHub Actions Syntax

```yaml
on:                    # Triggers
  push:
  pull_request:
  schedule:
  workflow_dispatch:   # Manual trigger

jobs:                  # Define jobs
  job-name:
    runs-on: ubuntu-latest
    needs: [other-job]  # Dependencies
    if: condition       # Conditional
    steps:
      - uses: action@v1 # Use action
      - run: command    # Run command
      - env:            # Environment
          KEY: value
```

### Useful Actions

| Action | Purpose |
|--------|---------|
| `actions/checkout@v4` | Clone repo |
| `actions/cache@v3` | Cache dependencies |
| `actions/upload-artifact@v3` | Save build output |
| `docker/build-push-action@v5` | Build & push Docker |

---

*Next: [Docker](docker.md) | [Infrastructure Overview](_INFRASTRUCTURE_OVERVIEW.md)*

