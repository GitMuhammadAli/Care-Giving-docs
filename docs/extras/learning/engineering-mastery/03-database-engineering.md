# Chapter 03: Database Engineering

> "The database is the heart of most applications. Understand it deeply."

---

## 🎯 Database Types

### Relational (SQL)

```
┌─────────────────────────────────────────────────────────┐
│ PostgreSQL, MySQL, SQLite                               │
│                                                         │
│ Structure: Tables with rows and columns                 │
│ Relationships: Foreign keys, JOINs                      │
│ Guarantees: ACID transactions                           │
│                                                         │
│ Best for:                                               │
│ - Complex queries                                       │
│ - Transactions (banking, e-commerce)                    │
│ - Data integrity is critical                            │
└─────────────────────────────────────────────────────────┘
```

### Document (NoSQL)

```
┌─────────────────────────────────────────────────────────┐
│ MongoDB, CouchDB                                        │
│                                                         │
│ Structure: JSON-like documents                          │
│ Schema: Flexible, per-document                          │
│                                                         │
│ {                                                       │
│   "_id": "user123",                                     │
│   "name": "John",                                       │
│   "orders": [                                           │
│     {"id": 1, "total": 100},                            │
│     {"id": 2, "total": 200}                             │
│   ]                                                     │
│ }                                                       │
│                                                         │
│ Best for:                                               │
│ - Rapid prototyping                                     │
│ - Varying data structures                               │
│ - Embedded documents (denormalized)                     │
└─────────────────────────────────────────────────────────┘
```

### Key-Value

```
┌─────────────────────────────────────────────────────────┐
│ Redis, DynamoDB, Memcached                              │
│                                                         │
│ Structure: Simple key → value pairs                     │
│                                                         │
│ SET user:123 "John Doe"                                 │
│ GET user:123 → "John Doe"                               │
│                                                         │
│ Best for:                                               │
│ - Caching                                               │
│ - Session storage                                       │
│ - Real-time leaderboards                                │
│ - Rate limiting                                         │
└─────────────────────────────────────────────────────────┘
```

### Wide-Column

```
┌─────────────────────────────────────────────────────────┐
│ Cassandra, HBase, ScyllaDB                              │
│                                                         │
│ Row Key │ Column Family: Profile    │ Column Family: Activity │
│ ────────┼──────────────────────────┼───────────────────────│
│ user123 │ name:John, age:30         │ login:2024-01-01      │
│                                                         │
│ Best for:                                               │
│ - Time-series data                                      │
│ - Write-heavy workloads                                 │
│ - Horizontal scaling                                    │
└─────────────────────────────────────────────────────────┘
```

### Graph

```
┌─────────────────────────────────────────────────────────┐
│ Neo4j, Amazon Neptune                                   │
│                                                         │
│      (Alice)──FRIENDS──(Bob)                            │
│         │                │                              │
│      LIKES            WORKS_AT                          │
│         │                │                              │
│      (Post1)         (Company)                          │
│                                                         │
│ Best for:                                               │
│ - Social networks                                       │
│ - Recommendation engines                                │
│ - Fraud detection                                       │
│ - Knowledge graphs                                      │
└─────────────────────────────────────────────────────────┘
```

---

## 🔐 ACID Properties

```
┌─────────────────────────────────────────────────────────────────┐
│ Atomicity:                                                      │
│   All operations succeed or all fail                            │
│   Transfer $100: Debit AND Credit, never just one               │
├─────────────────────────────────────────────────────────────────┤
│ Consistency:                                                    │
│   Database moves from valid state to valid state                │
│   Constraints (FK, unique) always enforced                      │
├─────────────────────────────────────────────────────────────────┤
│ Isolation:                                                      │
│   Concurrent transactions don't interfere                       │
│   Each transaction sees consistent snapshot                     │
├─────────────────────────────────────────────────────────────────┤
│ Durability:                                                     │
│   Committed transactions survive crashes                        │
│   Written to disk, not just memory                              │
└─────────────────────────────────────────────────────────────────┘
```

### Transaction Isolation Levels

```
Level              │ Dirty Read │ Non-Repeatable │ Phantom Read
───────────────────┼────────────┼────────────────┼─────────────
Read Uncommitted   │    Yes     │      Yes       │     Yes
Read Committed     │    No      │      Yes       │     Yes
Repeatable Read    │    No      │      No        │     Yes
Serializable       │    No      │      No        │     No

Higher isolation = More correct, but slower
PostgreSQL default: Read Committed
```

**Problems Explained:**
```
Dirty Read:
  T1: UPDATE balance = 100 (not committed)
  T2: SELECT balance → sees 100 (uncommitted data!)
  T1: ROLLBACK
  T2 read data that never existed

Non-Repeatable Read:
  T1: SELECT balance → 100
  T2: UPDATE balance = 200, COMMIT
  T1: SELECT balance → 200 (different!)

Phantom Read:
  T1: SELECT COUNT(*) WHERE age > 30 → 5
  T2: INSERT user (age: 35), COMMIT
  T1: SELECT COUNT(*) WHERE age > 30 → 6 (new row appeared!)
```

---

## 📊 Indexing Deep Dive

### How Indexes Work

```
Without index (Full table scan):
┌─────────────────────────────────────────┐
│ Scan ALL rows to find user_id = 1000    │
│ Time: O(n) - 1 million rows = 1M checks │
└─────────────────────────────────────────┘

With index (B-tree):
                    [500]
                   /     \
              [250]       [750]
             /    \       /    \
         [100]  [400] [600]  [900]
         
│ Jump directly to user_id = 1000         │
│ Time: O(log n) - 1M rows = ~20 checks   │
```

### B-Tree vs B+Tree

```
B-Tree (used in most DBs):
- Data stored in all nodes
- Good for: Single lookups

B+Tree (PostgreSQL, MySQL):
- Data only in leaf nodes
- Leaves linked together
- Good for: Range queries (WHERE age > 30)

         [50│100]           ← Internal nodes (keys only)
        /    |    \
   [10,20] [60,80] [110,120]  ← Leaf nodes (keys + data)
      ↔       ↔        ↔      ← Linked for range scans
```

### Index Types

```sql
-- 1. B-Tree (default, most common)
CREATE INDEX idx_user_email ON users(email);
-- Good for: =, <, >, BETWEEN, LIKE 'prefix%'

-- 2. Hash Index
CREATE INDEX idx_user_id ON users USING HASH(id);
-- Good for: = only (not range queries)
-- Faster for exact matches

-- 3. GIN (Generalized Inverted Index)
CREATE INDEX idx_tags ON posts USING GIN(tags);
-- Good for: Arrays, JSONB, full-text search

-- 4. GiST (Generalized Search Tree)
CREATE INDEX idx_location ON places USING GiST(location);
-- Good for: Geometric data, ranges

-- 5. BRIN (Block Range Index)
CREATE INDEX idx_created ON logs USING BRIN(created_at);
-- Good for: Large tables with naturally ordered data
-- Much smaller than B-tree (1000x)
```

### Composite Indexes

```sql
-- Order matters!
CREATE INDEX idx_name ON users(last_name, first_name);

-- This index helps:
WHERE last_name = 'Smith'                    ✓
WHERE last_name = 'Smith' AND first_name = 'John' ✓
WHERE first_name = 'John'                    ✗ (can't use index)

-- Think of it like a phone book:
-- Sorted by last name, then first name
-- Can't look up by first name alone
```

### Index Anti-Patterns

```sql
-- 1. Over-indexing (slow writes, wasted space)
-- Every index slows INSERT/UPDATE/DELETE

-- 2. Function on indexed column
WHERE LOWER(email) = 'test@test.com'  -- Can't use index!
-- Fix: Create expression index
CREATE INDEX idx_email_lower ON users(LOWER(email));

-- 3. Wrong column order in composite index
INDEX(a, b, c) with query WHERE b = 1  -- Can't use!

-- 4. Low selectivity columns
INDEX(is_active)  -- Only 2 values, not useful

-- 5. LIKE with leading wildcard
WHERE email LIKE '%@gmail.com'  -- Full scan!
```

---

## 🔄 Replication

### Primary-Replica (Master-Slave)

```
                    ┌─────────────┐
     Writes ───────►│   PRIMARY   │
                    │   (Master)  │
                    └──────┬──────┘
                           │ Replication
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ REPLICA  │ │ REPLICA  │ │ REPLICA  │
        │   (R1)   │ │   (R2)   │ │   (R3)   │
        └──────────┘ └──────────┘ └──────────┘
              ▲            ▲            ▲
              └────────────┴────────────┘
                      All Reads

Benefits:
- Read scaling (add more replicas)
- High availability (promote replica if primary fails)
- Geographic distribution (replica in each region)

Challenges:
- Replication lag (reads might be stale)
- Failover complexity
- Write bottleneck (single primary)
```

### Synchronous vs Asynchronous Replication

```
Synchronous:
Primary ──write──► Replica ──ack──► Primary ──ack──► Client
                                         └─ Wait for all replicas
Pros: No data loss
Cons: Slower, availability depends on replicas

Asynchronous:
Primary ──write──► Primary ──ack──► Client
         └──────────────────────► Replica (background)
         
Pros: Fast, primary doesn't wait
Cons: Potential data loss if primary crashes
```

### Multi-Primary (Multi-Master)

```
┌─────────────┐     ┌─────────────┐
│  PRIMARY 1  │◄───►│  PRIMARY 2  │
│  (Region A) │     │  (Region B) │
└─────────────┘     └─────────────┘
      ▲                   ▲
      │                   │
   Writes              Writes
   from A              from B
   
Challenges:
- Conflict resolution (same row updated in both)
- More complex
- Used by: CockroachDB, Cassandra
```

---

## 🔀 Sharding (Partitioning)

### Why Shard?

```
Single database limits:
- Storage: Can't fit 100TB on one machine
- Write throughput: One machine = one disk
- Query performance: Huge tables are slow
```

### Sharding Strategies

**1. Range-Based Sharding:**
```
┌─────────────────────────────────────────────────────┐
│ Shard 1: users with ID 1 - 1,000,000              │
│ Shard 2: users with ID 1,000,001 - 2,000,000      │
│ Shard 3: users with ID 2,000,001 - 3,000,000      │
└─────────────────────────────────────────────────────┘

Pros: Simple, range queries are efficient
Cons: Hotspots (new users all go to last shard)
```

**2. Hash-Based Sharding:**
```
shard_id = hash(user_id) % num_shards

user_id = 12345
hash(12345) = 7823456
7823456 % 4 = 0 → Shard 0

Pros: Even distribution
Cons: Range queries require all shards
```

**3. Consistent Hashing:**
```
Hash ring (0 to 2^32):

         0
       /   \
    Shard D   Shard A
      │         │
  270°─┼─────────┼─ 90°
      │         │
    Shard C   Shard B
       \   /
       180°
       
user_123 → hash → position on ring → nearest shard clockwise

Adding/removing shard only affects neighbors
```

**4. Directory-Based Sharding:**
```
┌─────────────────────────────────────────┐
│         Lookup Service                  │
│  user_123 → Shard 2                     │
│  user_456 → Shard 1                     │
│  user_789 → Shard 3                     │
└─────────────────────────────────────────┘

Pros: Flexible, can move users between shards
Cons: Lookup service is SPOF, additional latency
```

### Sharding Challenges

```
1. Cross-shard queries:
   "Get all orders for users in California"
   → Must query ALL shards!
   
2. Transactions across shards:
   "Transfer money between two users on different shards"
   → Need distributed transactions (complex!)
   
3. Rebalancing:
   Adding new shard → need to redistribute data
   
4. Consistent ID generation:
   Can't use auto-increment (would conflict)
   → Use UUIDs, Snowflake IDs
```

---

## 🆔 ID Generation Strategies

### Auto-Increment (Simple, not scalable)
```sql
-- Works for single database
CREATE TABLE users (
  id SERIAL PRIMARY KEY
);

-- Problem: Multiple databases = ID conflicts
DB1: 1, 2, 3, 4...
DB2: 1, 2, 3, 4... -- Collision!
```

### UUID (Universally Unique)
```
550e8400-e29b-41d4-a716-446655440000

Pros:
- No coordination needed
- Generate anywhere

Cons:
- 128 bits (16 bytes) - larger indexes
- Not sortable by time
- Bad for B-tree (random, causes page splits)
```

### Snowflake ID (Twitter's solution)
```
64 bits total:
┌─────────────────────────────────────────────────────────────┐
│ 1 bit │  41 bits timestamp  │ 10 bits machine │ 12 bits seq │
│  (0)  │  (69 years of ms)   │  (1024 machines) │  (4096/ms) │
└─────────────────────────────────────────────────────────────┘

Example: 1382971839452749824

Pros:
- 64 bits (half of UUID)
- Time-sortable
- Unique across machines
- 4 million IDs per second per machine

Used by: Twitter, Discord, Instagram
```

### ULID (Universally Unique Lexicographically Sortable)
```
01ARZ3NDEKTSV4RRFFQ69G5FAV
├──────────┬──────────────┤
│ timestamp│   randomness │
│ (48 bits)│   (80 bits)  │

Pros:
- Lexicographically sortable (works with string comparison)
- Case insensitive
- URL safe
```

---

## 📈 Query Optimization

### EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@test.com';

-- Output:
Seq Scan on users  (cost=0.00..35.50 rows=1 width=100)
  Filter: (email = 'test@test.com')
  Rows Removed by Filter: 999
  Planning Time: 0.1 ms
  Execution Time: 5.2 ms

-- vs with index:
Index Scan using idx_email on users  (cost=0.42..8.44 rows=1 width=100)
  Index Cond: (email = 'test@test.com')
  Planning Time: 0.1 ms
  Execution Time: 0.05 ms  ← 100x faster!
```

### Common Query Patterns

```sql
-- 1. Pagination (offset is slow for large offsets)
-- Bad:
SELECT * FROM posts ORDER BY created_at DESC LIMIT 10 OFFSET 100000;
-- Scans 100,010 rows!

-- Good (keyset pagination):
SELECT * FROM posts 
WHERE created_at < '2024-01-01' 
ORDER BY created_at DESC 
LIMIT 10;
-- Only scans 10 rows!

-- 2. Counting large tables
-- Bad:
SELECT COUNT(*) FROM users;  -- Full table scan

-- Good:
SELECT reltuples FROM pg_class WHERE relname = 'users';  -- Estimate

-- 3. EXISTS vs IN
-- EXISTS (stops at first match):
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- IN (fetches all):
SELECT * FROM users
WHERE id IN (SELECT user_id FROM orders);
```

---

## 🛠️ Database Tools

### Connection Pooling (PgBouncer)

```
Without pooling:
100 app instances × 10 connections = 1,000 DB connections
Each connection uses memory (5-10MB)
→ 10GB just for connections!

With PgBouncer:
100 app instances → PgBouncer → 50 DB connections
                    (connection multiplexing)
```

### Query Analysis Tools

```
PostgreSQL:
- pg_stat_statements (slow query log)
- auto_explain (automatic query analysis)
- pgBadger (log analysis)

MySQL:
- slow query log
- Performance Schema
- pt-query-digest
```

---

## 📖 Further Reading

- "Designing Data-Intensive Applications" Ch. 2-7
- "High Performance MySQL"
- "PostgreSQL Internals"
- Use The Index, Luke (website)

---

**Next:** [Chapter 04: Caching Strategies →](./04-caching-strategies.md)


