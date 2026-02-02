# Cloud Design Patterns - Complete Guide

> **MUST REMEMBER**: Key patterns: Circuit Breaker (prevent cascade failures), Bulkhead (isolate resources), Retry (handle transient failures with exponential backoff), Timeout (bound wait time). Also: Throttling (rate limit), Cache-Aside (lazy caching), CQRS (separate reads/writes), Saga (distributed transactions). These patterns handle the reality of distributed systems - failures happen, networks are unreliable.

---

## How to Explain Like a Senior Developer

"Cloud applications are distributed systems, and distributed systems fail in interesting ways. These patterns help you build resilient applications. Circuit Breaker prevents one failing service from bringing down everything - after N failures, stop calling it for a while. Bulkhead isolates resources so one component's failure doesn't exhaust shared resources. Retry handles transient failures but with exponential backoff to avoid thundering herd. Timeout ensures you don't wait forever for a slow service. The key insight: assume failures will happen, design for graceful degradation, and always have fallbacks."

---

## Core Implementation

### Circuit Breaker Pattern

```typescript
// patterns/circuit-breaker.ts

enum CircuitState {
  CLOSED = 'CLOSED',       // Normal operation
  OPEN = 'OPEN',           // Failing, reject requests
  HALF_OPEN = 'HALF_OPEN', // Testing recovery
}

interface CircuitBreakerOptions {
  failureThreshold: number;    // Failures before opening
  successThreshold: number;    // Successes to close from half-open
  timeout: number;             // Time in open state before half-open
  resetTimeout?: number;       // Time to reset failure count
}

class CircuitBreaker {
  private state: CircuitState = CircuitState.CLOSED;
  private failures = 0;
  private successes = 0;
  private lastFailureTime?: number;
  private nextAttempt?: number;
  
  constructor(private options: CircuitBreakerOptions) {}
  
  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === CircuitState.OPEN) {
      if (Date.now() >= (this.nextAttempt || 0)) {
        // Transition to half-open
        this.state = CircuitState.HALF_OPEN;
        this.successes = 0;
      } else {
        throw new CircuitBreakerOpenError('Circuit is open');
      }
    }
    
    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  private onSuccess(): void {
    this.failures = 0;
    
    if (this.state === CircuitState.HALF_OPEN) {
      this.successes++;
      
      if (this.successes >= this.options.successThreshold) {
        this.state = CircuitState.CLOSED;
        console.log('Circuit closed - service recovered');
      }
    }
  }
  
  private onFailure(): void {
    this.failures++;
    this.lastFailureTime = Date.now();
    
    if (this.state === CircuitState.HALF_OPEN) {
      // Back to open on any failure
      this.state = CircuitState.OPEN;
      this.nextAttempt = Date.now() + this.options.timeout;
      console.log('Circuit reopened - still failing');
    } else if (this.failures >= this.options.failureThreshold) {
      this.state = CircuitState.OPEN;
      this.nextAttempt = Date.now() + this.options.timeout;
      console.log('Circuit opened - too many failures');
    }
  }
  
  getState(): CircuitState {
    return this.state;
  }
  
  getStats(): object {
    return {
      state: this.state,
      failures: this.failures,
      successes: this.successes,
      lastFailure: this.lastFailureTime,
    };
  }
}

class CircuitBreakerOpenError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'CircuitBreakerOpenError';
  }
}

// Usage
const breaker = new CircuitBreaker({
  failureThreshold: 5,
  successThreshold: 2,
  timeout: 30000, // 30 seconds
});

async function callExternalService(): Promise<any> {
  return breaker.execute(async () => {
    const response = await fetch('https://api.example.com/data');
    if (!response.ok) throw new Error('Service error');
    return response.json();
  });
}
```

### Bulkhead Pattern

```typescript
// patterns/bulkhead.ts

interface BulkheadOptions {
  maxConcurrent: number;
  maxQueued: number;
  timeout?: number;
}

class Bulkhead {
  private running = 0;
  private queue: Array<{
    resolve: (value: any) => void;
    reject: (error: Error) => void;
    operation: () => Promise<any>;
    timeout?: NodeJS.Timeout;
  }> = [];
  
  constructor(private options: BulkheadOptions) {}
  
  async execute<T>(operation: () => Promise<T>): Promise<T> {
    // Check if we can run immediately
    if (this.running < this.options.maxConcurrent) {
      return this.runOperation(operation);
    }
    
    // Check if queue is full
    if (this.queue.length >= this.options.maxQueued) {
      throw new BulkheadFullError('Bulkhead queue is full');
    }
    
    // Add to queue
    return new Promise((resolve, reject) => {
      const queueItem: any = { resolve, reject, operation };
      
      if (this.options.timeout) {
        queueItem.timeout = setTimeout(() => {
          const index = this.queue.indexOf(queueItem);
          if (index > -1) {
            this.queue.splice(index, 1);
            reject(new Error('Bulkhead queue timeout'));
          }
        }, this.options.timeout);
      }
      
      this.queue.push(queueItem);
    });
  }
  
  private async runOperation<T>(operation: () => Promise<T>): Promise<T> {
    this.running++;
    
    try {
      return await operation();
    } finally {
      this.running--;
      this.processQueue();
    }
  }
  
  private processQueue(): void {
    if (this.queue.length > 0 && this.running < this.options.maxConcurrent) {
      const item = this.queue.shift()!;
      
      if (item.timeout) {
        clearTimeout(item.timeout);
      }
      
      this.runOperation(item.operation)
        .then(item.resolve)
        .catch(item.reject);
    }
  }
  
  getStats(): object {
    return {
      running: this.running,
      queued: this.queue.length,
      maxConcurrent: this.options.maxConcurrent,
      maxQueued: this.options.maxQueued,
    };
  }
}

class BulkheadFullError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'BulkheadFullError';
  }
}

// Usage: Isolate different services
const paymentBulkhead = new Bulkhead({
  maxConcurrent: 10,
  maxQueued: 20,
  timeout: 5000,
});

const notificationBulkhead = new Bulkhead({
  maxConcurrent: 5,
  maxQueued: 100,
  timeout: 10000,
});

async function processPayment(data: any): Promise<any> {
  return paymentBulkhead.execute(() => callPaymentService(data));
}

async function sendNotification(data: any): Promise<any> {
  return notificationBulkhead.execute(() => callNotificationService(data));
}

async function callPaymentService(data: any): Promise<any> { return data; }
async function callNotificationService(data: any): Promise<any> { return data; }
```

### Retry Pattern with Exponential Backoff

```typescript
// patterns/retry.ts

interface RetryOptions {
  maxAttempts: number;
  initialDelay: number;      // ms
  maxDelay: number;          // ms
  backoffMultiplier: number;
  retryableErrors?: (error: Error) => boolean;
  onRetry?: (error: Error, attempt: number, delay: number) => void;
}

async function retry<T>(
  operation: () => Promise<T>,
  options: RetryOptions
): Promise<T> {
  let lastError: Error;
  let delay = options.initialDelay;
  
  for (let attempt = 1; attempt <= options.maxAttempts; attempt++) {
    try {
      return await operation();
    } catch (error: any) {
      lastError = error;
      
      // Check if error is retryable
      const isRetryable = options.retryableErrors
        ? options.retryableErrors(error)
        : isTransientError(error);
      
      if (!isRetryable || attempt === options.maxAttempts) {
        throw error;
      }
      
      // Calculate delay with jitter
      const jitter = Math.random() * 0.3 * delay; // 0-30% jitter
      const actualDelay = Math.min(delay + jitter, options.maxDelay);
      
      options.onRetry?.(error, attempt, actualDelay);
      
      await sleep(actualDelay);
      
      // Exponential backoff
      delay = Math.min(delay * options.backoffMultiplier, options.maxDelay);
    }
  }
  
  throw lastError!;
}

function isTransientError(error: any): boolean {
  // Network errors
  if (error.code === 'ECONNRESET' || error.code === 'ETIMEDOUT') {
    return true;
  }
  
  // HTTP status codes
  if (error.status) {
    // 408 Request Timeout, 429 Too Many Requests, 5xx Server Errors
    return error.status === 408 || 
           error.status === 429 || 
           (error.status >= 500 && error.status < 600);
  }
  
  return false;
}

function sleep(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Usage
async function fetchWithRetry(url: string): Promise<any> {
  return retry(
    async () => {
      const response = await fetch(url);
      if (!response.ok) {
        const error: any = new Error(`HTTP ${response.status}`);
        error.status = response.status;
        throw error;
      }
      return response.json();
    },
    {
      maxAttempts: 3,
      initialDelay: 1000,
      maxDelay: 10000,
      backoffMultiplier: 2,
      onRetry: (error, attempt, delay) => {
        console.log(`Retry ${attempt} after ${delay}ms: ${error.message}`);
      },
    }
  );
}

// Decorator version
function withRetry(options: RetryOptions) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const original = descriptor.value;
    
    descriptor.value = async function (...args: any[]) {
      return retry(() => original.apply(this, args), options);
    };
    
    return descriptor;
  };
}

// Usage with decorator
class ApiService {
  @withRetry({
    maxAttempts: 3,
    initialDelay: 1000,
    maxDelay: 10000,
    backoffMultiplier: 2,
  })
  async fetchData(id: string): Promise<any> {
    const response = await fetch(`/api/data/${id}`);
    return response.json();
  }
}
```

### Timeout Pattern

```typescript
// patterns/timeout.ts

class TimeoutError extends Error {
  constructor(message: string = 'Operation timed out') {
    super(message);
    this.name = 'TimeoutError';
  }
}

async function withTimeout<T>(
  operation: () => Promise<T>,
  timeoutMs: number,
  options?: {
    onTimeout?: () => void;
    errorMessage?: string;
  }
): Promise<T> {
  let timeoutId: NodeJS.Timeout;
  
  const timeoutPromise = new Promise<never>((_, reject) => {
    timeoutId = setTimeout(() => {
      options?.onTimeout?.();
      reject(new TimeoutError(options?.errorMessage));
    }, timeoutMs);
  });
  
  try {
    return await Promise.race([operation(), timeoutPromise]);
  } finally {
    clearTimeout(timeoutId!);
  }
}

// Cascading timeouts for service calls
interface CascadingTimeoutOptions {
  totalTimeout: number;
  serviceTimeouts: Record<string, number>;
}

class CascadingTimeout {
  private startTime: number;
  private totalTimeout: number;
  private serviceTimeouts: Record<string, number>;
  
  constructor(options: CascadingTimeoutOptions) {
    this.startTime = Date.now();
    this.totalTimeout = options.totalTimeout;
    this.serviceTimeouts = options.serviceTimeouts;
  }
  
  getRemainingTime(): number {
    return Math.max(0, this.totalTimeout - (Date.now() - this.startTime));
  }
  
  getServiceTimeout(service: string): number {
    const remaining = this.getRemainingTime();
    const serviceTimeout = this.serviceTimeouts[service] || remaining;
    return Math.min(remaining, serviceTimeout);
  }
  
  async execute<T>(service: string, operation: () => Promise<T>): Promise<T> {
    const timeout = this.getServiceTimeout(service);
    
    if (timeout <= 0) {
      throw new TimeoutError(`No time remaining for ${service}`);
    }
    
    return withTimeout(operation, timeout, {
      errorMessage: `${service} timed out after ${timeout}ms`,
    });
  }
}

// Usage
async function processOrder(orderId: string): Promise<any> {
  const timeout = new CascadingTimeout({
    totalTimeout: 5000,
    serviceTimeouts: {
      inventory: 1000,
      payment: 2000,
      notification: 500,
    },
  });
  
  // Check inventory
  const inventory = await timeout.execute('inventory', () =>
    checkInventory(orderId)
  );
  
  // Process payment
  const payment = await timeout.execute('payment', () =>
    processPayment(orderId)
  );
  
  // Send notification (can fail gracefully)
  try {
    await timeout.execute('notification', () =>
      sendNotification(orderId)
    );
  } catch (error) {
    console.warn('Notification failed, continuing...');
  }
  
  return { inventory, payment };
}

async function checkInventory(orderId: string): Promise<any> { return {}; }
async function processPayment(orderId: string): Promise<any> { return {}; }
async function sendNotification(orderId: string): Promise<any> { return {}; }
```

### Throttling Pattern

```typescript
// patterns/throttling.ts

interface ThrottleOptions {
  requestsPerSecond: number;
  burstSize?: number;
}

class TokenBucketThrottle {
  private tokens: number;
  private lastRefill: number;
  private maxTokens: number;
  private refillRate: number; // tokens per ms
  
  constructor(options: ThrottleOptions) {
    this.maxTokens = options.burstSize || options.requestsPerSecond;
    this.tokens = this.maxTokens;
    this.refillRate = options.requestsPerSecond / 1000;
    this.lastRefill = Date.now();
  }
  
  private refill(): void {
    const now = Date.now();
    const elapsed = now - this.lastRefill;
    const newTokens = elapsed * this.refillRate;
    
    this.tokens = Math.min(this.maxTokens, this.tokens + newTokens);
    this.lastRefill = now;
  }
  
  tryAcquire(tokens: number = 1): boolean {
    this.refill();
    
    if (this.tokens >= tokens) {
      this.tokens -= tokens;
      return true;
    }
    
    return false;
  }
  
  async acquire(tokens: number = 1): Promise<void> {
    while (!this.tryAcquire(tokens)) {
      // Wait for next token
      const waitTime = (tokens - this.tokens) / this.refillRate;
      await new Promise(resolve => setTimeout(resolve, Math.ceil(waitTime)));
    }
  }
  
  getAvailableTokens(): number {
    this.refill();
    return this.tokens;
  }
}

// Middleware for Express
import { Request, Response, NextFunction } from 'express';

const clientThrottles = new Map<string, TokenBucketThrottle>();

function throttleMiddleware(options: ThrottleOptions) {
  return (req: Request, res: Response, next: NextFunction) => {
    const clientId = req.ip || 'unknown';
    
    let throttle = clientThrottles.get(clientId);
    if (!throttle) {
      throttle = new TokenBucketThrottle(options);
      clientThrottles.set(clientId, throttle);
    }
    
    if (throttle.tryAcquire()) {
      // Add rate limit headers
      res.setHeader('X-RateLimit-Remaining', Math.floor(throttle.getAvailableTokens()));
      res.setHeader('X-RateLimit-Limit', options.requestsPerSecond);
      next();
    } else {
      res.status(429).json({
        error: 'Too Many Requests',
        retryAfter: Math.ceil(1000 / options.requestsPerSecond),
      });
    }
  };
}
```

### Cache-Aside Pattern

```typescript
// patterns/cache-aside.ts

interface CacheOptions {
  ttl: number; // seconds
  staleWhileRevalidate?: number;
}

interface CacheEntry<T> {
  data: T;
  expiresAt: number;
  staleAt?: number;
}

class CacheAside<T> {
  private cache: Map<string, CacheEntry<T>> = new Map();
  private pending: Map<string, Promise<T>> = new Map();
  
  constructor(
    private fetcher: (key: string) => Promise<T>,
    private options: CacheOptions
  ) {}
  
  async get(key: string): Promise<T> {
    const cached = this.cache.get(key);
    const now = Date.now();
    
    // Cache hit - fresh data
    if (cached && now < (cached.staleAt || cached.expiresAt)) {
      return cached.data;
    }
    
    // Cache hit - stale data (revalidate in background)
    if (cached && this.options.staleWhileRevalidate && now < cached.expiresAt) {
      this.revalidate(key);
      return cached.data;
    }
    
    // Cache miss - fetch fresh data
    return this.fetch(key);
  }
  
  private async fetch(key: string): Promise<T> {
    // Dedupe concurrent requests
    const pending = this.pending.get(key);
    if (pending) return pending;
    
    const promise = this.fetcher(key);
    this.pending.set(key, promise);
    
    try {
      const data = await promise;
      this.set(key, data);
      return data;
    } finally {
      this.pending.delete(key);
    }
  }
  
  private set(key: string, data: T): void {
    const now = Date.now();
    const entry: CacheEntry<T> = {
      data,
      expiresAt: now + this.options.ttl * 1000,
    };
    
    if (this.options.staleWhileRevalidate) {
      entry.staleAt = now + (this.options.ttl - this.options.staleWhileRevalidate) * 1000;
    }
    
    this.cache.set(key, entry);
  }
  
  private async revalidate(key: string): Promise<void> {
    try {
      const data = await this.fetcher(key);
      this.set(key, data);
    } catch (error) {
      console.error(`Background revalidation failed for ${key}:`, error);
    }
  }
  
  invalidate(key: string): void {
    this.cache.delete(key);
  }
  
  invalidateAll(): void {
    this.cache.clear();
  }
}

// Usage
const userCache = new CacheAside<User>(
  async (userId) => {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  },
  {
    ttl: 300,                  // 5 minutes
    staleWhileRevalidate: 60,  // Last minute is stale
  }
);

interface User {
  id: string;
  name: string;
}

async function getUser(userId: string): Promise<User> {
  return userCache.get(userId);
}
```

---

## Real-World Scenarios

### Scenario 1: Resilient Service Client

```typescript
// resilient-client.ts

interface ResilientClientOptions {
  baseUrl: string;
  timeout: number;
  circuitBreaker: {
    failureThreshold: number;
    timeout: number;
  };
  retry: {
    maxAttempts: number;
    initialDelay: number;
  };
  bulkhead: {
    maxConcurrent: number;
    maxQueued: number;
  };
}

class ResilientServiceClient {
  private circuitBreaker: CircuitBreaker;
  private bulkhead: Bulkhead;
  private options: ResilientClientOptions;
  
  constructor(options: ResilientClientOptions) {
    this.options = options;
    
    this.circuitBreaker = new CircuitBreaker({
      failureThreshold: options.circuitBreaker.failureThreshold,
      successThreshold: 2,
      timeout: options.circuitBreaker.timeout,
    });
    
    this.bulkhead = new Bulkhead({
      maxConcurrent: options.bulkhead.maxConcurrent,
      maxQueued: options.bulkhead.maxQueued,
      timeout: options.timeout,
    });
  }
  
  async request<T>(
    path: string,
    options?: RequestInit
  ): Promise<T> {
    // Bulkhead: Limit concurrency
    return this.bulkhead.execute(async () => {
      // Circuit Breaker: Prevent cascade failures
      return this.circuitBreaker.execute(async () => {
        // Retry: Handle transient failures
        return retry(
          async () => {
            // Timeout: Bound wait time
            return withTimeout(
              async () => {
                const response = await fetch(
                  `${this.options.baseUrl}${path}`,
                  options
                );
                
                if (!response.ok) {
                  const error: any = new Error(`HTTP ${response.status}`);
                  error.status = response.status;
                  throw error;
                }
                
                return response.json();
              },
              this.options.timeout
            );
          },
          {
            maxAttempts: this.options.retry.maxAttempts,
            initialDelay: this.options.retry.initialDelay,
            maxDelay: 10000,
            backoffMultiplier: 2,
          }
        );
      });
    });
  }
  
  getHealth(): object {
    return {
      circuitBreaker: this.circuitBreaker.getStats(),
      bulkhead: this.bulkhead.getStats(),
    };
  }
}

// Usage
const paymentService = new ResilientServiceClient({
  baseUrl: 'https://payment-api.internal',
  timeout: 5000,
  circuitBreaker: {
    failureThreshold: 5,
    timeout: 30000,
  },
  retry: {
    maxAttempts: 3,
    initialDelay: 1000,
  },
  bulkhead: {
    maxConcurrent: 10,
    maxQueued: 50,
  },
});
```

### Scenario 2: Saga Pattern for Distributed Transactions

```typescript
// patterns/saga.ts

interface SagaStep<T> {
  name: string;
  execute: (context: T) => Promise<void>;
  compensate: (context: T) => Promise<void>;
}

class Saga<T extends object> {
  private steps: SagaStep<T>[] = [];
  
  addStep(step: SagaStep<T>): this {
    this.steps.push(step);
    return this;
  }
  
  async execute(context: T): Promise<void> {
    const completedSteps: SagaStep<T>[] = [];
    
    try {
      for (const step of this.steps) {
        console.log(`Executing step: ${step.name}`);
        await step.execute(context);
        completedSteps.push(step);
      }
    } catch (error) {
      console.error('Saga failed, compensating...', error);
      
      // Compensate in reverse order
      for (const step of completedSteps.reverse()) {
        try {
          console.log(`Compensating step: ${step.name}`);
          await step.compensate(context);
        } catch (compensateError) {
          console.error(`Compensation failed for ${step.name}:`, compensateError);
          // Log for manual intervention
        }
      }
      
      throw error;
    }
  }
}

// Order processing saga
interface OrderContext {
  orderId: string;
  userId: string;
  amount: number;
  inventoryReserved?: boolean;
  paymentProcessed?: boolean;
  shipmentCreated?: boolean;
}

const orderSaga = new Saga<OrderContext>()
  .addStep({
    name: 'Reserve Inventory',
    execute: async (ctx) => {
      await inventoryService.reserve(ctx.orderId);
      ctx.inventoryReserved = true;
    },
    compensate: async (ctx) => {
      if (ctx.inventoryReserved) {
        await inventoryService.release(ctx.orderId);
      }
    },
  })
  .addStep({
    name: 'Process Payment',
    execute: async (ctx) => {
      await paymentService.charge(ctx.userId, ctx.amount);
      ctx.paymentProcessed = true;
    },
    compensate: async (ctx) => {
      if (ctx.paymentProcessed) {
        await paymentService.refund(ctx.userId, ctx.amount);
      }
    },
  })
  .addStep({
    name: 'Create Shipment',
    execute: async (ctx) => {
      await shippingService.createShipment(ctx.orderId);
      ctx.shipmentCreated = true;
    },
    compensate: async (ctx) => {
      if (ctx.shipmentCreated) {
        await shippingService.cancelShipment(ctx.orderId);
      }
    },
  });

// Placeholder services
const inventoryService = {
  reserve: async (orderId: string) => {},
  release: async (orderId: string) => {},
};
const paymentService = {
  charge: async (userId: string, amount: number) => {},
  refund: async (userId: string, amount: number) => {},
};
const shippingService = {
  createShipment: async (orderId: string) => {},
  cancelShipment: async (orderId: string) => {},
};
```

---

## Common Pitfalls

### 1. Retry Without Backoff

```typescript
// ❌ BAD: Immediate retries create thundering herd
for (let i = 0; i < 3; i++) {
  try {
    return await fetch(url);
  } catch (e) {
    continue; // Immediately retry!
  }
}

// ✅ GOOD: Exponential backoff with jitter
let delay = 1000;
for (let i = 0; i < 3; i++) {
  try {
    return await fetch(url);
  } catch (e) {
    await sleep(delay + Math.random() * delay * 0.3);
    delay *= 2;
  }
}

const url = '';
function sleep(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

### 2. Circuit Breaker Without Monitoring

```typescript
// ❌ BAD: Circuit breaker with no visibility
const breaker = new CircuitBreaker(options);
// Can't tell when circuit opens!

// ✅ GOOD: Add monitoring and alerting
const breaker = new CircuitBreaker({
  ...options,
  onStateChange: (from, to) => {
    metrics.recordCircuitState(serviceName, to);
    if (to === 'OPEN') {
      alerting.send(`Circuit opened for ${serviceName}`);
    }
  },
});

const options = { failureThreshold: 5, successThreshold: 2, timeout: 30000 };
const serviceName = 'payment';
const metrics = { recordCircuitState: (name: string, state: string) => {} };
const alerting = { send: (msg: string) => {} };
```

### 3. Bulkhead Too Restrictive

```typescript
// ❌ BAD: Too few concurrent requests
const bulkhead = new Bulkhead({
  maxConcurrent: 2,  // Way too low for high-traffic API!
  maxQueued: 5,
});

// ✅ GOOD: Size based on downstream capacity
const bulkhead = new Bulkhead({
  maxConcurrent: 50,  // Match what downstream can handle
  maxQueued: 100,
  timeout: 5000,
});
```

---

## Interview Questions

### Q1: Explain the Circuit Breaker pattern and when to use it.

**A:** Circuit Breaker prevents cascade failures by stopping calls to a failing service. Three states: Closed (normal), Open (failing, reject immediately), Half-Open (testing recovery). Use when: calling external services, microservices, databases. Benefits: fail fast, give services time to recover, prevent resource exhaustion.

### Q2: How does retry with exponential backoff work?

**A:** After each failed attempt, wait increasingly longer: 1s, 2s, 4s, 8s... This prevents overwhelming a recovering service. Add jitter (random variation) to prevent synchronized retries from multiple clients. Cap maximum delay. Only retry idempotent operations or transient errors.

### Q3: What's the difference between Bulkhead and Rate Limiting?

**A:** **Bulkhead** isolates resources - limits concurrent operations to prevent one component from exhausting shared resources. **Rate Limiting** controls throughput - limits requests per time period. Bulkhead protects your system; rate limiting protects downstream systems or enforces quotas.

### Q4: When would you use the Saga pattern?

**A:** For distributed transactions across multiple services where 2PC isn't practical. Each step has a compensating action. If any step fails, compensate completed steps in reverse order. Use for: order processing, booking systems, any multi-service workflow requiring consistency.

---

## Quick Reference Checklist

### Circuit Breaker
- [ ] Set appropriate failure threshold
- [ ] Configure timeout before half-open
- [ ] Add monitoring for state changes
- [ ] Log when circuit opens/closes

### Retry
- [ ] Use exponential backoff
- [ ] Add jitter to prevent thundering herd
- [ ] Only retry transient errors
- [ ] Set maximum attempts and delay

### Timeout
- [ ] Set timeouts on all external calls
- [ ] Use cascading timeouts in chains
- [ ] Handle timeout errors gracefully
- [ ] Log slow operations

### Bulkhead
- [ ] Size based on downstream capacity
- [ ] Separate pools for critical services
- [ ] Monitor queue length
- [ ] Alert on rejections

---

*Last updated: February 2026*

