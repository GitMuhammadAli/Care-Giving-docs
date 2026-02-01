# 📄 NoSQL Patterns - Complete Guide

> A comprehensive guide to NoSQL patterns - document design, denormalization, when to use NoSQL, and modeling data for MongoDB, DynamoDB, and other NoSQL databases.

---

## 🧠 MUST REMEMBER TO IMPRESS

### 1-Liner Definition
> "NoSQL databases sacrifice some relational features (joins, ACID) for horizontal scalability, schema flexibility, and optimized access patterns - requiring you to model data around your queries, not your entities."

### Key Terms
| Term | Meaning |
|------|---------|
| **Document** | Self-contained JSON/BSON record |
| **Denormalization** | Duplicating data to avoid joins |
| **Embedding** | Nesting related data in one document |
| **Reference** | Storing ID to link documents |
| **Single table design** | DynamoDB pattern: all entities in one table |

---

## Core Concepts

```
SQL vs NOSQL MODELING:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SQL: Model your ENTITIES (normalize)                           │
│  ───────────────────────────────────                            │
│  users table ←──── orders table ←──── items table              │
│  Join to get complete order                                    │
│                                                                  │
│  NoSQL: Model your QUERIES (denormalize)                       │
│  ─────────────────────────────────────────                      │
│  {                                                              │
│    "orderId": "123",                                           │
│    "user": { "name": "John", "email": "..." },  // Embedded   │
│    "items": [                                                  │
│      { "name": "Widget", "price": 10 },         // Embedded   │
│      { "name": "Gadget", "price": 20 }                        │
│    ]                                                           │
│  }                                                              │
│  One read gets everything!                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When to Embed vs Reference

```
EMBEDDING (denormalize):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  USE WHEN:                                                      │
│  ✓ Data is read together (order + items)                       │
│  ✓ Related data is owned by parent                             │
│  ✓ 1:1 or 1:few relationships                                  │
│  ✓ Data doesn't change independently                           │
│                                                                  │
│  {                                                              │
│    "_id": "post123",                                           │
│    "title": "NoSQL Guide",                                     │
│    "author": { "name": "John", "avatar": "..." },  // Embed   │
│    "comments": [ { "text": "Great!" } ]             // Embed   │
│  }                                                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

REFERENCING (normalize):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  USE WHEN:                                                      │
│  ✓ Data is accessed independently                              │
│  ✓ Data changes frequently                                     │
│  ✓ 1:many or many:many relationships                           │
│  ✓ Document would exceed size limit                            │
│                                                                  │
│  // Post document                                              │
│  { "_id": "post123", "authorId": "user456" }                   │
│                                                                  │
│  // Separate user document (updated independently)             │
│  { "_id": "user456", "name": "John", "followers": 1000 }       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### DynamoDB Single-Table Design

```
SINGLE TABLE DESIGN (DynamoDB):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  All entities in ONE table, differentiated by PK/SK patterns   │
│                                                                  │
│  PK              │ SK              │ Data                       │
│  ────────────────┼─────────────────┼───────────────────────     │
│  USER#123        │ PROFILE         │ { name: "John", ... }     │
│  USER#123        │ ORDER#001       │ { total: 50, ... }        │
│  USER#123        │ ORDER#002       │ { total: 75, ... }        │
│  ORDER#001       │ ITEM#001        │ { product: "Widget" }     │
│  ORDER#001       │ ITEM#002        │ { product: "Gadget" }     │
│                                                                  │
│  Query user's orders: PK = "USER#123", SK begins_with "ORDER#" │
│  Query order items: PK = "ORDER#001", SK begins_with "ITEM#"   │
│                                                                  │
│  Benefits:                                                      │
│  • One table = simpler ops                                     │
│  • Adjacent data = fast queries                                │
│  • No joins needed                                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## When to Use NoSQL

```
USE NOSQL WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ✓ Schema changes frequently                                   │
│  ✓ Need horizontal scaling                                     │
│  ✓ Access patterns are known and simple                       │
│  ✓ Data is naturally hierarchical (documents)                  │
│  ✓ High write throughput needed                                │
│                                                                  │
│  DON'T USE WHEN:                                                │
│  ✗ Complex queries/joins needed                                │
│  ✗ ACID transactions required                                  │
│  ✗ Data relationships are complex                              │
│  ✗ Ad-hoc queries are common                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "When would you choose NoSQL over SQL?"**
> "When I need horizontal scaling, schema flexibility, and have well-defined access patterns. Document DBs for hierarchical data, key-value for sessions/cache, wide-column for time-series. I'd avoid NoSQL when I need complex queries or strong consistency."

**Q: "Explain denormalization in NoSQL"**
> "Storing duplicate data to avoid joins. In SQL, you normalize to avoid duplication. In NoSQL, you duplicate intentionally because there are no joins. Trade-off: faster reads, harder updates (must update in multiple places)."

---

## Quick Reference

```
NOSQL PATTERNS CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  MODELING RULES:                                                │
│  • Model for queries, not entities                             │
│  • Embed for 1:1 and 1:few                                     │
│  • Reference for 1:many and many:many                          │
│  • Denormalize for read performance                            │
│                                                                  │
│  DATABASE TYPES:                                                │
│  • Document: MongoDB, CouchDB (flexible JSON)                  │
│  • Key-Value: Redis, DynamoDB (simple lookups)                 │
│  • Wide-Column: Cassandra (time-series)                        │
│  • Graph: Neo4j (relationships)                                │
│                                                                  │
│  TRADE-OFFS:                                                    │
│  ✓ Scale, flexibility, speed                                   │
│  ✗ No joins, eventual consistency, duplicated data             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```
