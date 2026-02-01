# ⚡ Caching Strategies Complete Guide

> A comprehensive guide to caching at every layer - from browser to database, with real-world patterns and interview preparation.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Caching stores frequently accessed data in faster storage to reduce latency and backend load. The hard part isn't caching - it's knowing when to invalidate."

### The 5 Key Concepts (Remember These!)
```
1. CACHE LAYERS      → Browser → CDN → App (Redis) → Database
2. CACHE-ASIDE       → App checks cache, misses go to DB, then cache
3. TTL + EVENTS      → Use TTL as safety, events for real-time invalidation
4. CACHE STAMPEDE    → Many requests hit DB when cache expires - use locking!
5. HIT RATE          → Target >90%, alert if drops
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Cache-aside"** | "We use cache-aside pattern with Redis" |
| **"Stale-while-revalidate"** | "HTTP uses stale-while-revalidate for fast + fresh" |
| **"Cache stampede"** | "We prevent cache stampede with locking" |
| **"Write-through"** | "Critical data uses write-through for consistency" |
| **"Cache penetration"** | "We cache null results to prevent cache penetration" |
| **"Backpressure"** | "Under load, we slow responses instead of crashing" |
| **"TTL"** | "TTL is 5 minutes with event-driven invalidation" |

### Key Numbers to Remember
| Metric | Target | Alert If |
|--------|--------|----------|
| Hit rate | **>90%** | <80% |
| Latency (Redis) | **<5ms** | >10ms |
| Memory usage | **<80%** | >80% |
| TTL (common) | **5-60 min** | - |

### HTTP Cache Headers (Memorize These!)
```
max-age=3600           → Cache 1 hour
no-cache               → Always revalidate (can still cache)
no-store               → Never cache (sensitive data)
private                → Browser only, not CDN
s-maxage=60            → CDN caches 60s
stale-while-revalidate → Serve stale, refresh background
immutable              → Never changes (versioned assets)
```

### The "Wow" Statement (Memorize This!)
> "Caching happens at multiple layers - browser, CDN, application, database - each with different tradeoffs. Phil Karlton said the two hard things in CS are cache invalidation and naming things. I agree. The key is combining TTL as a safety net with event-driven invalidation for consistency."

### Quick Architecture Drawing (Draw This!)
```
Browser Cache (1ms)
      ↓
CDN Edge Cache (20ms)
      ↓
Redis/App Cache (2-5ms)
      ↓
Database (50-100ms)
```

### Interview Rapid Fire (Practice These!)

**Q: "What is caching?"**
> "Storing data in faster storage. Multiple layers: browser, CDN, Redis, database."

**Q: "Redis vs Memcached?"**
> "Redis: data structures, persistence, pub/sub. Memcached: simpler, more memory efficient."

**Q: "How do you invalidate cache?"**
> "TTL as safety net + event-driven invalidation on data changes."

**Q: "What is cache stampede?"**
> "When cache expires, many requests hit DB simultaneously. Solve with locking."

**Q: "What's the hardest part?"**
> "Cache invalidation - knowing when data is stale."

### Caching Patterns (Memorize This!)
| Pattern | How It Works | Use When |
|---------|--------------|----------|
| **Cache-aside** | App manages cache | Most cases |
| **Read-through** | Cache fetches from DB | Simpler code |
| **Write-through** | Write cache + DB together | Need consistency |
| **Write-behind** | Write cache, DB async | Need speed |

---

## 🎯 How to Explain Like a Senior Developer

### The Perfect Answer: "What is Caching?"

**❌ Junior Answer:**
> "Caching is storing data in memory so you don't have to fetch it again."

**✅ Senior Answer:**
> "Caching is a strategy to reduce latency and load by storing computed results or frequently accessed data closer to where it's needed. 
>
> The key insight is that caching happens at multiple layers - browser, CDN, application, and database - and each layer has different tradeoffs between freshness, consistency, and performance.
>
> The hardest part isn't implementing a cache - it's cache invalidation. Knowing when data is stale and needs refreshing is what Phil Karlton meant when he said 'the two hard things in computer science are cache invalidation and naming things.'
>
> I typically evaluate caching needs by asking: What's the read/write ratio? How stale can data be? What's the cost of a cache miss? That determines whether we need Redis, a CDN, or just HTTP cache headers."

---

### Follow-up Questions They WILL Ask

| They Ask | Your Answer Should Cover |
|----------|--------------------------|
| "How do you invalidate cache?" | TTL, event-driven, cache-aside, write-through |
| "Redis vs Memcached?" | Data structures, persistence, clustering |
| "What's cache stampede?" | Thundering herd, locking, probabilistic early expiry |
| "CDN caching?" | Edge locations, cache headers, purging |
| "Browser caching?" | Cache-Control, ETag, Service Worker |
| "What can go wrong?" | Stale data, memory pressure, cold start |

---

### Power Phrases That Show Expertise

| Phrase | What It Shows |
|--------|---------------|
| "Cache-aside pattern" | You know patterns |
| "Write-through vs write-behind" | You understand consistency |
| "Cache stampede / thundering herd" | You know failure modes |
| "Stale-while-revalidate" | You know modern strategies |
| "Cache warming" | You think about cold starts |
| "TTL vs event-driven invalidation" | You understand tradeoffs |
| "Read-through cache" | You know advanced patterns |
| "Cache hit ratio" | You measure effectiveness |

---

## Table of Contents

1. [Why Caching Matters](#why-caching-matters)
2. [Caching Layers](#caching-layers)
3. [Browser Caching](#browser-caching)
4. [CDN Caching](#cdn-caching)
5. [Application Caching (Redis)](#application-caching-redis)
6. [Database Caching](#database-caching)
7. [Caching Patterns](#caching-patterns)
8. [Cache Invalidation](#cache-invalidation)
9. [Common Problems & Solutions](#common-problems--solutions)
10. [Implementation Examples](#implementation-examples)
11. [Monitoring & Metrics](#monitoring--metrics)
12. [Interview Questions](#interview-questions)

---

## Why Caching Matters

### The Performance Impact

```
Without Caching:
User → Server → Database → Server → User
        └──────── 200ms ────────┘

With Caching:
User → Cache → User
        └─ 2ms ─┘

= 100x faster!
```

### When to Cache

| Cache When | Don't Cache When |
|------------|------------------|
| Read-heavy workloads | Write-heavy workloads |
| Data changes infrequently | Data changes constantly |
| Same data requested often | Every request is unique |
| Expensive computations | Cheap computations |
| External API calls | Real-time requirements |

### The Cost of NOT Caching

```
Example: E-commerce product page

Without cache:
- 10,000 users view same product
- 10,000 database queries
- Database overloaded

With cache:
- 10,000 users view same product  
- 1 database query + 9,999 cache hits
- Database relaxed, users happy
```

---

## Caching Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                      CACHING LAYERS                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────────┐                                              │
│   │   Browser    │  HTTP Cache, Service Worker, localStorage    │
│   │    Cache     │  ← Fastest, closest to user                  │
│   └──────┬───────┘                                              │
│          │                                                       │
│          ▼                                                       │
│   ┌──────────────┐                                              │
│   │     CDN      │  Edge servers, static assets                 │
│   │    Cache     │  ← Geographic distribution                   │
│   └──────┬───────┘                                              │
│          │                                                       │
│          ▼                                                       │
│   ┌──────────────┐                                              │
│   │ Application  │  Redis, Memcached, in-memory                 │
│   │    Cache     │  ← Most flexible                             │
│   └──────┬───────┘                                              │
│          │                                                       │
│          ▼                                                       │
│   ┌──────────────┐                                              │
│   │  Database    │  Query cache, materialized views             │
│   │    Cache     │  ← Closest to data                           │
│   └──────────────┘                                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Layer Comparison

| Layer | Speed | Capacity | Use Case |
|-------|-------|----------|----------|
| **Browser** | ~1ms | Limited | User-specific, static assets |
| **CDN** | ~20ms | Large | Static content, global distribution |
| **Application** | ~2-5ms | Medium | Session, computed data, API responses |
| **Database** | ~10ms | Small | Query results, frequent queries |

---

## Browser Caching

### HTTP Cache Headers

```
┌─────────────────────────────────────────────────────────────────┐
│                    HTTP CACHE HEADERS                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Cache-Control: max-age=3600                                   │
│   └── Cache for 1 hour, then revalidate                         │
│                                                                  │
│   Cache-Control: no-cache                                       │
│   └── Always revalidate with server (can still cache)          │
│                                                                  │
│   Cache-Control: no-store                                       │
│   └── Never cache (sensitive data)                              │
│                                                                  │
│   Cache-Control: private                                        │
│   └── Only browser can cache (not CDN)                          │
│                                                                  │
│   Cache-Control: public, max-age=31536000, immutable            │
│   └── Cache forever (for versioned assets)                      │
│                                                                  │
│   ETag: "abc123"                                                │
│   └── Fingerprint for conditional requests                      │
│                                                                  │
│   Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT                  │
│   └── Timestamp for conditional requests                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Cache-Control Strategies

```javascript
// Express.js examples

// Static assets with versioned URLs (cache forever)
app.use('/static', express.static('public', {
  maxAge: '1y',
  immutable: true
}));
// Files: /static/app.a1b2c3.js

// API responses (cache briefly, revalidate)
app.get('/api/products', (req, res) => {
  res.set({
    'Cache-Control': 'public, max-age=60, stale-while-revalidate=300',
    'ETag': generateETag(products)
  });
  res.json(products);
});

// User-specific data (private, short cache)
app.get('/api/me', (req, res) => {
  res.set('Cache-Control', 'private, max-age=0, must-revalidate');
  res.json(req.user);
});

// Sensitive data (never cache)
app.get('/api/secrets', (req, res) => {
  res.set('Cache-Control', 'no-store');
  res.json(secrets);
});
```

### Conditional Requests (ETag)

```
First Request:
GET /api/product/123
→ 200 OK
   ETag: "abc123"
   Body: { product data }

Second Request:
GET /api/product/123
If-None-Match: "abc123"
→ 304 Not Modified (no body, save bandwidth)

After Update:
GET /api/product/123  
If-None-Match: "abc123"
→ 200 OK
   ETag: "def456"
   Body: { updated product data }
```

```javascript
// ETag implementation
const crypto = require('crypto');

function generateETag(data) {
  return crypto
    .createHash('md5')
    .update(JSON.stringify(data))
    .digest('hex');
}

app.get('/api/product/:id', async (req, res) => {
  const product = await db.getProduct(req.params.id);
  const etag = generateETag(product);
  
  // Check if client has current version
  if (req.headers['if-none-match'] === etag) {
    return res.status(304).end();
  }
  
  res.set('ETag', etag);
  res.json(product);
});
```

### Service Worker Caching

```javascript
// sw.js - Different strategies

// Cache First (offline-first)
async function cacheFirst(request) {
  const cached = await caches.match(request);
  if (cached) return cached;
  
  const response = await fetch(request);
  const cache = await caches.open('v1');
  cache.put(request, response.clone());
  return response;
}

// Network First (fresh-first)  
async function networkFirst(request) {
  try {
    const response = await fetch(request);
    const cache = await caches.open('v1');
    cache.put(request, response.clone());
    return response;
  } catch {
    return caches.match(request);
  }
}

// Stale While Revalidate (fast + fresh)
async function staleWhileRevalidate(request) {
  const cached = await caches.match(request);
  
  const fetchPromise = fetch(request).then(response => {
    caches.open('v1').then(cache => {
      cache.put(request, response.clone());
    });
    return response;
  });
  
  return cached || fetchPromise;
}

// Route to strategy
self.addEventListener('fetch', event => {
  const url = new URL(event.request.url);
  
  if (url.pathname.startsWith('/api/')) {
    event.respondWith(networkFirst(event.request));
  } else if (url.pathname.startsWith('/static/')) {
    event.respondWith(cacheFirst(event.request));
  } else {
    event.respondWith(staleWhileRevalidate(event.request));
  }
});
```

---

## CDN Caching

### How CDN Works

```
┌─────────────────────────────────────────────────────────────────┐
│                        CDN ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   User (NYC) ──────┐                                            │
│                    ├──▶ Edge (NYC) ─── Cache HIT ──▶ Response   │
│   User (NYC) ──────┘         │                                  │
│                              │ Cache MISS                       │
│                              ▼                                  │
│   User (London) ──▶ Edge (London) ──▶ Origin Server            │
│                              │              │                   │
│                              └──── Cache ◀──┘                   │
│                                                                  │
│   Benefits:                                                     │
│   • Reduced latency (serve from nearby edge)                    │
│   • Reduced origin load                                         │
│   • DDoS protection                                             │
│   • Automatic failover                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### CDN Cache Headers

```javascript
// Next.js API route with CDN caching
export default function handler(req, res) {
  // Cache on CDN for 60s, allow stale for 60s while revalidating
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=60, stale-while-revalidate=60'
  );
  
  res.json({ data: 'cached on CDN' });
}

// Vercel/Next.js specific
export const config = {
  runtime: 'edge', // Run at edge for faster response
};
```

### Cache Purging

```javascript
// Cloudflare cache purge
async function purgeCloudflareCache(urls) {
  await fetch(
    `https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/purge_cache`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${API_TOKEN}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ files: urls })
    }
  );
}

// Purge when content changes
async function updateProduct(id, data) {
  await db.updateProduct(id, data);
  
  // Purge CDN cache
  await purgeCloudflareCache([
    `https://mysite.com/api/products/${id}`,
    `https://mysite.com/products/${id}`
  ]);
}
```

### CDN Best Practices

```javascript
// 1. Version your assets
// ❌ Bad: /js/app.js (can't cache long)
// ✅ Good: /js/app.a1b2c3.js (cache forever)

// 2. Set appropriate TTLs
const cacheRules = {
  // Static assets - cache forever (versioned)
  '*.js': 'public, max-age=31536000, immutable',
  '*.css': 'public, max-age=31536000, immutable',
  '*.png': 'public, max-age=31536000, immutable',
  
  // HTML - short cache, revalidate
  '*.html': 'public, max-age=0, must-revalidate',
  
  // API - varies by endpoint
  '/api/products': 'public, s-maxage=300',      // 5 min on CDN
  '/api/user': 'private, no-store',              // Never cache
};

// 3. Use Vary header for dynamic content
app.get('/api/data', (req, res) => {
  res.set('Vary', 'Accept-Encoding, Accept-Language');
  res.json(data);
});
```

---

## Application Caching (Redis)

### Why Redis?

| Feature | Redis | Memcached |
|---------|-------|-----------|
| **Data Structures** | Strings, Lists, Sets, Hashes, Sorted Sets | Strings only |
| **Persistence** | Yes (RDB, AOF) | No |
| **Pub/Sub** | Yes | No |
| **Clustering** | Yes | Yes |
| **Lua Scripting** | Yes | No |
| **Memory Efficiency** | Good | Better |

**Use Redis when:** Need data structures, persistence, pub/sub
**Use Memcached when:** Simple key-value, maximum memory efficiency

### Redis Connection

```javascript
// redis-client.js
const Redis = require('ioredis');

// Single instance
const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: 6379,
  password: process.env.REDIS_PASSWORD,
  retryDelayOnFailover: 100,
  maxRetriesPerRequest: 3
});

// Cluster
const cluster = new Redis.Cluster([
  { host: 'node1', port: 6379 },
  { host: 'node2', port: 6379 },
  { host: 'node3', port: 6379 }
]);

// Connection events
redis.on('connect', () => console.log('Redis connected'));
redis.on('error', (err) => console.error('Redis error:', err));

module.exports = redis;
```

### Basic Operations

```javascript
const redis = require('./redis-client');

// String operations
await redis.set('user:123', JSON.stringify(user));
await redis.set('user:123', JSON.stringify(user), 'EX', 3600); // 1 hour TTL

const user = JSON.parse(await redis.get('user:123'));

// Check existence
const exists = await redis.exists('user:123');

// Delete
await redis.del('user:123');

// Set multiple
await redis.mset('key1', 'val1', 'key2', 'val2');

// Get multiple
const values = await redis.mget('key1', 'key2');

// Increment (atomic)
await redis.incr('page:views');
await redis.incrby('page:views', 10);
```

### Redis Data Structures

```javascript
// HASH - Perfect for objects
await redis.hset('user:123', {
  name: 'John',
  email: 'john@example.com',
  age: 30
});
const user = await redis.hgetall('user:123');
const name = await redis.hget('user:123', 'name');

// LIST - For queues, recent items
await redis.lpush('recent:products', productId);  // Add to front
await redis.ltrim('recent:products', 0, 9);       // Keep only 10
const recent = await redis.lrange('recent:products', 0, 9);

// SET - For unique items, tags
await redis.sadd('product:123:tags', 'electronics', 'sale');
const tags = await redis.smembers('product:123:tags');
const isMember = await redis.sismember('product:123:tags', 'sale');

// SORTED SET - For leaderboards, rankings
await redis.zadd('leaderboard', 100, 'player1', 200, 'player2');
const top10 = await redis.zrevrange('leaderboard', 0, 9, 'WITHSCORES');

// Increment score
await redis.zincrby('leaderboard', 50, 'player1');
```

### Cache Service Implementation

```javascript
// cache.service.js
class CacheService {
  constructor(redis) {
    this.redis = redis;
    this.defaultTTL = 3600; // 1 hour
  }
  
  // Basic get/set with JSON serialization
  async get(key) {
    const value = await this.redis.get(key);
    return value ? JSON.parse(value) : null;
  }
  
  async set(key, value, ttl = this.defaultTTL) {
    await this.redis.set(key, JSON.stringify(value), 'EX', ttl);
  }
  
  async delete(key) {
    await this.redis.del(key);
  }
  
  // Pattern-based deletion
  async deletePattern(pattern) {
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
  
  // Get or compute (cache-aside pattern)
  async getOrSet(key, computeFn, ttl = this.defaultTTL) {
    const cached = await this.get(key);
    if (cached !== null) {
      return cached;
    }
    
    const value = await computeFn();
    await this.set(key, value, ttl);
    return value;
  }
  
  // Memoize function results
  memoize(fn, keyFn, ttl = this.defaultTTL) {
    return async (...args) => {
      const key = keyFn(...args);
      return this.getOrSet(key, () => fn(...args), ttl);
    };
  }
}

module.exports = new CacheService(redis);
```

### Usage Examples

```javascript
const cache = require('./cache.service');

// Simple caching
async function getProduct(id) {
  return cache.getOrSet(
    `product:${id}`,
    () => db.products.findById(id),
    3600 // 1 hour
  );
}

// Memoized function
const getExpensiveData = cache.memoize(
  async (userId, date) => {
    // Expensive computation
    return await computeAnalytics(userId, date);
  },
  (userId, date) => `analytics:${userId}:${date}`,
  86400 // 24 hours
);

// Invalidation on update
async function updateProduct(id, data) {
  await db.products.update(id, data);
  await cache.delete(`product:${id}`);
  await cache.deletePattern(`products:list:*`); // Clear list caches
}
```

---

## Database Caching

### Query Result Caching

```javascript
// Cache expensive queries
class ProductRepository {
  constructor(db, cache) {
    this.db = db;
    this.cache = cache;
  }
  
  async findById(id) {
    const cacheKey = `product:${id}`;
    
    // Try cache first
    const cached = await this.cache.get(cacheKey);
    if (cached) return cached;
    
    // Query database
    const product = await this.db.query(
      'SELECT * FROM products WHERE id = ?',
      [id]
    );
    
    // Cache result
    await this.cache.set(cacheKey, product, 3600);
    
    return product;
  }
  
  async findByCategory(category, page = 1, limit = 20) {
    const cacheKey = `products:category:${category}:${page}:${limit}`;
    
    return this.cache.getOrSet(cacheKey, async () => {
      return this.db.query(
        `SELECT * FROM products 
         WHERE category = ? 
         LIMIT ? OFFSET ?`,
        [category, limit, (page - 1) * limit]
      );
    }, 300); // 5 minutes
  }
  
  async update(id, data) {
    await this.db.query(
      'UPDATE products SET ? WHERE id = ?',
      [data, id]
    );
    
    // Invalidate related caches
    await this.cache.delete(`product:${id}`);
    await this.cache.deletePattern('products:category:*');
    await this.cache.deletePattern('products:list:*');
  }
}
```

### Materialized Views

```sql
-- PostgreSQL Materialized View
CREATE MATERIALIZED VIEW product_stats AS
SELECT 
  category,
  COUNT(*) as product_count,
  AVG(price) as avg_price,
  SUM(stock) as total_stock
FROM products
GROUP BY category;

-- Refresh periodically
REFRESH MATERIALIZED VIEW product_stats;

-- Refresh concurrently (no lock)
REFRESH MATERIALIZED VIEW CONCURRENTLY product_stats;
```

```javascript
// Auto-refresh materialized view
const cron = require('node-cron');

// Refresh every hour
cron.schedule('0 * * * *', async () => {
  await db.query('REFRESH MATERIALIZED VIEW CONCURRENTLY product_stats');
  console.log('Materialized view refreshed');
});
```

### Database Query Cache (MySQL)

```sql
-- Enable query cache (MySQL 5.7 and earlier)
SET GLOBAL query_cache_type = ON;
SET GLOBAL query_cache_size = 268435456; -- 256MB

-- Check cache status
SHOW STATUS LIKE 'Qcache%';

-- Hint to use/skip cache
SELECT SQL_CACHE * FROM products WHERE category = 'electronics';
SELECT SQL_NO_CACHE * FROM products WHERE id = 123;
```

> **Note:** MySQL 8.0 removed query cache. Use application-level caching instead.

---

## Caching Patterns

### 1. Cache-Aside (Lazy Loading)

Application manages cache explicitly.

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│   App    │────▶│  Cache   │     │    DB    │
│          │◀────│          │     │          │
└──────────┘     └──────────┘     └──────────┘
     │                                  ▲
     │         Cache Miss               │
     └──────────────────────────────────┘
```

```javascript
async function getUser(id) {
  // 1. Check cache
  const cached = await cache.get(`user:${id}`);
  if (cached) return cached;
  
  // 2. Cache miss - query DB
  const user = await db.users.findById(id);
  
  // 3. Populate cache
  await cache.set(`user:${id}`, user, 3600);
  
  return user;
}
```

**Pros:** Only cache what's needed, simple
**Cons:** Cache miss = slow, potential stampede

---

### 2. Read-Through

Cache handles DB reads automatically.

```javascript
class ReadThroughCache {
  constructor(cache, loader) {
    this.cache = cache;
    this.loader = loader; // Function to load from DB
  }
  
  async get(key) {
    let value = await this.cache.get(key);
    
    if (value === null) {
      value = await this.loader(key);
      await this.cache.set(key, value);
    }
    
    return value;
  }
}

// Usage
const userCache = new ReadThroughCache(
  redis,
  (key) => db.users.findById(key.replace('user:', ''))
);

const user = await userCache.get('user:123');
```

---

### 3. Write-Through

Write to cache and DB synchronously.

```
Write Request
     │
     ▼
┌──────────┐     ┌──────────┐
│  Cache   │────▶│    DB    │
│ (write)  │     │ (write)  │
└──────────┘     └──────────┘
```

```javascript
async function updateUser(id, data) {
  // Write to DB first
  await db.users.update(id, data);
  
  // Then update cache
  const user = await db.users.findById(id);
  await cache.set(`user:${id}`, user);
  
  return user;
}
```

**Pros:** Cache always consistent
**Cons:** Write latency increased

---

### 4. Write-Behind (Write-Back)

Write to cache immediately, DB asynchronously.

```
Write Request
     │
     ▼
┌──────────┐     ┌──────────┐
│  Cache   │ ─ ─▶│    DB    │
│ (write)  │async│ (write)  │
└──────────┘     └──────────┘
```

```javascript
class WriteBehindCache {
  constructor(cache, db) {
    this.cache = cache;
    this.db = db;
    this.writeQueue = [];
    this.flushInterval = setInterval(() => this.flush(), 1000);
  }
  
  async set(key, value) {
    // Write to cache immediately
    await this.cache.set(key, value);
    
    // Queue for DB write
    this.writeQueue.push({ key, value, timestamp: Date.now() });
  }
  
  async flush() {
    if (this.writeQueue.length === 0) return;
    
    const batch = this.writeQueue.splice(0, 100);
    
    for (const { key, value } of batch) {
      try {
        await this.db.set(key, value);
      } catch (error) {
        console.error('Write-behind failed:', key, error);
        // Re-queue or send to dead letter
      }
    }
  }
}
```

**Pros:** Fast writes
**Cons:** Data loss risk if cache fails before flush

---

### 5. Refresh-Ahead

Proactively refresh before expiry.

```javascript
class RefreshAheadCache {
  constructor(cache, loader, ttl = 3600, refreshThreshold = 0.8) {
    this.cache = cache;
    this.loader = loader;
    this.ttl = ttl;
    this.refreshThreshold = refreshThreshold; // Refresh at 80% of TTL
  }
  
  async get(key) {
    const item = await this.cache.getWithTTL(key);
    
    if (!item) {
      // Cache miss
      return this.loadAndCache(key);
    }
    
    const { value, remainingTTL } = item;
    const threshold = this.ttl * (1 - this.refreshThreshold);
    
    // Refresh in background if close to expiry
    if (remainingTTL < threshold) {
      this.loadAndCache(key); // Don't await
    }
    
    return value;
  }
  
  async loadAndCache(key) {
    const value = await this.loader(key);
    await this.cache.set(key, value, this.ttl);
    return value;
  }
}
```

---

## Cache Invalidation

### Strategies Comparison

| Strategy | Consistency | Complexity | Use Case |
|----------|-------------|------------|----------|
| **TTL** | Eventually | Low | General purpose |
| **Event-driven** | Strong | Medium | Real-time needs |
| **Write-through** | Strong | Medium | Critical data |
| **Version/Tag** | Strong | Medium | Grouped invalidation |

### TTL-Based Invalidation

```javascript
// Simple TTL
await cache.set('product:123', product, 300); // 5 minutes

// Sliding expiration
async function getWithSliding(key, ttl) {
  const value = await cache.get(key);
  if (value) {
    await cache.expire(key, ttl); // Reset TTL on access
  }
  return value;
}
```

### Event-Driven Invalidation

```javascript
// Event emitter for cache invalidation
const events = require('events');
const cacheEvents = new events.EventEmitter();

// Subscribe to invalidation events
cacheEvents.on('product:updated', async ({ id }) => {
  await cache.delete(`product:${id}`);
  await cache.deletePattern(`products:list:*`);
});

cacheEvents.on('product:deleted', async ({ id }) => {
  await cache.delete(`product:${id}`);
  await cache.deletePattern(`products:*`);
});

// Emit on changes
async function updateProduct(id, data) {
  await db.products.update(id, data);
  cacheEvents.emit('product:updated', { id });
}

// With message queue (distributed)
async function handleProductUpdate(message) {
  const { productId } = message;
  await cache.delete(`product:${productId}`);
}

messageQueue.subscribe('product.updated', handleProductUpdate);
```

### Tag-Based Invalidation

```javascript
class TaggedCache {
  constructor(cache) {
    this.cache = cache;
  }
  
  async set(key, value, tags = [], ttl = 3600) {
    // Store value
    await this.cache.set(key, value, ttl);
    
    // Associate key with tags
    for (const tag of tags) {
      await this.cache.sadd(`tag:${tag}`, key);
    }
  }
  
  async invalidateTag(tag) {
    const keys = await this.cache.smembers(`tag:${tag}`);
    
    if (keys.length > 0) {
      await this.cache.del(...keys);
      await this.cache.del(`tag:${tag}`);
    }
  }
}

// Usage
const taggedCache = new TaggedCache(redis);

// Cache with tags
await taggedCache.set(
  'product:123',
  product,
  ['products', 'category:electronics', 'brand:apple'],
  3600
);

// Invalidate all products
await taggedCache.invalidateTag('products');

// Invalidate by category
await taggedCache.invalidateTag('category:electronics');
```

---

## Common Problems & Solutions

### 1. Cache Stampede (Thundering Herd)

**Problem:** When cache expires, many requests hit the database simultaneously.

```
Cache expires at 12:00:00
─────────────────────────────────────────────
12:00:01  Request 1 → Cache MISS → DB Query
12:00:01  Request 2 → Cache MISS → DB Query  
12:00:01  Request 3 → Cache MISS → DB Query
12:00:01  Request 4 → Cache MISS → DB Query
          ... 1000 concurrent DB queries!
─────────────────────────────────────────────
```

**Solutions:**

**A) Locking (Single Flight)**
```javascript
const locks = new Map();

async function getWithLock(key, loader, ttl) {
  // Check cache
  const cached = await cache.get(key);
  if (cached) return cached;
  
  // Check if already loading
  if (locks.has(key)) {
    return locks.get(key);
  }
  
  // Acquire lock and load
  const promise = (async () => {
    try {
      const value = await loader();
      await cache.set(key, value, ttl);
      return value;
    } finally {
      locks.delete(key);
    }
  })();
  
  locks.set(key, promise);
  return promise;
}
```

**B) Probabilistic Early Expiration**
```javascript
async function getWithEarlyExpiry(key, loader, ttl) {
  const item = await cache.getWithMeta(key);
  
  if (!item) {
    return loadAndCache(key, loader, ttl);
  }
  
  const { value, remainingTTL } = item;
  
  // Probabilistically refresh before expiry
  // Higher chance to refresh as we approach expiry
  const expiryRatio = remainingTTL / ttl;
  const refreshProbability = Math.pow(1 - expiryRatio, 2);
  
  if (Math.random() < refreshProbability) {
    // Refresh in background
    loadAndCache(key, loader, ttl);
  }
  
  return value;
}
```

**C) Distributed Lock (Redis)**
```javascript
async function getWithDistributedLock(key, loader, ttl) {
  const cached = await cache.get(key);
  if (cached) return cached;
  
  const lockKey = `lock:${key}`;
  const lockTTL = 10; // 10 seconds
  
  // Try to acquire lock
  const acquired = await redis.set(lockKey, '1', 'NX', 'EX', lockTTL);
  
  if (acquired) {
    try {
      const value = await loader();
      await cache.set(key, value, ttl);
      return value;
    } finally {
      await redis.del(lockKey);
    }
  } else {
    // Wait and retry
    await sleep(100);
    return getWithDistributedLock(key, loader, ttl);
  }
}
```

---

### 2. Cache Penetration

**Problem:** Requests for non-existent data always hit database.

```
GET /user/999999 (doesn't exist)
→ Cache MISS → DB Query → null → No cache → Repeat forever
```

**Solutions:**

**A) Cache Negative Results**
```javascript
async function getUser(id) {
  const cacheKey = `user:${id}`;
  const cached = await cache.get(cacheKey);
  
  // Check for cached "not found"
  if (cached === 'NULL') return null;
  if (cached) return cached;
  
  const user = await db.users.findById(id);
  
  if (user) {
    await cache.set(cacheKey, user, 3600);
  } else {
    // Cache the absence (shorter TTL)
    await cache.set(cacheKey, 'NULL', 300);
  }
  
  return user;
}
```

**B) Bloom Filter**
```javascript
const BloomFilter = require('bloom-filters').BloomFilter;

// Initialize with all valid IDs
const validIds = new BloomFilter(10000, 4);
const allUserIds = await db.users.getAllIds();
allUserIds.forEach(id => validIds.add(id));

async function getUser(id) {
  // Quick check - if not in bloom filter, definitely doesn't exist
  if (!validIds.has(id)) {
    return null;
  }
  
  // Might exist - check cache/DB
  return getUserFromCacheOrDB(id);
}
```

---

### 3. Cache Avalanche

**Problem:** Many cache entries expire at the same time.

```
All caches set at app start with same TTL
→ All expire at same time
→ Massive DB load
```

**Solutions:**

**A) Randomize TTL**
```javascript
function randomizeTTL(baseTTL, variance = 0.1) {
  const min = baseTTL * (1 - variance);
  const max = baseTTL * (1 + variance);
  return Math.floor(Math.random() * (max - min) + min);
}

// Usage
await cache.set('key', value, randomizeTTL(3600)); // 3240-3960 seconds
```

**B) Staggered Warming**
```javascript
async function warmCache(keys) {
  for (const key of keys) {
    await loadAndCache(key);
    await sleep(100); // Stagger loads
  }
}
```

---

### 4. Hot Key Problem

**Problem:** Single key gets too many requests.

```
"trending:products" → 100,000 requests/second
→ Single Redis node overloaded
```

**Solutions:**

**A) Local Cache + Remote Cache**
```javascript
const localCache = new Map();
const LOCAL_TTL = 1000; // 1 second local cache

async function getHotKey(key) {
  // Check local cache first
  const local = localCache.get(key);
  if (local && local.expiry > Date.now()) {
    return local.value;
  }
  
  // Fall back to Redis
  const value = await redis.get(key);
  
  // Cache locally
  localCache.set(key, {
    value,
    expiry: Date.now() + LOCAL_TTL
  });
  
  return value;
}
```

**B) Key Replication**
```javascript
// Replicate hot key across multiple keys
async function setHotKey(key, value, replicas = 10) {
  const promises = [];
  for (let i = 0; i < replicas; i++) {
    promises.push(redis.set(`${key}:${i}`, value));
  }
  await Promise.all(promises);
}

async function getHotKey(key, replicas = 10) {
  const replica = Math.floor(Math.random() * replicas);
  return redis.get(`${key}:${replica}`);
}
```

---

### 5. Stale Data

**Problem:** Cache serves outdated data.

**Solutions:**

**A) Versioning**
```javascript
// Include version in cache key
const version = await redis.get('products:version');
const cacheKey = `products:list:v${version}`;

// Bump version on any product change
async function onProductChange() {
  await redis.incr('products:version');
}
```

**B) Event-Driven Invalidation**
```javascript
// Subscribe to changes
pubsub.subscribe('product:*', async (channel, message) => {
  const productId = channel.split(':')[1];
  await cache.delete(`product:${productId}`);
});
```

---

## Implementation Examples

### Full Caching Layer

```javascript
// cache/index.js
const Redis = require('ioredis');

class CacheLayer {
  constructor(config) {
    this.redis = new Redis(config.redis);
    this.defaultTTL = config.defaultTTL || 3600;
    this.prefix = config.prefix || '';
    this.locks = new Map();
  }
  
  key(name) {
    return this.prefix ? `${this.prefix}:${name}` : name;
  }
  
  // Basic operations
  async get(key) {
    const value = await this.redis.get(this.key(key));
    return value ? JSON.parse(value) : null;
  }
  
  async set(key, value, ttl = this.defaultTTL) {
    await this.redis.set(
      this.key(key),
      JSON.stringify(value),
      'EX',
      ttl
    );
  }
  
  async delete(key) {
    await this.redis.del(this.key(key));
  }
  
  async deletePattern(pattern) {
    const keys = await this.redis.keys(this.key(pattern));
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
  
  // Cache-aside with stampede protection
  async getOrSet(key, loader, options = {}) {
    const { ttl = this.defaultTTL, lock = true } = options;
    
    // Check cache
    const cached = await this.get(key);
    if (cached !== null) return cached;
    
    // With lock (prevent stampede)
    if (lock) {
      return this.withLock(key, async () => {
        // Double-check after acquiring lock
        const rechecked = await this.get(key);
        if (rechecked !== null) return rechecked;
        
        const value = await loader();
        await this.set(key, value, ttl);
        return value;
      });
    }
    
    // Without lock
    const value = await loader();
    await this.set(key, value, ttl);
    return value;
  }
  
  // Lock mechanism
  async withLock(key, fn) {
    const lockKey = `lock:${key}`;
    
    // Check local lock (in-process)
    if (this.locks.has(lockKey)) {
      return this.locks.get(lockKey);
    }
    
    const promise = (async () => {
      // Distributed lock
      const acquired = await this.redis.set(
        this.key(lockKey),
        '1',
        'NX',
        'EX',
        10
      );
      
      if (!acquired) {
        // Wait for other process
        await this.sleep(100);
        return this.get(key);
      }
      
      try {
        return await fn();
      } finally {
        await this.redis.del(this.key(lockKey));
        this.locks.delete(lockKey);
      }
    })();
    
    this.locks.set(lockKey, promise);
    return promise;
  }
  
  // Multi-get with cache-aside
  async getMany(keys, loader) {
    const cacheKeys = keys.map(k => this.key(k));
    const cached = await this.redis.mget(...cacheKeys);
    
    const result = {};
    const missing = [];
    
    keys.forEach((key, i) => {
      if (cached[i]) {
        result[key] = JSON.parse(cached[i]);
      } else {
        missing.push(key);
      }
    });
    
    if (missing.length > 0) {
      const loaded = await loader(missing);
      
      const pipeline = this.redis.pipeline();
      for (const [key, value] of Object.entries(loaded)) {
        result[key] = value;
        pipeline.set(this.key(key), JSON.stringify(value), 'EX', this.defaultTTL);
      }
      await pipeline.exec();
    }
    
    return result;
  }
  
  // Invalidation helpers
  async invalidate(keys) {
    if (!Array.isArray(keys)) keys = [keys];
    const fullKeys = keys.map(k => this.key(k));
    await this.redis.del(...fullKeys);
  }
  
  async invalidatePrefix(prefix) {
    await this.deletePattern(`${prefix}*`);
  }
  
  // Utility
  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  // Health check
  async healthCheck() {
    try {
      await this.redis.ping();
      return { healthy: true };
    } catch (error) {
      return { healthy: false, error: error.message };
    }
  }
}

module.exports = CacheLayer;
```

### Using the Cache Layer

```javascript
const CacheLayer = require('./cache');

const cache = new CacheLayer({
  redis: { host: 'localhost', port: 6379 },
  defaultTTL: 3600,
  prefix: 'myapp'
});

// Product repository with caching
class ProductRepository {
  constructor(db, cache) {
    this.db = db;
    this.cache = cache;
  }
  
  async findById(id) {
    return this.cache.getOrSet(
      `product:${id}`,
      () => this.db.products.findById(id),
      { ttl: 3600 }
    );
  }
  
  async findByCategory(category) {
    return this.cache.getOrSet(
      `products:category:${category}`,
      () => this.db.products.findByCategory(category),
      { ttl: 300 }
    );
  }
  
  async findMany(ids) {
    return this.cache.getMany(
      ids.map(id => `product:${id}`),
      async (missingKeys) => {
        const missingIds = missingKeys.map(k => k.split(':')[1]);
        const products = await this.db.products.findByIds(missingIds);
        return Object.fromEntries(
          products.map(p => [`product:${p.id}`, p])
        );
      }
    );
  }
  
  async update(id, data) {
    const product = await this.db.products.update(id, data);
    
    // Invalidate related caches
    await this.cache.invalidate([
      `product:${id}`,
      `products:category:${product.category}`
    ]);
    await this.cache.invalidatePrefix('products:list');
    
    return product;
  }
  
  async delete(id) {
    const product = await this.db.products.findById(id);
    await this.db.products.delete(id);
    
    await this.cache.invalidate(`product:${id}`);
    await this.cache.invalidatePrefix('products:');
  }
}
```

---

## Monitoring & Metrics

### Key Metrics to Track

```javascript
// Metrics collection
class CacheMetrics {
  constructor() {
    this.hits = 0;
    this.misses = 0;
    this.errors = 0;
    this.latencies = [];
  }
  
  recordHit(latencyMs) {
    this.hits++;
    this.latencies.push(latencyMs);
  }
  
  recordMiss(latencyMs) {
    this.misses++;
    this.latencies.push(latencyMs);
  }
  
  recordError() {
    this.errors++;
  }
  
  getStats() {
    const total = this.hits + this.misses;
    return {
      hitRate: total > 0 ? (this.hits / total) * 100 : 0,
      missRate: total > 0 ? (this.misses / total) * 100 : 0,
      totalRequests: total,
      errors: this.errors,
      avgLatency: this.latencies.length > 0 
        ? this.latencies.reduce((a, b) => a + b) / this.latencies.length 
        : 0,
      p99Latency: this.percentile(99)
    };
  }
  
  percentile(p) {
    if (this.latencies.length === 0) return 0;
    const sorted = [...this.latencies].sort((a, b) => a - b);
    const index = Math.ceil((p / 100) * sorted.length) - 1;
    return sorted[index];
  }
}
```

### Redis Metrics

```javascript
// Monitor Redis
async function getRedisMetrics(redis) {
  const info = await redis.info();
  
  // Parse INFO output
  const metrics = {};
  info.split('\n').forEach(line => {
    const [key, value] = line.split(':');
    if (key && value) {
      metrics[key.trim()] = value.trim();
    }
  });
  
  return {
    // Memory
    usedMemory: parseInt(metrics.used_memory),
    usedMemoryHuman: metrics.used_memory_human,
    maxMemory: parseInt(metrics.maxmemory),
    memoryFragmentation: parseFloat(metrics.mem_fragmentation_ratio),
    
    // Stats
    totalConnections: parseInt(metrics.total_connections_received),
    connectedClients: parseInt(metrics.connected_clients),
    commandsProcessed: parseInt(metrics.total_commands_processed),
    
    // Keyspace
    keys: parseInt(metrics.db0?.split(',')[0]?.split('=')[1] || 0),
    
    // Hit rate (if available)
    keyspaceHits: parseInt(metrics.keyspace_hits),
    keyspaceMisses: parseInt(metrics.keyspace_misses),
    hitRate: metrics.keyspace_hits && metrics.keyspace_misses
      ? (parseInt(metrics.keyspace_hits) / 
         (parseInt(metrics.keyspace_hits) + parseInt(metrics.keyspace_misses))) * 100
      : null
  };
}
```

### Dashboard Queries (Prometheus/Grafana)

```yaml
# prometheus.yml scrape config
- job_name: 'redis'
  static_configs:
    - targets: ['redis-exporter:9121']

# Grafana queries
# Hit rate
rate(redis_keyspace_hits_total[5m]) / 
(rate(redis_keyspace_hits_total[5m]) + rate(redis_keyspace_misses_total[5m])) * 100

# Memory usage percentage
redis_memory_used_bytes / redis_memory_max_bytes * 100

# Commands per second
rate(redis_commands_processed_total[1m])

# Connected clients
redis_connected_clients
```

---

## Interview Questions & Answers

### Basic Level

#### Q1: What is caching and why do we use it?

**Answer:**
"Caching is storing frequently accessed data in a faster storage layer to reduce latency and backend load.

We use caching because:
1. **Speed** - Memory is 100x faster than disk, local is faster than network
2. **Reduced load** - Fewer database queries, less computation
3. **Cost** - Cheaper to serve from cache than to compute/query repeatedly
4. **Scalability** - Can handle more traffic without scaling backend

The tradeoff is data freshness - cached data might be stale. That's why choosing the right TTL and invalidation strategy is crucial."

---

#### Q2: What's the difference between Redis and Memcached?

**Answer:**

| Aspect | Redis | Memcached |
|--------|-------|-----------|
| **Data types** | Strings, Lists, Sets, Hashes, Sorted Sets | Strings only |
| **Persistence** | RDB snapshots, AOF | None |
| **Replication** | Built-in | External |
| **Pub/Sub** | Yes | No |
| **Lua scripting** | Yes | No |
| **Memory efficiency** | Good | Better |
| **Multi-threaded** | Single (multi in 6.0+) | Yes |

"I'd choose **Redis** when I need data structures, persistence, or pub/sub. I'd choose **Memcached** for simple key-value caching with maximum memory efficiency."

---

#### Q3: Explain cache-aside pattern.

**Answer:**
"Cache-aside, also called lazy loading, is where the application explicitly manages the cache.

The flow is:
1. Check cache for data
2. If cache hit, return cached data
3. If cache miss, query database
4. Store result in cache
5. Return data

```javascript
async function getUser(id) {
  let user = await cache.get(`user:${id}`);
  if (!user) {
    user = await db.findUser(id);
    await cache.set(`user:${id}`, user, 3600);
  }
  return user;
}
```

**Pros:** Only caches what's needed, simple to understand
**Cons:** First request is slow, potential cache stampede on popular keys"

---

### Intermediate Level

#### Q4: What is cache stampede and how do you prevent it?

**Answer:**
"Cache stampede, or thundering herd, happens when a popular cache entry expires and many concurrent requests all hit the database simultaneously.

**Prevention strategies:**

1. **Locking** - Only one request fetches from DB, others wait
```javascript
if (!cache.has(key)) {
  await acquireLock(key);
  // Double-check after lock
  if (!cache.has(key)) {
    const value = await db.fetch();
    await cache.set(key, value);
  }
  await releaseLock(key);
}
```

2. **Probabilistic early expiration** - Randomly refresh before actual expiry
3. **Background refresh** - Refresh cache before it expires
4. **Never expire** - Use event-driven invalidation instead of TTL

In production, I typically use locking for critical paths and probabilistic early expiration for high-traffic endpoints."

---

#### Q5: How do you handle cache invalidation?

**Answer:**
"Cache invalidation is the hardest part of caching. I use different strategies based on requirements:

1. **TTL-based** - Set expiration time
   - Simple but data can be stale until TTL
   - Good for data where staleness is acceptable

2. **Event-driven** - Invalidate on data changes
   - More complex but ensures consistency
   - Use pub/sub or message queues

3. **Write-through** - Update cache on every write
   - Always consistent but slower writes

4. **Version/Tag-based** - Include version in key
   - Bump version to invalidate all related caches

For most applications, I combine TTL (as a safety net) with event-driven invalidation (for consistency).

```javascript
// Event-driven invalidation
async function updateProduct(id, data) {
  await db.update(id, data);
  await cache.delete(`product:${id}`);
  await eventBus.publish('product.updated', { id });
}
```"

---

#### Q6: Explain write-through vs write-behind caching.

**Answer:**

"**Write-through:**
- Write to cache AND database synchronously
- Data is always consistent
- Higher write latency
- Use when: consistency is critical

```javascript
async function writeThrough(key, value) {
  await db.save(key, value);     // DB first
  await cache.set(key, value);   // Then cache
}
```

**Write-behind (write-back):**
- Write to cache immediately, database asynchronously
- Lower write latency
- Risk of data loss if cache fails
- Use when: write performance matters more than durability

```javascript
async function writeBehind(key, value) {
  await cache.set(key, value);   // Cache immediately
  writeQueue.push({ key, value }); // Async DB write
}
```

I prefer write-through for critical data and write-behind for analytics or logs where losing some data is acceptable."

---

### Advanced Level

#### Q7: How would you design a caching strategy for a high-traffic e-commerce site?

**Answer:**
"I'd implement multi-layer caching:

**1. Browser/CDN Layer:**
- Static assets: `Cache-Control: public, max-age=31536000, immutable`
- Product images: CDN with 24-hour TTL
- HTML pages: Short TTL (60s) + stale-while-revalidate

**2. Application Layer (Redis):**
- Product catalog: Cache with 5-minute TTL
- User sessions: Redis with sliding expiration
- Shopping cart: Redis hash per user
- Inventory counts: Short TTL (30s) to balance freshness

**3. Database Layer:**
- Connection pooling
- Materialized views for complex queries
- Read replicas for read-heavy workloads

**Key considerations:**
- Hot products get local in-memory cache (1s TTL) to reduce Redis load
- Cache stampede protection with locking
- Event-driven invalidation when products update
- Separate cache clusters for different data types

**Invalidation strategy:**
```javascript
// Product update triggers
async function onProductUpdate(productId) {
  await cache.delete(`product:${productId}`);
  await cache.deletePattern(`products:category:*`);
  await cdn.purge(`/products/${productId}`);
}
```"

---

#### Q8: How do you handle hot keys in Redis?

**Answer:**
"Hot keys are keys with extremely high access rates that can overwhelm a single Redis node.

**Solutions:**

1. **Local caching** - Add in-memory cache with very short TTL (100ms-1s)
```javascript
const localCache = new LRU({ maxAge: 1000 });

async function getHotData(key) {
  let value = localCache.get(key);
  if (!value) {
    value = await redis.get(key);
    localCache.set(key, value);
  }
  return value;
}
```

2. **Key replication** - Spread load across multiple keys
```javascript
// Write to multiple replicas
for (let i = 0; i < 10; i++) {
  await redis.set(`hot:key:${i}`, value);
}

// Read from random replica
const replica = Math.floor(Math.random() * 10);
const value = await redis.get(`hot:key:${replica}`);
```

3. **Redis Cluster** - Distribute keys across nodes

4. **Read replicas** - Route reads to replicas

I typically combine local caching with key replication for extremely hot keys like trending items or global counters."

---

#### Q9: What metrics would you monitor for a caching system?

**Answer:**
"Key metrics I'd monitor:

**1. Hit/Miss Rate:**
- Target: >90% hit rate for most applications
- Alert if drops below threshold

**2. Latency:**
- P50, P95, P99 response times
- Compare cache hits vs misses

**3. Memory:**
- Used memory vs max memory
- Eviction rate
- Memory fragmentation ratio

**4. Throughput:**
- Operations per second
- Commands by type (GET, SET, DEL)

**5. Connections:**
- Connected clients
- Connection errors/timeouts

**6. Keys:**
- Total keys
- Keys by pattern
- Expired keys

**Alerts I'd set up:**
- Hit rate < 80%
- Memory > 80%
- Latency P99 > 10ms
- Error rate > 1%
- Evictions increasing rapidly

```javascript
// Health check endpoint
app.get('/health/cache', async (req, res) => {
  const metrics = await getCacheMetrics();
  const healthy = metrics.hitRate > 80 && metrics.errorRate < 1;
  res.status(healthy ? 200 : 503).json(metrics);
});
```"

---

### Scenario-Based Questions

#### Q10: Your cache hit rate dropped from 95% to 60%. How do you debug?

**Answer:**
"I'd investigate systematically:

**1. Check what changed:**
- Recent deployments?
- Traffic patterns changed?
- New features launched?

**2. Analyze cache behavior:**
- Are keys being evicted? (Memory pressure)
- Is TTL too short for new patterns?
- Are new endpoints not caching?

**3. Look at specific metrics:**
```javascript
// Check eviction rate
const info = await redis.info('stats');
// If evictions high → memory pressure

// Check key patterns
const keys = await redis.keys('*');
// Look for missing expected patterns
```

**4. Common causes:**
- Memory limit reached → evictions
- New traffic pattern → different keys accessed
- Code change removed caching
- Cache key changed (version bump)
- TTL too aggressive

**5. Quick fixes:**
- Increase memory if evicting
- Adjust TTL based on access patterns
- Add caching to new endpoints
- Check for cache penetration (null results)

I'd also add better monitoring to catch this earlier next time."

---

#### Q11: How would you migrate from Memcached to Redis with zero downtime?

**Answer:**
"I'd do a gradual migration:

**Phase 1: Dual-write**
```javascript
async function set(key, value, ttl) {
  await Promise.all([
    memcached.set(key, value, ttl),
    redis.set(key, value, 'EX', ttl)
  ]);
}
```

**Phase 2: Read from Redis, fallback to Memcached**
```javascript
async function get(key) {
  let value = await redis.get(key);
  if (!value) {
    value = await memcached.get(key);
    if (value) {
      // Backfill Redis
      await redis.set(key, value);
    }
  }
  return value;
}
```

**Phase 3: Monitor and compare**
- Compare hit rates
- Compare latencies
- Look for discrepancies

**Phase 4: Switch to Redis-only reads**
```javascript
async function get(key) {
  return redis.get(key);
}
```

**Phase 5: Stop Memcached writes**
```javascript
async function set(key, value, ttl) {
  await redis.set(key, value, 'EX', ttl);
}
```

**Phase 6: Decommission Memcached**

Throughout, I'd have feature flags to quickly rollback if issues arise."

---

## Quick Reference Cheat Sheet

```
┌─────────────────────────────────────────────────────────────┐
│                 CACHING CHEAT SHEET                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  CACHE LAYERS:                                              │
│  Browser → CDN → App (Redis) → Database                     │
│                                                              │
│  PATTERNS:                                                   │
│  • Cache-aside    → App manages cache explicitly            │
│  • Read-through   → Cache loads from DB automatically       │
│  • Write-through  → Write cache + DB synchronously          │
│  • Write-behind   → Write cache, DB asynchronously          │
│                                                              │
│  HTTP HEADERS:                                               │
│  • max-age=3600        → Cache for 1 hour                   │
│  • no-cache            → Always revalidate                  │
│  • no-store            → Never cache                        │
│  • private             → Browser only, not CDN              │
│  • s-maxage            → CDN TTL (overrides max-age)        │
│  • stale-while-reval   → Serve stale, refresh background    │
│                                                              │
│  INVALIDATION:                                               │
│  • TTL              → Simple, eventually fresh              │
│  • Event-driven     → Real-time, more complex               │
│  • Version-based    → Bump version = invalidate all         │
│                                                              │
│  PROBLEMS & SOLUTIONS:                                       │
│  • Stampede    → Locking, early expiry                      │
│  • Penetration → Cache nulls, bloom filter                  │
│  • Avalanche   → Random TTL, staggered warming              │
│  • Hot keys    → Local cache, key replication               │
│                                                              │
│  KEY METRICS:                                                │
│  • Hit rate > 90%                                           │
│  • Latency P99 < 10ms                                       │
│  • Memory < 80%                                             │
│  • Error rate < 1%                                          │
│                                                              │
│  REDIS VS MEMCACHED:                                         │
│  • Redis: Data structures, persistence, pub/sub             │
│  • Memcached: Simple, memory efficient                      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Resources

### Tools
| Tool | Purpose |
|------|---------|
| **Redis** | In-memory data store |
| **Memcached** | Distributed cache |
| **Varnish** | HTTP accelerator |
| **Cloudflare/Fastly** | CDN |
| **Workbox** | Service worker caching |

### Further Reading
- [Redis Documentation](https://redis.io/documentation)
- [HTTP Caching - MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)
- [Caching Strategies - AWS](https://aws.amazon.com/caching/best-practices/)
- [Google Web Fundamentals - Caching](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)

---

*Last updated: January 2026*
