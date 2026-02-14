# CareCircle Scalable Infrastructure - DRAG Framework

## Branch Strategy

```
main (stable)
  │
  └── feature/scalable-infrastructure  ← ALL work happens here
        └── merge to main after testing
```

---

## DRAG PHASE 1: RESEARCH FINDINGS

### 1. REDIS - What It Does Here

| Aspect | Finding |
|--------|---------|
| Library | `ioredis v5.3.2` (overridden to `v5.9.2` at root) |
| Config File | `apps/api/src/config/index.ts` → `redisConfig` |
| Provider | `apps/api/src/system/module/cache/redis.provider.ts` |
| Token | `REDIS_CLIENT` for dependency injection |

**Redis is used for:**

- **Caching** - `CacheService` with TTL (15 min default)
- **BullMQ Queues** - Job queue storage backend
- **Rate Limiting** - `ThrottlerGuard` storage
- **OTP Storage** - Temporary codes with expiry
- **Distributed Locking** - `LockHelper` for concurrency control

**What happens if Redis goes down:**

- App runs in "degraded mode" (caching disabled)
- BullMQ queues FAIL (requires Redis)
- Rate limiting may fall back to memory
- Graceful degradation logged as warning

**Environment Variables:**

```env
REDIS_HOST (default: localhost)
REDIS_PORT (default: 6379)
REDIS_PASSWORD (optional)
REDIS_TLS (default: false)
REDIS_URL (alternative: full connection string)
```

---

### 2. QUEUE SYSTEM - DUAL SYSTEM DETECTED

#### Finding: TWO Queue Systems Running

| System | Library | Storage | Purpose |
|--------|---------|---------|---------|
| **BullMQ** | `bull v4.12.0`, `bullmq v5.1.0`, `@nestjs/bull v10.0.1` | Redis | Background job processing |
| **RabbitMQ** | `@golevelup/nestjs-rabbitmq v5.3.0` | AMQP | Event-driven pub/sub |

#### BullMQ Queues (Redis-based)

| Queue Name | Purpose | Processor Location |
|------------|---------|-------------------|
| `medication-reminders` | Medication reminder jobs | `apps/workers/src/workers/medication-reminder.worker.ts` |
| `appointment-reminders` | Appointment reminder jobs | `apps/workers/src/workers/appointment-reminder.worker.ts` |
| `shift-reminders` | Shift reminder jobs | `apps/workers/src/workers/shift-reminder.worker.ts` |
| `notifications` | Push/email/SMS delivery | `apps/workers/src/workers/notification.worker.ts` |
| `refill-alerts` | Medication refill alerts | `apps/workers/src/workers/refill-alert.worker.ts` |
| `document-upload` | Document processing | `apps/api/src/documents/documents.processor.ts` |
| `dead-letter-queue` | Failed job handling | `apps/workers/src/workers/dead-letter.worker.ts` |

#### RabbitMQ Exchanges (AMQP-based)

| Exchange | Type | Purpose |
|----------|------|---------|
| `carecircle.domain.events` | Topic | Domain events (`medication.*`, `appointment.*`, etc.) |
| `carecircle.notifications` | Direct | Push/email/SMS routing |
| `carecircle.audit` | Fanout | Audit logging broadcast |
| `carecircle.dlx` | Direct | Dead letter exchange |

#### RabbitMQ Consumers

| Consumer | File | Handles |
|----------|------|---------|
| `NotificationConsumer` | `apps/api/src/events/consumers/notification.consumer.ts` | Push, email, emergency alerts |
| `WebSocketConsumer` | `apps/api/src/events/consumers/websocket.consumer.ts` | Real-time WebSocket broadcasts |
| `AuditConsumer` | `apps/api/src/events/consumers/audit.consumer.ts` | Audit logging |

---

### 3. WORKERS - `apps/workers/` Inventory

**Entry Point:** `apps/workers/src/index.ts`  
**Process Model:** SEPARATE process from API  
**Health Port:** `3002`

#### Worker Files

| File | Queue | Concurrency | Trigger | Purpose |
|------|-------|-------------|---------|---------|
| `medication-reminder.worker.ts` | `medication-reminders` | 10 | Scheduler (30,15,5,0 min before) | Send medication reminders |
| `appointment-reminder.worker.ts` | `appointment-reminders` | 10 | Scheduler (1day,1hr,30min before) | Send appointment reminders |
| `shift-reminder.worker.ts` | `shift-reminders` | 10 | Scheduler (60,15 min before) | Send shift reminders |
| `notification.worker.ts` | `notifications` | 20 | Other workers queue | Deliver push/email/SMS |
| `refill-alert.worker.ts` | `refill-alerts` | 5 | Scheduler (hourly) | Alert low medication supply |
| `dead-letter.worker.ts` | `dead-letter-queue` | 1 | Failed jobs | Log and alert failures |

#### Scheduler

- **File:** `apps/workers/src/scheduler.ts`
- **Class:** `ReminderScheduler`
- **Interval:** 60 seconds
- **Method:** Queries database for upcoming events, queues jobs with idempotent IDs

#### How Workers Start

```bash
# Development
npm run dev  # ts-node-dev --respawn src/index.ts

# Production
npm run build && npm start  # node dist/index.js
```

---

### 4. WEBSOCKETS - Real-time Features

| Aspect | Finding |
|--------|---------|
| Library | `socket.io v4.7.2`, `@nestjs/websockets v10.3.0` |
| Namespaces | `/carecircle` (main), `/` (generic) |
| Redis Adapter | **NOT CONFIGURED** (single-instance only) |

#### Gateway Files

| File | Namespace | Purpose |
|------|-----------|---------|
| `apps/api/src/gateway/carecircle.gateway.ts` | `/carecircle` | Family room broadcasts |
| `apps/api/src/gateway/events.gateway.ts` | `/` | Generic events |

#### Events Emitted (25+)

`medication_logged`, `appointment_created`, `appointment_updated`, `emergency_alert`, `emergency_resolved`, `shift_update`, `timeline_entry`, `family_member_removed`, `document_uploaded`, etc.

#### Frontend Hooks

| File | Purpose |
|------|---------|
| `apps/web/src/lib/websocket.ts` | WebSocket client singleton |
| `apps/web/src/hooks/use-websocket.ts` | React hook for events |
| `apps/web/src/components/providers/realtime-provider.tsx` | Context provider |

---

### 5. WEB PUSH NOTIFICATIONS

| Aspect | Finding |
|--------|---------|
| Library | `web-push v3.6.7` |
| Config | VAPID keys via environment variables |
| Storage | `PushToken` model in Prisma |

#### Notification Types

- `EMERGENCY_ALERT` (high priority, requires interaction)
- `MEDICATION_REMINDER`
- `APPOINTMENT_REMINDER`
- `SHIFT_REMINDER`
- `CHAT_MESSAGE`
- `GENERAL`

#### Files

| File | Purpose |
|------|---------|
| `apps/api/src/notifications/web-push.service.ts` | Send push notifications |
| `apps/web/src/lib/push-notifications.ts` | Subscribe/unsubscribe |
| `apps/web/public/sw.js` | Service worker handler |

---

### 6. EMAIL SYSTEM

| Aspect | Finding |
|--------|---------|
| Library | `nodemailer v6.9.7` |
| Primary Provider | **Brevo** (HTTP API configured) |
| Fallback Provider | Mailtrap SMTP (currently active) |
| **FREE TIER LIMIT** | **300 emails/day** (~9,000/month) |

#### Email Types Sent (11 templates)

1. `welcome`
2. `email-verification`
3. `password-reset`
4. `password-reset-by-admin`
5. `family-invitation`
6. `emergency-alert`
7. `medication-reminder`
8. `appointment-reminder`
9. `refill-alert`
10. `shift-reminder`
11. `data-export`

#### Email Sending

- Sent **INLINE** by `NotificationConsumer` (not separately queued)
- Rate limited via `LimitsService` (daily tracking)

---

### 7. DOCUMENT PROCESSING

| Aspect | Finding |
|--------|---------|
| Queue | `document-upload` (Bull) |
| Processor | `apps/api/src/documents/documents.processor.ts` |
| Storage | **Cloudinary** (`cloudinary v1.41.0`) |
| Also Available | AWS S3 (`@aws-sdk/client-s3 v3.966.0`) |

---

### 8. KEY DEPENDENCIES INVENTORY

| Library | Version | Purpose | Status |
|---------|---------|---------|--------|
| `@nestjs/schedule` | `^4.0.0` | Cron jobs | ✅ ALREADY INSTALLED |
| `@nestjs/bull` | `^10.0.1` | Queue integration | ✅ Installed |
| `bull` | `^4.12.0` | Job queue | ✅ Installed |
| `bullmq` | `^5.1.0` | Modern queue (workers) | ✅ Installed |
| `@golevelup/nestjs-rabbitmq` | `^5.3.0` | RabbitMQ | ✅ Installed |
| `ioredis` | `^5.3.2` | Redis client | ✅ Installed |
| `socket.io` | `^4.7.2` | WebSocket server | ✅ Installed |
| `web-push` | `^3.6.7` | Push notifications | ✅ Installed |
| `nodemailer` | `^6.9.7` | Email | ✅ Installed |
| `cloudinary` | `^1.41.0` | Image storage | ✅ Installed |

---

## DRAG PHASE 2: ANALYSIS

### Section A: BullMQ vs RabbitMQ Explained

#### What is BullMQ?

```
┌─────────────────────────────────────────┐
│              BullMQ                      │
├─────────────────────────────────────────┤
│ • Job queue LIBRARY (not a server)      │
│ • Built on top of Redis                 │
│ • Stores jobs in Redis lists            │
│ • Workers poll Redis for jobs           │
│ • Good for: delayed jobs, retries       │
│ • Requires: Redis only                  │
└─────────────────────────────────────────┘
```

#### What is RabbitMQ?

```
┌─────────────────────────────────────────┐
│              RabbitMQ                    │
├─────────────────────────────────────────┤
│ • Message broker SERVER                 │
│ • Runs as separate process/service      │
│ • Supports complex routing (exchanges)  │
│ • Pushes messages to consumers          │
│ • Good for: pub/sub, microservices      │
│ • Requires: RabbitMQ server running     │
└─────────────────────────────────────────┘
```

#### Are They the Same? **NO**

| Aspect | BullMQ | RabbitMQ |
|--------|--------|----------|
| Type | Library | Server/Service |
| Storage | Redis | Own storage |
| Message Delivery | Worker polls | Broker pushes |
| Complexity | Simple | Complex routing |
| Deployment | Just Redis | Separate server |
| Free Tier | Upstash (10K cmd/day) | CloudAMQP (1M msg/month) |

#### What THIS Project Uses (Evidence)

**BOTH** are used for **DIFFERENT purposes:**

**BullMQ (Redis)** - Background job processing
- Scheduled reminders (medication, appointment, shift)
- Document uploads
- Notification delivery
- Dead letter handling
- *Evidence:* `apps/workers/` folder, `@nestjs/bull` in `package.json`

**RabbitMQ (AMQP)** - Event-driven communication
- Domain event broadcasting
- WebSocket event bridging
- Audit logging
- *Evidence:* `@golevelup/nestjs-rabbitmq` in `package.json`, `apps/api/src/events/`

---

### Section B: Architecture Map

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              USER ACTION                                     │
│                    (Log medication, Create emergency, etc.)                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              API SERVER                                      │
│                         (NestJS Controllers)                                 │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SERVICE LAYER                                      │
│              (MedicationsService, EmergencyService, etc.)                    │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
                    ▼               ▼               ▼
┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────────────────┐
│   EventPublisher    │ │    BullMQ Queue     │ │        Database             │
│   (RabbitMQ)        │ │    (Redis)          │ │        (PostgreSQL)         │
└─────────────────────┘ └─────────────────────┘ └─────────────────────────────┘
         │                        │
         ▼                        ▼
┌─────────────────────┐ ┌─────────────────────────────────────────────────────┐
│  RabbitMQ Exchange  │ │              WORKERS PROCESS                        │
│  (domain.events)    │ │              (apps/workers/)                         │
└─────────────────────┘ │  ┌─────────────────┐  ┌─────────────────┐           │
         │              │  │ MedicationWorker│  │ NotificationWorker│         │
         │              │  └─────────────────┘  └─────────────────┘           │
         ▼              └─────────────────────────────────────────────────────┘
┌─────────────────────────────────────────┐              │
│           RabbitMQ Consumers            │              │
│  ┌─────────────────────────────────┐    │              ▼
│  │ NotificationConsumer            │    │    ┌─────────────────────┐
│  │ (creates in-app notifications)  │    │    │     WebPush         │
│  └─────────────────────────────────┘    │    │ (browser push)      │
│  ┌─────────────────────────────────┐    │    └─────────────────────┘
│  │ WebSocketConsumer               │    │              │
│  │ (emits to EventEmitter2)        │    │              ▼
│  └─────────────────────────────────┘    │    ┌─────────────────────┐
│  ┌─────────────────────────────────┐    │    │     Email           │
│  │ AuditConsumer                   │    │    │ (nodemailer/Brevo)  │
│  │ (logs events)                   │    │    └─────────────────────┘
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│        WebSocket Gateway                │
│        (Socket.IO)                      │
│  Broadcasts to rooms:                   │
│  - family:{familyId}                    │
│  - care-recipient:{careRecipientId}     │
└─────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│        BROWSER / MOBILE                 │
│  - Real-time updates (WebSocket)        │
│  - Push notifications (ServiceWorker)   │
│  - Offline sync (IndexedDB)             │
└─────────────────────────────────────────┘
```

---

### Section C: What Breaks Without Each Piece?

| Component | If Removed/Down | Impact |
|-----------|-----------------|--------|
| **Redis** | BullMQ fails, caching fails, rate limiting degrades | 🔴 **HIGH** - Background jobs stop |
| **RabbitMQ** | Event broadcasting fails, WebSocket updates stop | 🟡 **MEDIUM** - Real-time broken |
| **Workers Process** | Reminders not sent, notifications not delivered | 🔴 **HIGH** - Core features broken |
| **WebSockets** | No real-time updates | 🟡 **MEDIUM** - Users must refresh |
| **WebPush** | No browser push notifications | 🟢 **LOW** - In-app notifications still work |
| **PostgreSQL** | Everything fails | 🔴 **CRITICAL** - Complete outage |

---

### Section D: Risk Analysis for 10,000 Users

| Risk Area | Current State | Risk Level | Mitigation |
|-----------|---------------|------------|------------|
| Database | Single Neon instance | 🟡 MEDIUM | Connection pooling, indexes |
| Redis | Upstash 10K cmd/day | 🔴 HIGH | Optimize polling, caching strategy |
| RabbitMQ | CloudAMQP 1M msg/month | 🟡 MEDIUM | Batch events, reduce noise |
| WebSockets | No Redis adapter | 🔴 HIGH | Add adapter for horizontal scaling |
| Email | Brevo 300/day | 🟡 MEDIUM | Priority queue, track usage |
| Workers | Separate deployment | 🔴 HIGH | Move to cron jobs (simpler) |

#### What Breaks First Under Load:

1. Redis command limit (Upstash free tier)
2. WebSocket single-instance bottleneck
3. Worker process overwhelmed
4. Email daily limit exceeded

---

## DRAG PHASE 3: DRAFTING

### Plan A: Replace Workers with Cron Jobs

#### Why?

- Workers require **SEPARATE deployment** (another process)
- Cron jobs run **INSIDE the API** (one deployment)
- `@nestjs/schedule` is **ALREADY INSTALLED** (v4.0.0)
- Same logic, simpler deployment

#### Migration Map

| Worker | → Cron Job | Schedule | Logic |
|--------|------------|----------|-------|
| `medication-reminder.worker.ts` | `medication-reminder.cron.ts` | Every 1 minute | Query due medications, send notifications |
| `appointment-reminder.worker.ts` | `appointment-reminder.cron.ts` | Every 1 minute | Query upcoming appointments, send reminders |
| `shift-reminder.worker.ts` | `shift-reminder.cron.ts` | Every 1 minute | Query upcoming shifts, send reminders |
| `refill-alert.worker.ts` | `refill-alert.cron.ts` | Every hour (at :00) | Query low supply medications, send alerts |
| `notification.worker.ts` | Keep as BullMQ OR inline | On-demand | Notification delivery |
| `dead-letter.worker.ts` | `failed-jobs.cron.ts` | Every 5 minutes | Log and alert failed jobs |

#### Implementation Approach

```
apps/api/src/cron/
├── cron.module.ts           (Module registration)
├── medication-reminder.cron.ts
├── appointment-reminder.cron.ts
├── shift-reminder.cron.ts
├── refill-alert.cron.ts
├── failed-jobs.cron.ts
└── cron.lock.service.ts     (Distributed locking)
```

#### Cron Job Template

```typescript
@Injectable()
export class MedicationReminderCron {
  private readonly logger = new Logger(MedicationReminderCron.name);

  constructor(
    private readonly prisma: PrismaService,
    private readonly notifications: NotificationsService,
    private readonly lock: CronLockService,
  ) {}

  @Cron('* * * * *') // Every minute
  async handleMedicationReminders() {
    // Acquire distributed lock (prevent duplicate runs)
    const lockAcquired = await this.lock.acquire('medication-reminders', 55);
    if (!lockAcquired) return;

    try {
      const now = new Date();
      const reminderWindows = [30, 15, 5, 0]; // minutes before

      for (const minutesBefore of reminderWindows) {
        const targetTime = addMinutes(now, minutesBefore);

        // Query medications due at target time
        const medications = await this.prisma.medication.findMany({
          where: {
            isActive: true,
            scheduledTimes: { has: format(targetTime, 'HH:mm') },
            // Not already logged today
          },
          include: { careRecipient: { include: { family: { include: { members: true } } } } },
        });

        for (const med of medications) {
          // Check idempotency (already sent this reminder?)
          const jobId = `med-${med.id}-${format(now, 'yyyy-MM-dd-HH-mm')}-${minutesBefore}`;
          if (await this.lock.exists(jobId)) continue;

          // Send notification
          await this.notifications.sendMedicationReminder(med, minutesBefore);

          // Mark as sent
          await this.lock.set(jobId, '1', 3600); // 1 hour TTL
        }
      }
    } finally {
      await this.lock.release('medication-reminders');
    }
  }
}
```

---

### Plan B: Keep Queue Optional

#### Configuration Flag

```env
QUEUE_ENABLED=true   # Use BullMQ queues
QUEUE_ENABLED=false  # Use cron jobs instead
```

#### Job Dispatch Service

```typescript
@Injectable()
export class JobDispatchService {
  constructor(
    @InjectQueue('notifications') private notificationQueue: Queue,
    private readonly notificationService: NotificationsService,
    private readonly configService: ConfigService,
  ) {}

  async sendNotification(data: NotificationData) {
    if (this.configService.get('QUEUE_ENABLED') === 'true') {
      // Queue for async processing
      await this.notificationQueue.add('send', data);
    } else {
      // Execute inline (cron job mode)
      await this.notificationService.send(data);
    }
  }
}
```

---

### Plan C: Simplify Event System

#### Current (Complex)

```
Service → EventPublisher → RabbitMQ → Consumer → Action
```

#### Simplified (Keep RabbitMQ Optional)

```
Service → EventBus (NestJS EventEmitter2) → Handlers
           ↓ (if RABBITMQ_ENABLED)
        RabbitMQ → External consumers
```

#### Why Keep EventEmitter2?

- Already used for WebSocket bridging
- Works without external dependencies
- RabbitMQ becomes optional for scaling

---

### Plan D: Scalability Improvements

| Area | Improvement |
|------|-------------|
| **Database Optimization** | Add missing indexes (query analysis needed), Connection pooling configuration, Query batching for bulk operations |
| **Redis Optimization** | Cache frequently accessed data (family, user), TTL strategy (15 min default, 1 hour for static), Reduce BullMQ polling (already optimized) |
| **WebSocket Scaling** | Add Redis adapter for horizontal scaling, Connection limit monitoring, Heartbeat configuration |
| **Email Rate Limiting** | Track daily usage (already implemented), Priority queue (emergencies first), Warn at 80% of limit (240/day) |

---

## DRAG PHASE 4: IMPLEMENTATION STEPS

### Step 1: Create Branch

```bash
git checkout main
git pull origin main
git checkout -b feature/scalable-infrastructure
```

### Step 2: Create Cron Module Structure

```
apps/api/src/cron/
├── cron.module.ts
├── cron.lock.service.ts
├── medication-reminder.cron.ts
├── appointment-reminder.cron.ts
├── shift-reminder.cron.ts
├── refill-alert.cron.ts
└── failed-jobs.cron.ts
```

### Step 3: Implement Cron Lock Service

- Use Redis for distributed locking
- Prevent duplicate cron executions
- Idempotency tracking

### Step 4: Migrate Each Worker to Cron

- Copy logic from worker files
- Add `@Cron` decorators
- Add distributed locking
- Add logging
- Test each individually

### Step 5: Add Configuration Flags

```env
ENABLE_CRON_JOBS=true
QUEUE_ENABLED=false
RABBITMQ_ENABLED=false  # Optional for later
```

### Step 6: Update app.module.ts

- Import `CronModule`
- Conditional queue registration based on `QUEUE_ENABLED`

### Step 7: Add Health Checks

- Cron job execution monitoring
- Last run timestamps
- Failed job tracking

### Step 8: Test Everything

- All existing features work
- Cron jobs execute correctly
- Notifications delivered
- Real-time updates work
- No duplicate notifications

### Step 9: Documentation

- Environment variables guide
- Deployment guide (simple vs full infra)
- Architecture diagram update

---

## VERIFICATION PLAN

### Build Check

```bash
cd apps/api
pnpm build  # Should compile without errors
```

### Local Test

```bash
pnpm dev

# Test scenarios:
# 1. Log medication → family member gets notification
# 2. Create appointment → reminder sent at correct time
# 3. Emergency alert → all family members notified
# 4. Document upload → processed successfully
```

### Cron Job Verification

```bash
# Check cron execution logs
tail -f logs/cron.log

# Verify no duplicates
# Verify correct timing
# Verify notifications sent
```

### Load Test (Optional)

```bash
# Test with k6 or artillery
# Verify system handles load
# Check for bottlenecks
```

---

## FILES TO CREATE

| File | Purpose |
|------|---------|
| `apps/api/src/cron/cron.module.ts` | Module registration |
| `apps/api/src/cron/cron.lock.service.ts` | Distributed locking |
| `apps/api/src/cron/medication-reminder.cron.ts` | Medication reminders |
| `apps/api/src/cron/appointment-reminder.cron.ts` | Appointment reminders |
| `apps/api/src/cron/shift-reminder.cron.ts` | Shift reminders |
| `apps/api/src/cron/refill-alert.cron.ts` | Refill alerts |
| `apps/api/src/cron/failed-jobs.cron.ts` | Failed job monitoring |

---

## FILES TO MODIFY

| File | Changes |
|------|---------|
| `apps/api/src/app.module.ts` | Import `CronModule`, conditional queue loading |
| `apps/api/src/config/index.ts` | Add `ENABLE_CRON_JOBS`, `QUEUE_ENABLED` flags |
| `.env.example` | Document new environment variables |

---

## FILES TO KEEP (Not Delete)

| File | Reason |
|------|--------|
| `apps/workers/` folder | Keep as fallback, don't delete |
| RabbitMQ consumers | Keep for optional event broadcasting |
| BullMQ configuration | Keep for optional queue mode |

---

## SUMMARY

### Before

- ❌ 2 queue systems (BullMQ + RabbitMQ)
- ❌ Separate workers process required
- ❌ Complex deployment
- ❌ Multiple external dependencies

### After

- ✅ Cron jobs for scheduled tasks (in-process)
- ✅ Queue system optional via config
- ✅ Single deployment (API includes crons)
- ✅ Simpler infrastructure

### Scalability Maintained

- ✅ Redis for caching + locking
- ✅ WebSockets for real-time
- ✅ Push notifications for offline
- ✅ Queue mode available when scaling

### Non-Breaking

- ✅ All existing features work
- ✅ Workers folder kept as fallback
- ✅ Configuration-based switching
- ✅ Easy rollback (just switch branch)

---

**✅ User approved the plan**