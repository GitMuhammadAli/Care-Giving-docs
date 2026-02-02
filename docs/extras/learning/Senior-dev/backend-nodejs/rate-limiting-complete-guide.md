# Rate Limiting Implementation - Complete Guide

> **MUST REMEMBER**: Rate limiting protects your API from abuse and ensures fair resource distribution. Common algorithms: token bucket (smooth traffic), sliding window (accurate counts), fixed window (simple but bursty). Use Redis for distributed rate limiting across multiple servers. Always include rate limit headers (X-RateLimit-*) so clients can self-regulate.

---

## How to Explain Like a Senior Developer

"Rate limiting is about protecting your system while being fair to legitimate users. The choice of algorithm matters: fixed window is simple but allows burst at window boundaries, sliding window is more accurate but needs more storage, token bucket allows bursts within limits which is often desired. For distributed systems, you need a shared store like Redis. The key insight is that rate limiting should be configurable per endpoint, per user tier, and should communicate limits clearly via headers. Advanced setups use different limits for authenticated vs anonymous users, and implement graceful degradation rather than hard cutoffs."

---

## Core Implementation

### Express Rate Limiter with Redis

```typescript
// rate-limiter.ts
import { Request, Response, NextFunction } from 'express';
import Redis from 'ioredis';

const redis = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT || '6379'),
});

interface RateLimitConfig {
  windowMs: number;      // Time window in milliseconds
  maxRequests: number;   // Max requests per window
  keyGenerator?: (req: Request) => string;
  skip?: (req: Request) => boolean;
  handler?: (req: Request, res: Response) => void;
}

interface RateLimitInfo {
  remaining: number;
  reset: number;
  total: number;
}

// Fixed Window Rate Limiter
export function fixedWindowRateLimiter(config: RateLimitConfig) {
  const {
    windowMs,
    maxRequests,
    keyGenerator = defaultKeyGenerator,
    skip = () => false,
    handler = defaultHandler,
  } = config;
  
  return async (req: Request, res: Response, next: NextFunction) => {
    if (skip(req)) {
      return next();
    }
    
    const key = `ratelimit:fixed:${keyGenerator(req)}`;
    const windowKey = `${key}:${Math.floor(Date.now() / windowMs)}`;
    
    const current = await redis.incr(windowKey);
    
    if (current === 1) {
      await redis.pexpire(windowKey, windowMs);
    }
    
    const reset = Math.ceil((Math.floor(Date.now() / windowMs) + 1) * windowMs / 1000);
    
    setRateLimitHeaders(res, {
      remaining: Math.max(0, maxRequests - current),
      reset,
      total: maxRequests,
    });
    
    if (current > maxRequests) {
      return handler(req, res);
    }
    
    next();
  };
}

// Sliding Window Rate Limiter (more accurate)
export function slidingWindowRateLimiter(config: RateLimitConfig) {
  const {
    windowMs,
    maxRequests,
    keyGenerator = defaultKeyGenerator,
    skip = () => false,
    handler = defaultHandler,
  } = config;
  
  return async (req: Request, res: Response, next: NextFunction) => {
    if (skip(req)) {
      return next();
    }
    
    const key = `ratelimit:sliding:${keyGenerator(req)}`;
    const now = Date.now();
    const windowStart = now - windowMs;
    
    // Use Redis sorted set for sliding window
    const multi = redis.multi();
    multi.zremrangebyscore(key, 0, windowStart);
    multi.zadd(key, now.toString(), `${now}-${Math.random()}`);
    multi.zcard(key);
    multi.pexpire(key, windowMs);
    
    const results = await multi.exec();
    const requestCount = results?.[2]?.[1] as number || 0;
    
    const reset = Math.ceil((now + windowMs) / 1000);
    
    setRateLimitHeaders(res, {
      remaining: Math.max(0, maxRequests - requestCount),
      reset,
      total: maxRequests,
    });
    
    if (requestCount > maxRequests) {
      return handler(req, res);
    }
    
    next();
  };
}

// Token Bucket Rate Limiter (allows controlled bursts)
export function tokenBucketRateLimiter(config: {
  bucketSize: number;       // Max tokens (burst capacity)
  refillRate: number;       // Tokens added per second
  keyGenerator?: (req: Request) => string;
  skip?: (req: Request) => boolean;
  handler?: (req: Request, res: Response) => void;
}) {
  const {
    bucketSize,
    refillRate,
    keyGenerator = defaultKeyGenerator,
    skip = () => false,
    handler = defaultHandler,
  } = config;
  
  return async (req: Request, res: Response, next: NextFunction) => {
    if (skip(req)) {
      return next();
    }
    
    const key = `ratelimit:bucket:${keyGenerator(req)}`;
    const now = Date.now();
    
    // Lua script for atomic token bucket operation
    const luaScript = `
      local key = KEYS[1]
      local bucket_size = tonumber(ARGV[1])
      local refill_rate = tonumber(ARGV[2])
      local now = tonumber(ARGV[3])
      local requested = tonumber(ARGV[4])
      
      local bucket = redis.call('HMGET', key, 'tokens', 'last_refill')
      local tokens = tonumber(bucket[1]) or bucket_size
      local last_refill = tonumber(bucket[2]) or now
      
      -- Calculate tokens to add
      local elapsed = (now - last_refill) / 1000
      local tokens_to_add = elapsed * refill_rate
      tokens = math.min(bucket_size, tokens + tokens_to_add)
      
      -- Try to consume token
      local allowed = 0
      if tokens >= requested then
        tokens = tokens - requested
        allowed = 1
      end
      
      -- Update bucket
      redis.call('HMSET', key, 'tokens', tokens, 'last_refill', now)
      redis.call('PEXPIRE', key, math.ceil(bucket_size / refill_rate * 1000))
      
      return {allowed, tokens}
    `;
    
    const result = await redis.eval(
      luaScript,
      1,
      key,
      bucketSize.toString(),
      refillRate.toString(),
      now.toString(),
      '1'
    ) as [number, number];
    
    const [allowed, tokensRemaining] = result;
    
    setRateLimitHeaders(res, {
      remaining: Math.floor(tokensRemaining),
      reset: Math.ceil(now / 1000 + (bucketSize - tokensRemaining) / refillRate),
      total: bucketSize,
    });
    
    if (!allowed) {
      return handler(req, res);
    }
    
    next();
  };
}

// Helper functions
function defaultKeyGenerator(req: Request): string {
  // Use authenticated user ID or IP
  return (req as any).user?.id || 
         req.headers['x-forwarded-for']?.toString().split(',')[0] || 
         req.ip || 
         'anonymous';
}

function defaultHandler(req: Request, res: Response): void {
  res.status(429).json({
    error: 'Too Many Requests',
    message: 'Rate limit exceeded. Please try again later.',
    retryAfter: res.getHeader('Retry-After'),
  });
}

function setRateLimitHeaders(res: Response, info: RateLimitInfo): void {
  res.setHeader('X-RateLimit-Limit', info.total);
  res.setHeader('X-RateLimit-Remaining', info.remaining);
  res.setHeader('X-RateLimit-Reset', info.reset);
  
  if (info.remaining === 0) {
    res.setHeader('Retry-After', info.reset - Math.floor(Date.now() / 1000));
  }
}
```

### Tiered Rate Limiting

```typescript
// tiered-rate-limiter.ts
import { Request, Response, NextFunction } from 'express';

interface TierConfig {
  name: string;
  requestsPerMinute: number;
  requestsPerHour: number;
  requestsPerDay: number;
}

const tiers: Record<string, TierConfig> = {
  anonymous: {
    name: 'anonymous',
    requestsPerMinute: 10,
    requestsPerHour: 100,
    requestsPerDay: 500,
  },
  free: {
    name: 'free',
    requestsPerMinute: 30,
    requestsPerHour: 500,
    requestsPerDay: 5000,
  },
  pro: {
    name: 'pro',
    requestsPerMinute: 100,
    requestsPerHour: 5000,
    requestsPerDay: 50000,
  },
  enterprise: {
    name: 'enterprise',
    requestsPerMinute: 1000,
    requestsPerHour: 50000,
    requestsPerDay: 500000,
  },
};

// Get user's tier
function getUserTier(req: Request): TierConfig {
  const user = (req as any).user;
  
  if (!user) {
    return tiers.anonymous;
  }
  
  return tiers[user.tier] || tiers.free;
}

// Multi-window rate limiter
export function tieredRateLimiter() {
  return async (req: Request, res: Response, next: NextFunction) => {
    const tier = getUserTier(req);
    const userId = (req as any).user?.id || req.ip;
    
    const checks = [
      { window: 'minute', limit: tier.requestsPerMinute, ttl: 60 },
      { window: 'hour', limit: tier.requestsPerHour, ttl: 3600 },
      { window: 'day', limit: tier.requestsPerDay, ttl: 86400 },
    ];
    
    for (const check of checks) {
      const key = `ratelimit:${check.window}:${userId}`;
      const current = await redis.incr(key);
      
      if (current === 1) {
        await redis.expire(key, check.ttl);
      }
      
      if (current > check.limit) {
        const ttl = await redis.ttl(key);
        
        res.setHeader('X-RateLimit-Limit', check.limit);
        res.setHeader('X-RateLimit-Remaining', 0);
        res.setHeader('X-RateLimit-Reset', Math.floor(Date.now() / 1000) + ttl);
        res.setHeader('X-RateLimit-Window', check.window);
        res.setHeader('Retry-After', ttl);
        
        return res.status(429).json({
          error: 'Rate limit exceeded',
          tier: tier.name,
          window: check.window,
          limit: check.limit,
          retryAfter: ttl,
        });
      }
    }
    
    // Set headers for the most restrictive limit
    res.setHeader('X-RateLimit-Tier', tier.name);
    
    next();
  };
}

import Redis from 'ioredis';
const redis = new Redis();
```

### Per-Endpoint Rate Limiting

```typescript
// endpoint-rate-limiter.ts
import { Router, Request, Response, NextFunction } from 'express';
import { slidingWindowRateLimiter, tokenBucketRateLimiter } from './rate-limiter';

const router = Router();

// Different limits for different endpoints
const endpointLimits = {
  // Auth endpoints - strict limits
  'POST:/auth/login': { windowMs: 60000, maxRequests: 5 },
  'POST:/auth/register': { windowMs: 3600000, maxRequests: 3 },
  'POST:/auth/forgot-password': { windowMs: 3600000, maxRequests: 3 },
  
  // API endpoints - standard limits
  'GET:/api/users': { windowMs: 60000, maxRequests: 100 },
  'POST:/api/posts': { windowMs: 60000, maxRequests: 10 },
  
  // Heavy operations - very strict
  'POST:/api/export': { windowMs: 3600000, maxRequests: 5 },
  'POST:/api/import': { windowMs: 3600000, maxRequests: 3 },
};

// Middleware to apply endpoint-specific limits
export function endpointRateLimiter() {
  return async (req: Request, res: Response, next: NextFunction) => {
    const endpointKey = `${req.method}:${req.route?.path || req.path}`;
    const config = endpointLimits[endpointKey as keyof typeof endpointLimits];
    
    if (!config) {
      // Default limit for unspecified endpoints
      return slidingWindowRateLimiter({
        windowMs: 60000,
        maxRequests: 60,
      })(req, res, next);
    }
    
    return slidingWindowRateLimiter({
      ...config,
      keyGenerator: (req) => {
        const userId = (req as any).user?.id || req.ip;
        return `${endpointKey}:${userId}`;
      },
    })(req, res, next);
  };
}

// Decorator-style rate limiting for routes
function rateLimit(windowMs: number, maxRequests: number) {
  return slidingWindowRateLimiter({ windowMs, maxRequests });
}

// Usage with routes
router.post('/login',
  rateLimit(60000, 5), // 5 per minute
  async (req, res) => {
    // Login logic
  }
);

router.get('/users',
  rateLimit(60000, 100), // 100 per minute
  async (req, res) => {
    // Get users logic
  }
);
```

### Distributed Rate Limiting with Lua Scripts

```typescript
// distributed-rate-limiter.ts
import Redis from 'ioredis';

const redis = new Redis.Cluster([
  { host: 'redis-1', port: 6379 },
  { host: 'redis-2', port: 6379 },
  { host: 'redis-3', port: 6379 },
]);

// Atomic sliding window counter using Lua
const slidingWindowLua = `
  local key = KEYS[1]
  local window_size = tonumber(ARGV[1])
  local max_requests = tonumber(ARGV[2])
  local now = tonumber(ARGV[3])
  local window_start = now - window_size

  -- Remove old entries
  redis.call('ZREMRANGEBYSCORE', key, '-inf', window_start)

  -- Count current entries
  local current = redis.call('ZCARD', key)

  -- Check if allowed
  if current < max_requests then
    -- Add new entry
    redis.call('ZADD', key, now, now .. '-' .. math.random())
    redis.call('PEXPIRE', key, window_size)
    return {1, max_requests - current - 1}
  else
    return {0, 0}
  end
`;

// Atomic token bucket using Lua
const tokenBucketLua = `
  local key = KEYS[1]
  local capacity = tonumber(ARGV[1])
  local fill_rate = tonumber(ARGV[2])
  local now = tonumber(ARGV[3])
  local tokens_requested = tonumber(ARGV[4])

  -- Get current state
  local bucket = redis.call('HMGET', key, 'tokens', 'timestamp')
  local tokens = tonumber(bucket[1])
  local timestamp = tonumber(bucket[2])

  -- Initialize if new bucket
  if tokens == nil then
    tokens = capacity
    timestamp = now
  end

  -- Calculate new tokens
  local elapsed = now - timestamp
  local new_tokens = math.min(capacity, tokens + (elapsed / 1000 * fill_rate))

  -- Try to consume
  local allowed = 0
  local remaining = new_tokens

  if new_tokens >= tokens_requested then
    remaining = new_tokens - tokens_requested
    allowed = 1
  end

  -- Update state
  redis.call('HMSET', key, 'tokens', remaining, 'timestamp', now)
  redis.call('PEXPIRE', key, math.ceil(capacity / fill_rate * 1000) * 2)

  return {allowed, math.floor(remaining)}
`;

export class DistributedRateLimiter {
  constructor(private redis: Redis.Cluster) {}
  
  async checkSlidingWindow(
    key: string,
    windowMs: number,
    maxRequests: number
  ): Promise<{ allowed: boolean; remaining: number }> {
    const result = await this.redis.eval(
      slidingWindowLua,
      1,
      key,
      windowMs.toString(),
      maxRequests.toString(),
      Date.now().toString()
    ) as [number, number];
    
    return {
      allowed: result[0] === 1,
      remaining: result[1],
    };
  }
  
  async checkTokenBucket(
    key: string,
    capacity: number,
    fillRate: number,
    tokensRequested: number = 1
  ): Promise<{ allowed: boolean; remaining: number }> {
    const result = await this.redis.eval(
      tokenBucketLua,
      1,
      key,
      capacity.toString(),
      fillRate.toString(),
      Date.now().toString(),
      tokensRequested.toString()
    ) as [number, number];
    
    return {
      allowed: result[0] === 1,
      remaining: result[1],
    };
  }
}
```

---

## Real-World Scenarios

### Scenario 1: API Gateway Rate Limiting

```typescript
// api-gateway-limiter.ts
import express from 'express';
import { slidingWindowRateLimiter, tokenBucketRateLimiter } from './rate-limiter';

const app = express();

// Global rate limit
app.use(slidingWindowRateLimiter({
  windowMs: 60000,
  maxRequests: 1000,
  keyGenerator: (req) => req.ip || 'unknown',
}));

// API key based rate limiting
app.use('/api', async (req, res, next) => {
  const apiKey = req.headers['x-api-key'] as string;
  
  if (!apiKey) {
    return res.status(401).json({ error: 'API key required' });
  }
  
  // Look up API key limits
  const keyConfig = await getApiKeyConfig(apiKey);
  if (!keyConfig) {
    return res.status(401).json({ error: 'Invalid API key' });
  }
  
  // Apply custom rate limit
  return tokenBucketRateLimiter({
    bucketSize: keyConfig.burstLimit,
    refillRate: keyConfig.requestsPerSecond,
    keyGenerator: () => apiKey,
  })(req, res, next);
});

// Cost-based rate limiting (for expensive operations)
const operationCosts: Record<string, number> = {
  'GET:/api/simple': 1,
  'GET:/api/complex': 5,
  'POST:/api/export': 50,
  'POST:/api/ai/generate': 100,
};

app.use('/api', tokenBucketRateLimiter({
  bucketSize: 1000,
  refillRate: 10, // 10 credits per second
  keyGenerator: (req) => {
    const apiKey = req.headers['x-api-key'] as string;
    return `credits:${apiKey}`;
  },
  // Custom handler to check cost
}));

async function getApiKeyConfig(apiKey: string): Promise<{
  burstLimit: number;
  requestsPerSecond: number;
} | null> {
  // Look up from database/cache
  return { burstLimit: 100, requestsPerSecond: 10 };
}
```

### Scenario 2: DDoS Protection Layer

```typescript
// ddos-protection.ts
import { Request, Response, NextFunction } from 'express';
import Redis from 'ioredis';

const redis = new Redis();

interface SuspiciousActivity {
  score: number;
  reasons: string[];
}

async function detectSuspiciousActivity(req: Request): Promise<SuspiciousActivity> {
  const ip = req.ip || 'unknown';
  const reasons: string[] = [];
  let score = 0;
  
  // Check request rate
  const requestKey = `requests:${ip}:${Math.floor(Date.now() / 1000)}`;
  const requestCount = await redis.incr(requestKey);
  await redis.expire(requestKey, 60);
  
  if (requestCount > 100) {
    score += 30;
    reasons.push('High request rate');
  }
  
  // Check for missing headers
  if (!req.headers['user-agent']) {
    score += 20;
    reasons.push('Missing User-Agent');
  }
  
  // Check for suspicious patterns
  if (req.path.includes('..') || req.path.includes('//')) {
    score += 40;
    reasons.push('Suspicious path pattern');
  }
  
  // Check if IP is in known bad list
  const isBadIP = await redis.sismember('bad-ips', ip);
  if (isBadIP) {
    score += 100;
    reasons.push('Known bad IP');
  }
  
  return { score, reasons };
}

export function ddosProtection() {
  return async (req: Request, res: Response, next: NextFunction) => {
    const { score, reasons } = await detectSuspiciousActivity(req);
    
    if (score >= 100) {
      // Block completely
      console.warn('Blocked suspicious request', {
        ip: req.ip,
        score,
        reasons,
      });
      return res.status(403).json({ error: 'Access denied' });
    }
    
    if (score >= 50) {
      // Apply strict rate limiting
      const key = `ratelimit:suspicious:${req.ip}`;
      const count = await redis.incr(key);
      await redis.expire(key, 60);
      
      if (count > 10) {
        return res.status(429).json({
          error: 'Rate limit exceeded',
          retryAfter: 60,
        });
      }
    }
    
    next();
  };
}

// Automatically add IPs to bad list after repeated violations
export async function trackViolation(ip: string): Promise<void> {
  const key = `violations:${ip}`;
  const count = await redis.incr(key);
  await redis.expire(key, 3600); // 1 hour window
  
  if (count >= 10) {
    await redis.sadd('bad-ips', ip);
    await redis.expire('bad-ips', 86400); // 24 hour block
    console.warn(`Added ${ip} to bad IP list after ${count} violations`);
  }
}
```

---

## Common Pitfalls

### 1. Race Conditions in Rate Limiting

```typescript
// ❌ BAD: Race condition between read and write
async function checkRateLimit(key: string, limit: number): Promise<boolean> {
  const current = await redis.get(key);
  if (parseInt(current || '0') >= limit) {
    return false;
  }
  await redis.incr(key); // Another request could have incremented!
  return true;
}

// ✅ GOOD: Atomic operation
async function checkRateLimitAtomic(key: string, limit: number): Promise<boolean> {
  const current = await redis.incr(key);
  if (current === 1) {
    await redis.expire(key, 60);
  }
  return current <= limit;
}
```

### 2. Not Handling Redis Failures

```typescript
// ❌ BAD: Redis failure blocks all requests
async function rateLimitMiddleware(req: Request, res: Response, next: NextFunction) {
  const allowed = await redis.get(`limit:${req.ip}`); // Throws on failure
  // ...
}

// ✅ GOOD: Fail open with logging
async function rateLimitMiddleware(req: Request, res: Response, next: NextFunction) {
  try {
    const allowed = await redis.get(`limit:${req.ip}`);
    // ...
  } catch (error) {
    console.error('Rate limiter Redis error:', error);
    // Fail open - allow request but log
    next();
  }
}
```

### 3. Inconsistent Key Generation

```typescript
// ❌ BAD: Inconsistent across servers
const key = `limit:${req.connection.remoteAddress}`; // Different behind proxy

// ✅ GOOD: Consistent key generation
const key = `limit:${
  req.headers['x-forwarded-for']?.toString().split(',')[0].trim() ||
  req.ip ||
  'unknown'
}`;
```

---

## Interview Questions

### Q1: What's the difference between token bucket and sliding window algorithms?

**A:** **Token bucket** allows bursts up to bucket capacity, then throttles to refill rate. Good when you want to allow temporary spikes. **Sliding window** counts requests over a moving time window, providing smooth rate limiting without burst allowance. Sliding window is more accurate but requires more storage (tracking each request timestamp).

### Q2: How do you implement rate limiting in a distributed system?

**A:** Use a centralized store like Redis that all servers can access. Use Lua scripts for atomic operations to prevent race conditions. Consider Redis Cluster for high availability. Account for network latency in timeout calculations. Implement fallback behavior when Redis is unavailable.

### Q3: What rate limit headers should an API return?

**A:** Standard headers include:
- `X-RateLimit-Limit`: Maximum requests allowed
- `X-RateLimit-Remaining`: Requests remaining in window
- `X-RateLimit-Reset`: Unix timestamp when the limit resets
- `Retry-After`: Seconds until client should retry (on 429)

### Q4: How do you prevent rate limit bypass via IP rotation?

**A:** Require authentication for API access, limit by API key/user ID instead of IP. Use CAPTCHA for anonymous endpoints. Implement fingerprinting for anonymous users. Apply stricter limits to new accounts. Use behavioral analysis to detect bot patterns.

---

## Quick Reference Checklist

### Algorithm Selection
- [ ] Token bucket: When burst tolerance is acceptable
- [ ] Sliding window: When accurate counting is required
- [ ] Fixed window: For simplicity (accept boundary bursts)

### Implementation
- [ ] Use atomic operations (Lua scripts)
- [ ] Handle Redis failures gracefully
- [ ] Consistent key generation across servers
- [ ] Include all rate limit headers

### Configuration
- [ ] Different limits per endpoint
- [ ] Tiered limits by user plan
- [ ] Stricter limits for auth endpoints
- [ ] Cost-based limits for expensive operations

### Monitoring
- [ ] Track rate limit hits
- [ ] Alert on unusual patterns
- [ ] Monitor Redis performance
- [ ] Log blocked requests

---

*Last updated: February 2026*

