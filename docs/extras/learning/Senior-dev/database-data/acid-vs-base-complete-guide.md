# ⚖️ ACID vs BASE - Complete Guide

> A comprehensive guide to database consistency models - ACID transactions, eventual consistency, CAP theorem, and choosing the right model for your use case.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definitions
> **ACID**: "Atomicity, Consistency, Isolation, Durability - transactions are all-or-nothing, data is always valid, transactions don't interfere, committed data survives failures."
> 
> **BASE**: "Basically Available, Soft state, Eventually consistent - prioritizes availability over immediate consistency, accepts temporary inconsistency."

### The "Wow" Statement
> "We used PostgreSQL (ACID) for our payment system because money transfers must be atomic - you can't debit one account without crediting another. But for our activity feed, we use Cassandra (BASE) with eventual consistency - it's okay if a user sees a post 2 seconds late, and we need the availability and scale. The key is matching consistency requirements to business needs: bank transfers need ACID, social feeds can use BASE."

### CAP Theorem (Memorize!)
```
CAP THEOREM:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  You can only have 2 of 3 during a network partition:          │
│                                                                  │
│              CONSISTENCY                                        │
│                  /\                                             │
│                 /  \                                            │
│                /    \                                           │
│           CP /      \ CA                                        │
│             /        \                                          │
│            /          \                                         │
│   PARTITION ──────────── AVAILABILITY                           │
│   TOLERANCE      AP                                             │
│                                                                  │
│  CP: Consistent + Partition tolerant (sacrifice availability)  │
│      PostgreSQL, MongoDB (w/ majority writes)                   │
│      → Returns error if can't guarantee consistency            │
│                                                                  │
│  AP: Available + Partition tolerant (sacrifice consistency)    │
│      Cassandra, DynamoDB, CouchDB                              │
│      → Returns stale data rather than error                    │
│                                                                  │
│  CA: Consistent + Available (no partition tolerance)           │
│      Single-node databases only - impractical                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## ACID Properties

```
ACID EXPLAINED:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  A - ATOMICITY                                                  │
│  Transaction is all-or-nothing                                 │
│  Either all operations succeed or none do                      │
│                                                                  │
│  BEGIN;                                                        │
│    UPDATE accounts SET balance = balance - 100 WHERE id = 1;   │
│    UPDATE accounts SET balance = balance + 100 WHERE id = 2;   │
│  COMMIT;  -- Both happen or neither happens                    │
│                                                                  │
│  C - CONSISTENCY                                                │
│  Data always valid according to rules (constraints)            │
│  Transaction moves DB from one valid state to another          │
│                                                                  │
│  I - ISOLATION                                                  │
│  Concurrent transactions don't interfere                       │
│  Each transaction sees consistent snapshot                     │
│                                                                  │
│  D - DURABILITY                                                 │
│  Committed data survives crashes                               │
│  Written to disk, not just memory                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Isolation Levels

```sql
-- ════════════════════════════════════════════════════════════════
-- ISOLATION LEVELS (from weakest to strongest)
-- ════════════════════════════════════════════════════════════════

-- READ UNCOMMITTED: See uncommitted changes (dirty reads)
-- Rarely used - data can disappear if other transaction rolls back

-- READ COMMITTED: See only committed changes
-- Default in PostgreSQL - safe but can see changes mid-transaction
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- REPEATABLE READ: Same query returns same result within transaction
-- Prevents non-repeatable reads
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- SERIALIZABLE: Transactions execute as if sequential
-- Strongest isolation, lowest concurrency
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

/*
ISOLATION LEVEL     | Dirty Read | Non-Repeatable | Phantom
────────────────────┼────────────┼────────────────┼─────────
READ UNCOMMITTED    | Possible   | Possible       | Possible
READ COMMITTED      | No         | Possible       | Possible
REPEATABLE READ     | No         | No             | Possible
SERIALIZABLE        | No         | No             | No
*/
```

---

## BASE Properties

```
BASE EXPLAINED:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  BA - BASICALLY AVAILABLE                                       │
│  System always responds (may be stale data)                    │
│  Prioritizes availability over consistency                     │
│                                                                  │
│  S - SOFT STATE                                                 │
│  State may change over time without input                      │
│  Due to eventual consistency propagation                       │
│                                                                  │
│  E - EVENTUALLY CONSISTENT                                      │
│  System will become consistent given enough time               │
│  All replicas converge to same value                           │
│                                                                  │
│  EXAMPLE:                                                       │
│  User A posts → Replica 1 has it                               │
│  User B reads → Hits Replica 2 → No post yet                   │
│  ... 2 seconds later ...                                       │
│  User B reads → Hits Replica 2 → Now sees post                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## When to Use Each

```
CHOOSING CONSISTENCY MODEL:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  USE ACID WHEN:                                                 │
│  • Financial transactions (money transfers)                    │
│  • Inventory management (prevent overselling)                  │
│  • User authentication                                         │
│  • Order processing                                            │
│  • Anything where inconsistency = money loss or security risk  │
│                                                                  │
│  USE BASE WHEN:                                                 │
│  • Social media feeds (okay if post appears late)              │
│  • Analytics and metrics (approximate is fine)                 │
│  • Session data (can regenerate)                               │
│  • Caching layers                                              │
│  • High-scale reads where availability > consistency           │
│                                                                  │
│  HYBRID APPROACH:                                               │
│  • ACID for writes (PostgreSQL)                                │
│  • BASE for reads (Redis cache, Elasticsearch)                 │
│  • Sync via CDC or events                                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "Explain ACID properties"**
> "Atomicity: all-or-nothing transactions. Consistency: data always valid per constraints. Isolation: concurrent transactions don't interfere. Durability: committed data survives crashes. PostgreSQL, MySQL give you ACID guarantees."

**Q: "What is eventual consistency?"**
> "After a write, replicas may temporarily have different values, but will converge to the same value given enough time. Used in distributed systems like Cassandra, DynamoDB where availability is more important than immediate consistency."

**Q: "Explain CAP theorem"**
> "In a distributed system during network partition, you can only have 2 of 3: Consistency, Availability, Partition tolerance. Since partitions happen, you choose CP (return error if can't be consistent) or AP (return stale data to stay available)."

---

## Quick Reference

```
ACID vs BASE CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ACID:                           BASE:                          │
│  • Strong consistency            • Eventual consistency        │
│  • Transactions                  • Availability first          │
│  • Single source of truth        • Multiple replicas           │
│  • PostgreSQL, MySQL             • Cassandra, DynamoDB         │
│  • Financial, inventory          • Social, analytics           │
│                                                                  │
│  CAP: Pick 2 during partition                                  │
│  • CP: Consistent + Partition (PostgreSQL)                     │
│  • AP: Available + Partition (Cassandra)                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```
