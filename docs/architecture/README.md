# 🏗️ Architecture Documentation

> Technical architecture documentation for CareCircle.

---

## Guides in This Section

| Guide | Description | Focus |
|-------|-------------|-------|
| [Overview](./OVERVIEW.md) | High-level system architecture | System design |
| [API Architecture](./API_ARCHITECTURE.md) | NestJS backend patterns | Backend |
| [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) | Next.js frontend patterns | Frontend |
| [Event-Driven](./EVENT_DRIVEN.md) | RabbitMQ & event patterns | Events |

---

## System Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CARECIRCLE ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   CLIENTS                                                                   │
│   ═══════                                                                   │
│   ┌───────────┐  ┌───────────┐  ┌───────────┐                              │
│   │  Browser  │  │  Mobile   │  │  Service  │                              │
│   │   (PWA)   │  │   (PWA)   │  │  Worker   │                              │
│   └─────┬─────┘  └─────┬─────┘  └─────┬─────┘                              │
│         │              │              │                                     │
│         └──────────────┼──────────────┘                                     │
│                        │                                                    │
│   FRONTEND LAYER       ▼                                                    │
│   ═══════════════════════════════════════════════════════════════          │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │                     Next.js 14 (App Router)                 │          │
│   │  • Server Components  • React Query  • Tailwind CSS         │          │
│   │  • PWA Support        • Offline Sync • Push Notifications   │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                        │                                                    │
│                        │ REST API + WebSocket                               │
│                        ▼                                                    │
│   BACKEND LAYER        ════════════════════════════════════════            │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │                     NestJS API Server                       │          │
│   │  • JWT Auth         • RBAC Guards    • TypeORM              │          │
│   │  • Validation       • Rate Limiting  • Swagger              │          │
│   └─────────────────────────────────────────────────────────────┘          │
│            │                    │                    │                      │
│            ▼                    ▼                    ▼                      │
│   ┌─────────────┐      ┌─────────────┐      ┌─────────────┐                │
│   │ PostgreSQL  │      │    Redis    │      │  RabbitMQ   │                │
│   │  Database   │      │    Cache    │      │   Events    │                │
│   └─────────────┘      └─────────────┘      └──────┬──────┘                │
│                                                    │                        │
│   EVENT CONSUMERS      ════════════════════════════════════════            │
│   ┌────────────────────────────────────────────────┼────────────┐          │
│   │         ┌───────────┐  ┌───────────┐  ┌───────┴─────┐      │          │
│   │         │ WebSocket │  │  Notify   │  │   Audit     │      │          │
│   │         │ Consumer  │  │ Consumer  │  │  Consumer   │      │          │
│   │         └───────────┘  └───────────┘  └─────────────┘      │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Frontend** | Next.js 14 | React framework with App Router |
| **UI** | Tailwind CSS + Shadcn | Styling & components |
| **State** | TanStack Query | Server state management |
| **Backend** | NestJS | TypeScript API framework |
| **Database** | PostgreSQL | Primary data store |
| **ORM** | TypeORM | Database abstraction |
| **Cache** | Redis | Caching & sessions |
| **Events** | RabbitMQ | Message broker |
| **Jobs** | BullMQ | Background processing |
| **Real-time** | Socket.io | WebSocket connections |

---

## Key Patterns

### 1. Multi-Tenancy (Family-Based)

All data is scoped to families. Users can belong to multiple families.

```typescript
// Every query is family-scoped
const medications = await repo.find({
  where: {
    careRecipient: {
      familyId: user.currentFamilyId  // Always scoped!
    }
  }
});
```

### 2. Event-Driven Architecture

Domain events are published to RabbitMQ, consumed by multiple services.

```
Service → Outbox Table → RabbitMQ → Consumers
                                  ├─→ WebSocket (real-time)
                                  ├─→ Notifications (push)
                                  └─→ Audit (logging)
```

### 3. JWT with HTTP-Only Cookies

Secure, stateless authentication with refresh token rotation.

```
Access Token:  15 min lifetime, in HTTP-only cookie
Refresh Token: 7 days lifetime, in HTTP-only cookie + DB hash
```

### 4. Optimistic Updates

Frontend assumes success, rolls back on failure.

```typescript
// Update UI immediately
onMutate: async (data) => {
  await queryClient.cancelQueries(['medications']);
  const previous = queryClient.getQueryData(['medications']);
  queryClient.setQueryData(['medications'], (old) => /* update */);
  return { previous };
},
// Rollback on error
onError: (err, vars, context) => {
  queryClient.setQueryData(['medications'], context.previous);
}
```

---

## Data Flow

### Request Flow (REST API)

```
Client Request
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ 1. Middleware (logging, CORS, helmet)                       │
├─────────────────────────────────────────────────────────────┤
│ 2. Guards (JwtAuthGuard, RolesGuard)                        │
├─────────────────────────────────────────────────────────────┤
│ 3. Pipes (ValidationPipe)                                   │
├─────────────────────────────────────────────────────────────┤
│ 4. Controller → Service → Repository                        │
├─────────────────────────────────────────────────────────────┤
│ 5. Event Publishing (if write operation)                    │
├─────────────────────────────────────────────────────────────┤
│ 6. Response Interceptor (transform)                         │
└─────────────────────────────────────────────────────────────┘
    │
    ▼
Client Response
```

### Real-Time Flow (WebSocket)

```
Domain Event Published
    │
    ▼
RabbitMQ Queue
    │
    ▼
WebSocket Consumer
    │
    ▼
Socket.io Gateway
    │
    ▼
Family Room Broadcast
    │
    ▼
Client React Query Invalidation
    │
    ▼
UI Update
```

---

## Security Layers

| Layer | Implementation |
|-------|----------------|
| **Transport** | HTTPS, Secure cookies |
| **Authentication** | JWT, HTTP-only cookies |
| **Authorization** | RBAC (Admin/Caregiver/Viewer) |
| **Input Validation** | class-validator DTOs |
| **SQL Injection** | TypeORM parameterized queries |
| **Rate Limiting** | NestJS Throttler |
| **CORS** | Configured origins only |

---

## Folder Structure

```
apps/
├── api/                      # NestJS Backend
│   └── src/
│       ├── auth/             # Authentication
│       ├── user/             # User management
│       ├── families/         # Family & invites
│       ├── care-recipients/  # Care recipients
│       ├── medications/      # Medications
│       ├── appointments/     # Calendar
│       ├── documents/        # Document vault
│       ├── emergency/        # Emergency alerts
│       ├── timeline/         # Health timeline
│       ├── caregivers/       # Shift management
│       ├── notifications/    # Push notifications
│       ├── gateway/          # WebSocket
│       └── events/           # Event system
│
├── web/                      # Next.js Frontend
│   └── src/
│       ├── app/              # App Router pages
│       ├── components/       # React components
│       ├── hooks/            # Custom hooks
│       └── lib/              # Utilities & API
│
└── workers/                  # Background Jobs
    └── src/
        └── processors/       # Job processors
```

---

## Next Steps

- [API Architecture](./API_ARCHITECTURE.md) - Deep dive into backend
- [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) - Deep dive into frontend
- [Event-Driven](./EVENT_DRIVEN.md) - Event system details

---

_Back to [Documentation Index](../README.md)_

