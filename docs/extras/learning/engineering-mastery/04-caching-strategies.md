# Chapter 04: Caching Strategies

> "There are only two hard things in CS: cache invalidation and naming things."

---

## 🎯 Why Caching?

```
Without cache:
User ──► API ──► Database (100ms) ──► Response
         Total: 100ms

With cache:
User ──► API ──► Cache (1ms) ──► Response
         Total: 1ms (100x faster!)
         
Cache miss:
User ──► API ──► Cache (miss) ──► Database ──► Update Cache ──► Response
```

---

## 📊 The Cache Hierarchy

```
Speed      Location          Size        Persistence
───────────────────────────────────────────────────────
Fastest    Browser Cache     10MB        Per user
  │        CDN               Unlimited   Per region
  │        App Memory        1GB         Per instance
  │        Redis             100GB       Shared
  │        Database Cache    10GB        Per DB
Slowest    Disk              TB          Permanent
```

---

## 🔄 Caching Patterns

### 1. Cache-Aside (Lazy Loading)

```
Read:
┌────────┐    1. Check     ┌────────┐
│  App   │ ───────────────►│ Cache  │
│        │◄─── 2. Miss ────│        │
│        │                 └────────┘
│        │    3. Query     ┌────────┐
│        │ ───────────────►│   DB   │
│        │◄─── 4. Data ────│        │
│        │                 └────────┘
│        │    5. Store     ┌────────┐
│        │ ───────────────►│ Cache  │
└────────┘                 └────────┘

Code:
```

```javascript
async function getUser(id) {
  // 1. Check cache
  let user = await cache.get(`user:${id}`);
  
  if (user) {
    return user; // Cache hit!
  }
  
  // 2. Cache miss - query database
  user = await db.users.findById(id);
  
  // 3. Store in cache for next time
  await cache.set(`user:${id}`, user, 'EX', 3600); // 1 hour TTL
  
  return user;
}
```

```
Pros: Simple, only caches what's needed
Cons: Cache miss = slow, stale data possible
Best for: Read-heavy workloads
```

### 2. Write-Through

```
Write:
┌────────┐    1. Write     ┌────────┐    2. Write    ┌────────┐
│  App   │ ───────────────►│ Cache  │──────────────►│   DB   │
│        │                 │        │               │        │
│        │◄───────────────────────────── 3. Ack ───│        │
└────────┘                 └────────┘               └────────┘

Code:
```

```javascript
async function updateUser(id, data) {
  // Write to cache AND database together
  await Promise.all([
    cache.set(`user:${id}`, data, 'EX', 3600),
    db.users.update(id, data)
  ]);
  
  return data;
}
```

```
Pros: Cache always consistent with DB
Cons: Higher write latency
Best for: Data that's read immediately after write
```

### 3. Write-Behind (Write-Back)

```
Write:
┌────────┐    1. Write     ┌────────┐
│  App   │ ───────────────►│ Cache  │
│        │◄─── 2. Ack ─────│  │     │
└────────┘                 │  │     │
                           │  │     │
                           │  ▼     │   3. Async write
                           │ Queue  │──────────────────►┌────────┐
                           └────────┘                   │   DB   │
                                                        └────────┘
Code:
```

```javascript
async function logActivity(data) {
  // Write to cache immediately (fast response)
  await cache.lpush('activity_queue', JSON.stringify(data));
  return { success: true }; // Return immediately
}

// Background worker (runs periodically)
async function flushToDatabase() {
  while (true) {
    const item = await cache.rpop('activity_queue');
    if (item) {
      await db.activities.insert(JSON.parse(item));
    }
  }
}
```

```
Pros: Very fast writes, batching possible
Cons: Data loss risk if cache crashes before flush
Best for: High-write scenarios (analytics, logs)
```

### 4. Read-Through

```
┌────────┐         ┌────────────────────────┐        ┌────────┐
│  App   │ ───────►│        Cache           │───────►│   DB   │
│        │◄────────│  (handles DB queries)  │◄───────│        │
└────────┘         └────────────────────────┘        └────────┘

Cache automatically fetches from DB on miss
```

```
Pros: Simpler app code
Cons: Cache needs DB knowledge
Best for: Simple caching needs
```

### 5. Refresh-Ahead

```
Key expires in 60 seconds
At 50 seconds, background refresh:

┌────────┐                     ┌────────┐
│  App   │                     │ Cache  │
│        │                     │        │
│        │   Still serving     │  ┌──►  │ Background
│        │   from cache        │  │     │ refresh at 50s
│        │                     │  │     │
└────────┘                     └──┼─────┘
                                  │
                                  ▼
                              ┌────────┐
                              │   DB   │
                              └────────┘
```

```
Pros: No cache miss latency spike
Cons: More complex, might refresh unused data
Best for: Frequently accessed hot data
```

---

## 🔑 Cache Key Design

```javascript
// Bad: Generic keys
cache.set('user', userData);  // Which user?

// Good: Specific, namespaced keys
cache.set('users:123', userData);
cache.set('users:123:profile', profileData);
cache.set('users:123:orders:recent', recentOrders);

// Pattern: {entity}:{id}:{sub-entity}:{qualifier}

// For queries:
cache.set('users:list:active:page:1:limit:20', userList);

// For aggregates:
cache.set('stats:users:daily:2024-01-15', dailyStats);
```

### Key Naming Conventions

```
Hierarchical:
  user:123:profile
  user:123:settings
  user:123:friends:list

With versions (for cache busting):
  user:v2:123:profile

With tenant (multi-tenant apps):
  tenant:acme:user:123

With hash (for complex queries):
  query:users:${md5(queryParams)}
```

---

## ⏰ Cache Invalidation Strategies

### 1. Time-To-Live (TTL)

```javascript
// Set expiration
await redis.set('user:123', data, 'EX', 3600); // 1 hour

// Different TTLs for different data:
const TTL = {
  userProfile: 3600,      // 1 hour (changes rarely)
  userSession: 86400,     // 24 hours
  stockPrice: 60,         // 1 minute (changes often)
  staticConfig: 604800,   // 1 week
};
```

### 2. Event-Based Invalidation

```javascript
// When user updates profile
async function updateUserProfile(userId, data) {
  await db.users.update(userId, data);
  
  // Invalidate all related cache keys
  await cache.del(`user:${userId}`);
  await cache.del(`user:${userId}:profile`);
  await cache.del(`team:${data.teamId}:members`);
  
  // Publish event for other services
  await eventBus.publish('user.updated', { userId, data });
}

// Other services listen and invalidate their caches
eventBus.subscribe('user.updated', async ({ userId }) => {
  await cache.del(`recommendations:${userId}`);
});
```

### 3. Version-Based (Cache Busting)

```javascript
let cacheVersion = 1;

function getCacheKey(userId) {
  return `user:v${cacheVersion}:${userId}`;
}

// To invalidate ALL user caches:
function invalidateAllUsers() {
  cacheVersion++;
  // Old keys naturally expire (orphaned)
}
```

### 4. Tag-Based Invalidation

```javascript
// Redis doesn't have native tags, but you can implement:

// Store with tags
async function setWithTags(key, value, tags, ttl) {
  await cache.set(key, value, 'EX', ttl);
  for (const tag of tags) {
    await cache.sadd(`tag:${tag}`, key);
  }
}

// Invalidate by tag
async function invalidateTag(tag) {
  const keys = await cache.smembers(`tag:${tag}`);
  if (keys.length > 0) {
    await cache.del(...keys);
  }
  await cache.del(`tag:${tag}`);
}

// Usage:
await setWithTags('product:123', product, ['products', 'category:electronics'], 3600);
await invalidateTag('category:electronics'); // Invalidates all electronics
```

---

## 🌐 CDN (Content Delivery Network)

### How CDNs Work

```
Without CDN:
User (Tokyo) ──── 200ms ────► Origin (US)

With CDN:
User (Tokyo) ──── 20ms ────► Edge (Tokyo) ──── 200ms ────► Origin (US)
                                   │
                                   └─ Cached! Next request = 20ms
```

### CDN Headers

```http
# Cache-Control header
Cache-Control: public, max-age=3600, s-maxage=86400

public          # CDN can cache
max-age=3600    # Browser cache: 1 hour
s-maxage=86400  # CDN cache: 24 hours

# Other important headers:
Cache-Control: private           # Only browser, not CDN
Cache-Control: no-cache          # Must revalidate
Cache-Control: no-store          # Never cache
Cache-Control: immutable         # Never changes (versioned assets)

# Vary header (cache different versions)
Vary: Accept-Encoding            # Different cache for gzip vs plain
Vary: Accept-Language            # Different cache per language
```

### CDN Purge Strategies

```javascript
// 1. Purge specific URL
await cdn.purge('https://example.com/api/users/123');

// 2. Purge by tag
await cdn.purgeTag('user-123');

// 3. Purge by prefix
await cdn.purgePrefix('/api/users/');

// 4. Versioned URLs (no purge needed)
'/static/app.v2.3.4.js'  // Just change version
```

---

## 🔴 Redis Deep Dive

### Data Structures

```redis
# Strings (most common)
SET user:123 "John"
GET user:123

# With expiration
SET session:abc123 "data" EX 3600

# Atomic increment
INCR page:views:homepage
INCRBY user:123:points 10

# Hash (objects)
HSET user:123 name "John" email "john@test.com"
HGET user:123 name
HGETALL user:123

# List (queues, feeds)
LPUSH notifications:123 "new message"
RPOP notifications:123
LRANGE notifications:123 0 9  # Get first 10

# Set (unique items)
SADD user:123:followers "user:456"
SISMEMBER user:123:followers "user:456"
SMEMBERS user:123:followers

# Sorted Set (leaderboards, ranking)
ZADD leaderboard 100 "user:123"
ZADD leaderboard 95 "user:456"
ZREVRANGE leaderboard 0 9 WITHSCORES  # Top 10

# Geospatial
GEOADD locations 13.361389 52.519444 "Berlin"
GEORADIUS locations 15 37 200 km  # Within 200km
```

### Redis Patterns

```javascript
// 1. Rate Limiting
async function isRateLimited(userId, limit = 100, window = 60) {
  const key = `rate:${userId}`;
  const current = await redis.incr(key);
  
  if (current === 1) {
    await redis.expire(key, window);
  }
  
  return current > limit;
}

// 2. Distributed Lock
async function acquireLock(resource, ttl = 10000) {
  const lockKey = `lock:${resource}`;
  const lockValue = uuid();
  
  const acquired = await redis.set(lockKey, lockValue, 'NX', 'PX', ttl);
  
  return acquired ? lockValue : null;
}

async function releaseLock(resource, lockValue) {
  const script = `
    if redis.call("get", KEYS[1]) == ARGV[1] then
      return redis.call("del", KEYS[1])
    else
      return 0
    end
  `;
  await redis.eval(script, 1, `lock:${resource}`, lockValue);
}

// 3. Pub/Sub
// Publisher
await redis.publish('user.updated', JSON.stringify({ userId: 123 }));

// Subscriber
const subscriber = redis.duplicate();
subscriber.subscribe('user.updated');
subscriber.on('message', (channel, message) => {
  console.log(`${channel}: ${message}`);
});

// 4. Caching with Stampede Protection
async function getWithLock(key, fetchFn, ttl) {
  let data = await redis.get(key);
  if (data) return JSON.parse(data);
  
  const lockKey = `lock:${key}`;
  const acquired = await redis.set(lockKey, '1', 'NX', 'EX', 10);
  
  if (!acquired) {
    // Wait and retry
    await sleep(100);
    return getWithLock(key, fetchFn, ttl);
  }
  
  try {
    data = await fetchFn();
    await redis.set(key, JSON.stringify(data), 'EX', ttl);
    return data;
  } finally {
    await redis.del(lockKey);
  }
}
```

### Redis Cluster

```
┌─────────────────────────────────────────────────────────┐
│                    Redis Cluster                        │
│                                                         │
│  ┌─────────┐     ┌─────────┐     ┌─────────┐          │
│  │Master 1 │     │Master 2 │     │Master 3 │          │
│  │Slots    │     │Slots    │     │Slots    │          │
│  │0-5460   │     │5461-10922│    │10923-16383│         │
│  └────┬────┘     └────┬────┘     └────┬────┘          │
│       │               │               │                │
│       │               │               │                │
│  ┌────┴────┐     ┌────┴────┐     ┌────┴────┐          │
│  │Replica 1│     │Replica 2│     │Replica 3│          │
│  └─────────┘     └─────────┘     └─────────┘          │
│                                                         │
└─────────────────────────────────────────────────────────┘

- 16384 hash slots distributed across masters
- Key → hash slot → master
- Automatic failover (replica becomes master)
```

---

## ⚠️ Cache Problems & Solutions

### 1. Cache Stampede (Thundering Herd)

```
Problem:
Cache expires → 1000 requests hit database simultaneously

Solutions:
1. Lock (only one fetches)
2. Staggered TTL (random expiration)
3. Background refresh (refresh before expiry)
```

```javascript
// Staggered TTL
const baseTTL = 3600;
const jitter = Math.random() * 600; // 0-10 minutes
await cache.set(key, data, 'EX', baseTTL + jitter);
```

### 2. Cache Penetration

```
Problem:
Requests for non-existent data bypass cache every time
Example: Querying user:999999 that doesn't exist

Solutions:
1. Cache negative results (with shorter TTL)
2. Bloom filter (probabilistic check)
```

```javascript
async function getUser(id) {
  const cached = await cache.get(`user:${id}`);
  
  if (cached === 'NULL') {
    return null; // Cached negative result
  }
  if (cached) {
    return JSON.parse(cached);
  }
  
  const user = await db.users.findById(id);
  
  if (user) {
    await cache.set(`user:${id}`, JSON.stringify(user), 'EX', 3600);
  } else {
    // Cache the fact that it doesn't exist
    await cache.set(`user:${id}`, 'NULL', 'EX', 300);
  }
  
  return user;
}
```

### 3. Cache Avalanche

```
Problem:
Many keys expire simultaneously → massive DB load

Solutions:
1. Staggered TTL
2. Never expire (background refresh)
3. Multi-level caching
```

### 4. Hot Key Problem

```
Problem:
Single key gets millions of requests
Example: Celebrity's profile

Solutions:
1. Local cache (in-memory) + distributed cache
2. Replicate hot keys across Redis instances
3. Break into multiple keys with random suffix
```

```javascript
// Hot key with local cache
const localCache = new Map();

async function getHotData(key) {
  // Check local cache first
  if (localCache.has(key)) {
    return localCache.get(key);
  }
  
  // Check Redis
  const data = await redis.get(key);
  
  // Store locally (short TTL)
  localCache.set(key, data);
  setTimeout(() => localCache.delete(key), 1000);
  
  return data;
}
```

---

## 📖 Further Reading

- "Redis in Action"
- Cloudflare Learning Center
- AWS Caching Best Practices

---

**Next:** [Chapter 05: Message Queues & Events →](./05-message-queues.md)


