# Background Jobs - Complete Guide

> **MUST REMEMBER**: Background jobs process work asynchronously outside the request-response cycle. Use job queues (BullMQ, Agenda) for tasks like sending emails, processing images, or generating reports. Key concepts: job persistence, retries with backoff, concurrency control, job priorities, and dead letter queues for failed jobs.

---

## How to Explain Like a Senior Developer

"Background jobs let you offload work that doesn't need to happen in real-time. Instead of making users wait 30 seconds while you process their uploaded video, you queue a job and return immediately. The job runs later, maybe on a different server, with proper retry handling if it fails. BullMQ with Redis is my go-to - it's fast, supports priorities, has built-in retry logic, and the dashboard helps debug production issues. The key patterns are: idempotent jobs (safe to run twice), proper error handling with dead letter queues, and graceful shutdown that finishes in-progress jobs."

---

## Core Implementation

### BullMQ Setup

```typescript
// queue-setup.ts
import { Queue, Worker, Job, QueueEvents } from 'bullmq';
import Redis from 'ioredis';

// Shared Redis connection
const connection = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT || '6379'),
  maxRetriesPerRequest: null, // Required for BullMQ
});

// Create a queue
export function createQueue(name: string): Queue {
  return new Queue(name, {
    connection,
    defaultJobOptions: {
      attempts: 3,
      backoff: {
        type: 'exponential',
        delay: 1000,
      },
      removeOnComplete: {
        age: 24 * 3600, // Keep completed jobs for 24 hours
        count: 1000,    // Keep last 1000 completed jobs
      },
      removeOnFail: {
        age: 7 * 24 * 3600, // Keep failed jobs for 7 days
      },
    },
  });
}

// Create a worker
export function createWorker<T>(
  queueName: string,
  processor: (job: Job<T>) => Promise<void>,
  options: { concurrency?: number } = {}
): Worker {
  const worker = new Worker(queueName, processor, {
    connection,
    concurrency: options.concurrency ?? 5,
  });
  
  worker.on('completed', (job) => {
    console.log(`Job ${job.id} completed`);
  });
  
  worker.on('failed', (job, err) => {
    console.error(`Job ${job?.id} failed:`, err.message);
  });
  
  worker.on('error', (err) => {
    console.error('Worker error:', err);
  });
  
  return worker;
}

// Queue events for monitoring
export function createQueueEvents(queueName: string): QueueEvents {
  return new QueueEvents(queueName, { connection });
}
```

### Email Queue Example

```typescript
// queues/email-queue.ts
import { Queue, Worker, Job } from 'bullmq';
import { createQueue, createWorker } from './queue-setup';
import { sendEmail, EmailOptions } from '../services/email-service';

interface EmailJobData {
  to: string;
  subject: string;
  template: string;
  context: Record<string, unknown>;
  attachments?: Array<{ filename: string; path: string }>;
}

// Create the email queue
export const emailQueue = createQueue('email');

// Email worker
export const emailWorker = createWorker<EmailJobData>(
  'email',
  async (job: Job<EmailJobData>) => {
    const { to, subject, template, context, attachments } = job.data;
    
    console.log(`Sending email to ${to}: ${subject}`);
    
    await sendEmail({
      to,
      subject,
      template,
      context,
      attachments,
    });
    
    console.log(`Email sent to ${to}`);
  },
  { concurrency: 10 } // 10 concurrent email sends
);

// Helper functions
export async function queueEmail(
  data: EmailJobData,
  options?: { priority?: number; delay?: number }
): Promise<string> {
  const job = await emailQueue.add('send', data, {
    priority: options?.priority,
    delay: options?.delay,
  });
  return job.id!;
}

export async function queueBulkEmails(
  emails: EmailJobData[]
): Promise<string[]> {
  const jobs = await emailQueue.addBulk(
    emails.map(data => ({
      name: 'send',
      data,
    }))
  );
  return jobs.map(j => j.id!);
}

// Usage
// await queueEmail({
//   to: 'user@example.com',
//   subject: 'Welcome!',
//   template: 'welcome',
//   context: { name: 'John' },
// });
```

### Image Processing Queue with Progress

```typescript
// queues/image-queue.ts
import { Queue, Worker, Job } from 'bullmq';
import { createQueue, createWorker } from './queue-setup';
import sharp from 'sharp';
import path from 'path';
import fs from 'fs/promises';

interface ImageJobData {
  userId: string;
  inputPath: string;
  outputDir: string;
  sizes: Array<{ width: number; height: number; suffix: string }>;
}

interface ImageJobResult {
  outputs: Array<{ size: string; path: string }>;
}

export const imageQueue = createQueue('image-processing');

export const imageWorker = createWorker<ImageJobData>(
  'image-processing',
  async (job: Job<ImageJobData>): Promise<ImageJobResult> => {
    const { inputPath, outputDir, sizes } = job.data;
    const outputs: Array<{ size: string; path: string }> = [];
    
    // Ensure output directory exists
    await fs.mkdir(outputDir, { recursive: true });
    
    const totalSteps = sizes.length;
    let completedSteps = 0;
    
    for (const size of sizes) {
      const outputPath = path.join(
        outputDir,
        `${path.basename(inputPath, path.extname(inputPath))}_${size.suffix}.webp`
      );
      
      await sharp(inputPath)
        .resize(size.width, size.height, { fit: 'cover' })
        .webp({ quality: 85 })
        .toFile(outputPath);
      
      outputs.push({
        size: size.suffix,
        path: outputPath,
      });
      
      completedSteps++;
      await job.updateProgress(Math.round((completedSteps / totalSteps) * 100));
    }
    
    return { outputs };
  },
  { concurrency: 2 } // CPU-intensive, limit concurrency
);

// Monitor progress
imageWorker.on('progress', (job, progress) => {
  console.log(`Image job ${job.id}: ${progress}% complete`);
});

// Queue image processing
export async function queueImageProcessing(
  data: ImageJobData
): Promise<string> {
  const job = await imageQueue.add('process', data, {
    priority: 1, // Normal priority
  });
  return job.id!;
}

// Wait for job completion
export async function waitForImageJob(
  jobId: string
): Promise<ImageJobResult | null> {
  const job = await imageQueue.getJob(jobId);
  if (!job) return null;
  
  return job.waitUntilFinished(
    createQueueEvents('image-processing'),
    30000 // 30 second timeout
  );
}

import { createQueueEvents } from './queue-setup';
```

### Scheduled Jobs (Cron-like)

```typescript
// queues/scheduled-jobs.ts
import { Queue, Worker, Job } from 'bullmq';
import { createQueue, createWorker } from './queue-setup';

// Queue for scheduled/repeatable jobs
export const scheduledQueue = createQueue('scheduled');

// Different job types
type ScheduledJobType = 
  | 'cleanup-expired-sessions'
  | 'send-daily-digest'
  | 'generate-reports'
  | 'sync-external-data';

interface ScheduledJobData {
  type: ScheduledJobType;
  params?: Record<string, unknown>;
}

// Worker
export const scheduledWorker = createWorker<ScheduledJobData>(
  'scheduled',
  async (job: Job<ScheduledJobData>) => {
    const { type, params } = job.data;
    
    console.log(`Running scheduled job: ${type}`);
    
    switch (type) {
      case 'cleanup-expired-sessions':
        await cleanupExpiredSessions();
        break;
        
      case 'send-daily-digest':
        await sendDailyDigest();
        break;
        
      case 'generate-reports':
        await generateReports(params);
        break;
        
      case 'sync-external-data':
        await syncExternalData();
        break;
        
      default:
        throw new Error(`Unknown job type: ${type}`);
    }
  },
  { concurrency: 1 } // Run one at a time
);

// Register repeatable jobs
export async function registerScheduledJobs(): Promise<void> {
  // Cleanup expired sessions every hour
  await scheduledQueue.add(
    'cleanup-expired-sessions',
    { type: 'cleanup-expired-sessions' },
    {
      repeat: {
        pattern: '0 * * * *', // Every hour
      },
      jobId: 'cleanup-sessions', // Prevent duplicates
    }
  );
  
  // Daily digest at 9 AM
  await scheduledQueue.add(
    'send-daily-digest',
    { type: 'send-daily-digest' },
    {
      repeat: {
        pattern: '0 9 * * *',
        tz: 'America/New_York',
      },
      jobId: 'daily-digest',
    }
  );
  
  // Weekly reports every Monday at 6 AM
  await scheduledQueue.add(
    'generate-reports',
    { type: 'generate-reports', params: { type: 'weekly' } },
    {
      repeat: {
        pattern: '0 6 * * 1', // Monday at 6 AM
      },
      jobId: 'weekly-reports',
    }
  );
  
  console.log('Scheduled jobs registered');
}

// List all repeatable jobs
export async function listRepeatableJobs(): Promise<void> {
  const repeatableJobs = await scheduledQueue.getRepeatableJobs();
  console.log('Repeatable jobs:', repeatableJobs);
}

// Job implementations
async function cleanupExpiredSessions(): Promise<void> {
  console.log('Cleaning up expired sessions...');
}

async function sendDailyDigest(): Promise<void> {
  console.log('Sending daily digest...');
}

async function generateReports(params?: Record<string, unknown>): Promise<void> {
  console.log('Generating reports...', params);
}

async function syncExternalData(): Promise<void> {
  console.log('Syncing external data...');
}
```

### Job Priorities and Dependencies

```typescript
// queues/priority-queue.ts
import { Queue, Worker, Job, FlowProducer } from 'bullmq';
import { createQueue, createWorker } from './queue-setup';
import Redis from 'ioredis';

const connection = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT || '6379'),
  maxRetriesPerRequest: null,
});

export const taskQueue = createQueue('tasks');

// Higher priority = processed first (1 is highest)
export enum JobPriority {
  CRITICAL = 1,
  HIGH = 2,
  NORMAL = 5,
  LOW = 10,
}

// Add job with priority
export async function addTask(
  name: string,
  data: unknown,
  priority: JobPriority = JobPriority.NORMAL
): Promise<string> {
  const job = await taskQueue.add(name, data, { priority });
  return job.id!;
}

// Job with dependencies (flow)
const flowProducer = new FlowProducer({ connection });

interface OrderProcessingData {
  orderId: string;
  items: Array<{ productId: string; quantity: number }>;
}

export async function queueOrderProcessing(
  data: OrderProcessingData
): Promise<void> {
  // Create a flow: parent job waits for children
  await flowProducer.add({
    name: 'complete-order',
    queueName: 'tasks',
    data: { orderId: data.orderId },
    children: [
      {
        name: 'validate-inventory',
        queueName: 'tasks',
        data: { orderId: data.orderId, items: data.items },
      },
      {
        name: 'process-payment',
        queueName: 'tasks',
        data: { orderId: data.orderId },
        children: [
          {
            name: 'validate-payment-method',
            queueName: 'tasks',
            data: { orderId: data.orderId },
          },
        ],
      },
      {
        name: 'reserve-shipping',
        queueName: 'tasks',
        data: { orderId: data.orderId },
      },
    ],
  });
}

// Worker that handles different job types
export const taskWorker = createWorker<unknown>(
  'tasks',
  async (job: Job) => {
    switch (job.name) {
      case 'validate-inventory':
        console.log('Validating inventory for order:', job.data);
        break;
        
      case 'validate-payment-method':
        console.log('Validating payment method:', job.data);
        break;
        
      case 'process-payment':
        console.log('Processing payment:', job.data);
        // Can access children results
        const childResults = await job.getChildrenValues();
        console.log('Child results:', childResults);
        break;
        
      case 'reserve-shipping':
        console.log('Reserving shipping:', job.data);
        break;
        
      case 'complete-order':
        console.log('Completing order:', job.data);
        // All children completed at this point
        const allResults = await job.getChildrenValues();
        console.log('All child results:', allResults);
        break;
        
      default:
        console.log(`Processing task: ${job.name}`, job.data);
    }
  }
);
```

### Dead Letter Queue and Error Handling

```typescript
// queues/dlq-handler.ts
import { Queue, Worker, Job, QueueEvents } from 'bullmq';
import { createQueue, createWorker } from './queue-setup';

// Dead letter queue for failed jobs
export const deadLetterQueue = createQueue('dead-letter');

// Main processing queue with DLQ integration
export const processingQueue = createQueue('processing');

// Worker with custom error handling
export const processingWorker = new Worker(
  'processing',
  async (job: Job) => {
    const { data } = job;
    
    try {
      // Attempt processing
      await processItem(data);
    } catch (error) {
      const err = error as Error;
      
      // Check if max attempts reached
      if (job.attemptsMade >= (job.opts.attempts || 3) - 1) {
        // Move to DLQ
        await deadLetterQueue.add('failed-job', {
          originalQueue: 'processing',
          originalJobId: job.id,
          originalData: job.data,
          error: {
            message: err.message,
            stack: err.stack,
          },
          failedAt: new Date().toISOString(),
          attempts: job.attemptsMade + 1,
        });
        
        console.error(`Job ${job.id} moved to DLQ after ${job.attemptsMade + 1} attempts`);
      }
      
      throw error; // Re-throw to trigger retry or mark as failed
    }
  },
  {
    connection: processingQueue.opts.connection,
    concurrency: 5,
  }
);

async function processItem(data: unknown): Promise<void> {
  // Processing logic
}

// DLQ worker for manual review/reprocessing
export const dlqWorker = createWorker(
  'dead-letter',
  async (job: Job) => {
    // Log for alerting/monitoring
    console.error('Dead letter job:', {
      id: job.id,
      originalJobId: job.data.originalJobId,
      error: job.data.error,
      failedAt: job.data.failedAt,
    });
    
    // Could send to monitoring system, Slack, etc.
    await sendAlert({
      type: 'dead-letter',
      jobId: job.data.originalJobId,
      error: job.data.error.message,
    });
  },
  { concurrency: 1 }
);

async function sendAlert(data: object): Promise<void> {
  console.log('Alert:', data);
}

// Utility to retry DLQ jobs
export async function retryDeadLetterJob(jobId: string): Promise<void> {
  const job = await deadLetterQueue.getJob(jobId);
  if (!job) {
    throw new Error(`DLQ job ${jobId} not found`);
  }
  
  // Re-add to original queue
  await processingQueue.add('retry', job.data.originalData, {
    attempts: 3,
    jobId: `retry-${job.data.originalJobId}`,
  });
  
  // Remove from DLQ
  await job.remove();
  
  console.log(`Retried DLQ job ${jobId}`);
}

// Get DLQ statistics
export async function getDLQStats(): Promise<{
  total: number;
  byError: Record<string, number>;
}> {
  const jobs = await deadLetterQueue.getJobs(['completed', 'waiting', 'active']);
  
  const byError: Record<string, number> = {};
  
  for (const job of jobs) {
    const errorMessage = job.data.error?.message || 'Unknown';
    byError[errorMessage] = (byError[errorMessage] || 0) + 1;
  }
  
  return {
    total: jobs.length,
    byError,
  };
}
```

---

## Real-World Scenarios

### Scenario 1: Video Processing Pipeline

```typescript
// video-pipeline.ts
import { Queue, Worker, Job, FlowProducer } from 'bullmq';

interface VideoJobData {
  videoId: string;
  userId: string;
  inputUrl: string;
}

const videoQueue = new Queue('video-processing');
const connection = videoQueue.opts.connection!;
const flowProducer = new FlowProducer({ connection });

// Multi-stage video processing pipeline
export async function processVideo(data: VideoJobData): Promise<void> {
  await flowProducer.add({
    name: 'complete-processing',
    queueName: 'video-processing',
    data: { videoId: data.videoId },
    children: [
      {
        name: 'generate-thumbnails',
        queueName: 'video-processing',
        data: { videoId: data.videoId, inputUrl: data.inputUrl },
      },
      {
        name: 'transcode',
        queueName: 'video-processing',
        data: { videoId: data.videoId, inputUrl: data.inputUrl },
        children: [
          {
            name: 'transcode-1080p',
            queueName: 'video-processing',
            data: { ...data, quality: '1080p' },
          },
          {
            name: 'transcode-720p',
            queueName: 'video-processing',
            data: { ...data, quality: '720p' },
          },
          {
            name: 'transcode-480p',
            queueName: 'video-processing',
            data: { ...data, quality: '480p' },
          },
        ],
      },
      {
        name: 'extract-audio',
        queueName: 'video-processing',
        data: { videoId: data.videoId, inputUrl: data.inputUrl },
      },
    ],
  });
}

// Worker for video jobs
const videoWorker = new Worker(
  'video-processing',
  async (job: Job) => {
    console.log(`Processing ${job.name} for video ${job.data.videoId}`);
    
    switch (job.name) {
      case 'transcode-1080p':
      case 'transcode-720p':
      case 'transcode-480p':
        await transcodeVideo(job.data.inputUrl, job.data.quality);
        break;
        
      case 'generate-thumbnails':
        await generateThumbnails(job.data.inputUrl);
        break;
        
      case 'extract-audio':
        await extractAudio(job.data.inputUrl);
        break;
        
      case 'transcode':
        // Parent job - just collects results
        const transcodeResults = await job.getChildrenValues();
        console.log('Transcode results:', transcodeResults);
        break;
        
      case 'complete-processing':
        // Final job - all processing done
        const allResults = await job.getChildrenValues();
        await notifyUser(job.data.videoId, allResults);
        break;
    }
  },
  {
    connection,
    concurrency: 2, // Limit concurrent video processing
  }
);

async function transcodeVideo(url: string, quality: string): Promise<void> {
  console.log(`Transcoding to ${quality}...`);
  await new Promise(resolve => setTimeout(resolve, 5000));
}

async function generateThumbnails(url: string): Promise<void> {
  console.log('Generating thumbnails...');
  await new Promise(resolve => setTimeout(resolve, 2000));
}

async function extractAudio(url: string): Promise<void> {
  console.log('Extracting audio...');
  await new Promise(resolve => setTimeout(resolve, 3000));
}

async function notifyUser(videoId: string, results: unknown): Promise<void> {
  console.log(`Video ${videoId} processing complete!`, results);
}
```

### Scenario 2: Rate-Limited External API Calls

```typescript
// rate-limited-queue.ts
import { Queue, Worker, Job, RateLimiter } from 'bullmq';
import Redis from 'ioredis';

const connection = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  maxRetriesPerRequest: null,
});

// Queue with rate limiting
const apiQueue = new Queue('external-api', {
  connection,
  defaultJobOptions: {
    attempts: 5,
    backoff: {
      type: 'exponential',
      delay: 2000,
    },
  },
});

// Rate-limited worker: max 100 requests per minute
const apiWorker = new Worker(
  'external-api',
  async (job: Job) => {
    const { endpoint, payload } = job.data;
    
    const response = await fetch(endpoint, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    });
    
    if (response.status === 429) {
      // Rate limited by API - throw to retry
      const retryAfter = response.headers.get('Retry-After');
      throw new Error(`Rate limited. Retry after ${retryAfter}s`);
    }
    
    if (!response.ok) {
      throw new Error(`API error: ${response.status}`);
    }
    
    return response.json();
  },
  {
    connection,
    concurrency: 10,
    limiter: {
      max: 100,      // Max 100 jobs
      duration: 60000, // Per minute
    },
  }
);

// Add API calls with automatic rate limiting
export async function queueAPICall(
  endpoint: string,
  payload: unknown
): Promise<string> {
  const job = await apiQueue.add('api-call', { endpoint, payload });
  return job.id!;
}
```

---

## Common Pitfalls

### 1. Non-Idempotent Jobs

```typescript
// ❌ BAD: Not idempotent - running twice charges customer twice
async function processPayment(job: Job): Promise<void> {
  await chargeCustomer(job.data.customerId, job.data.amount);
}

// ✅ GOOD: Idempotent - safe to run multiple times
async function processPayment(job: Job): Promise<void> {
  const { customerId, paymentId, amount } = job.data;
  
  // Check if already processed
  const existing = await db.payments.findByPaymentId(paymentId);
  if (existing) {
    console.log(`Payment ${paymentId} already processed`);
    return;
  }
  
  await chargeCustomer(customerId, amount);
  await db.payments.markProcessed(paymentId);
}

async function chargeCustomer(id: string, amount: number): Promise<void> {}
const db = { payments: { findByPaymentId: async (id: string) => null, markProcessed: async (id: string) => {} } };
```

### 2. Not Handling Worker Shutdown

```typescript
// ❌ BAD: Abrupt shutdown loses jobs
process.on('SIGTERM', () => process.exit(0));

// ✅ GOOD: Graceful shutdown
process.on('SIGTERM', async () => {
  console.log('Shutting down workers...');
  
  await Promise.all([
    emailWorker.close(),
    imageWorker.close(),
  ]);
  
  console.log('Workers closed');
  process.exit(0);
});

// Mock workers
const emailWorker = { close: async () => {} };
const imageWorker = { close: async () => {} };
```

### 3. Missing Error Context

```typescript
// ❌ BAD: Lost error context
worker.on('failed', (job, err) => {
  console.log('Job failed');
});

// ✅ GOOD: Full error context
worker.on('failed', (job, err) => {
  console.error('Job failed', {
    jobId: job?.id,
    jobName: job?.name,
    data: job?.data,
    error: err.message,
    stack: err.stack,
    attempts: job?.attemptsMade,
  });
});

const worker = { on: (event: string, handler: Function) => {} };
```

---

## Interview Questions

### Q1: What's the difference between BullMQ and Agenda?

**A:** BullMQ uses Redis and excels at high-throughput, real-time job processing with features like priorities, rate limiting, and job flows. Agenda uses MongoDB and is better for scheduled/cron jobs with more flexible scheduling. Choose BullMQ for queue-heavy workloads, Agenda for scheduling-focused needs.

### Q2: How do you ensure jobs are processed exactly once?

**A:** Make jobs idempotent - safe to run multiple times with the same result. Use unique job IDs to check if already processed. Store job completion status in a database. For critical operations like payments, use distributed locks or database transactions to prevent duplicate processing.

### Q3: How do you handle job failures?

**A:** Configure retry strategies with exponential backoff. After max retries, move failed jobs to a dead letter queue for manual review. Log comprehensive error context. Set up alerts for DLQ growth. Implement retry mechanisms for DLQ jobs after fixing the underlying issue.

### Q4: How do you scale background job processing?

**A:** Add more worker instances - they automatically distribute work from Redis. Tune concurrency per worker based on job type (CPU vs I/O bound). Use job priorities for important work. Implement rate limiting for external API calls. Monitor queue depth and worker throughput.

---

## Quick Reference Checklist

### Job Design
- [ ] Make jobs idempotent
- [ ] Include all necessary data in job payload
- [ ] Set appropriate retry strategies
- [ ] Add job timeouts

### Queue Configuration
- [ ] Configure dead letter queues
- [ ] Set job retention policies
- [ ] Implement rate limiting if needed
- [ ] Use priorities for critical jobs

### Worker Management
- [ ] Handle graceful shutdown
- [ ] Tune concurrency appropriately
- [ ] Log job lifecycle events
- [ ] Monitor worker health

### Monitoring
- [ ] Track queue depth
- [ ] Monitor job processing times
- [ ] Alert on DLQ growth
- [ ] Track job success/failure rates

---

*Last updated: February 2026*

