# CareCircle Documentation

> Complete documentation for the CareCircle caregiving platform.

---

## Quick Start

- **New here?** Start with [getting-started/QUICK_START.md](getting-started/QUICK_START.md)
- **Need the full picture?** Read [CARECIRCLE_HANDBOOK-1.md](CARECIRCLE_HANDBOOK-1.md)
- **Looking for something specific?** See the navigation below

---

## Documentation Map

### Root Level Files

| File | Description |
|------|-------------|
| [DOCUMENTATION_INDEX.md](DOCUMENTATION_INDEX.md) | Comprehensive navigation guide |
| [CARECIRCLE_HANDBOOK-1.md](CARECIRCLE_HANDBOOK-1.md) | System overview, architecture, domain model |
| [CARECIRCLE_HANDBOOK-2.md](CARECIRCLE_HANDBOOK-2.md) | Auth, runbooks, troubleshooting, FAQ |
| [PRODUCTION_INFRASTRUCTURE_COMPLETE.md](PRODUCTION_INFRASTRUCTURE_COMPLETE.md) | Infrastructure, workers, env variables |

---

### 📁 deployment/

Guides for deploying CareCircle to various environments.

| File | Description |
|------|-------------|
| [DEPLOYMENT_CHECKLIST.md](deployment/DEPLOYMENT_CHECKLIST.md) | Pre-deployment checklist |
| [FREE_DEPLOYMENT_GUIDE.md](deployment/FREE_DEPLOYMENT_GUIDE.md) | Deploy using free tier services |
| [ORACLE_CLOUD_FREE_TIER_GUIDE.md](deployment/ORACLE_CLOUD_FREE_TIER_GUIDE.md) | Oracle Cloud deployment |
| [PRODUCTION_DEPLOYMENT_GUIDE.md](deployment/PRODUCTION_DEPLOYMENT_GUIDE.md) | Full production deployment |

---

### 📁 features/

Feature implementation documentation.

| File | Description |
|------|-------------|
| [COMPLETE_FEATURES_IMPLEMENTATION.md](features/COMPLETE_FEATURES_IMPLEMENTATION.md) | All features, API endpoints, status |

---

### 📁 getting-started/

New developer onboarding.

| File | Description |
|------|-------------|
| [QUICK_START.md](getting-started/QUICK_START.md) | Get running in 10 minutes |
| [FREE_SERVICES_SETUP.md](getting-started/FREE_SERVICES_SETUP.md) | Set up free third-party services |

---

### 📁 guides/

Specific how-to guides.

| File | Description |
|------|-------------|
| [AUTHENTICATION.md](guides/AUTHENTICATION.md) | Authentication implementation guide |
| [DOCKER_DEPLOYMENT.md](guides/DOCKER_DEPLOYMENT.md) | Docker deployment details |
| [QA_TEST_REPORT.md](guides/QA_TEST_REPORT.md) | QA testing results |

---

### 📁 operations/

Operational procedures.

| File | Description |
|------|-------------|
| [BACKUP_IMPLEMENTATION_SUMMARY.md](operations/BACKUP_IMPLEMENTATION_SUMMARY.md) | Backup system overview |
| [BACKUP_PROCEDURES.md](operations/BACKUP_PROCEDURES.md) | How to backup/restore |

---

### 📁 testing/

Testing documentation.

| File | Description |
|------|-------------|
| [TESTING.md](testing/TESTING.md) | Testing strategy and guidelines |

---

### 📁 tech-stack/

Conceptual documentation for all technologies used.

#### Overview
| File | Description |
|------|-------------|
| [README.md](tech-stack/README.md) | Tech stack overview & architecture |

#### Frontend (tech-stack/frontend/)
| File | Description |
|------|-------------|
| [_FRONTEND_OVERVIEW.md](tech-stack/frontend/_FRONTEND_OVERVIEW.md) | How frontend pieces connect |
| [react.md](tech-stack/frontend/react.md) | React concepts, hooks, rendering |
| [nextjs.md](tech-stack/frontend/nextjs.md) | Next.js App Router, SSR |
| [state-management.md](tech-stack/frontend/state-management.md) | Zustand, React Query |
| [tailwindcss.md](tech-stack/frontend/tailwindcss.md) | Utility-first CSS |
| [tanstack-query.md](tech-stack/frontend/tanstack-query.md) | Server state management |
| [zustand.md](tech-stack/frontend/zustand.md) | Client state management |
| [react-hook-form.md](tech-stack/frontend/react-hook-form.md) | Form handling |
| [zod.md](tech-stack/frontend/zod.md) | Schema validation |
| [radix-ui.md](tech-stack/frontend/radix-ui.md) | Accessible UI components |
| [framer-motion.md](tech-stack/frontend/framer-motion.md) | Animations |
| [socket-io-client.md](tech-stack/frontend/socket-io-client.md) | WebSocket client |
| [stream-chat-react.md](tech-stack/frontend/stream-chat-react.md) | Chat UI components |

#### Backend (tech-stack/backend/)
| File | Description |
|------|-------------|
| [_BACKEND_OVERVIEW.md](tech-stack/backend/_BACKEND_OVERVIEW.md) | How backend pieces connect |
| [nestjs.md](tech-stack/backend/nestjs.md) | NestJS framework concepts |
| [authentication.md](tech-stack/backend/authentication.md) | JWT, sessions, tokens |
| [authorization.md](tech-stack/backend/authorization.md) | RBAC, guards, permissions |
| [rest-api-design.md](tech-stack/backend/rest-api-design.md) | API conventions |
| [prisma.md](tech-stack/backend/prisma.md) | Prisma in NestJS context |
| [class-validator.md](tech-stack/backend/class-validator.md) | DTO validation |
| [swagger.md](tech-stack/backend/swagger.md) | API documentation |
| [socket-io.md](tech-stack/backend/socket-io.md) | WebSocket server |
| [rabbitmq.md](tech-stack/backend/rabbitmq.md) | Message broker |
| [nodemailer.md](tech-stack/backend/nodemailer.md) | Email sending |

#### Database (tech-stack/database/)
| File | Description |
|------|-------------|
| [_DATABASE_OVERVIEW.md](tech-stack/database/_DATABASE_OVERVIEW.md) | Database layer overview |
| [prisma.md](tech-stack/database/prisma.md) | Prisma ORM concepts |
| [postgresql.md](tech-stack/database/postgresql.md) | PostgreSQL fundamentals |
| [redis.md](tech-stack/database/redis.md) | Caching, sessions |
| [migrations.md](tech-stack/database/migrations.md) | Database migrations |

#### Workers (tech-stack/workers/)
| File | Description |
|------|-------------|
| [_WORKERS_OVERVIEW.md](tech-stack/workers/_WORKERS_OVERVIEW.md) | Worker system overview |
| [bullmq.md](tech-stack/workers/bullmq.md) | Job queue concepts |
| [background-jobs.md](tech-stack/workers/background-jobs.md) | Background processing |

#### Infrastructure (tech-stack/infrastructure/)
| File | Description |
|------|-------------|
| [_INFRASTRUCTURE_OVERVIEW.md](tech-stack/infrastructure/_INFRASTRUCTURE_OVERVIEW.md) | Infrastructure overview |
| [docker.md](tech-stack/infrastructure/docker.md) | Container concepts |
| [ci-cd.md](tech-stack/infrastructure/ci-cd.md) | CI/CD pipelines |

#### Architecture (tech-stack/architecture/)
| File | Description |
|------|-------------|
| [principles.md](tech-stack/architecture/principles.md) | Architectural principles |
| [monorepo.md](tech-stack/architecture/monorepo.md) | Monorepo structure |

#### Security (tech-stack/security/)
| File | Description |
|------|-------------|
| [principles.md](tech-stack/security/principles.md) | Security principles |
| [owasp-security.md](tech-stack/security/owasp-security.md) | OWASP Top 10 |

#### Testing (tech-stack/testing/)
| File | Description |
|------|-------------|
| [philosophy.md](tech-stack/testing/philosophy.md) | Testing philosophy |
| [unit-testing.md](tech-stack/testing/unit-testing.md) | Unit testing patterns |

#### Concepts (tech-stack/concepts/)
| File | Description |
|------|-------------|
| [error-handling.md](tech-stack/concepts/error-handling.md) | Error handling patterns |
| [caching.md](tech-stack/concepts/caching.md) | Caching strategies |
| [real-time.md](tech-stack/concepts/real-time.md) | Real-time communication |

---

### 📁 extras/

Non-essential documentation (historical, learning material, superseded docs).

| Subfolder | Description |
|-----------|-------------|
| [extras/historical/](extras/historical/) | Old audit reports, project status |
| [extras/learning/](extras/learning/) | Generic engineering guides (not CareCircle-specific) |
| [extras/superseded/](extras/superseded/) | Old docs replaced by newer versions |
| [extras/misc/](extras/misc/) | Miscellaneous documents |
| [extras/backups/](extras/backups/) | Backup files |

#### Learning Material (extras/learning/)
- **engineering-mastery/** - 14 deep-dive engineering guides
- **Senior-dev/** - 40+ senior developer guides covering security, databases, architecture, performance

---

## Find by Topic

| Topic | Location |
|-------|----------|
| API Endpoints | [features/COMPLETE_FEATURES_IMPLEMENTATION.md](features/COMPLETE_FEATURES_IMPLEMENTATION.md) |
| Authentication | [tech-stack/backend/authentication.md](tech-stack/backend/authentication.md) |
| Authorization/RBAC | [tech-stack/backend/authorization.md](tech-stack/backend/authorization.md) |
| Background Jobs | [tech-stack/workers/bullmq.md](tech-stack/workers/bullmq.md) |
| Database Schema | [CARECIRCLE_HANDBOOK-1.md](CARECIRCLE_HANDBOOK-1.md) |
| Deployment | [deployment/](deployment/) |
| Docker | [tech-stack/infrastructure/docker.md](tech-stack/infrastructure/docker.md) |
| Environment Variables | [PRODUCTION_INFRASTRUCTURE_COMPLETE.md](PRODUCTION_INFRASTRUCTURE_COMPLETE.md) |
| Error Handling | [tech-stack/concepts/error-handling.md](tech-stack/concepts/error-handling.md) |
| Forms | [tech-stack/frontend/react-hook-form.md](tech-stack/frontend/react-hook-form.md) |
| Local Setup | [CARECIRCLE_HANDBOOK-2.md](CARECIRCLE_HANDBOOK-2.md) → Profile A |
| Production Deploy | [CARECIRCLE_HANDBOOK-2.md](CARECIRCLE_HANDBOOK-2.md) → Profile B |
| Real-time/WebSocket | [tech-stack/concepts/real-time.md](tech-stack/concepts/real-time.md) |
| Security | [tech-stack/security/owasp-security.md](tech-stack/security/owasp-security.md) |
| State Management | [tech-stack/frontend/state-management.md](tech-stack/frontend/state-management.md) |
| Testing | [tech-stack/testing/unit-testing.md](tech-stack/testing/unit-testing.md) |
| Troubleshooting | [CARECIRCLE_HANDBOOK-2.md](CARECIRCLE_HANDBOOK-2.md) → Troubleshooting |
| Workers | [PRODUCTION_INFRASTRUCTURE_COMPLETE.md](PRODUCTION_INFRASTRUCTURE_COMPLETE.md) |

---

## Contributing

When adding new documentation:
1. CareCircle-specific docs → Add to appropriate main folder
2. Generic learning material → Add to `extras/learning/`
3. Update this README with the new file location

---

*Last Updated: January 30, 2026*

