# Metrics & Dashboards - Complete Guide

> **MUST REMEMBER**: Metrics are numerical measurements over time (counters, gauges, histograms). Use RED method for services (Rate, Errors, Duration) and USE method for resources (Utilization, Saturation, Errors). Prometheus scrapes metrics, Grafana visualizes them. Custom metrics track business KPIs. Dashboards should answer specific questions, not show everything.

---

## How to Explain Like a Senior Developer

"Metrics tell you IF something is wrong, logs and traces tell you WHY. RED method for services: how many requests (Rate), how many fail (Errors), how long they take (Duration). USE method for resources: how busy is it (Utilization), is work queuing (Saturation), are there failures (Errors). Prometheus is the standard - expose a /metrics endpoint, Prometheus scrapes it, Grafana graphs it. The key is asking 'what questions do I need to answer?' and building dashboards for those. Don't create dashboards with 50 graphs - create focused ones for specific troubleshooting scenarios."

---

## Core Implementation

### Prometheus Metrics in Node.js

```typescript
// metrics/prometheus.ts
import { 
  Counter, 
  Histogram, 
  Gauge, 
  Registry, 
  collectDefaultMetrics 
} from 'prom-client';

// Create custom registry
const register = new Registry();

// Collect default Node.js metrics (CPU, memory, event loop)
collectDefaultMetrics({ register });

// HTTP Request metrics (RED method)
export const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'path', 'status_code'],
  registers: [register],
});

export const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'path', 'status_code'],
  buckets: [0.01, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10],
  registers: [register],
});

export const httpRequestsInProgress = new Gauge({
  name: 'http_requests_in_progress',
  help: 'Number of HTTP requests currently in progress',
  labelNames: ['method'],
  registers: [register],
});

// Database metrics
export const dbQueryDuration = new Histogram({
  name: 'db_query_duration_seconds',
  help: 'Database query duration in seconds',
  labelNames: ['query_type', 'table'],
  buckets: [0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1, 5],
  registers: [register],
});

export const dbConnectionPool = new Gauge({
  name: 'db_connection_pool_size',
  help: 'Number of connections in the database pool',
  labelNames: ['state'], // idle, active, waiting
  registers: [register],
});

// Business metrics
export const ordersCreated = new Counter({
  name: 'orders_created_total',
  help: 'Total number of orders created',
  labelNames: ['status', 'payment_method'],
  registers: [register],
});

export const orderValue = new Histogram({
  name: 'order_value_dollars',
  help: 'Order value in dollars',
  buckets: [10, 25, 50, 100, 250, 500, 1000],
  registers: [register],
});

export const activeUsers = new Gauge({
  name: 'active_users',
  help: 'Number of active users',
  registers: [register],
});

// Queue metrics
export const queueSize = new Gauge({
  name: 'job_queue_size',
  help: 'Number of jobs in queue',
  labelNames: ['queue_name', 'status'],
  registers: [register],
});

export const jobProcessingDuration = new Histogram({
  name: 'job_processing_duration_seconds',
  help: 'Job processing duration in seconds',
  labelNames: ['queue_name', 'job_type'],
  buckets: [0.1, 0.5, 1, 5, 10, 30, 60],
  registers: [register],
});

// Export registry
export { register };
```

### Express Metrics Middleware

```typescript
// metrics/middleware.ts
import { Request, Response, NextFunction } from 'express';
import { 
  httpRequestsTotal, 
  httpRequestDuration, 
  httpRequestsInProgress,
  register 
} from './prometheus';

// Metrics middleware
export function metricsMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  // Skip metrics endpoint
  if (req.path === '/metrics') {
    return next();
  }
  
  const start = process.hrtime.bigint();
  const method = req.method;
  
  // Track in-progress requests
  httpRequestsInProgress.labels(method).inc();
  
  // When response finishes
  res.on('finish', () => {
    const end = process.hrtime.bigint();
    const durationSeconds = Number(end - start) / 1e9;
    
    // Normalize path to avoid cardinality explosion
    const path = normalizePath(req.route?.path || req.path);
    const statusCode = res.statusCode.toString();
    
    // Record metrics
    httpRequestsTotal.labels(method, path, statusCode).inc();
    httpRequestDuration.labels(method, path, statusCode).observe(durationSeconds);
    httpRequestsInProgress.labels(method).dec();
  });
  
  next();
}

// Normalize paths to avoid high cardinality
function normalizePath(path: string): string {
  return path
    // Replace UUIDs
    .replace(/[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}/gi, ':id')
    // Replace numeric IDs
    .replace(/\/\d+/g, '/:id')
    // Replace MongoDB ObjectIds
    .replace(/[0-9a-f]{24}/gi, ':id');
}

// Metrics endpoint
export async function metricsEndpoint(
  req: Request,
  res: Response
): Promise<void> {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
}

// Setup
import express from 'express';

const app = express();
app.use(metricsMiddleware);
app.get('/metrics', metricsEndpoint);
```

### Custom Business Metrics

```typescript
// metrics/business.ts
import { 
  ordersCreated, 
  orderValue, 
  activeUsers,
  queueSize,
  jobProcessingDuration 
} from './prometheus';

// Track order creation
export function trackOrderCreated(
  status: 'success' | 'failed',
  paymentMethod: string,
  value: number
): void {
  ordersCreated.labels(status, paymentMethod).inc();
  
  if (status === 'success') {
    orderValue.observe(value);
  }
}

// Track active users (call periodically)
export function updateActiveUsers(count: number): void {
  activeUsers.set(count);
}

// Track queue metrics
export function updateQueueMetrics(
  queueName: string,
  waiting: number,
  active: number,
  completed: number,
  failed: number
): void {
  queueSize.labels(queueName, 'waiting').set(waiting);
  queueSize.labels(queueName, 'active').set(active);
  queueSize.labels(queueName, 'completed').set(completed);
  queueSize.labels(queueName, 'failed').set(failed);
}

// Track job processing
export function trackJobProcessed(
  queueName: string,
  jobType: string,
  durationSeconds: number
): void {
  jobProcessingDuration.labels(queueName, jobType).observe(durationSeconds);
}

// Example usage in order service
async function createOrder(orderData: any): Promise<void> {
  const start = Date.now();
  
  try {
    await processOrder(orderData);
    
    trackOrderCreated('success', orderData.paymentMethod, orderData.total);
  } catch (error) {
    trackOrderCreated('failed', orderData.paymentMethod, orderData.total);
    throw error;
  }
}

async function processOrder(orderData: any): Promise<void> {}
```

### Database Query Metrics

```typescript
// metrics/database.ts
import { dbQueryDuration, dbConnectionPool } from './prometheus';
import { Pool, PoolClient } from 'pg';

// Wrap database client with metrics
class MetricsDatabaseClient {
  constructor(private pool: Pool) {
    // Update pool metrics periodically
    setInterval(() => this.updatePoolMetrics(), 5000);
  }
  
  private updatePoolMetrics(): void {
    dbConnectionPool.labels('idle').set(this.pool.idleCount);
    dbConnectionPool.labels('active').set(this.pool.totalCount - this.pool.idleCount);
    dbConnectionPool.labels('waiting').set(this.pool.waitingCount);
  }
  
  async query<T>(
    queryType: string,
    table: string,
    sql: string,
    params?: any[]
  ): Promise<T[]> {
    const start = process.hrtime.bigint();
    
    try {
      const result = await this.pool.query(sql, params);
      return result.rows;
    } finally {
      const end = process.hrtime.bigint();
      const durationSeconds = Number(end - start) / 1e9;
      
      dbQueryDuration.labels(queryType, table).observe(durationSeconds);
    }
  }
  
  // Convenience methods
  async select<T>(table: string, sql: string, params?: any[]): Promise<T[]> {
    return this.query('SELECT', table, sql, params);
  }
  
  async insert(table: string, sql: string, params?: any[]): Promise<void> {
    await this.query('INSERT', table, sql, params);
  }
  
  async update(table: string, sql: string, params?: any[]): Promise<void> {
    await this.query('UPDATE', table, sql, params);
  }
  
  async delete(table: string, sql: string, params?: any[]): Promise<void> {
    await this.query('DELETE', table, sql, params);
  }
}
```

### RED Method Implementation

```typescript
// metrics/red.ts
import { Counter, Histogram, register } from 'prom-client';

/**
 * RED Method:
 * - Rate: Number of requests per second
 * - Errors: Number of failed requests per second
 * - Duration: Time to process requests
 */

interface REDMetrics {
  requestsTotal: Counter;
  errorsTotal: Counter;
  requestDuration: Histogram;
}

export function createREDMetrics(serviceName: string): REDMetrics {
  const requestsTotal = new Counter({
    name: `${serviceName}_requests_total`,
    help: `Total ${serviceName} requests`,
    labelNames: ['method', 'endpoint'],
    registers: [register],
  });
  
  const errorsTotal = new Counter({
    name: `${serviceName}_errors_total`,
    help: `Total ${serviceName} errors`,
    labelNames: ['method', 'endpoint', 'error_type'],
    registers: [register],
  });
  
  const requestDuration = new Histogram({
    name: `${serviceName}_request_duration_seconds`,
    help: `${serviceName} request duration`,
    labelNames: ['method', 'endpoint'],
    buckets: [0.01, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10],
    registers: [register],
  });
  
  return { requestsTotal, errorsTotal, requestDuration };
}

// Usage
const paymentMetrics = createREDMetrics('payment_service');

async function processPayment(paymentId: string): Promise<void> {
  const method = 'process';
  const endpoint = '/payments';
  const end = paymentMetrics.requestDuration.startTimer({ method, endpoint });
  
  paymentMetrics.requestsTotal.labels(method, endpoint).inc();
  
  try {
    await doProcessPayment(paymentId);
  } catch (error) {
    paymentMetrics.errorsTotal.labels(method, endpoint, (error as Error).name).inc();
    throw error;
  } finally {
    end();
  }
}

async function doProcessPayment(paymentId: string): Promise<void> {}
```

### USE Method Implementation

```typescript
// metrics/use.ts
import { Gauge, Counter, register } from 'prom-client';

/**
 * USE Method (for resources):
 * - Utilization: Percentage of time the resource is busy
 * - Saturation: Amount of work queued
 * - Errors: Number of error events
 */

interface USEMetrics {
  utilization: Gauge;
  saturation: Gauge;
  errors: Counter;
}

export function createUSEMetrics(resourceName: string): USEMetrics {
  const utilization = new Gauge({
    name: `${resourceName}_utilization_ratio`,
    help: `${resourceName} utilization (0-1)`,
    registers: [register],
  });
  
  const saturation = new Gauge({
    name: `${resourceName}_saturation`,
    help: `${resourceName} saturation (queued work)`,
    registers: [register],
  });
  
  const errors = new Counter({
    name: `${resourceName}_errors_total`,
    help: `${resourceName} error count`,
    labelNames: ['error_type'],
    registers: [register],
  });
  
  return { utilization, saturation, errors };
}

// Database pool USE metrics
const dbPoolMetrics = createUSEMetrics('db_pool');

function updateDatabaseMetrics(pool: {
  totalCount: number;
  idleCount: number;
  waitingCount: number;
}): void {
  // Utilization: active connections / total connections
  const active = pool.totalCount - pool.idleCount;
  dbPoolMetrics.utilization.set(active / pool.totalCount);
  
  // Saturation: waiting queries
  dbPoolMetrics.saturation.set(pool.waitingCount);
}

// Worker thread USE metrics
const workerMetrics = createUSEMetrics('worker_threads');

function updateWorkerMetrics(
  activeWorkers: number,
  totalWorkers: number,
  queuedJobs: number
): void {
  workerMetrics.utilization.set(activeWorkers / totalWorkers);
  workerMetrics.saturation.set(queuedJobs);
}
```

---

## Real-World Scenarios

### Scenario 1: Grafana Dashboard Configuration

```typescript
// dashboards/service-health.json
/*
{
  "dashboard": {
    "title": "Service Health Dashboard",
    "panels": [
      {
        "title": "Request Rate",
        "type": "timeseries",
        "targets": [{
          "expr": "rate(http_requests_total[5m])",
          "legendFormat": "{{method}} {{path}}"
        }]
      },
      {
        "title": "Error Rate",
        "type": "timeseries",
        "targets": [{
          "expr": "rate(http_requests_total{status_code=~\"5..\"}[5m]) / rate(http_requests_total[5m])",
          "legendFormat": "Error Rate"
        }]
      },
      {
        "title": "Request Duration (p99)",
        "type": "timeseries",
        "targets": [{
          "expr": "histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))",
          "legendFormat": "p99 {{path}}"
        }]
      },
      {
        "title": "Requests In Progress",
        "type": "gauge",
        "targets": [{
          "expr": "sum(http_requests_in_progress)"
        }]
      }
    ]
  }
}
*/

// PromQL queries for common scenarios

const commonQueries = {
  // Request rate per endpoint
  requestRate: 'rate(http_requests_total[5m])',
  
  // Error rate percentage
  errorRate: '100 * rate(http_requests_total{status_code=~"5.."}[5m]) / rate(http_requests_total[5m])',
  
  // P50, P95, P99 latencies
  p50Latency: 'histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))',
  p95Latency: 'histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))',
  p99Latency: 'histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))',
  
  // Apdex score (target: 0.5s)
  apdex: `(
    sum(rate(http_request_duration_seconds_bucket{le="0.5"}[5m])) +
    sum(rate(http_request_duration_seconds_bucket{le="2.0"}[5m])) / 2
  ) / sum(rate(http_request_duration_seconds_count[5m]))`,
  
  // Database connection utilization
  dbUtilization: '(sum(db_connection_pool_size{state="active"}) / sum(db_connection_pool_size)) * 100',
  
  // Queue depth
  queueDepth: 'sum(job_queue_size{status="waiting"}) by (queue_name)',
  
  // Memory usage
  memoryUsage: 'process_resident_memory_bytes / 1024 / 1024',
  
  // CPU usage
  cpuUsage: 'rate(process_cpu_seconds_total[5m]) * 100',
};
```

### Scenario 2: Alert Rules

```yaml
# prometheus/alerts.yml
groups:
  - name: service-alerts
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          (
            rate(http_requests_total{status_code=~"5.."}[5m])
            / rate(http_requests_total[5m])
          ) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }} for {{ $labels.service }}"
      
      # High latency
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "P95 latency is {{ $value }}s for {{ $labels.service }}"
      
      # Low request rate (potential outage)
      - alert: LowTraffic
        expr: |
          rate(http_requests_total[5m]) < 1 
          unless on() hour() >= 0 and hour() < 6
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Unusually low traffic"
          description: "Request rate dropped to {{ $value }}/s"
      
      # Database connection pool exhausted
      - alert: DBPoolExhausted
        expr: |
          db_connection_pool_size{state="waiting"} > 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Database connection pool exhausted"
          description: "{{ $value }} queries waiting for connections"
      
      # Job queue backing up
      - alert: JobQueueBacklog
        expr: |
          job_queue_size{status="waiting"} > 1000
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Job queue backlog"
          description: "{{ $value }} jobs waiting in {{ $labels.queue_name }}"
```

---

## Common Pitfalls

### 1. High Cardinality Labels

```typescript
// ❌ BAD: User ID as label (millions of unique values)
httpRequestsTotal.labels(method, path, userId).inc();

// ✅ GOOD: Use bounded labels
httpRequestsTotal.labels(method, path, statusCode).inc();

// Track user-specific metrics differently
// Use logs or traces for user-level debugging
```

### 2. Missing Labels for Filtering

```typescript
// ❌ BAD: Can't filter by method or endpoint
const requestsTotal = new Counter({
  name: 'requests_total',
  help: 'Total requests',
});
requestsTotal.inc();

// ✅ GOOD: Labels allow filtering
const requestsTotal = new Counter({
  name: 'requests_total',
  help: 'Total requests',
  labelNames: ['method', 'endpoint', 'status'],
});
requestsTotal.labels('GET', '/users', '200').inc();
```

### 3. Not Using Histograms for Latency

```typescript
// ❌ BAD: Average hides outliers
const avgDuration = new Gauge({
  name: 'request_duration_avg',
  help: 'Average request duration',
});

// ✅ GOOD: Histogram shows distribution
const requestDuration = new Histogram({
  name: 'request_duration_seconds',
  help: 'Request duration histogram',
  buckets: [0.01, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10],
});
// Can calculate p50, p95, p99
```

---

## Interview Questions

### Q1: What's the difference between Counter, Gauge, and Histogram?

**A:** **Counter** only goes up (requests, errors) - use rate() to get per-second. **Gauge** can go up or down (active connections, temperature). **Histogram** buckets values into ranges, allowing percentile calculations. Use counters for "how many", gauges for "how much right now", histograms for "how long/how big".

### Q2: Explain the RED and USE methods.

**A:** **RED** (for services): Rate (requests/sec), Errors (failed requests/sec), Duration (latency). **USE** (for resources like CPU, memory, connections): Utilization (% time busy), Saturation (work waiting), Errors (failures). RED answers "is the service healthy?", USE answers "is the resource healthy?".

### Q3: What is label cardinality and why does it matter?

**A:** Cardinality is the number of unique label combinations. High cardinality (e.g., user ID as label) creates millions of time series, causing storage explosion and query slowness. Keep cardinality bounded - use fixed values like status codes, HTTP methods. For high-cardinality data, use logs or traces instead.

### Q4: How do you calculate percentiles from Prometheus histograms?

**A:** Use `histogram_quantile()` function. For p99: `histogram_quantile(0.99, rate(request_duration_seconds_bucket[5m]))`. The function interpolates between bucket boundaries. Choose bucket boundaries carefully for accurate percentiles in your expected latency range.

---

## Quick Reference Checklist

### Metric Types
- [ ] Counters for totals (requests, errors)
- [ ] Gauges for current values (connections, queue size)
- [ ] Histograms for latency/size distributions

### RED Method (Services)
- [ ] Request rate
- [ ] Error rate
- [ ] Request duration (percentiles)

### USE Method (Resources)
- [ ] Utilization (% busy)
- [ ] Saturation (queued work)
- [ ] Error count

### Best Practices
- [ ] Keep label cardinality bounded
- [ ] Use meaningful bucket sizes
- [ ] Normalize paths in labels
- [ ] Expose /metrics endpoint

---

*Last updated: February 2026*

