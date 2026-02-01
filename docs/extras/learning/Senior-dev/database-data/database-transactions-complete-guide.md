# 🔐 Database Transactions - Complete Guide

> A comprehensive guide to database transactions - isolation levels, deadlocks, optimistic vs pessimistic locking, and ensuring data integrity in concurrent environments.

---

## 🧠 MUST REMEMBER TO IMPRESS

### 1-Liner Definition
> "A database transaction is an atomic unit of work that either fully completes or fully fails, with isolation levels controlling how concurrent transactions see each other's changes, and locking strategies preventing conflicts."

### Key Terms
| Term | Meaning |
|------|---------|
| **Isolation level** | How much transactions see each other's uncommitted changes |
| **Deadlock** | Two transactions waiting for each other's locks |
| **Optimistic locking** | Check for conflicts at commit time (version field) |
| **Pessimistic locking** | Lock rows upfront to prevent conflicts |
| **SELECT FOR UPDATE** | Lock rows for update within transaction |

---

## Isolation Levels

```
ISOLATION LEVELS (weakest to strongest):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  READ UNCOMMITTED                                               │
│  • Can see uncommitted changes (dirty reads)                   │
│  • Rarely used                                                 │
│                                                                  │
│  READ COMMITTED (PostgreSQL default)                           │
│  • Only see committed changes                                  │
│  • Same query can return different results                     │
│                                                                  │
│  REPEATABLE READ (MySQL default)                               │
│  • Snapshot at transaction start                               │
│  • Same query returns same results                             │
│                                                                  │
│  SERIALIZABLE                                                   │
│  • As if transactions ran sequentially                         │
│  • Highest isolation, lowest concurrency                       │
│                                                                  │
│  PROBLEMS PREVENTED:                                            │
│  Level           │ Dirty │ Non-repeat │ Phantom │              │
│  ─────────────────┼───────┼────────────┼─────────│              │
│  Read Uncommitted │  ✗    │     ✗      │    ✗    │              │
│  Read Committed   │  ✓    │     ✗      │    ✗    │              │
│  Repeatable Read  │  ✓    │     ✓      │    ✗    │              │
│  Serializable     │  ✓    │     ✓      │    ✓    │              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Locking Strategies

```sql
-- ════════════════════════════════════════════════════════════════
-- PESSIMISTIC LOCKING (lock rows upfront)
-- ════════════════════════════════════════════════════════════════

-- SELECT FOR UPDATE: Lock row, others wait
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- Row is locked, other transactions wait
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;  -- Lock released

-- NOWAIT: Fail immediately if locked
SELECT * FROM accounts WHERE id = 1 FOR UPDATE NOWAIT;
-- ERROR if row is locked

-- SKIP LOCKED: Skip locked rows (for job queues)
SELECT * FROM jobs WHERE status = 'pending' 
FOR UPDATE SKIP LOCKED LIMIT 1;

-- ════════════════════════════════════════════════════════════════
-- OPTIMISTIC LOCKING (version check at update)
-- ════════════════════════════════════════════════════════════════

-- Add version column to table
ALTER TABLE accounts ADD COLUMN version INT DEFAULT 0;

-- Read with version
SELECT id, balance, version FROM accounts WHERE id = 1;
-- Returns: id=1, balance=100, version=5

-- Update with version check
UPDATE accounts 
SET balance = 90, version = version + 1
WHERE id = 1 AND version = 5;

-- If version changed (another update happened), 0 rows affected
-- Application must retry or fail
```

### Optimistic Locking Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// OPTIMISTIC LOCKING IN APPLICATION
// ════════════════════════════════════════════════════════════════

async function updateAccountBalance(
    accountId: string, 
    amount: number, 
    maxRetries = 3
) {
    for (let attempt = 0; attempt < maxRetries; attempt++) {
        // Read current state
        const account = await db.query(
            'SELECT balance, version FROM accounts WHERE id = $1',
            [accountId]
        );
        
        const newBalance = account.balance + amount;
        
        // Update with version check
        const result = await db.query(`
            UPDATE accounts 
            SET balance = $1, version = version + 1
            WHERE id = $2 AND version = $3
        `, [newBalance, accountId, account.version]);
        
        if (result.rowCount === 1) {
            return { success: true, balance: newBalance };
        }
        
        // Version mismatch - retry
        console.log(`Optimistic lock conflict, retry ${attempt + 1}`);
    }
    
    throw new Error('Failed after max retries');
}

// Prisma optimistic locking
await prisma.account.update({
    where: { 
        id: accountId,
        version: currentVersion  // Implicit version check
    },
    data: { 
        balance: newBalance,
        version: { increment: 1 }
    }
});
```

### Deadlock Prevention

```sql
-- ════════════════════════════════════════════════════════════════
-- DEADLOCK SCENARIO
-- ════════════════════════════════════════════════════════════════

-- Transaction 1:
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;  -- Locks row 1
UPDATE accounts SET balance = balance + 100 WHERE id = 2;  -- Waits for row 2

-- Transaction 2 (concurrent):
BEGIN;
UPDATE accounts SET balance = balance - 50 WHERE id = 2;   -- Locks row 2
UPDATE accounts SET balance = balance + 50 WHERE id = 1;   -- Waits for row 1

-- DEADLOCK! Both waiting for each other

-- ════════════════════════════════════════════════════════════════
-- PREVENTION: Consistent lock ordering
-- ════════════════════════════════════════════════════════════════

-- Always lock in same order (by ID)
async function transfer(fromId, toId, amount) {
    // Sort IDs to ensure consistent lock order
    const [firstId, secondId] = [fromId, toId].sort();
    
    await db.query('BEGIN');
    await db.query('SELECT * FROM accounts WHERE id = $1 FOR UPDATE', [firstId]);
    await db.query('SELECT * FROM accounts WHERE id = $1 FOR UPDATE', [secondId]);
    
    // Now safe to update
    await db.query('UPDATE accounts SET balance = balance - $1 WHERE id = $2', 
        [amount, fromId]);
    await db.query('UPDATE accounts SET balance = balance + $1 WHERE id = $2', 
        [amount, toId]);
    await db.query('COMMIT');
}
```

---

## Interview Questions

**Q: "Optimistic vs pessimistic locking?"**
> "Pessimistic: Lock rows upfront with SELECT FOR UPDATE - guaranteed no conflicts but blocks other transactions. Optimistic: Use version field, check at update time - more concurrency but must handle conflicts. Use pessimistic for high contention, optimistic for low contention."

**Q: "How do you prevent deadlocks?"**
> "Consistent lock ordering - always acquire locks in same order (e.g., by ID). If all transactions lock account 1 before account 2, no circular wait can occur. Also keep transactions short and use appropriate isolation level."

**Q: "Explain isolation levels"**
> "Read Committed (Postgres default): see only committed data, but same query can return different results. Repeatable Read: snapshot isolation, same results throughout transaction. Serializable: as if sequential, highest isolation but lowest concurrency. Choose based on consistency vs performance needs."

---

## Quick Reference

```
TRANSACTIONS CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ISOLATION LEVELS:                                              │
│  • Read Committed: Default, sees committed changes             │
│  • Repeatable Read: Snapshot, consistent reads                 │
│  • Serializable: Strongest, slowest                            │
│                                                                  │
│  LOCKING:                                                       │
│  • Pessimistic: FOR UPDATE (block others)                      │
│  • Optimistic: Version check at commit                         │
│  • SKIP LOCKED: For job queues                                 │
│                                                                  │
│  DEADLOCK PREVENTION:                                           │
│  • Consistent lock ordering                                    │
│  • Keep transactions short                                     │
│  • Use timeouts (NOWAIT)                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```
