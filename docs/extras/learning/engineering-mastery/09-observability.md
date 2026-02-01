# Chapter 09: Observability

> "You can't fix what you can't see."

---

## 🎯 The Three Pillars

```
┌─────────────────────────────────────────────────────────────────┐
│                       OBSERVABILITY                             │
│                                                                 │
│    ┌───────────────┐  ┌───────────────┐  ┌───────────────┐     │
│    │     LOGS      │  │    METRICS    │  │    TRACES     │     │
│    │               │  │               │  │               │     │
│    │  What         │  │  How          │  │  Why          │     │
│    │  happened     │  │  is it        │  │  did it       │     │
│    │               │  │  performing   │  │  happen       │     │
│    └───────────────┘  └───────────────┘  └───────────────┘     │
│                                                                 │
│    Debug specific    Monitor overall    Trace request          │
│    events            health             flow                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Logging

### Log Levels

```
TRACE   - Most detailed, for debugging internals
DEBUG   - Detailed debugging info
INFO    - General operational events
WARN    - Potential issues, recoverable
ERROR   - Errors that need attention
FATAL   - System is unusable

Production typically: INFO and above
```

### Structured Logging

```javascript
// BAD: Unstructured
console.log('User login failed for user123');

// GOOD: Structured (JSON)
logger.info({
  event: 'user.login.failed',
  userId: 'user123',
  reason: 'invalid_password',
  ip: '192.168.1.1',
  timestamp: '2024-01-15T10:30:00Z',
  requestId: 'req-abc123',
  traceId: 'trace-xyz789'
});

// Benefits:
// - Searchable: event:"user.login.failed" AND reason:invalid_password
// - Parseable: Can aggregate, analyze
// - Correlatable: requestId links related logs
```

### Logging Best Practices

```javascript
// 1. Include context
logger.error({
  event: 'payment.failed',
  orderId: order.id,
  userId: user.id,
  amount: order.total,
  error: err.message,
  stack: err.stack
});

// 2. Use correlation IDs
app.use((req, res, next) => {
  req.requestId = uuid();
  req.traceId = req.headers['x-trace-id'] || uuid();
  next();
});

// 3. Don't log sensitive data
logger.info({
  event: 'user.created',
  userId: user.id,
  email: maskEmail(user.email),  // ja***@example.com
  // password: user.password  ← NEVER!
});

// 4. Log at boundaries
// - Incoming requests
// - Outgoing requests (to APIs, databases)
// - Business events (order created, payment received)
```

### Log Aggregation Stack

```
┌──────────────────────────────────────────────────────────────┐
│                     ELK Stack                                │
│                                                              │
│  Apps → Filebeat → Logstash → Elasticsearch → Kibana        │
│         (collect)   (process)   (store/search)  (visualize) │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                   Grafana Loki Stack                         │
│                                                              │
│  Apps → Promtail → Loki → Grafana                           │
│         (collect)  (store) (visualize)                       │
│                                                              │
│  Pros: Less storage than ELK, integrates with Grafana       │
└──────────────────────────────────────────────────────────────┘
```

---

## 📊 Metrics

### Metric Types

```
Counter: Only increases (requests, errors)
┌─────────────────────────────────────┐
│ http_requests_total{status="200"}   │
│ Value: 12,345 → 12,346 → 12,347     │
└─────────────────────────────────────┘

Gauge: Can go up or down (temperature, queue size)
┌─────────────────────────────────────┐
│ queue_length{queue="orders"}        │
│ Value: 50 → 75 → 30 → 45            │
└─────────────────────────────────────┘

Histogram: Distribution of values (latency)
┌─────────────────────────────────────┐
│ http_request_duration_seconds       │
│ Buckets: <0.1, <0.5, <1.0, <5.0     │
│ Count per bucket: 100, 500, 50, 10  │
└─────────────────────────────────────┘

Summary: Percentiles (p50, p95, p99)
┌─────────────────────────────────────┐
│ request_latency{quantile="0.99"}    │
│ Value: 0.35 (99% of requests < 350ms)│
└─────────────────────────────────────┘
```

### Key Metrics (RED Method)

```
For Services:
┌─────────────────────────────────────────────────────────────┐
│ Rate    - Requests per second                               │
│ Errors  - Failed requests per second                        │
│ Duration - Latency (p50, p95, p99)                          │
└─────────────────────────────────────────────────────────────┘

For Resources (USE Method):
┌─────────────────────────────────────────────────────────────┐
│ Utilization - % of resource used (CPU, memory)              │
│ Saturation  - Amount of queued work                         │
│ Errors      - Error count                                   │
└─────────────────────────────────────────────────────────────┘
```

### Prometheus + Grafana

```javascript
const prometheus = require('prom-client');

// Counter
const httpRequests = new prometheus.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'path', 'status']
});

// Histogram
const httpDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'path'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

// Middleware
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    
    httpRequests.inc({
      method: req.method,
      path: req.route?.path || req.path,
      status: res.statusCode
    });
    
    httpDuration.observe({
      method: req.method,
      path: req.route?.path || req.path
    }, duration);
  });
  
  next();
});

// Expose metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', prometheus.register.contentType);
  res.end(await prometheus.register.metrics());
});
```

### Grafana Dashboard Example

```
┌────────────────────────────────────────────────────────────────────┐
│                        API Dashboard                               │
├────────────────────────────────────────────────────────────────────┤
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌────────────┐ │
│ │ Requests/sec │ │ Error Rate   │ │ p99 Latency  │ │ Uptime     │ │
│ │    1,234     │ │    0.5%      │ │    250ms     │ │   99.99%   │ │
│ └──────────────┘ └──────────────┘ └──────────────┘ └────────────┘ │
├────────────────────────────────────────────────────────────────────┤
│                     Request Rate Over Time                         │
│     ▲                                                              │
│ 2K  │     ╭─╮                                                     │
│     │    ╭╯ ╰─╮   ╭──╮                                            │
│ 1K  │───╯     ╰──╯  ╰───                                          │
│     │                                                              │
│     └──────────────────────────────────────────►                   │
│            Time                                                    │
├────────────────────────────────────────────────────────────────────┤
│                     Latency Percentiles                            │
│     ▲                                                              │
│500ms│ ─ ─ ─ ─ ─ ─ ─ ─ ─ p99                                       │
│     │ ─────────────── p95                                          │
│100ms│ ══════════════ p50                                          │
│     └──────────────────────────────────────────►                   │
└────────────────────────────────────────────────────────────────────┘
```

---

## 🔍 Distributed Tracing

### How Tracing Works

```
Request flows through multiple services:

User → API Gateway → Auth Service → Order Service → Database
         │                              │
         └──► Payment Service ──────────┘

Trace: The entire journey
Span: One hop (API Gateway → Auth Service)

┌─────────────────────────────────────────────────────────────────┐
│ Trace ID: abc-123                                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ API Gateway ────────────────────────────────────────── 200ms    │
│   │                                                             │
│   └─► Auth Service ─────────────────────────────────── 50ms     │
│   │                                                             │
│   └─► Order Service ────────────────────────────────── 100ms    │
│         │                                                       │
│         ├─► Database Query ─────────────────────────── 30ms     │
│         │                                                       │
│         └─► Payment Service ────────────────────────── 40ms     │
│               │                                                 │
│               └─► Payment API ──────────────────────── 35ms     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### OpenTelemetry

```javascript
const { trace, context, SpanStatusCode } = require('@opentelemetry/api');
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger');

// Initialize
const provider = new NodeTracerProvider();
provider.addSpanProcessor(
  new SimpleSpanProcessor(new JaegerExporter())
);
provider.register();

const tracer = trace.getTracer('my-service');

// Create spans
async function processOrder(orderId) {
  return tracer.startActiveSpan('processOrder', async (span) => {
    try {
      span.setAttribute('order.id', orderId);
      
      // Child span for database
      await tracer.startActiveSpan('database.query', async (dbSpan) => {
        const order = await db.orders.findById(orderId);
        dbSpan.setAttribute('db.statement', 'SELECT * FROM orders');
        dbSpan.end();
        return order;
      });
      
      // Child span for payment
      await tracer.startActiveSpan('payment.process', async (paySpan) => {
        const result = await paymentService.charge(order);
        paySpan.setAttribute('payment.amount', order.total);
        paySpan.end();
        return result;
      });
      
      span.setStatus({ code: SpanStatusCode.OK });
    } catch (error) {
      span.setStatus({
        code: SpanStatusCode.ERROR,
        message: error.message
      });
      span.recordException(error);
      throw error;
    } finally {
      span.end();
    }
  });
}

// Propagate context in HTTP calls
async function callService(url, data) {
  const headers = {};
  propagation.inject(context.active(), headers);
  
  return fetch(url, {
    method: 'POST',
    headers: {
      ...headers,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
  });
}
```

---

## 🚨 Alerting

### Alert Design

```
Good alerts:
✓ Actionable - Someone can do something
✓ Specific - Clear what's wrong
✓ Urgent - Needs immediate attention
✓ Meaningful - Affects users/business

Bad alerts:
✗ CPU > 80% (might be fine)
✗ 1 error occurred (might be normal)
✗ Disk space < 50% (not urgent)

Better:
✓ Error rate > 1% for 5 minutes
✓ p99 latency > 2s for 5 minutes
✓ No successful requests for 2 minutes
✓ Disk space < 10% (predict when it runs out)
```

### SLO-Based Alerting

```
SLI (Service Level Indicator):
  Measurement of service behavior
  Example: % of requests < 200ms

SLO (Service Level Objective):
  Target for SLI
  Example: 99.9% of requests < 200ms

Error Budget:
  Allowed failures before SLO breach
  Example: 0.1% = 43 minutes/month downtime

Alert when:
┌────────────────────────────────────────────────────────────┐
│ Burn rate alert:                                           │
│                                                            │
│ Fast burn: >14x budget consumption for 1 hour              │
│   → Page on-call immediately                               │
│                                                            │
│ Slow burn: >6x budget consumption for 6 hours              │
│   → Create ticket, investigate                             │
└────────────────────────────────────────────────────────────┘
```

### Alert Routing

```yaml
# AlertManager configuration
route:
  group_by: ['alertname', 'service']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'slack-default'
  
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty-oncall'
      
    - match:
        severity: warning
      receiver: 'slack-warnings'

receivers:
  - name: 'pagerduty-oncall'
    pagerduty_configs:
      - service_key: '<key>'
        
  - name: 'slack-default'
    slack_configs:
      - channel: '#alerts'
```

---

## 📖 Further Reading

- "Observability Engineering" by Charity Majors
- "Site Reliability Engineering" by Google
- "Distributed Tracing in Practice"
- OpenTelemetry documentation

---

**Next:** [Chapter 10: DevOps & SRE →](./10-devops-sre.md)


