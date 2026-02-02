# 🔍 APM (Application Performance Monitoring) - Complete Guide

> A comprehensive guide to APM - distributed tracing, spans, New Relic, Datadog APM, OpenTelemetry, and performance optimization.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "APM (Application Performance Monitoring) provides deep visibility into application behavior through distributed tracing, error tracking, and performance metrics - allowing you to trace individual requests across microservices, identify bottlenecks, and understand exactly where time is spent."

### The 7 Key Concepts (Remember These!)
```
1. TRACE           → End-to-end journey of a request
2. SPAN            → Single operation within a trace
3. CONTEXT         → Trace/span IDs propagated across services
4. INSTRUMENTATION → Code that captures trace data
5. SAMPLING        → Deciding which traces to collect
6. SERVICE MAP     → Visual representation of service dependencies
7. APDEX           → Application Performance Index score
```

### Distributed Tracing Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                  DISTRIBUTED TRACING                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  User Request                                                  │
│       │                                                        │
│       ▼                                                        │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │ API Gateway                                              │  │
│  │ Trace ID: abc123                                        │  │
│  │ Span: gateway (0-50ms)                                  │  │
│  └─────────────────────────────────────────────────────────┘  │
│       │                                                        │
│       ▼                                                        │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │ Order Service                                           │  │
│  │ Trace ID: abc123 (propagated)                          │  │
│  │ Span: order-service (50-200ms)                         │  │
│  │   └── Span: validate-order (50-75ms)                   │  │
│  │   └── Span: db-query (75-150ms)                        │  │
│  │   └── Span: call-payment (150-200ms)                   │  │
│  └─────────────────────────────────────────────────────────┘  │
│       │                                                        │
│       ▼                                                        │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │ Payment Service                                         │  │
│  │ Trace ID: abc123 (propagated)                          │  │
│  │ Span: payment-service (200-450ms)                      │  │
│  │   └── Span: fraud-check (200-250ms)                    │  │
│  │   └── Span: charge-card (250-400ms)                    │  │
│  │   └── Span: save-transaction (400-450ms)               │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                                 │
│  COMPLETE TRACE VIEW:                                          │
│  ─────────────────────────────────────────────────────────────  │
│  |──gateway────|                                               │
│                |──order-service────────|                       │
│                |─validate─|                                    │
│                           |──db-query──|                       │
│                                        |─call─|                │
│                                             |──payment-service──│
│                                             |─fraud─|           │
│                                                     |─charge───|│
│                                                                ││
│  0ms                    200ms                    400ms    450ms │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Span Structure
```
┌─────────────────────────────────────────────────────────────────┐
│                      SPAN ANATOMY                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  {                                                             │
│    "traceId": "abc123def456",                                  │
│    "spanId": "span789",                                        │
│    "parentSpanId": "span456",                                  │
│    "operationName": "POST /api/orders",                        │
│    "serviceName": "order-service",                             │
│    "startTime": "2024-01-15T10:30:00.000Z",                   │
│    "duration": 150,  // milliseconds                           │
│    "status": "OK",                                             │
│                                                                 │
│    "tags": {                                                   │
│      "http.method": "POST",                                    │
│      "http.url": "/api/orders",                                │
│      "http.status_code": 201,                                  │
│      "user.id": "user_123",                                    │
│      "db.type": "postgresql"                                   │
│    },                                                          │
│                                                                 │
│    "logs": [                                                   │
│      {                                                         │
│        "timestamp": "2024-01-15T10:30:00.050Z",               │
│        "message": "Order validated successfully"               │
│      }                                                         │
│    ],                                                          │
│                                                                 │
│    "events": [                                                 │
│      {                                                         │
│        "name": "order.created",                                │
│        "timestamp": "2024-01-15T10:30:00.150Z",               │
│        "attributes": { "orderId": "order_456" }                │
│      }                                                         │
│    ]                                                           │
│  }                                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Trace context"** | "We propagate trace context via W3C Trace Context headers" |
| **"Span attributes"** | "We enrich spans with custom attributes for debugging" |
| **"Head-based sampling"** | "We use head-based sampling at 10% for cost control" |
| **"Tail-based sampling"** | "Tail-based sampling captures all errors regardless of rate" |
| **"Service map"** | "The service map shows our dependency graph and latency" |
| **"Flame graph"** | "Flame graphs help identify CPU hot spots in code" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Apdex satisfactory | **< 500ms** | Typical web response target |
| Apdex tolerating | **< 2000ms** | Acceptable threshold |
| Sampling rate | **1-10%** | Balance cost vs visibility |
| Trace retention | **7-30 days** | Analysis window |
| Error trace | **100%** | Always capture errors |
| Span limit | **100-1000/trace** | Prevent runaway traces |

### The "Wow" Statement (Memorize This!)
> "We use OpenTelemetry for distributed tracing across 50+ microservices. The SDK auto-instruments HTTP, database, and message queue calls. We propagate trace context using W3C Trace Context headers. For cost control, we use probabilistic sampling at 10% for normal traces, but tail-based sampling captures 100% of errors and slow requests (>2s). Traces export to Jaeger for analysis. Service maps show dependencies and latency between services. When debugging, we can trace a single user request end-to-end, seeing exactly which service added latency and why. Custom span attributes capture business context like orderId and userId. Flame graphs help identify CPU bottlenecks in hot code paths. Our Apdex score stays above 0.95."

---

## 📚 Table of Contents

1. [OpenTelemetry](#1-opentelemetry)
2. [Distributed Tracing](#2-distributed-tracing)
3. [New Relic](#3-new-relic)
4. [Datadog APM](#4-datadog-apm)
5. [Jaeger](#5-jaeger)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. OpenTelemetry

```typescript
// ════════════════════════════════════════════════════════════════
// OPENTELEMETRY SETUP - NODE.JS
// tracing.ts (must be imported first!)
// ════════════════════════════════════════════════════════════════

import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { OTLPMetricExporter } from '@opentelemetry/exporter-metrics-otlp-http';
import { Resource } from '@opentelemetry/resources';
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions';
import { BatchSpanProcessor } from '@opentelemetry/sdk-trace-base';
import { PeriodicExportingMetricReader } from '@opentelemetry/sdk-metrics';

// Configure resource (service identity)
const resource = new Resource({
  [SemanticResourceAttributes.SERVICE_NAME]: process.env.SERVICE_NAME || 'api-service',
  [SemanticResourceAttributes.SERVICE_VERSION]: process.env.APP_VERSION || '1.0.0',
  [SemanticResourceAttributes.DEPLOYMENT_ENVIRONMENT]: process.env.NODE_ENV || 'development',
});

// Configure exporters
const traceExporter = new OTLPTraceExporter({
  url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT || 'http://otel-collector:4318/v1/traces',
});

const metricExporter = new OTLPMetricExporter({
  url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT || 'http://otel-collector:4318/v1/metrics',
});

// Initialize SDK
const sdk = new NodeSDK({
  resource,
  spanProcessor: new BatchSpanProcessor(traceExporter),
  metricReader: new PeriodicExportingMetricReader({
    exporter: metricExporter,
    exportIntervalMillis: 30000,
  }),
  instrumentations: [
    getNodeAutoInstrumentations({
      // Enable specific instrumentations
      '@opentelemetry/instrumentation-http': {
        enabled: true,
        ignoreIncomingPaths: ['/health', '/metrics'],
      },
      '@opentelemetry/instrumentation-express': { enabled: true },
      '@opentelemetry/instrumentation-pg': { enabled: true },
      '@opentelemetry/instrumentation-redis': { enabled: true },
      '@opentelemetry/instrumentation-mongodb': { enabled: true },
    }),
  ],
});

// Start SDK before app code
sdk.start();

// Graceful shutdown
process.on('SIGTERM', () => {
  sdk.shutdown()
    .then(() => console.log('Tracing terminated'))
    .catch((error) => console.log('Error terminating tracing', error))
    .finally(() => process.exit(0));
});

export { sdk };


// ════════════════════════════════════════════════════════════════
// CUSTOM SPANS AND ATTRIBUTES
// ════════════════════════════════════════════════════════════════

import { trace, SpanStatusCode, context } from '@opentelemetry/api';

const tracer = trace.getTracer('order-service');

class OrderService {
  async createOrder(orderData: OrderInput): Promise<Order> {
    // Create a custom span
    return tracer.startActiveSpan('createOrder', async (span) => {
      try {
        // Add attributes (searchable tags)
        span.setAttribute('order.user_id', orderData.userId);
        span.setAttribute('order.item_count', orderData.items.length);
        span.setAttribute('order.total', orderData.total);

        // Nested span for validation
        const validatedOrder = await tracer.startActiveSpan(
          'validateOrder',
          async (validationSpan) => {
            try {
              const result = await this.validateOrder(orderData);
              validationSpan.setAttribute('validation.passed', true);
              return result;
            } finally {
              validationSpan.end();
            }
          }
        );

        // Nested span for database
        const order = await tracer.startActiveSpan(
          'saveToDatabase',
          { attributes: { 'db.system': 'postgresql' } },
          async (dbSpan) => {
            try {
              const savedOrder = await this.repository.save(validatedOrder);
              dbSpan.setAttribute('db.operation', 'INSERT');
              dbSpan.setAttribute('order.id', savedOrder.id);
              return savedOrder;
            } finally {
              dbSpan.end();
            }
          }
        );

        // Add event (point-in-time occurrence)
        span.addEvent('order.created', {
          'order.id': order.id,
        });

        span.setStatus({ code: SpanStatusCode.OK });
        return order;
      } catch (error) {
        // Record error in span
        span.recordException(error);
        span.setStatus({
          code: SpanStatusCode.ERROR,
          message: error.message,
        });
        throw error;
      } finally {
        span.end();
      }
    });
  }
}


// ════════════════════════════════════════════════════════════════
// CONTEXT PROPAGATION
// ════════════════════════════════════════════════════════════════

import { propagation, context } from '@opentelemetry/api';

// Extract context from incoming request
function extractContext(req: Request) {
  return propagation.extract(context.active(), req.headers);
}

// Inject context into outgoing request
async function callExternalService(url: string, data: any) {
  const headers: Record<string, string> = {};
  propagation.inject(context.active(), headers);

  return fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      ...headers,  // Includes traceparent, tracestate
    },
    body: JSON.stringify(data),
  });
}

// Middleware to extract and use trace context
function tracingMiddleware(req: Request, res: Response, next: NextFunction) {
  const extractedContext = extractContext(req);
  
  context.with(extractedContext, () => {
    // All code in this callback uses the extracted context
    next();
  });
}
```

---

## 2. Distributed Tracing

```yaml
# ════════════════════════════════════════════════════════════════
# OPENTELEMETRY COLLECTOR CONFIGURATION
# otel-collector-config.yaml
# ════════════════════════════════════════════════════════════════

receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 5s
    send_batch_size: 1000

  memory_limiter:
    check_interval: 1s
    limit_mib: 1000
    spike_limit_mib: 200

  # Resource detection
  resourcedetection:
    detectors: [env, system, docker]
    timeout: 5s

  # Tail-based sampling
  tail_sampling:
    decision_wait: 10s
    num_traces: 100000
    policies:
      # Always sample errors
      - name: errors
        type: status_code
        status_code: { status_codes: [ERROR] }
      
      # Always sample slow traces
      - name: slow-traces
        type: latency
        latency: { threshold_ms: 2000 }
      
      # Sample 10% of normal traces
      - name: probabilistic
        type: probabilistic
        probabilistic: { sampling_percentage: 10 }

exporters:
  # Jaeger
  jaeger:
    endpoint: jaeger:14250
    tls:
      insecure: true

  # Prometheus (for metrics)
  prometheus:
    endpoint: 0.0.0.0:8889

  # Logging (for debugging)
  logging:
    loglevel: debug

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch, tail_sampling]
      exporters: [jaeger]
    
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [prometheus]

---
# ════════════════════════════════════════════════════════════════
# KUBERNETES DEPLOYMENT
# ════════════════════════════════════════════════════════════════

apiVersion: apps/v1
kind: Deployment
metadata:
  name: otel-collector
  namespace: monitoring
spec:
  replicas: 2
  selector:
    matchLabels:
      app: otel-collector
  template:
    metadata:
      labels:
        app: otel-collector
    spec:
      containers:
        - name: otel-collector
          image: otel/opentelemetry-collector-contrib:latest
          args:
            - --config=/conf/otel-collector-config.yaml
          ports:
            - containerPort: 4317  # gRPC
            - containerPort: 4318  # HTTP
            - containerPort: 8889  # Prometheus metrics
          resources:
            limits:
              cpu: 1
              memory: 2Gi
            requests:
              cpu: 200m
              memory: 400Mi
          volumeMounts:
            - name: config
              mountPath: /conf
      volumes:
        - name: config
          configMap:
            name: otel-collector-config
```

---

## 3. New Relic

```typescript
// ════════════════════════════════════════════════════════════════
// NEW RELIC AGENT - NODE.JS
// newrelic.js (must be in project root)
// ════════════════════════════════════════════════════════════════

'use strict';

exports.config = {
  app_name: [process.env.NEW_RELIC_APP_NAME || 'My Application'],
  license_key: process.env.NEW_RELIC_LICENSE_KEY,
  distributed_tracing: {
    enabled: true
  },
  logging: {
    level: 'info'
  },
  allow_all_headers: true,
  attributes: {
    exclude: [
      'request.headers.cookie',
      'request.headers.authorization',
      'request.headers.proxyAuthorization',
      'request.headers.setCookie*',
      'request.headers.x*',
      'response.headers.cookie',
      'response.headers.authorization',
      'response.headers.proxyAuthorization',
      'response.headers.setCookie*',
      'response.headers.x*'
    ]
  },
  transaction_tracer: {
    enabled: true,
    transaction_threshold: 'apdex_f',
    record_sql: 'obfuscated',
    explain_threshold: 500
  },
  slow_sql: {
    enabled: true,
    max_samples: 10
  },
  error_collector: {
    enabled: true,
    ignore_status_codes: [404]
  }
};

// ════════════════════════════════════════════════════════════════
// NEW RELIC CUSTOM INSTRUMENTATION
// ════════════════════════════════════════════════════════════════

// In application code
const newrelic = require('newrelic');

class OrderService {
  async createOrder(orderData: OrderInput) {
    // Custom transaction name
    newrelic.setTransactionName('Order/Create');

    // Add custom attributes
    newrelic.addCustomAttribute('userId', orderData.userId);
    newrelic.addCustomAttribute('orderTotal', orderData.total);

    // Create custom segment (span)
    return newrelic.startSegment('validateOrder', true, async () => {
      await this.validateOrder(orderData);

      return newrelic.startSegment('saveOrder', true, async () => {
        const order = await this.repository.save(orderData);
        
        // Record custom event
        newrelic.recordCustomEvent('OrderCreated', {
          orderId: order.id,
          total: order.total,
          itemCount: order.items.length
        });

        return order;
      });
    });
  }
}

// Error handling with New Relic
app.use((err, req, res, next) => {
  newrelic.noticeError(err, {
    userId: req.user?.id,
    path: req.path
  });
  
  res.status(500).json({ error: 'Internal Server Error' });
});

// Custom metric
newrelic.recordMetric('Custom/OrderValue', orderTotal);
```

---

## 4. Datadog APM

```typescript
// ════════════════════════════════════════════════════════════════
// DATADOG APM - NODE.JS
// tracer.ts (import first!)
// ════════════════════════════════════════════════════════════════

import tracer from 'dd-trace';

tracer.init({
  service: process.env.DD_SERVICE || 'api-service',
  env: process.env.DD_ENV || 'production',
  version: process.env.DD_VERSION || '1.0.0',
  
  // Sampling
  sampleRate: 1,  // 100% (adjust for high volume)
  
  // Runtime metrics
  runtimeMetrics: true,
  
  // Log injection
  logInjection: true,
  
  // Profiling
  profiling: true,
  
  // Debug
  debug: process.env.DD_TRACE_DEBUG === 'true',
});

export default tracer;


// ════════════════════════════════════════════════════════════════
// CUSTOM SPANS AND TAGS
// ════════════════════════════════════════════════════════════════

import tracer from './tracer';

class OrderService {
  async createOrder(orderData: OrderInput) {
    // Create custom span
    const span = tracer.startSpan('order.create', {
      tags: {
        'user.id': orderData.userId,
        'order.item_count': orderData.items.length,
      }
    });

    try {
      // Child span
      const validateSpan = tracer.startSpan('order.validate', {
        childOf: span
      });
      await this.validateOrder(orderData);
      validateSpan.finish();

      // Database span (usually auto-instrumented)
      const order = await tracer.trace('order.save', async () => {
        return this.repository.save(orderData);
      });

      // Add more tags after operation
      span.setTag('order.id', order.id);
      span.setTag('order.total', order.total);

      return order;
    } catch (error) {
      span.setTag('error', true);
      span.setTag('error.message', error.message);
      span.setTag('error.stack', error.stack);
      throw error;
    } finally {
      span.finish();
    }
  }
}

// Using scope manager for async context
async function processOrder(orderId: string) {
  return tracer.trace('process.order', { tags: { orderId } }, async (span) => {
    // All nested async operations will be children of this span
    await validatePayment();
    await updateInventory();
    await sendNotification();
  });
}
```

---

## 5. Jaeger

```yaml
# ════════════════════════════════════════════════════════════════
# JAEGER DEPLOYMENT
# jaeger-all-in-one.yaml (development)
# ════════════════════════════════════════════════════════════════

apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jaeger
  template:
    metadata:
      labels:
        app: jaeger
    spec:
      containers:
        - name: jaeger
          image: jaegertracing/all-in-one:latest
          ports:
            - containerPort: 6831   # UDP - Thrift compact
            - containerPort: 6832   # UDP - Thrift binary
            - containerPort: 5778   # HTTP - configs
            - containerPort: 16686  # HTTP - UI
            - containerPort: 14268  # HTTP - spans
            - containerPort: 14250  # gRPC - model
            - containerPort: 4317   # gRPC - OTLP
            - containerPort: 4318   # HTTP - OTLP
          env:
            - name: COLLECTOR_OTLP_ENABLED
              value: "true"
          resources:
            limits:
              memory: 1Gi
              cpu: 500m

---
# ════════════════════════════════════════════════════════════════
# JAEGER PRODUCTION (with Elasticsearch)
# ════════════════════════════════════════════════════════════════

apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger-collector
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: jaeger-collector
          image: jaegertracing/jaeger-collector:latest
          args:
            - --es.server-urls=http://elasticsearch:9200
            - --es.index-prefix=jaeger
            - --collector.num-workers=50
          env:
            - name: SPAN_STORAGE_TYPE
              value: elasticsearch
          ports:
            - containerPort: 14268
            - containerPort: 14250
            - containerPort: 4317
            - containerPort: 4318

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger-query
spec:
  replicas: 2
  template:
    spec:
      containers:
        - name: jaeger-query
          image: jaegertracing/jaeger-query:latest
          args:
            - --es.server-urls=http://elasticsearch:9200
          env:
            - name: SPAN_STORAGE_TYPE
              value: elasticsearch
          ports:
            - containerPort: 16686
            - containerPort: 16687
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# APM PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Not propagating context
# Bad - Context lost between services
async function callPaymentService(order) {
  const response = await fetch('http://payment/charge', {
    method: 'POST',
    body: JSON.stringify(order)
  });
  // No trace context propagated!
}

# Good - Propagate trace context
async function callPaymentService(order) {
  const headers = {};
  propagation.inject(context.active(), headers);
  
  const response = await fetch('http://payment/charge', {
    method: 'POST',
    headers: {
      ...headers,  // traceparent, tracestate
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(order)
  });
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: High cardinality span names
# Bad
span.name = `GET /users/${userId}`;  # Millions of unique names!

# Good
span.name = 'GET /users/:id';
span.setAttribute('user.id', userId);

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: No sampling strategy
# Bad - 100% sampling in production
sampling_rate: 1.0  # Huge storage costs, performance impact

# Good - Smart sampling
policies:
  - name: errors
    type: status_code  # 100% of errors
  - name: slow
    type: latency      # 100% of slow requests
  - name: normal
    type: probabilistic
    percentage: 10     # 10% of normal requests

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Missing error details
# Bad
span.setStatus({ code: SpanStatusCode.ERROR });
# What error? No details!

# Good
span.recordException(error);
span.setStatus({
  code: SpanStatusCode.ERROR,
  message: error.message
});
span.setAttribute('error.type', error.name);

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Not ending spans
# Bad
const span = tracer.startSpan('operation');
doSomething();
# Span never ended - memory leak, inaccurate timing

# Good
const span = tracer.startSpan('operation');
try {
  doSomething();
} finally {
  span.end();  # Always end spans!
}

# Or use startActiveSpan which handles this
tracer.startActiveSpan('operation', (span) => {
  doSomething();
  span.end();
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Sensitive data in spans
# Bad
span.setAttribute('user.password', password);
span.setAttribute('credit_card', cardNumber);

# Good
span.setAttribute('user.id', userId);
span.setAttribute('card.last4', cardNumber.slice(-4));
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is distributed tracing?"**
> "Following a single request as it flows through multiple services. Each service adds spans (timed operations) to the trace. Trace ID propagates via headers (W3C Trace Context). Enables debugging latency, finding where errors occur, understanding service dependencies."

**Q: "What is a span?"**
> "Single operation within a trace. Has: name, start time, duration, parent span ID, tags/attributes, logs/events, status. Can be nested (parent-child). Examples: HTTP request handling, database query, cache lookup."

**Q: "What is OpenTelemetry?"**
> "Vendor-neutral observability framework. Provides APIs and SDKs for traces, metrics, logs. Auto-instrumentation for common libraries. Collector for processing and exporting. Replaces OpenTracing and OpenCensus. Export to any backend (Jaeger, Datadog, etc.)."

### Intermediate Questions

**Q: "How do you handle sampling?"**
> "Head-based: Decision at trace start (random percentage). Simple but misses interesting traces. Tail-based: Decision after trace complete. Can sample 100% errors and slow requests, X% normal. More resource-intensive but captures important traces. Use both: probabilistic head + tail for errors/latency."

**Q: "How is context propagated?"**
> "Via HTTP headers (traceparent, tracestate for W3C). Extract context on incoming requests. Inject context on outgoing requests. Async local storage maintains context through async operations. Message queues: context in message headers."

**Q: "What attributes should spans have?"**
> "Semantic conventions: http.method, http.status_code, db.system, db.statement. Business context: user.id, order.id, tenant.id. Error details: exception type, message, stack. Resource info: service.name, service.version, deployment.environment."

### Advanced Questions

**Q: "How do you optimize APM for high-traffic services?"**
> "Sampling: probabilistic head-based (1-10%) plus tail-based for errors/slow. Batching: batch span export. Async export: don't block request path. Rate limiting on collector. Reduce attributes on hot paths. Short retention for normal traces, longer for errors."

**Q: "How do you debug a slow request using APM?"**
> "1) Find slow trace by latency filter. 2) View waterfall/timeline of spans. 3) Identify longest span. 4) Check span attributes for context. 5) If database: check query, indexes. 6) If external call: check that service's trace. 7) Correlate with logs using trace ID. 8) Check concurrent traces for patterns."

**Q: "How do you instrument a custom library?"**
> "Create tracer for library. Wrap key methods with spans. Add relevant attributes (inputs, outputs, timing). Propagate context through async operations. Handle errors properly (recordException, setStatus). Consider auto-instrumentation patterns. Follow semantic conventions for attribute names."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                      APM CHECKLIST                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  INSTRUMENTATION:                                               │
│  □ Auto-instrument HTTP, DB, cache                             │
│  □ Custom spans for business operations                        │
│  □ Context propagation configured                              │
│  □ Sensitive data excluded                                     │
│                                                                 │
│  SPANS:                                                         │
│  □ Meaningful operation names                                  │
│  □ Relevant attributes (user, order, etc.)                     │
│  □ Error details captured                                      │
│  □ Spans always ended (finally block)                          │
│                                                                 │
│  SAMPLING:                                                      │
│  □ Probabilistic for normal traffic                            │
│  □ 100% for errors                                             │
│  □ 100% for slow requests                                      │
│                                                                 │
│  ANALYSIS:                                                      │
│  □ Service map configured                                      │
│  □ Dashboards for latency, errors                              │
│  □ Alerts on trace-based metrics                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

W3C TRACE CONTEXT HEADERS:
┌────────────────────────────────────────────────────────────────┐
│ traceparent: 00-<trace-id>-<span-id>-<flags>                   │
│ Example: 00-abc123def456-span789-01                            │
│                                                                │
│ tracestate: vendor1=value1,vendor2=value2                      │
│ Example: dd=s:1;t.dm:abc                                       │
└────────────────────────────────────────────────────────────────┘

APDEX SCORE:
┌────────────────────────────────────────────────────────────────┐
│ Satisfactory:  < T (e.g., 500ms)     → Score: 1.0             │
│ Tolerating:    T to 4T (500ms-2s)    → Score: 0.5             │
│ Frustrated:    > 4T (> 2s)           → Score: 0.0             │
│                                                                │
│ Apdex = (Satisfied + Tolerating/2) / Total                    │
│ Target: > 0.9 (90% satisfied/tolerating)                       │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

