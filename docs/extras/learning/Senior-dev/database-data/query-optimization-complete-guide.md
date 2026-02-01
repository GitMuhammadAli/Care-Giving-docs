# 🔍 Query Optimization - Complete Guide

> A comprehensive guide to query optimization - EXPLAIN plans, N+1 problem, eager/lazy loading, query analysis, and how to make your database queries blazing fast.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Query optimization is analyzing execution plans, eliminating N+1 queries, choosing appropriate loading strategies, and restructuring queries to minimize database work - often achieving 10-1000x performance improvements without changing indexes."

### The Query Optimization Mental Model
```
QUERY PERFORMANCE FACTORS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SLOW QUERY                        FAST QUERY                   │
│  ──────────                        ──────────                   │
│                                                                  │
│  • Full table scan                 • Index scan                 │
│  • N+1 queries (100 queries)       • Single JOIN (1 query)     │
│  • SELECT * (all columns)          • SELECT specific columns   │
│  • No LIMIT                        • LIMIT + pagination        │
│  • Subquery in SELECT              • JOIN or CTE               │
│  • DISTINCT on large result        • Proper GROUP BY           │
│  • ORDER BY unindexed column       • ORDER BY indexed column   │
│                                                                  │
│  TIME BREAKDOWN (typical slow query):                          │
│  ┌──────────────────────────────────────────────────────┐     │
│  │ Network latency: 5ms                                  │     │
│  │ Query parsing:   1ms                                  │     │
│  │ Planning:        2ms                                  │     │
│  │ Execution:       5000ms  ← THE PROBLEM               │     │
│  │   └─ Seq scan:   4500ms                              │     │
│  │   └─ Sorting:    500ms                               │     │
│  └──────────────────────────────────────────────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Numbers to Remember
| Metric | Value | Context |
|--------|-------|---------|
| N+1 impact | **100+ queries** | 100 items = 101 queries vs 1-2 queries |
| Network latency | **1-5ms per query** | N+1 adds up fast |
| Index vs seq scan | **100-1000x faster** | For selective queries |
| SELECT * overhead | **2-10x more data** | Especially with large columns |
| JOIN vs subquery | **Often 10x faster** | Optimizer handles JOINs better |

### The "Wow" Statement (Memorize This!)
> "At my previous company, we had an API endpoint that took 8 seconds to load user dashboards. Using `pg_stat_statements`, I identified the culprit: a classic N+1 problem loading 50 orders with their items - that's 51 queries! I refactored to eager load with a single JOIN query and added a covering index. Response time dropped to 80ms - that's 100x faster. The key insight came from EXPLAIN ANALYZE showing 47ms per query × 51 queries = 2.4s just in query execution, plus connection overhead. The fix was literally changing `.map(async order => await getItems(order.id))` to a single query with `JOIN order_items ON orders.id = order_items.order_id`."

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Execution plan"** | "The execution plan shows a sequential scan - we need an index here" |
| **"N+1 problem"** | "That's a classic N+1 - we're making 101 queries instead of 2" |
| **"Eager loading"** | "Switched to eager loading with includes to fetch related data in one query" |
| **"Query cost"** | "EXPLAIN shows cost of 50000 - way too high for this simple lookup" |
| **"Correlated subquery"** | "That correlated subquery executes once per row - rewrite as JOIN" |
| **"Seq scan vs index scan"** | "Seq scan on 10M rows means reading every row - need better index" |
| **"Data loader pattern"** | "Using DataLoader to batch and cache queries within a request" |

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How do you optimize slow database queries?"

**Junior Answer:**
> "Add an index or use a faster database."

**Senior Answer:**
> "I follow a systematic approach:

**1. Measure first**
- Enable slow query logging (>100ms)
- Use `pg_stat_statements` to find frequent slow queries
- Run EXPLAIN ANALYZE on suspects

**2. Analyze the execution plan**
- Look for seq scans on large tables
- Check if indexes are being used
- Identify high-cost operations
- Compare estimated vs actual rows

**3. Common fixes (in order of impact)**
- Fix N+1 with eager loading or batching
- Add missing indexes
- Rewrite correlated subqueries as JOINs
- Add covering indexes for frequently accessed columns
- Use pagination instead of loading all rows
- Cache results that don't change often

**4. Verify improvement**
- Re-run EXPLAIN ANALYZE
- Measure actual response time
- Monitor in production

The key is measuring before and after - sometimes 'optimizations' actually make things worse due to different data distributions or query patterns."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "What's the N+1 problem?" | "Loading 100 users then 100 separate queries for their orders. Solution: JOIN or eager load - 1-2 queries instead of 101." |
| "Eager vs lazy loading?" | "Eager: fetch related data immediately (good when you'll need it). Lazy: fetch on access (good when you might not need it). Wrong choice = N+1 or over-fetching." |
| "How do you read EXPLAIN?" | "Look for: Seq Scan (bad on big tables), high cost numbers, rows estimate vs actual (big diff = stale stats), nested loops with high iterations." |
| "When is a seq scan okay?" | "Small tables (<1000 rows), low selectivity (returning >15% of rows), or when combined with index on another table in a JOIN." |

---

## 📚 Table of Contents

1. [EXPLAIN Deep Dive](#1-explain-deep-dive)
2. [N+1 Problem](#2-n1-problem)
3. [Eager vs Lazy Loading](#3-eager-vs-lazy-loading)
4. [Query Patterns & Anti-Patterns](#4-query-patterns--anti-patterns)
5. [ORM Optimization](#5-orm-optimization)
6. [Query Analysis Tools](#6-query-analysis-tools)
7. [Common Pitfalls](#7-common-pitfalls)
8. [Interview Questions](#8-interview-questions)

---

## 1. EXPLAIN Deep Dive

### Understanding EXPLAIN Output

```sql
-- ════════════════════════════════════════════════════════════════
-- EXPLAIN vs EXPLAIN ANALYZE
-- ════════════════════════════════════════════════════════════════

-- EXPLAIN: Shows plan without executing (estimates only)
EXPLAIN SELECT * FROM orders WHERE user_id = 123;

-- EXPLAIN ANALYZE: Actually runs query (real timing)
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 123;

-- EXPLAIN with all options (most useful)
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM orders WHERE user_id = 123 AND status = 'active';
```

### Reading the Execution Plan

```sql
-- ════════════════════════════════════════════════════════════════
-- EXAMPLE EXECUTION PLAN
-- ════════════════════════════════════════════════════════════════

EXPLAIN (ANALYZE, BUFFERS)
SELECT o.*, u.email 
FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.status = 'active' AND o.created_at > '2024-01-01';

/*
EXAMPLE OUTPUT:

Hash Join  (cost=1.05..29.35 rows=12 width=200) 
           (actual time=0.025..0.156 rows=15 loops=1)
  Hash Cond: (o.user_id = u.id)
  Buffers: shared hit=8
  ->  Index Scan using idx_orders_status_date on orders o  
      (cost=0.29..28.15 rows=12 width=150) 
      (actual time=0.012..0.089 rows=15 loops=1)
        Index Cond: ((status = 'active') AND (created_at > '2024-01-01'))
        Buffers: shared hit=4
  ->  Hash  (cost=1.05..1.05 rows=5 width=50) 
            (actual time=0.008..0.008 rows=5 loops=1)
        Buckets: 1024  Batches: 1
        ->  Seq Scan on users u  
            (cost=0.00..1.05 rows=5 width=50) 
            (actual time=0.003..0.004 rows=5 loops=1)
              Buffers: shared hit=1
Planning Time: 0.150 ms
Execution Time: 0.189 ms
*/
```

### Decoding Plan Elements

```
EXECUTION PLAN ELEMENTS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SCAN TYPES:                                                    │
│  ───────────                                                    │
│  Seq Scan         → Reading entire table row by row             │
│  Index Scan       → Using index to find rows, then fetch        │
│  Index Only Scan  → All data from index, no table access        │
│  Bitmap Heap Scan → Build bitmap from index, then fetch         │
│                                                                  │
│  JOIN TYPES:                                                    │
│  ───────────                                                    │
│  Nested Loop      → For each row in A, scan B (O(n*m))         │
│  Hash Join        → Build hash of smaller table, probe          │
│  Merge Join       → Both sorted, merge (good for big tables)   │
│                                                                  │
│  COST FORMAT: (startup..total rows=N width=N)                   │
│  ────────────                                                   │
│  startup cost     → Cost before first row returned             │
│  total cost       → Cost to return all rows                    │
│  rows             → Estimated number of rows                   │
│  width            → Average row size in bytes                  │
│                                                                  │
│  ACTUAL FORMAT: (actual time=start..end rows=N loops=N)        │
│  ─────────────                                                  │
│  actual time      → Real time in milliseconds                  │
│  rows             → Actual rows returned                       │
│  loops            → How many times this step ran               │
│                                                                  │
│  RED FLAGS:                                                     │
│  ──────────                                                     │
│  • Seq Scan on large table (>10K rows)                         │
│  • Nested Loop with high loops count                           │
│  • rows estimate very different from actual                    │
│  • Sort or Hash operations spilling to disk                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Cost Analysis

```sql
-- ════════════════════════════════════════════════════════════════
-- UNDERSTANDING COST NUMBERS
-- ════════════════════════════════════════════════════════════════

-- Cost is in arbitrary units (based on seq_page_cost = 1.0)
-- Lower is better, but only compare within same query

-- Seq Scan cost calculation:
-- cost = (pages * seq_page_cost) + (rows * cpu_tuple_cost)
-- For 10000 row table with 100 pages:
-- cost = (100 * 1.0) + (10000 * 0.01) = 200

-- Index Scan cost includes:
-- - Reading index pages (random_page_cost = 4.0)
-- - Reading heap pages for matching rows
-- - Processing tuples

-- COMPARE PLANS:

-- Plan A: Seq Scan (cost=0.00..1000.00)
-- Plan B: Index Scan (cost=0.29..50.00)
-- → Index scan is ~20x cheaper

-- BUT if returning 80% of rows:
-- Plan A: Seq Scan (cost=0.00..1000.00)  ← Actually better!
-- Plan B: Index Scan (cost=0.29..3500.00)  ← Random I/O hurts
```

---

## 2. N+1 Problem

### Understanding N+1

```
THE N+1 PROBLEM VISUALIZED:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  GOAL: Display 100 orders with their items                     │
│                                                                  │
│  N+1 APPROACH (101 queries):                                   │
│  ───────────────────────────                                    │
│  Query 1:   SELECT * FROM orders LIMIT 100                     │
│  Query 2:   SELECT * FROM items WHERE order_id = 1             │
│  Query 3:   SELECT * FROM items WHERE order_id = 2             │
│  Query 4:   SELECT * FROM items WHERE order_id = 3             │
│  ...                                                           │
│  Query 101: SELECT * FROM items WHERE order_id = 100           │
│                                                                  │
│  Time: 101 queries × 5ms = 505ms minimum                       │
│  (Plus connection overhead, network latency)                    │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  OPTIMIZED APPROACH (2 queries):                               │
│  ───────────────────────────────                                │
│  Query 1: SELECT * FROM orders LIMIT 100                       │
│  Query 2: SELECT * FROM items WHERE order_id IN (1,2,3...100) │
│                                                                  │
│  Time: 2 queries × 10ms = 20ms                                 │
│  25x faster!                                                   │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  EVEN BETTER (1 query with JOIN):                              │
│  ──────────────────────────────────                             │
│  Query 1: SELECT o.*, i.*                                      │
│           FROM orders o                                        │
│           LEFT JOIN items i ON o.id = i.order_id               │
│           WHERE o.id IN (1,2,3...100)                         │
│                                                                  │
│  Time: 1 query × 15ms = 15ms                                   │
│  33x faster than N+1!                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### N+1 in Code Examples

```typescript
// ════════════════════════════════════════════════════════════════
// N+1 PROBLEM IN CODE
// ════════════════════════════════════════════════════════════════

// ❌ BAD: N+1 Problem
async function getOrdersWithItems() {
  // Query 1: Get orders
  const orders = await db.query('SELECT * FROM orders LIMIT 100');
  
  // Queries 2-101: Get items for each order (N queries!)
  for (const order of orders) {
    order.items = await db.query(
      'SELECT * FROM items WHERE order_id = $1', 
      [order.id]
    );
  }
  
  return orders;
}
// Total: 101 queries! 🔴

// ════════════════════════════════════════════════════════════════

// ✅ GOOD: Batched queries (2 queries)
async function getOrdersWithItemsBatched() {
  // Query 1: Get orders
  const orders = await db.query('SELECT * FROM orders LIMIT 100');
  const orderIds = orders.map(o => o.id);
  
  // Query 2: Get ALL items in one query
  const items = await db.query(
    'SELECT * FROM items WHERE order_id = ANY($1)',
    [orderIds]
  );
  
  // Group items by order in memory
  const itemsByOrder = items.reduce((acc, item) => {
    acc[item.order_id] = acc[item.order_id] || [];
    acc[item.order_id].push(item);
    return acc;
  }, {});
  
  // Attach items to orders
  for (const order of orders) {
    order.items = itemsByOrder[order.id] || [];
  }
  
  return orders;
}
// Total: 2 queries! 🟢

// ════════════════════════════════════════════════════════════════

// ✅ BEST: Single JOIN query
async function getOrdersWithItemsJoin() {
  const result = await db.query(`
    SELECT 
      o.id as order_id,
      o.status,
      o.total,
      o.created_at,
      i.id as item_id,
      i.name as item_name,
      i.quantity,
      i.price
    FROM orders o
    LEFT JOIN items i ON o.id = i.order_id
    WHERE o.created_at > NOW() - INTERVAL '30 days'
    ORDER BY o.id
    LIMIT 100
  `);
  
  // Transform flat result to nested structure
  const ordersMap = new Map();
  
  for (const row of result.rows) {
    if (!ordersMap.has(row.order_id)) {
      ordersMap.set(row.order_id, {
        id: row.order_id,
        status: row.status,
        total: row.total,
        created_at: row.created_at,
        items: []
      });
    }
    
    if (row.item_id) {
      ordersMap.get(row.order_id).items.push({
        id: row.item_id,
        name: row.item_name,
        quantity: row.quantity,
        price: row.price
      });
    }
  }
  
  return Array.from(ordersMap.values());
}
// Total: 1 query! 🟢🟢
```

### DataLoader Pattern (GraphQL)

```typescript
// ════════════════════════════════════════════════════════════════
// DATALOADER: Batching and Caching within a request
// ════════════════════════════════════════════════════════════════

import DataLoader from 'dataloader';

// Create loader that batches individual loads into one query
const itemsLoader = new DataLoader(async (orderIds: number[]) => {
  // This runs ONCE with all orderIds collected in the event loop tick
  const items = await db.query(
    'SELECT * FROM items WHERE order_id = ANY($1)',
    [orderIds]
  );
  
  // Must return array in same order as input keys
  const itemsByOrder = new Map();
  for (const item of items) {
    const existing = itemsByOrder.get(item.order_id) || [];
    existing.push(item);
    itemsByOrder.set(item.order_id, existing);
  }
  
  return orderIds.map(id => itemsByOrder.get(id) || []);
});

// GraphQL resolver
const resolvers = {
  Order: {
    // Each order calls load() - but DataLoader batches them!
    items: (order) => itemsLoader.load(order.id)
  }
};

// Request for 100 orders:
// - Without DataLoader: 100 separate queries
// - With DataLoader: 1 batched query

// DataLoader also caches within request:
// - First load(1) → query
// - Second load(1) → returns cached result
```

### Detecting N+1 Problems

```typescript
// ════════════════════════════════════════════════════════════════
// DETECTING N+1 IN DEVELOPMENT
// ════════════════════════════════════════════════════════════════

// Method 1: Query logging with count
let queryCount = 0;

const originalQuery = db.query.bind(db);
db.query = async (...args) => {
  queryCount++;
  console.log(`Query #${queryCount}:`, args[0].substring(0, 100));
  return originalQuery(...args);
};

// Run your endpoint
await getOrdersWithItems();
console.log(`Total queries: ${queryCount}`);
// If > 5-10 for a simple request, investigate!

// Method 2: PostgreSQL query logging
// In postgresql.conf:
// log_min_duration_statement = 0  -- Log all queries
// log_statement = 'all'

// Method 3: ORM query logging (Prisma example)
const prisma = new PrismaClient({
  log: ['query', 'info', 'warn', 'error'],
});

// Method 4: Express middleware to count queries per request
app.use((req, res, next) => {
  req.queryCount = 0;
  const originalQuery = db.query.bind(db);
  
  db.query = async (...args) => {
    req.queryCount++;
    return originalQuery(...args);
  };
  
  res.on('finish', () => {
    if (req.queryCount > 10) {
      console.warn(`⚠️ ${req.method} ${req.path}: ${req.queryCount} queries`);
    }
  });
  
  next();
});
```

---

## 3. Eager vs Lazy Loading

### Understanding Loading Strategies

```
LOADING STRATEGIES COMPARISON:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  LAZY LOADING (Load on demand)                                  │
│  ─────────────────────────────                                  │
│                                                                  │
│  const user = await User.findById(1);                          │
│  // Query 1: SELECT * FROM users WHERE id = 1                  │
│                                                                  │
│  const orders = await user.orders;  // Access triggers load    │
│  // Query 2: SELECT * FROM orders WHERE user_id = 1            │
│                                                                  │
│  ✓ Pros: Only loads what you need                              │
│  ✗ Cons: Can cause N+1 if looping                              │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  EAGER LOADING (Load upfront)                                   │
│  ────────────────────────────                                   │
│                                                                  │
│  const user = await User.findById(1, {                         │
│    include: ['orders', 'profile']                              │
│  });                                                           │
│  // Query 1: SELECT u.*, o.*, p.*                              │
│  //          FROM users u                                      │
│  //          LEFT JOIN orders o ON u.id = o.user_id            │
│  //          LEFT JOIN profiles p ON u.id = p.user_id          │
│  //          WHERE u.id = 1                                    │
│                                                                  │
│  ✓ Pros: Single query, no N+1 risk                             │
│  ✗ Cons: May load data you don't need                          │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  EXPLICIT LOADING (Manual control)                              │
│  ─────────────────────────────────                              │
│                                                                  │
│  const user = await User.findById(1);                          │
│  // Later, explicitly load if needed:                          │
│  if (needOrders) {                                             │
│    await user.loadRelation('orders');                          │
│  }                                                              │
│                                                                  │
│  ✓ Pros: Full control                                          │
│  ✗ Cons: More code, easy to forget                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When to Use Each Strategy

```typescript
// ════════════════════════════════════════════════════════════════
// EAGER LOADING: Use when you KNOW you'll need related data
// ════════════════════════════════════════════════════════════════

// ✅ Good: Always display user with their orders on dashboard
async function getUserDashboard(userId: string) {
  return await prisma.user.findUnique({
    where: { id: userId },
    include: {
      orders: {
        take: 10,
        orderBy: { createdAt: 'desc' }
      },
      profile: true
    }
  });
}

// ✅ Good: API returns user with related data
app.get('/api/users/:id', async (req, res) => {
  const user = await prisma.user.findUnique({
    where: { id: req.params.id },
    include: {
      orders: true,
      addresses: true
    }
  });
  res.json(user);
});

// ════════════════════════════════════════════════════════════════
// LAZY LOADING: Use when related data is conditional
// ════════════════════════════════════════════════════════════════

// ✅ Good: Only load orders if user clicks "View Orders"
async function getUser(userId: string) {
  return await prisma.user.findUnique({
    where: { id: userId }
  });
}

async function getUserOrders(userId: string) {
  return await prisma.order.findMany({
    where: { userId }
  });
}

// ✅ Good: GraphQL - client requests what it needs
const resolvers = {
  User: {
    // Only runs if client requests 'orders' field
    orders: (user) => prisma.order.findMany({ 
      where: { userId: user.id } 
    })
  }
};

// ════════════════════════════════════════════════════════════════
// HYBRID: Different strategies for different use cases
// ════════════════════════════════════════════════════════════════

// List view: minimal data (lazy/no relations)
async function getUserList() {
  return await prisma.user.findMany({
    select: {
      id: true,
      name: true,
      email: true
      // No relations!
    }
  });
}

// Detail view: full data (eager relations)
async function getUserDetail(id: string) {
  return await prisma.user.findUnique({
    where: { id },
    include: {
      orders: true,
      profile: true,
      addresses: true
    }
  });
}
```

### ORM-Specific Eager Loading

```typescript
// ════════════════════════════════════════════════════════════════
// PRISMA: include and select
// ════════════════════════════════════════════════════════════════

// Include related models
const user = await prisma.user.findUnique({
  where: { id: 1 },
  include: {
    orders: true,                    // All orders
    orders: { take: 5 },             // Limited
    orders: {                        // Nested
      include: { items: true }
    }
  }
});

// Select specific fields only
const user = await prisma.user.findUnique({
  where: { id: 1 },
  select: {
    id: true,
    name: true,
    orders: {
      select: {
        id: true,
        total: true
      }
    }
  }
});

// ════════════════════════════════════════════════════════════════
// TYPEORM: relations and query builder
// ════════════════════════════════════════════════════════════════

// Using relations option
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ['orders', 'orders.items', 'profile']
});

// Using query builder (more control)
const user = await userRepository
  .createQueryBuilder('user')
  .leftJoinAndSelect('user.orders', 'order')
  .leftJoinAndSelect('order.items', 'item')
  .where('user.id = :id', { id: 1 })
  .getOne();

// ════════════════════════════════════════════════════════════════
// SEQUELIZE: include option
// ════════════════════════════════════════════════════════════════

const user = await User.findByPk(1, {
  include: [
    { model: Order, as: 'orders' },
    { 
      model: Order, 
      as: 'orders',
      include: [{ model: Item, as: 'items' }]  // Nested
    }
  ]
});

// Eager loading for findAll
const users = await User.findAll({
  include: [{ model: Order }]
});

// ════════════════════════════════════════════════════════════════
// DRIZZLE: with() for relations
// ════════════════════════════════════════════════════════════════

const user = await db.query.users.findFirst({
  where: eq(users.id, 1),
  with: {
    orders: true,
    profile: true
  }
});
```

---

## 4. Query Patterns & Anti-Patterns

### Anti-Pattern: SELECT *

```sql
-- ════════════════════════════════════════════════════════════════
-- ANTI-PATTERN: SELECT *
-- ════════════════════════════════════════════════════════════════

-- ❌ BAD: Fetches all 50 columns including large TEXT/BLOB
SELECT * FROM products WHERE category_id = 5;
-- Returns: id, name, description (10KB), specs (5KB), images (JSON)...
-- Network: 15KB × 1000 rows = 15MB transfer

-- ✅ GOOD: Only columns you need
SELECT id, name, price, thumbnail_url 
FROM products 
WHERE category_id = 5;
-- Network: 200 bytes × 1000 rows = 200KB transfer
-- 75x less data!

-- ════════════════════════════════════════════════════════════════
-- WHY SELECT * IS PROBLEMATIC
-- ════════════════════════════════════════════════════════════════

-- 1. More data transfer (network bandwidth)
-- 2. More memory usage (application RAM)
-- 3. Prevents covering index optimization
-- 4. Breaks if columns are added/renamed
-- 5. Exposes sensitive columns accidentally
```

### Anti-Pattern: Correlated Subquery

```sql
-- ════════════════════════════════════════════════════════════════
-- ANTI-PATTERN: Correlated Subquery (runs for EACH row)
-- ════════════════════════════════════════════════════════════════

-- ❌ BAD: Subquery executes once per order (N+1 in SQL!)
SELECT 
    o.id,
    o.total,
    (SELECT COUNT(*) FROM items i WHERE i.order_id = o.id) as item_count,
    (SELECT SUM(i.price) FROM items i WHERE i.order_id = o.id) as items_total
FROM orders o
WHERE o.user_id = 123;
-- For 100 orders: 1 + 100×2 = 201 subquery executions!

-- ✅ GOOD: Use JOIN with aggregation
SELECT 
    o.id,
    o.total,
    COUNT(i.id) as item_count,
    COALESCE(SUM(i.price), 0) as items_total
FROM orders o
LEFT JOIN items i ON o.id = i.order_id
WHERE o.user_id = 123
GROUP BY o.id, o.total;
-- Single pass through both tables!

-- ════════════════════════════════════════════════════════════════
-- ANTI-PATTERN: Subquery in WHERE for existence check
-- ════════════════════════════════════════════════════════════════

-- ❌ BAD: Subquery for each user
SELECT * FROM users u
WHERE (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) > 0;

-- ✅ GOOD: Use EXISTS (stops at first match)
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- ✅ ALSO GOOD: Use JOIN
SELECT DISTINCT u.* FROM users u
JOIN orders o ON u.id = o.user_id;
```

### Anti-Pattern: OR Conditions

```sql
-- ════════════════════════════════════════════════════════════════
-- ANTI-PATTERN: OR conditions that prevent index use
-- ════════════════════════════════════════════════════════════════

-- ❌ BAD: OR on different columns - hard to optimize
SELECT * FROM users 
WHERE email = 'john@example.com' OR phone = '555-1234';
-- May require scanning both indexes and merging, or seq scan

-- ✅ GOOD: UNION of separate queries (each uses its index)
SELECT * FROM users WHERE email = 'john@example.com'
UNION
SELECT * FROM users WHERE phone = '555-1234';

-- ✅ ALTERNATIVE: Create combined index if this is common
CREATE INDEX idx_users_email_phone ON users(email, phone);

-- ════════════════════════════════════════════════════════════════
-- ANTI-PATTERN: OR with same column (usually okay)
-- ════════════════════════════════════════════════════════════════

-- This is actually fine - can use index
SELECT * FROM users WHERE status = 'active' OR status = 'pending';

-- But better written as:
SELECT * FROM users WHERE status IN ('active', 'pending');
```

### Pattern: Pagination

```sql
-- ════════════════════════════════════════════════════════════════
-- PAGINATION STRATEGIES
-- ════════════════════════════════════════════════════════════════

-- ❌ BAD: OFFSET pagination (gets slower as offset increases)
SELECT * FROM orders ORDER BY created_at DESC LIMIT 20 OFFSET 10000;
-- Must scan and discard 10000 rows before returning 20!
-- Page 500 takes 500x longer than page 1

-- ✅ GOOD: Cursor-based pagination (consistent performance)
-- First page:
SELECT * FROM orders 
ORDER BY created_at DESC, id DESC 
LIMIT 20;

-- Next pages (use last row's values as cursor):
SELECT * FROM orders 
WHERE (created_at, id) < ('2024-01-15 10:30:00', 12345)
ORDER BY created_at DESC, id DESC 
LIMIT 20;
-- Uses index, no scanning of skipped rows!

-- ════════════════════════════════════════════════════════════════
-- CURSOR PAGINATION IMPLEMENTATION
-- ════════════════════════════════════════════════════════════════

async function getOrders(cursor?: string, limit = 20) {
  let query = db.select()
    .from(orders)
    .orderBy(desc(orders.createdAt), desc(orders.id))
    .limit(limit + 1);  // Fetch one extra to check if more
  
  if (cursor) {
    const [timestamp, id] = decodeCursor(cursor);
    query = query.where(
      or(
        lt(orders.createdAt, timestamp),
        and(
          eq(orders.createdAt, timestamp),
          lt(orders.id, id)
        )
      )
    );
  }
  
  const results = await query;
  const hasMore = results.length > limit;
  const items = hasMore ? results.slice(0, -1) : results;
  
  return {
    items,
    nextCursor: hasMore 
      ? encodeCursor(items[items.length - 1]) 
      : null
  };
}
```

### Pattern: Conditional Aggregation

```sql
-- ════════════════════════════════════════════════════════════════
-- CONDITIONAL AGGREGATION (avoid multiple queries)
-- ════════════════════════════════════════════════════════════════

-- ❌ BAD: Multiple queries for dashboard stats
SELECT COUNT(*) FROM orders WHERE status = 'pending';
SELECT COUNT(*) FROM orders WHERE status = 'shipped';
SELECT COUNT(*) FROM orders WHERE status = 'delivered';
SELECT SUM(total) FROM orders WHERE status = 'delivered';

-- ✅ GOOD: Single query with conditional aggregation
SELECT 
    COUNT(*) FILTER (WHERE status = 'pending') as pending_count,
    COUNT(*) FILTER (WHERE status = 'shipped') as shipped_count,
    COUNT(*) FILTER (WHERE status = 'delivered') as delivered_count,
    SUM(total) FILTER (WHERE status = 'delivered') as delivered_total
FROM orders;

-- MySQL syntax (no FILTER):
SELECT 
    SUM(CASE WHEN status = 'pending' THEN 1 ELSE 0 END) as pending_count,
    SUM(CASE WHEN status = 'shipped' THEN 1 ELSE 0 END) as shipped_count,
    SUM(CASE WHEN status = 'delivered' THEN 1 ELSE 0 END) as delivered_count,
    SUM(CASE WHEN status = 'delivered' THEN total ELSE 0 END) as delivered_total
FROM orders;
```

---

## 5. ORM Optimization

### Prisma Optimization

```typescript
// ════════════════════════════════════════════════════════════════
// PRISMA: Common optimizations
// ════════════════════════════════════════════════════════════════

// 1. Use select instead of include when possible
// ❌ Fetches all user fields
const users = await prisma.user.findMany({
  include: { orders: true }
});

// ✅ Fetches only needed fields
const users = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    orders: {
      select: { id: true, total: true }
    }
  }
});

// 2. Use findMany with where IN instead of loops
// ❌ N queries
for (const id of userIds) {
  await prisma.user.findUnique({ where: { id } });
}

// ✅ 1 query
await prisma.user.findMany({
  where: { id: { in: userIds } }
});

// 3. Use transactions for multiple operations
// ❌ Multiple round trips
await prisma.user.update({ where: { id: 1 }, data: { balance: 100 } });
await prisma.order.create({ data: { userId: 1, total: 50 } });

// ✅ Single transaction
await prisma.$transaction([
  prisma.user.update({ where: { id: 1 }, data: { balance: 100 } }),
  prisma.order.create({ data: { userId: 1, total: 50 } })
]);

// 4. Use raw queries for complex operations
const result = await prisma.$queryRaw`
  SELECT u.*, COUNT(o.id) as order_count
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  GROUP BY u.id
  HAVING COUNT(o.id) > 5
`;

// 5. Enable query logging for debugging
const prisma = new PrismaClient({
  log: [
    { level: 'query', emit: 'event' }
  ]
});

prisma.$on('query', (e) => {
  console.log(`Query: ${e.query}`);
  console.log(`Duration: ${e.duration}ms`);
});
```

### Query Builder Best Practices

```typescript
// ════════════════════════════════════════════════════════════════
// KNEX/DRIZZLE: Building efficient queries
// ════════════════════════════════════════════════════════════════

// 1. Batch inserts
// ❌ N insert queries
for (const item of items) {
  await db.insert(items).values(item);
}

// ✅ Single batch insert
await db.insert(items).values(itemsArray);
// Or with chunking for large datasets
const chunks = chunkArray(itemsArray, 1000);
for (const chunk of chunks) {
  await db.insert(items).values(chunk);
}

// 2. Use specific columns, not *
// ❌ SELECT *
await db.select().from(users);

// ✅ SELECT specific columns
await db.select({
  id: users.id,
  name: users.name
}).from(users);

// 3. Efficient counting
// ❌ Fetches all rows then counts in JS
const allUsers = await db.select().from(users);
const count = allUsers.length;

// ✅ Count in database
const [{ count }] = await db
  .select({ count: sql`count(*)` })
  .from(users);

// 4. Upsert pattern
await db.insert(users)
  .values({ id: 1, name: 'John', email: 'john@ex.com' })
  .onConflictDoUpdate({
    target: users.id,
    set: { name: 'John', email: 'john@ex.com' }
  });
```

---

## 6. Query Analysis Tools

### PostgreSQL Tools

```sql
-- ════════════════════════════════════════════════════════════════
-- pg_stat_statements: Find slow queries
-- ════════════════════════════════════════════════════════════════

-- Enable extension
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Find slowest queries by total time
SELECT 
    substring(query, 1, 100) as short_query,
    calls,
    round(total_exec_time::numeric, 2) as total_ms,
    round(mean_exec_time::numeric, 2) as avg_ms,
    rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;

-- Find queries by frequency (most called)
SELECT 
    substring(query, 1, 100) as short_query,
    calls,
    round(mean_exec_time::numeric, 2) as avg_ms
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 20;

-- Reset stats (after fixing issues)
SELECT pg_stat_statements_reset();

-- ════════════════════════════════════════════════════════════════
-- pg_stat_user_tables: Table statistics
-- ════════════════════════════════════════════════════════════════

-- Find tables with most sequential scans (missing indexes)
SELECT 
    relname as table_name,
    seq_scan,
    seq_tup_read,
    idx_scan,
    n_live_tup as row_count
FROM pg_stat_user_tables
WHERE seq_scan > 0
ORDER BY seq_scan DESC
LIMIT 10;

-- ════════════════════════════════════════════════════════════════
-- Auto-explain: Log slow query plans
-- ════════════════════════════════════════════════════════════════

-- In postgresql.conf:
-- shared_preload_libraries = 'auto_explain'
-- auto_explain.log_min_duration = '100ms'  -- Log plans for queries > 100ms
-- auto_explain.log_analyze = true
-- auto_explain.log_buffers = true
```

### Application-Level Monitoring

```typescript
// ════════════════════════════════════════════════════════════════
// QUERY PERFORMANCE MONITORING
// ════════════════════════════════════════════════════════════════

// Middleware to track slow queries
class QueryMonitor {
  private slowThreshold = 100; // ms
  
  async trackQuery<T>(
    name: string,
    queryFn: () => Promise<T>
  ): Promise<T> {
    const start = performance.now();
    
    try {
      const result = await queryFn();
      const duration = performance.now() - start;
      
      if (duration > this.slowThreshold) {
        console.warn(`🐢 Slow query: ${name} took ${duration.toFixed(2)}ms`);
        // Send to monitoring service
        metrics.recordSlowQuery(name, duration);
      }
      
      return result;
    } catch (error) {
      const duration = performance.now() - start;
      metrics.recordFailedQuery(name, duration, error);
      throw error;
    }
  }
}

// Usage
const monitor = new QueryMonitor();

const users = await monitor.trackQuery(
  'getActiveUsers',
  () => prisma.user.findMany({ where: { active: true } })
);

// ════════════════════════════════════════════════════════════════
// QUERY LOGGING WITH EXPLAIN (development only!)
// ════════════════════════════════════════════════════════════════

// Automatically EXPLAIN slow queries
async function queryWithExplain<T>(
  sql: string,
  params: any[]
): Promise<T> {
  const start = performance.now();
  const result = await db.query(sql, params);
  const duration = performance.now() - start;
  
  if (duration > 100 && process.env.NODE_ENV === 'development') {
    const explain = await db.query(`EXPLAIN ANALYZE ${sql}`, params);
    console.log('Slow query explain plan:', explain.rows);
  }
  
  return result;
}
```

---

## 7. Common Pitfalls

```
QUERY OPTIMIZATION PITFALLS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. PREMATURE OPTIMIZATION                                      │
│     ─────────────────────                                       │
│     Problem: Optimizing queries that aren't slow               │
│     Solution: Measure first! Use pg_stat_statements            │
│                                                                  │
│  2. OVER-EAGER LOADING                                          │
│     ────────────────────                                        │
│     Problem: Always eager load "just in case"                  │
│     Solution: Profile actual usage, load only what's needed    │
│                                                                  │
│  3. NOT USING EXPLAIN                                           │
│     ────────────────────                                        │
│     Problem: Guessing why query is slow                        │
│     Solution: EXPLAIN ANALYZE every suspect query              │
│                                                                  │
│  4. IGNORING STATISTICS                                         │
│     ─────────────────────                                       │
│     Problem: Optimizer makes bad choices                       │
│     Solution: Run ANALYZE after major data changes             │
│                                                                  │
│  5. N+1 IN BATCH JOBS                                          │
│     ────────────────────                                        │
│     Problem: Loop queries in background jobs                   │
│     Solution: Use bulk operations, batch queries               │
│                                                                  │
│  6. FETCHING LARGE DATASETS TO APP                             │
│     ─────────────────────────────                               │
│     Problem: Filter/aggregate in application code              │
│     Solution: Let database do filtering and aggregation        │
│                                                                  │
│  7. WRONG PAGINATION                                            │
│     ─────────────────                                           │
│     Problem: OFFSET 10000 - very slow                          │
│     Solution: Cursor-based pagination                          │
│                                                                  │
│  8. UNBOUNDED QUERIES                                           │
│     ────────────────────                                        │
│     Problem: SELECT without LIMIT on unknown data size         │
│     Solution: Always have LIMIT, even if high (10000)          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. Interview Questions

### Conceptual Questions

**Q: "What is the N+1 problem and how do you solve it?"**
> "N+1 occurs when you fetch N items, then make N additional queries for related data. For 100 orders with items: 1 query for orders + 100 queries for items = 101 queries. 
>
> Solutions:
> 1. **Eager loading** - JOIN or include in ORM
> 2. **Batch loading** - WHERE id IN (...) for all IDs at once
> 3. **DataLoader** - for GraphQL, batches within event loop tick
>
> The fix usually takes 101 queries down to 1-2."

**Q: "When would you use lazy loading vs eager loading?"**
> "**Eager loading** when you know you'll need the related data - prevents N+1, fewer round trips. Good for list views that always show relations.
>
> **Lazy loading** when related data is conditional - might not need it. Good for detail views where user might not expand all sections.
>
> The key is understanding your access patterns. Wrong choice: eager = over-fetching, lazy = N+1."

**Q: "How do you read an EXPLAIN ANALYZE output?"**
> "I look for:
> 1. **Scan type** - Seq Scan on big table = red flag, need index
> 2. **Cost numbers** - Higher = more work
> 3. **Rows: estimated vs actual** - Big difference = stale statistics
> 4. **Loops** - High loop count in nested loop = potential issue
> 5. **Buffers** - shared read = disk I/O, shared hit = cache hit
>
> Then I identify the slowest node and optimize that first."

### Scenario Questions

**Q: "An API endpoint is slow. How do you debug it?"**
> "Step by step:
> 1. **Measure** - Add timing logs around DB calls
> 2. **Count queries** - Log query count per request (N+1?)
> 3. **EXPLAIN ANALYZE** - On slow queries
> 4. **Check pg_stat_statements** - Is this query pattern slow?
> 5. **Profile** - Is time in DB or application?
>
> Usually it's N+1 (fix with eager loading), missing index (add index), or fetching too much data (use SELECT specific columns)."

**Q: "Dashboard shows stats from multiple tables. How would you optimize?"**
> "Use conditional aggregation in a single query:
> ```sql
> SELECT 
>   COUNT(*) FILTER (WHERE status = 'active') as active,
>   COUNT(*) FILTER (WHERE status = 'pending') as pending,
>   SUM(amount) FILTER (WHERE status = 'paid') as revenue
> FROM orders;
> ```
> One query instead of 3+. If still slow, consider:
> - Materialized view refreshed periodically
> - Redis cache with TTL
> - Background job updating stats table"

**Q: "Query uses index but is still slow. What could be wrong?"**
> "Several possibilities:
> 1. **Index scan returning many rows** - selectivity too low
> 2. **Not a covering index** - many heap fetches
> 3. **Index bloat** - needs REINDEX
> 4. **Cold cache** - first run after restart
> 5. **Lock contention** - check pg_stat_activity
> 6. **Large result set** - add LIMIT or pagination
> 7. **Network latency** - many round trips (N+1)
>
> Check EXPLAIN ANALYZE buffers and actual vs estimated rows."

### Quick Fire

| Question | Answer |
|----------|--------|
| "N+1 with 100 items?" | "101 queries - 1 + N for relations" |
| "Fix for SELECT *?" | "Select only needed columns" |
| "Cursor vs offset pagination?" | "Cursor = consistent O(1), offset = O(n) as page increases" |
| "ORM causes N+1?" | "Enable query logging, use eager loading/includes" |
| "EXPLAIN shows seq scan?" | "Add index on WHERE columns, run ANALYZE" |

---

## Quick Reference

```
QUERY OPTIMIZATION CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  N+1 PROBLEM:                                                   │
│  • Symptom: 101 queries for 100 items                          │
│  • Fix: Eager loading, JOIN, DataLoader                        │
│  • Detect: Query count logging, slow response                  │
│                                                                  │
│  LOADING STRATEGIES:                                            │
│  • Eager: Always need relations → include/JOIN                 │
│  • Lazy: Might need relations → load on access                 │
│  • Explicit: Full control → manual load when needed            │
│                                                                  │
│  EXPLAIN ANALYSIS:                                              │
│  • Seq Scan (big table) = need index                           │
│  • High cost = expensive operation                             │
│  • Est vs actual rows differ = run ANALYZE                     │
│  • Nested loop + high loops = consider hash/merge join         │
│                                                                  │
│  ANTI-PATTERNS:                                                 │
│  • SELECT * → Select specific columns                          │
│  • Correlated subquery → JOIN with GROUP BY                    │
│  • OFFSET 10000 → Cursor pagination                            │
│  • Loop queries → Batch/bulk operations                        │
│  • OR on different columns → UNION                             │
│                                                                  │
│  TOOLS:                                                         │
│  • EXPLAIN ANALYZE - see real execution                        │
│  • pg_stat_statements - find slow queries                      │
│  • Query logging - count queries per request                   │
│  • auto_explain - log plans for slow queries                   │
│                                                                  │
│  QUICK WINS:                                                    │
│  1. Fix N+1 (biggest impact usually)                           │
│  2. Add missing indexes                                        │
│  3. Limit result sets                                          │
│  4. Select only needed columns                                 │
│  5. Use pagination                                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

*Guide covers patterns applicable to PostgreSQL, MySQL, and common ORMs (Prisma, TypeORM, Sequelize, Drizzle).*


