# ⚡ Performance Testing - Complete Guide

> A comprehensive guide to performance testing - k6, JMeter, load testing, stress testing, and identifying performance bottlenecks.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Performance testing validates that your application meets speed, scalability, and stability requirements under various load conditions - measuring response times, throughput, resource utilization, and identifying bottlenecks before they impact production users."

### The 7 Key Concepts (Remember These!)
```
1. LOAD TESTING    → Test with expected concurrent users
2. STRESS TESTING  → Push beyond limits to find breaking point
3. SPIKE TESTING   → Sudden traffic surges
4. SOAK TESTING    → Extended duration for memory leaks
5. THROUGHPUT      → Requests per second (RPS)
6. LATENCY         → Response time (p50, p95, p99)
7. SATURATION      → Resource utilization (CPU, memory)
```

### Performance Test Types
```
┌─────────────────────────────────────────────────────────────────┐
│                  PERFORMANCE TEST TYPES                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LOAD TEST                                                     │
│  ─────────                                                      │
│  Users:  Expected production load                              │
│  Goal:   Validate performance under normal conditions          │
│  Example: 1000 concurrent users for 30 minutes                 │
│                                                                 │
│  STRESS TEST                                                    │
│  ───────────                                                    │
│  Users:  Beyond expected capacity                              │
│  Goal:   Find breaking point, observe degradation              │
│  Example: Ramp from 1000 → 5000 users, observe failures        │
│                                                                 │
│  SPIKE TEST                                                     │
│  ──────────                                                     │
│  Users:  Sudden large increase                                 │
│  Goal:   Test auto-scaling, sudden traffic handling            │
│  Example: 100 → 5000 → 100 users in minutes                    │
│                                                                 │
│  SOAK TEST (Endurance)                                         │
│  ─────────────────────                                          │
│  Users:  Moderate sustained load                               │
│  Goal:   Find memory leaks, resource exhaustion                │
│  Example: 500 users for 24 hours                               │
│                                                                 │
│  BREAKPOINT TEST                                                │
│  ────────────────                                               │
│  Users:  Continuously increasing                               │
│  Goal:   Find maximum capacity                                 │
│  Example: Add 100 users/minute until failure                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Metrics
```
┌─────────────────────────────────────────────────────────────────┐
│                     KEY METRICS                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LATENCY (Response Time)                                       │
│  • p50 (median): 50% of requests faster                        │
│  • p95: 95% of requests faster                                 │
│  • p99: 99% of requests faster (tail latency)                  │
│  • Target: p99 < 500ms for APIs                                │
│                                                                 │
│  THROUGHPUT                                                     │
│  • Requests per second (RPS)                                   │
│  • Transactions per second (TPS)                               │
│  • Target: Varies by application                               │
│                                                                 │
│  ERROR RATE                                                     │
│  • Percentage of failed requests                               │
│  • Target: < 1% under load                                     │
│                                                                 │
│  RESOURCE UTILIZATION                                          │
│  • CPU: < 80% sustained                                        │
│  • Memory: Stable, no growth                                   │
│  • Network: Below bandwidth limits                             │
│  • Disk I/O: Not saturated                                     │
│                                                                 │
│  APDEX (Application Performance Index)                         │
│  • Satisfied: < T (e.g., 500ms)                                │
│  • Tolerating: T to 4T                                         │
│  • Frustrated: > 4T                                            │
│  • Score: (Satisfied + Tolerating/2) / Total                   │
│  • Target: > 0.9                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"p99 latency"** | "We monitor p99 latency to catch tail latency issues" |
| **"Throughput"** | "We sustained 5000 RPS with p99 under 200ms" |
| **"Breaking point"** | "Stress tests found breaking point at 8000 users" |
| **"Soak test"** | "24-hour soak test revealed memory leak" |
| **"Virtual users"** | "We simulate 1000 virtual users with realistic think time" |
| **"Ramp up"** | "Gradual ramp up prevents thundering herd" |

### Key Numbers to Remember
| Metric | Good Target | Notes |
|--------|-------------|-------|
| p50 latency | **< 100ms** | Median response |
| p95 latency | **< 300ms** | Most users |
| p99 latency | **< 500ms** | Tail latency |
| Error rate | **< 1%** | Under load |
| CPU | **< 80%** | Leave headroom |
| Apdex | **> 0.9** | User satisfaction |

### The "Wow" Statement (Memorize This!)
> "We run comprehensive performance tests in CI using k6. Load tests simulate expected traffic - 2000 concurrent users making realistic API calls with think time. Stress tests ramp to 10,000 users to find breaking points. We measure p50/p95/p99 latency, throughput, and error rate. SLOs define pass/fail: p99 < 500ms, error rate < 1%. Weekly soak tests (24 hours) catch memory leaks. Tests run against staging with production-like data. Results feed into Grafana dashboards. We identified database connection pool exhaustion at 3000 users - fixed by increasing pool size. Performance budgets prevent regressions - PRs fail if latency increases by >10%."

---

## 📚 Table of Contents

1. [k6 Load Testing](#1-k6-load-testing)
2. [Test Scenarios](#2-test-scenarios)
3. [Metrics & Thresholds](#3-metrics--thresholds)
4. [JMeter](#4-jmeter)
5. [CI Integration](#5-ci-integration)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. k6 Load Testing

```javascript
// ════════════════════════════════════════════════════════════════
// K6 LOAD TEST
// ════════════════════════════════════════════════════════════════

import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend, Counter } from 'k6/metrics';

// Custom metrics
const errorRate = new Rate('errors');
const apiLatency = new Trend('api_latency');
const ordersCreated = new Counter('orders_created');

// Test configuration
export const options = {
  // Stages define ramp-up pattern
  stages: [
    { duration: '2m', target: 100 },   // Ramp up to 100 users
    { duration: '5m', target: 100 },   // Stay at 100 users
    { duration: '2m', target: 200 },   // Ramp up to 200 users
    { duration: '5m', target: 200 },   // Stay at 200 users
    { duration: '2m', target: 0 },     // Ramp down
  ],
  
  // Pass/fail thresholds
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'], // ms
    http_req_failed: ['rate<0.01'],                  // <1% errors
    errors: ['rate<0.01'],
  },
};

// Setup - runs once before test
export function setup() {
  // Login and get auth token
  const loginRes = http.post(`${__ENV.BASE_URL}/api/auth/login`, JSON.stringify({
    email: 'loadtest@example.com',
    password: 'password123',
  }), {
    headers: { 'Content-Type': 'application/json' },
  });

  const token = loginRes.json('token');
  return { token };
}

// Main test function - runs repeatedly
export default function(data) {
  const baseUrl = __ENV.BASE_URL || 'http://localhost:3000';
  const headers = {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${data.token}`,
  };

  // GET request - list products
  const productsRes = http.get(`${baseUrl}/api/products`, { headers });
  check(productsRes, {
    'products status 200': (r) => r.status === 200,
    'products has data': (r) => r.json('products').length > 0,
  });
  apiLatency.add(productsRes.timings.duration);
  errorRate.add(productsRes.status !== 200);

  sleep(1); // Think time

  // POST request - create order
  const orderRes = http.post(`${baseUrl}/api/orders`, JSON.stringify({
    items: [{ productId: 'prod-1', quantity: 1 }],
  }), { headers });
  
  const orderSuccess = check(orderRes, {
    'order status 201': (r) => r.status === 201,
    'order has id': (r) => r.json('id') !== undefined,
  });
  
  if (orderSuccess) {
    ordersCreated.add(1);
  }
  errorRate.add(orderRes.status !== 201);

  sleep(Math.random() * 3 + 1); // Random think time 1-4 seconds
}

// Teardown - runs once after test
export function teardown(data) {
  // Cleanup if needed
  console.log('Test completed');
}

// ════════════════════════════════════════════════════════════════
// RUN COMMAND
// ════════════════════════════════════════════════════════════════

// k6 run --env BASE_URL=https://staging.example.com load-test.js
// k6 run --out influxdb=http://localhost:8086/k6 load-test.js  # Send to InfluxDB
// k6 run --out json=results.json load-test.js  # Save results
```

---

## 2. Test Scenarios

```javascript
// ════════════════════════════════════════════════════════════════
// STRESS TEST - Find Breaking Point
// ════════════════════════════════════════════════════════════════

export const options = {
  stages: [
    { duration: '2m', target: 500 },
    { duration: '5m', target: 500 },
    { duration: '2m', target: 1000 },
    { duration: '5m', target: 1000 },
    { duration: '2m', target: 2000 },
    { duration: '5m', target: 2000 },
    { duration: '2m', target: 3000 },  // Keep pushing
    { duration: '5m', target: 3000 },
    { duration: '5m', target: 0 },
  ],
  thresholds: {
    http_req_duration: ['p(99)<2000'],  // More lenient
    http_req_failed: ['rate<0.1'],       // 10% errors acceptable in stress
  },
};

// ════════════════════════════════════════════════════════════════
// SPIKE TEST - Sudden Traffic Surge
// ════════════════════════════════════════════════════════════════

export const options = {
  stages: [
    { duration: '1m', target: 100 },   // Normal load
    { duration: '30s', target: 2000 }, // Spike!
    { duration: '3m', target: 2000 },  // Stay at spike
    { duration: '30s', target: 100 },  // Drop back
    { duration: '2m', target: 100 },   // Recovery
    { duration: '30s', target: 0 },
  ],
};

// ════════════════════════════════════════════════════════════════
// SOAK TEST - Extended Duration
// ════════════════════════════════════════════════════════════════

export const options = {
  stages: [
    { duration: '5m', target: 500 },   // Ramp up
    { duration: '24h', target: 500 },  // Hold for 24 hours
    { duration: '5m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(99)<1000'],
    http_req_failed: ['rate<0.01'],
  },
};

// Monitor for memory leaks, connection exhaustion, etc.

// ════════════════════════════════════════════════════════════════
// SCENARIOS - Multiple User Types
// ════════════════════════════════════════════════════════════════

export const options = {
  scenarios: {
    // Browsing users (read-heavy)
    browsing: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { duration: '2m', target: 500 },
        { duration: '10m', target: 500 },
        { duration: '2m', target: 0 },
      ],
      exec: 'browsingScenario',
    },
    
    // Purchasing users (write-heavy)
    purchasing: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { duration: '2m', target: 100 },
        { duration: '10m', target: 100 },
        { duration: '2m', target: 0 },
      ],
      exec: 'purchasingScenario',
    },
    
    // API integrations (constant rate)
    api_partners: {
      executor: 'constant-arrival-rate',
      rate: 50,      // 50 RPS
      timeUnit: '1s',
      duration: '10m',
      preAllocatedVUs: 100,
      exec: 'apiScenario',
    },
  },
};

export function browsingScenario() {
  http.get(`${BASE_URL}/api/products`);
  sleep(Math.random() * 5 + 2); // Browse slowly
  http.get(`${BASE_URL}/api/products/featured`);
  sleep(Math.random() * 3 + 1);
}

export function purchasingScenario() {
  http.get(`${BASE_URL}/api/cart`);
  http.post(`${BASE_URL}/api/cart/items`, JSON.stringify({
    productId: 'prod-1',
    quantity: 1,
  }));
  sleep(1);
  http.post(`${BASE_URL}/api/orders`);
  sleep(5);
}

export function apiScenario() {
  http.get(`${BASE_URL}/api/v1/inventory`, {
    headers: { 'X-API-Key': __ENV.API_KEY },
  });
}
```

---

## 3. Metrics & Thresholds

```javascript
// ════════════════════════════════════════════════════════════════
// COMPREHENSIVE THRESHOLDS
// ════════════════════════════════════════════════════════════════

import http from 'k6/http';
import { Trend, Rate, Counter, Gauge } from 'k6/metrics';

// Custom metrics
const loginDuration = new Trend('login_duration');
const checkoutDuration = new Trend('checkout_duration');
const errorsByEndpoint = new Rate('errors_by_endpoint');
const activeUsers = new Gauge('active_users');
const ordersCompleted = new Counter('orders_completed');

export const options = {
  thresholds: {
    // HTTP metrics (built-in)
    http_req_duration: [
      'p(50)<200',   // Median under 200ms
      'p(95)<500',   // 95th percentile under 500ms
      'p(99)<1000',  // 99th percentile under 1s
      'max<5000',    // Max under 5s
    ],
    http_req_failed: ['rate<0.01'], // <1% error rate
    
    // Custom metrics
    login_duration: ['p(95)<1000'],     // Login under 1s
    checkout_duration: ['p(95)<3000'],  // Checkout under 3s
    
    // Per-endpoint thresholds using tags
    'http_req_duration{endpoint:products}': ['p(95)<300'],
    'http_req_duration{endpoint:checkout}': ['p(95)<2000'],
    
    // Checks (assertions)
    checks: ['rate>0.99'], // 99% of checks pass
  },
};

export default function() {
  // Tag requests for per-endpoint metrics
  const productsRes = http.get(`${BASE_URL}/api/products`, {
    tags: { endpoint: 'products' },
  });
  
  const loginStart = Date.now();
  const loginRes = http.post(`${BASE_URL}/api/auth/login`, ...);
  loginDuration.add(Date.now() - loginStart);
  
  const checkoutStart = Date.now();
  const checkoutRes = http.post(`${BASE_URL}/api/checkout`, ...);
  checkoutDuration.add(Date.now() - checkoutStart);
  
  if (checkoutRes.status === 201) {
    ordersCompleted.add(1);
  }
  
  activeUsers.add(__VU); // Track concurrent users
}

// ════════════════════════════════════════════════════════════════
// RESULT ANALYSIS
// ════════════════════════════════════════════════════════════════

/*
Results example:

     ✓ http_req_duration..............: avg=234ms  min=45ms   med=189ms  max=2.3s   p(90)=456ms  p(95)=678ms
     ✓ http_req_failed................: 0.23%  ✓ 23        ✗ 9977
     ✓ login_duration.................: avg=567ms  p(95)=890ms
     ✓ checkout_duration..............: avg=1.2s   p(95)=2.3s
       orders_completed...............: 4521
       
     checks...........................: 99.2%  ✓ 19840     ✗ 160

     thresholds:
       ✓ http_req_duration p(95)<500ms
       ✓ http_req_failed rate<0.01
       ✓ login_duration p(95)<1000ms
       ✗ checkout_duration p(95)<2000ms  <-- FAILED!
*/
```

---

## 4. JMeter

```xml
<!-- ════════════════════════════════════════════════════════════════ -->
<!-- JMETER TEST PLAN -->
<!-- ════════════════════════════════════════════════════════════════ -->

<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2">
  <hashTree>
    <TestPlan guiclass="TestPlanGui" testclass="TestPlan" testname="API Load Test">
      <elementProp name="TestPlan.user_defined_variables" elementType="Arguments">
        <collectionProp name="Arguments.arguments">
          <elementProp name="BASE_URL" elementType="Argument">
            <stringProp name="Argument.name">BASE_URL</stringProp>
            <stringProp name="Argument.value">https://api.example.com</stringProp>
          </elementProp>
        </collectionProp>
      </elementProp>
    </TestPlan>
    <hashTree>
      <!-- Thread Group -->
      <ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup" testname="Users">
        <intProp name="ThreadGroup.num_threads">100</intProp>
        <intProp name="ThreadGroup.ramp_time">60</intProp>
        <boolProp name="ThreadGroup.scheduler">true</boolProp>
        <stringProp name="ThreadGroup.duration">600</stringProp>
      </ThreadGroup>
      <hashTree>
        <!-- HTTP Request -->
        <HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="Get Products">
          <stringProp name="HTTPSampler.domain">${BASE_URL}</stringProp>
          <stringProp name="HTTPSampler.path">/api/products</stringProp>
          <stringProp name="HTTPSampler.method">GET</stringProp>
        </HTTPSamplerProxy>
        
        <!-- Assertions -->
        <ResponseAssertion guiclass="AssertionGui" testclass="ResponseAssertion" testname="Response Code">
          <stringProp name="Assertion.test_field">Assertion.response_code</stringProp>
          <stringProp name="Assertion.test_strings">200</stringProp>
        </ResponseAssertion>
        
        <!-- Timer (Think Time) -->
        <UniformRandomTimer guiclass="UniformRandomTimerGui" testclass="UniformRandomTimer">
          <stringProp name="RandomTimer.range">2000</stringProp>
          <stringProp name="ConstantTimer.delay">1000</stringProp>
        </UniformRandomTimer>
      </hashTree>
    </hashTree>
  </hashTree>
</jmeterTestPlan>
```

```bash
# Run JMeter from command line
jmeter -n -t test-plan.jmx -l results.jtl -e -o report/

# With properties
jmeter -n -t test-plan.jmx \
  -JBASE_URL=https://staging.example.com \
  -JTHREADS=200 \
  -JDURATION=1800 \
  -l results.jtl
```

---

## 5. CI Integration

```yaml
# ════════════════════════════════════════════════════════════════
# GITHUB ACTIONS - K6 PERFORMANCE TESTS
# ════════════════════════════════════════════════════════════════

name: Performance Tests

on:
  schedule:
    - cron: '0 2 * * *'  # Nightly at 2 AM
  workflow_dispatch:
    inputs:
      test_type:
        description: 'Test type'
        required: true
        default: 'load'
        type: choice
        options:
          - load
          - stress
          - spike

jobs:
  performance-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup k6
        uses: grafana/setup-k6-action@v1

      - name: Run k6 load test
        run: |
          k6 run \
            --env BASE_URL=${{ secrets.STAGING_URL }} \
            --out json=results.json \
            tests/performance/${{ github.event.inputs.test_type || 'load' }}.js

      - name: Upload results
        uses: actions/upload-artifact@v3
        with:
          name: k6-results
          path: results.json

      - name: Post results to Slack
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Performance test completed: ${{ job.status }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Performance Test Results*\nStatus: ${{ job.status }}\nTest: ${{ github.event.inputs.test_type || 'load' }}"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

# ════════════════════════════════════════════════════════════════
# PERFORMANCE BUDGET IN PR
# ════════════════════════════════════════════════════════════════

name: Performance Budget

on:
  pull_request:
    branches: [main]

jobs:
  perf-budget:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Start application
        run: |
          docker-compose up -d
          sleep 30  # Wait for startup

      - name: Run baseline test
        run: |
          k6 run --out json=pr-results.json tests/performance/baseline.js

      - name: Compare with main
        run: |
          # Fetch main branch results
          aws s3 cp s3://perf-results/main-results.json main-results.json
          
          # Compare
          node scripts/compare-perf.js main-results.json pr-results.json

      - name: Fail if budget exceeded
        run: |
          # Check if p95 latency increased >10%
          node scripts/check-budget.js pr-results.json
```

```javascript
// scripts/check-budget.js
const results = require(process.argv[2]);

const p95 = results.metrics.http_req_duration.values['p(95)'];
const errorRate = results.metrics.http_req_failed.values.rate;

const budgets = {
  p95_latency: 500,  // ms
  error_rate: 0.01,  // 1%
};

let failed = false;

if (p95 > budgets.p95_latency) {
  console.error(`❌ p95 latency ${p95}ms exceeds budget of ${budgets.p95_latency}ms`);
  failed = true;
}

if (errorRate > budgets.error_rate) {
  console.error(`❌ Error rate ${errorRate * 100}% exceeds budget of ${budgets.error_rate * 100}%`);
  failed = true;
}

if (failed) {
  process.exit(1);
}

console.log('✅ Performance budget passed');
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# PERFORMANCE TESTING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Testing in wrong environment
# Bad
# Testing against local machine
# Results don't reflect production

# Good
# Test against production-like staging
# Same instance types, database size, network

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: No think time
# Bad
while (true) {
  http.get('/api/products');
  // No delay - unrealistic DoS attack
}

# Good
http.get('/api/products');
sleep(Math.random() * 3 + 1);  // 1-4 seconds think time

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Ignoring warm-up
# Bad
# Measure from first request
# Cold cache, cold connections skew results

# Good
stages: [
  { duration: '2m', target: 100 },  # Warm-up
  { duration: '10m', target: 100 }, # Actual test
]
# Exclude warm-up from metrics

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Only testing happy path
# Bad
# All requests succeed
# No error handling tested

# Good
# Include scenarios that fail
# Test rate limiting, validation errors
# Verify graceful degradation

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Averages instead of percentiles
# Bad
threshold: avg < 200ms
# Average hides tail latency
# 1% of users might wait 10 seconds

# Good
threshold: p99 < 500ms
# Catches tail latency issues

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Not monitoring system resources
# Bad
# Only measure response time
# Miss database CPU saturation, memory leaks

# Good
# Monitor: CPU, memory, connections, disk I/O
# Correlate with response times
# Identify bottlenecks
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is load testing vs stress testing?"**
> "Load testing: Verify performance under expected load (1000 users). Validates SLAs are met. Stress testing: Push beyond limits to find breaking point. Gradually increase until failure. Reveals how system degrades and recovers."

**Q: "What metrics do you measure?"**
> "Latency: p50, p95, p99 response times. Throughput: Requests per second. Error rate: Failed requests percentage. Resource utilization: CPU, memory, connections. p99 is crucial - catches tail latency affecting 1% of users."

**Q: "Why use percentiles instead of averages?"**
> "Averages hide outliers. If 99 requests take 100ms and 1 takes 10 seconds, average is 199ms - looks fine. But 1% of users have terrible experience. p99 = 10 seconds reveals the problem. Always use p95/p99 for latency."

### Intermediate Questions

**Q: "What is think time and why is it important?"**
> "Delay between user actions simulating real behavior (reading page, filling forms). Without think time, tests become unrealistic DoS attacks - all users fire continuously. Include random think time (1-5 seconds) for realistic load patterns."

**Q: "How do you identify bottlenecks?"**
> "Monitor everything during load test: application latency, database CPU/connections, cache hit rates, network bandwidth. Look for: CPU saturation, connection pool exhaustion, slow queries, memory growth. Correlate resource metrics with latency spikes."

**Q: "What is a soak test?"**
> "Extended duration test (hours/days) at moderate load. Catches: Memory leaks (memory grows over time), connection leaks, resource exhaustion, time-based bugs. Run 24-48 hours. Monitor memory trend - should be flat, not growing."

### Advanced Questions

**Q: "How do you performance test microservices?"**
> "Test individual services AND full system. Use distributed tracing to identify slow services. Test service-to-service calls under load. Verify circuit breakers work. Test cascading failures. Use service mesh metrics for per-service latency."

**Q: "How do you integrate performance testing in CI/CD?"**
> "Run baseline tests on PRs (fail if >10% regression). Nightly full load tests on staging. Performance budgets: p99 < 500ms, error rate < 1%. Store historical results for trend analysis. Alert on degradation. Block deployments that fail budgets."

**Q: "How do you test auto-scaling?"**
> "Spike tests: Sudden traffic increase, verify scale-out triggers. Measure time to scale. Verify performance during scaling. Test scale-in doesn't drop connections. Verify correct instance types spin up. Test with realistic traffic patterns."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│               PERFORMANCE TESTING CHECKLIST                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SETUP:                                                         │
│  □ Production-like environment                                 │
│  □ Realistic test data                                         │
│  □ Monitoring configured                                       │
│                                                                 │
│  TEST SCENARIOS:                                                │
│  □ Load test (expected traffic)                                │
│  □ Stress test (find limits)                                   │
│  □ Spike test (sudden surges)                                  │
│  □ Soak test (memory leaks)                                    │
│                                                                 │
│  METRICS:                                                       │
│  □ Latency: p50, p95, p99                                      │
│  □ Throughput: RPS                                             │
│  □ Error rate                                                  │
│  □ Resource utilization                                        │
│                                                                 │
│  THRESHOLDS:                                                    │
│  □ p99 < 500ms (or your SLO)                                   │
│  □ Error rate < 1%                                             │
│  □ CPU < 80%                                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

K6 QUICK START:
┌────────────────────────────────────────────────────────────────┐
│ k6 run script.js                    # Run test                 │
│ k6 run --vus 100 --duration 30s     # Quick test               │
│ k6 run --out json=results.json      # Save results             │
│ k6 run --env BASE_URL=https://...   # Pass variables           │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

