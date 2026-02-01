# 🔍 Full-Text Search - Complete Guide

> A comprehensive guide to full-text search - Elasticsearch, Algolia, indexing strategies, relevance tuning, and building powerful search experiences.

---

## 🧠 MUST REMEMBER TO IMPRESS

### 1-Liner Definition
> "Full-text search uses inverted indexes to find documents containing search terms, with features like tokenization, stemming, and relevance scoring - enabling Google-like search in milliseconds across millions of documents."

### Key Terms
| Term | Meaning |
|------|---------|
| **Inverted index** | Maps terms → documents (opposite of document → terms) |
| **Tokenization** | Breaking text into searchable tokens |
| **Stemming** | "running" → "run" for broader matching |
| **TF-IDF** | Term frequency × inverse document frequency (relevance) |
| **BM25** | Modern relevance scoring algorithm |

---

## Core Concepts

```
INVERTED INDEX:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  DOCUMENTS:                     INVERTED INDEX:                  │
│  Doc1: "React hooks tutorial"   ┌────────┬─────────────┐        │
│  Doc2: "React state guide"      │ Term   │ Documents   │        │
│  Doc3: "Vue hooks intro"        ├────────┼─────────────┤        │
│                                 │ react  │ Doc1, Doc2  │        │
│                                 │ hooks  │ Doc1, Doc3  │        │
│                                 │ tutorial│ Doc1       │        │
│                                 │ state  │ Doc2        │        │
│                                 │ vue    │ Doc3        │        │
│                                 └────────┴─────────────┘        │
│                                                                  │
│  Search "react hooks":                                         │
│  → "react" in Doc1, Doc2                                       │
│  → "hooks" in Doc1, Doc3                                       │
│  → Intersection: Doc1 (most relevant)                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Elasticsearch vs Algolia vs PostgreSQL

```
COMPARISON:
┌───────────────┬─────────────────┬─────────────┬────────────────┐
│ Feature       │ Elasticsearch   │ Algolia     │ PostgreSQL FTS │
├───────────────┼─────────────────┼─────────────┼────────────────┤
│ Hosting       │ Self/Cloud      │ Managed     │ Your DB        │
│ Speed         │ Fast            │ Fastest     │ Good           │
│ Scale         │ Massive         │ Large       │ Medium         │
│ Complexity    │ High            │ Low         │ Low            │
│ Cost          │ Infrastructure  │ Per search  │ Free           │
│ Best for      │ Large scale     │ SaaS search │ Simple search  │
└───────────────┴─────────────────┴─────────────┴────────────────┘
```

### Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// ELASTICSEARCH EXAMPLE
// ════════════════════════════════════════════════════════════════

import { Client } from '@elastic/elasticsearch';

const client = new Client({ node: 'http://localhost:9200' });

// Create index with mapping
await client.indices.create({
    index: 'products',
    body: {
        mappings: {
            properties: {
                name: { type: 'text', analyzer: 'english' },
                description: { type: 'text', analyzer: 'english' },
                category: { type: 'keyword' },  // Exact match
                price: { type: 'float' }
            }
        }
    }
});

// Index a document
await client.index({
    index: 'products',
    id: '1',
    body: {
        name: 'Wireless Bluetooth Headphones',
        description: 'High quality noise cancelling headphones',
        category: 'electronics',
        price: 99.99
    }
});

// Search with relevance
const result = await client.search({
    index: 'products',
    body: {
        query: {
            bool: {
                must: [
                    { match: { name: 'headphones' } }
                ],
                filter: [
                    { term: { category: 'electronics' } },
                    { range: { price: { lte: 150 } } }
                ]
            }
        }
    }
});

// ════════════════════════════════════════════════════════════════
// POSTGRESQL FULL-TEXT SEARCH
// ════════════════════════════════════════════════════════════════

-- Create search vector column
ALTER TABLE products ADD COLUMN search_vector tsvector;

UPDATE products SET search_vector = 
    to_tsvector('english', name || ' ' || description);

-- Create GIN index
CREATE INDEX idx_products_search ON products USING gin(search_vector);

-- Search
SELECT * FROM products
WHERE search_vector @@ to_tsquery('english', 'headphones & wireless')
ORDER BY ts_rank(search_vector, to_tsquery('english', 'headphones')) DESC;
```

---

## Interview Questions

**Q: "How does full-text search differ from LIKE queries?"**
> "LIKE does string matching, no relevance scoring, can't use indexes with leading wildcards. Full-text uses inverted indexes for O(1) term lookup, scores by relevance (TF-IDF/BM25), handles stemming ('running' matches 'run'), and scales to billions of documents."

**Q: "When would you use Elasticsearch vs PostgreSQL FTS?"**
> "PostgreSQL FTS for simple search in existing Postgres app - no extra infrastructure. Elasticsearch for large scale, complex queries, faceted search, or when you need sub-100ms response on millions of documents."

---

## Quick Reference

```
FULL-TEXT SEARCH CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  KEY CONCEPTS:                                                  │
│  • Inverted index: term → documents                            │
│  • Tokenization: text → searchable terms                       │
│  • Relevance: TF-IDF, BM25 scoring                            │
│                                                                  │
│  OPTIMIZATION:                                                  │
│  • Denormalize for search (flatten nested data)                │
│  • Use keyword type for filters, text for search               │
│  • Tune relevance with field boosting                          │
│                                                                  │
│  TOOLS:                                                         │
│  • Elasticsearch: Most powerful, complex                       │
│  • Algolia: Fastest, managed                                   │
│  • PostgreSQL: Simple, built-in                                │
│  • Meilisearch: Modern, easy                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```
