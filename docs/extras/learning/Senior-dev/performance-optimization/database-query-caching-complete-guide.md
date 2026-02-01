# 💾 Database Query Caching - Complete Guide

> A comprehensive guide to database query caching - query result caching, materialized views, Redis caching, cache invalidation strategies, and patterns for high-performance data access.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Query caching stores the results of database queries in faster storage (memory/Redis), reducing database load and latency from 50ms+ to <1ms - but the hard part is knowing when to invalidate the cache."

### The Query Caching Mental Model
```
WITHOUT CACHING:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Request → Application → Database → Query (50ms) → Response    │
│  Request → Application → Database → Query (50ms) → Response    │
│  Request → Application → Database → Query (50ms) → Response    │
│                                                                  │
│  Same query executed 1000x = 1000 database hits               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

WITH CACHING:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Request 1 → Check Cache (miss) → DB Query (50ms) → Cache it  │
│  Request 2 → Check Cache (HIT) → Return cached (<1ms)         │
│  Request 3 → Check Cache (HIT) → Return cached (<1ms)         │
│  ...                                                            │
│  Request 1000 → Check Cache (HIT) → Return cached (<1ms)      │
│                                                                  │
│  1 database hit instead of 1000!                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Numbers to Remember
| Metric | Value | Context |
|--------|-------|---------|
| Redis GET latency | **<1ms** | vs 5-50ms for database |
| Cache hit ratio goal | **>90%** | Below 80% = check strategy |
| TTL for user data | **5-60 seconds** | Balance freshness vs load |
| TTL for static data | **1-24 hours** | Product catalogs, configs |
| Materialized view refresh | **Seconds to minutes** | Depends on data size |

### The "Wow" Statement
> "Our product listing page was hitting the database 10,000 times per minute with the same query. I implemented a read-through cache with Redis - first request queries DB and caches for 60 seconds, subsequent requests get cached results in <1ms. Database load dropped 95%, page latency from 200ms to 15ms. The tricky part was cache invalidation: when a product updates, we invalidate that specific product's cache key plus any listing caches containing it. We use a pub/sub pattern where the product update event triggers cache invalidation across all app servers."

### Key Terms to Drop
| Term | Use It Like This |
|------|------------------|
| **"Cache-aside"** | "Using cache-aside pattern - app checks cache, falls back to DB, then populates cache" |
| **"Write-through"** | "Write-through cache ensures cache is always in sync with database" |
| **"TTL"** | "Set TTL to 60 seconds - balance between freshness and cache hit ratio" |
| **"Cache stampede"** | "Implemented locking to prevent cache stampede when TTL expires" |
| **"Materialized view"** | "Using materialized view for complex aggregations - refreshes every 5 minutes" |

---

## 📚 Core Concepts

### Caching Patterns

```
CACHING PATTERNS OVERVIEW:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  CACHE-ASIDE (Lazy Loading)                                     │
│  ───────────────────────────                                    │
│  1. App checks cache                                           │
│  2. Cache miss → Query database                                │
│  3. Store result in cache                                      │
│  4. Return result                                              │
│                                                                  │
│  ✓ Only caches what's needed                                   │
│  ✗ First request always slow (cache miss)                      │
│                                                                  │
│  READ-THROUGH                                                   │
│  ────────────                                                   │
│  1. App always reads from cache                                │
│  2. Cache handles DB fallback internally                       │
│                                                                  │
│  ✓ Simple app code                                             │
│  ✗ Cache library must support DB integration                   │
│                                                                  │
│  WRITE-THROUGH                                                  │
│  ─────────────                                                  │
│  1. App writes to cache                                        │
│  2. Cache writes to database synchronously                     │
│  3. Both updated before returning                              │
│                                                                  │
│  ✓ Cache always consistent                                     │
│  ✗ Higher write latency                                        │
│                                                                  │
│  WRITE-BEHIND (Write-Back)                                     │
│  ─────────────────────────                                      │
│  1. App writes to cache only                                   │
│  2. Cache async writes to database                             │
│                                                                  │
│  ✓ Fast writes                                                 │
│  ✗ Risk of data loss if cache fails                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Cache-Aside Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// CACHE-ASIDE PATTERN WITH REDIS
// ════════════════════════════════════════════════════════════════

import Redis from 'ioredis';

const redis = new Redis();
const CACHE_TTL = 60; // seconds

class ProductService {
    // Cache-aside: Check cache first, fallback to DB
    async getProduct(id: string): Promise<Product> {
        const cacheKey = `product:${id}`;
        
        // 1. Check cache
        const cached = await redis.get(cacheKey);
        if (cached) {
            return JSON.parse(cached);
        }
        
        // 2. Cache miss - query database
        const product = await db.query(
            'SELECT * FROM products WHERE id = $1',
            [id]
        );
        
        // 3. Store in cache with TTL
        await redis.setex(cacheKey, CACHE_TTL, JSON.stringify(product));
        
        // 4. Return result
        return product;
    }
    
    // Invalidate cache on update
    async updateProduct(id: string, data: Partial<Product>): Promise<void> {
        // Update database
        await db.query(
            'UPDATE products SET name = $1, price = $2 WHERE id = $3',
            [data.name, data.price, id]
        );
        
        // Invalidate cache
        await redis.del(`product:${id}`);
        
        // Also invalidate any listing caches that might contain this product
        await this.invalidateListingCaches(id);
    }
    
    private async invalidateListingCaches(productId: string): Promise<void> {
        // Pattern-based deletion for listing caches
        const keys = await redis.keys('products:list:*');
        if (keys.length > 0) {
            await redis.del(...keys);
        }
    }
}

// ════════════════════════════════════════════════════════════════
// CACHING WITH STAMPEDE PROTECTION
// ════════════════════════════════════════════════════════════════

async function getWithLock(key: string, fetchFn: () => Promise<any>): Promise<any> {
    const cached = await redis.get(key);
    if (cached) return JSON.parse(cached);
    
    // Try to acquire lock
    const lockKey = `lock:${key}`;
    const acquired = await redis.set(lockKey, '1', 'EX', 10, 'NX');
    
    if (!acquired) {
        // Another process is fetching, wait and retry
        await sleep(100);
        return getWithLock(key, fetchFn);
    }
    
    try {
        // Double-check cache (another process might have populated it)
        const cached = await redis.get(key);
        if (cached) return JSON.parse(cached);
        
        // Fetch from source
        const data = await fetchFn();
        await redis.setex(key, CACHE_TTL, JSON.stringify(data));
        return data;
    } finally {
        await redis.del(lockKey);
    }
}
```

### Materialized Views

```sql
-- ════════════════════════════════════════════════════════════════
-- POSTGRESQL MATERIALIZED VIEWS
-- ════════════════════════════════════════════════════════════════

-- Complex aggregation that would be slow to compute on every request
CREATE MATERIALIZED VIEW product_stats AS
SELECT 
    p.category_id,
    COUNT(*) as product_count,
    AVG(p.price) as avg_price,
    SUM(oi.quantity) as total_sold,
    SUM(oi.quantity * oi.price) as total_revenue
FROM products p
LEFT JOIN order_items oi ON p.id = oi.product_id
LEFT JOIN orders o ON oi.order_id = o.id
WHERE o.created_at > NOW() - INTERVAL '30 days'
GROUP BY p.category_id;

-- Create index on materialized view for fast queries
CREATE INDEX idx_product_stats_category ON product_stats(category_id);

-- Query the materialized view (instant results)
SELECT * FROM product_stats WHERE category_id = 5;

-- Refresh the materialized view (run periodically)
REFRESH MATERIALIZED VIEW product_stats;

-- Refresh concurrently (doesn't lock reads, requires unique index)
CREATE UNIQUE INDEX idx_product_stats_unique ON product_stats(category_id);
REFRESH MATERIALIZED VIEW CONCURRENTLY product_stats;

-- ════════════════════════════════════════════════════════════════
-- AUTOMATIC REFRESH WITH PG_CRON
-- ════════════════════════════════════════════════════════════════

-- Install pg_cron extension
CREATE EXTENSION pg_cron;

-- Refresh every 5 minutes
SELECT cron.schedule('*/5 * * * *', 'REFRESH MATERIALIZED VIEW CONCURRENTLY product_stats');
```

### Application-Level Query Cache

```typescript
// ════════════════════════════════════════════════════════════════
// QUERY RESULT CACHING WITH CACHE TAGS
// ════════════════════════════════════════════════════════════════

class QueryCache {
    private redis: Redis;
    
    // Cache query result with tags for invalidation
    async cacheQuery<T>(
        key: string,
        tags: string[],
        ttl: number,
        queryFn: () => Promise<T>
    ): Promise<T> {
        // Check cache
        const cached = await this.redis.get(key);
        if (cached) return JSON.parse(cached);
        
        // Execute query
        const result = await queryFn();
        
        // Store result
        await this.redis.setex(key, ttl, JSON.stringify(result));
        
        // Store tag associations for invalidation
        for (const tag of tags) {
            await this.redis.sadd(`tag:${tag}`, key);
        }
        
        return result;
    }
    
    // Invalidate all caches with a specific tag
    async invalidateTag(tag: string): Promise<void> {
        const keys = await this.redis.smembers(`tag:${tag}`);
        if (keys.length > 0) {
            await this.redis.del(...keys);
        }
        await this.redis.del(`tag:${tag}`);
    }
}

// Usage
const cache = new QueryCache();

// Cache product listing with tags
const products = await cache.cacheQuery(
    'products:category:5:page:1',
    ['products', 'category:5'],  // Tags for invalidation
    60,
    () => db.query('SELECT * FROM products WHERE category_id = 5 LIMIT 20')
);

// When product in category 5 is updated, invalidate related caches
await cache.invalidateTag('category:5');
```

---

## Cache Invalidation Strategies

```
INVALIDATION STRATEGIES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. TIME-BASED (TTL)                                            │
│     Simplest: cache expires after X seconds                    │
│     ✓ Easy to implement                                        │
│     ✗ Data can be stale until TTL                              │
│                                                                  │
│  2. EVENT-BASED                                                 │
│     Invalidate when data changes (pub/sub)                     │
│     ✓ Always fresh                                             │
│     ✗ Complex to track all dependencies                        │
│                                                                  │
│  3. VERSION-BASED                                               │
│     Include version in cache key, increment on change          │
│     ✓ Simple invalidation                                      │
│     ✗ Old versions stay in cache (memory waste)               │
│                                                                  │
│  4. TAG-BASED                                                   │
│     Associate caches with tags, invalidate by tag              │
│     ✓ Flexible grouping                                        │
│     ✗ Overhead of tracking tags                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "How do you handle cache invalidation?"**
> "Depends on consistency requirements. For eventual consistency, TTL-based is simplest - cache expires after 60 seconds. For stronger consistency, event-based: when data changes, publish event that triggers cache deletion. I also use tag-based invalidation for related caches - updating a product invalidates all listing caches containing it."

**Q: "What is a cache stampede and how do you prevent it?"**
> "When cache expires, many concurrent requests all miss cache and hit database simultaneously - can overwhelm DB. Solutions: 1) Lock while fetching - only one request queries DB, others wait. 2) Probabilistic early expiration - some requests refresh before TTL. 3) Background refresh - refresh cache before expiration."

**Q: "When would you use a materialized view vs application caching?"**
> "Materialized view for complex aggregations that are expensive to compute - the database pre-computes and stores results. Application cache (Redis) for frequently accessed data that's already fast to query but accessed thousands of times. Often use both: materialized view simplifies the query, Redis caches the result."

---

## Quick Reference

```
QUERY CACHING CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  PATTERNS:                                                      │
│  • Cache-aside: App manages cache + DB                         │
│  • Read-through: Cache handles DB fallback                     │
│  • Write-through: Sync write to cache + DB                     │
│  • Write-behind: Async write to DB                             │
│                                                                  │
│  INVALIDATION:                                                  │
│  • TTL: Simple, eventual consistency                           │
│  • Event-based: Immediate, complex                             │
│  • Tag-based: Group invalidation                               │
│                                                                  │
│  PITFALLS:                                                      │
│  • Cache stampede → Use locking                                │
│  • Stale data → Proper invalidation                            │
│  • Memory overflow → Set TTL, max memory                       │
│                                                                  │
│  TOOLS:                                                         │
│  • Redis: Fast, versatile                                      │
│  • Memcached: Simple, fast                                     │
│  • Materialized Views: Complex aggregations                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


