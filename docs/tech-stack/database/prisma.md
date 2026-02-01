# Prisma ORM Concepts

> Understanding type-safe database access with Prisma.

---

## 1. What Is Prisma?

### Plain English Explanation

Prisma is an **ORM (Object-Relational Mapper)** that lets you work with databases using TypeScript instead of raw SQL.

Think of it like a **translator**:
- You speak TypeScript
- Database speaks SQL
- Prisma translates between them
- Plus, it catches your mistakes at compile time!

### The Core Problem Prisma Solves

```
WITHOUT ORM:
────────────
const result = await db.query(
  'SELECT * FROM users WHERE id = $1', 
  [userId]
);
// result.rows[0].name - Is 'name' correct? No way to know until runtime!

WITH PRISMA:
────────────
const user = await prisma.user.findUnique({ where: { id: userId } });
// user.name - TypeScript knows this exists! Autocomplete works!
```

---

## 2. Core Concepts & Terminology

### The Prisma Ecosystem

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PRISMA COMPONENTS                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PRISMA SCHEMA (schema.prisma)                                              │
│  ─────────────────────────────                                              │
│  • Define your data models                                                  │
│  • Single source of truth                                                   │
│  • Database-agnostic                                                        │
│                                                                              │
│  PRISMA CLIENT (@prisma/client)                                             │
│  ──────────────────────────────                                             │
│  • Auto-generated TypeScript client                                         │
│  • Type-safe queries                                                        │
│  • What you import in your code                                             │
│                                                                              │
│  PRISMA MIGRATE                                                             │
│  ─────────────────                                                          │
│  • Manage database schema changes                                           │
│  • Version-controlled migrations                                            │
│  • Safe production deployments                                              │
│                                                                              │
│  PRISMA STUDIO                                                              │
│  ─────────────                                                              │
│  • Visual database browser                                                  │
│  • View and edit data                                                       │
│  • Great for debugging                                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Terminology

| Term | Definition |
|------|------------|
| **Model** | Database table representation |
| **Field** | Column in a table |
| **Relation** | Connection between models |
| **Migration** | Schema change script |
| **Client** | Generated query interface |

---

## 3. How Prisma Works

### The Workflow

```
1. DEFINE SCHEMA
   Write models in schema.prisma

2. GENERATE CLIENT
   npx prisma generate
   Creates TypeScript client from schema

3. MIGRATE DATABASE
   npx prisma migrate dev
   Creates and applies SQL migrations

4. USE IN CODE
   Import and use the typed client
```

### Schema → Types → Queries

```prisma
// schema.prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String
  createdAt DateTime @default(now())
}
```

Generates:

```typescript
// Auto-generated types
type User = {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
}

// Auto-generated queries
prisma.user.findUnique({ where: { id } })
prisma.user.findMany({ where: { name: 'John' } })
prisma.user.create({ data: { email, name } })
prisma.user.update({ where: { id }, data: { name } })
prisma.user.delete({ where: { id } })
```

---

## 4. Why Prisma for CareCircle

| Requirement | Why Prisma Fits |
|-------------|-----------------|
| Type safety | Full TypeScript integration |
| Complex relations | Families, medications, users all connected |
| Migration management | Safe schema evolution |
| Developer experience | Autocomplete, error prevention |
| PostgreSQL support | First-class Neon DB support |

---

## 5. When to Use Prisma Patterns ✅

### Use `include` When:
- You need related data
- You'll use the related data immediately

### Use `select` When:
- You only need specific fields
- Performance matters (large tables)

### Use Transactions When:
- Multiple writes must all succeed
- Data consistency is critical

### Use Raw Queries When:
- Prisma can't express the query
- Performance-critical aggregations
- Database-specific features

---

## 6. When to AVOID Patterns ❌

### DON'T Over-include

```
❌ BAD: Fetch everything
const user = await prisma.user.findUnique({
  where: { id },
  include: {
    families: {
      include: {
        members: {
          include: { user: true }
        }
      }
    }
  }
});
// Huge data payload!

✅ GOOD: Fetch what you need
const user = await prisma.user.findUnique({
  where: { id },
  select: {
    id: true,
    name: true,
    families: {
      select: { id: true, name: true }
    }
  }
});
```

### DON'T Forget Pagination

```
❌ BAD: Fetch all records
const medications = await prisma.medication.findMany();
// Could be 100,000 rows!

✅ GOOD: Paginate
const medications = await prisma.medication.findMany({
  take: 20,
  skip: page * 20,
});
```

---

## 7. Best Practices & Recommendations

### Query Patterns

```typescript
// Find one
const user = await prisma.user.findUnique({ where: { id } });
const user = await prisma.user.findFirst({ where: { email } });

// Find many
const users = await prisma.user.findMany({
  where: { role: 'ADMIN' },
  orderBy: { createdAt: 'desc' },
  take: 10,
});

// Create
const user = await prisma.user.create({
  data: { email, name },
});

// Update
const user = await prisma.user.update({
  where: { id },
  data: { name: 'New Name' },
});

// Upsert (create or update)
const user = await prisma.user.upsert({
  where: { email },
  create: { email, name },
  update: { name },
});

// Delete
await prisma.user.delete({ where: { id } });
```

### Relation Queries

```typescript
// Include relation
const family = await prisma.family.findUnique({
  where: { id },
  include: { members: true },
});

// Nested include
const family = await prisma.family.findUnique({
  where: { id },
  include: {
    members: {
      include: { user: true }
    }
  },
});

// Filter relations
const family = await prisma.family.findUnique({
  where: { id },
  include: {
    members: {
      where: { role: 'ADMIN' }
    }
  },
});
```

---

## 8. Common Mistakes & How to Avoid Them

### Mistake 1: N+1 Queries

```typescript
❌ BAD: Query in a loop
const families = await prisma.family.findMany();
for (const family of families) {
  const members = await prisma.familyMember.findMany({
    where: { familyId: family.id }
  });
}
// N+1 queries!

✅ GOOD: Use include
const families = await prisma.family.findMany({
  include: { members: true }
});
// 1 query with JOIN
```

### Mistake 2: Forgetting to Generate

After changing `schema.prisma`, run:
```bash
npx prisma generate
```

### Mistake 3: Not Using Transactions

```typescript
❌ BAD: Separate operations
await prisma.account.update({ where: { id: from }, data: { balance: { decrement: 100 } } });
await prisma.account.update({ where: { id: to }, data: { balance: { increment: 100 } } });
// If second fails, money disappears!

✅ GOOD: Transaction
await prisma.$transaction([
  prisma.account.update({ where: { id: from }, data: { balance: { decrement: 100 } } }),
  prisma.account.update({ where: { id: to }, data: { balance: { increment: 100 } } }),
]);
```

---

## 9. Migration Workflow

### Development

```bash
# Create migration from schema changes
npx prisma migrate dev --name add_medications_table

# Reset database (DANGER: deletes data)
npx prisma migrate reset

# Push without migration (prototyping only)
npx prisma db push
```

### Production

```bash
# Apply pending migrations
npx prisma migrate deploy
```

---

## 10. Quick Reference

### CLI Commands

| Command | Purpose |
|---------|---------|
| `npx prisma generate` | Generate client from schema |
| `npx prisma migrate dev` | Create and apply migration |
| `npx prisma migrate deploy` | Apply migrations in production |
| `npx prisma studio` | Open visual database browser |
| `npx prisma db push` | Push schema without migration |

### Common Query Methods

| Method | Purpose |
|--------|---------|
| `findUnique` | Find by unique field |
| `findFirst` | Find first match |
| `findMany` | Find all matches |
| `create` | Create one record |
| `createMany` | Create multiple records |
| `update` | Update one record |
| `updateMany` | Update multiple records |
| `upsert` | Create or update |
| `delete` | Delete one record |
| `deleteMany` | Delete multiple records |
| `count` | Count records |
| `aggregate` | Sum, avg, min, max |

---

## 11. Learning Resources

- [Prisma Documentation](https://www.prisma.io/docs)
- [Prisma's YouTube Channel](https://www.youtube.com/c/PrismaData)
- [Prisma Day Talks](https://www.prisma.io/day)

---

*Next: [Database Design](data-modeling.md) | [Migrations](migrations.md)*

