# 🔀 Database Sharding - Complete Guide

> A comprehensive guide to database sharding - horizontal scaling, shard keys, consistent hashing, and strategies for distributing data across multiple databases.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Database sharding is horizontal partitioning where data is split across multiple database instances based on a shard key - like having multiple filing cabinets organized by last name instead of one giant cabinet."

### The Sharding Mental Model
```
SINGLE DATABASE (Vertical Scaling):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    ONE BIG DATABASE                      │   │
│  │                                                          │   │
│  │   Users: 100 million                                    │   │
│  │   Orders: 500 million                                   │   │
│  │   Items: 2 billion                                      │   │
│  │                                                          │   │
│  │   Problem: Can't fit on one machine!                    │   │
│  │   Solution: Add more RAM/CPU... has limits             │   │
│  │                                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

SHARDED DATABASE (Horizontal Scaling):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Shard Key: user_id                                             │
│                                                                  │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐      │
│  │   SHARD 1     │  │   SHARD 2     │  │   SHARD 3     │      │
│  │   user_id     │  │   user_id     │  │   user_id     │      │
│  │   1-33M       │  │   34M-66M     │  │   67M-100M    │      │
│  │               │  │               │  │               │      │
│  │   33M users   │  │   33M users   │  │   34M users   │      │
│  │   166M orders │  │   166M orders │  │   168M orders │      │
│  │   666M items  │  │   666M items  │  │   668M items  │      │
│  └───────────────┘  └───────────────┘  └───────────────┘      │
│                                                                  │
│  ✓ Each shard is manageable                                    │
│  ✓ Add more shards as needed                                   │
│  ✓ Queries route to specific shard                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Numbers to Remember
| Metric | Value | Context |
|--------|-------|---------|
| Single DB practical limit | **~1-5 TB** | Before performance degrades |
| Shard count start | **4-8 shards** | Plan for growth from start |
| Resharding pain | **Massive** | Design shard key carefully! |
| Cross-shard query overhead | **2-10x slower** | Avoid at all costs |
| Connection overhead | **Per shard** | 8 shards × 50 connections = 400 |

### The "Wow" Statement (Memorize This!)
> "At my previous company, our single PostgreSQL hit 2TB and queries were taking 30+ seconds. We implemented tenant-based sharding with consistent hashing - each tenant's data lives entirely on one shard, eliminating cross-shard queries. We used a routing layer that hashes tenant_id to determine the shard. Started with 8 shards, now at 32 handling 50TB total. The tricky part was the migration - we used dual-writes for 2 weeks: write to both old and new location, then switched reads. Key lesson: choosing tenant_id as shard key meant all queries for a tenant hit one shard, keeping JOINs fast."

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Shard key"** | "We chose tenant_id as shard key to keep each tenant's data colocated" |
| **"Consistent hashing"** | "Using consistent hashing so adding shards only redistributes 1/n of data" |
| **"Hotspot"** | "One shard became a hotspot - 80% of traffic hitting single shard" |
| **"Cross-shard query"** | "Had to eliminate cross-shard queries - they were 10x slower" |
| **"Shard routing"** | "Application layer handles shard routing via hash ring lookup" |
| **"Resharding"** | "Resharding took 3 months - that's why we over-provisioned initially" |
| **"Scatter-gather"** | "Analytics required scatter-gather across all shards" |

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "What is database sharding?"

**Junior Answer:**
> "Splitting data across multiple databases."

**Senior Answer:**
> "Sharding is horizontal partitioning where you distribute rows across multiple database instances based on a shard key. Unlike read replicas (copies of same data), each shard holds a unique subset.

**Key decisions:**
1. **Shard key selection** - Determines data distribution. Bad key = uneven shards, cross-shard queries
2. **Sharding strategy** - Range (user_id 1-1M), hash (hash(id) mod N), directory (lookup table)
3. **Routing layer** - How app knows which shard to query

**Trade-offs:**
- Pros: Horizontal scaling, fault isolation
- Cons: Complexity (routing, migrations, cross-shard queries), operational overhead

**When to shard:**
- Single DB exceeds 1-5TB practical limit
- Read replicas aren't enough (write bottleneck)
- Need fault isolation per tenant

I'd delay sharding as long as possible - the operational complexity is significant. Often read replicas + caching solve the problem first."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "How do you choose a shard key?" | "High cardinality, even distribution, used in most queries, avoids cross-shard JOINs. Usually tenant_id or user_id." |
| "What about cross-shard queries?" | "Avoid them! Design schema so queries hit single shard. If unavoidable, use scatter-gather (query all, merge in app) or denormalize." |
| "How do you add more shards?" | "Consistent hashing minimizes data movement. Still painful - dual-write period, then migrate reads. Plan for 2x capacity from start." |
| "What about transactions?" | "Within shard: normal transactions. Across shards: need distributed transactions (2PC) or saga pattern. Avoid cross-shard transactions!" |

---

## 📚 Table of Contents

1. [Sharding Strategies](#1-sharding-strategies)
2. [Shard Key Selection](#2-shard-key-selection)
3. [Consistent Hashing](#3-consistent-hashing)
4. [Implementation Patterns](#4-implementation-patterns)
5. [Cross-Shard Operations](#5-cross-shard-operations)
6. [Migration & Resharding](#6-migration--resharding)
7. [Common Pitfalls](#7-common-pitfalls)
8. [Interview Questions](#8-interview-questions)

---

## 1. Sharding Strategies

### Range-Based Sharding

```
RANGE-BASED SHARDING:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Shard by ranges of the shard key                               │
│                                                                  │
│  Example: user_id ranges                                        │
│                                                                  │
│  Shard 1: user_id 1 - 1,000,000                                │
│  Shard 2: user_id 1,000,001 - 2,000,000                        │
│  Shard 3: user_id 2,000,001 - 3,000,000                        │
│                                                                  │
│  ✓ Pros:                                                        │
│    • Simple to understand and implement                        │
│    • Range queries within shard are efficient                  │
│    • Easy to split: just pick a midpoint                      │
│                                                                  │
│  ✗ Cons:                                                        │
│    • Hot spots: newest users on one shard                      │
│    • Uneven distribution: some ranges busier                   │
│    • Shard 1 might have 500K active, Shard 3 has 50K          │
│                                                                  │
│  GOOD FOR:                                                      │
│  • Time-series data (shard by date ranges)                     │
│  • Geographic data (shard by region)                           │
│  • Known distribution patterns                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Hash-Based Sharding

```
HASH-BASED SHARDING:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Shard = hash(shard_key) % num_shards                          │
│                                                                  │
│  Example: 4 shards                                              │
│                                                                  │
│  user_id = 12345                                               │
│  hash(12345) = 789234                                          │
│  789234 % 4 = 2 → Shard 2                                      │
│                                                                  │
│  user_id = 67890                                               │
│  hash(67890) = 123456                                          │
│  123456 % 4 = 0 → Shard 0                                      │
│                                                                  │
│  ✓ Pros:                                                        │
│    • Even distribution of data                                 │
│    • No hot spots (assuming good hash)                         │
│    • Works with any shard key                                  │
│                                                                  │
│  ✗ Cons:                                                        │
│    • Range queries span all shards                             │
│    • Adding shards requires rehashing everything!              │
│    • Can't easily split one shard                              │
│                                                                  │
│  GOOD FOR:                                                      │
│  • Random access patterns                                      │
│  • Need even distribution                                      │
│  • Lookup by ID primarily                                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Directory-Based Sharding

```
DIRECTORY-BASED SHARDING:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Lookup table maps shard key → shard                           │
│                                                                  │
│  ┌─────────────────────────────────────┐                       │
│  │        SHARD DIRECTORY              │                       │
│  │   (Central lookup service)          │                       │
│  ├─────────────────────────────────────┤                       │
│  │  tenant_id │ shard                  │                       │
│  │  ──────────┼───────                 │                       │
│  │  acme-corp │ shard-us-east-1        │                       │
│  │  globex    │ shard-eu-west-1        │                       │
│  │  initech   │ shard-us-east-1        │                       │
│  │  wayne-ent │ shard-eu-west-1        │                       │
│  └─────────────────────────────────────┘                       │
│                                                                  │
│  ✓ Pros:                                                        │
│    • Maximum flexibility                                       │
│    • Easy to move single tenant                                │
│    • Geographic/compliance placement                           │
│    • Can handle uneven tenants                                 │
│                                                                  │
│  ✗ Cons:                                                        │
│    • Directory is single point of failure                      │
│    • Extra lookup on every query                               │
│    • Directory must be highly available                        │
│                                                                  │
│  GOOD FOR:                                                      │
│  • Multi-tenant SaaS                                           │
│  • Data residency requirements                                 │
│  • Uneven tenant sizes                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Comparison Table

| Strategy | Distribution | Flexibility | Resharding | Range Queries |
|----------|-------------|-------------|------------|---------------|
| Range | Can be uneven | Medium | Easy to split | Single shard |
| Hash | Even | Low | Rehash all | All shards |
| Directory | Controlled | High | Move entries | Depends |
| Consistent Hash | Even | High | Minimal impact | All shards |

---

## 2. Shard Key Selection

### What Makes a Good Shard Key?

```
SHARD KEY SELECTION CRITERIA:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. HIGH CARDINALITY                                            │
│     Many unique values to distribute evenly                     │
│     ✗ Bad: status (3 values)                                   │
│     ✗ Bad: country (200 values)                                │
│     ✓ Good: user_id (millions of values)                       │
│     ✓ Good: tenant_id (thousands of values)                    │
│                                                                  │
│  2. EVEN DISTRIBUTION                                           │
│     Each shard should have similar load                        │
│     ✗ Bad: company_id (1 company = 50% of data)               │
│     ✓ Good: user_id (users roughly equal activity)             │
│                                                                  │
│  3. QUERY ISOLATION                                             │
│     Most queries should hit single shard                       │
│     ✗ Bad: created_at (range queries span shards)              │
│     ✓ Good: tenant_id (all tenant queries → one shard)        │
│                                                                  │
│  4. IMMUTABLE (or rarely changed)                               │
│     Changing shard key = moving data between shards            │
│     ✗ Bad: email (user changes email)                          │
│     ✓ Good: user_id (never changes)                            │
│                                                                  │
│  5. FREQUENTLY USED IN QUERIES                                  │
│     Key should be available in most queries                    │
│     ✗ Bad: obscure_field (not in most queries)                 │
│     ✓ Good: user_id (always have user context)                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Shard Key Examples by Use Case

```typescript
// ════════════════════════════════════════════════════════════════
// MULTI-TENANT SAAS: tenant_id is ideal
// ════════════════════════════════════════════════════════════════

// All tenant data colocated on one shard
// Schema: tenant_id in every table

// Users table
CREATE TABLE users (
    id UUID PRIMARY KEY,
    tenant_id UUID NOT NULL,  -- Shard key
    email TEXT,
    created_at TIMESTAMPTZ
);

// Orders table - same shard key
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    tenant_id UUID NOT NULL,  -- Shard key (matches users)
    user_id UUID,
    total DECIMAL
);

// Query: Always include tenant_id
const orders = await db.query(`
    SELECT * FROM orders 
    WHERE tenant_id = $1 AND user_id = $2
`, [tenantId, userId]);
// Routes to single shard - fast!

// ════════════════════════════════════════════════════════════════
// SOCIAL NETWORK: user_id as shard key
// ════════════════════════════════════════════════════════════════

// User's posts, likes, comments on same shard as user
// Challenge: What about posts by user A viewed by user B?

// Solution: Denormalize for reads
CREATE TABLE posts (
    id UUID PRIMARY KEY,
    author_user_id UUID NOT NULL,  -- Shard key for writes
    content TEXT
);

// Denormalized feed per user (on user's shard)
CREATE TABLE user_feed (
    user_id UUID NOT NULL,  -- Shard key
    post_id UUID,
    author_id UUID,
    content TEXT,
    created_at TIMESTAMPTZ
);

// ════════════════════════════════════════════════════════════════
// E-COMMERCE: Compound shard key
// ════════════════════════════════════════════════════════════════

// Orders: shard by customer_id
// But need to query by merchant too...

// Solution: Shard by customer, replicate to merchant shard
// Or: Use compound key hash(customer_id + merchant_id)

function getShardForOrder(customerId: string, merchantId: string) {
    // Customer-centric queries → shard by customer
    return hashToShard(customerId);
}

// For merchant queries: scatter-gather or separate index
```

---

## 3. Consistent Hashing

### The Problem with Simple Hashing

```
SIMPLE HASH PROBLEM:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Original: 4 shards                                             │
│  Formula: shard = hash(key) % 4                                 │
│                                                                  │
│  key "user_123" → hash = 789 → 789 % 4 = 1 → Shard 1          │
│  key "user_456" → hash = 234 → 234 % 4 = 2 → Shard 2          │
│  key "user_789" → hash = 567 → 567 % 4 = 3 → Shard 3          │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  Add 1 shard: Now 5 shards                                     │
│  Formula: shard = hash(key) % 5                                 │
│                                                                  │
│  key "user_123" → hash = 789 → 789 % 5 = 4 → Shard 4 ❌ MOVED │
│  key "user_456" → hash = 234 → 234 % 5 = 4 → Shard 4 ❌ MOVED │
│  key "user_789" → hash = 567 → 567 % 5 = 2 → Shard 2 ❌ MOVED │
│                                                                  │
│  ALMOST ALL DATA NEEDS TO MOVE!                                │
│  4 shards → 5 shards = 80% of keys rehashed                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Consistent Hashing Solution

```
CONSISTENT HASHING (Hash Ring):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Imagine shards placed on a circular ring (0 to 2^32)          │
│                                                                  │
│                        0                                        │
│                        │                                        │
│                   ┌────┴────┐                                   │
│                   │         │                                   │
│               ┌───┤ Shard A ├───┐                              │
│               │   │         │   │                              │
│               │   └────┬────┘   │                              │
│               │        │        │                              │
│       Shard D ●        │        ● Shard B                      │
│               │        │        │                              │
│               │   ┌────┴────┐   │                              │
│               │   │         │   │                              │
│               └───┤ Shard C ├───┘                              │
│                   │         │                                   │
│                   └────┬────┘                                   │
│                        │                                        │
│                       2^31                                      │
│                                                                  │
│  KEY PLACEMENT:                                                 │
│  • Hash the key to get position on ring                        │
│  • Walk clockwise to find first shard                          │
│  • That shard owns the key                                     │
│                                                                  │
│  ADDING A SHARD:                                                │
│  • New shard takes over portion of ring                        │
│  • Only keys between new shard and previous move               │
│  • ~1/n of keys move (not 80%!)                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// CONSISTENT HASHING IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

import * as crypto from 'crypto';

class ConsistentHash {
    private ring: Map<number, string> = new Map();
    private sortedKeys: number[] = [];
    private virtualNodes: number;
    
    constructor(virtualNodes = 100) {
        // Virtual nodes improve distribution
        this.virtualNodes = virtualNodes;
    }
    
    private hash(key: string): number {
        const hash = crypto.createHash('md5').update(key).digest();
        return hash.readUInt32BE(0);
    }
    
    addShard(shardId: string): void {
        // Add multiple virtual nodes per shard for better distribution
        for (let i = 0; i < this.virtualNodes; i++) {
            const virtualKey = `${shardId}:${i}`;
            const position = this.hash(virtualKey);
            this.ring.set(position, shardId);
            this.sortedKeys.push(position);
        }
        this.sortedKeys.sort((a, b) => a - b);
    }
    
    removeShard(shardId: string): void {
        for (let i = 0; i < this.virtualNodes; i++) {
            const virtualKey = `${shardId}:${i}`;
            const position = this.hash(virtualKey);
            this.ring.delete(position);
            this.sortedKeys = this.sortedKeys.filter(k => k !== position);
        }
    }
    
    getShardForKey(key: string): string {
        if (this.ring.size === 0) {
            throw new Error('No shards available');
        }
        
        const position = this.hash(key);
        
        // Find first shard with position >= key position (clockwise)
        for (const shardPosition of this.sortedKeys) {
            if (shardPosition >= position) {
                return this.ring.get(shardPosition)!;
            }
        }
        
        // Wrap around to first shard
        return this.ring.get(this.sortedKeys[0])!;
    }
}

// Usage
const hashRing = new ConsistentHash(100);

// Add shards
hashRing.addShard('shard-1');
hashRing.addShard('shard-2');
hashRing.addShard('shard-3');

// Route queries
function getDbConnection(userId: string) {
    const shardId = hashRing.getShardForKey(userId);
    return shardConnections.get(shardId);
}

// When adding new shard:
hashRing.addShard('shard-4');
// Only ~25% of keys now map to shard-4 (moved from others)
```

---

## 4. Implementation Patterns

### Application-Level Routing

```typescript
// ════════════════════════════════════════════════════════════════
// SHARD ROUTING LAYER
// ════════════════════════════════════════════════════════════════

interface ShardConfig {
    id: string;
    host: string;
    port: number;
    database: string;
}

class ShardRouter {
    private shardConfigs: Map<string, ShardConfig>;
    private shardPools: Map<string, Pool>;
    private hashRing: ConsistentHash;
    
    constructor(configs: ShardConfig[]) {
        this.shardConfigs = new Map();
        this.shardPools = new Map();
        this.hashRing = new ConsistentHash();
        
        for (const config of configs) {
            this.shardConfigs.set(config.id, config);
            this.shardPools.set(config.id, new Pool({
                host: config.host,
                port: config.port,
                database: config.database,
                max: 20
            }));
            this.hashRing.addShard(config.id);
        }
    }
    
    getShardForTenant(tenantId: string): string {
        return this.hashRing.getShardForKey(tenantId);
    }
    
    getPool(shardId: string): Pool {
        const pool = this.shardPools.get(shardId);
        if (!pool) throw new Error(`Unknown shard: ${shardId}`);
        return pool;
    }
    
    async query(tenantId: string, sql: string, params: any[]) {
        const shardId = this.getShardForTenant(tenantId);
        const pool = this.getPool(shardId);
        return pool.query(sql, params);
    }
    
    // Execute on all shards (scatter)
    async queryAll(sql: string, params: any[]) {
        const promises = Array.from(this.shardPools.values())
            .map(pool => pool.query(sql, params));
        return Promise.all(promises);
    }
}

// ════════════════════════════════════════════════════════════════
// USAGE IN APPLICATION
// ════════════════════════════════════════════════════════════════

const router = new ShardRouter([
    { id: 'shard-1', host: 'db1.example.com', port: 5432, database: 'app' },
    { id: 'shard-2', host: 'db2.example.com', port: 5432, database: 'app' },
    { id: 'shard-3', host: 'db3.example.com', port: 5432, database: 'app' },
]);

// Single-shard query (most common)
async function getOrdersForTenant(tenantId: string) {
    return router.query(
        tenantId,
        'SELECT * FROM orders WHERE tenant_id = $1',
        [tenantId]
    );
}

// Cross-shard aggregation (rare, expensive)
async function getGlobalStats() {
    const results = await router.queryAll(
        'SELECT COUNT(*) as count, SUM(total) as revenue FROM orders',
        []
    );
    
    // Aggregate in application
    return results.reduce((acc, r) => ({
        count: acc.count + parseInt(r.rows[0].count),
        revenue: acc.revenue + parseFloat(r.rows[0].revenue)
    }), { count: 0, revenue: 0 });
}
```

---

## 5. Cross-Shard Operations

### Scatter-Gather Pattern

```typescript
// ════════════════════════════════════════════════════════════════
// SCATTER-GATHER: Query all shards, aggregate results
// ════════════════════════════════════════════════════════════════

async function searchAllShards(query: string, limit: number) {
    // Scatter: Query each shard in parallel
    const shardResults = await Promise.all(
        shards.map(shard => 
            shard.query(`
                SELECT * FROM products 
                WHERE name ILIKE $1 
                ORDER BY relevance DESC 
                LIMIT $2
            `, [`%${query}%`, limit])
        )
    );
    
    // Gather: Merge and re-sort
    const allResults = shardResults.flatMap(r => r.rows);
    
    return allResults
        .sort((a, b) => b.relevance - a.relevance)
        .slice(0, limit);
}

// Pagination across shards is HARD
async function paginatedSearchAllShards(
    query: string, 
    page: number, 
    pageSize: number
) {
    // Must fetch (page * pageSize) from each shard, then merge
    // Gets expensive fast!
    
    const offset = (page - 1) * pageSize;
    const fetchPerShard = offset + pageSize;  // Over-fetch
    
    const shardResults = await Promise.all(
        shards.map(shard => 
            shard.query(`
                SELECT * FROM products 
                WHERE name ILIKE $1 
                ORDER BY created_at DESC 
                LIMIT $2
            `, [`%${query}%`, fetchPerShard])
        )
    );
    
    // Merge, sort, then take page
    const allResults = shardResults
        .flatMap(r => r.rows)
        .sort((a, b) => b.created_at - a.created_at)
        .slice(offset, offset + pageSize);
    
    return allResults;
}
```

### Cross-Shard Transactions

```typescript
// ════════════════════════════════════════════════════════════════
// SAGA PATTERN: Distributed transactions without 2PC
// ════════════════════════════════════════════════════════════════

// Transfer money between users on different shards
async function transferMoney(
    fromUserId: string,
    toUserId: string,
    amount: number
) {
    const fromShard = router.getShardForKey(fromUserId);
    const toShard = router.getShardForKey(toUserId);
    
    // Same shard? Easy - use regular transaction
    if (fromShard === toShard) {
        return router.query(fromUserId, `
            BEGIN;
            UPDATE accounts SET balance = balance - $1 WHERE user_id = $2;
            UPDATE accounts SET balance = balance + $1 WHERE user_id = $3;
            COMMIT;
        `, [amount, fromUserId, toUserId]);
    }
    
    // Different shards? Use saga pattern
    const transferId = uuid();
    
    try {
        // Step 1: Debit from sender (with compensation data)
        await router.query(fromUserId, `
            UPDATE accounts 
            SET balance = balance - $1,
                pending_transfer = $2
            WHERE user_id = $3 AND balance >= $1
        `, [amount, transferId, fromUserId]);
        
        // Step 2: Credit to receiver
        await router.query(toUserId, `
            UPDATE accounts 
            SET balance = balance + $1
            WHERE user_id = $2
        `, [amount, toUserId]);
        
        // Step 3: Clear pending on sender
        await router.query(fromUserId, `
            UPDATE accounts 
            SET pending_transfer = NULL
            WHERE user_id = $1
        `, [fromUserId]);
        
    } catch (error) {
        // Compensating transaction: rollback debit
        await router.query(fromUserId, `
            UPDATE accounts 
            SET balance = balance + $1,
                pending_transfer = NULL
            WHERE user_id = $2 AND pending_transfer = $3
        `, [amount, fromUserId, transferId]);
        
        throw error;
    }
}
```

---

## 6. Migration & Resharding

```typescript
// ════════════════════════════════════════════════════════════════
// RESHARDING: Adding a new shard
// ════════════════════════════════════════════════════════════════

async function addShard(newShardConfig: ShardConfig) {
    // Phase 1: Add shard to routing (no data yet)
    const keysToMove = hashRing.addShard(newShardConfig.id);
    
    // Phase 2: Dual-write mode
    // New writes go to BOTH old and new shard location
    setDualWriteMode(true);
    
    // Phase 3: Background migration
    for (const key of keysToMove) {
        const oldShard = getOldShardForKey(key);
        const newShard = newShardConfig.id;
        
        // Copy data
        const data = await router.query(oldShard, 
            'SELECT * FROM data WHERE key = $1', [key]);
        await router.query(newShard,
            'INSERT INTO data SELECT * FROM $1', [data]);
    }
    
    // Phase 4: Switch reads to new shard
    setDualWriteMode(false);
    
    // Phase 5: Cleanup old data (optional, after verification)
    for (const key of keysToMove) {
        const oldShard = getOldShardForKey(key);
        await router.query(oldShard,
            'DELETE FROM data WHERE key = $1', [key]);
    }
}
```

---

## 7. Common Pitfalls

```
SHARDING PITFALLS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. WRONG SHARD KEY                                             │
│     Problem: Cross-shard queries everywhere                    │
│     Solution: Choose key used in 95%+ of queries              │
│                                                                  │
│  2. HOT SPOTS                                                   │
│     Problem: One shard gets 80% of traffic                     │
│     Solution: Better hash function, more virtual nodes         │
│                                                                  │
│  3. CROSS-SHARD JOINS                                           │
│     Problem: JOINs across shards are very slow                 │
│     Solution: Denormalize, keep related data together          │
│                                                                  │
│  4. SHARDING TOO EARLY                                          │
│     Problem: Complexity without benefit                        │
│     Solution: Read replicas + caching first                    │
│                                                                  │
│  5. NOT ENOUGH SHARDS                                           │
│     Problem: Need to reshard after 6 months                    │
│     Solution: Start with 2x expected capacity                  │
│                                                                  │
│  6. FORGETTING SEQUENCES                                        │
│     Problem: Auto-increment IDs conflict across shards         │
│     Solution: Use UUIDs or shard-prefixed IDs                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. Interview Questions

**Q: "When would you use sharding vs read replicas?"**
> "Read replicas for read scaling - same data copied to multiple servers. Sharding for write scaling and data size - different data on different servers. If problem is read load, replicas first. If problem is write load or data doesn't fit on one machine, then sharding."

**Q: "How do you handle cross-shard JOINs?"**
> "Avoid them! Design schema so JOINs happen within shard. If unavoidable: 1) Denormalize data, 2) Scatter-gather (query all, merge in app), 3) Maintain a separate aggregation database. Cross-shard JOINs are 10x+ slower."

**Q: "What happens when you add a new shard?"**
> "With consistent hashing, only ~1/n of data moves. Process: 1) Add shard to ring, 2) Enable dual-writes, 3) Background migrate affected keys, 4) Switch reads, 5) Cleanup old data. Takes days/weeks for large datasets."

---

## Quick Reference

```
SHARDING CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  STRATEGIES:                                                    │
│  • Range: Easy, but can have hot spots                         │
│  • Hash: Even distribution, hard to reshard                    │
│  • Directory: Flexible, extra lookup overhead                  │
│  • Consistent Hash: Best for dynamic scaling                   │
│                                                                  │
│  SHARD KEY CRITERIA:                                            │
│  • High cardinality                                             │
│  • Even distribution                                            │
│  • Used in most queries                                         │
│  • Immutable                                                    │
│                                                                  │
│  AVOID:                                                         │
│  • Cross-shard JOINs                                           │
│  • Cross-shard transactions                                    │
│  • Sharding too early                                          │
│  • Auto-increment IDs (use UUIDs)                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

*Guide covers application-level sharding patterns. Cloud-managed solutions (CockroachDB, Vitess, Citus) handle much of this automatically.*


