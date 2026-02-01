# Database Architecture Overview

> Understanding CareCircle's data layer.

---

## The Mental Model

Think of the database layer like a **library system**:

- **PostgreSQL** = The library building (stores all books)
- **Tables** = Different sections (fiction, non-fiction, reference)
- **Prisma** = The librarian (knows where everything is, helps you find things)
- **Redis** = Your personal bookmarks (quick access to frequently used pages)
- **Migrations** = The renovation crew (changes the building structure safely)

---

## Why PostgreSQL?

### The Decision Matrix

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    WHY POSTGRESQL FOR CARECIRCLE?                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  REQUIREMENT                    │  POSTGRESQL ANSWER                        │
│  ───────────                    │  ─────────────────                        │
│  Family relationships           │  Foreign keys, JOIN operations            │
│  Medication schedules           │  Date/time types, constraints             │
│  Data integrity                 │  ACID transactions                        │
│  Flexible queries               │  Rich SQL, complex WHERE clauses          │
│  JSON for settings              │  JSONB column type                        │
│  Full-text search               │  Built-in tsvector                        │
│  Serverless hosting             │  Neon DB support                          │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                    WHY NOT ALTERNATIVES?                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  MongoDB                                                                     │
│  ───────                                                                     │
│  ❌ CareCircle has strong relationships (families, medications, users)      │
│  ❌ We need JOIN operations constantly                                      │
│  ❌ Document model = duplicate data or multiple queries                     │
│                                                                              │
│  MySQL                                                                       │
│  ─────                                                                       │
│  ❌ JSONB support less mature                                               │
│  ❌ Neon DB only supports PostgreSQL                                        │
│  ⚠️  Would work, but PostgreSQL is industry preference for new projects    │
│                                                                              │
│  SQLite                                                                      │
│  ──────                                                                      │
│  ❌ Can't scale horizontally                                                │
│  ❌ Not suitable for web applications with concurrent users                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Data Model Philosophy

### Entity Relationships

```
                                  ┌─────────────┐
                                  │    User     │
                                  │  ─────────  │
                                  │  id         │
                                  │  email      │
                                  │  password   │
                                  └──────┬──────┘
                                         │
                        ┌────────────────┼────────────────┐
                        │                │                │
                        ▼                ▼                ▼
               ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
               │FamilyMember │  │  Session    │  │PushToken    │
               │ ─────────── │  │  ───────    │  │ ─────────   │
               │ role        │  │  token      │  │ endpoint    │
               │ familyId    │  │  expiresAt  │  │ keys        │
               └──────┬──────┘  └─────────────┘  └─────────────┘
                      │
         ┌────────────┴────────────┐
         │                         │
         ▼                         ▼
┌─────────────────┐       ┌─────────────────┐
│     Family      │       │ CareRecipient   │
│  ────────────   │       │  ─────────────  │
│  name           │◄──────│  familyId       │
│  settings       │       │  name           │
└─────────────────┘       │  medicalInfo    │
                          └────────┬────────┘
                                   │
           ┌───────────────┬───────┴───────┬───────────────┐
           │               │               │               │
           ▼               ▼               ▼               ▼
    ┌───────────┐   ┌───────────┐   ┌───────────┐   ┌───────────┐
    │Medication │   │Appointment│   │  Doctor   │   │ Emergency │
    │───────────│   │───────────│   │ ───────   │   │ Contact   │
    │name       │   │title      │   │name       │   │───────────│
    │dosage     │   │datetime   │   │specialty  │   │name       │
    │schedule   │   │recurring  │   │phone      │   │phone      │
    └───────────┘   └───────────┘   └───────────┘   └───────────┘
```

### Key Design Decisions

#### 1. Multi-tenancy via Family

```
All care data is scoped to a Family:

- User → FamilyMember → Family → CareRecipient → Medication

This means:
• Query: "Get medications" always includes familyId in WHERE
• Security: Users can only see their family's data
• Isolation: Families cannot see each other's data
```

#### 2. Soft Deletes vs Hard Deletes

```
HARD DELETE (we use for):
─────────────────────────
• Sessions (security, cleanup)
• Push tokens (no historical value)

SOFT DELETE (we use for):
─────────────────────────
• Medications (audit trail, restore)
• Appointments (history matters)
• Users (legal compliance)

Implementation:
  deletedAt: DateTime?  // null = active, timestamp = deleted
  
Query pattern:
  WHERE deletedAt IS NULL  // Exclude soft-deleted
```

#### 3. UUID vs Auto-increment IDs

```
We use UUIDs because:
─────────────────────

✅ No sequence guessing (security)
   Auto-increment: /users/1, /users/2, /users/3...
   UUID: /users/a1b2c3d4-e5f6-... (unpredictable)

✅ Merge-friendly (distributed systems)
   IDs generated by any server without coordination

✅ URL-safe (with proper encoding)

Trade-off:
❌ Larger storage (36 chars vs 4 bytes for int)
❌ Slightly slower indexes (but negligible at our scale)
```

---

## Why Prisma?

### The ORM Decision

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PRISMA VS ALTERNATIVES                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PRISMA                                                                      │
│  ──────                                                                      │
│  ✅ Type-safe queries (TypeScript integration)                              │
│  ✅ Auto-generated types from schema                                        │
│  ✅ Declarative schema (easy to read)                                       │
│  ✅ Great migration tooling                                                 │
│  ✅ Excellent DX (developer experience)                                     │
│  ❌ Can't do everything raw SQL can                                         │
│  ❌ Generated client is large                                               │
│                                                                              │
│  TypeORM                                                                     │
│  ───────                                                                     │
│  ❌ Decorator-heavy (magic)                                                 │
│  ❌ Types often out of sync with DB                                         │
│  ❌ More complex query builder                                              │
│  ✅ More SQL-like flexibility                                               │
│                                                                              │
│  Drizzle                                                                     │
│  ───────                                                                     │
│  ✅ Lighter weight                                                          │
│  ✅ SQL-like syntax                                                         │
│  ❌ Newer, smaller ecosystem                                                │
│  ❌ Migration tooling less mature                                           │
│                                                                              │
│  Raw SQL                                                                     │
│  ───────                                                                     │
│  ✅ Maximum flexibility                                                     │
│  ✅ Maximum performance                                                     │
│  ❌ No type safety                                                          │
│  ❌ SQL injection risk                                                      │
│  ❌ Manual migration management                                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Prisma Mental Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PRISMA WORKFLOW                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. SCHEMA (Single source of truth)                                         │
│     │                                                                        │
│     │  schema.prisma:                                                       │
│     │  model User {                                                         │
│     │    id    String @id @default(uuid())                                  │
│     │    email String @unique                                               │
│     │    name  String                                                       │
│     │  }                                                                    │
│     │                                                                        │
│     ▼                                                                        │
│  2. GENERATE (Types & Client)                                               │
│     │                                                                        │
│     │  npx prisma generate                                                  │
│     │  → Creates TypeScript types                                           │
│     │  → Creates query methods                                              │
│     │                                                                        │
│     ▼                                                                        │
│  3. MIGRATE (Database changes)                                              │
│     │                                                                        │
│     │  npx prisma migrate dev                                               │
│     │  → Creates SQL migration file                                         │
│     │  → Applies to database                                                │
│     │                                                                        │
│     ▼                                                                        │
│  4. USE (Type-safe queries)                                                 │
│                                                                              │
│     const user = await prisma.user.findUnique({                             │
│       where: { email: 'test@example.com' },                                 │
│     });                                                                      │
│     // TypeScript knows user.name is string                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Query Patterns

### The N+1 Problem (And How to Avoid It)

```
THE PROBLEM:
────────────

// Get families with members
const families = await prisma.family.findMany();  // 1 query

for (const family of families) {
  const members = await prisma.familyMember.findMany({
    where: { familyId: family.id }  // N queries!
  });
}

// Total: 1 + N queries (if 100 families = 101 queries!)


THE SOLUTION:
─────────────

// Use Prisma's include
const families = await prisma.family.findMany({
  include: {
    members: true,  // Fetched in same query
  }
});

// Total: 1 query with JOIN
```

### When to Use What Query Pattern

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      QUERY PATTERN DECISION TREE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Need related data?                                                          │
│       │                                                                      │
│   YES │                                                                      │
│       ▼                                                                      │
│  All fields needed?                                                          │
│       │                                                                      │
│   YES │ NO                                                                   │
│       ▼ ▼                                                                    │
│  include:    select:                                                         │
│  {           { relation: { select: {...} } }                                │
│    relation: true                                                            │
│  }                                                                           │
│                                                                              │
│  Need to filter/sort related?                                                │
│       │                                                                      │
│   YES │                                                                      │
│       ▼                                                                      │
│  include: {                                                                  │
│    relation: {                                                               │
│      where: { status: 'active' },                                           │
│      orderBy: { createdAt: 'desc' }                                         │
│    }                                                                         │
│  }                                                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Transactions

### When to Use Transactions

```
USE TRANSACTION WHEN:
─────────────────────

• Multiple writes that must ALL succeed or ALL fail
• Reading data that informs a write (read-modify-write)
• Money/credits movement (never lose or duplicate)

EXAMPLES:

1. Medication Logged + Notification Created
   → If notification fails, medication log should also fail

2. User Deleted + All Related Data Deleted
   → Must clean up everything or nothing

3. Transfer Credits Between Users
   → Must debit one AND credit other


DON'T NEED TRANSACTION FOR:
───────────────────────────

• Independent operations that can partially fail
• Read-only queries
• Single create/update/delete
```

### Transaction Patterns

```typescript
// Pattern 1: Interactive Transaction
const result = await prisma.$transaction(async (tx) => {
  // tx is a transaction client - use it instead of prisma
  
  const medication = await tx.medication.update({
    where: { id: medicationId },
    data: { lastLoggedAt: new Date() },
  });
  
  const log = await tx.medicationLog.create({
    data: { medicationId, timestamp: new Date(), status: 'taken' },
  });
  
  return { medication, log };
});
// If either fails, both are rolled back


// Pattern 2: Batch Operations
const [deletedUser, deletedMemberships] = await prisma.$transaction([
  prisma.user.delete({ where: { id: userId } }),
  prisma.familyMember.deleteMany({ where: { userId } }),
]);
// Array of operations, all or nothing
```

---

## Indexing Strategy

### The Mental Model

Think of indexes like a **book's index**:

```
WITHOUT INDEX:
──────────────
"Find all medications for care recipient X"
→ Database scans EVERY medication row
→ 1,000,000 rows = 1,000,000 checks
→ SLOW


WITH INDEX on careRecipientId:
──────────────────────────────
"Find all medications for care recipient X"
→ Database looks up X in index
→ Jumps directly to matching rows
→ Only checks relevant rows
→ FAST
```

### CareCircle's Index Strategy

```
INDEXES WE HAVE:
────────────────

Primary Keys (automatic):
• @id on every table

Foreign Keys (explicit for performance):
• @@index([familyId])           // Filter by family
• @@index([careRecipientId])    // Filter by care recipient
• @@index([userId])             // Filter by user
• @@index([createdById])        // Filter by creator

Composite for common queries:
• @@index([careRecipientId, status])  // Active meds for recipient
• @@index([familyId, scheduledAt])    // Appointments in family

Unique constraints:
• @unique([email])              // One account per email
• @unique([familyId, userId])   // One membership per family per user
```

### When to Add Indexes

```
ADD INDEX IF:
─────────────
• Column used in WHERE frequently
• Column used in JOIN
• Column used in ORDER BY
• Query is slow in production

DON'T ADD INDEX IF:
───────────────────
• Table is small (< 1000 rows)
• Column has low cardinality (true/false)
• Writes >> Reads on this table
• "Just in case" (over-indexing hurts writes)
```

---

## Redis as Cache & Queue Storage

### Why Redis?

```
PostgreSQL is for:                Redis is for:
──────────────────               ───────────────
• Source of truth                • Speed (in-memory)
• Complex queries                • Simple key-value
• Relationships                  • Ephemeral data
• ACID guarantees                • Pub/sub
• Long-term storage              • Job queues
```

### CareCircle's Redis Usage

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         REDIS USE CASES                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  CACHING                                                                     │
│  ───────                                                                     │
│  Key: "family:{id}:medications"                                             │
│  Value: JSON array of medications                                           │
│  TTL: 5 minutes                                                             │
│  Purpose: Avoid repeated DB queries                                         │
│                                                                              │
│  JOB QUEUES (BullMQ)                                                        │
│  ───────────────────                                                        │
│  Key: "bull:notifications:*"                                                │
│  Value: Job data and state                                                  │
│  Purpose: Background job processing                                         │
│                                                                              │
│  RATE LIMITING                                                              │
│  ─────────────                                                              │
│  Key: "ratelimit:{ip}:{endpoint}"                                          │
│  Value: Request count                                                       │
│  TTL: 1 minute                                                              │
│  Purpose: Prevent abuse                                                     │
│                                                                              │
│  SESSION TOKENS (optional)                                                  │
│  ────────────────────────                                                   │
│  Key: "session:{token}"                                                     │
│  Value: User ID and metadata                                                │
│  TTL: Token expiration time                                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Common Mistakes & How to Avoid Them

### Mistake 1: Over-fetching Data

```typescript
❌ WRONG: Fetch everything

const medications = await prisma.medication.findMany({
  include: {
    careRecipient: {
      include: {
        family: {
          include: {
            members: {
              include: {
                user: true  // HUGE amount of data!
              }
            }
          }
        }
      }
    }
  }
});

✅ RIGHT: Fetch what you need

const medications = await prisma.medication.findMany({
  select: {
    id: true,
    name: true,
    dosage: true,
    careRecipient: {
      select: { name: true }
    }
  }
});
```

### Mistake 2: Forgetting Pagination

```typescript
❌ WRONG: No limit

const allLogs = await prisma.medicationLog.findMany();
// Could be 100,000 rows!

✅ RIGHT: Always paginate

const logs = await prisma.medicationLog.findMany({
  take: 50,
  skip: page * 50,
  orderBy: { createdAt: 'desc' }
});
```

### Mistake 3: Storing Derived Data

```typescript
❌ WRONG: Store calculated values

model User {
  id          String @id
  firstName   String
  lastName    String
  fullName    String  // Redundant! Gets out of sync!
}

✅ RIGHT: Calculate when needed

// In application code
const fullName = `${user.firstName} ${user.lastName}`;

// Or use Prisma computed fields (client extension)
```

---

## Quick Reference

### Common Prisma Operations

| Operation | Method | Example |
|-----------|--------|---------|
| Find one | `findUnique` | `prisma.user.findUnique({ where: { id } })` |
| Find first match | `findFirst` | `prisma.user.findFirst({ where: { email } })` |
| Find all | `findMany` | `prisma.user.findMany({ where: { active: true } })` |
| Create | `create` | `prisma.user.create({ data: {...} })` |
| Update | `update` | `prisma.user.update({ where: { id }, data: {...} })` |
| Upsert | `upsert` | `prisma.user.upsert({ where: { email }, create: {...}, update: {...} })` |
| Delete | `delete` | `prisma.user.delete({ where: { id } })` |
| Count | `count` | `prisma.user.count({ where: {...} })` |
| Aggregate | `aggregate` | `prisma.order.aggregate({ _sum: { total: true } })` |

### Database Commands Cheatsheet

```bash
# Generate Prisma Client after schema changes
npx prisma generate

# Create a migration
npx prisma migrate dev --name descriptive_name

# Apply migrations in production
npx prisma migrate deploy

# View database in browser
npx prisma studio

# Reset database (DANGER: deletes all data)
npx prisma migrate reset

# Push schema without migration (dev only)
npx prisma db push
```

---

*Next: [PostgreSQL Deep Dive](postgresql.md) | [Prisma Patterns](prisma.md)*


