# 🔭 Architecture Overview

> High-level view of the CareCircle system architecture.

---

## What is CareCircle?

CareCircle is a **family caregiving coordination platform** that helps families manage care for elderly or ill loved ones. It solves the problem of scattered information, delayed communication, and coordination challenges among family members in different locations.

---

## Core Architecture Principles

### 1. **Monorepo Structure**

All applications and packages live in one repository for atomic changes and shared code.

```
carecircle/
├── apps/
│   ├── api/       # NestJS backend
│   ├── web/       # Next.js frontend
│   └── workers/   # Background jobs
└── packages/
    └── shared/    # Shared types & utils
```

### 2. **Multi-Tenancy (Family-Based)**

Data is isolated by family. Users can belong to multiple families with different roles.

```
Family "Thompson"
├── Members: Sarah (Admin), Mike (Caregiver), Jennifer (Viewer)
└── Care Recipients: Margaret "Grandma Maggie"
    ├── Medications
    ├── Appointments
    ├── Documents
    └── Timeline
```

### 3. **Event-Driven Communication**

Services communicate through events, not direct calls. This enables:
- Real-time updates to all family members
- Decoupled, scalable services
- Reliable message delivery (outbox pattern)

### 4. **Offline-First PWA**

The frontend works offline with:
- Service worker caching
- Offline action queue
- Emergency info always available

---

## System Components

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              SYSTEM COMPONENTS                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         PRESENTATION LAYER                          │   │
│   │                                                                     │   │
│   │   Next.js 14 Frontend (PWA)                                         │   │
│   │   • App Router for file-based routing                               │   │
│   │   • React Query for data fetching                                   │   │
│   │   • Tailwind CSS + "Warm Hearth" design system                      │   │
│   │   • Service Worker for offline support                              │   │
│   │                                                                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    │ HTTP + WebSocket                       │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         APPLICATION LAYER                           │   │
│   │                                                                     │   │
│   │   NestJS API Server                                                 │   │
│   │   • Modular architecture (feature modules)                          │   │
│   │   • JWT authentication with HTTP-only cookies                       │   │
│   │   • Role-based access control (Admin/Caregiver/Viewer)              │   │
│   │   • Input validation with class-validator                           │   │
│   │   • Swagger API documentation                                       │   │
│   │                                                                     │   │
│   │   Socket.io Gateway                                                 │   │
│   │   • Real-time updates to family members                             │   │
│   │   • Room-based broadcasting per family                              │   │
│   │                                                                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                        │
│                                    │                                        │
│         ┌──────────────────────────┼──────────────────────────┐             │
│         │                          │                          │             │
│         ▼                          ▼                          ▼             │
│   ┌───────────┐            ┌───────────┐            ┌───────────┐          │
│   │PostgreSQL │            │   Redis   │            │ RabbitMQ  │          │
│   │           │            │           │            │           │          │
│   │• Entities │            │• Sessions │            │• Domain   │          │
│   │• Relations│            │• Cache    │            │  Events   │          │
│   │• Audit    │            │• Rate     │            │• Queues   │          │
│   │  Trail    │            │  Limits   │            │           │          │
│   └───────────┘            └───────────┘            └─────┬─────┘          │
│                                                          │                 │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                          EVENT CONSUMERS                            │   │
│   │                                    │                                │   │
│   │   ┌────────────┐   ┌────────────┐   │   ┌────────────┐               │   │
│   │   │ WebSocket  │   │Notification│   │   │   Audit    │               │   │
│   │   │ Consumer   │   │ Consumer   │◀──┘   │  Consumer  │               │   │
│   │   │            │   │            │       │            │               │   │
│   │   │ Broadcasts │   │ Sends push │       │ Logs all   │               │   │
│   │   │ to family  │   │ & email    │       │ events     │               │   │
│   │   └────────────┘   └────────────┘       └────────────┘               │   │
│   │                                                                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Data Model (Simplified)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                              CORE ENTITIES                                    │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   User ─────────────┬──────────── FamilyMember ──────────── Family           │
│   │                 │                                        │               │
│   │ has sessions    │ belongs to (with role)                 │ has           │
│   │                 │                                        │               │
│   ▼                 │                                        ▼               │
│   Session           │                                    CareRecipient       │
│                     │                                        │               │
│                     │                    ┌───────────────────┼───────────┐   │
│                     │                    │                   │           │   │
│                     │                    ▼                   ▼           ▼   │
│                     │              Medication          Appointment    Document│
│                     │                    │                                   │
│                     │                    ▼                                   │
│                     │              MedicationLog                             │
│                     │                                                        │
│                     └──── Timeline Entry ◄──────── Caregiver Shift           │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## Request Flow Example

**Scenario**: User logs a medication as "given"

```
1. USER ACTION
   User clicks "Given" button on medication card
   
2. FRONTEND (Next.js)
   • Optimistic update: UI shows "Given" immediately
   • API call: POST /api/v1/medications/123/log
   • Cookies: accessToken sent automatically

3. API SERVER (NestJS)
   • JwtAuthGuard: Validates access token
   • RolesGuard: Checks user is Admin or Caregiver
   • ValidationPipe: Validates request body
   • MedicationsService: Creates log entry in DB
   • EventPublisher: Publishes "medication.logged" event

4. EVENT SYSTEM (RabbitMQ)
   • Event stored in outbox table (reliability)
   • Event published to RabbitMQ exchange
   • Multiple consumers receive the event

5. CONSUMERS
   • WebSocketConsumer: Emits to family room
   • NotificationConsumer: Sends push notifications
   • AuditConsumer: Logs the event

6. OTHER FAMILY MEMBERS
   • Receive WebSocket event
   • React Query cache invalidated
   • UI updates automatically
   • Push notification appears on phone

7. RESPONSE
   • Original request returns 201 Created
   • Frontend confirms optimistic update was correct
```

---

## Security Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SECURITY LAYERS                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   TRANSPORT LAYER                                                           │
│   ═══════════════                                                           │
│   • HTTPS only in production                                                │
│   • Secure, HTTP-only, SameSite cookies                                     │
│   • CORS restricted to frontend origin                                      │
│                                                                             │
│   AUTHENTICATION LAYER                                                      │
│   ════════════════════                                                      │
│   • JWT tokens (access + refresh)                                           │
│   • Access token: 15 min, in cookie                                         │
│   • Refresh token: 7 days, in cookie + DB hash                              │
│   • Automatic token refresh on 401                                          │
│                                                                             │
│   AUTHORIZATION LAYER                                                       │
│   ═══════════════════                                                       │
│   • Role-based: Admin, Caregiver, Viewer                                    │
│   • Family-scoped: All queries filtered by familyId                         │
│   • Resource ownership: Users can only access their data                    │
│                                                                             │
│   INPUT VALIDATION                                                          │
│   ════════════════                                                          │
│   • DTOs with class-validator decorators                                    │
│   • Whitelist mode: Unknown properties stripped                             │
│   • Type transformation: Strings to numbers, etc.                           │
│                                                                             │
│   DATA LAYER                                                                │
│   ══════════                                                                │
│   • TypeORM parameterized queries (SQL injection prevention)                │
│   • Argon2 password hashing                                                 │
│   • Soft deletes for audit trail                                            │
│                                                                             │
│   RATE LIMITING                                                             │
│   ════════════                                                              │
│   • Global: 100 requests/minute                                             │
│   • Auth endpoints: 5 attempts/minute                                       │
│   • Redis-backed for distributed rate limiting                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Scalability Considerations

| Component | Current | Scalable To |
|-----------|---------|-------------|
| API Servers | 1 instance | Horizontal scaling (load balancer) |
| Database | Single PostgreSQL | Read replicas, connection pooling |
| Cache | Single Redis | Redis Cluster |
| Events | Single RabbitMQ | RabbitMQ Cluster |
| Frontend | Vercel/Docker | CDN + edge caching |

---

## Next Steps

- [API Architecture](./API_ARCHITECTURE.md) - Backend deep dive
- [Frontend Architecture](./FRONTEND_ARCHITECTURE.md) - Frontend deep dive
- [Event-Driven](./EVENT_DRIVEN.md) - Event system details

---

_Back to [Architecture Index](./README.md) | [Documentation Index](../README.md)_

