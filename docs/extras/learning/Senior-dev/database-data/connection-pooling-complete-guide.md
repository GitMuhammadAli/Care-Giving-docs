# 🔌 Connection Pooling - Complete Guide

> A comprehensive guide to database connection pooling - PgBouncer, connection limits, pool sizing, and optimizing database connections for high-performance applications.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Connection pooling reuses database connections across requests instead of creating new ones each time - reducing the overhead of TCP handshake, SSL negotiation, and authentication from ~50ms to ~1ms per query."

### The "Wow" Statement
> "Our Node.js app was timing out under load. Each Lambda created its own PostgreSQL connection - at 500 concurrent requests, we hit the 100 connection limit and queries queued for 30+ seconds. I added PgBouncer in transaction mode with a pool of 50 connections. Now 500 concurrent requests share 50 connections, query latency dropped from 30s to 15ms. The key was understanding that most queries take <10ms, so connections can be reused rapidly."

### Key Numbers
| Metric | Value | Context |
|--------|-------|---------|
| Connection overhead | **30-100ms** | TCP + SSL + auth |
| Pooled connection | **<1ms** | Already established |
| PostgreSQL max connections | **~100-500** | Default, memory-bound |
| Connections per core | **~25-50** | PostgreSQL rule of thumb |
| Pool size formula | **connections = (cores * 2) + disks** | PostgreSQL recommendation |

---

## 🎯 Core Concepts

### Why Pooling Matters

```
WITHOUT POOLING:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Request 1 ──► New connection (50ms) ──► Query ──► Close       │
│  Request 2 ──► New connection (50ms) ──► Query ──► Close       │
│  Request 3 ──► New connection (50ms) ──► Query ──► Close       │
│                                                                  │
│  500 requests = 500 connections at once = DB overloaded!       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

WITH POOLING:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Pool: 20 pre-established connections                          │
│                                                                  │
│  Request 1 ──► Get from pool (<1ms) ──► Query ──► Return       │
│  Request 2 ──► Get from pool (<1ms) ──► Query ──► Return       │
│  Request 3 ──► Get from pool (<1ms) ──► Query ──► Return       │
│                                                                  │
│  500 requests share 20 connections = DB happy!                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### PgBouncer Modes

```
PGBOUNCER POOLING MODES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SESSION MODE (default)                                         │
│  • Connection held for entire client session                   │
│  • Most compatible, least efficient                            │
│  • Good for: Connection testing, simple apps                   │
│                                                                  │
│  TRANSACTION MODE (recommended)                                 │
│  • Connection returned after each transaction                  │
│  • Best balance of compatibility and efficiency               │
│  • Good for: Most web applications                            │
│  • Limitation: No session-level state (prepared statements)    │
│                                                                  │
│  STATEMENT MODE (aggressive)                                    │
│  • Connection returned after each statement                    │
│  • Most efficient, least compatible                            │
│  • Good for: Simple queries, high-throughput                   │
│  • Limitation: No multi-statement transactions!               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Pool Sizing

```typescript
// ════════════════════════════════════════════════════════════════
// POOL SIZING GUIDELINES
// ════════════════════════════════════════════════════════════════

// PostgreSQL recommendation:
// connections = (core_count * 2) + effective_spindle_count
// For SSD: connections = cores * 2 + 1 ≈ 4-core server = 9 connections

// Node.js pool configuration
const pool = new Pool({
    host: 'localhost',
    database: 'app',
    max: 20,              // Maximum connections in pool
    min: 5,               // Minimum idle connections
    idleTimeoutMillis: 30000,  // Close idle connections after 30s
    connectionTimeoutMillis: 2000, // Fail if can't get connection in 2s
});

// Rule of thumb for web apps:
// pool_size = expected_concurrent_requests / 10
// 200 concurrent users ≈ 20 pool connections

// Lambda/Serverless special case:
// Each Lambda instance needs own pool
// Use external pooler (PgBouncer/RDS Proxy) for Lambda
```

---

## Implementation

### PgBouncer Setup

```ini
; /etc/pgbouncer/pgbouncer.ini
[databases]
app = host=localhost port=5432 dbname=app

[pgbouncer]
listen_addr = *
listen_port = 6432
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt

; Pool settings
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 20
min_pool_size = 5
reserve_pool_size = 5

; Timeouts
server_idle_timeout = 60
client_idle_timeout = 0
```

### Application Pool

```typescript
// ════════════════════════════════════════════════════════════════
// NODE.JS WITH PG POOL
// ════════════════════════════════════════════════════════════════

import { Pool } from 'pg';

const pool = new Pool({
    connectionString: process.env.DATABASE_URL,
    max: 20,
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 2000,
});

// Always release connections!
async function query(sql: string, params: any[]) {
    const client = await pool.connect();
    try {
        return await client.query(sql, params);
    } finally {
        client.release();  // CRITICAL: Always release!
    }
}

// Or use pool.query() which handles release automatically
async function simpleQuery(sql: string, params: any[]) {
    return pool.query(sql, params);
}

// Monitor pool health
pool.on('error', (err) => console.error('Pool error:', err));
pool.on('connect', () => console.log('New connection'));
pool.on('remove', () => console.log('Connection removed'));
```

---

## Interview Questions

**Q: "Why is connection pooling important?"**
> "Creating a database connection is expensive: TCP handshake, SSL negotiation, authentication takes 30-100ms. Pooling reuses connections, reducing overhead to <1ms. Also prevents hitting database connection limits under load."

**Q: "How do you size a connection pool?"**
> "Start with PostgreSQL formula: (cores × 2) + 1. For web apps, divide expected concurrent requests by average query time. 20 connections can serve 200 concurrent users if queries take <100ms. Monitor and adjust based on pool wait times."

**Q: "Session vs transaction pooling mode?"**
> "Session mode holds connection for entire client session - most compatible but least efficient. Transaction mode returns connection after each transaction - best for web apps. Transaction mode breaks prepared statements and session variables."

---

## Quick Reference

```
CONNECTION POOLING CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SIZING:                                                        │
│  • PostgreSQL: (cores × 2) + 1                                 │
│  • Web apps: concurrent_requests / 10                          │
│  • Start small, increase based on metrics                      │
│                                                                  │
│  MODES:                                                         │
│  • Session: Most compatible, hold for session                  │
│  • Transaction: Best for web, release after txn               │
│  • Statement: Most efficient, no transactions                  │
│                                                                  │
│  TOOLS:                                                         │
│  • PgBouncer: External pooler for PostgreSQL                   │
│  • RDS Proxy: AWS managed pooler                               │
│  • pg Pool: Application-level pooling                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```
