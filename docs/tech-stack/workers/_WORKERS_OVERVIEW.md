# Workers & Background Jobs Overview

> Understanding asynchronous processing in CareCircle.

---

## The Mental Model

Think of background workers like a **postal service**:

- **Main API** = The post office counter (handles immediate requests)
- **Job Queue** = The sorting room (jobs waiting to be processed)
- **Workers** = Mail carriers (process jobs asynchronously)
- **Redis** = The warehouse (stores jobs until processed)

### Why Not Do Everything in the API?

```
                    WITHOUT WORKERS
                    ─────────────────

User clicks "Send Reminder"
        │
        ▼
┌─────────────────────────────────────┐
│            API Request              │
├─────────────────────────────────────┤
│  1. Validate input (50ms)           │
│  2. Save to database (100ms)        │
│  3. Send push to 50 users (5000ms)  │  ← User waits 5 seconds!
│  4. Send 50 emails (10000ms)        │  ← Now 15 seconds!
│  5. Return response                 │
└─────────────────────────────────────┘
Total: ~15 seconds (terrible UX)


                    WITH WORKERS
                    ────────────────

User clicks "Send Reminder"
        │
        ▼
┌─────────────────────────────────────┐
│            API Request              │
├─────────────────────────────────────┤
│  1. Validate input (50ms)           │
│  2. Save to database (100ms)        │
│  3. Add job to queue (10ms)         │
│  4. Return response                 │
└─────────────────────────────────────┘
Total: ~160ms (instant for user!)

        │
        │ Meanwhile, in the background...
        ▼

┌─────────────────────────────────────┐
│           Worker Process            │
├─────────────────────────────────────┤
│  Picks up job from queue            │
│  Sends 50 push notifications        │
│  Sends 50 emails                    │
│  (User doesn't wait for this)       │
└─────────────────────────────────────┘
```

---

## Core Concepts & Terminology

### The Queue Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           JOB QUEUE ANATOMY                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   PRODUCER                    QUEUE                     CONSUMER             │
│   ────────                    ─────                     ────────             │
│                                                                              │
│   API adds jobs     →     Redis stores jobs     →     Worker processes      │
│                                                                              │
│   "Send reminder"         ┌─────────────┐                                   │
│         │                 │  Job 1 (new)│───────────► Worker picks up       │
│         ▼                 │  Job 2 (new)│             Processes              │
│   Queue.add({...})        │  Job 3 (waiting)│         Marks complete        │
│                           │  Job 4 (processing)│                            │
│                           │  Job 5 (completed)│                             │
│                           └─────────────┘                                   │
│                                                                              │
│   Job States:                                                                │
│   ┌─────────┐    ┌─────────┐    ┌────────────┐    ┌───────────┐            │
│   │ waiting │ → │ active  │ → │ completed  │    │  failed   │            │
│   └─────────┘    └─────────┘    └────────────┘    └───────────┘            │
│                                       │                 │                    │
│                                       │         ┌───────┴───────┐           │
│                                       │         │   retrying    │           │
│                                       │         └───────────────┘           │
│                                       ▼                                      │
│                              ┌─────────────┐                                │
│                              │ dead letter │  (after max retries)           │
│                              └─────────────┘                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Terminology

| Term | Definition | CareCircle Example |
|------|------------|-------------------|
| **Job** | Unit of work to be processed | "Send medication reminder" |
| **Queue** | Named channel for jobs | `medication-reminders` |
| **Worker** | Process that consumes jobs | BullMQ worker instance |
| **Producer** | Code that creates jobs | API service adding a job |
| **Consumer** | Code that processes jobs | Worker handler function |
| **Scheduler** | Creates jobs on a schedule | Daily appointment reminder |
| **Dead Letter Queue (DLQ)** | Where failed jobs go | For manual investigation |
| **TTL** | Time-to-live for jobs | Job expires if not processed |
| **Backoff** | Delay between retries | Exponential: 1s, 2s, 4s, 8s |

---

## CareCircle's Queue Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         QUEUE TOPOLOGY                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   NOTIFICATION QUEUE              │   REMINDER QUEUES                       │
│   ──────────────────              │   ───────────────                       │
│   • Push notifications            │   • medication-reminders                │
│   • In-app notifications          │   • appointment-reminders               │
│   • Email notifications           │   • shift-reminders                     │
│                                   │                                         │
│   High priority                   │   Scheduled jobs                        │
│   Fast processing                 │   Time-sensitive                        │
│                                                                              │
│   DOCUMENT QUEUE                  │   SYSTEM QUEUES                         │
│   ──────────────                  │   ─────────────                         │
│   • File uploads to Cloudinary    │   • Dead letter queue (DLQ)             │
│   • Document processing           │   • System maintenance                   │
│                                   │                                          │
│   Async, can be slow              │   Monitoring & recovery                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Queue Details

| Queue | Purpose | Priority | Retry Strategy |
|-------|---------|----------|----------------|
| `notifications` | Push/email delivery | High | 3 attempts, exponential backoff |
| `medication-reminders` | Med schedules | High | 2 attempts, 5-minute delay |
| `appointment-reminders` | Appointment alerts | Medium | 3 attempts, 15-minute delay |
| `shift-reminders` | Shift notifications | Medium | 3 attempts, 15-minute delay |
| `document-upload` | File processing | Low | 5 attempts, exponential backoff |
| `dead-letter` | Failed jobs | N/A | Manual processing |

---

## How Scheduled Jobs Work

### The Scheduler Pattern

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         SCHEDULED JOB FLOW                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. SCHEDULER WAKES UP                                                       │
│     │                                                                        │
│     │  Cron: "Every 5 minutes"                                              │
│     │                                                                        │
│     ▼                                                                        │
│  2. QUERY: "What reminders are due?"                                        │
│     │                                                                        │
│     │  SELECT * FROM medication_schedules                                   │
│     │  WHERE next_reminder_time < NOW() + 15 minutes                        │
│     │  AND status = 'active'                                                │
│     │                                                                        │
│     ▼                                                                        │
│  3. CREATE JOBS FOR EACH                                                     │
│     │                                                                        │
│     │  Found 23 medications due for reminder                                │
│     │  Adding 23 jobs to medication-reminders queue                         │
│     │                                                                        │
│     ▼                                                                        │
│  4. WORKERS PROCESS JOBS                                                     │
│     │                                                                        │
│     │  Worker 1: Processing med-reminder-001                                │
│     │  Worker 1: Sending push to user ABC                                   │
│     │  Worker 2: Processing med-reminder-002                                │
│     │  Worker 2: Sending push to user DEF                                   │
│     │  ...                                                                   │
│     │                                                                        │
│     ▼                                                                        │
│  5. MARK AS SENT, UPDATE NEXT TIME                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Scheduler Implementation Concept

```typescript
// The scheduler runs on a cron schedule
@Cron('*/5 * * * *')  // Every 5 minutes
async checkMedicationReminders() {
  // 1. Find medications that need reminders soon
  const dueMedications = await this.findDueReminders();
  
  // 2. Add a job for each (don't process inline!)
  for (const med of dueMedications) {
    await this.reminderQueue.add('medication-reminder', {
      medicationId: med.id,
      userId: med.userId,
      scheduledTime: med.scheduledTime,
    });
  }
  
  // Scheduler's job is done. Workers handle the rest.
}
```

---

## Job Processing Patterns

### The Worker Handler Pattern

```typescript
// Mental model: "What should happen when this job runs?"

Worker.process('medication-reminder', async (job) => {
  const { medicationId, userId } = job.data;
  
  // 1. Fetch current state (might have changed since job was created)
  const medication = await db.medication.findUnique({ where: { id: medicationId } });
  
  // 2. Guard clauses (reasons NOT to process)
  if (!medication) return { skipped: 'medication_deleted' };
  if (medication.status !== 'active') return { skipped: 'medication_inactive' };
  if (await alreadySent(medicationId)) return { skipped: 'already_sent' };
  
  // 3. Do the actual work
  await sendPushNotification(userId, {
    title: 'Medication Reminder',
    body: `Time to take ${medication.name}`,
  });
  
  // 4. Record that we sent it
  await db.notificationLog.create({
    data: { medicationId, sentAt: new Date() }
  });
  
  return { success: true };
});
```

### Idempotency: The Most Important Concept

**Idempotency** = Running something multiple times has the same effect as running it once.

```
WHY IT MATTERS:
───────────────

Jobs can be processed multiple times due to:
• Network failures during acknowledgment
• Worker crashes mid-processing
• Retry logic
• Queue bugs

If your job handler isn't idempotent:
  User gets 5 push notifications instead of 1 😱


HOW TO ACHIEVE IT:
──────────────────

1. CHECK before acting
   if (await alreadySent(jobId)) return; // Skip if done

2. USE unique identifiers
   await db.notification.upsert({
     where: { jobId: job.id },  // Job ID is unique
     create: { ... },
     update: {},  // Do nothing if exists
   });

3. DESIGN for re-runs
   "What if this code runs twice? Will it cause problems?"
```

---

## Error Handling & Retries

### The Retry Philosophy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           RETRY DECISION TREE                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                         Job Failed!                                          │
│                             │                                                │
│                             ▼                                                │
│              ┌─────────────────────────────┐                                │
│              │   Is it a transient error?  │                                │
│              └──────────────┬──────────────┘                                │
│                             │                                                │
│         ┌────────YES────────┴────────NO────────┐                            │
│         │                                      │                            │
│         ▼                                      ▼                            │
│  ┌─────────────────┐                  ┌─────────────────┐                   │
│  │ Retry with      │                  │ Permanent error │                   │
│  │ backoff         │                  │ → DLQ           │                   │
│  │                 │                  │                 │                   │
│  │ Network timeout │                  │ Invalid data    │                   │
│  │ 503 from API    │                  │ User deleted    │                   │
│  │ Database busy   │                  │ Business rule   │                   │
│  └─────────────────┘                  └─────────────────┘                   │
│                                                                              │
│  TRANSIENT ERRORS (retry):           PERMANENT ERRORS (don't retry):        │
│  • Network failures                  • Validation errors                    │
│  • Rate limits (429)                 • Resource not found (404)             │
│  • Server errors (5xx)               • Permission denied (403)              │
│  • Database connection issues        • Bad request (400)                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Backoff Strategies

```
LINEAR BACKOFF:
  Attempt 1: fail → wait 1 min
  Attempt 2: fail → wait 1 min
  Attempt 3: fail → wait 1 min
  (Same delay every time)

EXPONENTIAL BACKOFF:
  Attempt 1: fail → wait 1 min
  Attempt 2: fail → wait 2 min
  Attempt 3: fail → wait 4 min
  Attempt 4: fail → wait 8 min
  (Doubles each time - good for overloaded services)

EXPONENTIAL WITH JITTER:
  Attempt 1: fail → wait 1 min + random(0-30s)
  Attempt 2: fail → wait 2 min + random(0-30s)
  (Prevents thundering herd when many jobs retry simultaneously)
```

### CareCircle's Retry Configuration

```typescript
// BullMQ job options
{
  attempts: 3,
  backoff: {
    type: 'exponential',
    delay: 60000,  // 1 minute initial delay
  },
  removeOnComplete: {
    count: 1000,   // Keep last 1000 completed jobs for debugging
  },
  removeOnFail: false,  // Keep failed jobs for investigation
}
```

---

## Scaling Considerations

### Single Worker vs Multiple Workers

```
SINGLE WORKER:
┌───────────────────────┐
│  Worker Process       │
│  ─────────────────   │
│  Processing: 1 at a  │
│  time (sequential)   │
│                      │
│  Good for:           │
│  • Order-dependent   │
│  • Resource-limited  │
│  • Simple debugging  │
└───────────────────────┘


CONCURRENT WORKERS:
┌───────────────────────┐     ┌───────────────────────┐
│  Worker Process 1     │     │  Worker Process 2     │
│  ─────────────────   │     │  ─────────────────   │
│  concurrency: 5       │     │  concurrency: 5       │
│                      │     │                      │
│  Processing:         │     │  Processing:         │
│  • Job A            │     │  • Job F             │
│  • Job B            │     │  • Job G             │
│  • Job C            │     │  • Job H             │
│  • Job D            │     │  • Job I             │
│  • Job E            │     │  • Job J             │
└───────────────────────┘     └───────────────────────┘

Total: 10 jobs processing simultaneously!
```

### Concurrency Trade-offs

| Higher Concurrency | Lower Concurrency |
|-------------------|-------------------|
| ✅ Faster throughput | ✅ Less memory usage |
| ✅ Better resource utilization | ✅ Simpler debugging |
| ❌ More memory usage | ✅ Prevents overwhelming external APIs |
| ❌ Can overwhelm databases | ✅ More predictable behavior |
| ❌ Harder to debug | ❌ Slower overall |

### CareCircle's Approach

```typescript
// Different concurrency for different queues
const workerOptions = {
  'notifications': { concurrency: 10 },     // High throughput needed
  'medication-reminders': { concurrency: 5 }, // Medium
  'document-upload': { concurrency: 2 },    // Rate-limited external API
  'dead-letter': { concurrency: 1 },        // Manual, sequential
};
```

---

## Monitoring & Observability

### What to Monitor

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        WORKER HEALTH METRICS                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  QUEUE DEPTH                          PROCESSING RATE                       │
│  ───────────                          ───────────────                       │
│  How many jobs waiting?               Jobs per minute                       │
│                                                                              │
│  ⚠️  Alert if > 1000 waiting         ⚠️  Alert if drops significantly      │
│  📈  Growing = workers can't keep up  📈  Should be steady                  │
│                                                                              │
│  FAILURE RATE                         LATENCY (Time in Queue)               │
│  ────────────                         ───────────────────────               │
│  % of jobs failing                    How long jobs wait before processing  │
│                                                                              │
│  ⚠️  Alert if > 5%                   ⚠️  Alert if > 5 minutes              │
│  📈  Spikes indicate problems         📈  High = need more workers          │
│                                                                              │
│  DLQ SIZE                             WORKER MEMORY                         │
│  ────────                             ─────────────                         │
│  Jobs in dead letter queue            Memory per worker process             │
│                                                                              │
│  ⚠️  Alert if > 0 (needs attention)  ⚠️  Alert if > 512MB                  │
│  📈  Should always be empty ideally  📈  Growing = memory leak              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Logging Strategy

```typescript
// Good logging gives you debugging superpowers

// When job starts
logger.info('Job started', { 
  jobId: job.id,
  type: job.name,
  data: job.data,
  attempt: job.attemptsMade + 1,
});

// When job succeeds
logger.info('Job completed', {
  jobId: job.id,
  duration: Date.now() - startTime,
  result: 'success',
});

// When job fails (with context!)
logger.error('Job failed', {
  jobId: job.id,
  error: error.message,
  stack: error.stack,
  data: job.data,  // What data caused the failure?
  attempt: job.attemptsMade,
  willRetry: job.attemptsMade < job.opts.attempts,
});
```

---

## Common Mistakes & How to Avoid Them

### Mistake 1: Not Handling Graceful Shutdown

```typescript
❌ WRONG: Worker just stops

process.exit(0);  // Jobs in progress are lost!


✅ RIGHT: Graceful shutdown

async function shutdown() {
  logger.info('Shutting down worker...');
  
  // 1. Stop accepting new jobs
  await worker.close();
  
  // 2. Wait for current jobs to finish (with timeout)
  // BullMQ handles this automatically with worker.close()
  
  logger.info('Worker shutdown complete');
  process.exit(0);
}

process.on('SIGTERM', shutdown);
process.on('SIGINT', shutdown);
```

### Mistake 2: Putting Too Much in Job Data

```typescript
❌ WRONG: Entire object in job data

queue.add('process-medication', {
  medication: { /* entire medication object */ },
  user: { /* entire user object */ },
  family: { /* entire family object */ },
});
// Data might be stale when job processes!


✅ RIGHT: Just IDs, fetch fresh data in worker

queue.add('process-medication', {
  medicationId: '123',
  userId: '456',
});

// Worker fetches current data
Worker.process(async (job) => {
  const medication = await db.medication.findUnique({
    where: { id: job.data.medicationId },
    include: { user: true },
  });
  // Now we have fresh data
});
```

### Mistake 3: No Circuit Breaker for External APIs

```typescript
❌ WRONG: Keep hitting failing API

for (const userId of userIds) {
  await sendPushNotification(userId);  // If API is down, all fail
}


✅ RIGHT: Circuit breaker pattern

let failureCount = 0;
const FAILURE_THRESHOLD = 5;

async function sendWithCircuitBreaker(userId) {
  if (failureCount >= FAILURE_THRESHOLD) {
    throw new Error('Circuit breaker open - push service unavailable');
  }
  
  try {
    await sendPushNotification(userId);
    failureCount = 0;  // Reset on success
  } catch (error) {
    failureCount++;
    throw error;
  }
}
```

---

## Decision Flowchart: Should This Be a Background Job?

```
                    ┌─────────────────────────────────────┐
                    │   Should this be a background job?  │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │  Does the user need to wait for it? │
                    └─────────────────┬───────────────────┘
                                      │
              ┌───────────YES─────────┴─────────NO────────────┐
              │                                                │
              ▼                                                ▼
    ┌─────────────────┐                        ┌─────────────────────────────┐
    │ Do it inline    │                        │ Is it time-sensitive?       │
    │ (in the request)│                        │ (must happen within minutes)│
    └─────────────────┘                        └──────────────┬──────────────┘
                                                              │
                                           ┌───────YES────────┴────────NO────┐
                                           │                                  │
                                           ▼                                  ▼
                                  ┌─────────────────┐            ┌─────────────────┐
                                  │ Background Job  │            │ Could also be   │
                                  │ (high priority) │            │ cron job or     │
                                  │                 │            │ lower priority  │
                                  │ Examples:       │            │                 │
                                  │ • Notifications │            │ Examples:       │
                                  │ • Alerts        │            │ • Analytics     │
                                  └─────────────────┘            │ • Cleanup       │
                                                                 │ • Reports       │
                                                                 └─────────────────┘
```

---

## Quick Reference

### When to Use Each Queue Pattern

| Pattern | Use When | Example |
|---------|----------|---------|
| **Fire-and-forget** | Don't need result | Analytics event |
| **Job with callback** | Need to track completion | File upload progress |
| **Scheduled job** | Run at specific time | Daily summary email |
| **Repeatable job** | Run on interval | Every 5 min reminder check |
| **Priority queue** | Some jobs more urgent | Emergency alerts first |
| **Rate-limited** | External API limits | 100 emails per minute |

### BullMQ Cheatsheet

```typescript
// Add job (producer)
await queue.add('job-name', { data }, { 
  delay: 5000,           // Wait 5s before processing
  attempts: 3,           // Retry 3 times
  priority: 1,           // Lower = higher priority
  removeOnComplete: true, // Clean up after
});

// Add scheduled job
await queue.add('job-name', { data }, {
  repeat: { cron: '0 9 * * *' },  // Every day at 9am
});

// Process jobs (consumer)
const worker = new Worker('queue-name', async (job) => {
  // job.data = your payload
  // job.id = unique ID
  // job.attemptsMade = retry count
  return result;
});

// Events
worker.on('completed', (job, result) => { /* ... */ });
worker.on('failed', (job, error) => { /* ... */ });
```

---

*Next: [BullMQ Deep Dive](bullmq.md) | [Queue Concepts](queue-concepts.md)*


