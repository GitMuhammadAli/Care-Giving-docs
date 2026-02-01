# 🔄 Replication & Failover - Complete Guide

> A comprehensive guide to database replication - master-slave setup, read replicas, consistency models, failover strategies, and high availability patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Database replication copies data from a primary to one or more replicas for read scaling, fault tolerance, and geographic distribution - while failover ensures automatic recovery when the primary fails."

### The Replication Mental Model
```
REPLICATION ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│                    WRITE REQUESTS                                │
│                         │                                        │
│                         ▼                                        │
│                 ┌───────────────┐                               │
│                 │    PRIMARY    │                               │
│                 │   (Master)    │                               │
│                 │               │                               │
│                 │  Writes ✓    │                               │
│                 │  Reads ✓     │                               │
│                 └───────┬───────┘                               │
│                         │                                        │
│            ┌────────────┼────────────┐                          │
│            │  Replication Stream     │                          │
│            │  (WAL / Binlog)         │                          │
│            ▼            ▼            ▼                          │
│    ┌───────────┐ ┌───────────┐ ┌───────────┐                   │
│    │ REPLICA 1 │ │ REPLICA 2 │ │ REPLICA 3 │                   │
│    │           │ │           │ │           │                   │
│    │ Reads ✓  │ │ Reads ✓  │ │ Reads ✓  │                   │
│    │ Writes ✗ │ │ Writes ✗ │ │ Writes ✗ │                   │
│    └───────────┘ └───────────┘ └───────────┘                   │
│            ▲            ▲            ▲                          │
│            │            │            │                          │
│            └────────────┴────────────┘                          │
│                         │                                        │
│                  READ REQUESTS                                   │
│              (Load balanced)                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Numbers to Remember
| Metric | Value | Context |
|--------|-------|---------|
| Replication lag | **0-100ms typical** | Can spike to seconds under load |
| Failover time (manual) | **Minutes** | Human intervention required |
| Failover time (auto) | **30-60 seconds** | With proper setup |
| Read replica scaling | **5-15 replicas** | Practical limit |
| Sync replication latency | **2-10ms extra** | Per write operation |

### The "Wow" Statement (Memorize This!)
> "We had a single PostgreSQL handling 10K reads/sec and hit a ceiling. I set up 3 read replicas with PgBouncer connection pooling - reads went to replicas, writes to primary. Immediately handled 40K reads/sec. The tricky part was replication lag: some users would create a post and not see it immediately because their read hit a lagging replica. We solved this with 'read-your-writes' consistency - after a write, that user's reads route to primary for 5 seconds. When our primary failed last year, our automated failover promoted a replica to primary in 45 seconds. We use synchronous replication to one replica for zero data loss, async to the others for performance."

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"WAL shipping"** | "Using WAL shipping for continuous replication to replicas" |
| **"Replication lag"** | "Seeing 500ms replication lag - need to investigate" |
| **"Synchronous vs async"** | "Sync replication for zero data loss, async for performance" |
| **"Read-your-writes"** | "Implemented read-your-writes consistency to avoid stale reads" |
| **"Failover"** | "Automatic failover promoted replica to primary in under a minute" |
| **"Split-brain"** | "Need fencing to prevent split-brain during failover" |
| **"Quorum"** | "Using quorum-based writes: write succeeds when majority confirms" |

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How does database replication work?"

**Junior Answer:**
> "Data is copied to multiple servers."

**Senior Answer:**
> "The primary captures all changes in a log (WAL for PostgreSQL, binlog for MySQL). Replicas consume this log and apply changes locally.

**Two modes:**
- **Asynchronous** - Primary doesn't wait for replicas. Fast, but data can be lost if primary fails before replica receives changes.
- **Synchronous** - Primary waits for at least one replica to confirm. Slower writes, but zero data loss.

**Trade-offs:**
- More replicas = better read scaling and fault tolerance
- Replication lag = stale reads (consistency issues)
- Sync replication = higher write latency

**Common patterns:**
- Sync to 1 replica (durability), async to others (read scaling)
- Route reads to replicas, writes to primary
- Read-your-writes for consistency-sensitive operations

The key challenge is handling replication lag - you need to decide if your application can tolerate stale reads or needs special handling."

---

## 📚 Table of Contents

1. [Replication Types](#1-replication-types)
2. [Consistency Models](#2-consistency-models)
3. [Failover Strategies](#3-failover-strategies)
4. [Implementation](#4-implementation)
5. [Common Pitfalls](#5-common-pitfalls)
6. [Interview Questions](#6-interview-questions)

---

## 1. Replication Types

### Synchronous vs Asynchronous

```
SYNCHRONOUS REPLICATION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. Client sends write                                          │
│  2. Primary writes to local WAL                                 │
│  3. Primary sends to replica                                    │
│  4. Replica writes and ACKs                                    │
│  5. Primary commits                                            │
│  6. Client gets success                                        │
│                                                                  │
│  Client ──► Primary ──► Replica                                │
│         │           │         │                                 │
│         │    WAL    │   ACK   │                                │
│         │   write   │         │                                 │
│         │           ◄─────────┘                                 │
│         ◄───────────┘ commit                                   │
│                                                                  │
│  ✓ Zero data loss (committed = replicated)                     │
│  ✗ Higher latency (wait for replica ACK)                       │
│  ✗ Replica failure blocks writes                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

ASYNCHRONOUS REPLICATION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. Client sends write                                          │
│  2. Primary writes to local WAL                                 │
│  3. Primary commits immediately                                │
│  4. Client gets success                                        │
│  5. (Later) Primary sends to replica                           │
│                                                                  │
│  Client ──► Primary ──────────────►                            │
│         │           │                                           │
│         │    WAL    │                                           │
│         │   write   │                                           │
│         ◄───────────┘ commit                                   │
│                                                                  │
│           (async) Primary ──► Replica                          │
│                                                                  │
│  ✓ Low latency writes                                          │
│  ✓ Replica failure doesn't affect writes                       │
│  ✗ Potential data loss (unreplicated transactions)             │
│  ✗ Replication lag (reads may be stale)                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### PostgreSQL Configuration

```sql
-- ════════════════════════════════════════════════════════════════
-- POSTGRESQL STREAMING REPLICATION SETUP
-- ════════════════════════════════════════════════════════════════

-- PRIMARY: postgresql.conf
/*
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
synchronous_commit = on          -- 'on' for sync, 'off' for async
synchronous_standby_names = 'replica1'  -- For sync replication
*/

-- PRIMARY: pg_hba.conf
/*
host replication replicator replica1.example.com/32 scram-sha-256
host replication replicator replica2.example.com/32 scram-sha-256
*/

-- Check replication status:
SELECT 
    client_addr,
    state,
    sent_lsn,
    write_lsn,
    flush_lsn,
    replay_lsn,
    sync_state
FROM pg_stat_replication;

-- Check replication lag:
SELECT 
    client_addr,
    pg_wal_lsn_diff(sent_lsn, replay_lsn) AS lag_bytes,
    replay_lag
FROM pg_stat_replication;
```

---

## 2. Consistency Models

### Read-Your-Writes Consistency

```typescript
// ════════════════════════════════════════════════════════════════
// PROBLEM: User creates post, doesn't see it (hit stale replica)
// SOLUTION: Route recent writers to primary for reads
// ════════════════════════════════════════════════════════════════

class DatabaseRouter {
    private redis: Redis;
    private primaryPool: Pool;
    private replicaPool: Pool;
    
    private WRITE_STICKY_SECONDS = 5;
    
    async write(userId: string, sql: string, params: any[]) {
        // Always write to primary
        const result = await this.primaryPool.query(sql, params);
        
        // Mark user as recent writer
        await this.redis.setex(
            `recent_writer:${userId}`,
            this.WRITE_STICKY_SECONDS,
            '1'
        );
        
        return result;
    }
    
    async read(userId: string, sql: string, params: any[]) {
        // Check if user recently wrote
        const recentlyWrote = await this.redis.get(`recent_writer:${userId}`);
        
        if (recentlyWrote) {
            // Route to primary for consistency
            return this.primaryPool.query(sql, params);
        }
        
        // Safe to use replica
        return this.replicaPool.query(sql, params);
    }
}

// Usage
const router = new DatabaseRouter();

// User creates a post
await router.write(userId, 'INSERT INTO posts (user_id, content) VALUES ($1, $2)', 
    [userId, 'Hello!']);

// Immediately reads their posts - goes to PRIMARY
const posts = await router.read(userId, 'SELECT * FROM posts WHERE user_id = $1', 
    [userId]);
// User sees their post!

// 5 seconds later, reads go back to replica
```

### Monotonic Reads

```typescript
// ════════════════════════════════════════════════════════════════
// PROBLEM: User refreshes page, sees older data (different replica)
// SOLUTION: Sticky sessions to same replica
// ════════════════════════════════════════════════════════════════

class MonotonicReadRouter {
    private replicas: Pool[];
    
    getReplicaForUser(userId: string): Pool {
        // Consistent hash to always route user to same replica
        const replicaIndex = this.hash(userId) % this.replicas.length;
        return this.replicas[replicaIndex];
    }
    
    private hash(str: string): number {
        let hash = 0;
        for (let i = 0; i < str.length; i++) {
            hash = ((hash << 5) - hash) + str.charCodeAt(i);
            hash = hash & hash;
        }
        return Math.abs(hash);
    }
}

// Or track last-seen position
class PositionBasedRouter {
    async read(userId: string, sql: string, params: any[]) {
        const lastPosition = await this.getLastSeenPosition(userId);
        
        // Find a replica that has caught up to this position
        for (const replica of this.replicas) {
            const position = await replica.query('SELECT pg_last_wal_replay_lsn()');
            if (position >= lastPosition) {
                const result = await replica.query(sql, params);
                await this.updateLastSeenPosition(userId, position);
                return result;
            }
        }
        
        // Fallback to primary if no replica is caught up
        return this.primary.query(sql, params);
    }
}
```

---

## 3. Failover Strategies

### Automatic Failover

```
AUTOMATIC FAILOVER PROCESS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. DETECTION                                                   │
│     • Health checks (every 1-5 seconds)                        │
│     • Consensus: multiple checkers agree primary is down       │
│     • Avoid false positives (network partition vs crash)       │
│                                                                  │
│  2. FENCING (Prevent Split-Brain)                               │
│     • Ensure old primary can't accept writes                   │
│     • STONITH: "Shoot The Other Node In The Head"              │
│     • Revoke network access, power off, or use lease           │
│                                                                  │
│  3. PROMOTION                                                   │
│     • Select best replica (most up-to-date)                    │
│     • Promote: pg_promote() or mysqladmin                      │
│     • Replica becomes new primary                              │
│                                                                  │
│  4. RECONFIGURATION                                             │
│     • Update connection strings / DNS                          │
│     • Point other replicas to new primary                      │
│     • Notify applications                                      │
│                                                                  │
│  TIMELINE:                                                      │
│  Detection: 5-10 seconds                                       │
│  Fencing: 5-10 seconds                                         │
│  Promotion: 5-15 seconds                                       │
│  DNS/Config: 5-30 seconds                                      │
│  Total: 30-60 seconds typical                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### PostgreSQL Failover with Patroni

```yaml
# ════════════════════════════════════════════════════════════════
# PATRONI: HA PostgreSQL cluster management
# ════════════════════════════════════════════════════════════════

# patroni.yml
scope: postgres-cluster
name: node1

restapi:
  listen: 0.0.0.0:8008
  connect_address: node1.example.com:8008

etcd:
  hosts: etcd1:2379,etcd2:2379,etcd3:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576  # 1MB
    synchronous_mode: true
    postgresql:
      use_pg_rewind: true
      parameters:
        max_connections: 100
        synchronous_commit: on
        
postgresql:
  listen: 0.0.0.0:5432
  connect_address: node1.example.com:5432
  data_dir: /var/lib/postgresql/data
  authentication:
    replication:
      username: replicator
      password: secret
```

```typescript
// Application connects via HAProxy or pgbouncer
// that handles routing to current primary

const config = {
    host: 'haproxy.example.com',  // HAProxy routes to current primary
    port: 5432,
    database: 'app'
};

// Or use Patroni REST API to find primary
async function getPrimaryHost(): Promise<string> {
    const response = await fetch('http://patroni:8008/leader');
    const data = await response.json();
    return data.conn_url;
}
```

---

## 4. Implementation

### Connection Pooling with Read/Write Splitting

```typescript
// ════════════════════════════════════════════════════════════════
// READ/WRITE SPLITTING WITH PGBOUNCER
// ════════════════════════════════════════════════════════════════

interface DatabaseConfig {
    primary: PoolConfig;
    replicas: PoolConfig[];
}

class ReadWritePool {
    private primaryPool: Pool;
    private replicaPools: Pool[];
    private currentReplica = 0;
    
    constructor(config: DatabaseConfig) {
        this.primaryPool = new Pool(config.primary);
        this.replicaPools = config.replicas.map(c => new Pool(c));
    }
    
    async query(sql: string, params: any[], options?: { write?: boolean }) {
        if (options?.write || this.isWriteQuery(sql)) {
            return this.primaryPool.query(sql, params);
        }
        return this.getReplicaPool().query(sql, params);
    }
    
    private isWriteQuery(sql: string): boolean {
        const normalized = sql.trim().toUpperCase();
        return normalized.startsWith('INSERT') ||
               normalized.startsWith('UPDATE') ||
               normalized.startsWith('DELETE') ||
               normalized.startsWith('CREATE') ||
               normalized.startsWith('DROP') ||
               normalized.startsWith('ALTER');
    }
    
    private getReplicaPool(): Pool {
        // Round-robin across replicas
        const pool = this.replicaPools[this.currentReplica];
        this.currentReplica = (this.currentReplica + 1) % this.replicaPools.length;
        return pool;
    }
    
    // For transactions (must use primary)
    async transaction<T>(fn: (client: PoolClient) => Promise<T>): Promise<T> {
        const client = await this.primaryPool.connect();
        try {
            await client.query('BEGIN');
            const result = await fn(client);
            await client.query('COMMIT');
            return result;
        } catch (error) {
            await client.query('ROLLBACK');
            throw error;
        } finally {
            client.release();
        }
    }
}
```

---

## 5. Common Pitfalls

```
REPLICATION PITFALLS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. IGNORING REPLICATION LAG                                    │
│     Problem: User writes, then reads stale data                │
│     Solution: Read-your-writes, sticky sessions                │
│                                                                  │
│  2. SYNC REPLICATION TO ALL REPLICAS                           │
│     Problem: One slow replica blocks all writes                │
│     Solution: Sync to 1, async to others                       │
│                                                                  │
│  3. NO MONITORING                                               │
│     Problem: Replication breaks, nobody notices               │
│     Solution: Alert on lag > threshold, replication status     │
│                                                                  │
│  4. MANUAL FAILOVER ONLY                                        │
│     Problem: 3am failure, 30 min to respond                    │
│     Solution: Automated failover (Patroni, RDS Multi-AZ)      │
│                                                                  │
│  5. SPLIT-BRAIN                                                 │
│     Problem: Two primaries accepting writes                    │
│     Solution: Proper fencing, quorum-based decisions          │
│                                                                  │
│  6. WRITES TO REPLICA                                           │
│     Problem: Accidental writes break replication               │
│     Solution: Read-only mode, connection routing               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Interview Questions

**Q: "Synchronous vs asynchronous replication?"**
> "Sync: Primary waits for replica ACK before commit. Zero data loss, but higher latency and replica failure affects writes. Async: Primary commits immediately, replicates later. Fast, but potential data loss on failure. Best practice: Sync to one replica, async to others."

**Q: "What is replication lag and how do you handle it?"**
> "Time/data difference between primary and replica. A write on primary isn't immediately visible on replica. Handle with: 1) Read-your-writes (route recent writers to primary), 2) Sticky sessions (same replica per user), 3) Accept eventual consistency for non-critical reads."

**Q: "How does failover work?"**
> "1) Detection: Health checks identify primary failure. 2) Fencing: Ensure old primary can't write (prevent split-brain). 3) Promotion: Best replica becomes primary. 4) Reconfiguration: Update DNS/routing. Automated with tools like Patroni takes 30-60 seconds."

**Q: "What is split-brain?"**
> "When two nodes both think they're the primary and accept writes. Causes data divergence. Prevent with: proper fencing (STONITH), quorum-based decisions, lease-based leadership. Critical for data integrity."

---

## Quick Reference

```
REPLICATION CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SYNC VS ASYNC:                                                 │
│  • Sync: Zero data loss, higher latency                        │
│  • Async: Low latency, potential data loss                     │
│  • Best: Sync to 1, async to others                            │
│                                                                  │
│  CONSISTENCY PATTERNS:                                          │
│  • Read-your-writes: Route recent writers to primary           │
│  • Monotonic reads: Sticky sessions to same replica            │
│  • Causal consistency: Track dependencies                      │
│                                                                  │
│  FAILOVER:                                                      │
│  • Detection → Fencing → Promotion → Reconfiguration           │
│  • Automated: 30-60 seconds                                    │
│  • Tools: Patroni, RDS Multi-AZ                                │
│                                                                  │
│  MONITORING:                                                    │
│  • Replication lag (bytes and time)                            │
│  • Replica state (streaming, catching up)                      │
│  • Connection count per pool                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

*Guide covers PostgreSQL and MySQL replication. Cloud services (RDS, Cloud SQL) handle much of this automatically.*
