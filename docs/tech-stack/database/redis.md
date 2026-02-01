# Redis

> In-memory data store for caching and queues.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | In-memory key-value store |
| **Why** | Caching, session storage, job queues |
| **Version** | 7.x |
| **Provider** | Upstash (cloud) / Docker (local) |

## Local Setup

```yaml
# docker-compose.yml
services:
  redis:
    image: redis:7-alpine
    container_name: carecircle-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    command: redis-server --appendonly yes
```

## Connection

```env
# Local
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_TLS=false

# Upstash (Cloud)
REDIS_HOST=bright-ladybird-xxxxx.upstash.io
REDIS_PORT=6379
REDIS_PASSWORD=your-upstash-password
REDIS_TLS=true
```

## Use Cases

### 1. Caching

```typescript
// Cache user profile
const cacheKey = `user:${userId}`;
const ttl = 300; // 5 minutes

// Get from cache
const cached = await redis.get(cacheKey);
if (cached) {
  return JSON.parse(cached);
}

// Fetch and cache
const user = await prisma.user.findUnique({ where: { id: userId } });
await redis.setex(cacheKey, ttl, JSON.stringify(user));
return user;
```

### 2. Session Storage

```typescript
// Store session
const sessionKey = `session:${refreshToken}`;
await redis.setex(sessionKey, 60 * 60 * 24 * 7, JSON.stringify({
  userId,
  userAgent,
  ipAddress,
}));

// Validate session
const session = await redis.get(sessionKey);
if (!session) throw new UnauthorizedException();

// Revoke session
await redis.del(sessionKey);
```

### 3. Rate Limiting

```typescript
// Simple rate limiter
const key = `ratelimit:${ip}:${endpoint}`;
const current = await redis.incr(key);

if (current === 1) {
  await redis.expire(key, 60); // 1 minute window
}

if (current > 100) {
  throw new TooManyRequestsException();
}
```

### 4. Job Queues (BullMQ)

```typescript
// BullMQ uses Redis for queue storage
const queue = new Queue('notifications', {
  connection: {
    host: process.env.REDIS_HOST,
    port: parseInt(process.env.REDIS_PORT),
    password: process.env.REDIS_PASSWORD,
    tls: process.env.REDIS_TLS === 'true' ? {} : undefined,
  },
});
```

## Cache Patterns

### Cache-Aside (Lazy Loading)

```typescript
async getUser(userId: string) {
  // 1. Check cache
  const cached = await this.cache.get(`user:${userId}`);
  if (cached) return cached;

  // 2. Load from DB
  const user = await this.prisma.user.findUnique({ where: { id: userId } });
  
  // 3. Store in cache
  await this.cache.set(`user:${userId}`, user, 300);
  
  return user;
}
```

### Write-Through

```typescript
async updateUser(userId: string, data: UpdateUserDto) {
  // 1. Update DB
  const user = await this.prisma.user.update({
    where: { id: userId },
    data,
  });

  // 2. Update cache
  await this.cache.set(`user:${userId}`, user, 300);

  return user;
}
```

### Cache Invalidation

```typescript
// Invalidate on update
async invalidateUserCache(userId: string) {
  await redis.del(`user:${userId}`);
}

// Pattern-based invalidation
async invalidateFamilyCache(familyId: string) {
  const keys = await redis.keys(`family:${familyId}:*`);
  if (keys.length > 0) {
    await redis.del(...keys);
  }
}
```

## Cache Service

```typescript
// system/module/cache/cache.service.ts
@Injectable()
export class CacheService {
  constructor(@InjectRedis() private redis: Redis) {}

  async get<T>(key: string): Promise<T | null> {
    const value = await this.redis.get(key);
    return value ? JSON.parse(value) : null;
  }

  async set(key: string, value: any, ttlSeconds: number): Promise<void> {
    await this.redis.setex(key, ttlSeconds, JSON.stringify(value));
  }

  async del(key: string): Promise<void> {
    await this.redis.del(key);
  }

  async getOrSet<T>(
    key: string,
    fetcher: () => Promise<T>,
    ttlSeconds: number
  ): Promise<T> {
    const cached = await this.get<T>(key);
    if (cached) return cached;

    const value = await fetcher();
    await this.set(key, value, ttlSeconds);
    return value;
  }
}
```

## Upstash Setup

1. **Create Account**: [upstash.com](https://upstash.com)
2. **Create Database**: Choose region close to your app
3. **Get Credentials**: REST URL, REST Token, Redis URL
4. **Enable TLS**: Always use TLS for cloud

### Upstash Features

- **Serverless**: Pay per request
- **Global**: Multi-region replication
- **REST API**: HTTP access available
- **Free Tier**: 10K commands/day

### Upstash Optimization

```typescript
// Reduce Redis requests to stay in free tier
const defaultJobOptions = {
  // Increase drain delay (less polling)
  settings: {
    drainDelay: process.env.NODE_ENV === 'development' ? 5000 : 1000,
  },
};

// Disable QueueEvents in development (saves requests)
const shouldDisableQueueEvents = process.env.NODE_ENV === 'development';
```

## Key Naming Convention

```
user:{userId}                    # User profile cache
session:{token}                  # Session data
family:{familyId}:members        # Family members cache
medication:{id}:schedule         # Medication schedule
ratelimit:{ip}:{endpoint}        # Rate limiting
bull:{queueName}:{jobId}         # BullMQ internal
```

## Monitoring

### Memory Usage
```bash
redis-cli INFO memory
```

### Key Statistics
```bash
redis-cli INFO keyspace
redis-cli DBSIZE
```

### Slow Log
```bash
redis-cli SLOWLOG GET 10
```

## Common Commands

```bash
# Connect
redis-cli -h localhost -p 6379

# Basic operations
SET key value
GET key
DEL key
EXPIRE key 300

# List keys
KEYS user:*
SCAN 0 MATCH user:* COUNT 100

# Check TTL
TTL key

# Clear all (careful!)
FLUSHDB
```

## Troubleshooting

### Connection Issues
- Check host/port/password
- Verify TLS settings for cloud
- Check firewall rules

### Memory Issues
- Set appropriate TTLs
- Use SCAN instead of KEYS
- Monitor with INFO memory

### Slow Performance
- Check network latency
- Use pipelining for bulk ops
- Consider connection pooling

---

*See also: [BullMQ](../workers/bullmq.md), [NestJS](../backend/nestjs.md)*


