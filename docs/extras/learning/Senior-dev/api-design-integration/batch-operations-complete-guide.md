# 📦 Batch Operations - Complete Guide

> A comprehensive guide to batch operations in APIs - bulk endpoints, partial failures, transactions, and efficiently processing multiple items in a single request.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Batch operations allow clients to perform multiple actions in a single API request, reducing network overhead, improving performance, and enabling atomic operations across multiple resources."

### The 7 Key Concepts (Remember These!)
```
1. BULK CREATE      → POST /users/batch with array of items
2. BULK UPDATE      → PATCH /users/batch with array of updates
3. BULK DELETE      → DELETE /users/batch or POST /users/batch-delete
4. PARTIAL FAILURE  → Some items fail, others succeed
5. TRANSACTIONS     → All or nothing atomicity
6. ASYNC BATCH      → Long-running batch jobs
7. IDEMPOTENCY      → Safe retries for batches
```

### Batch Response Patterns
```
┌─────────────────────────────────────────────────────────────────┐
│                 BATCH RESPONSE PATTERNS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PATTERN 1: All-or-Nothing (Transactional)                     │
│  ─────────────────────────────────────────                      │
│  • Either all succeed or all fail                              │
│  • Database transaction wraps all operations                   │
│  • HTTP 200 (all success) or 400/500 (all failed)              │
│  • Simpler error handling                                      │
│  Use: Financial operations, related items                      │
│                                                                 │
│  PATTERN 2: Partial Success                                    │
│  ──────────────────────────                                     │
│  • Some items succeed, some fail                               │
│  • Return status per item                                      │
│  • HTTP 207 Multi-Status                                       │
│  • More complex but more flexible                              │
│  Use: Bulk imports, independent items                          │
│                                                                 │
│  PATTERN 3: Async Processing                                   │
│  ───────────────────────────                                    │
│  • Return 202 Accepted immediately                             │
│  • Process in background                                       │
│  • Client polls status endpoint                                │
│  • Or webhook notification                                     │
│  Use: Large batches, long processing time                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Batch API Design
```
┌─────────────────────────────────────────────────────────────────┐
│                    BATCH API DESIGN                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  REQUEST                                                        │
│  ───────                                                        │
│  POST /users/batch                                             │
│  {                                                             │
│    "operations": [                                             │
│      { "method": "create", "data": { "name": "John" } },       │
│      { "method": "update", "id": "123", "data": {...} },       │
│      { "method": "delete", "id": "456" }                       │
│    ]                                                           │
│  }                                                             │
│                                                                 │
│  RESPONSE (Partial Success - 207)                              │
│  ────────────────────────────────                               │
│  {                                                             │
│    "results": [                                                │
│      { "index": 0, "status": 201, "data": {...} },            │
│      { "index": 1, "status": 200, "data": {...} },            │
│      { "index": 2, "status": 404, "error": "Not found" }      │
│    ],                                                          │
│    "summary": {                                                │
│      "total": 3,                                               │
│      "succeeded": 2,                                           │
│      "failed": 1                                               │
│    }                                                           │
│  }                                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Multi-Status"** | "We return HTTP 207 Multi-Status for partial success" |
| **"Transactional batch"** | "Financial operations use transactional batches" |
| **"Idempotency key"** | "Batch requests require idempotency keys for safe retries" |
| **"Fan-out"** | "Gateway fans out batch to individual service calls" |
| **"Bulk write"** | "Database bulk write for efficient inserts" |
| **"Partial failure"** | "We handle partial failures with per-item status" |

### Key Numbers to Remember
| Metric | Recommendation | Why |
|--------|----------------|-----|
| Max batch size | **100-1000** | Balance performance vs timeout |
| Timeout | **30-60 seconds** | For sync batches |
| Async threshold | **> 100 items** | Switch to async |
| Retry delay | **Exponential** | 1s, 2s, 4s, 8s... |

### The "Wow" Statement (Memorize This!)
> "We support batch operations for bulk imports and multi-item updates. For user imports, POST /users/batch accepts up to 1000 items. We return 207 Multi-Status with per-item results - some can succeed while others fail. Each result includes index, status, and data or error. Idempotency keys allow safe retries - same key returns cached result. For large batches (>100 items), we switch to async: return 202 with job ID, client polls /jobs/{id} for status or receives webhook. Transactional batches available for related items where all-or-nothing is needed. Database uses bulk inserts for performance - 1000 items in one query instead of 1000 queries."

---

## 📚 Table of Contents

1. [Sync Batch Operations](#1-sync-batch-operations)
2. [Partial Failure Handling](#2-partial-failure-handling)
3. [Transactional Batches](#3-transactional-batches)
4. [Async Batch Processing](#4-async-batch-processing)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Sync Batch Operations

```typescript
// ════════════════════════════════════════════════════════════════
// BULK CREATE ENDPOINT
// ════════════════════════════════════════════════════════════════

// POST /users/batch
interface BatchCreateRequest {
  items: CreateUserInput[];
  options?: {
    stopOnError?: boolean;      // All-or-nothing
    validateOnly?: boolean;     // Dry run
  };
}

interface BatchResult<T> {
  index: number;
  status: number;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: any;
  };
}

interface BatchResponse<T> {
  results: BatchResult<T>[];
  summary: {
    total: number;
    succeeded: number;
    failed: number;
  };
}

// Controller
app.post('/users/batch', async (req, res) => {
  const { items, options } = req.body as BatchCreateRequest;

  // Validate batch size
  if (items.length > 1000) {
    return res.status(400).json({
      error: 'batch_too_large',
      message: 'Maximum batch size is 1000 items',
      max_size: 1000,
    });
  }

  const results: BatchResult<User>[] = [];
  let succeeded = 0;
  let failed = 0;

  for (let i = 0; i < items.length; i++) {
    try {
      // Validate
      const validationErrors = validateUser(items[i]);
      if (validationErrors.length > 0) {
        results.push({
          index: i,
          status: 422,
          error: {
            code: 'validation_error',
            message: 'Validation failed',
            details: validationErrors,
          },
        });
        failed++;
        
        if (options?.stopOnError) {
          break;
        }
        continue;
      }

      // Create user
      const user = await userService.create(items[i]);
      results.push({
        index: i,
        status: 201,
        data: user,
      });
      succeeded++;
    } catch (error) {
      results.push({
        index: i,
        status: error.status || 500,
        error: {
          code: error.code || 'internal_error',
          message: error.message,
        },
      });
      failed++;
      
      if (options?.stopOnError) {
        break;
      }
    }
  }

  // Determine HTTP status
  const httpStatus = failed === 0 ? 200 : (succeeded === 0 ? 400 : 207);

  res.status(httpStatus).json({
    results,
    summary: {
      total: items.length,
      succeeded,
      failed,
    },
  });
});

// ════════════════════════════════════════════════════════════════
// BULK UPDATE ENDPOINT
// ════════════════════════════════════════════════════════════════

// PATCH /users/batch
interface BatchUpdateRequest {
  updates: Array<{
    id: string;
    data: UpdateUserInput;
  }>;
}

app.patch('/users/batch', async (req, res) => {
  const { updates } = req.body as BatchUpdateRequest;

  if (updates.length > 100) {
    return res.status(400).json({ error: 'batch_too_large' });
  }

  const results: BatchResult<User>[] = [];

  // Process updates in parallel (with concurrency limit)
  await Promise.all(
    updates.map(async (update, index) => {
      try {
        const user = await userService.update(update.id, update.data);
        results[index] = {
          index,
          status: 200,
          data: user,
        };
      } catch (error) {
        results[index] = {
          index,
          status: error.status || 500,
          error: {
            code: error.code || 'update_failed',
            message: error.message,
          },
        };
      }
    })
  );

  const summary = results.reduce(
    (acc, r) => {
      if (r.status >= 200 && r.status < 300) acc.succeeded++;
      else acc.failed++;
      return acc;
    },
    { total: results.length, succeeded: 0, failed: 0 }
  );

  const httpStatus = summary.failed === 0 ? 200 : (summary.succeeded === 0 ? 400 : 207);
  res.status(httpStatus).json({ results, summary });
});

// ════════════════════════════════════════════════════════════════
// BULK DELETE ENDPOINT
// ════════════════════════════════════════════════════════════════

// Using POST (DELETE with body is not always supported)
// POST /users/batch-delete
interface BatchDeleteRequest {
  ids: string[];
}

app.post('/users/batch-delete', async (req, res) => {
  const { ids } = req.body as BatchDeleteRequest;

  if (ids.length > 100) {
    return res.status(400).json({ error: 'batch_too_large' });
  }

  const results: BatchResult<{ id: string }>[] = [];

  for (let i = 0; i < ids.length; i++) {
    try {
      await userService.delete(ids[i]);
      results.push({
        index: i,
        status: 204,
        data: { id: ids[i] },
      });
    } catch (error) {
      results.push({
        index: i,
        status: error.status || 500,
        error: {
          code: error.code,
          message: error.message,
        },
      });
    }
  }

  const succeeded = results.filter(r => r.status === 204).length;
  const failed = results.length - succeeded;

  const httpStatus = failed === 0 ? 200 : (succeeded === 0 ? 400 : 207);
  res.status(httpStatus).json({
    results,
    summary: { total: ids.length, succeeded, failed },
  });
});
```

---

## 2. Partial Failure Handling

```typescript
// ════════════════════════════════════════════════════════════════
// DETAILED PARTIAL FAILURE RESPONSE
// ════════════════════════════════════════════════════════════════

interface DetailedBatchResponse<T> {
  results: Array<{
    index: number;
    correlationId?: string;  // Client-provided ID for matching
    status: number;
    success: boolean;
    data?: T;
    error?: {
      code: string;
      message: string;
      field?: string;
      details?: any;
    };
  }>;
  summary: {
    total: number;
    succeeded: number;
    failed: number;
    errors: {
      [errorCode: string]: number;  // Count per error type
    };
  };
  _links: {
    retry?: string;  // URL to retry failed items
  };
}

// Example response
const exampleResponse: DetailedBatchResponse<User> = {
  results: [
    {
      index: 0,
      correlationId: 'client-id-1',
      status: 201,
      success: true,
      data: { id: 'usr_123', name: 'John', email: 'john@example.com' },
    },
    {
      index: 1,
      correlationId: 'client-id-2',
      status: 409,
      success: false,
      error: {
        code: 'duplicate_email',
        message: 'User with this email already exists',
        field: 'email',
      },
    },
    {
      index: 2,
      correlationId: 'client-id-3',
      status: 422,
      success: false,
      error: {
        code: 'validation_error',
        message: 'Validation failed',
        details: [
          { field: 'email', message: 'Invalid email format' },
          { field: 'name', message: 'Name is required' },
        ],
      },
    },
  ],
  summary: {
    total: 3,
    succeeded: 1,
    failed: 2,
    errors: {
      duplicate_email: 1,
      validation_error: 1,
    },
  },
  _links: {
    retry: '/users/batch?retry_failed=true',
  },
};

// ════════════════════════════════════════════════════════════════
// CLIENT-SIDE PARTIAL FAILURE HANDLING
// ════════════════════════════════════════════════════════════════

async function batchCreateUsers(users: CreateUserInput[]): Promise<{
  created: User[];
  failed: Array<{ input: CreateUserInput; error: any }>;
}> {
  const response = await fetch('/api/users/batch', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ items: users }),
  });

  const result: DetailedBatchResponse<User> = await response.json();

  const created: User[] = [];
  const failed: Array<{ input: CreateUserInput; error: any }> = [];

  for (const item of result.results) {
    if (item.success && item.data) {
      created.push(item.data);
    } else {
      failed.push({
        input: users[item.index],
        error: item.error,
      });
    }
  }

  return { created, failed };
}

// Usage with retry logic
async function batchCreateWithRetry(users: CreateUserInput[], maxRetries = 3) {
  let remaining = users;
  let allCreated: User[] = [];
  let retryCount = 0;

  while (remaining.length > 0 && retryCount < maxRetries) {
    const { created, failed } = await batchCreateUsers(remaining);
    allCreated = [...allCreated, ...created];

    // Filter out permanent failures (don't retry validation errors)
    remaining = failed
      .filter(f => isRetryableError(f.error))
      .map(f => f.input);

    if (remaining.length > 0) {
      retryCount++;
      await sleep(Math.pow(2, retryCount) * 1000); // Exponential backoff
    }
  }

  return {
    created: allCreated,
    permanentlyFailed: remaining,
  };
}

function isRetryableError(error: any): boolean {
  // Don't retry validation or duplicate errors
  const nonRetryable = ['validation_error', 'duplicate_email', 'not_found'];
  return !nonRetryable.includes(error?.code);
}
```

---

## 3. Transactional Batches

```typescript
// ════════════════════════════════════════════════════════════════
// ALL-OR-NOTHING TRANSACTIONAL BATCH
// ════════════════════════════════════════════════════════════════

// POST /orders/batch-create
interface TransactionalBatchRequest {
  items: CreateOrderInput[];
  transaction: {
    required: true;  // All-or-nothing
  };
}

app.post('/orders/batch-create', async (req, res) => {
  const { items, transaction } = req.body;

  if (items.length > 100) {
    return res.status(400).json({ error: 'batch_too_large' });
  }

  // Wrap in database transaction
  try {
    const orders = await prisma.$transaction(async (tx) => {
      const created: Order[] = [];

      for (const item of items) {
        // Validate stock
        const product = await tx.product.findUnique({
          where: { id: item.productId },
        });

        if (!product || product.stock < item.quantity) {
          // Throwing here rolls back entire transaction
          throw new Error(`Insufficient stock for product ${item.productId}`);
        }

        // Create order
        const order = await tx.order.create({
          data: item,
        });

        // Decrement stock
        await tx.product.update({
          where: { id: item.productId },
          data: { stock: { decrement: item.quantity } },
        });

        created.push(order);
      }

      return created;
    });

    res.status(201).json({
      success: true,
      data: orders,
      summary: {
        total: orders.length,
        succeeded: orders.length,
        failed: 0,
      },
    });
  } catch (error) {
    // Transaction rolled back
    res.status(400).json({
      success: false,
      error: {
        code: 'transaction_failed',
        message: error.message,
      },
      summary: {
        total: items.length,
        succeeded: 0,
        failed: items.length,
      },
    });
  }
});

// ════════════════════════════════════════════════════════════════
// MIXED OPERATIONS IN TRANSACTION
// ════════════════════════════════════════════════════════════════

// POST /batch-operations
interface MixedBatchRequest {
  operations: Array<{
    type: 'create' | 'update' | 'delete';
    resource: string;
    id?: string;
    data?: any;
  }>;
  atomic?: boolean;
}

app.post('/batch-operations', async (req, res) => {
  const { operations, atomic = false } = req.body;

  if (atomic) {
    // All-or-nothing
    try {
      const results = await prisma.$transaction(async (tx) => {
        const results = [];

        for (const op of operations) {
          const result = await executeOperation(tx, op);
          results.push(result);
        }

        return results;
      });

      res.json({ success: true, results });
    } catch (error) {
      res.status(400).json({ success: false, error: error.message });
    }
  } else {
    // Partial success allowed
    const results = [];
    
    for (const op of operations) {
      try {
        const result = await executeOperation(prisma, op);
        results.push({ success: true, ...result });
      } catch (error) {
        results.push({ success: false, error: error.message });
      }
    }

    const succeeded = results.filter(r => r.success).length;
    const httpStatus = succeeded === results.length ? 200 : 
                       succeeded === 0 ? 400 : 207;

    res.status(httpStatus).json({ results });
  }
});

async function executeOperation(client: PrismaClient, op: any) {
  switch (op.type) {
    case 'create':
      return { type: 'create', data: await client[op.resource].create({ data: op.data }) };
    case 'update':
      return { type: 'update', data: await client[op.resource].update({ where: { id: op.id }, data: op.data }) };
    case 'delete':
      await client[op.resource].delete({ where: { id: op.id } });
      return { type: 'delete', id: op.id };
    default:
      throw new Error(`Unknown operation type: ${op.type}`);
  }
}
```

---

## 4. Async Batch Processing

```typescript
// ════════════════════════════════════════════════════════════════
// ASYNC BATCH JOB
// ════════════════════════════════════════════════════════════════

import { Queue, Job } from 'bullmq';

const batchQueue = new Queue('batch-processing');

// POST /users/batch-import
interface BatchImportRequest {
  items: CreateUserInput[];
  webhookUrl?: string;  // Notify when complete
}

interface BatchJob {
  id: string;
  status: 'pending' | 'processing' | 'completed' | 'failed';
  progress: number;
  total: number;
  succeeded: number;
  failed: number;
  results?: BatchResult<any>[];
  error?: string;
  createdAt: string;
  completedAt?: string;
}

// Submit batch job
app.post('/users/batch-import', async (req, res) => {
  const { items, webhookUrl } = req.body as BatchImportRequest;

  // Validate
  if (items.length > 10000) {
    return res.status(400).json({ error: 'batch_too_large', max: 10000 });
  }

  // For small batches, process synchronously
  if (items.length <= 100) {
    const result = await processBatchSync(items);
    return res.json(result);
  }

  // Create async job
  const jobId = crypto.randomUUID();
  
  await batchQueue.add('import-users', {
    jobId,
    items,
    webhookUrl,
  }, {
    jobId,
    removeOnComplete: false,
    removeOnFail: false,
  });

  // Store job metadata
  await redis.hset(`batch:${jobId}`, {
    status: 'pending',
    total: items.length,
    progress: 0,
    succeeded: 0,
    failed: 0,
    createdAt: new Date().toISOString(),
  });

  // Return 202 Accepted with job location
  res.status(202)
    .header('Location', `/jobs/${jobId}`)
    .json({
      jobId,
      status: 'pending',
      total: items.length,
      statusUrl: `/jobs/${jobId}`,
    });
});

// Job status endpoint
app.get('/jobs/:jobId', async (req, res) => {
  const { jobId } = req.params;
  
  const jobData = await redis.hgetall(`batch:${jobId}`);
  
  if (!jobData || Object.keys(jobData).length === 0) {
    return res.status(404).json({ error: 'Job not found' });
  }

  const job: BatchJob = {
    id: jobId,
    status: jobData.status as BatchJob['status'],
    progress: parseInt(jobData.progress),
    total: parseInt(jobData.total),
    succeeded: parseInt(jobData.succeeded),
    failed: parseInt(jobData.failed),
    createdAt: jobData.createdAt,
    completedAt: jobData.completedAt,
  };

  // Include results if completed
  if (job.status === 'completed' || job.status === 'failed') {
    const results = await redis.get(`batch:${jobId}:results`);
    if (results) {
      job.results = JSON.parse(results);
    }
    if (jobData.error) {
      job.error = jobData.error;
    }
  }

  res.json(job);
});

// ════════════════════════════════════════════════════════════════
// BATCH WORKER
// ════════════════════════════════════════════════════════════════

import { Worker } from 'bullmq';

const worker = new Worker('batch-processing', async (job: Job) => {
  const { jobId, items, webhookUrl } = job.data;
  const results: BatchResult<User>[] = [];
  let succeeded = 0;
  let failed = 0;

  // Update status to processing
  await redis.hset(`batch:${jobId}`, { status: 'processing' });

  // Process in chunks
  const chunkSize = 50;
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    
    // Process chunk
    const chunkResults = await Promise.all(
      chunk.map(async (item, index) => {
        try {
          const user = await userService.create(item);
          succeeded++;
          return {
            index: i + index,
            status: 201,
            success: true,
            data: user,
          };
        } catch (error) {
          failed++;
          return {
            index: i + index,
            status: error.status || 500,
            success: false,
            error: {
              code: error.code,
              message: error.message,
            },
          };
        }
      })
    );

    results.push(...chunkResults);

    // Update progress
    const progress = Math.floor(((i + chunk.length) / items.length) * 100);
    await redis.hset(`batch:${jobId}`, {
      progress,
      succeeded,
      failed,
    });

    // Update job progress for BullMQ
    await job.updateProgress(progress);
  }

  // Store results
  await redis.set(
    `batch:${jobId}:results`,
    JSON.stringify(results),
    'EX',
    86400 // 24 hour expiry
  );

  // Update final status
  await redis.hset(`batch:${jobId}`, {
    status: failed === items.length ? 'failed' : 'completed',
    progress: 100,
    completedAt: new Date().toISOString(),
  });

  // Webhook notification
  if (webhookUrl) {
    await fetch(webhookUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        jobId,
        status: 'completed',
        summary: { total: items.length, succeeded, failed },
      }),
    }).catch(console.error);
  }

  return { succeeded, failed };
});

// ════════════════════════════════════════════════════════════════
// POLLING CLIENT
// ════════════════════════════════════════════════════════════════

async function submitAndWaitForBatch(items: CreateUserInput[]): Promise<BatchJob> {
  // Submit batch
  const response = await fetch('/api/users/batch-import', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ items }),
  });

  if (response.status === 200) {
    // Processed synchronously
    return response.json();
  }

  // Async processing - poll for completion
  const { jobId, statusUrl } = await response.json();
  
  return pollJobStatus(statusUrl);
}

async function pollJobStatus(statusUrl: string): Promise<BatchJob> {
  const maxAttempts = 60;
  const intervalMs = 2000;

  for (let attempt = 0; attempt < maxAttempts; attempt++) {
    const response = await fetch(statusUrl);
    const job: BatchJob = await response.json();

    if (job.status === 'completed' || job.status === 'failed') {
      return job;
    }

    // Log progress
    console.log(`Job progress: ${job.progress}%`);

    await sleep(intervalMs);
  }

  throw new Error('Job timed out');
}
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# BATCH OPERATIONS BEST PRACTICES
# ════════════════════════════════════════════════════════════════

api_design:
  endpoints:
    - POST /resources/batch (create)
    - PATCH /resources/batch (update)
    - POST /resources/batch-delete (delete)
    
  request_format:
    - Accept array of items
    - Include operation type for mixed batches
    - Support correlation IDs for tracking
    - Include options (atomic, validateOnly)
    
  response_format:
    - Per-item status with index
    - Summary (total, succeeded, failed)
    - Use 207 Multi-Status for partial success
    - Include error details per item

limits:
  batch_size:
    sync_max: 100-1000
    async_threshold: 100+
    absolute_max: 10000
    
  timeouts:
    sync: 30-60 seconds
    async: hours/days
    
  rate_limits:
    - Per batch request
    - Per total items across requests

idempotency:
  - Require idempotency key for batch mutations
  - Cache results by idempotency key
  - Return cached result on retry
  - Set reasonable TTL (24-48 hours)

database:
  - Use bulk insert for creates
  - Use batch updates (updateMany)
  - Process in chunks to avoid timeouts
  - Use transactions for related items

monitoring:
  - Track batch sizes
  - Track success/failure rates
  - Alert on high failure rates
  - Monitor processing times
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# BATCH OPERATIONS PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: No batch size limit
# Bad
POST /users/batch
{ "items": [... 1 million items ...] }
# Server times out or runs out of memory

# Good
if (items.length > 1000) {
  return res.status(400).json({ error: 'batch_too_large', max: 1000 });
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Insert one by one
# Bad
for (const item of items) {
  await db.user.create({ data: item });  # N queries!
}

# Good
await db.user.createMany({ data: items });  # 1 query

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Unclear partial failure
# Bad
HTTP 200 OK
{ "message": "Some items failed" }
# Which items? Why?

# Good
HTTP 207 Multi-Status
{
  "results": [
    { "index": 0, "status": 201, "data": {...} },
    { "index": 1, "status": 409, "error": {...} }
  ],
  "summary": { "succeeded": 1, "failed": 1 }
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: No idempotency
# Bad
# Client retries failed request
# Items created twice

# Good
# Require idempotency key
# Cache and return same result on retry

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Long sync processing
# Bad
# 10000 items processed synchronously
# Request times out at 30s, items partially created

# Good
# Switch to async for large batches
# Return 202 with job ID
# Client polls for status

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: No progress tracking
# Bad
# Large async batch
# No way to know progress
# User refreshes and resubmits

# Good
# Store progress in Redis
# Expose progress endpoint
# Optional: webhook on completion
```

---

## 7. Interview Questions

### Basic Questions

**Q: "Why would you use batch operations?"**
> "Batch operations reduce network overhead (one request vs many), improve performance (bulk database operations), and enable atomicity for related items. Instead of 1000 HTTP requests, send one with 1000 items. Database can optimize bulk inserts."

**Q: "What HTTP status code for partial failure?"**
> "207 Multi-Status. Indicates some items succeeded, some failed. Response body contains per-item results with individual status codes. 200 if all succeed, 400 if all fail, 207 for mixed. Makes it clear to clients there's partial success."

**Q: "How do you handle errors in batch operations?"**
> "Return per-item results with index, status, and error details. Include summary (total, succeeded, failed). Client can identify which items failed and why. Support retry with idempotency key. Distinguish retriable vs permanent failures."

### Intermediate Questions

**Q: "When would you use transactional vs partial success batches?"**
> "Transactional (all-or-nothing): Related items that must all succeed - financial operations, order with items. Partial success: Independent items - user import, email sending. Choose based on business requirements. Offer both via option flag."

**Q: "How do you handle large batches?"**
> "For small batches (< 100), process synchronously. For large batches, switch to async: return 202 Accepted with job ID, process in background, client polls status endpoint. Process in chunks to avoid timeouts. Store progress for monitoring."

**Q: "How do you make batch operations idempotent?"**
> "Require Idempotency-Key header. Hash and cache result by key. On retry, return cached result instead of processing again. Set TTL (24-48h). Include idempotency key in per-item results for tracking. Critical for retry safety."

### Advanced Questions

**Q: "How would you implement a generic batch endpoint?"**
> "Accept array of operations with type, resource, data. Support create/update/delete in single request. Process in order or parallel based on option. Return per-operation results. Support atomic mode for transactions. Example: Google's batch API pattern."

**Q: "How do you scale batch processing?"**
> "Queue-based processing (BullMQ, SQS). Multiple workers processing jobs. Chunk large batches (process 50 at a time). Horizontal scaling of workers. Priority queues for different batch sizes. Dead letter queue for failures. Monitor queue depth."

**Q: "How do you handle partial failures in distributed systems?"**
> "Saga pattern for distributed transactions. Each step has compensating action. On failure, execute compensations for completed steps. Or: accept eventual consistency, have reconciliation process. Track correlation IDs across services. Consider outbox pattern."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│               BATCH OPERATIONS CHECKLIST                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  API DESIGN:                                                    │
│  □ POST /resource/batch for batch ops                          │
│  □ Accept array of items                                       │
│  □ Return per-item results with index                          │
│  □ Use 207 Multi-Status for partial success                    │
│  □ Include summary (total, succeeded, failed)                  │
│                                                                 │
│  SAFETY:                                                        │
│  □ Enforce max batch size                                      │
│  □ Require idempotency key                                     │
│  □ Support atomic/transactional option                         │
│  □ Use bulk database operations                                │
│                                                                 │
│  LARGE BATCHES:                                                 │
│  □ Switch to async (202 Accepted)                              │
│  □ Return job ID and status URL                                │
│  □ Progress tracking                                           │
│  □ Optional webhook notification                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

HTTP STATUS CODES:
┌────────────────────────────────────────────────────────────────┐
│ 200: All succeeded                                             │
│ 202: Accepted for async processing                             │
│ 207: Multi-Status (partial success)                            │
│ 400: All failed / Invalid request                              │
└────────────────────────────────────────────────────────────────┘

RESPONSE STRUCTURE:
┌────────────────────────────────────────────────────────────────┐
│ {                                                              │
│   "results": [                                                 │
│     { "index": 0, "status": 201, "data": {...} },             │
│     { "index": 1, "status": 400, "error": {...} }             │
│   ],                                                           │
│   "summary": { "total": 2, "succeeded": 1, "failed": 1 }      │
│ }                                                              │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

