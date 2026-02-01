# Chapter 11: Performance Engineering

> "Premature optimization is the root of all evil, but mature optimization is the root of all speed."

---

## 🎯 Performance Mindset

```
1. Measure before optimizing
2. Optimize the bottleneck
3. Know when to stop
4. Performance is a feature
```

---

## 📊 Latency Breakdown

```
Typical web request breakdown:

DNS Lookup         │████                          │ 20-50ms
TCP Connection     │████████                      │ 30-100ms
TLS Handshake      │████████████                  │ 50-200ms
Server Processing  │████████████████████          │ 100-500ms
Content Download   │████████████████              │ 50-300ms
                   └──────────────────────────────┘
                   Total: 250ms - 1.5s

Where to optimize:
- CDN: Reduces DNS, TCP, TLS latency
- Caching: Reduces server processing
- Compression: Reduces download time
- Code optimization: Reduces processing time
```

---

## 🔍 Profiling

### CPU Profiling

```javascript
// Node.js CPU profiling
const { Session } = require('inspector');
const fs = require('fs');

// Start profiling
const session = new Session();
session.connect();
session.post('Profiler.enable');
session.post('Profiler.start');

// ... run your code ...

// Stop and save
session.post('Profiler.stop', (err, { profile }) => {
  fs.writeFileSync('profile.cpuprofile', JSON.stringify(profile));
  // Open in Chrome DevTools
});
```

### Memory Profiling

```javascript
// Check memory usage
const used = process.memoryUsage();
console.log({
  rss: `${Math.round(used.rss / 1024 / 1024)} MB`,      // Resident Set Size
  heapTotal: `${Math.round(used.heapTotal / 1024 / 1024)} MB`,
  heapUsed: `${Math.round(used.heapUsed / 1024 / 1024)} MB`,
  external: `${Math.round(used.external / 1024 / 1024)} MB`
});

// Find memory leaks
// Take heap snapshots at intervals
// Compare snapshots to find growing objects
```

### Database Query Profiling

```sql
-- PostgreSQL: Analyze slow queries
EXPLAIN ANALYZE 
SELECT * FROM users 
WHERE email = 'test@test.com';

-- Output:
-- Seq Scan on users  (cost=0.00..35.50 rows=1 width=100) 
--   (actual time=5.123..5.125 rows=1 loops=1)
-- Planning Time: 0.1 ms
-- Execution Time: 5.2 ms

-- With index:
-- Index Scan using idx_email on users  (cost=0.42..8.44 rows=1)
--   (actual time=0.045..0.046 rows=1 loops=1)
-- Execution Time: 0.1 ms  ← 50x faster!
```

---

## ⚡ Code Optimization

### Algorithm Optimization

```javascript
// BAD: O(n²)
function findDuplicates(arr) {
  const duplicates = [];
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] === arr[j]) {
        duplicates.push(arr[i]);
      }
    }
  }
  return duplicates;
}

// GOOD: O(n)
function findDuplicates(arr) {
  const seen = new Set();
  const duplicates = new Set();
  
  for (const item of arr) {
    if (seen.has(item)) {
      duplicates.add(item);
    }
    seen.add(item);
  }
  
  return [...duplicates];
}

// 1000 items: 500,000 vs 1,000 operations
```

### Memory Optimization

```javascript
// BAD: Creates many intermediate arrays
const result = data
  .filter(x => x.active)
  .map(x => x.value)
  .reduce((a, b) => a + b, 0);

// GOOD: Single pass
const result = data.reduce((sum, x) => {
  return x.active ? sum + x.value : sum;
}, 0);

// BAD: Holding large data in memory
const allUsers = await db.users.findAll(); // 1M users in memory!

// GOOD: Stream processing
const stream = db.users.createStream();
for await (const user of stream) {
  await processUser(user);
}
```

### Async Optimization

```javascript
// BAD: Sequential
async function fetchAll() {
  const user = await fetchUser();      // 100ms
  const orders = await fetchOrders();  // 100ms
  const reviews = await fetchReviews(); // 100ms
  // Total: 300ms
}

// GOOD: Parallel
async function fetchAll() {
  const [user, orders, reviews] = await Promise.all([
    fetchUser(),   // 100ms
    fetchOrders(), // 100ms  } All at once
    fetchReviews() // 100ms  }
  ]);
  // Total: ~100ms
}

// GOOD: Parallel with error handling
async function fetchAll() {
  const results = await Promise.allSettled([
    fetchUser(),
    fetchOrders(),
    fetchReviews()
  ]);
  
  return results.map(r => 
    r.status === 'fulfilled' ? r.value : null
  );
}
```

---

## 🗄️ Database Optimization

### Indexing Strategy

```sql
-- Identify slow queries
SELECT query, calls, mean_time, total_time
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;

-- Check missing indexes
SELECT relname, seq_scan, seq_tup_read, 
       idx_scan, idx_tup_fetch
FROM pg_stat_user_tables
WHERE seq_scan > 0
ORDER BY seq_tup_read DESC;

-- Create strategic indexes
-- For WHERE clauses
CREATE INDEX idx_users_email ON users(email);

-- For range queries
CREATE INDEX idx_orders_date ON orders(created_at DESC);

-- Composite for multi-column queries
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- Partial indexes (save space)
CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;
```

### Query Optimization

```sql
-- BAD: SELECT *
SELECT * FROM orders WHERE user_id = 123;

-- GOOD: Select only needed columns
SELECT id, status, total FROM orders WHERE user_id = 123;

-- BAD: N+1 queries
-- App code: for each user, fetch orders separately

-- GOOD: JOIN or subquery
SELECT u.*, o.* 
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
WHERE u.id IN (1, 2, 3);

-- BAD: Large OFFSET
SELECT * FROM products LIMIT 10 OFFSET 100000;

-- GOOD: Keyset pagination
SELECT * FROM products 
WHERE id > 100000 
ORDER BY id 
LIMIT 10;
```

### Connection Pooling

```javascript
// Without pooling: New connection per request
// 1000 requests = 1000 connections = CRASH

// With pooling
const pool = new Pool({
  host: 'localhost',
  database: 'mydb',
  max: 20,           // Maximum connections
  min: 5,            // Minimum connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Connection reused from pool
const result = await pool.query('SELECT * FROM users');
```

---

## 🌐 Frontend Performance

### Core Web Vitals

```
LCP (Largest Contentful Paint): < 2.5s
  - How long until main content loads
  
FID (First Input Delay): < 100ms
  - How long until page is interactive
  
CLS (Cumulative Layout Shift): < 0.1
  - Visual stability (no jumping content)
```

### Loading Optimization

```javascript
// Code splitting (lazy loading)
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// Image optimization
<Image
  src="/photo.jpg"
  loading="lazy"
  width={800}
  height={600}
  placeholder="blur"
/>

// Preloading critical resources
<link rel="preload" href="/font.woff2" as="font" crossorigin>
<link rel="preconnect" href="https://api.example.com">

// Resource hints
<link rel="dns-prefetch" href="https://cdn.example.com">
<link rel="prefetch" href="/next-page.js">
```

### Bundle Optimization

```javascript
// webpack-bundle-analyzer
// Visualize bundle size

// Tree shaking
import { debounce } from 'lodash-es';  // Not: import _ from 'lodash'

// Dynamic imports
const moment = await import('moment');

// Replace heavy libraries
// moment.js (300KB) → date-fns (30KB)
// lodash (70KB) → native methods or lodash-es
```

---

## 📈 Load Testing

### Tools

```bash
# k6 (modern, scriptable)
k6 run script.js

# Apache Bench (simple)
ab -n 1000 -c 100 http://localhost:3000/api/users

# wrk (high performance)
wrk -t12 -c400 -d30s http://localhost:3000/api/users

# Locust (Python, distributed)
locust -f locustfile.py
```

### k6 Script Example

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '1m', target: 100 },   // Ramp up
    { duration: '3m', target: 100 },   // Stay
    { duration: '1m', target: 200 },   // Spike
    { duration: '2m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% under 500ms
    http_req_failed: ['rate<0.01'],    // <1% errors
  },
};

export default function () {
  const res = http.get('http://localhost:3000/api/users');
  
  check(res, {
    'status is 200': (r) => r.status === 200,
    'duration < 500ms': (r) => r.timings.duration < 500,
  });
  
  sleep(1);
}
```

### Understanding Results

```
k6 output:
     data_received........: 1.5 GB  50 MB/s
     data_sent............: 100 MB  3.3 MB/s
     http_req_duration....: avg=120ms  p(95)=350ms  p(99)=500ms
     http_req_failed......: 0.5%
     http_reqs............: 100000  3333/s

Key metrics:
- Throughput: 3333 requests/second
- Latency p95: 350ms (95% of requests under 350ms)
- Error rate: 0.5%
```

---

## 🎯 Performance Checklist

```
Backend:
□ Database queries optimized (indexes, no N+1)
□ Caching implemented (Redis, CDN)
□ Connection pooling configured
□ Async operations parallelized
□ Background jobs for heavy tasks
□ Response compression enabled

Frontend:
□ Bundle size minimized (<200KB initial)
□ Images optimized and lazy loaded
□ Code splitting implemented
□ Critical CSS inlined
□ Service worker caching
□ Core Web Vitals passing

Infrastructure:
□ CDN configured
□ HTTP/2 or HTTP/3 enabled
□ Compression (gzip/brotli)
□ Keep-alive connections
□ Load balancing configured
□ Auto-scaling enabled
```

---

## 📖 Further Reading

- "High Performance Browser Networking"
- "Web Performance in Action"
- Google Web Fundamentals (Performance)
- k6 Documentation

---

**Next:** [Chapter 12: Cloud Architecture →](./12-cloud-architecture.md)


