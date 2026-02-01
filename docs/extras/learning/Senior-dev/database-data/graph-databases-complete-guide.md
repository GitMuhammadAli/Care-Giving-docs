# 🕸️ Graph Databases - Complete Guide

> A comprehensive guide to graph databases - Neo4j, relationships, traversals, and when graph databases outperform relational databases.

---

## 🧠 MUST REMEMBER TO IMPRESS

### 1-Liner Definition
> "Graph databases store data as nodes and relationships, enabling efficient traversal of connected data - finding paths, recommendations, and patterns that would require expensive recursive JOINs in relational databases."

### Key Terms
| Term | Meaning |
|------|---------|
| **Node** | Entity (Person, Product, Post) |
| **Relationship** | Connection between nodes (FOLLOWS, PURCHASED) |
| **Property** | Key-value data on nodes or relationships |
| **Traversal** | Walking through the graph following relationships |
| **Cypher** | Neo4j's query language |

---

## Core Concepts

```
GRAPH vs RELATIONAL:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  RELATIONAL (JOIN nightmare):                                  │
│  ┌───────┐     ┌────────────┐     ┌───────┐                   │
│  │ users │────►│ friendships│◄────│ users │                   │
│  └───────┘     └────────────┘     └───────┘                   │
│                                                                  │
│  "Find friends of friends of friends"                          │
│  = 3 self-joins, exponentially slow                            │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  GRAPH (natural):                                               │
│                                                                  │
│   (Alice)──FRIENDS──►(Bob)──FRIENDS──►(Carol)                  │
│      │                                    │                     │
│      └────────────FRIENDS────────────────┘                     │
│                                                                  │
│  "Find friends of friends of friends"                          │
│  = Walk the graph, consistently fast                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Neo4j Cypher Examples

```cypher
// ════════════════════════════════════════════════════════════════
// CREATING DATA
// ════════════════════════════════════════════════════════════════

// Create nodes
CREATE (alice:Person {name: 'Alice', age: 30})
CREATE (bob:Person {name: 'Bob', age: 25})
CREATE (post:Post {title: 'Graph DBs are cool'})

// Create relationships
MATCH (a:Person {name: 'Alice'}), (b:Person {name: 'Bob'})
CREATE (a)-[:FRIENDS_WITH {since: 2020}]->(b)

// ════════════════════════════════════════════════════════════════
// QUERYING
// ════════════════════════════════════════════════════════════════

// Find friends
MATCH (p:Person {name: 'Alice'})-[:FRIENDS_WITH]->(friend)
RETURN friend.name

// Friends of friends (2 hops) - EASY in graph!
MATCH (p:Person {name: 'Alice'})-[:FRIENDS_WITH*2]->(fof)
RETURN DISTINCT fof.name

// Shortest path
MATCH path = shortestPath(
    (a:Person {name: 'Alice'})-[:FRIENDS_WITH*]-(b:Person {name: 'Zoe'})
)
RETURN path

// Recommendation: People who bought this also bought
MATCH (p:Person)-[:PURCHASED]->(product:Product {name: 'iPhone'})
MATCH (p)-[:PURCHASED]->(other:Product)
WHERE other.name <> 'iPhone'
RETURN other.name, COUNT(*) as purchases
ORDER BY purchases DESC
LIMIT 5
```

---

## When to Use Graph DBs

```
USE GRAPH DATABASE WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ✓ Social networks (friends, followers)                        │
│  ✓ Recommendation engines                                      │
│  ✓ Fraud detection (connected patterns)                       │
│  ✓ Knowledge graphs                                            │
│  ✓ Network/IT infrastructure                                   │
│  ✓ Path finding, routing                                       │
│                                                                  │
│  DON'T USE WHEN:                                                │
│  ✗ Simple CRUD with no relationships                           │
│  ✗ Heavy aggregations (sum, avg)                               │
│  ✗ Time-series data                                            │
│  ✗ Full-text search as primary use                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "When would you use a graph database over relational?"**
> "When relationships are as important as data. Finding 'friends of friends' in SQL requires recursive self-JOINs that get exponentially slower. In Neo4j, it's a simple traversal that stays fast regardless of depth. Use cases: social networks, recommendations, fraud detection."

**Q: "What's the performance difference for relationship queries?"**
> "In relational DB, 'friends of friends of friends' might take seconds or time out. In graph DB, it's milliseconds because it follows direct pointers instead of JOINing tables. Graph DBs have index-free adjacency - relationships are physical pointers."

---

## Quick Reference

```
GRAPH DATABASE CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  CONCEPTS:                                                      │
│  • Nodes: Entities with labels and properties                  │
│  • Relationships: Typed, directional connections               │
│  • Traversal: Walking relationships to find patterns           │
│                                                                  │
│  TOOLS:                                                         │
│  • Neo4j: Most popular graph DB                                │
│  • Amazon Neptune: AWS managed                                 │
│  • ArangoDB: Multi-model (graph + document)                    │
│                                                                  │
│  USE CASES:                                                     │
│  • Social: Friends, followers, connections                     │
│  • Recommendations: People who bought X also bought Y          │
│  • Fraud: Unusual patterns in transactions                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```
