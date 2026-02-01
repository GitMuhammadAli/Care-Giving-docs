# 🗃️ Database Indexing Deep Dive - Complete Guide

> A comprehensive guide to database indexing - B-trees, compound indexes, covering indexes, partial indexes, and how to optimize queries for maximum performance.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "A database index is a data structure (usually B-tree) that maintains a sorted copy of selected columns, enabling O(log n) lookups instead of O(n) full table scans - like a book's index that points to pages instead of reading every page."

### The INDEX Mental Model
```
WITHOUT INDEX (Full Table Scan):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Query: SELECT * FROM users WHERE email = 'john@example.com'    │
│                                                                  │
│  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐                     │
│  │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │ 9 │...│  ← Scan ALL rows    │
│  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘                     │
│                                                                  │
│  Time: O(n) - 1 million rows = 1 million checks                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

WITH INDEX (B-Tree Lookup):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Index on email column:                                         │
│                                                                  │
│              ┌───────────────┐                                  │
│              │   M (root)    │                                  │
│              └───────┬───────┘                                  │
│           ┌──────────┴──────────┐                               │
│     ┌─────▼─────┐         ┌─────▼─────┐                        │
│     │  A-L      │         │  M-Z      │                        │
│     └─────┬─────┘         └───────────┘                        │
│           │                                                     │
│     ┌─────▼─────┐                                              │
│     │ john@ →   │ ──► Row pointer                              │
│     │ Row #547  │                                              │
│     └───────────┘                                              │
│                                                                  │
│  Time: O(log n) - 1 million rows = ~20 comparisons             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Numbers to Remember
| Metric | Value | Why It Matters |
|--------|-------|----------------|
| B-tree height for 1M rows | **~3-4 levels** | Only 3-4 disk reads to find any row |
| B-tree height for 1B rows | **~6-7 levels** | Still only 6-7 disk reads! |
| Index overhead | **2-3x column size** | Index storage = data + pointers |
| Write penalty | **~10-30% slower** | Each insert updates all indexes |
| Index selectivity goal | **< 10-15%** | Index useful if filtering > 85% rows |

### Types of Indexes (Quick Reference)
```
INDEX TYPES CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  B-TREE (Default)          │  Best for: =, <, >, BETWEEN       │
│  ─────────────────          │  Most common, balanced tree       │
│                                                                  │
│  HASH                       │  Best for: = only (exact match)   │
│  ────                        │  O(1) lookup, no range queries    │
│                                                                  │
│  GIN (Generalized Inverted) │  Best for: arrays, JSONB, text   │
│  ───                         │  Full-text search, contains      │
│                                                                  │
│  GiST (Generalized Search)  │  Best for: geometry, ranges      │
│  ────                        │  PostGIS, IP ranges              │
│                                                                  │
│  BRIN (Block Range Index)   │  Best for: sorted data           │
│  ────                        │  Time-series, append-only        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement (Memorize This!)
> "At my last company, we had a dashboard that took 12 seconds to load because of a full table scan on 50 million orders. I added a compound index on (user_id, created_at) and a partial index for status='active' orders. Query time dropped to 15ms - that's 800x faster. The key insight was checking EXPLAIN ANALYZE first - the sequential scan was reading 400GB of data when we only needed 50KB. The partial index was especially clever because 95% of queries were for active orders, so we avoided indexing the 40 million archived rows."

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Selectivity"** | "The index has low selectivity - it returns 60% of rows, so the optimizer chose a seq scan" |
| **"Covering index"** | "I made it a covering index so the query is index-only - no heap fetches needed" |
| **"Index scan vs bitmap scan"** | "With 10% selectivity, PostgreSQL chose bitmap heap scan over index scan" |
| **"Partial index"** | "Created a partial index WHERE deleted_at IS NULL - 80% smaller, faster writes" |
| **"Index bloat"** | "After heavy updates, the index had 40% bloat - REINDEX dropped it to normal" |
| **"Cardinality"** | "Put high cardinality columns first in compound indexes" |
| **"Index-only scan"** | "EXPLAIN shows 'Index Only Scan' - we're not touching the heap at all" |

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How do database indexes work?"

**Junior Answer:**
> "Indexes make queries faster by creating a lookup table."

**Senior Answer:**
> "Indexes are auxiliary data structures - typically B-trees - that maintain sorted copies of column values with pointers to the actual rows.

**How B-trees work:**
- Self-balancing tree with high fan-out (100s of children per node)
- All data at leaf level, internal nodes are just routing
- Height grows logarithmically: 1 billion rows ≈ 6-7 levels
- Each level = 1 disk read, so any lookup is ~6-7 reads

**Trade-offs:**
- Reads: O(log n) instead of O(n) - massive win
- Writes: slower - must update index + data
- Storage: 2-3x the column size
- Maintenance: bloat from updates, need periodic reindex

**When indexes hurt:**
- Low selectivity (< 10-15% of rows filtered out)
- Write-heavy tables with many indexes
- Small tables (seq scan is actually faster)

The key is using EXPLAIN ANALYZE to verify the index is actually being used and measuring the real-world impact."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "Why not index every column?" | "Each index has write overhead and storage cost. 5 indexes = 5 extra writes per INSERT. Only index columns you query on." |
| "What's a covering index?" | "Include all columns the query needs so it never reads the actual table. Index-only scan is fastest possible query." |
| "When is an index not used?" | "Low selectivity, outdated statistics (run ANALYZE), type mismatch, function on column, OR conditions, leading wildcard LIKE." |
| "How do compound indexes work?" | "Like phone book: sorted by last name, then first name. Can search by last name, or last+first, but NOT first name alone." |

---

## 📚 Table of Contents

1. [B-Tree Deep Dive](#1-b-tree-deep-dive)
2. [Index Types](#2-index-types)
3. [Compound Indexes](#3-compound-indexes)
4. [Covering Indexes](#4-covering-indexes)
5. [Partial Indexes](#5-partial-indexes)
6. [Expression Indexes](#6-expression-indexes)
7. [Index Selection & EXPLAIN](#7-index-selection--explain)
8. [Common Pitfalls](#8-common-pitfalls)
9. [Interview Questions](#9-interview-questions)

---

## 1. B-Tree Deep Dive

### How B-Trees Actually Work

```
B-TREE STRUCTURE (Simplified):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Properties:                                                    │
│  • Self-balancing (all leaves at same depth)                   │
│  • High fan-out: each node has many children (100-500)         │
│  • Data stored only at leaf nodes                              │
│  • Leaves linked for range scans                               │
│                                                                  │
│                    ROOT NODE                                    │
│                  ┌──────────┐                                   │
│                  │ 50 | 100 │                                   │
│                  └──┬───┬───┘                                   │
│           ┌────────┘   │   └────────┐                          │
│           ▼            ▼            ▼                          │
│      ┌────────┐   ┌────────┐   ┌────────┐                     │
│      │ 20|40  │   │ 70|90  │   │120|150 │   INTERNAL NODES    │
│      └─┬──┬─┬─┘   └─┬──┬─┬─┘   └─┬──┬─┬─┘                     │
│        │  │ │       │  │ │       │  │ │                        │
│        ▼  ▼ ▼       ▼  ▼ ▼       ▼  ▼ ▼                       │
│      ┌───────┐   ┌───────┐   ┌───────┐                        │
│      │1,5,10,│──►│21,30,.│──►│51,60,.│ ...  LEAF NODES       │
│      │15,18  │   │35,40  │   │65,70  │      (Linked list)    │
│      └───────┘   └───────┘   └───────┘                        │
│                                                                  │
│  Lookup path for value 65:                                     │
│  1. Root: 65 > 50, go right                                   │
│  2. Internal: 65 < 70, go left                                │
│  3. Leaf: scan for 65, found! → Row pointer                   │
│                                                                  │
│  Only 3 disk reads for millions of rows!                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### B-Tree Height Calculation

```sql
-- B-tree height estimation (PostgreSQL)
-- Fan-out ≈ (page_size / entry_size) ≈ 8192 / 16 ≈ 500

-- Height calculation:
-- Height 1: 500 entries
-- Height 2: 500 * 500 = 250,000 entries  
-- Height 3: 500^3 = 125,000,000 entries
-- Height 4: 500^4 = 62,500,000,000 entries

-- Check actual index stats in PostgreSQL:
SELECT 
    indexrelname,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY pg_relation_size(indexrelid) DESC;

-- Get B-tree height (requires pageinspect extension):
CREATE EXTENSION IF NOT EXISTS pageinspect;

SELECT * FROM bt_metap('users_email_idx');
-- Returns: magic, version, root, level (height), fastroot, etc.
```

### Creating Indexes

```sql
-- ════════════════════════════════════════════════════════════════
-- BASIC INDEX CREATION
-- ════════════════════════════════════════════════════════════════

-- Simple B-tree index (default)
CREATE INDEX idx_users_email ON users(email);

-- Unique index (also enforces constraint)
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);

-- Create index concurrently (no table lock - use in production!)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- Drop index
DROP INDEX idx_users_email;
DROP INDEX CONCURRENTLY idx_users_email;  -- No lock

-- ════════════════════════════════════════════════════════════════
-- INDEX NAMING CONVENTIONS
-- ════════════════════════════════════════════════════════════════

-- Pattern: idx_{table}_{columns}_{type}
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_created_at_status ON orders(created_at, status);
CREATE INDEX idx_products_name_gin ON products USING gin(to_tsvector('english', name));
CREATE INDEX idx_users_email_partial_active ON users(email) WHERE active = true;
```

---

## 2. Index Types

### B-Tree Index (Default)

```sql
-- ════════════════════════════════════════════════════════════════
-- B-TREE: The workhorse index (95% of cases)
-- ════════════════════════════════════════════════════════════════

-- Best for: equality, ranges, sorting, BETWEEN
-- Supports: =, <, >, <=, >=, BETWEEN, IN, IS NULL, LIKE 'prefix%'
-- NOT good for: LIKE '%suffix', array contains, full-text search

CREATE INDEX idx_users_email ON users(email);  -- B-tree is default

-- B-tree handles these efficiently:
SELECT * FROM users WHERE email = 'john@example.com';      -- Equality
SELECT * FROM users WHERE created_at > '2024-01-01';       -- Range
SELECT * FROM users WHERE age BETWEEN 18 AND 65;           -- Between
SELECT * FROM users WHERE email LIKE 'john%';              -- Prefix match
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;     -- Sorting

-- B-tree CANNOT help with:
SELECT * FROM users WHERE email LIKE '%@gmail.com';        -- Suffix (no!)
SELECT * FROM users WHERE LOWER(email) = 'john@ex.com';    -- Function (no!)
```

### Hash Index

```sql
-- ════════════════════════════════════════════════════════════════
-- HASH INDEX: O(1) lookup for exact matches only
-- ════════════════════════════════════════════════════════════════

-- Best for: exact equality only
-- Supports: = 
-- NOT good for: ranges, sorting, LIKE, NULL

CREATE INDEX idx_users_email_hash ON users USING hash(email);

-- Hash can handle:
SELECT * FROM users WHERE email = 'john@example.com';  -- O(1) lookup!

-- Hash CANNOT handle (will seq scan):
SELECT * FROM users WHERE email > 'john@';      -- Range
SELECT * FROM users WHERE email LIKE 'john%';   -- Pattern
SELECT * FROM users ORDER BY email;             -- Sorting

-- When to use hash:
-- ✓ Very large tables with only equality lookups
-- ✓ High cardinality columns (UUIDs, hashes)
-- ✗ Most cases B-tree is better (more flexible)
```

### GIN Index (Generalized Inverted)

```sql
-- ════════════════════════════════════════════════════════════════
-- GIN INDEX: For arrays, JSONB, full-text search
-- ════════════════════════════════════════════════════════════════

-- Best for: contains, any, all, full-text search
-- Structure: Maps each element → rows containing it

-- Example: Tags array
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title TEXT,
    tags TEXT[]
);

CREATE INDEX idx_posts_tags ON posts USING gin(tags);

-- GIN enables fast:
SELECT * FROM posts WHERE tags @> ARRAY['javascript'];  -- Contains
SELECT * FROM posts WHERE tags && ARRAY['react', 'vue']; -- Overlaps (any)

-- JSONB indexing
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    data JSONB
);

CREATE INDEX idx_events_data ON events USING gin(data);

-- JSONB queries:
SELECT * FROM events WHERE data @> '{"type": "click"}';
SELECT * FROM events WHERE data ? 'user_id';           -- Has key
SELECT * FROM events WHERE data ?| ARRAY['a', 'b'];    -- Has any key

-- Full-text search
CREATE INDEX idx_posts_search ON posts 
    USING gin(to_tsvector('english', title || ' ' || body));

SELECT * FROM posts 
WHERE to_tsvector('english', title || ' ' || body) @@ to_tsquery('react & hooks');
```

### GiST Index (Generalized Search Tree)

```sql
-- ════════════════════════════════════════════════════════════════
-- GiST INDEX: For geometric data, ranges, nearest neighbor
-- ════════════════════════════════════════════════════════════════

-- Best for: PostGIS, IP ranges, tsrange, box, point

-- Range types (PostgreSQL)
CREATE TABLE reservations (
    id SERIAL PRIMARY KEY,
    room_id INT,
    during TSTZRANGE  -- Time range
);

CREATE INDEX idx_reservations_during ON reservations USING gist(during);

-- Overlap queries:
SELECT * FROM reservations 
WHERE during && '[2024-03-01, 2024-03-15]'::tstzrange;

-- PostGIS geometry
CREATE INDEX idx_locations_geom ON locations USING gist(geom);

SELECT * FROM locations 
WHERE ST_DWithin(geom, ST_MakePoint(-73.99, 40.73), 1000);  -- Within 1km
```

### BRIN Index (Block Range Index)

```sql
-- ════════════════════════════════════════════════════════════════
-- BRIN INDEX: Tiny index for naturally ordered data
-- ════════════════════════════════════════════════════════════════

-- Best for: append-only tables, time-series, log data
-- Size: ~1000x smaller than B-tree!
-- Trade-off: Less precise, but great for sequential data

CREATE TABLE logs (
    id BIGSERIAL PRIMARY KEY,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    message TEXT
);

-- BRIN stores min/max per block of pages
CREATE INDEX idx_logs_created_at_brin ON logs USING brin(created_at);

-- Works well because data is inserted in order:
SELECT * FROM logs 
WHERE created_at > NOW() - INTERVAL '1 day';

-- Check BRIN effectiveness:
SELECT 
    indexrelname,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size
FROM pg_stat_user_indexes
WHERE indexrelname LIKE '%brin%';

-- BRIN typically: 1MB index for 1TB table
-- B-tree would be: 50GB+ for same table
```

### Comparison Table

```
INDEX TYPE COMPARISON:
┌─────────────────────────────────────────────────────────────────────────────┐
│ Type   │ Use Case                  │ Operators           │ Size    │ Speed │
├────────┼───────────────────────────┼─────────────────────┼─────────┼───────┤
│ B-tree │ General purpose           │ =,<,>,<=,>=,BETWEEN │ Medium  │ Fast  │
│ Hash   │ Equality only             │ =                   │ Medium  │ O(1)  │
│ GIN    │ Arrays, JSONB, text       │ @>,&&,@@            │ Large   │ Fast  │
│ GiST   │ Geometry, ranges          │ &&,@>,<@,~=         │ Medium  │ Fast  │
│ BRIN   │ Sorted/time-series        │ =,<,>,<=,>=         │ Tiny    │ Ok    │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Compound Indexes

### The Phone Book Analogy

```
COMPOUND INDEX = PHONE BOOK:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Phone book sorted by: LAST NAME, then FIRST NAME               │
│                                                                  │
│  ✓ Can find: "Smith" (last name only)                          │
│  ✓ Can find: "Smith, John" (last + first)                      │
│  ✗ Cannot find: all "John" (first name only!)                  │
│                                                                  │
│  INDEX on (last_name, first_name):                              │
│                                                                  │
│  Adams, Alice      ← Sorted by last, then first                │
│  Adams, Bob                                                     │
│  Baker, Alice                                                   │
│  Baker, Charlie                                                 │
│  Smith, Alice                                                   │
│  Smith, John       ← Can jump to "Smith" then scan to "John"   │
│  Smith, Zoe                                                     │
│                                                                  │
│  Without last_name, can't use the index efficiently!           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Column Order Matters!

```sql
-- ════════════════════════════════════════════════════════════════
-- COLUMN ORDER: Critical for compound indexes
-- ════════════════════════════════════════════════════════════════

-- Index: (user_id, created_at, status)
CREATE INDEX idx_orders_user_date_status 
    ON orders(user_id, created_at, status);

-- ✓ Uses index (leftmost columns):
SELECT * FROM orders WHERE user_id = 123;
SELECT * FROM orders WHERE user_id = 123 AND created_at > '2024-01-01';
SELECT * FROM orders WHERE user_id = 123 AND created_at > '2024-01-01' AND status = 'active';

-- ✗ Cannot use index (skips user_id):
SELECT * FROM orders WHERE created_at > '2024-01-01';  -- Seq scan!
SELECT * FROM orders WHERE status = 'active';          -- Seq scan!

-- ⚠️ Partial use (uses user_id only, then filter):
SELECT * FROM orders WHERE user_id = 123 AND status = 'active';
-- Can use index for user_id, but must filter status afterward

-- ════════════════════════════════════════════════════════════════
-- RULE: Equality columns first, then range columns
-- ════════════════════════════════════════════════════════════════

-- BAD order: range column first
CREATE INDEX idx_bad ON orders(created_at, user_id, status);
-- Query: WHERE user_id = 123 AND created_at > '2024-01-01'
-- Can only use created_at range, then filter user_id

-- GOOD order: equality first, range last
CREATE INDEX idx_good ON orders(user_id, status, created_at);
-- Query: WHERE user_id = 123 AND status = 'active' AND created_at > '2024-01-01'
-- Uses all three columns efficiently!
```

### Index Column Order Guidelines

```
COMPOUND INDEX COLUMN ORDER RULES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. EQUALITY CONDITIONS FIRST                                   │
│     Columns with = comparisons                                  │
│     WHERE user_id = 123 AND status = 'active'                  │
│     → INDEX(user_id, status, ...)                              │
│                                                                  │
│  2. RANGE CONDITIONS LAST                                       │
│     Columns with <, >, BETWEEN                                 │
│     WHERE created_at > '2024-01-01'                            │
│     → INDEX(..., created_at)                                   │
│                                                                  │
│  3. HIGH CARDINALITY FIRST (among equals)                      │
│     user_id (1M values) before status (5 values)               │
│     More selective = fewer rows to scan                        │
│                                                                  │
│  4. FREQUENTLY FILTERED FIRST                                  │
│     If you query by user_id 90% of time, put it first          │
│                                                                  │
│  EXAMPLE:                                                       │
│  Query: user_id = ? AND status = ? AND created_at > ?          │
│                                                                  │
│  INDEX(user_id, status, created_at)                            │
│       ↑ equals    ↑ equals  ↑ range (last!)                    │
│       ↑ high card ↑ low card                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Multiple Indexes vs Compound Index

```sql
-- ════════════════════════════════════════════════════════════════
-- MULTIPLE SINGLE vs ONE COMPOUND
-- ════════════════════════════════════════════════════════════════

-- Option A: Multiple single-column indexes
CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_date ON orders(created_at);

-- Option B: One compound index
CREATE INDEX idx_orders_compound ON orders(user_id, status, created_at);

-- For query: WHERE user_id = 123 AND status = 'active' AND created_at > '2024-01-01'
-- Option A: Bitmap index scan (combine indexes) - slower
-- Option B: Single index scan - faster

-- ════════════════════════════════════════════════════════════════
-- WHEN TO USE EACH
-- ════════════════════════════════════════════════════════════════

-- USE COMPOUND INDEX when:
-- ✓ Queries always use these columns together
-- ✓ Need to optimize specific query patterns
-- ✓ ORDER BY matches index order

-- USE MULTIPLE SINGLE INDEXES when:
-- ✓ Columns are used in different combinations
-- ✓ Different queries filter by different columns
-- ✓ Need flexibility

-- REAL EXAMPLE:
-- These queries need different strategies:
SELECT * FROM orders WHERE user_id = 123;                    -- idx(user_id)
SELECT * FROM orders WHERE status = 'pending';               -- idx(status)
SELECT * FROM orders WHERE user_id = 123 AND status = 'x';   -- idx(user_id, status)

-- Best solution: compound + single
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
CREATE INDEX idx_orders_status ON orders(status);  -- For status-only queries
```

---

## 4. Covering Indexes

### What is a Covering Index?

```
COVERING INDEX = INDEX-ONLY SCAN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  REGULAR INDEX (needs heap fetch):                              │
│  ───────────────────────────────────                            │
│                                                                  │
│  Query: SELECT email, name FROM users WHERE email = 'x@y.com'   │
│                                                                  │
│  1. Search index for email     → Find row pointer               │
│  2. Jump to heap (table)       → Fetch actual row               │
│  3. Extract name from row      → Return result                  │
│                                                                  │
│  [Index: email] ──► [Row pointer] ──► [Heap: full row]         │
│                                        Random I/O!              │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  COVERING INDEX (no heap fetch):                                │
│  ───────────────────────────────                                │
│                                                                  │
│  Index on (email) INCLUDE (name)                                │
│                                                                  │
│  1. Search index for email     → Find entry                     │
│  2. Entry CONTAINS name        → Return immediately!            │
│                                                                  │
│  [Index: email, name] ──► Done! No heap access!                │
│                          Sequential I/O only                    │
│                                                                  │
│  RESULT: 10-100x faster for covered queries                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Creating Covering Indexes

```sql
-- ════════════════════════════════════════════════════════════════
-- INCLUDE CLAUSE (PostgreSQL 11+)
-- ════════════════════════════════════════════════════════════════

-- Query to optimize:
SELECT user_id, email, created_at 
FROM users 
WHERE email = 'john@example.com';

-- Regular index (requires heap fetch for created_at):
CREATE INDEX idx_users_email ON users(email);

-- Covering index (all columns in index):
CREATE INDEX idx_users_email_covering ON users(email) 
    INCLUDE (user_id, created_at);

-- INCLUDE columns:
-- ✓ Stored in leaf nodes only (not for searching)
-- ✓ No search overhead
-- ✓ Enable index-only scan

-- ════════════════════════════════════════════════════════════════
-- ALTERNATIVE: All columns in index (older approach)
-- ════════════════════════════════════════════════════════════════

-- Without INCLUDE (all searchable):
CREATE INDEX idx_users_email_all ON users(email, user_id, created_at);

-- Difference:
-- (email, user_id, created_at) = Can search by any prefix
-- (email) INCLUDE (user_id, created_at) = Can only search by email

-- INCLUDE is better when:
-- ✓ Extra columns only for SELECT, not WHERE
-- ✓ Smaller index size (included cols not in internal nodes)
```

### Verify Index-Only Scan

```sql
-- ════════════════════════════════════════════════════════════════
-- CHECK IF QUERY USES INDEX-ONLY SCAN
-- ════════════════════════════════════════════════════════════════

-- Create covering index
CREATE INDEX idx_orders_covering ON orders(user_id, status) 
    INCLUDE (total, created_at);

-- Check with EXPLAIN:
EXPLAIN (ANALYZE, BUFFERS) 
SELECT user_id, status, total, created_at 
FROM orders 
WHERE user_id = 123 AND status = 'active';

-- GOOD output (look for "Index Only Scan"):
-- Index Only Scan using idx_orders_covering on orders
--   Index Cond: ((user_id = 123) AND (status = 'active'))
--   Heap Fetches: 0    ← No heap access!
--   Buffers: shared hit=3

-- BAD output (regular index scan):
-- Index Scan using idx_orders_user on orders
--   Index Cond: (user_id = 123)
--   Filter: (status = 'active')
--   Buffers: shared hit=3 read=150   ← Heap reads!

-- ════════════════════════════════════════════════════════════════
-- WHY "Heap Fetches" MIGHT NOT BE ZERO
-- ════════════════════════════════════════════════════════════════

-- Even with covering index, heap fetches occur when:
-- 1. Visibility check needed (MVCC - row recently modified)
-- 2. Table not vacuumed (visibility map not set)

-- Fix: Run VACUUM
VACUUM ANALYZE orders;

-- Now re-run EXPLAIN - Heap Fetches should be 0
```

### When to Use Covering Indexes

```sql
-- ════════════════════════════════════════════════════════════════
-- GOOD CANDIDATES FOR COVERING INDEXES
-- ════════════════════════════════════════════════════════════════

-- 1. Dashboard queries (frequent, specific columns)
-- Query: Show user's recent orders summary
CREATE INDEX idx_orders_dashboard ON orders(user_id, created_at DESC)
    INCLUDE (status, total);

SELECT status, total, created_at 
FROM orders 
WHERE user_id = 123 
ORDER BY created_at DESC 
LIMIT 10;

-- 2. Aggregation queries
-- Query: Count and sum by status
CREATE INDEX idx_orders_status_agg ON orders(status)
    INCLUDE (total);

SELECT status, COUNT(*), SUM(total)
FROM orders
GROUP BY status;

-- 3. Lookup tables (return few columns)
-- Query: Get user name for display
CREATE INDEX idx_users_lookup ON users(id)
    INCLUDE (name, avatar_url);

SELECT name, avatar_url FROM users WHERE id = ANY(ARRAY[1,2,3,4,5]);

-- ════════════════════════════════════════════════════════════════
-- AVOID COVERING INDEXES WHEN:
-- ════════════════════════════════════════════════════════════════

-- ✗ SELECT * queries (can't cover everything)
-- ✗ Frequently updated columns (index update overhead)
-- ✗ Large columns (bloats index)
-- ✗ Infrequent queries (not worth the space)
```

---

## 5. Partial Indexes

### What is a Partial Index?

```
PARTIAL INDEX = INDEX WITH WHERE CLAUSE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  FULL INDEX (all rows):                                         │
│  ─────────────────────                                          │
│  100 million orders indexed                                    │
│  95 million are completed (rarely queried)                     │
│  5 million are active (frequently queried)                     │
│                                                                  │
│  Index size: 10 GB                                             │
│  Update cost: Every INSERT/UPDATE touches index                │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  PARTIAL INDEX (only matching rows):                           │
│  ────────────────────────────────────                          │
│  Only 5 million active orders indexed                          │
│  95 million completed orders NOT indexed                       │
│                                                                  │
│  Index size: 0.5 GB (95% smaller!)                            │
│  Update cost: Only updates when status = 'active'              │
│                                                                  │
│  Perfect when queries always filter same condition!            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Creating Partial Indexes

```sql
-- ════════════════════════════════════════════════════════════════
-- PARTIAL INDEX SYNTAX
-- ════════════════════════════════════════════════════════════════

-- Index only active orders
CREATE INDEX idx_orders_active ON orders(user_id, created_at)
    WHERE status = 'active';

-- Index only non-deleted users
CREATE INDEX idx_users_not_deleted ON users(email)
    WHERE deleted_at IS NULL;

-- Index only recent data
CREATE INDEX idx_logs_recent ON logs(level, message)
    WHERE created_at > '2024-01-01';

-- Index only NULL values (useful for "find unprocessed")
CREATE INDEX idx_jobs_unprocessed ON jobs(created_at)
    WHERE processed_at IS NULL;

-- ════════════════════════════════════════════════════════════════
-- QUERY MUST MATCH WHERE CLAUSE
-- ════════════════════════════════════════════════════════════════

-- Index: WHERE status = 'active'

-- ✓ Uses partial index:
SELECT * FROM orders WHERE user_id = 123 AND status = 'active';

-- ✗ Cannot use partial index (different status):
SELECT * FROM orders WHERE user_id = 123 AND status = 'pending';

-- ✗ Cannot use partial index (no status filter):
SELECT * FROM orders WHERE user_id = 123;
```

### Real-World Partial Index Examples

```sql
-- ════════════════════════════════════════════════════════════════
-- EXAMPLE 1: Soft Delete Pattern
-- ════════════════════════════════════════════════════════════════

-- Table with soft delete
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email TEXT NOT NULL,
    deleted_at TIMESTAMPTZ  -- NULL = not deleted
);

-- Unique constraint only on active users
CREATE UNIQUE INDEX idx_users_email_active ON users(email)
    WHERE deleted_at IS NULL;

-- This allows:
-- User 1: email='john@ex.com', deleted_at=NULL     ✓
-- User 2: email='john@ex.com', deleted_at='2024'   ✓ (deleted, can reuse email)
-- User 3: email='john@ex.com', deleted_at=NULL     ✗ (conflict with User 1)

-- ════════════════════════════════════════════════════════════════
-- EXAMPLE 2: Job Queue Pattern
-- ════════════════════════════════════════════════════════════════

CREATE TABLE jobs (
    id SERIAL PRIMARY KEY,
    queue TEXT NOT NULL,
    payload JSONB,
    scheduled_at TIMESTAMPTZ,
    started_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ
);

-- Index only pending jobs (not started)
CREATE INDEX idx_jobs_pending ON jobs(queue, scheduled_at)
    WHERE started_at IS NULL;

-- Fetch next job query (very fast):
SELECT * FROM jobs 
WHERE queue = 'emails' 
  AND started_at IS NULL 
  AND scheduled_at <= NOW()
ORDER BY scheduled_at
LIMIT 1
FOR UPDATE SKIP LOCKED;

-- ════════════════════════════════════════════════════════════════
-- EXAMPLE 3: Multi-tenant with Hot Tenants
-- ════════════════════════════════════════════════════════════════

-- Large tenant needs special index
CREATE INDEX idx_orders_tenant_123 ON orders(created_at, status)
    WHERE tenant_id = 123;

-- Other tenants use general index
CREATE INDEX idx_orders_tenant_general ON orders(tenant_id, created_at);
```

---

## 6. Expression Indexes

### What is an Expression Index?

```sql
-- ════════════════════════════════════════════════════════════════
-- EXPRESSION INDEX = INDEX ON COMPUTED VALUE
-- ════════════════════════════════════════════════════════════════

-- Problem: This query can't use index on email
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';
-- Index on email is useless - we're searching LOWER(email)!

-- Solution: Index the expression
CREATE INDEX idx_users_email_lower ON users(LOWER(email));

-- Now this query uses the index:
SELECT * FROM users WHERE LOWER(email) = 'john@example.com';

-- ════════════════════════════════════════════════════════════════
-- COMMON EXPRESSION INDEXES
-- ════════════════════════════════════════════════════════════════

-- Case-insensitive search
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
CREATE INDEX idx_products_name_lower ON products(LOWER(name));

-- Date extraction
CREATE INDEX idx_orders_year ON orders(EXTRACT(YEAR FROM created_at));
CREATE INDEX idx_orders_month ON orders(DATE_TRUNC('month', created_at));

-- Query:
SELECT * FROM orders WHERE EXTRACT(YEAR FROM created_at) = 2024;

-- JSONB field
CREATE INDEX idx_events_user_id ON events((data->>'user_id'));

-- Query:
SELECT * FROM events WHERE data->>'user_id' = '12345';

-- Computed value
CREATE INDEX idx_orders_total_cents ON orders((amount * 100)::integer);
```

### Expression Index Caveats

```sql
-- ════════════════════════════════════════════════════════════════
-- EXPRESSION MUST MATCH EXACTLY
-- ════════════════════════════════════════════════════════════════

-- Index on LOWER(email)
CREATE INDEX idx_email_lower ON users(LOWER(email));

-- ✓ Uses index:
SELECT * FROM users WHERE LOWER(email) = 'john@ex.com';

-- ✗ Does NOT use index (different expression):
SELECT * FROM users WHERE email ILIKE 'john@ex.com';
SELECT * FROM users WHERE UPPER(email) = 'JOHN@EX.COM';
SELECT * FROM users WHERE lower(email) = 'john@ex.com';  -- lowercase 'lower'

-- ════════════════════════════════════════════════════════════════
-- IMMUTABLE FUNCTIONS ONLY
-- ════════════════════════════════════════════════════════════════

-- Must use IMMUTABLE functions (same input = same output always)
-- ✓ LOWER(), UPPER(), DATE_TRUNC()
-- ✗ NOW(), RANDOM(), CURRENT_DATE

-- This won't work:
-- CREATE INDEX idx_bad ON events((created_at - NOW()));  -- Error!
```

---

## 7. Index Selection & EXPLAIN

### Reading EXPLAIN ANALYZE

```sql
-- ════════════════════════════════════════════════════════════════
-- EXPLAIN ANALYZE: Your Best Friend
-- ════════════════════════════════════════════════════════════════

EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM orders WHERE user_id = 123 AND status = 'active';

-- Output breakdown:
/*
Index Scan using idx_orders_user_status on orders  
    (cost=0.43..8.45 rows=1 width=100) 
    (actual time=0.025..0.027 rows=5 loops=1)
  Index Cond: ((user_id = 123) AND (status = 'active'))
  Buffers: shared hit=4
Planning Time: 0.150 ms
Execution Time: 0.045 ms

WHAT TO LOOK FOR:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  "Index Scan"        → Good! Using index                       │
│  "Index Only Scan"   → Great! Not touching heap                │
│  "Seq Scan"          → Bad! Full table scan                    │
│  "Bitmap Heap Scan"  → OK, combining indexes                   │
│                                                                  │
│  "rows=1" vs "rows=5" → Estimate vs actual (bad estimates =    │
│                          need ANALYZE)                          │
│                                                                  │
│  "shared hit=4"      → 4 pages from cache (good)               │
│  "shared read=100"   → 100 pages from disk (slow)              │
│                                                                  │
│  "Execution Time"    → Actual query time                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
*/
```

### Why Index Might Not Be Used

```sql
-- ════════════════════════════════════════════════════════════════
-- REASONS OPTIMIZER IGNORES YOUR INDEX
-- ════════════════════════════════════════════════════════════════

-- 1. LOW SELECTIVITY (returning too many rows)
-- Index on status (only 3 values), table has 1M rows
SELECT * FROM orders WHERE status = 'completed';
-- 90% of rows are 'completed' → Seq scan is faster!

-- 2. STATISTICS OUTDATED
-- After bulk insert, optimizer doesn't know about new data
ANALYZE orders;  -- Update statistics!

-- 3. TYPE MISMATCH
-- Index on integer, query with string
SELECT * FROM users WHERE id = '123';  -- Cast prevents index use!
SELECT * FROM users WHERE id = 123;    -- Use correct type

-- 4. FUNCTION ON COLUMN
-- Index on email, but function applied
SELECT * FROM users WHERE LOWER(email) = 'x';  -- No index!
-- Solution: Expression index on LOWER(email)

-- 5. LEADING WILDCARD
SELECT * FROM users WHERE email LIKE '%@gmail.com';  -- No index!
-- Solution: Reverse index + reverse query, or full-text search

-- 6. OR CONDITIONS
SELECT * FROM users WHERE email = 'x' OR phone = 'y';
-- May use bitmap scan, or seq scan
-- Solution: UNION of two queries, or separate indexes

-- 7. NULL HANDLING
-- B-tree CAN index NULL, but IS NULL might still seq scan
-- Check statistics and consider partial index

-- ════════════════════════════════════════════════════════════════
-- FORCE INDEX USE (for testing only!)
-- ════════════════════════════════════════════════════════════════

-- PostgreSQL: disable seq scan
SET enable_seqscan = off;
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'active';
SET enable_seqscan = on;  -- Turn back on!

-- MySQL: force index
SELECT * FROM orders FORCE INDEX (idx_orders_status) WHERE status = 'active';
```

### Index Monitoring

```sql
-- ════════════════════════════════════════════════════════════════
-- FIND UNUSED INDEXES (candidates for removal)
-- ════════════════════════════════════════════════════════════════

SELECT 
    schemaname,
    relname AS table_name,
    indexrelname AS index_name,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    idx_scan AS times_used,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE idx_scan = 0  -- Never used!
ORDER BY pg_relation_size(indexrelid) DESC;

-- ════════════════════════════════════════════════════════════════
-- FIND MISSING INDEXES (seq scans that should be indexed)
-- ════════════════════════════════════════════════════════════════

SELECT 
    relname AS table_name,
    seq_scan,
    seq_tup_read,
    idx_scan,
    seq_tup_read / NULLIF(seq_scan, 0) AS avg_rows_per_seq_scan
FROM pg_stat_user_tables
WHERE seq_scan > 100  -- Many seq scans
  AND seq_tup_read / NULLIF(seq_scan, 0) > 1000  -- Reading many rows
ORDER BY seq_tup_read DESC;

-- ════════════════════════════════════════════════════════════════
-- INDEX BLOAT (after many updates/deletes)
-- ════════════════════════════════════════════════════════════════

-- Check index bloat
SELECT 
    indexrelname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    pg_size_pretty(pg_relation_size(indrelid)) AS table_size
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC;

-- Fix bloat
REINDEX INDEX CONCURRENTLY idx_orders_user;  -- Rebuild without lock
-- Or
VACUUM (VERBOSE) orders;  -- Clean up dead tuples
```

---

## 8. Common Pitfalls

### Index Anti-Patterns

```
INDEXING ANTI-PATTERNS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. OVER-INDEXING                                               │
│     ────────────────                                            │
│     Problem: Index on every column "just in case"              │
│     Impact: Slow writes, wasted storage, maintenance burden     │
│     Rule: Only index columns you actually query                │
│                                                                  │
│  2. DUPLICATE INDEXES                                           │
│     ──────────────────                                          │
│     Problem: idx(a) AND idx(a, b) - first is redundant         │
│     Impact: Wasted space, extra write overhead                  │
│     Rule: Compound index covers leftmost prefix queries         │
│                                                                  │
│  3. WRONG COLUMN ORDER                                          │
│     ────────────────────                                        │
│     Problem: idx(status, user_id) for user_id queries          │
│     Impact: Index unused, seq scan                             │
│     Rule: Most selective column first, range columns last       │
│                                                                  │
│  4. INDEXING LOW-CARDINALITY                                    │
│     ───────────────────────────                                 │
│     Problem: Index on boolean or status with 3 values          │
│     Impact: Index scan not faster than seq scan                │
│     Rule: Only index columns with many distinct values          │
│                                                                  │
│  5. IGNORING PARTIAL INDEXES                                    │
│     ─────────────────────────                                   │
│     Problem: Full index when 90% of rows never queried         │
│     Impact: Large index, slow updates                          │
│     Rule: Use WHERE clause for frequently filtered conditions   │
│                                                                  │
│  6. NOT USING COVERING INDEXES                                  │
│     ───────────────────────────                                 │
│     Problem: Index-only queries still fetch from heap          │
│     Impact: Extra I/O for every query                          │
│     Rule: INCLUDE columns needed for SELECT                     │
│                                                                  │
│  7. FORGETTING TO ANALYZE                                       │
│     ──────────────────────                                      │
│     Problem: After bulk operations, stats are stale            │
│     Impact: Optimizer makes wrong choices                       │
│     Rule: Run ANALYZE after major data changes                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Index Maintenance Checklist

```sql
-- ════════════════════════════════════════════════════════════════
-- REGULAR INDEX MAINTENANCE
-- ════════════════════════════════════════════════════════════════

-- 1. Update statistics (run weekly or after bulk operations)
ANALYZE orders;

-- 2. Check for unused indexes (run monthly)
SELECT indexrelname, idx_scan 
FROM pg_stat_user_indexes 
WHERE idx_scan < 10;

-- 3. Check for index bloat (run monthly)
SELECT indexrelname, pg_size_pretty(pg_relation_size(indexrelid))
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC
LIMIT 10;

-- 4. Rebuild bloated indexes (as needed)
REINDEX INDEX CONCURRENTLY idx_orders_user;

-- 5. Check for missing indexes (review slow query log)
-- pg_stat_statements extension helps identify slow queries
```

---

## 9. Interview Questions

### Conceptual Questions

**Q: "What is a database index and how does it work?"**
> "An index is a data structure, typically a B-tree, that maintains sorted copies of column values with pointers to actual rows. Instead of scanning every row O(n), we do a tree traversal O(log n). For 1 billion rows, that's ~30 comparisons vs 1 billion. Trade-off: faster reads, slower writes, extra storage."

**Q: "When would you NOT use an index?"**
> "Several cases:
> 1. **Low selectivity** - if query returns >15% of rows, seq scan is faster
> 2. **Small tables** - overhead not worth it for <1000 rows
> 3. **Write-heavy tables** - each index slows down inserts
> 4. **Columns rarely queried** - storage waste
> 5. **Wide columns** - large index size
> The key is measuring with EXPLAIN ANALYZE."

**Q: "Explain the difference between a compound index and multiple single-column indexes."**
> "Compound index (a, b) stores sorted pairs and can answer queries on (a), (a, b), but NOT (b) alone - like a phone book sorted by last name then first name. Multiple single indexes can be combined via bitmap scan, but it's slower than a purpose-built compound index. Use compound when columns are always queried together."

**Q: "What is a covering index?"**
> "A covering index includes all columns a query needs, enabling an index-only scan without touching the main table. Created with `INCLUDE (col1, col2)` in PostgreSQL. For example, if you query `SELECT name, email WHERE user_id = ?`, an index on `(user_id) INCLUDE (name, email)` avoids heap fetches entirely."

### Scenario Questions

**Q: "Query is slow despite having an index. How do you debug?"**
> "Step by step:
> 1. **EXPLAIN ANALYZE** - is index actually used? Look for 'Index Scan'
> 2. **Check selectivity** - if returning many rows, seq scan might be chosen
> 3. **Run ANALYZE** - statistics might be outdated
> 4. **Check for type mismatch** - `WHERE id = '123'` won't use integer index
> 5. **Check for function** - `WHERE LOWER(email)` needs expression index
> 6. **Check column order** - compound index on (a, b) won't help query on (b)
> 7. **Verify visibility map** - recently modified rows need heap fetch"

**Q: "How would you index a soft-delete table?"**
> "Use a partial index:
> ```sql
> CREATE UNIQUE INDEX idx_users_email ON users(email) 
>     WHERE deleted_at IS NULL;
> ```
> This ensures uniqueness only among active users, allows email reuse after deletion, and keeps the index small. Queries with `WHERE deleted_at IS NULL` will use this index."

**Q: "You have a table with 100M rows, 95% are archived. How do you optimize queries for active rows?"**
> "Partial index is perfect here:
> ```sql
> CREATE INDEX idx_orders_active ON orders(user_id, created_at) 
>     WHERE status = 'active';
> ```
> This is 95% smaller than a full index, faster to update, and queries filtering `status = 'active'` use it automatically. You might also consider table partitioning if the active/archive split is time-based."

### Quick Fire

| Question | Answer |
|----------|--------|
| "B-tree height for 1M rows?" | "~3-4 levels (fan-out ~500 per node)" |
| "Index overhead estimate?" | "2-3x the column size for storage, 10-30% write slowdown" |
| "INCLUDE vs compound columns?" | "INCLUDE = not searchable, smaller index; compound = can search any prefix" |
| "When does index-only scan fail?" | "When visibility map not set (need VACUUM), or MVCC check needed" |
| "How to find unused indexes?" | "`pg_stat_user_indexes WHERE idx_scan = 0`" |

---

## Quick Reference

```
DATABASE INDEXING CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  INDEX TYPES:                                                   │
│  • B-tree (default): =, <, >, BETWEEN, ORDER BY                │
│  • Hash: = only, O(1) lookup                                   │
│  • GIN: arrays, JSONB, full-text                               │
│  • GiST: geometry, ranges                                      │
│  • BRIN: sorted/time-series, tiny size                         │
│                                                                  │
│  COMPOUND INDEX RULES:                                          │
│  1. Equality columns first                                     │
│  2. Range columns last                                         │
│  3. High cardinality first (among equals)                      │
│  4. Can use leftmost prefix only                               │
│                                                                  │
│  SPECIAL INDEXES:                                               │
│  • Covering: INCLUDE (col) - enables index-only scan           │
│  • Partial: WHERE condition - smaller, faster                  │
│  • Expression: ON (LOWER(email)) - for functions               │
│                                                                  │
│  WHEN INDEX NOT USED:                                           │
│  • Low selectivity (>15% rows)                                 │
│  • Type mismatch                                                │
│  • Function on column                                          │
│  • Leading wildcard LIKE                                       │
│  • Outdated statistics                                         │
│                                                                  │
│  MONITORING:                                                    │
│  • EXPLAIN ANALYZE - see actual plan                           │
│  • pg_stat_user_indexes - usage stats                          │
│  • ANALYZE - update statistics                                 │
│  • REINDEX CONCURRENTLY - fix bloat                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

*Guide covers PostgreSQL primarily. MySQL/MariaDB syntax differs slightly but concepts are identical.*


