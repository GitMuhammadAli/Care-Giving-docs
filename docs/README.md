# 📚 CareCircle Documentation

> Complete documentation for the CareCircle family caregiving coordination platform.

---

## 🗂️ Documentation Structure

```
docs/
│
├── 📋 CARECIRCLE_HANDBOOK-1.md  # Complete System Handbook (Part 1: Overview & Architecture)
├── 📋 CARECIRCLE_HANDBOOK-2.md  # Complete System Handbook (Part 2: Implementation & Technical)
│
├── 📖 getting-started/          # Setup & Installation
│   ├── QUICK_START.md           # 5-minute setup guide
│   ├── SETUP.md                 # Detailed setup instructions
│   └── FREE_SERVICES_SETUP.md   # Configure free tier services
│
├── 🏗️ architecture/             # System Architecture
│   ├── OVERVIEW.md              # High-level architecture
│   ├── API_ARCHITECTURE.md      # Backend (NestJS)
│   ├── FRONTEND_ARCHITECTURE.md # Frontend (Next.js)
│   └── EVENT_DRIVEN.md          # Events (RabbitMQ)
│
├── 📘 guides/                   # Feature Guides
│   ├── PROJECT_OVERVIEW.md      # Complete project guide
│   ├── AUTHENTICATION.md        # Auth system guide
│   ├── DOCKER_DEPLOYMENT.md     # Docker deployment guide
│   └── QA_TEST_REPORT.md        # QA testing report
│
├── 🚀 deployment/               # Deployment Guides
│   ├── PRODUCTION_DEPLOYMENT_GUIDE.md
│   ├── FREE_DEPLOYMENT_GUIDE.md
│   ├── ORACLE_CLOUD_FREE_TIER_GUIDE.md
│   ├── DEPLOYMENT_COMPARISON.md
│   └── COMPREHENSIVE_AUDIT_REPORT.md
│
├── ✨ features/                 # Feature Documentation
│   └── COMPLETE_FEATURES_IMPLEMENTATION.md
│
├── 🧪 testing/                  # Testing Documentation
│   └── TESTING.md               # Testing strategies & guides
│
├── 📊 project-status/           # Project Status
│   └── FINAL_STATUS.md          # Current implementation status
│
└── 🎓 engineering-mastery/      # Learning Resources
    ├── 01-fundamentals.md       # CS fundamentals
    ├── 02-system-design.md      # System design
    ├── ... (14 topics)          # Production engineering
    └── DEVOPS/                  # DevOps guides
```

---

## 🚀 Quick Navigation

### 📚 Start Here - Complete Handbooks

| Handbook | Description | Best For |
|----------|-------------|----------|
| [**Part 1: Overview & Architecture**](./CARECIRCLE_HANDBOOK-1.md) | System overview, architecture, monorepo structure, deployment | Engineers onboarding, stakeholders |
| [**Part 2: Implementation & Technical**](./CARECIRCLE_HANDBOOK-2.md) | Auth flows, bug fixes, Web Push, Stream Chat, FAQ, troubleshooting | Developers implementing features |

### New to CareCircle?

| Step | Guide | Description |
|------|-------|-------------|
| 1️⃣ | [Quick Start](./getting-started/QUICK_START.md) | Get running in 5 minutes |
| 2️⃣ | [Setup Guide](./getting-started/SETUP.md) | Detailed setup instructions |
| 3️⃣ | [Project Overview](./guides/PROJECT_OVERVIEW.md) | Understand the full system |
| 4️⃣ | [Free Services](./getting-started/FREE_SERVICES_SETUP.md) | Setup dev services |

### Building Features?

| Topic | Guide | Description |
|-------|-------|-------------|
| 🔐 Auth | [Authentication Guide](./guides/AUTHENTICATION.md) | JWT, sessions, email verification |
| 🔧 Backend | [API Architecture](./architecture/API_ARCHITECTURE.md) | NestJS modules, guards, DTOs |
| 🎨 Frontend | [Frontend Architecture](./architecture/FRONTEND_ARCHITECTURE.md) | Next.js, React Query, PWA |
| 📨 Events | [Event-Driven](./architecture/EVENT_DRIVEN.md) | RabbitMQ, consumers, outbox |
| ✨ Features | [Features Guide](./features/COMPLETE_FEATURES_IMPLEMENTATION.md) | Implementation status |

### Deploying to Production?

| Topic | Guide | Description |
|-------|-------|-------------|
| 🚀 Production | [Production Deployment](./deployment/PRODUCTION_DEPLOYMENT_GUIDE.md) | Full production deployment |
| 💰 Free Tier | [Free Deployment](./deployment/FREE_DEPLOYMENT_GUIDE.md) | Deploy on free services |
| ☁️ Oracle Cloud | [Oracle Free Tier](./deployment/ORACLE_CLOUD_FREE_TIER_GUIDE.md) | Always-free Oracle Cloud |
| 📊 Comparison | [Deployment Comparison](./deployment/DEPLOYMENT_COMPARISON.md) | Compare deployment options |

### Testing & Status

| Topic | Guide | Description |
|-------|-------|-------------|
| 🧪 Testing | [Testing Guide](./testing/TESTING.md) | Test strategies, unit tests, E2E tests |
| 📊 Status | [Final Status](./project-status/FINAL_STATUS.md) | Current implementation status (January 2026) |
| ✅ QA Report | [QA Test Report](./guides/QA_TEST_REPORT.md) | Complete QA test results |

### Learning Production Engineering?

| Level | Topic | Guide |
|-------|-------|-------|
| 🟢 Beginner | CS Fundamentals | [01-fundamentals.md](./engineering-mastery/01-fundamentals.md) |
| 🟡 Intermediate | System Design | [02-system-design.md](./engineering-mastery/02-system-design.md) |
| 🟡 Intermediate | Database Engineering | [03-database-engineering.md](./engineering-mastery/03-database-engineering.md) |
| 🔴 Advanced | Distributed Systems | [06-distributed-systems.md](./engineering-mastery/06-distributed-systems.md) |
| 🔴 Advanced | Security Engineering | [08-security-engineering.md](./engineering-mastery/08-security-engineering.md) |

👉 See [Engineering Mastery Index](./engineering-mastery/README.md) for the complete learning path.

---

## 🏃 Quick Start

```bash
# 1. Install dependencies
pnpm install

# 2. Setup environment
cp env.example .env

# 3. Start infrastructure (PostgreSQL, Redis, RabbitMQ, Mailpit)
docker-compose up -d

# 4. Run database migrations
pnpm db:migrate

# 5. Start development servers
pnpm dev
```

### What's Running

| Service | URL | Purpose |
|---------|-----|---------|
| 🌐 Web App | http://localhost:3000 | Next.js frontend |
| 🔌 API Server | http://localhost:3001 | NestJS backend |
| 📚 Swagger | http://localhost:3001/api/docs | API documentation |
| 📧 Mailpit | http://localhost:8025 | Email testing |
| 🐰 RabbitMQ | http://localhost:15672 | Message queue UI |

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CARECIRCLE ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                              ┌──────────────┐                               │
│                              │   Browser    │                               │
│                              │   (PWA)      │                               │
│                              └──────┬───────┘                               │
│                                     │                                       │
│                    ┌────────────────┼────────────────┐                      │
│                    │                │                │                      │
│                    ▼                ▼                ▼                      │
│            ┌───────────┐    ┌───────────┐    ┌───────────┐                  │
│            │  Next.js  │    │  NestJS   │    │ Socket.io │                  │
│            │  Frontend │◀──▶│    API    │◀──▶│  Gateway  │                  │
│            └───────────┘    └─────┬─────┘    └───────────┘                  │
│                                   │                                         │
│              ┌────────────────────┼────────────────────┐                    │
│              │                    │                    │                    │
│              ▼                    ▼                    ▼                    │
│       ┌───────────┐        ┌───────────┐        ┌───────────┐              │
│       │PostgreSQL │        │   Redis   │        │ RabbitMQ  │              │
│       │  Database │        │   Cache   │        │   Events  │              │
│       └───────────┘        └───────────┘        └─────┬─────┘              │
│                                                       │                     │
│                                                       ▼                     │
│                                                ┌───────────┐                │
│                                                │  BullMQ   │                │
│                                                │  Workers  │                │
│                                                └───────────┘                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📂 Codebase Structure

```
carecircle/
├── apps/
│   ├── api/                 # NestJS Backend
│   ├── web/                 # Next.js Frontend
│   └── workers/             # Background Jobs
│
├── packages/
│   └── shared/              # Shared Types & Utils
│
├── docs/                    # ← You are here
│
├── docker-compose.yml       # Local infrastructure
├── pnpm-workspace.yaml      # Monorepo config
└── env.example              # Environment template
```

---

## 🔑 Key Features

| Feature | Status | Guide |
|---------|--------|-------|
| 🔐 Authentication | ✅ Complete | [Auth Guide](./guides/AUTHENTICATION.md) |
| 👨‍👩‍👧‍👦 Family Management | ✅ Complete | [Project Overview](./guides/PROJECT_OVERVIEW.md) |
| 💊 Medications | ✅ Complete | [API Architecture](./architecture/API_ARCHITECTURE.md) |
| 📅 Appointments | ✅ Complete | [Frontend Architecture](./architecture/FRONTEND_ARCHITECTURE.md) |
| 🚨 Emergency Alerts | ✅ Complete | [Event-Driven](./architecture/EVENT_DRIVEN.md) |
| 📱 PWA & Offline | ✅ Complete | [Frontend Architecture](./architecture/FRONTEND_ARCHITECTURE.md) |
| 📨 Real-time Updates | ✅ Complete | [Event-Driven](./architecture/EVENT_DRIVEN.md) |

---

## 🛠️ Common Commands

```bash
# Development
pnpm dev                     # Start all apps
pnpm dev:api                # API only
pnpm dev:web                # Frontend only

# Database
pnpm db:migrate             # Run migrations
pnpm db:seed                # Seed data

# Testing
pnpm test                   # Run tests
pnpm test:e2e              # E2E tests

# Docker
docker-compose up -d        # Start services
docker-compose down         # Stop services
```

---

## 📞 Need Help?

- **Feature Questions**: Check [Project Overview](./guides/PROJECT_OVERVIEW.md)
- **Architecture Questions**: See [Architecture folder](./architecture/)
- **Implementation Details**: Read [Complete Handbooks](./CARECIRCLE_HANDBOOK-1.md)
- **Learning Path**: Follow [Engineering Mastery](./engineering-mastery/)

---

_CareCircle: Caregiving, coordinated. 🏡_

