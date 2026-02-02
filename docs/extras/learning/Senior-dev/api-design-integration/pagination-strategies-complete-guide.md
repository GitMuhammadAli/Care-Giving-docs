# 📄 Pagination Strategies - Complete Guide

> A comprehensive guide to API pagination - cursor vs offset, infinite scroll, keyset pagination, and building efficient paginated APIs.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Pagination is the practice of dividing large datasets into smaller, manageable chunks that clients can request incrementally, improving performance, reducing bandwidth, and providing better user experience when dealing with large collections."

### The 7 Key Concepts (Remember These!)
```
1. OFFSET/LIMIT     → page=2&limit=20 (skip N, take M)
2. CURSOR-BASED     → after=cursor123 (continue from marker)
3. KEYSET           → after_id=100&after_date=2024-01-01
4. PAGE NUMBER      → page=5 (simple page navigation)
5. LINK HEADERS     → RFC 5988 navigation links
6. TOTAL COUNT      → Optional, expensive for large datasets
7. RELAY CONNECTION → GraphQL standard pagination
```

### Pagination Strategies Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│               PAGINATION STRATEGIES                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  OFFSET/LIMIT (Traditional)                                    │
│  ──────────────────────────                                     │
│  GET /users?offset=20&limit=10                                 │
│  ✅ Simple to implement                                        │
│  ✅ Jump to any page                                           │
│  ❌ Inconsistent with changing data                            │
│  ❌ Performance degrades with large offsets                    │
│  Use: Small datasets, admin panels                             │
│                                                                 │
│  CURSOR-BASED (Opaque cursor)                                  │
│  ────────────────────────────                                   │
│  GET /users?after=eyJpZCI6MTAwfQ==                             │
│  ✅ Consistent pagination                                      │
│  ✅ Efficient for large datasets                               │
│  ❌ Can't jump to specific page                                │
│  ❌ Cursor may expire                                          │
│  Use: Feeds, timelines, infinite scroll                        │
│                                                                 │
│  KEYSET (Seek method)                                          │
│  ────────────────────                                           │
│  GET /users?after_id=100&after_created=2024-01-01             │
│  ✅ Best performance (uses index)                              │
│  ✅ Consistent results                                         │
│  ❌ Requires unique, sequential column                         │
│  ❌ More complex queries                                       │
│  Use: Large datasets, real-time data                           │
│                                                                 │
│  PAGE NUMBER (Simple)                                          │
│  ────────────────────                                           │
│  GET /users?page=5&per_page=20                                 │
│  ✅ Intuitive for users                                        │
│  ❌ Same issues as offset                                      │
│  Use: Traditional web pages                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### The Offset Problem
```
┌─────────────────────────────────────────────────────────────────┐
│                   THE OFFSET PROBLEM                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SCENARIO: User on page 2, new item added to page 1            │
│                                                                 │
│  BEFORE (Page 2, offset=10):                                   │
│  Page 1: [1,2,3,4,5,6,7,8,9,10]                                │
│  Page 2: [11,12,13,14,15,16,17,18,19,20] ← User here           │
│                                                                 │
│  AFTER (New item 0 added):                                     │
│  Page 1: [0,1,2,3,4,5,6,7,8,9]                                 │
│  Page 2: [10,11,12,13,14,15,16,17,18,19] ← Shifted!            │
│                                                                 │
│  PROBLEM:                                                      │
│  • Item 10 appears again on page 2                             │
│  • User sees duplicate                                         │
│  • Or worse: user misses item if deleted                       │
│                                                                 │
│  SOLUTION: Cursor-based pagination                             │
│  "Give me items after item 20" - always consistent             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

PERFORMANCE COMPARISON:
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  OFFSET: SELECT * FROM users LIMIT 10 OFFSET 1000000           │
│          Database scans 1,000,010 rows, discards 1,000,000     │
│          ⚠️ Gets slower as offset increases                    │
│                                                                 │
│  KEYSET: SELECT * FROM users WHERE id > 1000000 LIMIT 10       │
│          Database uses index, jumps directly to position       │
│          ✅ Constant time regardless of position               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Seek method"** | "We use seek method for efficient keyset pagination" |
| **"Opaque cursor"** | "Cursors are opaque - clients shouldn't parse them" |
| **"Deep pagination"** | "Offset pagination degrades with deep pagination" |
| **"Relay connection"** | "We follow Relay connection spec for GraphQL" |
| **"Link headers"** | "Navigation provided via RFC 5988 Link headers" |
| **"Stable sort"** | "Pagination requires stable sort order" |

### Key Numbers to Remember
| Metric | Recommendation | Why |
|--------|----------------|-----|
| Default page size | **20-50** | Balance UX and performance |
| Max page size | **100** | Prevent resource abuse |
| Total count threshold | **> 10,000** | Skip for large datasets |
| Cursor expiration | **24h** | Prevent stale cursors |

### The "Wow" Statement (Memorize This!)
> "We use cursor-based pagination for our feeds and infinite scroll. Cursors are base64-encoded, opaque to clients - they contain the last item's ID and sort field values. This gives consistent results even when data changes, unlike offset which can show duplicates or miss items. For performance, we use keyset pagination under the hood - `WHERE id > last_id` uses the index directly, constant time regardless of position. We skip total count for large collections (expensive COUNT query), instead indicating if there's a next page. Response follows RFC 5988 with Link headers for prev/next/first/last. For admin panels where page jumping is needed, we use offset but limit max offset to prevent abuse."

---

## 📚 Table of Contents

1. [Offset Pagination](#1-offset-pagination)
2. [Cursor Pagination](#2-cursor-pagination)
3. [Keyset Pagination](#3-keyset-pagination)
4. [Response Formats](#4-response-formats)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Offset Pagination

```typescript
// ════════════════════════════════════════════════════════════════
// OFFSET PAGINATION IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

// Request: GET /users?page=2&per_page=20
// Or:      GET /users?offset=20&limit=20

interface OffsetPaginationParams {
  page?: number;
  perPage?: number;
  offset?: number;
  limit?: number;
}

interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    total: number;
    page: number;
    perPage: number;
    totalPages: number;
    hasNext: boolean;
    hasPrev: boolean;
  };
}

class UserRepository {
  async findAllPaginated(params: OffsetPaginationParams): Promise<PaginatedResponse<User>> {
    const page = params.page || 1;
    const perPage = Math.min(params.perPage || 20, 100); // Max 100
    const offset = params.offset ?? (page - 1) * perPage;
    const limit = params.limit ?? perPage;

    // ⚠️ WARNING: This query gets slow with large offsets
    // SELECT * FROM users ORDER BY created_at DESC LIMIT 20 OFFSET 1000000
    // Database scans 1,000,020 rows!
    
    const [users, total] = await Promise.all([
      prisma.user.findMany({
        skip: offset,
        take: limit,
        orderBy: { createdAt: 'desc' },
      }),
      prisma.user.count(), // ⚠️ Can be expensive for large tables
    ]);

    const totalPages = Math.ceil(total / perPage);

    return {
      data: users,
      pagination: {
        total,
        page,
        perPage,
        totalPages,
        hasNext: page < totalPages,
        hasPrev: page > 1,
      },
    };
  }
}

// ════════════════════════════════════════════════════════════════
// EXPRESS CONTROLLER
// ════════════════════════════════════════════════════════════════

app.get('/users', async (req, res) => {
  const page = parseInt(req.query.page as string) || 1;
  const perPage = parseInt(req.query.per_page as string) || 20;

  // Validate
  if (page < 1) return res.status(400).json({ error: 'Invalid page' });
  if (perPage < 1 || perPage > 100) {
    return res.status(400).json({ error: 'per_page must be 1-100' });
  }

  const result = await userRepository.findAllPaginated({ page, perPage });

  // Set Link headers (RFC 5988)
  const baseUrl = `${req.protocol}://${req.get('host')}${req.path}`;
  const links: string[] = [];
  
  links.push(`<${baseUrl}?page=1&per_page=${perPage}>; rel="first"`);
  links.push(`<${baseUrl}?page=${result.pagination.totalPages}&per_page=${perPage}>; rel="last"`);
  
  if (result.pagination.hasPrev) {
    links.push(`<${baseUrl}?page=${page - 1}&per_page=${perPage}>; rel="prev"`);
  }
  if (result.pagination.hasNext) {
    links.push(`<${baseUrl}?page=${page + 1}&per_page=${perPage}>; rel="next"`);
  }

  res.setHeader('Link', links.join(', '));
  res.setHeader('X-Total-Count', result.pagination.total);
  
  res.json(result);
});

// ════════════════════════════════════════════════════════════════
// RESPONSE FORMAT
// ════════════════════════════════════════════════════════════════

/*
HTTP/1.1 200 OK
Link: <https://api.example.com/users?page=1&per_page=20>; rel="first",
      <https://api.example.com/users?page=5&per_page=20>; rel="last",
      <https://api.example.com/users?page=1&per_page=20>; rel="prev",
      <https://api.example.com/users?page=3&per_page=20>; rel="next"
X-Total-Count: 100

{
  "data": [
    { "id": "usr_21", "name": "User 21", ... },
    { "id": "usr_22", "name": "User 22", ... },
    ...
  ],
  "pagination": {
    "total": 100,
    "page": 2,
    "perPage": 20,
    "totalPages": 5,
    "hasNext": true,
    "hasPrev": true
  }
}
*/
```

---

## 2. Cursor Pagination

```typescript
// ════════════════════════════════════════════════════════════════
// CURSOR PAGINATION IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

// Request: GET /users?first=20&after=eyJpZCI6MTAwLCJjcmVhdGVkQXQiOiIyMDI0LTAxLTAxIn0=

interface CursorPaginationParams {
  first?: number;   // Forward pagination
  after?: string;   // Cursor to start after
  last?: number;    // Backward pagination  
  before?: string;  // Cursor to start before
}

interface Cursor {
  id: string;
  createdAt: string;
}

interface Edge<T> {
  node: T;
  cursor: string;
}

interface PageInfo {
  hasNextPage: boolean;
  hasPreviousPage: boolean;
  startCursor?: string;
  endCursor?: string;
}

interface Connection<T> {
  edges: Edge<T>[];
  pageInfo: PageInfo;
  totalCount?: number;
}

// Cursor encoding/decoding
function encodeCursor(data: Cursor): string {
  return Buffer.from(JSON.stringify(data)).toString('base64');
}

function decodeCursor(cursor: string): Cursor {
  try {
    return JSON.parse(Buffer.from(cursor, 'base64').toString('utf-8'));
  } catch {
    throw new Error('Invalid cursor');
  }
}

class UserRepository {
  async findAllCursor(params: CursorPaginationParams): Promise<Connection<User>> {
    const limit = Math.min(params.first || params.last || 20, 100);
    const isForward = !!params.first || !params.last;
    
    let whereClause = {};
    
    if (params.after) {
      const cursor = decodeCursor(params.after);
      whereClause = {
        OR: [
          { createdAt: { lt: new Date(cursor.createdAt) } },
          {
            createdAt: { equals: new Date(cursor.createdAt) },
            id: { lt: cursor.id },
          },
        ],
      };
    }

    if (params.before) {
      const cursor = decodeCursor(params.before);
      whereClause = {
        OR: [
          { createdAt: { gt: new Date(cursor.createdAt) } },
          {
            createdAt: { equals: new Date(cursor.createdAt) },
            id: { gt: cursor.id },
          },
        ],
      };
    }

    // Fetch one extra to determine hasNextPage
    const users = await prisma.user.findMany({
      where: whereClause,
      take: limit + 1,
      orderBy: [
        { createdAt: isForward ? 'desc' : 'asc' },
        { id: isForward ? 'desc' : 'asc' },
      ],
    });

    // Check if there's a next page
    const hasMore = users.length > limit;
    if (hasMore) {
      users.pop(); // Remove the extra item
    }

    // Reverse if backward pagination
    if (!isForward) {
      users.reverse();
    }

    // Create edges with cursors
    const edges: Edge<User>[] = users.map(user => ({
      node: user,
      cursor: encodeCursor({
        id: user.id,
        createdAt: user.createdAt.toISOString(),
      }),
    }));

    return {
      edges,
      pageInfo: {
        hasNextPage: isForward ? hasMore : !!params.before,
        hasPreviousPage: isForward ? !!params.after : hasMore,
        startCursor: edges[0]?.cursor,
        endCursor: edges[edges.length - 1]?.cursor,
      },
    };
  }
}

// ════════════════════════════════════════════════════════════════
// EXPRESS CONTROLLER
// ════════════════════════════════════════════════════════════════

app.get('/users', async (req, res) => {
  const first = req.query.first ? parseInt(req.query.first as string) : undefined;
  const after = req.query.after as string | undefined;
  const last = req.query.last ? parseInt(req.query.last as string) : undefined;
  const before = req.query.before as string | undefined;

  // Validate
  if (first && last) {
    return res.status(400).json({ error: 'Cannot use both first and last' });
  }

  try {
    const result = await userRepository.findAllCursor({ first, after, last, before });
    res.json(result);
  } catch (error) {
    if (error.message === 'Invalid cursor') {
      return res.status(400).json({ error: 'Invalid cursor' });
    }
    throw error;
  }
});

// ════════════════════════════════════════════════════════════════
// RESPONSE FORMAT (Relay Connection Spec)
// ════════════════════════════════════════════════════════════════

/*
{
  "edges": [
    {
      "node": { "id": "usr_99", "name": "User 99", ... },
      "cursor": "eyJpZCI6InVzcl85OSIsImNyZWF0ZWRBdCI6IjIwMjQtMDEtMTUifQ=="
    },
    {
      "node": { "id": "usr_98", "name": "User 98", ... },
      "cursor": "eyJpZCI6InVzcl85OCIsImNyZWF0ZWRBdCI6IjIwMjQtMDEtMTQifQ=="
    }
  ],
  "pageInfo": {
    "hasNextPage": true,
    "hasPreviousPage": true,
    "startCursor": "eyJpZCI6InVzcl85OSIsImNyZWF0ZWRBdCI6IjIwMjQtMDEtMTUifQ==",
    "endCursor": "eyJpZCI6InVzcl84MCIsImNyZWF0ZWRBdCI6IjIwMjMtMTItMjgifQ=="
  }
}
*/
```

---

## 3. Keyset Pagination

```typescript
// ════════════════════════════════════════════════════════════════
// KEYSET (SEEK) PAGINATION
// ════════════════════════════════════════════════════════════════

// Request: GET /users?limit=20&after_id=100&after_created_at=2024-01-15

// This is the most performant pagination method
// Uses WHERE clause instead of OFFSET
// Database can use indexes efficiently

interface KeysetPaginationParams {
  limit?: number;
  afterId?: string;
  afterCreatedAt?: string;
  beforeId?: string;
  beforeCreatedAt?: string;
}

class UserRepository {
  async findAllKeyset(params: KeysetPaginationParams): Promise<{
    data: User[];
    hasMore: boolean;
    nextParams?: { afterId: string; afterCreatedAt: string };
    prevParams?: { beforeId: string; beforeCreatedAt: string };
  }> {
    const limit = Math.min(params.limit || 20, 100);
    
    // Build WHERE clause for seek method
    let whereClause = {};
    
    if (params.afterId && params.afterCreatedAt) {
      // Keyset condition for forward pagination
      // (created_at < X) OR (created_at = X AND id < Y)
      whereClause = {
        OR: [
          { createdAt: { lt: new Date(params.afterCreatedAt) } },
          {
            AND: [
              { createdAt: { equals: new Date(params.afterCreatedAt) } },
              { id: { lt: params.afterId } },
            ],
          },
        ],
      };
    }

    if (params.beforeId && params.beforeCreatedAt) {
      whereClause = {
        OR: [
          { createdAt: { gt: new Date(params.beforeCreatedAt) } },
          {
            AND: [
              { createdAt: { equals: new Date(params.beforeCreatedAt) } },
              { id: { gt: params.beforeId } },
            ],
          },
        ],
      };
    }

    // Fetch one extra to check for more
    const users = await prisma.user.findMany({
      where: whereClause,
      take: limit + 1,
      orderBy: [
        { createdAt: 'desc' },
        { id: 'desc' },  // Secondary sort for uniqueness
      ],
    });

    const hasMore = users.length > limit;
    if (hasMore) {
      users.pop();
    }

    const lastUser = users[users.length - 1];
    const firstUser = users[0];

    return {
      data: users,
      hasMore,
      nextParams: lastUser ? {
        afterId: lastUser.id,
        afterCreatedAt: lastUser.createdAt.toISOString(),
      } : undefined,
      prevParams: firstUser ? {
        beforeId: firstUser.id,
        beforeCreatedAt: firstUser.createdAt.toISOString(),
      } : undefined,
    };
  }
}

// ════════════════════════════════════════════════════════════════
// RAW SQL FOR MAXIMUM PERFORMANCE
// ════════════════════════════════════════════════════════════════

async function findUsersKeyset(afterId: string | null, afterDate: Date | null, limit: number) {
  // This query is O(log n) with proper index on (created_at, id)
  // Compared to OFFSET which is O(n)
  
  const sql = afterId && afterDate
    ? `
        SELECT * FROM users
        WHERE (created_at, id) < ($1, $2)
        ORDER BY created_at DESC, id DESC
        LIMIT $3
      `
    : `
        SELECT * FROM users
        ORDER BY created_at DESC, id DESC
        LIMIT $1
      `;

  const params = afterId && afterDate
    ? [afterDate, afterId, limit + 1]
    : [limit + 1];

  return prisma.$queryRawUnsafe(sql, ...params);
}

// ════════════════════════════════════════════════════════════════
// INDEX FOR KEYSET PAGINATION
// ════════════════════════════════════════════════════════════════

/*
-- Create composite index for keyset pagination
CREATE INDEX idx_users_keyset ON users (created_at DESC, id DESC);

-- For different sort orders, create appropriate indexes
CREATE INDEX idx_users_name_keyset ON users (name, id);
CREATE INDEX idx_users_updated_keyset ON users (updated_at DESC, id DESC);

-- Query plan should show "Index Scan" not "Seq Scan"
EXPLAIN ANALYZE
SELECT * FROM users
WHERE (created_at, id) < ('2024-01-15', 'usr_100')
ORDER BY created_at DESC, id DESC
LIMIT 20;
*/

// ════════════════════════════════════════════════════════════════
// EXPRESS CONTROLLER WITH KEYSET
// ════════════════════════════════════════════════════════════════

app.get('/users', async (req, res) => {
  const limit = Math.min(parseInt(req.query.limit as string) || 20, 100);
  const afterId = req.query.after_id as string | undefined;
  const afterCreatedAt = req.query.after_created_at as string | undefined;

  const result = await userRepository.findAllKeyset({
    limit,
    afterId,
    afterCreatedAt,
  });

  // Build next URL
  let nextUrl: string | null = null;
  if (result.hasMore && result.nextParams) {
    const params = new URLSearchParams({
      limit: String(limit),
      after_id: result.nextParams.afterId,
      after_created_at: result.nextParams.afterCreatedAt,
    });
    nextUrl = `${req.path}?${params}`;
  }

  res.json({
    data: result.data,
    hasMore: result.hasMore,
    nextUrl,
  });
});
```

---

## 4. Response Formats

```typescript
// ════════════════════════════════════════════════════════════════
// STANDARD RESPONSE FORMATS
// ════════════════════════════════════════════════════════════════

// Format 1: Simple envelope
interface SimplePaginatedResponse<T> {
  data: T[];
  meta: {
    total?: number;
    page?: number;
    perPage: number;
    hasMore: boolean;
  };
  links: {
    self: string;
    first?: string;
    last?: string;
    prev?: string;
    next?: string;
  };
}

// Format 2: Relay Connection (GraphQL standard)
interface RelayConnection<T> {
  edges: Array<{
    node: T;
    cursor: string;
  }>;
  pageInfo: {
    hasNextPage: boolean;
    hasPreviousPage: boolean;
    startCursor: string | null;
    endCursor: string | null;
  };
  totalCount?: number;
}

// Format 3: JSON:API style
interface JsonApiPaginatedResponse<T> {
  data: T[];
  meta: {
    totalItems: number;
    itemCount: number;
    itemsPerPage: number;
    totalPages: number;
    currentPage: number;
  };
  links: {
    first: string;
    previous: string | null;
    next: string | null;
    last: string;
  };
}

// ════════════════════════════════════════════════════════════════
// LINK HEADERS (RFC 5988)
// ════════════════════════════════════════════════════════════════

function setLinkHeaders(res: Response, links: {
  first?: string;
  last?: string;
  prev?: string;
  next?: string;
}) {
  const parts: string[] = [];
  
  if (links.first) parts.push(`<${links.first}>; rel="first"`);
  if (links.prev) parts.push(`<${links.prev}>; rel="prev"`);
  if (links.next) parts.push(`<${links.next}>; rel="next"`);
  if (links.last) parts.push(`<${links.last}>; rel="last"`);
  
  if (parts.length > 0) {
    res.setHeader('Link', parts.join(', '));
  }
}

// ════════════════════════════════════════════════════════════════
// GRAPHQL PAGINATION (Relay Spec)
// ════════════════════════════════════════════════════════════════

// GraphQL Schema
const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    email: String!
  }

  type UserEdge {
    node: User!
    cursor: String!
  }

  type UserConnection {
    edges: [UserEdge!]!
    pageInfo: PageInfo!
    totalCount: Int
  }

  type PageInfo {
    hasNextPage: Boolean!
    hasPreviousPage: Boolean!
    startCursor: String
    endCursor: String
  }

  type Query {
    users(
      first: Int
      after: String
      last: Int
      before: String
    ): UserConnection!
  }
`;

// Resolver
const resolvers = {
  Query: {
    users: async (_, args, context) => {
      return context.dataSources.users.findAllCursor({
        first: args.first,
        after: args.after,
        last: args.last,
        before: args.before,
      });
    },
  },
};

// Client Query
const GET_USERS = gql`
  query GetUsers($first: Int!, $after: String) {
    users(first: $first, after: $after) {
      edges {
        node {
          id
          name
          email
        }
        cursor
      }
      pageInfo {
        hasNextPage
        endCursor
      }
      totalCount
    }
  }
`;
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# PAGINATION BEST PRACTICES
# ════════════════════════════════════════════════════════════════

choosing_strategy:
  use_cursor_when:
    - Large datasets
    - Real-time/frequently changing data
    - Infinite scroll UX
    - Feed/timeline patterns
    
  use_offset_when:
    - Small, stable datasets
    - Need to jump to specific page
    - Admin panels
    - Export/download features

implementation:
  page_size:
    - Default: 20-50 items
    - Max: 100 items (prevent abuse)
    - Allow client to specify within limits
    
  sorting:
    - Always have consistent sort order
    - Use unique secondary sort (e.g., id)
    - Same request = same order
    
  cursors:
    - Make cursors opaque (base64)
    - Include all sort fields
    - Set expiration if needed
    - Validate on decode

performance:
  total_count:
    - Skip for large datasets (COUNT is expensive)
    - Use estimate: pg_class.reltuples
    - Or: only return if total < threshold
    
  indexing:
    - Create indexes matching sort order
    - Composite index: (sort_field, id)
    - Verify with EXPLAIN ANALYZE
    
  limits:
    - Max offset for offset pagination
    - Max page size
    - Rate limit pagination requests

response:
  include:
    - hasNextPage / hasPreviousPage
    - Navigation links (prev/next)
    - Page size used
    
  optional:
    - Total count (if not expensive)
    - First/last links
    
  headers:
    - Link header for navigation
    - X-Total-Count if providing total
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# PAGINATION PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Large offsets
# Bad
SELECT * FROM users LIMIT 20 OFFSET 1000000
# Database scans 1,000,020 rows!

# Good
# Use keyset/cursor pagination for large datasets
SELECT * FROM users WHERE id > 'last_id' LIMIT 20

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Non-unique sort order
# Bad
ORDER BY created_at DESC
# Ties can result in inconsistent pagination

# Good
ORDER BY created_at DESC, id DESC
# Secondary sort ensures uniqueness

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Always computing total count
# Bad
const [items, total] = await Promise.all([
  findItems({ limit }),
  countAll(),  # Expensive for millions of rows
]);

# Good
# Skip total or use estimate
# Or: only provide if total < 10000

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Exposing cursor internals
# Bad
cursor: { "id": 100, "createdAt": "2024-01-01" }
# Client might try to construct cursors

# Good
cursor: "eyJpZCI6MTAwLCJjcmVhdGVkQXQiOiIyMDI0LTAxLTAxIn0="
# Opaque, client can't manipulate

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: No max page size
# Bad
GET /users?per_page=100000
# Returns entire database!

# Good
const perPage = Math.min(requestedPerPage, 100);
# Always enforce maximum

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Inconsistent results during modification
# Bad - Using offset with changing data
# Page 2 might show item from page 1 after insert

# Good
# Use cursor pagination for feeds
# Or accept inconsistency for admin panels
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is pagination and why is it needed?"**
> "Pagination divides large datasets into smaller chunks. Needed for: performance (don't transfer millions of records), memory (server and client), UX (faster loading, manageable lists). Without pagination, fetching all users would crash browser and slow server."

**Q: "What's the difference between offset and cursor pagination?"**
> "Offset: `LIMIT 20 OFFSET 40` - simple but slow for large offsets, inconsistent with changing data. Cursor: `WHERE id > last_id` - uses index, consistent results, O(1) lookup. Offset for page jumping, cursor for infinite scroll/feeds."

**Q: "Why does offset pagination have performance issues?"**
> "Database must scan and discard all skipped rows. `OFFSET 1000000` scans million rows. Keyset uses index to jump directly to position - constant time. Always use keyset for large datasets or deep pagination."

### Intermediate Questions

**Q: "What is the Relay Connection specification?"**
> "GraphQL standard for pagination. Response has edges (array of {node, cursor}) and pageInfo (hasNextPage, hasPreviousPage, startCursor, endCursor). Supports forward (first/after) and backward (last/before) pagination. Cursors are opaque strings."

**Q: "How do you handle pagination with frequently changing data?"**
> "Cursor pagination. Cursor encodes position marker (e.g., last ID + timestamp). 'Items after X' always consistent regardless of insertions. Offset would show duplicates or skip items. Also consider: optimistic updates on client, eventual consistency for feeds."

**Q: "Should you always return total count?"**
> "No. COUNT(*) is expensive for large tables - full table scan without good index. Options: 1) Skip for large datasets, just return hasNextPage. 2) Use estimate (pg_class.reltuples). 3) Only provide if total < threshold. 4) Cache count, update periodically."

### Advanced Questions

**Q: "How would you implement pagination with multiple sort fields?"**
> "Keyset condition must include all sort fields. For ORDER BY date DESC, name ASC, id: WHERE (date, name, id) < (last_date, last_name, last_id). Composite index matching sort order. Cursor contains all field values. Gets complex with NULLs - consider NULLS FIRST/LAST."

**Q: "How do you handle pagination in distributed systems?"**
> "Challenges: data across shards, consistent cursors. Options: 1) Global sequence/UUID for cursor. 2) Fan-out to shards, merge results. 3) Query router maintains cursor state. 4) Cursor contains shard info. Consider: eventual consistency, total count aggregation."

**Q: "How do you prevent abuse of pagination endpoints?"**
> "1) Max page size (100 items). 2) Max offset (prevent offset=999999999). 3) Rate limiting per client. 4) Cursor expiration (24h). 5) Cost-based query limits. 6) Authentication required for expensive operations. Monitor for unusual patterns."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                 PAGINATION CHECKLIST                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  STRATEGY SELECTION:                                            │
│  □ Small/stable data → Offset OK                               │
│  □ Large/changing data → Cursor/Keyset                         │
│  □ Infinite scroll → Cursor                                    │
│  □ Page jumping needed → Offset                                │
│                                                                 │
│  IMPLEMENTATION:                                                │
│  □ Unique secondary sort (id)                                  │
│  □ Proper indexes                                              │
│  □ Max page size limit                                         │
│  □ Opaque cursors                                              │
│                                                                 │
│  RESPONSE:                                                      │
│  □ hasNextPage/hasPreviousPage                                 │
│  □ Navigation links                                            │
│  □ Total count (optional)                                      │
│  □ Link headers                                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

STRATEGY COMPARISON:
┌────────────────────────────────────────────────────────────────┐
│ Offset:  Simple, page jump, slow for large offsets            │
│ Cursor:  Consistent, efficient, no page jump                  │
│ Keyset:  Best performance, uses index, needs unique sort      │
└────────────────────────────────────────────────────────────────┘

QUERY PATTERNS:
┌────────────────────────────────────────────────────────────────┐
│ Offset:  LIMIT 20 OFFSET 40                                   │
│ Keyset:  WHERE (date, id) < (last_date, last_id) LIMIT 20    │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

