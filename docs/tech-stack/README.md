# CareCircle Technology Stack

> A comprehensive guide to understanding the technologies that power CareCircle.

## Philosophy

This documentation is designed to help you **understand**, not just use. Code can be generated easily—what developers truly need is:

- **Conceptual understanding** of how things work
- **Decision-making frameworks** for when to use what
- **Mental models** for thinking about problems
- **Anti-patterns** to avoid
- **Trade-offs** to consider

---

## System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              USER DEVICES                                    │
│                    Browser (PWA) │ Mobile │ API Clients                     │
└─────────────────────────────────────┬───────────────────────────────────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    │                 │                 │
                    ▼                 ▼                 ▼
            ┌───────────┐     ┌───────────┐     ┌───────────┐
            │   HTTP    │     │ WebSocket │     │   Push    │
            │  REST API │     │  Socket.io│     │Notifications│
            └─────┬─────┘     └─────┬─────┘     └─────┬─────┘
                  │                 │                 │
                  └────────────┬────┴─────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────────────────┐
│                           APPLICATION LAYER                                  │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                        FRONTEND (Next.js 14)                         │   │
│  │  ┌─────────┐ ┌────────┐ ┌────────┐ ┌─────────┐ ┌──────────────┐    │   │
│  │  │ React   │ │Tailwind│ │ Zustand│ │ Query   │ │ Framer Motion│    │   │
│  │  │Components│ │  CSS   │ │ State  │ │(Server) │ │  Animations  │    │   │
│  │  └─────────┘ └────────┘ └────────┘ └─────────┘ └──────────────┘    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         BACKEND (NestJS 10)                          │   │
│  │  ┌─────────┐ ┌────────┐ ┌────────┐ ┌─────────┐ ┌──────────────┐    │   │
│  │  │ Modules │ │ Guards │ │  DTOs  │ │Services │ │  Repositories│    │   │
│  │  │Controllers│ │ Auth  │ │Validation│ │Business│ │  Data Access │    │   │
│  │  └─────────┘ └────────┘ └────────┘ └─────────┘ └──────────────┘    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                        WORKERS (BullMQ)                              │   │
│  │  ┌──────────────┐ ┌────────────┐ ┌───────────┐ ┌────────────────┐  │   │
│  │  │  Scheduler   │ │  Reminder  │ │Notification│ │  Dead Letter   │  │   │
│  │  │  (Cron Jobs) │ │  Workers   │ │  Worker   │ │    Queue       │  │   │
│  │  └──────────────┘ └────────────┘ └───────────┘ └────────────────┘  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────────┘
                               │
         ┌─────────────────────┼─────────────────────┐
         │                     │                     │
         ▼                     ▼                     ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   PostgreSQL    │  │      Redis      │  │    RabbitMQ     │
│                 │  │                 │  │                 │
│  Primary Data   │  │  Cache + Queue  │  │   Event Bus     │
│    Storage      │  │    Storage      │  │   Messaging     │
└─────────────────┘  └─────────────────┘  └─────────────────┘
         │
         ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                          EXTERNAL SERVICES                                    │
│  ┌───────────┐  ┌────────────┐  ┌──────────┐  ┌─────────┐  ┌─────────────┐ │
│  │Cloudinary │  │  Mailtrap  │  │  Stream  │  │ Twilio  │  │   Neon DB   │ │
│  │  Storage  │  │   Email    │  │   Chat   │  │   SMS   │  │   Hosting   │ │
│  └───────────┘  └────────────┘  └──────────┘  └─────────┘  └─────────────┘ │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## Documentation Index

### 📁 Frontend
*Understanding client-side architecture*

| Document | Focus |
|----------|-------|
| [Frontend Overview](frontend/_FRONTEND_OVERVIEW.md) | How all frontend pieces connect |
| [React Concepts](frontend/react.md) | Component model, rendering, lifecycle |
| [State Management](frontend/state-management.md) | When to use what state solution |
| [Next.js](frontend/nextjs.md) | Server components, routing, data fetching |
| [Styling Architecture](frontend/styling.md) | Tailwind philosophy, design systems |
| [Form Handling](frontend/forms.md) | Validation, submission, UX patterns |
| [Real-Time Updates](frontend/real-time.md) | WebSocket, optimistic UI, sync |

### 📁 Backend
*Understanding server-side patterns*

| Document | Focus |
|----------|-------|
| [Backend Overview](backend/_BACKEND_OVERVIEW.md) | How backend pieces connect |
| [NestJS Architecture](backend/nestjs.md) | Modules, DI, decorators philosophy |
| [API Design](backend/api-design.md) | REST principles, endpoint design |
| [Authentication](backend/authentication.md) | JWT strategy, sessions, security |
| [Authorization](backend/authorization.md) | RBAC, guards, permissions model |
| [Validation](backend/validation.md) | DTO patterns, error handling |

### 📁 Database
*Understanding data layer*

| Document | Focus |
|----------|-------|
| [Database Overview](database/_DATABASE_OVERVIEW.md) | Data architecture decisions |
| [PostgreSQL](database/postgresql.md) | Relational modeling, when to use |
| [Redis](database/redis.md) | Caching strategies, queue usage |
| [Prisma ORM](database/prisma.md) | Type-safe queries, migrations |
| [Data Modeling](database/data-modeling.md) | Schema design principles |

### 📁 Workers
*Understanding background processing*

| Document | Focus |
|----------|-------|
| [Workers Overview](workers/_WORKERS_OVERVIEW.md) | Background job architecture |
| [Queue Concepts](workers/queue-concepts.md) | Why queues, when to use them |
| [Job Scheduling](workers/scheduling.md) | Timing, retries, reliability |
| [BullMQ](workers/bullmq.md) | Redis-based job processing |

### 📁 Infrastructure
*Understanding deployment & operations*

| Document | Focus |
|----------|-------|
| [Infrastructure Overview](infrastructure/_INFRASTRUCTURE_OVERVIEW.md) | How it all deploys |
| [Docker](infrastructure/docker.md) | Containerization concepts |
| [CI/CD](infrastructure/ci-cd.md) | Automation philosophy |
| [Environment Management](infrastructure/environments.md) | Config strategies |

### 📁 Architecture
*Understanding system design*

| Document | Focus |
|----------|-------|
| [Architecture Principles](architecture/principles.md) | Core design decisions |
| [Separation of Concerns](architecture/separation-of-concerns.md) | Layer boundaries |
| [Event-Driven Design](architecture/event-driven.md) | Async communication patterns |
| [Monorepo Structure](architecture/monorepo.md) | Code organization philosophy |

### 📁 Security
*Understanding security practices*

| Document | Focus |
|----------|-------|
| [Security Principles](security/principles.md) | Defense in depth |
| [Authentication Security](security/auth-security.md) | Token handling, session safety |
| [Data Protection](security/data-protection.md) | Encryption, PII handling |

### 📁 Testing
*Understanding quality assurance*

| Document | Focus |
|----------|-------|
| [Testing Philosophy](testing/philosophy.md) | What to test, what not to test |
| [Testing Strategies](testing/strategies.md) | Unit vs integration vs e2e |

### 📁 Concepts
*Cross-cutting concerns*

| Document | Focus |
|----------|-------|
| [Error Handling](concepts/error-handling.md) | Philosophy of errors |
| [Caching Strategies](concepts/caching.md) | When and how to cache |
| [Real-Time Communication](concepts/real-time.md) | WebSocket vs polling |
| [File Uploads](concepts/file-uploads.md) | Storage strategies |

---

## How to Use This Documentation

### For New Developers
1. Start with the **Overview** files in each folder
2. Read **Architecture Principles** to understand design decisions
3. Dive into specific technologies as needed

### For Debugging
1. Check the **Troubleshooting** section of relevant technology docs
2. Understand the **data flow** between components
3. Review **common mistakes** sections

### For Making Decisions
1. Read the **When to Use ✅** and **When to AVOID ❌** sections
2. Consider the **trade-offs** documented
3. Follow the **decision trees** provided

---

## Technology Decisions Summary

### Why These Choices?

| Technology | Why We Chose It | What We Gave Up |
|------------|-----------------|-----------------|
| **Next.js** | Server components, file routing, Vercel integration | Simpler SPA tooling |
| **NestJS** | TypeScript-first, modular, enterprise patterns | Learning curve, verbosity |
| **PostgreSQL** | ACID, relational integrity, Neon serverless | Document flexibility |
| **Redis** | Speed, BullMQ compatibility, Upstash free tier | Persistence guarantees |
| **BullMQ** | Redis-native, TypeScript support, reliability | Simpler alternatives |
| **Prisma** | Type safety, migrations, great DX | Raw SQL flexibility |

---

## Quick Reference: What Technology for What?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PROBLEM → SOLUTION MAP                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  "I need to store user data"          → PostgreSQL + Prisma         │
│  "I need to cache API responses"      → Redis (cache-aside pattern) │
│  "I need real-time updates"           → Socket.io (WebSocket)       │
│  "I need to send notifications"       → BullMQ → web-push           │
│  "I need to schedule tasks"           → BullMQ repeatable jobs      │
│  "I need to handle file uploads"      → Cloudinary (async queue)    │
│  "I need to send emails"              → Nodemailer → Mailtrap/Resend│
│  "I need chat functionality"          → Stream Chat (managed)       │
│  "I need form validation"             → Zod (client) + class-validator (server) │
│  "I need to manage global state"      → Zustand (simple) or React Query (server)│
│  "I need to handle auth"              → JWT + HTTP-only cookies     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Learning Path Recommendations

### Beginner Path (1-2 weeks)
1. React fundamentals → State Management
2. Next.js basics → App Router
3. Tailwind CSS → Component styling
4. REST API basics → Data fetching

### Intermediate Path (2-4 weeks)
1. NestJS architecture → Module system
2. PostgreSQL + Prisma → Data modeling
3. Authentication flow → JWT, sessions
4. Testing strategies → Unit tests

### Advanced Path (4+ weeks)
1. Event-driven architecture → RabbitMQ
2. Background jobs → BullMQ patterns
3. Performance optimization → Caching
4. Security hardening → OWASP practices

---

*Last Updated: January 2026*
