# BullMQ Concepts

> Understanding job queues and background processing.

---

## 1. What Is BullMQ?

### Plain English Explanation

BullMQ is a **job queue library** that lets you run tasks in the background, outside of your main application.

Think of it like a **restaurant kitchen ticket system**:
- Waiters (API) put orders (jobs) on the ticket rack (queue)
- Cooks (workers) grab tickets and prepare orders
- Orders can be prioritized (VIP first)
- If a cook makes a mistake, the ticket goes back for retry

### The Core Problem BullMQ Solves

Some tasks shouldn't block the user:
- Sending 50 emails
- Processing uploaded files
- Generating reports
- Scheduled reminders

BullMQ handles these in the background while your API responds instantly.

---

## 2. Core Concepts & Terminology

### The Queue Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         BULLMQ ARCHITECTURE                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   PRODUCER                    QUEUE                     CONSUMER             │
│   (adds jobs)                 (Redis)                   (processes)          │
│                                                                              │
│   API Service ──► Queue.add() ──► Redis Storage ──► Worker.process()        │
│                                                                              │
│                                                                              │
│   JOB LIFECYCLE:                                                             │
│                                                                              │
│   ┌──────────┐     ┌──────────┐     ┌───────────┐     ┌───────────┐        │
│   │ waiting  │ ──► │  active  │ ──► │ completed │     │  failed   │        │
│   └──────────┘     └──────────┘     └───────────┘     └─────┬─────┘        │
│                                                              │              │
│                                          ┌───────────────────┘              │
│                                          ▼                                  │
│                                    ┌───────────┐                            │
│                                    │  delayed  │ (waiting for retry)        │
│                                    └───────────┘                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Terminology

| Term | Definition |
|------|------------|
| **Job** | Unit of work with data payload |
| **Queue** | Named channel for jobs |
| **Worker** | Process that executes jobs |
| **Producer** | Code that creates jobs |
| **Consumer** | Code that processes jobs |
| **Scheduler** | Creates jobs on a schedule |
| **Dead Letter Queue** | Where failed jobs go |
| **Backoff** | Delay strategy for retries |

---

## 3. How BullMQ Works

### Job Flow

```
1. PRODUCE
   queue.add('email', { to: 'user@email.com', subject: 'Hello' })

2. STORE (Redis)
   Job stored in 'waiting' state
   
3. PICK UP
   Worker checks queue, finds job
   Job moves to 'active' state

4. PROCESS
   Worker executes your handler function
   
5. COMPLETE or FAIL
   Success → 'completed'
   Error → 'failed' (retry or dead letter)
```

### Why Redis?

- **In-memory**: Fast job storage and retrieval
- **Persistence**: Jobs survive restarts (optional)
- **Atomic operations**: Safe concurrent access
- **Pub/Sub**: Workers notified of new jobs instantly

---

## 4. Why BullMQ for CareCircle

| Requirement | Why BullMQ Fits |
|-------------|-----------------|
| Medication reminders | Scheduled, time-sensitive jobs |
| Push notifications | Don't block API response |
| Email sending | Async with retry on failure |
| File processing | Long-running tasks |
| Redis-based | Shares Redis with caching |

---

## 5. When to Use BullMQ ✅

### Use Background Jobs When:
- Task takes > 1 second
- User doesn't need to wait for result
- Task can fail and needs retry
- Task should run at a specific time
- Task involves external APIs (rate limits)

### Use Scheduled Jobs When:
- Running tasks at specific times (9 AM daily)
- Recurring tasks (every 5 minutes)
- Delayed execution (send in 1 hour)

### Use Priorities When:
- Some jobs are more urgent
- Emergency alerts before regular notifications

---

## 6. When to AVOID BullMQ ❌

### DON'T Use for Real-Time

```
❌ BAD: Job queue for instant response
"User needs medication list NOW"
→ Don't queue, query directly

✅ GOOD: Job queue for async work
"Send reminder in 15 minutes"
→ Perfect for queue
```

### DON'T Store Huge Payloads

```
❌ BAD: Entire file in job data
queue.add('process', { file: entireFileBuffer });
// Job data stored in Redis!

✅ GOOD: Reference to file
queue.add('process', { fileId: '123', filePath: '/uploads/123.pdf' });
// Worker fetches file when processing
```

---

## 7. Best Practices & Recommendations

### Job Design

```typescript
// Good job structure
queue.add('send-reminder', {
  medicationId: '123',      // Reference IDs
  userId: '456',            // Not full objects
  scheduledTime: new Date().toISOString(),
}, {
  attempts: 3,              // Retry 3 times
  backoff: {
    type: 'exponential',
    delay: 60000,           // Start at 1 minute
  },
  removeOnComplete: 100,    // Keep last 100 for debugging
  removeOnFail: false,      // Keep failed for investigation
});
```

### Worker Implementation

```typescript
const worker = new Worker('reminders', async (job) => {
  // 1. Fetch fresh data (not from job payload)
  const medication = await prisma.medication.findUnique({
    where: { id: job.data.medicationId }
  });
  
  // 2. Guard clauses (reasons to skip)
  if (!medication) return { skipped: 'not_found' };
  if (medication.status !== 'active') return { skipped: 'inactive' };
  
  // 3. Do the work
  await sendPushNotification(job.data.userId, {
    title: 'Medication Reminder',
    body: `Time to take ${medication.name}`,
  });
  
  // 4. Return result
  return { success: true, sentAt: new Date() };
}, {
  concurrency: 5,  // Process 5 jobs in parallel
});
```

### Idempotency

Jobs can run multiple times (retries, crashes). Design for this:

```typescript
// Check before acting
if (await wasAlreadySent(job.id)) {
  return { skipped: 'already_sent' };
}

// Or use unique constraints
await prisma.notificationLog.upsert({
  where: { jobId: job.id },
  create: { jobId: job.id, sentAt: new Date() },
  update: {},  // Do nothing if exists
});
```

---

## 8. Common Mistakes & How to Avoid Them

### Mistake 1: Not Handling Shutdown

```typescript
❌ BAD: Kill process immediately
process.exit(0);  // Jobs in progress lost!

✅ GOOD: Graceful shutdown
process.on('SIGTERM', async () => {
  await worker.close();  // Wait for current jobs
  process.exit(0);
});
```

### Mistake 2: Blocking the Event Loop

```typescript
❌ BAD: CPU-intensive sync code
worker.process(async (job) => {
  const result = heavySyncCalculation();  // Blocks everything!
});

✅ GOOD: Keep it async or spawn thread
worker.process(async (job) => {
  const result = await runInWorkerThread(heavyCalculation);
});
```

### Mistake 3: No Error Handling

```typescript
❌ BAD: Unhandled rejection
worker.process(async (job) => {
  await doSomething();  // If this throws, silent failure
});

✅ GOOD: Proper error handling
worker.on('failed', (job, err) => {
  logger.error('Job failed', { jobId: job.id, error: err.message });
});
```

---

## 9. Retry Strategies

### Exponential Backoff

```
Attempt 1: Wait 1 min
Attempt 2: Wait 2 min
Attempt 3: Wait 4 min
Attempt 4: Wait 8 min
...

Good for: Rate-limited APIs, temporary failures
```

### Fixed Delay

```
Attempt 1: Wait 5 min
Attempt 2: Wait 5 min
Attempt 3: Wait 5 min

Good for: Simple retry, predictable timing
```

### Configuration

```typescript
{
  attempts: 5,
  backoff: {
    type: 'exponential',  // or 'fixed'
    delay: 60000,         // 1 minute base
  }
}
```

---

## 10. Quick Reference

### Queue Operations

```typescript
// Add job
await queue.add('name', { data }, { options });

// Add scheduled job
await queue.add('name', { data }, {
  delay: 60000,  // Run in 1 minute
});

// Add repeating job
await queue.add('name', { data }, {
  repeat: { cron: '0 9 * * *' },  // Daily at 9 AM
});

// Get job counts
const counts = await queue.getJobCounts();
// { waiting: 5, active: 2, completed: 100, failed: 3 }
```

### Worker Events

```typescript
worker.on('completed', (job, result) => { });
worker.on('failed', (job, error) => { });
worker.on('progress', (job, progress) => { });
worker.on('error', (error) => { });
```

### Job Options

| Option | Purpose |
|--------|---------|
| `delay` | Wait before processing |
| `attempts` | Max retry attempts |
| `backoff` | Retry delay strategy |
| `priority` | Lower = higher priority |
| `removeOnComplete` | Clean up finished jobs |
| `removeOnFail` | Clean up failed jobs |

---

## 11. Learning Resources

- [BullMQ Documentation](https://docs.bullmq.io)
- [BullMQ GitHub](https://github.com/taskforcesh/bullmq)
- Redis basics (prerequisite)

---

*Next: [Queue Concepts](queue-concepts.md) | [Scheduling](scheduling.md)*
