# Prisma Migrations

> Database schema management and version control.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | Database migration system |
| **Why** | Schema versioning, team collaboration, deployments |
| **Tool** | Prisma Migrate |
| **Location** | `packages/database/prisma/migrations/` |

## Migration Commands

```bash
# Generate migration from schema changes
pnpm --filter @carecircle/database db:migrate:create

# Apply migrations (development)
pnpm --filter @carecircle/database db:migrate

# Apply migrations (production)
pnpm --filter @carecircle/database db:migrate:deploy

# Reset database (drops all data!)
pnpm --filter @carecircle/database db:reset

# Push schema without migration (development only)
pnpm --filter @carecircle/database db:push

# Check migration status
pnpm --filter @carecircle/database db:migrate:status

# Generate Prisma Client
pnpm --filter @carecircle/database db:generate
```

## Migration Workflow

### 1. Make Schema Changes

```prisma
// packages/database/prisma/schema.prisma

// Add new field
model User {
  // ... existing fields
  preferences Json? // NEW FIELD
}

// Add new model
model AuditLog {
  id           String   @id @default(uuid())
  action       String
  userId       String
  resourceType String
  resourceId   String
  createdAt    DateTime @default(now())
  
  user User @relation(fields: [userId], references: [id])
}
```

### 2. Create Migration

```bash
pnpm --filter @carecircle/database db:migrate:create

# Interactive prompt:
# ? Enter a name for the new migration: add_user_preferences_and_audit_log
```

### 3. Review Migration

```sql
-- migrations/20260130_add_user_preferences_and_audit_log/migration.sql

-- AlterTable
ALTER TABLE "User" ADD COLUMN "preferences" JSONB;

-- CreateTable
CREATE TABLE "AuditLog" (
    "id" TEXT NOT NULL,
    "action" TEXT NOT NULL,
    "userId" TEXT NOT NULL,
    "resourceType" TEXT NOT NULL,
    "resourceId" TEXT NOT NULL,
    "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT "AuditLog_pkey" PRIMARY KEY ("id")
);

-- CreateIndex
CREATE INDEX "AuditLog_userId_idx" ON "AuditLog"("userId");

-- AddForeignKey
ALTER TABLE "AuditLog" ADD CONSTRAINT "AuditLog_userId_fkey" 
    FOREIGN KEY ("userId") REFERENCES "User"("id") ON DELETE CASCADE ON UPDATE CASCADE;
```

### 4. Apply Migration

```bash
# Development
pnpm --filter @carecircle/database db:migrate

# Production (CI/CD)
pnpm --filter @carecircle/database db:migrate:deploy
```

## Migration Best Practices

### 1. Small, Focused Migrations

```bash
# ✅ Good: One change per migration
add_user_avatar_field
add_medication_refill_alert
create_audit_log_table

# ❌ Bad: Multiple unrelated changes
update_schema_v2
big_refactor
```

### 2. Make Migrations Reversible

```prisma
// Schema change that's easy to reverse
model User {
  preferences Json? // Nullable, can be removed later
}
```

### 3. Handle Data Migrations Separately

```sql
-- migrations/20260130_add_default_preferences/migration.sql

-- Update existing users with default preferences
UPDATE "User" 
SET preferences = '{"notifications": true, "theme": "system"}'::jsonb
WHERE preferences IS NULL;
```

### 4. Test Migrations

```bash
# 1. Create database backup
pg_dump -U postgres carecircle > backup.sql

# 2. Apply migration
pnpm db:migrate

# 3. Run tests
pnpm test

# 4. If issues, restore
psql -U postgres carecircle < backup.sql
```

## Migration File Structure

```
packages/database/prisma/
├── schema.prisma
└── migrations/
    ├── 20260101000000_initial/
    │   └── migration.sql
    ├── 20260115000000_add_medication_refill/
    │   └── migration.sql
    ├── 20260130000000_add_audit_log/
    │   └── migration.sql
    └── migration_lock.toml
```

## Handling Common Scenarios

### Adding Required Field

```sql
-- Step 1: Add as nullable
ALTER TABLE "Medication" ADD COLUMN "refillAt" INTEGER;

-- Step 2: Backfill data
UPDATE "Medication" SET "refillAt" = 10 WHERE "refillAt" IS NULL;

-- Step 3: Make required
ALTER TABLE "Medication" ALTER COLUMN "refillAt" SET NOT NULL;
```

### Renaming Field

```sql
-- Prisma uses @map for renames without data loss
ALTER TABLE "User" RENAME COLUMN "name" TO "fullName";
```

In schema:
```prisma
model User {
  fullName String @map("name") // Maps to old column
}
```

### Dropping Field

```prisma
// 1. Mark as deprecated in code (don't use)
// 2. Remove from schema
// 3. Create migration
model User {
  // oldField String // REMOVED
}
```

### Adding Index

```prisma
model Medication {
  careRecipientId String
  isActive        Boolean

  @@index([careRecipientId, isActive])
}
```

## Production Deployment

### CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
jobs:
  deploy:
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Generate Prisma Client
        run: pnpm db:generate
      
      - name: Run migrations
        run: pnpm db:migrate:deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
      
      - name: Deploy application
        run: # deployment steps
```

### Zero-Downtime Migrations

1. **Expand**: Add new columns/tables (nullable)
2. **Migrate**: Update application to use new schema
3. **Contract**: Remove old columns after verification

```sql
-- Phase 1: Expand
ALTER TABLE "User" ADD COLUMN "newField" TEXT;

-- Phase 2: Application deployment (uses both old and new)

-- Phase 3: Contract (after verification)
ALTER TABLE "User" DROP COLUMN "oldField";
```

## Troubleshooting

### Migration Drift

```bash
# Check if schema matches migrations
pnpm prisma migrate diff \
  --from-schema-datasource prisma/schema.prisma \
  --to-migrations prisma/migrations \
  --shadow-database-url "postgresql://..."
```

### Failed Migration

```bash
# Mark migration as applied (careful!)
pnpm prisma migrate resolve --applied 20260130000000_migration_name

# Or roll back manually
pnpm prisma migrate resolve --rolled-back 20260130000000_migration_name
```

### Reset Development Database

```bash
# Drops all data and re-applies migrations
pnpm db:reset

# Then seed with test data
pnpm db:seed
```

---

*See also: [PostgreSQL](postgresql.md), [Prisma](../backend/prisma.md)*


