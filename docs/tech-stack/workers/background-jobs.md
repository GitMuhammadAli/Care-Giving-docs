# Background Jobs

> Worker implementation for CareCircle background tasks.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | Background job processors |
| **Why** | Async tasks, scheduled reminders, notifications |
| **Location** | `apps/workers/src/workers/` |

## Worker Architecture

```
apps/workers/
├── src/
│   ├── index.ts              # Entry point, health server
│   ├── config.ts             # Configuration
│   ├── queues.ts             # Queue definitions
│   ├── scheduler.ts          # Job scheduler (cron)
│   └── workers/
│       ├── medication-reminder.worker.ts
│       ├── appointment-reminder.worker.ts
│       ├── shift-reminder.worker.ts
│       ├── notification.worker.ts
│       ├── refill-alert.worker.ts
│       └── dead-letter.worker.ts
└── package.json
```

## Entry Point

```typescript
// index.ts
import { createServer } from 'http';
import { getConfig, logger, getRedisConnection } from './config';
import { ReminderScheduler } from './scheduler';
import { medicationReminderWorker } from './workers/medication-reminder.worker';
import { appointmentReminderWorker } from './workers/appointment-reminder.worker';
import { shiftReminderWorker } from './workers/shift-reminder.worker';
import { notificationWorker } from './workers/notification.worker';
import { refillAlertWorker } from './workers/refill-alert.worker';
import { deadLetterWorker } from './workers/dead-letter.worker';
import { closeAllQueues } from './queues';
import { prisma } from '@carecircle/database';

// All workers
const workers = [
  medicationReminderWorker,
  appointmentReminderWorker,
  shiftReminderWorker,
  notificationWorker,
  refillAlertWorker,
  deadLetterWorker,
];

// Health check server
const healthServer = createServer(async (req, res) => {
  if (req.url === '/health' || req.url === '/healthz') {
    const health = await getHealthStatus();
    res.writeHead(health.status === 'healthy' ? 200 : 503);
    res.end(JSON.stringify(health));
  } else if (req.url === '/ready') {
    const ready = await checkReadiness();
    res.writeHead(ready ? 200 : 503);
    res.end(JSON.stringify({ ready }));
  } else {
    res.writeHead(404);
    res.end('Not Found');
  }
});

// Startup
async function main() {
  const config = getConfig();
  
  // Self-tests
  await runStartupTests();
  
  // Start scheduler
  const scheduler = new ReminderScheduler();
  scheduler.start();
  
  // Start health server
  const port = config.HEALTH_PORT || 3002;
  healthServer.listen(port, () => {
    logger.info({ port }, 'Health server started');
  });
  
  logger.info('All workers started successfully');
}

// Graceful shutdown
async function shutdown(signal: string) {
  logger.info({ signal }, 'Shutdown signal received');
  
  // Stop scheduler
  scheduler.stop();
  
  // Close workers
  await Promise.all(workers.map(w => w.close()));
  
  // Close queues
  await closeAllQueues();
  
  // Close database
  await prisma.$disconnect();
  
  // Close health server
  healthServer.close();
  
  logger.info('Graceful shutdown complete');
  process.exit(0);
}

process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));

main().catch((err) => {
  logger.fatal({ err }, 'Failed to start workers');
  process.exit(1);
});
```

## Worker Implementations

### Medication Reminder Worker

```typescript
// workers/medication-reminder.worker.ts
import { Worker, Job } from 'bullmq';
import { prisma } from '@carecircle/database';
import { notificationQueue } from '../queues';
import { logger, getRedisConnection } from '../config';

interface MedicationReminderJob {
  medicationId: string;
  scheduledTime: string;
  reminderMinutes: number;
}

export const medicationReminderWorker = new Worker<MedicationReminderJob>(
  'medication-reminders',
  async (job: Job<MedicationReminderJob>) => {
    const { medicationId, scheduledTime, reminderMinutes } = job.data;

    // Get medication with care recipient and family
    const medication = await prisma.medication.findUnique({
      where: { id: medicationId },
      include: {
        careRecipient: {
          include: {
            family: {
              include: {
                members: {
                  include: { user: true },
                },
              },
            },
          },
        },
      },
    });

    if (!medication || !medication.isActive) {
      logger.info({ medicationId }, 'Medication not active, skipping');
      return;
    }

    // Check if already logged
    const existingLog = await prisma.medicationLog.findFirst({
      where: {
        medicationId,
        scheduledTime: new Date(scheduledTime),
        status: { in: ['GIVEN', 'SKIPPED'] },
      },
    });

    if (existingLog) {
      logger.info({ medicationId }, 'Already logged, skipping reminder');
      return;
    }

    // Send notification to all family members
    const userIds = medication.careRecipient.family.members.map(m => m.userId);
    
    await notificationQueue.add('medication-reminder', {
      userIds,
      title: `💊 Medication Reminder`,
      body: `Time to give ${medication.name} (${medication.dosage}) to ${medication.careRecipient.name}`,
      url: `/medications?id=${medicationId}`,
      data: {
        type: 'MEDICATION_REMINDER',
        medicationId,
        careRecipientId: medication.careRecipientId,
      },
    });

    logger.info({
      medicationId,
      scheduledTime,
      userCount: userIds.length,
    }, 'Medication reminder sent');
  },
  {
    connection: getRedisConnection(),
    concurrency: 5,
  }
);
```

### Notification Worker

```typescript
// workers/notification.worker.ts
import { Worker, Job } from 'bullmq';
import webpush from 'web-push';
import { prisma } from '@carecircle/database';
import { logger, getConfig, getRedisConnection } from '../config';

interface NotificationPayload {
  userIds: string[];
  title: string;
  body: string;
  url?: string;
  data?: Record<string, any>;
}

// Configure web-push
const config = getConfig();
if (config.VAPID_PUBLIC_KEY && config.VAPID_PRIVATE_KEY) {
  webpush.setVapidDetails(
    config.VAPID_SUBJECT || 'mailto:admin@carecircle.com',
    config.VAPID_PUBLIC_KEY,
    config.VAPID_PRIVATE_KEY
  );
}

export const notificationWorker = new Worker<NotificationPayload>(
  'notifications',
  async (job: Job<NotificationPayload>) => {
    const { userIds, title, body, url, data } = job.data;

    for (const userId of userIds) {
      // Create in-app notification
      await prisma.notification.create({
        data: {
          userId,
          type: data?.type || 'GENERAL',
          title,
          body,
          data: data ? JSON.stringify(data) : null,
        },
      });

      // Send push notification
      const pushTokens = await prisma.pushToken.findMany({
        where: { userId },
      });

      for (const token of pushTokens) {
        try {
          const subscription = JSON.parse(token.subscriptionJson || '{}');
          
          await webpush.sendNotification(
            subscription,
            JSON.stringify({
              title,
              body,
              icon: '/icon-192x192.png',
              badge: '/badge-72x72.png',
              data: { url, ...data },
            })
          );
        } catch (error: any) {
          if (error.statusCode === 410) {
            // Subscription expired, remove it
            await prisma.pushToken.delete({ where: { id: token.id } });
            logger.info({ tokenId: token.id }, 'Removed expired push subscription');
          } else {
            logger.error({ error, tokenId: token.id }, 'Push notification failed');
          }
        }
      }
    }

    logger.info({ userCount: userIds.length }, 'Notifications sent');
  },
  {
    connection: getRedisConnection(),
    concurrency: 10,
    limiter: {
      max: 50,
      duration: 1000, // 50 notifications per second
    },
  }
);
```

### Refill Alert Worker

```typescript
// workers/refill-alert.worker.ts
import { Worker, Job } from 'bullmq';
import { prisma } from '@carecircle/database';
import { notificationQueue } from '../queues';
import { logger, getRedisConnection } from '../config';

interface RefillAlertJob {
  medicationId: string;
  medicationName: string;
  currentSupply: number;
  refillAt: number;
  familyId: string;
}

export const refillAlertWorker = new Worker<RefillAlertJob>(
  'refill-alerts',
  async (job: Job<RefillAlertJob>) => {
    const { medicationId, medicationName, currentSupply, refillAt, familyId } = job.data;

    // Get family members
    const members = await prisma.familyMember.findMany({
      where: { familyId },
      select: { userId: true },
    });

    const userIds = members.map(m => m.userId);

    // Queue notification
    await notificationQueue.add('refill-alert', {
      userIds,
      title: '💊 Refill Needed',
      body: `${medicationName} supply is low (${currentSupply} remaining). Please refill soon.`,
      url: `/medications?id=${medicationId}`,
      data: {
        type: 'REFILL_ALERT',
        medicationId,
      },
    });

    logger.info({
      medicationId,
      currentSupply,
      refillAt,
    }, 'Refill alert sent');
  },
  {
    connection: getRedisConnection(),
    concurrency: 5,
  }
);
```

## Job Types

| Worker | Job Type | Data |
|--------|----------|------|
| Medication Reminder | `reminder` | `{ medicationId, scheduledTime, reminderMinutes }` |
| Appointment Reminder | `reminder` | `{ appointmentId, dateTime, reminderMinutes }` |
| Shift Reminder | `reminder` | `{ shiftId, startTime, reminderMinutes }` |
| Notification | `notification` | `{ userIds[], title, body, url?, data? }` |
| Refill Alert | `alert` | `{ medicationId, currentSupply, refillAt }` |
| Dead Letter | `failed-job` | `{ originalQueue, jobId, data, error }` |

## Running Workers

```bash
# Development
pnpm --filter @carecircle/workers dev

# Production
pnpm --filter @carecircle/workers build
pnpm --filter @carecircle/workers start

# Docker
docker build -f apps/workers/Dockerfile -t carecircle-workers .
docker run -p 3002:3002 --env-file .env carecircle-workers
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `REDIS_HOST` | Redis host | Required |
| `REDIS_PORT` | Redis port | Required |
| `REDIS_PASSWORD` | Redis password | - |
| `DATABASE_URL` | PostgreSQL URL | Required |
| `VAPID_PUBLIC_KEY` | Push notification key | - |
| `VAPID_PRIVATE_KEY` | Push notification key | - |
| `HEALTH_PORT` | Health server port | `3002` |
| `LOG_LEVEL` | Logging level | `info` |

---

*See also: [BullMQ](bullmq.md), [Scheduled Tasks](scheduled-tasks.md)*


