# 🔁 Idempotency - Complete Guide

> A comprehensive guide to idempotency in APIs - idempotency keys, safe retries, designing idempotent operations, and preventing duplicate actions.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "An idempotent operation produces the same result regardless of how many times it's executed with the same input - making it safe to retry without side effects like duplicate charges, duplicate orders, or duplicate emails."

### The 7 Key Concepts (Remember These!)
```
1. IDEMPOTENT KEY    → Client-provided unique ID per operation
2. REQUEST HASH      → Hash of request body for duplicate detection
3. RESULT CACHING    → Store and return cached result on retry
4. NATURAL IDEMPOTENCY → GET, PUT, DELETE are inherently idempotent
5. ARTIFICIAL IDEMPOTENCY → POST made idempotent via idempotency key
6. AT-MOST-ONCE      → Operation happens at most one time
7. EXACTLY-ONCE     → Operation happens exactly one time (goal)
```

### HTTP Methods and Idempotency
```
┌─────────────────────────────────────────────────────────────────┐
│              HTTP METHODS AND IDEMPOTENCY                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  METHOD  │ IDEMPOTENT │ SAFE │ EXPLANATION                     │
│  ────────┼────────────┼──────┼──────────────────────────────── │
│  GET     │ Yes        │ Yes  │ Reading doesn't change state   │
│  HEAD    │ Yes        │ Yes  │ Same as GET without body       │
│  OPTIONS │ Yes        │ Yes  │ Metadata only                  │
│  PUT     │ Yes        │ No   │ Replace resource = same result │
│  DELETE  │ Yes        │ No   │ Deleting twice = still gone    │
│  POST    │ No         │ No   │ Each call may create new item  │
│  PATCH   │ No*        │ No   │ Depends on implementation      │
│                                                                 │
│  * PATCH can be idempotent if it sets absolute values          │
│    (name: "John") vs relative (increment: 1)                   │
│                                                                 │
│  SAFE: Doesn't modify resources                                │
│  IDEMPOTENT: Multiple calls = same result                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Idempotency Key Flow
```
┌─────────────────────────────────────────────────────────────────┐
│                IDEMPOTENCY KEY FLOW                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  First Request:                                                 │
│  ─────────────                                                  │
│  Client ──────────────────────────────────────> Server         │
│         POST /payments                                         │
│         Idempotency-Key: abc123                                │
│         { amount: 100 }                                        │
│                                                                 │
│  1. Check if key "abc123" exists in store                      │
│  2. Not found → Process payment                                │
│  3. Store: key="abc123", status="processing"                   │
│  4. Complete payment                                           │
│  5. Store: key="abc123", status="complete", result={...}       │
│  6. Return result                                              │
│                                                                 │
│  Client <────────────────────────────────────── Server         │
│         201 Created                                            │
│         { id: "pay_123", status: "succeeded" }                 │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  Retry (same key):                                             │
│  ─────────────────                                              │
│  Client ──────────────────────────────────────> Server         │
│         POST /payments                                         │
│         Idempotency-Key: abc123  (same key)                    │
│         { amount: 100 }                                        │
│                                                                 │
│  1. Check if key "abc123" exists                               │
│  2. Found with status="complete"                               │
│  3. Return cached result (no new payment!)                     │
│                                                                 │
│  Client <────────────────────────────────────── Server         │
│         200 OK (from cache)                                    │
│         { id: "pay_123", status: "succeeded" }                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Idempotency key"** | "We require idempotency keys for all payment endpoints" |
| **"At-least-once delivery"** | "Message queues provide at-least-once, we handle dupes" |
| **"Exactly-once semantics"** | "Idempotency enables exactly-once semantics" |
| **"Retry safety"** | "Idempotent endpoints are retry-safe" |
| **"Request fingerprint"** | "We validate the request fingerprint matches" |
| **"Deduplication"** | "Idempotency key enables request deduplication" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Key TTL | **24-48 hours** | Allow retries within window |
| Key format | **UUID v4** | Globally unique |
| Max key length | **64-256 chars** | Reasonable limit |
| Processing timeout | **5-60 minutes** | In-flight request protection |

### The "Wow" Statement (Memorize This!)
> "We make all mutation endpoints idempotent using the Idempotency-Key header. Clients generate a UUID per logical operation. First request: we store the key with 'processing' status, execute the operation, then store the result. Retry with same key returns cached result - no duplicate payments or orders. We validate that retry requests match the original (same body hash), returning 422 if they differ. Keys expire after 24 hours. For in-flight requests (key exists but no result), we return 409 'request in progress'. This pattern is critical for handling network failures - clients can safely retry without duplicates, enabling exactly-once semantics from the client's perspective."

---

## 📚 Table of Contents

1. [Idempotency Key Pattern](#1-idempotency-key-pattern)
2. [Implementation](#2-implementation)
3. [Database Patterns](#3-database-patterns)
4. [Error Handling](#4-error-handling)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Idempotency Key Pattern

```typescript
// ════════════════════════════════════════════════════════════════
// IDEMPOTENCY KEY INTERFACE
// ════════════════════════════════════════════════════════════════

interface IdempotencyRecord {
  key: string;
  status: 'processing' | 'complete' | 'error';
  requestHash: string;       // Hash of original request body
  httpStatus?: number;       // Response status code
  responseBody?: string;     // Cached response
  createdAt: Date;
  completedAt?: Date;
  expiresAt: Date;
}

// How clients generate keys
function generateIdempotencyKey(): string {
  return crypto.randomUUID(); // e.g., "550e8400-e29b-41d4-a716-446655440000"
}

// Alternative: Deterministic key from operation details
function generateDeterministicKey(userId: string, action: string, data: any): string {
  const payload = JSON.stringify({ userId, action, data, timestamp: Date.now() });
  return crypto.createHash('sha256').update(payload).digest('hex');
}

// ════════════════════════════════════════════════════════════════
// NATURALLY IDEMPOTENT OPERATIONS
// ════════════════════════════════════════════════════════════════

// PUT - Replace entire resource (idempotent)
// Multiple calls with same data = same result
app.put('/users/:id', async (req, res) => {
  const user = await userService.replace(req.params.id, req.body);
  res.json(user);
});

// DELETE - Remove resource (idempotent)
// Multiple calls = resource still doesn't exist
app.delete('/users/:id', async (req, res) => {
  const deleted = await userService.delete(req.params.id);
  if (deleted) {
    res.status(204).send();
  } else {
    // Resource already doesn't exist - still success!
    res.status(204).send();
  }
});

// GET - Read resource (idempotent + safe)
app.get('/users/:id', async (req, res) => {
  const user = await userService.findById(req.params.id);
  res.json(user);
});

// ════════════════════════════════════════════════════════════════
// NON-IDEMPOTENT OPERATIONS (Need idempotency key)
// ════════════════════════════════════════════════════════════════

// POST - Create resource (NOT idempotent by default)
// Multiple calls = multiple resources created!
app.post('/users', async (req, res) => {
  const user = await userService.create(req.body);
  res.status(201).json(user);
});

// POST - Payment (definitely NOT idempotent)
// Multiple calls = multiple charges!
app.post('/payments', async (req, res) => {
  const payment = await paymentService.charge(req.body);
  res.status(201).json(payment);
});

// PATCH with relative changes (NOT idempotent)
// Multiple calls = multiple increments!
app.patch('/accounts/:id/balance', async (req, res) => {
  const { increment } = req.body;
  await accountService.incrementBalance(req.params.id, increment);
  res.json({ success: true });
});
```

---

## 2. Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// IDEMPOTENCY MIDDLEWARE
// ════════════════════════════════════════════════════════════════

import { Request, Response, NextFunction } from 'express';
import crypto from 'crypto';
import Redis from 'ioredis';

const redis = new Redis();
const IDEMPOTENCY_TTL = 24 * 60 * 60; // 24 hours

interface IdempotencyOptions {
  required?: boolean;           // Require idempotency key
  ttl?: number;                 // TTL in seconds
  headerName?: string;          // Custom header name
  validateBody?: boolean;       // Check body matches original
}

function idempotencyMiddleware(options: IdempotencyOptions = {}) {
  const {
    required = true,
    ttl = IDEMPOTENCY_TTL,
    headerName = 'Idempotency-Key',
    validateBody = true,
  } = options;

  return async (req: Request, res: Response, next: NextFunction) => {
    // Only apply to mutation methods
    if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) {
      return next();
    }

    const idempotencyKey = req.headers[headerName.toLowerCase()] as string;

    // Check if key is required
    if (!idempotencyKey) {
      if (required) {
        return res.status(400).json({
          error: 'missing_idempotency_key',
          message: `${headerName} header is required`,
        });
      }
      return next();
    }

    // Validate key format
    if (idempotencyKey.length > 256) {
      return res.status(400).json({
        error: 'invalid_idempotency_key',
        message: 'Idempotency key must be 256 characters or less',
      });
    }

    // Create request hash for body validation
    const requestHash = crypto
      .createHash('sha256')
      .update(JSON.stringify(req.body || {}))
      .digest('hex');

    const recordKey = `idempotency:${req.path}:${idempotencyKey}`;

    // Check for existing record
    const existingRecord = await redis.hgetall(recordKey);

    if (existingRecord && Object.keys(existingRecord).length > 0) {
      // Validate request body matches (prevent key reuse with different data)
      if (validateBody && existingRecord.requestHash !== requestHash) {
        return res.status(422).json({
          error: 'idempotency_key_reused',
          message: 'Idempotency key was used with different request parameters',
        });
      }

      // Check if still processing
      if (existingRecord.status === 'processing') {
        return res.status(409).json({
          error: 'request_in_progress',
          message: 'A request with this idempotency key is still being processed',
        });
      }

      // Return cached response
      if (existingRecord.status === 'complete') {
        res.setHeader('Idempotent-Replayed', 'true');
        return res
          .status(parseInt(existingRecord.httpStatus))
          .json(JSON.parse(existingRecord.responseBody));
      }

      // Previous request errored - allow retry
      if (existingRecord.status === 'error') {
        await redis.del(recordKey);
      }
    }

    // Store processing status
    await redis.hset(recordKey, {
      status: 'processing',
      requestHash,
      createdAt: new Date().toISOString(),
    });
    await redis.expire(recordKey, ttl);

    // Capture response
    const originalJson = res.json.bind(res);
    res.json = (body: any) => {
      // Store completed response
      redis.hset(recordKey, {
        status: res.statusCode >= 400 ? 'error' : 'complete',
        httpStatus: res.statusCode.toString(),
        responseBody: JSON.stringify(body),
        completedAt: new Date().toISOString(),
      });

      return originalJson(body);
    };

    next();
  };
}

// ════════════════════════════════════════════════════════════════
// USAGE
// ════════════════════════════════════════════════════════════════

// Apply to specific endpoints
app.post('/payments', idempotencyMiddleware({ required: true }), async (req, res) => {
  const payment = await paymentService.charge(req.body);
  res.status(201).json(payment);
});

app.post('/orders', idempotencyMiddleware({ required: true }), async (req, res) => {
  const order = await orderService.create(req.body);
  res.status(201).json(order);
});

// Or apply globally to all mutations
app.use('/api', idempotencyMiddleware({ required: false }));

// ════════════════════════════════════════════════════════════════
// CLIENT-SIDE USAGE
// ════════════════════════════════════════════════════════════════

async function createPaymentWithRetry(data: PaymentInput): Promise<Payment> {
  // Generate key once per logical operation
  const idempotencyKey = crypto.randomUUID();
  
  const maxRetries = 3;
  let lastError: Error | null = null;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await fetch('/api/payments', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Idempotency-Key': idempotencyKey,  // Same key for all retries
        },
        body: JSON.stringify(data),
      });

      if (!response.ok) {
        const error = await response.json();
        
        // Don't retry client errors (except rate limit)
        if (response.status >= 400 && response.status < 500 && response.status !== 429) {
          throw new Error(error.message);
        }
        
        throw new Error(`Request failed: ${response.status}`);
      }

      return response.json();
    } catch (error) {
      lastError = error;
      
      // Exponential backoff
      await sleep(Math.pow(2, attempt) * 1000);
    }
  }

  throw lastError;
}
```

---

## 3. Database Patterns

```typescript
// ════════════════════════════════════════════════════════════════
// DATABASE-BASED IDEMPOTENCY
// ════════════════════════════════════════════════════════════════

// Schema (PostgreSQL)
/*
CREATE TABLE idempotency_keys (
  key VARCHAR(256) NOT NULL,
  path VARCHAR(512) NOT NULL,
  request_hash VARCHAR(64) NOT NULL,
  status VARCHAR(20) NOT NULL DEFAULT 'processing',
  http_status INTEGER,
  response_body JSONB,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  completed_at TIMESTAMP,
  expires_at TIMESTAMP NOT NULL,
  
  PRIMARY KEY (key, path)
);

CREATE INDEX idx_idempotency_expires ON idempotency_keys (expires_at);

-- Cleanup job
DELETE FROM idempotency_keys WHERE expires_at < NOW();
*/

// Prisma schema
/*
model IdempotencyKey {
  key          String
  path         String
  requestHash  String
  status       String   @default("processing")
  httpStatus   Int?
  responseBody Json?
  createdAt    DateTime @default(now())
  completedAt  DateTime?
  expiresAt    DateTime

  @@id([key, path])
  @@index([expiresAt])
}
*/

class IdempotencyService {
  async checkAndLock(
    key: string,
    path: string,
    requestHash: string
  ): Promise<{ 
    isNew: boolean; 
    cachedResponse?: { status: number; body: any };
    isProcessing?: boolean;
  }> {
    const ttlHours = 24;
    const expiresAt = new Date(Date.now() + ttlHours * 60 * 60 * 1000);

    try {
      // Try to insert (will fail if exists due to unique constraint)
      await prisma.idempotencyKey.create({
        data: {
          key,
          path,
          requestHash,
          status: 'processing',
          expiresAt,
        },
      });
      
      return { isNew: true };
    } catch (error) {
      // Key exists - check it
      if (error.code === 'P2002') { // Unique constraint violation
        const existing = await prisma.idempotencyKey.findUnique({
          where: { key_path: { key, path } },
        });

        if (!existing) {
          throw error; // Race condition, retry
        }

        // Validate request hash
        if (existing.requestHash !== requestHash) {
          throw new IdempotencyKeyReusedError();
        }

        // Check status
        if (existing.status === 'processing') {
          return { isNew: false, isProcessing: true };
        }

        if (existing.status === 'complete') {
          return {
            isNew: false,
            cachedResponse: {
              status: existing.httpStatus!,
              body: existing.responseBody,
            },
          };
        }

        // Error status - allow retry by updating to processing
        await prisma.idempotencyKey.update({
          where: { key_path: { key, path } },
          data: { status: 'processing', completedAt: null },
        });
        
        return { isNew: true };
      }
      
      throw error;
    }
  }

  async complete(
    key: string,
    path: string,
    httpStatus: number,
    responseBody: any
  ): Promise<void> {
    await prisma.idempotencyKey.update({
      where: { key_path: { key, path } },
      data: {
        status: httpStatus >= 400 ? 'error' : 'complete',
        httpStatus,
        responseBody,
        completedAt: new Date(),
      },
    });
  }
}

// ════════════════════════════════════════════════════════════════
// USING UNIQUE CONSTRAINTS FOR NATURAL IDEMPOTENCY
// ════════════════════════════════════════════════════════════════

// Instead of idempotency key, use natural unique constraints

// Example: User can only have one subscription per plan
/*
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL,
  plan_id UUID NOT NULL,
  status VARCHAR(20) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  
  UNIQUE (user_id, plan_id)  -- Natural idempotency
);
*/

async function createSubscription(userId: string, planId: string) {
  try {
    return await prisma.subscription.create({
      data: { userId, planId, status: 'active' },
    });
  } catch (error) {
    if (error.code === 'P2002') {
      // Already exists - return existing (idempotent!)
      return prisma.subscription.findUnique({
        where: { userId_planId: { userId, planId } },
      });
    }
    throw error;
  }
}

// Example: Order with client-provided reference
/*
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  client_reference VARCHAR(256) UNIQUE,  -- Client idempotency key
  user_id UUID NOT NULL,
  total DECIMAL(10,2) NOT NULL,
  ...
);
*/

async function createOrder(clientReference: string, orderData: any) {
  try {
    return await prisma.order.create({
      data: { clientReference, ...orderData },
    });
  } catch (error) {
    if (error.code === 'P2002' && error.meta?.target?.includes('client_reference')) {
      return prisma.order.findUnique({
        where: { clientReference },
      });
    }
    throw error;
  }
}
```

---

## 4. Error Handling

```typescript
// ════════════════════════════════════════════════════════════════
// IDEMPOTENCY ERROR RESPONSES
// ════════════════════════════════════════════════════════════════

// Error: Missing idempotency key
// HTTP 400 Bad Request
{
  "error": "missing_idempotency_key",
  "message": "Idempotency-Key header is required for this endpoint",
  "code": "IDEM_001"
}

// Error: Key reused with different parameters
// HTTP 422 Unprocessable Entity
{
  "error": "idempotency_key_reused",
  "message": "This idempotency key was already used with different request parameters. Please use a new key.",
  "code": "IDEM_002"
}

// Error: Request still in progress
// HTTP 409 Conflict
{
  "error": "request_in_progress",
  "message": "A request with this idempotency key is still being processed. Please wait and retry.",
  "code": "IDEM_003",
  "retry_after": 5
}

// Success: Replayed response (header indicates cached)
// HTTP 200 OK (or original status)
// Headers: Idempotent-Replayed: true
{
  "id": "pay_123",
  "status": "succeeded",
  "amount": 1000
}

// ════════════════════════════════════════════════════════════════
// ERROR HANDLING IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

class IdempotencyKeyReusedError extends Error {
  constructor() {
    super('Idempotency key was used with different request parameters');
    this.name = 'IdempotencyKeyReusedError';
  }
}

class RequestInProgressError extends Error {
  constructor() {
    super('A request with this idempotency key is still being processed');
    this.name = 'RequestInProgressError';
  }
}

// Error handler middleware
function idempotencyErrorHandler(err: Error, req: Request, res: Response, next: NextFunction) {
  if (err instanceof IdempotencyKeyReusedError) {
    return res.status(422).json({
      error: 'idempotency_key_reused',
      message: err.message,
      code: 'IDEM_002',
    });
  }

  if (err instanceof RequestInProgressError) {
    return res.status(409).json({
      error: 'request_in_progress',
      message: err.message,
      code: 'IDEM_003',
      retry_after: 5,
    });
  }

  next(err);
}

// ════════════════════════════════════════════════════════════════
// HANDLING PARTIAL FAILURES
// ════════════════════════════════════════════════════════════════

async function processPaymentWithIdempotency(
  idempotencyKey: string,
  paymentData: PaymentInput
): Promise<Payment> {
  const lockKey = `lock:payment:${idempotencyKey}`;
  
  // Acquire distributed lock
  const lock = await redlock.acquire([lockKey], 30000);
  
  try {
    // Check for existing result
    const existing = await idempotencyService.get(idempotencyKey);
    if (existing?.status === 'complete') {
      return existing.result;
    }

    // Process payment
    const payment = await paymentGateway.charge(paymentData);

    // Store result
    await idempotencyService.complete(idempotencyKey, 201, payment);

    return payment;
  } catch (error) {
    // Determine if error is retryable
    if (isRetryableError(error)) {
      // Don't store - allow retry
      throw error;
    }

    // Store error result (prevents retry)
    await idempotencyService.complete(idempotencyKey, 500, {
      error: error.message,
    });

    throw error;
  } finally {
    await lock.release();
  }
}
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# IDEMPOTENCY BEST PRACTICES
# ════════════════════════════════════════════════════════════════

key_management:
  format:
    - UUID v4 (recommended)
    - Or: deterministic hash of operation
  
  generation:
    - Client generates key
    - One key per logical operation
    - New key for each new operation
  
  validation:
    - Max length (256 chars)
    - No special characters
    - Unique per endpoint

request_validation:
  - Hash request body
  - Compare hash on retry
  - Reject if mismatch (422)
  - Include only relevant fields in hash

storage:
  options:
    - Redis (fast, distributed)
    - Database (durable, queryable)
    - Hybrid (Redis + DB backup)
  
  ttl:
    - 24-48 hours typical
    - Longer for critical operations
    - Consider cleanup job
  
  what_to_store:
    - Key
    - Request hash
    - Status
    - HTTP status code
    - Response body
    - Timestamps

endpoints_to_protect:
  required:
    - Payments / charges
    - Order creation
    - Account creation
    - Money transfers
  
  recommended:
    - Any POST creating resources
    - Operations with side effects
    - Email / notification triggers

response:
  headers:
    - Idempotent-Replayed: true (when cached)
  
  body:
    - Same as original response
    - Same status code

monitoring:
  - Track replay rate
  - Alert on high reuse errors
  - Monitor key TTL misses
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# IDEMPOTENCY PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Reusing same key for different operations
# Bad - Client
const key = "my-app-key";  // Same key always
await createPayment(key, payment1);
await createPayment(key, payment2);  // Returns payment1!

# Good
const key1 = uuid();
const key2 = uuid();
await createPayment(key1, payment1);
await createPayment(key2, payment2);

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Not validating request body
# Bad
# Key "abc" used with { amount: 100 }
# Retry with { amount: 200 } - returns cached { amount: 100 }!

# Good
# Hash request body
# Reject if hash mismatch on retry

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Race condition with concurrent requests
# Bad
# Two requests with same key arrive simultaneously
# Both check "not exists" → both process → duplicate!

# Good
# Use database unique constraint
# Or distributed lock
# Or optimistic locking

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Not handling in-progress requests
# Bad
# Request 1 processing (slow payment gateway)
# Request 2 (retry) arrives → starts processing too!

# Good
# Store "processing" status
# Return 409 for in-progress requests
# Include retry_after hint

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Too short TTL
# Bad
ttl: 5 * 60  # 5 minutes
# User retries after 10 minutes → duplicate!

# Good
ttl: 24 * 60 * 60  # 24 hours
# Long enough to cover typical retry scenarios

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Idempotency for entire flow, not individual steps
# Bad
# Idempotent endpoint calls non-idempotent external service
# Retry → external service called twice!

# Good
# Each external call needs its own idempotency
# Or: wrap entire flow in transaction with compensation
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is idempotency?"**
> "An operation is idempotent if calling it multiple times produces the same result as calling it once. GET, PUT, DELETE are naturally idempotent. POST is not - each call may create new resource. Idempotency enables safe retries without duplicate side effects."

**Q: "Which HTTP methods are idempotent?"**
> "GET, HEAD, OPTIONS, PUT, DELETE are idempotent. GET just reads. PUT replaces entire resource (same input = same result). DELETE removes (already gone = still gone). POST is not idempotent - creates new resource each time. PATCH depends on implementation."

**Q: "What is an idempotency key?"**
> "Client-provided unique identifier for an operation. Server stores key with result. On retry with same key, returns cached result instead of re-executing. Enables safe retries for non-idempotent operations like payments. Usually UUID, one per logical operation."

### Intermediate Questions

**Q: "How do you implement idempotency for payments?"**
> "Require Idempotency-Key header. Hash request body. On first request: store key with 'processing' status, charge payment, store result. On retry: check key exists, validate body hash matches, return cached result. Key expires after 24 hours. Prevents double charges."

**Q: "What happens if two requests with same key arrive simultaneously?"**
> "Race condition risk. Solutions: 1) Database unique constraint - second insert fails. 2) Distributed lock (Redis SETNX). 3) Optimistic locking. First request processes, second either waits or gets 409 'in progress'. Never both execute."

**Q: "How do you handle request body changes on retry?"**
> "Hash the request body and store with idempotency key. On retry, compare hashes. If different: return 422 'key reused with different parameters'. Client must use new key for different operation. Prevents accidental key reuse."

### Advanced Questions

**Q: "How do you make distributed systems idempotent?"**
> "Multiple levels: API layer (idempotency keys), database (unique constraints, upserts), message queues (message deduplication). Each service needs own idempotency. Correlation IDs track operation across services. Consider saga pattern with compensating actions."

**Q: "What's the difference between at-least-once and exactly-once?"**
> "At-least-once: Operation happens 1+ times (may have duplicates). Exactly-once: Operation happens exactly once. Networks provide at-most-once or at-least-once, not exactly-once. Idempotency + at-least-once = exactly-once from client perspective."

**Q: "How do you handle partial failures with idempotency?"**
> "If operation has multiple steps and fails midway: 1) Don't cache error result if retryable. 2) Use database transaction to rollback partial state. 3) For distributed: saga pattern with compensation. 4) Idempotent cleanup job to fix inconsistencies."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  IDEMPOTENCY CHECKLIST                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  HTTP METHODS:                                                  │
│  □ GET, PUT, DELETE are naturally idempotent                   │
│  □ POST needs idempotency key                                  │
│  □ PATCH may need idempotency key                              │
│                                                                 │
│  IMPLEMENTATION:                                                │
│  □ Require Idempotency-Key header                              │
│  □ Hash and validate request body                              │
│  □ Store: key, hash, status, response                          │
│  □ Return cached response on retry                             │
│  □ Handle concurrent requests (locks)                          │
│                                                                 │
│  CLIENT:                                                        │
│  □ Generate UUID per operation                                 │
│  □ Same key for retries                                        │
│  □ New key for new operation                                   │
│                                                                 │
│  STORAGE:                                                       │
│  □ TTL 24-48 hours                                             │
│  □ Redis or database                                           │
│  □ Cleanup expired keys                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

IDEMPOTENCY KEY FLOW:
┌────────────────────────────────────────────────────────────────┐
│ 1. Client generates key (UUID)                                 │
│ 2. Request with Idempotency-Key header                        │
│ 3. Server checks if key exists                                │
│    - New: process, store result                               │
│    - Exists: return cached result                             │
│ 4. Client can safely retry with same key                      │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

