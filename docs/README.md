# 📚 CareCircle Documentation

> Complete documentation for the CareCircle family caregiving coordination platform.

---

## 🗂️ Documentation Structure

```
docs/
│
├── 📖 getting-started/          # Setup & Installation
│   ├── QUICK_START.md           # 5-minute setup guide
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
│   └── AUTHENTICATION.md        # Auth system guide
│
└── 🎓 engineering-mastery/      # Learning Resources
    ├── 01-fundamentals.md       # CS fundamentals
    ├── 02-system-design.md      # System design
    └── ... (14 topics)          # Production engineering
```

---

## 🚀 Quick Navigation

### New to CareCircle?

| Step | Guide | Description |
|------|-------|-------------|
| 1️⃣ | [Quick Start](./getting-started/QUICK_START.md) | Get running in 5 minutes |
| 2️⃣ | [Project Overview](./guides/PROJECT_OVERVIEW.md) | Understand the full system |
| 3️⃣ | [Free Services](./getting-started/FREE_SERVICES_SETUP.md) | Setup dev services |

### Building Features?

| Topic | Guide | Description |
|-------|-------|-------------|
| 🔐 Auth | [Authentication Guide](./guides/AUTHENTICATION.md) | JWT, sessions, email verification |
| 🔧 Backend | [API Architecture](./architecture/API_ARCHITECTURE.md) | NestJS modules, guards, DTOs |
| 🎨 Frontend | [Frontend Architecture](./architecture/FRONTEND_ARCHITECTURE.md) | Next.js, React Query, PWA |
| 📨 Events | [Event-Driven](./architecture/EVENT_DRIVEN.md) | RabbitMQ, consumers, outbox |

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
# 1. Clone & Install
git clone https://github.com/yourorg/carecircle.git
cd carecircle
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

- **Documentation Issues**: Open an issue on GitHub
- **Feature Questions**: Check [Project Overview](./guides/PROJECT_OVERVIEW.md)
- **Architecture Questions**: See [Architecture folder](./architecture/)
- **Learning Path**: Follow [Engineering Mastery](./engineering-mastery/)

---

_CareCircle: Caregiving, coordinated. 🏡_

