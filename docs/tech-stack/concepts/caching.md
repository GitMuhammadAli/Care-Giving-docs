# Caching Strategies

> Understanding when and how to cache in CareCircle.

---

## The Mental Model

Think of caching like a **chef's mise en place**:

- **Database** = The walk-in refrigerator (everything stored, but takes time to get)
- **Cache** = The counter next to you (frequently used ingredients ready to grab)
- **Cache Hit** = Ingredient already on counter
- **Cache Miss** = Need to walk to the refrigerator

The goal: Keep the right things on the counter without it getting cluttered or stale.

---

## Why Cache?

### The Problem

```
WITHOUT CACHING:
────────────────

User clicks "View Medications"
  → API receives request
  → Query database
  → Database reads from disk
  → Returns to API
  → Returns to user
  
Time: 200-500ms

User clicks refresh
  → Same thing, same time

10 users viewing same family's medications
  → 10 identical database queries
  → Database under unnecessary load
```

### The Solution

```
WITH CACHING:
─────────────

First user clicks "View Medications"
  → API checks cache → MISS
  → Query database
  → Store result in cache
  → Return to user
  
Time: 200-500ms

User clicks refresh (within TTL)
  → API checks cache → HIT
  → Return cached result
  
Time: 5-20ms (10-100x faster!)

10 users viewing same family's medications
  → 1 database query, 9 cache hits
```

---

## Caching Patterns

### Pattern 1: Cache-Aside (Lazy Loading)

```
THE MOST COMMON PATTERN

┌─────────────────────────────────────────────────────────────────────────────┐
│                         CACHE-ASIDE PATTERN                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  READ:                                                                       │
│  ─────                                                                       │
│  1. Check cache first                                                        │
│  2. If cache hit → return cached data                                       │
│  3. If cache miss → query database → store in cache → return                │
│                                                                              │
│  WRITE:                                                                      │
│  ──────                                                                      │
│  1. Update database                                                          │
│  2. Invalidate cache (delete the key)                                       │
│  3. Next read will repopulate cache                                         │
│                                                                              │
│                                                                              │
│            Application                                                       │
│                │                                                             │
│        ┌───────┼───────┐                                                    │
│        │               │                                                     │
│        ▼               ▼                                                     │
│     Cache           Database                                                 │
│   (Redis)          (PostgreSQL)                                             │
│                                                                              │
│                                                                              │
│  PROS:                              CONS:                                   │
│  • Only caches what's actually used • First request always slow            │
│  • Simple to implement             • Cache and DB can be inconsistent      │
│  • Cache failure = slower, not broken                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

```typescript
// Implementation example
async getMedications(careRecipientId: string) {
  const cacheKey = `medications:${careRecipientId}`;
  
  // 1. Try cache first
  const cached = await this.cache.get<Medication[]>(cacheKey);
  if (cached) {
    return cached;  // Cache hit!
  }
  
  // 2. Cache miss - query database
  const medications = await this.prisma.medication.findMany({
    where: { careRecipientId }
  });
  
  // 3. Store in cache for next time
  await this.cache.set(cacheKey, medications, 300);  // 5 min TTL
  
  return medications;
}

// On write - invalidate
async createMedication(dto: CreateMedicationDto) {
  const medication = await this.prisma.medication.create({ data: dto });
  
  // Invalidate cache so next read gets fresh data
  await this.cache.del(`medications:${dto.careRecipientId}`);
  
  return medication;
}
```

### Pattern 2: Write-Through

```
UPDATES CACHE IMMEDIATELY ON WRITE

┌─────────────────────────────────────────────────────────────────────────────┐
│                         WRITE-THROUGH PATTERN                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  WRITE:                                                                      │
│  ──────                                                                      │
│  1. Update database                                                          │
│  2. Update cache with new data (not delete)                                 │
│  3. Return                                                                   │
│                                                                              │
│  READ:                                                                       │
│  ─────                                                                       │
│  1. Check cache → almost always hit                                         │
│  2. If miss → query database → store in cache                               │
│                                                                              │
│                                                                              │
│  PROS:                              CONS:                                   │
│  • Cache always has fresh data     • Caches data that may never be read    │
│  • Reads are always fast           • More complex write logic              │
│  • Good for read-heavy workloads   • Write latency includes cache write    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Pattern 3: Time-To-Live (TTL)

```
EVERY CACHED ITEM HAS AN EXPIRATION

┌─────────────────────────────────────────────────────────────────────────────┐
│                           TTL STRATEGIES                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  DATA TYPE                    │  SUGGESTED TTL   │  REASONING               │
│  ─────────────────────────────┼──────────────────┼────────────────────────  │
│  User profile                 │  5-15 minutes    │  Rarely changes          │
│  Family member list           │  5 minutes       │  Changes occasionally    │
│  Medication list              │  5 minutes       │  Balance freshness/perf  │
│  Notification count           │  30 seconds      │  Needs to be current     │
│  Static config                │  1 hour          │  Almost never changes    │
│  Session data                 │  Match JWT exp   │  Security requirement    │
│                                                                              │
│  RULE OF THUMB:                                                              │
│  • Acceptable staleness = maximum TTL                                        │
│  • "How bad if user sees data from X minutes ago?"                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## What to Cache (And What Not To)

### Cache Decision Matrix

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CACHE DECISION MATRIX                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ✅ GOOD CANDIDATES FOR CACHING                                             │
│  ────────────────────────────────                                           │
│  • Data that's read frequently                                              │
│  • Data that's expensive to compute/fetch                                   │
│  • Data that changes infrequently                                           │
│  • Data that's the same for many users                                      │
│                                                                              │
│  Examples:                                                                   │
│  • Family's medication list (read on every page load)                       │
│  • User profile (read on every request for auth)                            │
│  • Static configuration (countries, timezones)                              │
│                                                                              │
│  ❌ BAD CANDIDATES FOR CACHING                                              │
│  ──────────────────────────────                                             │
│  • Data that changes frequently                                              │
│  • Data where staleness is unacceptable                                     │
│  • User-specific transactional data                                          │
│  • Large datasets (consumes too much memory)                                │
│                                                                              │
│  Examples:                                                                   │
│  • Live emergency alert status                                               │
│  • Unread notification count (needs to be real-time)                        │
│  • Active shift check-in status                                              │
│  • Financial transactions                                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### The Caching Flowchart

```
                    ┌─────────────────────────────────────┐
                    │    Should I cache this data?        │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │    Is it read frequently?           │
                    └─────────────────┬───────────────────┘
                                      │
              ┌────────────NO─────────┴────────YES────────┐
              │                                           │
              ▼                                           ▼
    ┌─────────────────┐              ┌─────────────────────────────────┐
    │ Probably not    │              │ Can you tolerate stale data?    │
    │ worth caching   │              └──────────────┬──────────────────┘
    └─────────────────┘                             │
                                    ┌───────NO──────┴──────YES──────┐
                                    │                               │
                                    ▼                               ▼
                         ┌─────────────────┐          ┌─────────────────────┐
                         │ Use real-time   │          │ Is it expensive to  │
                         │ (WebSocket) or  │          │ fetch/compute?      │
                         │ very short TTL  │          └──────────┬──────────┘
                         └─────────────────┘                     │
                                                   ┌────NO───────┴────YES────┐
                                                   │                         │
                                                   ▼                         ▼
                                        ┌─────────────────┐      ┌─────────────────┐
                                        │ Maybe cache     │      │ DEFINITELY      │
                                        │ (depends on     │      │ cache this!     │
                                        │ load)           │      └─────────────────┘
                                        └─────────────────┘
```

---

## Cache Invalidation

### The Hard Problem

```
"There are only two hard things in Computer Science: 
 cache invalidation and naming things."
 - Phil Karlton


THE CHALLENGE:
──────────────

When data changes in the database, the cache becomes "stale."
How do you ensure users see fresh data?

STRATEGIES:
───────────

1. TIME-BASED (TTL)
   Cache expires automatically after X seconds
   Simple but allows temporary staleness

2. EVENT-BASED
   On write, explicitly invalidate related cache keys
   Fresh data but requires careful key management

3. VERSION-BASED
   Include version number in cache key
   On change, increment version
   Old cache automatically ignored
```

### Invalidation Patterns

```typescript
// Pattern 1: Single key invalidation
async updateMedication(id: string, data: UpdateDto) {
  await this.prisma.medication.update({ where: { id }, data });
  
  // Invalidate specific medication
  await this.cache.del(`medication:${id}`);
  
  // Also invalidate list (since list item changed)
  const med = await this.prisma.medication.findUnique({ where: { id } });
  await this.cache.del(`medications:${med.careRecipientId}`);
}


// Pattern 2: Pattern-based invalidation
async updateFamily(familyId: string, data: UpdateDto) {
  await this.prisma.family.update({ where: { id: familyId }, data });
  
  // Invalidate all keys matching pattern
  const keys = await this.cache.keys(`family:${familyId}:*`);
  if (keys.length > 0) {
    await this.cache.del(keys);
  }
}


// Pattern 3: Tag-based invalidation
// Store cache entries with tags, invalidate by tag
await this.cache.set(`medication:${id}`, data, {
  ttl: 300,
  tags: [`family:${familyId}`, `careRecipient:${recipientId}`]
});

// When family changes, invalidate all family-related cache
await this.cache.invalidateByTag(`family:${familyId}`);
```

---

## Redis as Cache

### Why Redis for CareCircle?

```
REDIS ADVANTAGES:
─────────────────
• In-memory: Microsecond read/write
• Persistence: Survives restarts (optional)
• Data structures: Not just key-value
• TTL built-in: Automatic expiration
• Atomic operations: Safe for concurrent access
• Pub/Sub: Real-time updates
• BullMQ: Job queue storage

VS. IN-MEMORY CACHE (e.g., node-cache):
───────────────────────────────────────
• Shared across multiple API instances
• Survives app restarts
• Can be monitored separately
```

### Redis Data Types for Caching

```
STRING (Most common for caching)
────────────────────────────────
SET medication:123 '{"name":"Aspirin","dosage":"100mg"}'
GET medication:123

Best for: Serialized objects, simple values


HASH (Object with fields)
─────────────────────────
HSET user:123 name "John" email "john@example.com"
HGET user:123 name

Best for: Objects where you read/write individual fields


LIST (Ordered collection)
─────────────────────────
LPUSH notifications:user:123 '{"id":1,"message":"..."}'
LRANGE notifications:user:123 0 9

Best for: Recent items, queues


SET (Unique collection)
───────────────────────
SADD family:123:members user:456 user:789
SISMEMBER family:123:members user:456

Best for: Membership checks, deduplication
```

---

## Caching at Different Layers

### Multi-Layer Cache Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MULTI-LAYER CACHING                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  LAYER 1: BROWSER CACHE                                                      │
│  ─────────────────────────                                                   │
│  Cache-Control headers                                                       │
│  Service worker cache                                                        │
│  Fastest, but per-user                                                       │
│                                                                              │
│  LAYER 2: CDN CACHE (Vercel Edge)                                           │
│  ─────────────────────────────────                                          │
│  Static assets (images, JS, CSS)                                            │
│  API responses (if headers allow)                                           │
│  Geographically distributed                                                 │
│                                                                              │
│  LAYER 3: APPLICATION CACHE (Redis)                                         │
│  ───────────────────────────────────                                        │
│  Database query results                                                      │
│  Computed data                                                               │
│  Session data                                                                │
│                                                                              │
│  LAYER 4: DATABASE QUERY CACHE                                              │
│  ─────────────────────────────                                              │
│  PostgreSQL's internal cache                                                │
│  Automatic, no management needed                                            │
│                                                                              │
│                                                                              │
│  REQUEST FLOW:                                                               │
│                                                                              │
│  Browser → CDN → API → Redis → Database                                     │
│    L1       L2    L3    L4       Source                                     │
│                                                                              │
│  Each layer checked before moving to next                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Common Caching Mistakes

### Mistake 1: Caching User-Specific Data Globally

```typescript
❌ WRONG: Same cache key for all users

async getNotifications() {
  const cached = await cache.get('notifications');  // Whose notifications?!
  // User A sees User B's notifications!
}


✅ RIGHT: Include user ID in key

async getNotifications(userId: string) {
  const cached = await cache.get(`notifications:${userId}`);
  // Scoped to specific user
}
```

### Mistake 2: Forgetting to Invalidate

```typescript
❌ WRONG: Cache never updated

async updateMedication(id: string, data: UpdateDto) {
  await this.prisma.medication.update({ where: { id }, data });
  // Forgot to invalidate cache!
  // Users see stale data until TTL expires
}


✅ RIGHT: Always invalidate on write

async updateMedication(id: string, data: UpdateDto) {
  await this.prisma.medication.update({ where: { id }, data });
  await this.cache.del(`medication:${id}`);
  await this.cache.del(`medications:${recipientId}`);
}
```

### Mistake 3: Cache Stampede

```
PROBLEM:
────────
Cache expires at T=0
100 requests come in at T=0.001
All 100 hit cache miss
All 100 query database simultaneously
Database overwhelmed!

SOLUTIONS:
──────────

1. LOCKING: Only one request queries DB, others wait
   if (await cache.acquireLock(key)) {
     const data = await queryDatabase();
     await cache.set(key, data);
     cache.releaseLock(key);
   } else {
     await waitForLock(key);
     return cache.get(key);
   }

2. STALE-WHILE-REVALIDATE: Serve stale, refresh in background
   const cached = await cache.get(key);
   if (cached && cached.isStale) {
     backgroundRefresh(key);  // Don't await
     return cached.data;      // Serve stale immediately
   }

3. PROBABILISTIC REFRESH: Refresh before expiry
   if (Math.random() < 0.1 && timeToExpiry < 30s) {
     await refreshCache(key);  // 10% chance of early refresh
   }
```

---

## Quick Reference

### Cache Key Conventions

```
PATTERN: resource:id:subresource

EXAMPLES:
  user:123                    → User object
  user:123:profile            → User's profile
  family:456:medications      → Family's medication list
  medication:789              → Single medication
  session:token:abc123        → Session data
```

### Redis Commands for Caching

```bash
# Set with TTL
SET key value EX 300        # 300 seconds

# Get
GET key

# Delete
DEL key

# Check existence
EXISTS key

# Set TTL on existing key
EXPIRE key 300

# Get TTL remaining
TTL key

# Delete keys by pattern
KEYS "user:*" | xargs DEL   # Warning: KEYS is slow in prod
SCAN 0 MATCH "user:*"       # Better for production
```

### TTL Quick Reference

| Data Type | Suggested TTL | Reason |
|-----------|---------------|--------|
| User profile | 15 min | Rarely changes |
| Family data | 5 min | Occasional changes |
| Medication list | 5 min | Changes daily |
| Dashboard stats | 1 min | Needs to be current |
| Static config | 1 hour | Almost never changes |
| Rate limit counter | 1 min | Must match window |

---

*Next: [Redis Patterns](../database/redis.md) | [Performance Optimization](performance.md)*


