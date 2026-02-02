# 🚦 API Rate Limiting - Complete Guide

> A comprehensive guide to API rate limiting - token bucket, sliding window, rate limit headers, and protecting your APIs from abuse.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Rate limiting is a technique to control the rate of requests a client can make to an API, protecting services from abuse, ensuring fair usage, and maintaining system stability by rejecting requests that exceed predefined thresholds."

### The 7 Key Concepts (Remember These!)
```
1. TOKEN BUCKET      → Tokens replenish over time, requests consume tokens
2. SLIDING WINDOW    → Rolling time window for counting requests
3. FIXED WINDOW      → Count resets at fixed intervals
4. LEAKY BUCKET      → Requests processed at constant rate
5. RATE LIMIT HEADERS→ X-RateLimit-Limit, Remaining, Reset
6. 429 STATUS        → Too Many Requests response code
7. BURST ALLOWANCE   → Allow temporary spikes above normal rate
```

### Rate Limiting Algorithms Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│               RATE LIMITING ALGORITHMS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TOKEN BUCKET                                                   │
│  ────────────                                                   │
│  • Tokens added at fixed rate (e.g., 10/second)                │
│  • Bucket has max capacity (burst size)                        │
│  • Request consumes 1 token                                    │
│  • No tokens = rejected                                        │
│  ✅ Allows bursts up to bucket size                            │
│  ✅ Smooth rate limiting                                       │
│  ⭐ Most common algorithm                                      │
│                                                                 │
│  SLIDING WINDOW LOG                                            │
│  ──────────────────                                             │
│  • Log timestamp of each request                               │
│  • Count requests in last N seconds                            │
│  • Reject if count exceeds limit                               │
│  ✅ Very accurate                                              │
│  ❌ Memory intensive (stores all timestamps)                   │
│                                                                 │
│  SLIDING WINDOW COUNTER                                        │
│  ───────────────────────                                        │
│  • Weighted combination of current + previous window           │
│  • Less memory than log                                        │
│  ✅ Good balance of accuracy and efficiency                    │
│  ⭐ Popular choice                                             │
│                                                                 │
│  FIXED WINDOW                                                  │
│  ────────────                                                   │
│  • Counter resets at fixed intervals                           │
│  • Simple implementation                                       │
│  ❌ Burst at window boundaries (2x limit)                      │
│                                                                 │
│  LEAKY BUCKET                                                  │
│  ───────────                                                    │
│  • Requests enter bucket                                       │
│  • Processed at constant rate                                  │
│  ✅ Very smooth output rate                                    │
│  ❌ May add latency (queuing)                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Rate Limit Headers (Standard)
```
┌─────────────────────────────────────────────────────────────────┐
│                  RATE LIMIT HEADERS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Standard Headers (IETF Draft):                                │
│  ───────────────────────────────                                │
│  RateLimit-Limit: 100                # Max requests allowed    │
│  RateLimit-Remaining: 75             # Requests remaining      │
│  RateLimit-Reset: 1640000000         # Unix timestamp reset    │
│                                                                 │
│  Alternative Headers (Common):                                 │
│  ─────────────────────────────                                  │
│  X-RateLimit-Limit: 100              # Max requests            │
│  X-RateLimit-Remaining: 75           # Remaining               │
│  X-RateLimit-Reset: 60               # Seconds until reset     │
│                                                                 │
│  Retry After (429 Response):                                   │
│  ──────────────────────────                                     │
│  Retry-After: 60                     # Seconds to wait         │
│  Retry-After: Wed, 21 Oct 2025 07:28:00 GMT  # Or date        │
│                                                                 │
│  Example Response:                                             │
│  ─────────────────                                              │
│  HTTP/1.1 429 Too Many Requests                                │
│  RateLimit-Limit: 100                                          │
│  RateLimit-Remaining: 0                                        │
│  RateLimit-Reset: 1640000060                                   │
│  Retry-After: 60                                               │
│  Content-Type: application/json                                │
│                                                                 │
│  {                                                             │
│    "error": "rate_limit_exceeded",                             │
│    "message": "Too many requests. Retry after 60 seconds.",    │
│    "retry_after": 60                                           │
│  }                                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Token bucket"** | "We use token bucket algorithm for smooth rate limiting" |
| **"Sliding window"** | "Sliding window counters avoid boundary spikes" |
| **"Burst allowance"** | "Token bucket allows burst up to bucket capacity" |
| **"Distributed rate limiting"** | "Redis provides distributed rate limiting across instances" |
| **"Rate limit tiers"** | "Different tiers for free vs paid users" |
| **"Graceful degradation"** | "We return 429 with retry-after for graceful degradation" |

### Key Numbers to Remember
| Metric | Example Value | Why |
|--------|---------------|-----|
| Free tier | **100 req/min** | Basic access |
| Pro tier | **1000 req/min** | Paid users |
| Enterprise | **10000 req/min** | High volume |
| Retry-After | **60 seconds** | Reasonable wait |

### The "Wow" Statement (Memorize This!)
> "We implement rate limiting with token bucket algorithm using Redis for distributed counting. Each API key has a tier (free: 100/min, pro: 1000/min). We return standard rate limit headers (RateLimit-Limit, Remaining, Reset) on every response so clients can self-throttle. 429 responses include Retry-After header. We have separate limits per endpoint - expensive operations like reports have lower limits. For abuse prevention, we also have IP-based limits as a fallback. Sliding window counters prevent the boundary burst problem of fixed windows. We monitor rate limit hits in our metrics to identify clients who need higher tiers."

---

## 📚 Table of Contents

1. [Token Bucket Algorithm](#1-token-bucket-algorithm)
2. [Sliding Window](#2-sliding-window)
3. [Implementation](#3-implementation)
4. [Rate Limit Tiers](#4-rate-limit-tiers)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Token Bucket Algorithm

```typescript
// ════════════════════════════════════════════════════════════════
// TOKEN BUCKET IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

interface TokenBucket {
  tokens: number;
  lastRefill: number;
  capacity: number;
  refillRate: number; // tokens per second
}

class TokenBucketRateLimiter {
  private buckets: Map<string, TokenBucket> = new Map();
  
  constructor(
    private capacity: number = 100,      // Max tokens (burst size)
    private refillRate: number = 10      // Tokens per second
  ) {}

  async isAllowed(key: string, tokensRequired: number = 1): Promise<{
    allowed: boolean;
    remaining: number;
    resetIn: number;
  }> {
    const now = Date.now();
    let bucket = this.buckets.get(key);

    if (!bucket) {
      bucket = {
        tokens: this.capacity,
        lastRefill: now,
        capacity: this.capacity,
        refillRate: this.refillRate,
      };
      this.buckets.set(key, bucket);
    }

    // Refill tokens based on time elapsed
    const timePassed = (now - bucket.lastRefill) / 1000;
    const tokensToAdd = timePassed * bucket.refillRate;
    bucket.tokens = Math.min(bucket.capacity, bucket.tokens + tokensToAdd);
    bucket.lastRefill = now;

    // Check if request is allowed
    if (bucket.tokens >= tokensRequired) {
      bucket.tokens -= tokensRequired;
      return {
        allowed: true,
        remaining: Math.floor(bucket.tokens),
        resetIn: Math.ceil((bucket.capacity - bucket.tokens) / bucket.refillRate),
      };
    }

    // Calculate when tokens will be available
    const tokensNeeded = tokensRequired - bucket.tokens;
    const waitTime = Math.ceil(tokensNeeded / bucket.refillRate);

    return {
      allowed: false,
      remaining: 0,
      resetIn: waitTime,
    };
  }
}

// ════════════════════════════════════════════════════════════════
// TOKEN BUCKET WITH REDIS (Distributed)
// ════════════════════════════════════════════════════════════════

import Redis from 'ioredis';

class RedisTokenBucket {
  constructor(
    private redis: Redis,
    private capacity: number = 100,
    private refillRate: number = 10
  ) {}

  async isAllowed(key: string, tokensRequired: number = 1): Promise<{
    allowed: boolean;
    remaining: number;
    resetIn: number;
  }> {
    const bucketKey = `ratelimit:bucket:${key}`;
    const now = Date.now();

    // Lua script for atomic token bucket operations
    const luaScript = `
      local bucket_key = KEYS[1]
      local capacity = tonumber(ARGV[1])
      local refill_rate = tonumber(ARGV[2])
      local tokens_required = tonumber(ARGV[3])
      local now = tonumber(ARGV[4])
      
      -- Get current bucket state
      local bucket = redis.call('HMGET', bucket_key, 'tokens', 'last_refill')
      local tokens = tonumber(bucket[1]) or capacity
      local last_refill = tonumber(bucket[2]) or now
      
      -- Calculate tokens to add
      local time_passed = (now - last_refill) / 1000
      local tokens_to_add = time_passed * refill_rate
      tokens = math.min(capacity, tokens + tokens_to_add)
      
      -- Check if allowed
      local allowed = 0
      if tokens >= tokens_required then
        tokens = tokens - tokens_required
        allowed = 1
      end
      
      -- Update bucket
      redis.call('HMSET', bucket_key, 'tokens', tokens, 'last_refill', now)
      redis.call('EXPIRE', bucket_key, 3600)  -- 1 hour TTL
      
      -- Calculate reset time
      local reset_in = math.ceil((capacity - tokens) / refill_rate)
      
      return {allowed, math.floor(tokens), reset_in}
    `;

    const result = await this.redis.eval(
      luaScript,
      1,
      bucketKey,
      this.capacity,
      this.refillRate,
      tokensRequired,
      now
    ) as [number, number, number];

    return {
      allowed: result[0] === 1,
      remaining: result[1],
      resetIn: result[2],
    };
  }
}

// ════════════════════════════════════════════════════════════════
// EXPRESS MIDDLEWARE WITH TOKEN BUCKET
// ════════════════════════════════════════════════════════════════

import { Request, Response, NextFunction } from 'express';

function tokenBucketMiddleware(rateLimiter: RedisTokenBucket) {
  return async (req: Request, res: Response, next: NextFunction) => {
    // Get rate limit key (API key, user ID, or IP)
    const key = req.headers['x-api-key'] as string || 
                req.user?.id || 
                req.ip;

    const result = await rateLimiter.isAllowed(key);

    // Always set rate limit headers
    res.setHeader('RateLimit-Limit', 100);
    res.setHeader('RateLimit-Remaining', result.remaining);
    res.setHeader('RateLimit-Reset', Math.floor(Date.now() / 1000) + result.resetIn);

    if (!result.allowed) {
      res.setHeader('Retry-After', result.resetIn);
      return res.status(429).json({
        error: 'rate_limit_exceeded',
        message: 'Too many requests',
        retry_after: result.resetIn,
      });
    }

    next();
  };
}

// Usage
const rateLimiter = new RedisTokenBucket(redis, 100, 10);
app.use('/api', tokenBucketMiddleware(rateLimiter));
```

---

## 2. Sliding Window

```typescript
// ════════════════════════════════════════════════════════════════
// SLIDING WINDOW LOG (Accurate but memory intensive)
// ════════════════════════════════════════════════════════════════

class SlidingWindowLog {
  constructor(
    private redis: Redis,
    private windowSize: number = 60,  // seconds
    private maxRequests: number = 100
  ) {}

  async isAllowed(key: string): Promise<{
    allowed: boolean;
    remaining: number;
    resetIn: number;
  }> {
    const windowKey = `ratelimit:window:${key}`;
    const now = Date.now();
    const windowStart = now - (this.windowSize * 1000);

    // Lua script for atomic sliding window
    const luaScript = `
      local window_key = KEYS[1]
      local now = tonumber(ARGV[1])
      local window_start = tonumber(ARGV[2])
      local max_requests = tonumber(ARGV[3])
      local window_size = tonumber(ARGV[4])
      
      -- Remove old entries
      redis.call('ZREMRANGEBYSCORE', window_key, 0, window_start)
      
      -- Count current requests
      local current_count = redis.call('ZCARD', window_key)
      
      if current_count < max_requests then
        -- Add new request
        redis.call('ZADD', window_key, now, now .. '-' .. math.random())
        redis.call('EXPIRE', window_key, window_size)
        return {1, max_requests - current_count - 1, window_size}
      else
        -- Get oldest entry to calculate reset time
        local oldest = redis.call('ZRANGE', window_key, 0, 0, 'WITHSCORES')
        local reset_in = 0
        if oldest[2] then
          reset_in = math.ceil((tonumber(oldest[2]) + (window_size * 1000) - now) / 1000)
        end
        return {0, 0, reset_in}
      end
    `;

    const result = await this.redis.eval(
      luaScript,
      1,
      windowKey,
      now,
      windowStart,
      this.maxRequests,
      this.windowSize
    ) as [number, number, number];

    return {
      allowed: result[0] === 1,
      remaining: result[1],
      resetIn: result[2],
    };
  }
}

// ════════════════════════════════════════════════════════════════
// SLIDING WINDOW COUNTER (Memory efficient)
// ════════════════════════════════════════════════════════════════

class SlidingWindowCounter {
  constructor(
    private redis: Redis,
    private windowSize: number = 60,  // seconds
    private maxRequests: number = 100
  ) {}

  async isAllowed(key: string): Promise<{
    allowed: boolean;
    remaining: number;
    resetIn: number;
  }> {
    const now = Date.now();
    const currentWindow = Math.floor(now / (this.windowSize * 1000));
    const previousWindow = currentWindow - 1;
    
    const currentKey = `ratelimit:${key}:${currentWindow}`;
    const previousKey = `ratelimit:${key}:${previousWindow}`;

    // Get counts for both windows
    const [currentCount, previousCount] = await Promise.all([
      this.redis.get(currentKey).then(v => parseInt(v || '0')),
      this.redis.get(previousKey).then(v => parseInt(v || '0')),
    ]);

    // Calculate weighted count
    const windowProgress = (now % (this.windowSize * 1000)) / (this.windowSize * 1000);
    const previousWeight = 1 - windowProgress;
    const weightedCount = currentCount + (previousCount * previousWeight);

    if (weightedCount < this.maxRequests) {
      // Increment current window counter
      await this.redis.multi()
        .incr(currentKey)
        .expire(currentKey, this.windowSize * 2)
        .exec();

      return {
        allowed: true,
        remaining: Math.floor(this.maxRequests - weightedCount - 1),
        resetIn: Math.ceil(this.windowSize * (1 - windowProgress)),
      };
    }

    return {
      allowed: false,
      remaining: 0,
      resetIn: Math.ceil(this.windowSize * (1 - windowProgress)),
    };
  }
}

// ════════════════════════════════════════════════════════════════
// FIXED WINDOW (Simple but has burst problem)
// ════════════════════════════════════════════════════════════════

class FixedWindowCounter {
  constructor(
    private redis: Redis,
    private windowSize: number = 60,
    private maxRequests: number = 100
  ) {}

  async isAllowed(key: string): Promise<{
    allowed: boolean;
    remaining: number;
    resetIn: number;
  }> {
    const now = Math.floor(Date.now() / 1000);
    const windowStart = now - (now % this.windowSize);
    const windowKey = `ratelimit:fixed:${key}:${windowStart}`;

    const count = await this.redis.incr(windowKey);
    
    if (count === 1) {
      await this.redis.expire(windowKey, this.windowSize);
    }

    const resetIn = this.windowSize - (now % this.windowSize);

    if (count <= this.maxRequests) {
      return {
        allowed: true,
        remaining: this.maxRequests - count,
        resetIn,
      };
    }

    return {
      allowed: false,
      remaining: 0,
      resetIn,
    };
  }
}

/*
 * FIXED WINDOW BOUNDARY PROBLEM
 * ─────────────────────────────
 * 
 * Window 1: [0:00 - 1:00]  Window 2: [1:00 - 2:00]
 * 
 * If limit is 100/minute:
 * - 100 requests at 0:59
 * - 100 requests at 1:01
 * = 200 requests in 2 seconds! 😱
 * 
 * Sliding window solves this by considering
 * weighted count from both windows.
 */
```

---

## 3. Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// COMPREHENSIVE RATE LIMITING MIDDLEWARE
// ════════════════════════════════════════════════════════════════

import { Request, Response, NextFunction } from 'express';
import Redis from 'ioredis';

interface RateLimitConfig {
  windowSize: number;       // seconds
  maxRequests: number;      // requests per window
  keyGenerator?: (req: Request) => string;
  skip?: (req: Request) => boolean;
  onRateLimited?: (req: Request, res: Response) => void;
}

interface RateLimitResult {
  allowed: boolean;
  limit: number;
  remaining: number;
  resetAt: number;
  retryAfter: number;
}

class RateLimiter {
  private redis: Redis;

  constructor(redis: Redis) {
    this.redis = redis;
  }

  async checkLimit(key: string, config: RateLimitConfig): Promise<RateLimitResult> {
    const { windowSize, maxRequests } = config;
    const now = Date.now();
    const windowKey = `ratelimit:${key}`;

    // Using sliding window counter
    const currentWindow = Math.floor(now / (windowSize * 1000));
    const previousWindow = currentWindow - 1;
    
    const currentKey = `${windowKey}:${currentWindow}`;
    const previousKey = `${windowKey}:${previousWindow}`;

    const [currentCount, previousCount] = await Promise.all([
      this.redis.get(currentKey).then(v => parseInt(v || '0')),
      this.redis.get(previousKey).then(v => parseInt(v || '0')),
    ]);

    const windowProgress = (now % (windowSize * 1000)) / (windowSize * 1000);
    const previousWeight = 1 - windowProgress;
    const weightedCount = currentCount + (previousCount * previousWeight);
    
    const resetAt = (currentWindow + 1) * windowSize * 1000;
    const retryAfter = Math.ceil((resetAt - now) / 1000);

    if (weightedCount < maxRequests) {
      await this.redis.multi()
        .incr(currentKey)
        .expire(currentKey, windowSize * 2)
        .exec();

      return {
        allowed: true,
        limit: maxRequests,
        remaining: Math.floor(maxRequests - weightedCount - 1),
        resetAt: Math.floor(resetAt / 1000),
        retryAfter,
      };
    }

    return {
      allowed: false,
      limit: maxRequests,
      remaining: 0,
      resetAt: Math.floor(resetAt / 1000),
      retryAfter,
    };
  }
}

// ════════════════════════════════════════════════════════════════
// EXPRESS RATE LIMIT MIDDLEWARE
// ════════════════════════════════════════════════════════════════

function rateLimitMiddleware(rateLimiter: RateLimiter, config: RateLimitConfig) {
  const defaultKeyGenerator = (req: Request): string => {
    return req.headers['x-api-key'] as string ||
           req.user?.id ||
           req.ip ||
           'anonymous';
  };

  const defaultOnRateLimited = (req: Request, res: Response): void => {
    res.status(429).json({
      error: 'rate_limit_exceeded',
      message: 'Too many requests. Please try again later.',
      retry_after: parseInt(res.getHeader('Retry-After') as string),
    });
  };

  return async (req: Request, res: Response, next: NextFunction) => {
    // Skip if configured
    if (config.skip?.(req)) {
      return next();
    }

    const key = (config.keyGenerator || defaultKeyGenerator)(req);
    const result = await rateLimiter.checkLimit(key, config);

    // Set headers on ALL responses
    res.setHeader('RateLimit-Limit', result.limit);
    res.setHeader('RateLimit-Remaining', result.remaining);
    res.setHeader('RateLimit-Reset', result.resetAt);

    if (!result.allowed) {
      res.setHeader('Retry-After', result.retryAfter);
      return (config.onRateLimited || defaultOnRateLimited)(req, res);
    }

    next();
  };
}

// ════════════════════════════════════════════════════════════════
// USAGE EXAMPLES
// ════════════════════════════════════════════════════════════════

const redis = new Redis();
const rateLimiter = new RateLimiter(redis);

// Global rate limit
app.use(rateLimitMiddleware(rateLimiter, {
  windowSize: 60,      // 1 minute
  maxRequests: 100,    // 100 requests per minute
}));

// Per-endpoint rate limits
app.use('/api/auth/login', rateLimitMiddleware(rateLimiter, {
  windowSize: 60,
  maxRequests: 5,      // Only 5 login attempts per minute
  keyGenerator: (req) => `login:${req.ip}`,
}));

app.use('/api/reports', rateLimitMiddleware(rateLimiter, {
  windowSize: 3600,    // 1 hour
  maxRequests: 10,     // 10 reports per hour (expensive operation)
  keyGenerator: (req) => `reports:${req.user.id}`,
}));

// Skip rate limiting for internal requests
app.use('/api/internal', rateLimitMiddleware(rateLimiter, {
  windowSize: 60,
  maxRequests: 1000,
  skip: (req) => req.headers['x-internal-secret'] === process.env.INTERNAL_SECRET,
}));
```

---

## 4. Rate Limit Tiers

```typescript
// ════════════════════════════════════════════════════════════════
// TIERED RATE LIMITING
// ════════════════════════════════════════════════════════════════

interface RateLimitTier {
  name: string;
  requestsPerMinute: number;
  requestsPerDay: number;
  burstSize: number;
  endpoints?: {
    [pattern: string]: {
      requestsPerMinute?: number;
      requestsPerHour?: number;
    };
  };
}

const rateLimitTiers: Record<string, RateLimitTier> = {
  free: {
    name: 'Free',
    requestsPerMinute: 60,
    requestsPerDay: 1000,
    burstSize: 10,
    endpoints: {
      '/api/ai/*': { requestsPerMinute: 5 },
      '/api/reports/*': { requestsPerHour: 10 },
    },
  },
  pro: {
    name: 'Pro',
    requestsPerMinute: 600,
    requestsPerDay: 50000,
    burstSize: 100,
    endpoints: {
      '/api/ai/*': { requestsPerMinute: 60 },
      '/api/reports/*': { requestsPerHour: 100 },
    },
  },
  enterprise: {
    name: 'Enterprise',
    requestsPerMinute: 6000,
    requestsPerDay: 1000000,
    burstSize: 500,
    endpoints: {
      '/api/ai/*': { requestsPerMinute: 600 },
      '/api/reports/*': { requestsPerHour: 1000 },
    },
  },
};

// ════════════════════════════════════════════════════════════════
// TIERED RATE LIMITER MIDDLEWARE
// ════════════════════════════════════════════════════════════════

class TieredRateLimiter {
  constructor(private redis: Redis) {}

  async checkLimits(
    userId: string,
    tier: RateLimitTier,
    endpoint: string
  ): Promise<{
    allowed: boolean;
    limits: {
      minute: { limit: number; remaining: number; resetAt: number };
      daily: { limit: number; remaining: number; resetAt: number };
    };
    retryAfter?: number;
  }> {
    const now = Date.now();
    const minuteKey = `ratelimit:${userId}:minute:${Math.floor(now / 60000)}`;
    const dailyKey = `ratelimit:${userId}:daily:${new Date().toISOString().split('T')[0]}`;

    // Check for endpoint-specific limits
    let minuteLimit = tier.requestsPerMinute;
    for (const [pattern, limits] of Object.entries(tier.endpoints || {})) {
      if (endpoint.match(new RegExp(pattern.replace('*', '.*')))) {
        minuteLimit = limits.requestsPerMinute || minuteLimit;
        break;
      }
    }

    // Get current counts
    const [minuteCount, dailyCount] = await Promise.all([
      this.redis.incr(minuteKey),
      this.redis.incr(dailyKey),
    ]);

    // Set expiry for new keys
    if (minuteCount === 1) await this.redis.expire(minuteKey, 60);
    if (dailyCount === 1) await this.redis.expire(dailyKey, 86400);

    const minuteRemaining = Math.max(0, minuteLimit - minuteCount);
    const dailyRemaining = Math.max(0, tier.requestsPerDay - dailyCount);

    const minuteResetAt = Math.ceil(now / 60000) * 60;
    const dailyResetAt = new Date();
    dailyResetAt.setUTCHours(24, 0, 0, 0);

    // Check if any limit exceeded
    if (minuteCount > minuteLimit) {
      return {
        allowed: false,
        limits: {
          minute: { limit: minuteLimit, remaining: 0, resetAt: minuteResetAt },
          daily: { limit: tier.requestsPerDay, remaining: dailyRemaining, resetAt: Math.floor(dailyResetAt.getTime() / 1000) },
        },
        retryAfter: Math.ceil((minuteResetAt * 1000 - now) / 1000),
      };
    }

    if (dailyCount > tier.requestsPerDay) {
      return {
        allowed: false,
        limits: {
          minute: { limit: minuteLimit, remaining: minuteRemaining, resetAt: minuteResetAt },
          daily: { limit: tier.requestsPerDay, remaining: 0, resetAt: Math.floor(dailyResetAt.getTime() / 1000) },
        },
        retryAfter: Math.ceil((dailyResetAt.getTime() - now) / 1000),
      };
    }

    return {
      allowed: true,
      limits: {
        minute: { limit: minuteLimit, remaining: minuteRemaining, resetAt: minuteResetAt },
        daily: { limit: tier.requestsPerDay, remaining: dailyRemaining, resetAt: Math.floor(dailyResetAt.getTime() / 1000) },
      },
    };
  }
}

// Middleware
function tieredRateLimitMiddleware(rateLimiter: TieredRateLimiter) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const user = req.user;
    if (!user) return next(); // Handle unauthenticated separately

    const tier = rateLimitTiers[user.tier] || rateLimitTiers.free;
    const result = await rateLimiter.checkLimits(user.id, tier, req.path);

    // Set comprehensive headers
    res.setHeader('X-RateLimit-Tier', tier.name);
    res.setHeader('RateLimit-Limit', result.limits.minute.limit);
    res.setHeader('RateLimit-Remaining', result.limits.minute.remaining);
    res.setHeader('RateLimit-Reset', result.limits.minute.resetAt);
    res.setHeader('X-RateLimit-Daily-Limit', result.limits.daily.limit);
    res.setHeader('X-RateLimit-Daily-Remaining', result.limits.daily.remaining);
    res.setHeader('X-RateLimit-Daily-Reset', result.limits.daily.resetAt);

    if (!result.allowed) {
      res.setHeader('Retry-After', result.retryAfter!);
      return res.status(429).json({
        error: 'rate_limit_exceeded',
        message: 'Rate limit exceeded',
        tier: tier.name,
        limits: result.limits,
        retry_after: result.retryAfter,
        upgrade_url: 'https://example.com/pricing',
      });
    }

    next();
  };
}
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# RATE LIMITING BEST PRACTICES
# ════════════════════════════════════════════════════════════════

header_standards:
  always_include:
    - RateLimit-Limit      # Max requests allowed
    - RateLimit-Remaining  # Requests remaining
    - RateLimit-Reset      # When limit resets (Unix timestamp)
  
  on_429_response:
    - Retry-After          # Seconds until retry allowed
  
  optional_extras:
    - X-RateLimit-Daily-Limit
    - X-RateLimit-Daily-Remaining
    - X-RateLimit-Tier

response_format:
  # 429 response body
  error_response:
    error: "rate_limit_exceeded"
    message: "Too many requests. Please retry after 60 seconds."
    retry_after: 60
    limit: 100
    remaining: 0
    reset_at: 1640000000
    # Optional: upgrade CTA
    upgrade_url: "https://api.example.com/pricing"

key_strategies:
  api_key: 
    - Best for authenticated APIs
    - Tied to billing
  
  user_id:
    - Per-user limits
    - Good for logged-in users
  
  ip_address:
    - Fallback for unauthenticated
    - Beware of shared IPs (NAT)
  
  composite:
    - Combine: "user:{userId}:endpoint:{path}"
    - Different limits per endpoint

limit_granularity:
  global:
    - 1000 requests per minute
    - Applies to all endpoints
  
  per_endpoint:
    - /login: 5 per minute (prevent brute force)
    - /search: 30 per minute (expensive)
    - /webhook: 1000 per minute (high volume)
  
  per_resource:
    - Different limits for different resources
    - "POST /orders limited to 10/min"

monitoring:
  metrics_to_track:
    - Rate limit hits per client
    - 429 response rate
    - Clients near limit (>80%)
    - Limit increase requests
  
  alerts:
    - Sudden spike in 429s
    - Client consistently hitting limits
    - Potential abuse patterns
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# RATE LIMITING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Fixed window boundary spike
# Bad - Using fixed window
# At 11:59:59: 100 requests (limit)
# At 12:00:01: 100 requests (new window)
# = 200 requests in 2 seconds!

# Good - Use sliding window or token bucket

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: No rate limit headers
# Bad
HTTP/1.1 429 Too Many Requests
{"error": "Rate limited"}
# Client doesn't know when to retry!

# Good
HTTP/1.1 429 Too Many Requests
RateLimit-Limit: 100
RateLimit-Remaining: 0
RateLimit-Reset: 1640000060
Retry-After: 60
{"error": "rate_limit_exceeded", "retry_after": 60}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: IP-only rate limiting
# Bad
# Rate limit by IP only
# Problem: NAT, shared IPs, proxies

# Good
# Use API key for authenticated requests
# IP as fallback for unauthenticated
# Consider X-Forwarded-For behind load balancers

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: No rate limiting on internal APIs
# Bad - Assuming internal = safe
# One misbehaving service can overwhelm others

# Good
# Rate limit ALL APIs
# Internal services get higher limits
# Circuit breaker for cascade protection

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Blocking instead of throttling
# Bad - Immediately reject everything when limited

# Good - Graceful degradation
# - Return cached response if available
# - Prioritize important requests
# - Queue requests instead of rejecting

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Same limits for all operations
# Bad
# GET /users: 100/min
# POST /heavy-report: 100/min (expensive!)

# Good
# GET /users: 100/min
# POST /heavy-report: 5/hour
# Different limits based on cost
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is rate limiting?"**
> "Rate limiting controls how many requests a client can make in a time period. Protects APIs from abuse, ensures fair usage, prevents resource exhaustion. Example: 100 requests per minute. Returns 429 Too Many Requests when exceeded."

**Q: "What are the common rate limiting algorithms?"**
> "Token bucket: tokens replenish over time, allows bursts. Fixed window: counter resets at intervals, has boundary spike problem. Sliding window: rolling time window, more accurate. Leaky bucket: constant output rate. Token bucket and sliding window are most common."

**Q: "What HTTP status code is used for rate limiting?"**
> "429 Too Many Requests. Should include Retry-After header telling client when to retry. Also include RateLimit headers (Limit, Remaining, Reset) on ALL responses so clients can self-throttle before hitting limits."

### Intermediate Questions

**Q: "What's the difference between fixed window and sliding window?"**
> "Fixed window: Counter resets at fixed times (every minute). Simple but has boundary problem - 2x limit possible at boundaries. Sliding window: Counts requests in rolling period. Can use log (store timestamps) or counter (weighted average of windows). More accurate, no boundary spike."

**Q: "How do you implement distributed rate limiting?"**
> "Use centralized counter in Redis. Lua scripts for atomic operations. Each instance checks Redis before allowing request. Alternatives: rate limit at API gateway level, or use approximate algorithms (allow small error for lower latency)."

**Q: "How do you handle different rate limits for different users?"**
> "Rate limit tiers: free (100/min), pro (1000/min), enterprise (10000/min). Store tier in user profile. Lookup tier at request time. Different limits per endpoint too - expensive operations get lower limits. Include tier in rate limit headers."

### Advanced Questions

**Q: "How does token bucket allow bursts?"**
> "Bucket has capacity (e.g., 100 tokens). Tokens refill at fixed rate (10/sec). Requests consume tokens. If bucket full, burst of 100 requests allowed immediately. Then limited to refill rate. Good for APIs where occasional bursts are acceptable but average should be controlled."

**Q: "How do you handle rate limiting behind load balancers?"**
> "Centralized counting (Redis) so all instances share state. Or rate limit at load balancer/API gateway level. For IP-based: use X-Forwarded-For header (trust first proxy). For local-only rate limiting: each instance gets fraction of total limit."

**Q: "How do you prevent rate limit circumvention?"**
> "Multiple limit keys: API key + IP + user agent. Detect API key sharing (geographic distribution). IP reputation scoring. CAPTCHAs for suspicious traffic. Adaptive limits (lower for suspicious behavior). Monitor for patterns of abuse. Block at WAF level for confirmed abuse."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                 RATE LIMITING CHECKLIST                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  HEADERS (Always include):                                      │
│  □ RateLimit-Limit: 100                                        │
│  □ RateLimit-Remaining: 75                                     │
│  □ RateLimit-Reset: 1640000060                                 │
│  □ Retry-After: 60 (on 429)                                    │
│                                                                 │
│  ALGORITHMS:                                                    │
│  □ Token bucket - smooth, allows bursts                        │
│  □ Sliding window - accurate, no boundary spike                │
│  □ Avoid fixed window for strict limits                        │
│                                                                 │
│  IMPLEMENTATION:                                                │
│  □ Redis for distributed counting                              │
│  □ Lua scripts for atomicity                                   │
│  □ Different limits per endpoint                               │
│  □ Different limits per tier                                   │
│                                                                 │
│  MONITORING:                                                    │
│  □ Track 429 rate                                              │
│  □ Alert on abuse patterns                                     │
│  □ Monitor clients near limits                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

429 RESPONSE EXAMPLE:
┌────────────────────────────────────────────────────────────────┐
│ HTTP/1.1 429 Too Many Requests                                 │
│ RateLimit-Limit: 100                                           │
│ RateLimit-Remaining: 0                                         │
│ RateLimit-Reset: 1640000060                                    │
│ Retry-After: 60                                                │
│ Content-Type: application/json                                 │
│                                                                │
│ {"error":"rate_limit_exceeded","retry_after":60}               │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

