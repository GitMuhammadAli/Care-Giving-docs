# CareCircle Documentation Index

> Your guide to finding the right documentation.

**Last Updated:** January 30, 2026

---

## Quick Navigation

| I want to... | Go to... |
|--------------|----------|
| Set up my local environment | [CARECIRCLE_HANDBOOK-2.md](CARECIRCLE_HANDBOOK-2.md) → Profile A Runbook |
| Deploy to production | [CARECIRCLE_HANDBOOK-2.md](CARECIRCLE_HANDBOOK-2.md) → Profile B Runbook |
| Understand the architecture | [tech-stack/README.md](tech-stack/README.md) |
| Learn about a specific technology | [tech-stack/](tech-stack/README.md) |
| Check what features are implemented | [features/COMPLETE_FEATURES_IMPLEMENTATION.md](features/COMPLETE_FEATURES_IMPLEMENTATION.md) |
| Debug a problem | [CARECIRCLE_HANDBOOK-2.md](CARECIRCLE_HANDBOOK-2.md) → Troubleshooting |
| Learn about workers & background jobs | [PRODUCTION_INFRASTRUCTURE_COMPLETE.md](PRODUCTION_INFRASTRUCTURE_COMPLETE.md) |
| Get started quickly | [getting-started/QUICK_START.md](getting-started/QUICK_START.md) |
| Deploy for free | [deployment/FREE_DEPLOYMENT_GUIDE.md](deployment/FREE_DEPLOYMENT_GUIDE.md) |

---

## Documentation Structure

```
docs/
├── DOCUMENTATION_INDEX.md          ← You are here!
│
├── CARECIRCLE_HANDBOOK-1.md        # Part 1: Overview, Architecture, Domain Model
├── CARECIRCLE_HANDBOOK-2.md        # Part 2: Auth, Runbooks, Troubleshooting, FAQ
├── PRODUCTION_INFRASTRUCTURE_COMPLETE.md  # Infrastructure & Workers
│
├── features/                       # Feature Documentation
│   └── COMPLETE_FEATURES_IMPLEMENTATION.md
│
├── tech-stack/                     # Conceptual Technology Documentation
│   ├── README.md                   # Overview & Architecture Diagram
│   ├── frontend/                   # React, Next.js, State Management
│   ├── backend/                    # NestJS, API Design, Auth
│   ├── database/                   # PostgreSQL, Prisma, Redis
│   ├── workers/                    # Background Jobs, BullMQ
│   ├── infrastructure/             # Docker, CI/CD
│   ├── architecture/               # Design Principles, Monorepo
│   ├── security/                   # OWASP, Security Principles
│   ├── testing/                    # Testing Philosophy, Unit Testing
│   └── concepts/                   # Error Handling, Caching, Real-time
│
├── getting-started/                # Onboarding
│   ├── QUICK_START.md
│   └── FREE_SERVICES_SETUP.md
│
├── deployment/                     # Deployment Guides
│   ├── DEPLOYMENT_CHECKLIST.md
│   ├── FREE_DEPLOYMENT_GUIDE.md
│   ├── ORACLE_CLOUD_FREE_TIER_GUIDE.md
│   └── PRODUCTION_DEPLOYMENT_GUIDE.md
│
├── guides/                         # How-To Guides
│   ├── AUTHENTICATION.md
│   ├── DOCKER_DEPLOYMENT.md
│   └── QA_TEST_REPORT.md
│
├── operations/                     # Operations Procedures
│   ├── BACKUP_IMPLEMENTATION_SUMMARY.md
│   └── BACKUP_PROCEDURES.md
│
├── testing/                        # Testing Documentation
│   └── TESTING.md
│
└── extras/                         # Non-Essential Documentation
    ├── historical/                 # Old reports, audit logs
    │   ├── CODEBASE_AUDIT_REPORT.md
    │   ├── FINAL_STATUS.md
    │   └── scans/                  # Historical scan reports
    │
    ├── learning/                   # Generic Learning Material
    │   ├── engineering-mastery/    # 14+ engineering deep-dives
    │   └── Senior-dev/             # 40+ senior dev guides
    │
    ├── superseded/                 # Replaced Documentation
    │   ├── TECHNOLOGY_STACK.md     # Replaced by tech-stack/
    │   ├── DEVOPS_ARCHITECTURE_MAP.md
    │   ├── STREAM_CHAT_SETUP.md
    │   └── architecture/           # Replaced by tech-stack/architecture
    │
    ├── misc/                       # Miscellaneous
    │   └── nova/
    │
    └── backups/                    # Backup Files
        └── *.backup
```

---

## Document Purposes

### Core Documentation (Essential)

| Document | Purpose |
|----------|---------|
| **CARECIRCLE_HANDBOOK-1.md** | System overview, architecture, domain model, authorization |
| **CARECIRCLE_HANDBOOK-2.md** | Auth flows, runbooks, troubleshooting, FAQ |
| **PRODUCTION_INFRASTRUCTURE_COMPLETE.md** | Deployment, workers, environment variables |
| **features/** | What's built, API endpoints, implementation status |
| **tech-stack/** | Deep conceptual understanding of each technology |

### Operational Documentation

| Folder | Purpose |
|--------|---------|
| **getting-started/** | New developer onboarding |
| **deployment/** | Production deployment guides |
| **guides/** | Specific how-to guides |
| **operations/** | Backup and maintenance procedures |
| **testing/** | Testing strategies |

### extras/ (Reference Only)

| Folder | Contents |
|--------|----------|
| **historical/** | Old audit reports and project status |
| **learning/** | Generic engineering education (not CareCircle-specific) |
| **superseded/** | Old docs replaced by newer versions |
| **misc/** | Documents with unclear purpose |
| **backups/** | File backups |

---

## Tech-Stack Quick Links

### Frontend
- [React](tech-stack/frontend/react.md) - Component model, hooks, rendering
- [Next.js](tech-stack/frontend/nextjs.md) - App Router, SSR, data fetching
- [State Management](tech-stack/frontend/state-management.md) - Zustand, React Query
- [Tailwind CSS](tech-stack/frontend/tailwindcss.md) - Utility-first styling
- [Form Handling](tech-stack/frontend/react-hook-form.md) - React Hook Form + Zod

### Backend
- [NestJS](tech-stack/backend/nestjs.md) - Modules, DI, decorators
- [Authentication](tech-stack/backend/authentication.md) - JWT, sessions, tokens
- [Authorization](tech-stack/backend/authorization.md) - RBAC, guards
- [REST API Design](tech-stack/backend/rest-api-design.md) - Conventions, endpoints

### Database
- [Prisma](tech-stack/database/prisma.md) - ORM concepts, queries
- [PostgreSQL](tech-stack/database/postgresql.md) - Database fundamentals
- [Redis](tech-stack/database/redis.md) - Caching, sessions

### Infrastructure
- [Docker](tech-stack/infrastructure/docker.md) - Containers, images
- [CI/CD](tech-stack/infrastructure/ci-cd.md) - Pipelines, deployment

### Security & Testing
- [OWASP Security](tech-stack/security/owasp-security.md) - Top 10 vulnerabilities
- [Unit Testing](tech-stack/testing/unit-testing.md) - Jest, mocking

---

## For Different Audiences

### New Developer
1. Start with [getting-started/QUICK_START.md](getting-started/QUICK_START.md)
2. Read [tech-stack/README.md](tech-stack/README.md) for the big picture
3. Follow Profile A Runbook in [CARECIRCLE_HANDBOOK-2.md](CARECIRCLE_HANDBOOK-2.md)
4. Dive into specific [tech-stack/](tech-stack/) docs as needed

### DevOps / Deployment
1. [PRODUCTION_INFRASTRUCTURE_COMPLETE.md](PRODUCTION_INFRASTRUCTURE_COMPLETE.md)
2. Profile B Runbook in [CARECIRCLE_HANDBOOK-2.md](CARECIRCLE_HANDBOOK-2.md)
3. [deployment/](deployment/) guides
4. [tech-stack/infrastructure/](tech-stack/infrastructure/)

### Debugging
1. Troubleshooting Playbook in [CARECIRCLE_HANDBOOK-2.md](CARECIRCLE_HANDBOOK-2.md)
2. Relevant [tech-stack/](tech-stack/) document for the failing component

### Understanding Features
1. [features/COMPLETE_FEATURES_IMPLEMENTATION.md](features/COMPLETE_FEATURES_IMPLEMENTATION.md)
2. Relevant controller files in `apps/api/src/`

### Learning (Generic)
- [extras/learning/engineering-mastery/](extras/learning/engineering-mastery/) - Deep engineering concepts
- [extras/learning/Senior-dev/](extras/learning/Senior-dev/) - Senior developer guides

---

## Key Links

| Resource | URL |
|----------|-----|
| Swagger API Docs | http://localhost:3001/api |
| RabbitMQ Management | http://localhost:15672 |
| Prisma Studio | `npx prisma studio` |

---

## Contributing to Documentation

### When to Update
- Added a new feature → Update `features/COMPLETE_FEATURES_IMPLEMENTATION.md`
- Changed architecture → Update relevant `tech-stack/` document
- Found a bug/fix → Add to Troubleshooting in `CARECIRCLE_HANDBOOK-2.md`
- Answer same question 3x → Add to FAQ

### Documentation Philosophy
- **Concepts over code** - Explain the "why", not just the "how"
- **Keep it current** - Stale docs are worse than no docs
- **Cross-reference** - Link to related documents
- **Be opinionated** - Give clear recommendations

### Where Files Go
- CareCircle-specific → Main docs folders
- Generic learning → `extras/learning/`
- Outdated/old → `extras/historical/`
- Replaced by newer → `extras/superseded/`

---

*This index is the starting point for all CareCircle documentation.*
