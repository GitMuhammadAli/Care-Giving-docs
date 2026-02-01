# 🔄 Database Migrations - Complete Guide

> A comprehensive guide to database migrations - schema versioning, zero-downtime migrations, rollbacks, and best practices for evolving database schemas safely.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Database migrations are versioned, sequential schema changes that transform your database from one state to another, with the ability to roll back - enabling safe evolution of your data model alongside application code."

### The "Wow" Statement (Memorize This!)
> "We needed to add a non-nullable column to a 50-million row table in production without downtime. Instead of one risky migration, I split it into 4 steps: 1) Add nullable column, 2) Backfill in batches of 10K rows with sleep intervals, 3) Deploy app code handling both states, 4) Add NOT NULL constraint. Each step was independently deployable and rollback-safe. The whole migration took 3 hours but zero user impact. The key was the expand-contract pattern: make the schema handle both old and new code, then contract once all code is updated."

### Key Terms to Drop
| Term | Use It Like This |
|------|------------------|
| **"Expand-contract"** | "Using expand-contract pattern: add new column, backfill, remove old" |
| **"Zero-downtime migration"** | "All our migrations are zero-downtime compatible" |
| **"Idempotent"** | "Migrations should be idempotent - safe to run multiple times" |
| **"Backfill"** | "Backfilling data in batches to avoid locking" |
| **"Forward-compatible"** | "Schema is forward-compatible with both old and new app versions" |

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How do you handle database migrations in production?"

**Senior Answer:**
> "I follow zero-downtime migration principles:

**1. Never break backward compatibility**
- New code must work with old schema
- Old code must work with new schema (during deployment)

**2. Expand-contract pattern**
- Expand: Add new structures (columns, tables)
- Migrate: Move/copy data
- Contract: Remove old structures (in separate deploy)

**3. Batch operations**
- Never UPDATE all rows at once
- Use batches with delays to prevent lock contention

**4. Separate deploy from migrate**
- Deploy code that handles both states
- Run migration
- Deploy code that uses new state

This adds complexity but prevents 3am incidents."

---

## 📚 Key Patterns

### Zero-Downtime Migration Steps

```
SAFE MIGRATION PATTERN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ADDING A NEW COLUMN:                                           │
│                                                                  │
│  Step 1: Add nullable column                                   │
│  ────────────────────────────                                   │
│  ALTER TABLE users ADD COLUMN phone TEXT;                      │
│  -- Instant, no data change                                    │
│                                                                  │
│  Step 2: Deploy code that writes to new column                 │
│  ──────────────────────────────────────────                    │
│  INSERT INTO users (name, phone) VALUES ('John', '555-1234');  │
│                                                                  │
│  Step 3: Backfill existing rows (if needed)                   │
│  ─────────────────────────────────────────                     │
│  UPDATE users SET phone = 'unknown' WHERE phone IS NULL        │
│  LIMIT 10000;  -- In batches!                                  │
│                                                                  │
│  Step 4: Add constraint (after all data populated)            │
│  ───────────────────────────────────────────────               │
│  ALTER TABLE users ALTER COLUMN phone SET NOT NULL;            │
│                                                                  │
│  RENAMING A COLUMN:                                             │
│                                                                  │
│  ❌ BAD: ALTER TABLE users RENAME email TO email_address;      │
│     -- Breaks old code immediately!                            │
│                                                                  │
│  ✓ GOOD: Expand-Contract                                       │
│  1. Add new column (email_address)                             │
│  2. Write to both columns                                      │
│  3. Backfill new from old                                      │
│  4. Deploy code using new column                               │
│  5. Stop writing to old column                                 │
│  6. Drop old column (separate deploy)                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Batch Updates

```sql
-- ════════════════════════════════════════════════════════════════
-- BATCH UPDATE PATTERN (PostgreSQL)
-- ════════════════════════════════════════════════════════════════

-- ❌ BAD: Locks entire table
UPDATE users SET status = 'active' WHERE status IS NULL;

-- ✓ GOOD: Process in batches
DO $$
DECLARE
    batch_size INT := 10000;
    rows_updated INT;
BEGIN
    LOOP
        UPDATE users 
        SET status = 'active'
        WHERE id IN (
            SELECT id FROM users 
            WHERE status IS NULL 
            LIMIT batch_size
            FOR UPDATE SKIP LOCKED
        );
        
        GET DIAGNOSTICS rows_updated = ROW_COUNT;
        
        IF rows_updated = 0 THEN
            EXIT;
        END IF;
        
        -- Pause to let other transactions proceed
        PERFORM pg_sleep(0.1);
        
        RAISE NOTICE 'Updated % rows', rows_updated;
    END LOOP;
END $$;
```

### Migration Tools

```typescript
// ════════════════════════════════════════════════════════════════
// PRISMA MIGRATIONS
// ════════════════════════════════════════════════════════════════

// Generate migration
// npx prisma migrate dev --name add_phone_column

// prisma/migrations/20240115_add_phone_column/migration.sql
/*
-- CreateColumn
ALTER TABLE "users" ADD COLUMN "phone" TEXT;

-- BackfillData (add manually for existing data)
UPDATE "users" SET "phone" = 'unknown' WHERE "phone" IS NULL;
*/

// Deploy migration
// npx prisma migrate deploy

// ════════════════════════════════════════════════════════════════
// KNEX MIGRATIONS
// ════════════════════════════════════════════════════════════════

export async function up(knex: Knex): Promise<void> {
    // Check if column exists (idempotent)
    const hasColumn = await knex.schema.hasColumn('users', 'phone');
    if (!hasColumn) {
        await knex.schema.alterTable('users', (table) => {
            table.string('phone').nullable();
        });
    }
}

export async function down(knex: Knex): Promise<void> {
    const hasColumn = await knex.schema.hasColumn('users', 'phone');
    if (hasColumn) {
        await knex.schema.alterTable('users', (table) => {
            table.dropColumn('phone');
        });
    }
}
```

---

## Common Pitfalls

```
MIGRATION PITFALLS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. ADDING NOT NULL WITHOUT DEFAULT                             │
│     Problem: Fails if table has existing rows                  │
│     Solution: Add nullable first, backfill, then add constraint│
│                                                                  │
│  2. LARGE TABLE UPDATES IN ONE TRANSACTION                     │
│     Problem: Locks table, blocks all queries                   │
│     Solution: Batch updates with commits between               │
│                                                                  │
│  3. DROPPING COLUMNS USED BY OLD CODE                          │
│     Problem: Rolling deployment = old code crashes             │
│     Solution: Drop columns in separate deploy after code update│
│                                                                  │
│  4. NON-IDEMPOTENT MIGRATIONS                                  │
│     Problem: Re-running migration fails                        │
│     Solution: Always check if change already applied           │
│                                                                  │
│  5. NO ROLLBACK PLAN                                            │
│     Problem: Migration fails midway, stuck state               │
│     Solution: Test rollback, have manual recovery steps        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "How do you add a NOT NULL column to a large table without downtime?"**
> "Four-step process: 1) Add column as nullable, 2) Backfill in batches of 10K with delays, 3) Deploy code that always populates the column, 4) Add NOT NULL constraint. Each step is safe to rollback."

**Q: "What is the expand-contract pattern?"**
> "A migration pattern where you first expand the schema to support both old and new, then contract by removing old structures. Example: Rename column by adding new, copying data, deploying code for new, then dropping old."

**Q: "How do you handle migrations with rolling deployments?"**
> "Schema must be forward and backward compatible. During rolling deploy, some pods have old code, some have new. Both must work with current schema. This means: never remove columns until all pods updated, never require columns until all pods write them."

---

## Quick Reference

```
MIGRATION CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SAFE OPERATIONS (instant):                                     │
│  • Add nullable column                                          │
│  • Add index CONCURRENTLY                                       │
│  • Add table                                                    │
│                                                                  │
│  DANGEROUS OPERATIONS:                                          │
│  • Add NOT NULL column → add nullable + backfill + constraint  │
│  • Rename column → add new + copy + drop old                   │
│  • Change column type → add new + cast + drop old              │
│  • Large UPDATE → batch with commits                           │
│                                                                  │
│  PATTERNS:                                                      │
│  • Expand-contract for schema changes                          │
│  • Batch updates for data migrations                           │
│  • Feature flags for gradual rollout                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

*Guide covers patterns applicable to PostgreSQL, MySQL, and common ORMs (Prisma, Knex, TypeORM).*
