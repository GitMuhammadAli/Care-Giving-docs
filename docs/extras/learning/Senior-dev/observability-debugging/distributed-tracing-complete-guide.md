# Distributed Tracing - Complete Guide

> **MUST REMEMBER**: Distributed tracing tracks requests as they flow through multiple services, creating a "trace" of connected "spans". Each span represents a unit of work (API call, database query). Use correlation IDs (trace IDs) to link all operations from a single request. OpenTelemetry is the standard; Jaeger and Zipkin are popular backends.

---

## How to Explain Like a Senior Developer

"In a monolith, debugging is easy - you look at one log file. In microservices, a single user request might touch 10 services. Distributed tracing solves this by assigning a unique trace ID to each request and propagating it through all services. Each service creates spans showing what it did and how long it took. When something fails, you can see the entire request path, identify which service is slow, and understand the cascade of calls. OpenTelemetry standardizes this - you instrument once and can send data to any backend. The key insight is that tracing shows you WHY things are slow, not just THAT they're slow."

---

## Core Implementation

### OpenTelemetry Setup for Node.js

```typescript
// tracing.ts
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { OTLPMetricExporter } from '@opentelemetry/exporter-metrics-otlp-http';
import { PeriodicExportingMetricReader } from '@opentelemetry/sdk-metrics';
import { Resource } from '@opentelemetry/resources';
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions';
import { diag, DiagConsoleLogger, DiagLogLevel } from '@opentelemetry/api';

// Enable debug logging (optional)
diag.setLogger(new DiagConsoleLogger(), DiagLogLevel.INFO);

// Configure the SDK
const sdk = new NodeSDK({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: 'user-service',
    [SemanticResourceAttributes.SERVICE_VERSION]: '1.0.0',
    [SemanticResourceAttributes.DEPLOYMENT_ENVIRONMENT]: process.env.NODE_ENV || 'development',
  }),
  
  // Trace exporter (to Jaeger/OTLP collector)
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT || 'http://localhost:4318/v1/traces',
  }),
  
  // Metric exporter
  metricReader: new PeriodicExportingMetricReader({
    exporter: new OTLPMetricExporter({
      url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT || 'http://localhost:4318/v1/metrics',
    }),
    exportIntervalMillis: 30000,
  }),
  
  // Auto-instrumentation for common libraries
  instrumentations: [
    getNodeAutoInstrumentations({
      // Customize instrumentation
      '@opentelemetry/instrumentation-http': {
        ignoreIncomingPaths: ['/health', '/ready'],
      },
      '@opentelemetry/instrumentation-fs': {
        enabled: false, // Disable noisy file system instrumentation
      },
    }),
  ],
});

// Start the SDK
sdk.start();

// Graceful shutdown
process.on('SIGTERM', () => {
  sdk.shutdown()
    .then(() => console.log('Tracing terminated'))
    .catch((error) => console.error('Error terminating tracing', error))
    .finally(() => process.exit(0));
});

export { sdk };
```

### Manual Span Creation

```typescript
// manual-tracing.ts
import { trace, SpanStatusCode, SpanKind, context } from '@opentelemetry/api';

// Get tracer for this module
const tracer = trace.getTracer('user-service', '1.0.0');

// Create a simple span
async function processOrder(orderId: string): Promise<void> {
  const span = tracer.startSpan('processOrder', {
    kind: SpanKind.INTERNAL,
    attributes: {
      'order.id': orderId,
    },
  });
  
  try {
    // Add events (logs within the span)
    span.addEvent('Starting order validation');
    
    await validateOrder(orderId);
    
    span.addEvent('Order validated, processing payment');
    
    await processPayment(orderId);
    
    span.addEvent('Payment processed, fulfilling order');
    
    await fulfillOrder(orderId);
    
    span.setStatus({ code: SpanStatusCode.OK });
  } catch (error) {
    span.setStatus({
      code: SpanStatusCode.ERROR,
      message: (error as Error).message,
    });
    span.recordException(error as Error);
    throw error;
  } finally {
    span.end();
  }
}

// Nested spans with context propagation
async function validateOrder(orderId: string): Promise<void> {
  return tracer.startActiveSpan('validateOrder', async (span) => {
    span.setAttribute('order.id', orderId);
    
    try {
      // Check inventory
      await tracer.startActiveSpan('checkInventory', async (inventorySpan) => {
        inventorySpan.setAttribute('check.type', 'inventory');
        await checkInventory(orderId);
        inventorySpan.end();
      });
      
      // Validate payment method
      await tracer.startActiveSpan('validatePaymentMethod', async (paymentSpan) => {
        paymentSpan.setAttribute('check.type', 'payment');
        await validatePaymentMethod(orderId);
        paymentSpan.end();
      });
      
      span.setStatus({ code: SpanStatusCode.OK });
    } catch (error) {
      span.setStatus({ code: SpanStatusCode.ERROR });
      span.recordException(error as Error);
      throw error;
    } finally {
      span.end();
    }
  });
}

async function checkInventory(orderId: string): Promise<void> {
  // Simulated check
}

async function validatePaymentMethod(orderId: string): Promise<void> {
  // Simulated validation
}

async function processPayment(orderId: string): Promise<void> {
  // Simulated payment
}

async function fulfillOrder(orderId: string): Promise<void> {
  // Simulated fulfillment
}
```

### Context Propagation Between Services

```typescript
// context-propagation.ts
import { context, propagation, trace, SpanKind } from '@opentelemetry/api';
import axios from 'axios';

const tracer = trace.getTracer('api-gateway');

// Extracting context from incoming request (Express middleware)
import { Request, Response, NextFunction } from 'express';

function tracingMiddleware(req: Request, res: Response, next: NextFunction): void {
  // Extract context from incoming headers
  const extractedContext = propagation.extract(context.active(), req.headers);
  
  // Create span within extracted context
  const span = tracer.startSpan(
    `${req.method} ${req.path}`,
    {
      kind: SpanKind.SERVER,
      attributes: {
        'http.method': req.method,
        'http.url': req.url,
        'http.route': req.route?.path,
        'http.user_agent': req.headers['user-agent'],
      },
    },
    extractedContext
  );
  
  // Store span in request for later use
  (req as any).span = span;
  
  // End span when response finishes
  res.on('finish', () => {
    span.setAttribute('http.status_code', res.statusCode);
    
    if (res.statusCode >= 400) {
      span.setStatus({ code: SpanStatusCode.ERROR });
    }
    
    span.end();
  });
  
  // Continue with span context active
  context.with(trace.setSpan(context.active(), span), () => {
    next();
  });
}

// Injecting context into outgoing requests
async function callDownstreamService(endpoint: string, data: any): Promise<any> {
  return tracer.startActiveSpan('callDownstreamService', {
    kind: SpanKind.CLIENT,
    attributes: {
      'http.url': endpoint,
      'http.method': 'POST',
    },
  }, async (span) => {
    try {
      // Inject current context into headers
      const headers: Record<string, string> = {};
      propagation.inject(context.active(), headers);
      
      const response = await axios.post(endpoint, data, { headers });
      
      span.setAttribute('http.status_code', response.status);
      span.setStatus({ code: SpanStatusCode.OK });
      
      return response.data;
    } catch (error) {
      span.setStatus({ code: SpanStatusCode.ERROR });
      span.recordException(error as Error);
      throw error;
    } finally {
      span.end();
    }
  });
}

import { SpanStatusCode } from '@opentelemetry/api';
```

### Database Query Tracing

```typescript
// database-tracing.ts
import { trace, SpanKind, SpanStatusCode } from '@opentelemetry/api';
import { Pool } from 'pg';

const tracer = trace.getTracer('database');

// Wrap database client with tracing
class TracedDatabaseClient {
  constructor(private pool: Pool) {}
  
  async query<T>(sql: string, params?: any[]): Promise<T[]> {
    return tracer.startActiveSpan('database.query', {
      kind: SpanKind.CLIENT,
      attributes: {
        'db.system': 'postgresql',
        'db.statement': this.sanitizeQuery(sql),
        'db.operation': this.extractOperation(sql),
      },
    }, async (span) => {
      const startTime = Date.now();
      
      try {
        const result = await this.pool.query(sql, params);
        
        span.setAttribute('db.rows_affected', result.rowCount || 0);
        span.setStatus({ code: SpanStatusCode.OK });
        
        return result.rows;
      } catch (error) {
        span.setStatus({
          code: SpanStatusCode.ERROR,
          message: (error as Error).message,
        });
        span.recordException(error as Error);
        throw error;
      } finally {
        span.setAttribute('db.duration_ms', Date.now() - startTime);
        span.end();
      }
    });
  }
  
  // Remove sensitive data from queries for logging
  private sanitizeQuery(sql: string): string {
    // Replace parameter values with placeholders
    return sql.replace(/\$\d+/g, '?');
  }
  
  private extractOperation(sql: string): string {
    const match = sql.trim().match(/^(\w+)/i);
    return match ? match[1].toUpperCase() : 'UNKNOWN';
  }
}
```

### Custom Span Attributes and Events

```typescript
// custom-attributes.ts
import { trace, SpanStatusCode } from '@opentelemetry/api';

const tracer = trace.getTracer('payment-service');

interface PaymentDetails {
  orderId: string;
  amount: number;
  currency: string;
  customerId: string;
  paymentMethod: string;
}

async function processPayment(payment: PaymentDetails): Promise<string> {
  return tracer.startActiveSpan('payment.process', async (span) => {
    // Add business-relevant attributes
    span.setAttributes({
      'payment.order_id': payment.orderId,
      'payment.amount': payment.amount,
      'payment.currency': payment.currency,
      'payment.method': payment.paymentMethod,
      'customer.id': payment.customerId,
    });
    
    try {
      // Step 1: Validate payment
      span.addEvent('payment.validation.started');
      const validationResult = await validatePayment(payment);
      span.addEvent('payment.validation.completed', {
        'validation.result': validationResult ? 'success' : 'failed',
      });
      
      if (!validationResult) {
        span.setStatus({ code: SpanStatusCode.ERROR, message: 'Validation failed' });
        throw new Error('Payment validation failed');
      }
      
      // Step 2: Process with payment provider
      span.addEvent('payment.provider.request.started');
      const transactionId = await chargePaymentProvider(payment);
      span.addEvent('payment.provider.request.completed', {
        'transaction.id': transactionId,
      });
      
      // Step 3: Record transaction
      span.addEvent('payment.recording.started');
      await recordTransaction(transactionId, payment);
      span.addEvent('payment.recording.completed');
      
      span.setAttribute('payment.transaction_id', transactionId);
      span.setStatus({ code: SpanStatusCode.OK });
      
      return transactionId;
    } catch (error) {
      span.setStatus({
        code: SpanStatusCode.ERROR,
        message: (error as Error).message,
      });
      span.recordException(error as Error);
      
      // Add error context
      span.setAttribute('error.type', (error as Error).name);
      
      throw error;
    } finally {
      span.end();
    }
  });
}

async function validatePayment(payment: PaymentDetails): Promise<boolean> {
  return true;
}

async function chargePaymentProvider(payment: PaymentDetails): Promise<string> {
  return 'txn_123';
}

async function recordTransaction(transactionId: string, payment: PaymentDetails): Promise<void> {
  // Record to database
}
```

### Sampling Strategies

```typescript
// sampling.ts
import { 
  ParentBasedSampler, 
  TraceIdRatioBasedSampler,
  AlwaysOnSampler,
  AlwaysOffSampler,
  Sampler,
  SamplingResult,
  SamplingDecision,
} from '@opentelemetry/sdk-trace-base';
import { Context, SpanKind, Attributes, Link } from '@opentelemetry/api';

// Rate-based sampling (sample 10% of traces)
const ratioSampler = new TraceIdRatioBasedSampler(0.1);

// Parent-based sampling (respect parent's sampling decision)
const parentBasedSampler = new ParentBasedSampler({
  root: new TraceIdRatioBasedSampler(0.1),
  remoteParentSampled: new AlwaysOnSampler(),
  remoteParentNotSampled: new AlwaysOffSampler(),
  localParentSampled: new AlwaysOnSampler(),
  localParentNotSampled: new AlwaysOffSampler(),
});

// Custom sampler (e.g., always sample errors and slow requests)
class CustomSampler implements Sampler {
  shouldSample(
    context: Context,
    traceId: string,
    spanName: string,
    spanKind: SpanKind,
    attributes: Attributes,
    links: Link[]
  ): SamplingResult {
    // Always sample error spans
    if (attributes['error'] === true) {
      return { decision: SamplingDecision.RECORD_AND_SAMPLED };
    }
    
    // Always sample slow endpoints
    if (spanName.includes('/api/slow')) {
      return { decision: SamplingDecision.RECORD_AND_SAMPLED };
    }
    
    // Always sample specific users (for debugging)
    const userId = attributes['user.id'] as string;
    if (userId && this.isDebugUser(userId)) {
      return { decision: SamplingDecision.RECORD_AND_SAMPLED };
    }
    
    // Default: sample 10%
    const shouldSample = Math.random() < 0.1;
    return {
      decision: shouldSample 
        ? SamplingDecision.RECORD_AND_SAMPLED 
        : SamplingDecision.NOT_RECORD,
    };
  }
  
  private isDebugUser(userId: string): boolean {
    const debugUsers = process.env.DEBUG_USER_IDS?.split(',') || [];
    return debugUsers.includes(userId);
  }
  
  toString(): string {
    return 'CustomSampler';
  }
}
```

---

## Real-World Scenarios

### Scenario 1: E-commerce Order Flow Tracing

```typescript
// order-tracing.ts
import { trace, context, SpanKind, SpanStatusCode } from '@opentelemetry/api';

const tracer = trace.getTracer('order-service');

interface Order {
  id: string;
  customerId: string;
  items: Array<{ productId: string; quantity: number }>;
  total: number;
}

async function handleOrderCreation(orderData: any): Promise<Order> {
  return tracer.startActiveSpan('order.create', {
    kind: SpanKind.SERVER,
    attributes: {
      'order.customer_id': orderData.customerId,
      'order.items_count': orderData.items.length,
    },
  }, async (span) => {
    try {
      // Step 1: Validate inventory across all items
      const inventorySpan = tracer.startSpan('order.validate_inventory', {
        kind: SpanKind.INTERNAL,
      });
      
      for (const item of orderData.items) {
        await tracer.startActiveSpan(`inventory.check.${item.productId}`, async (itemSpan) => {
          itemSpan.setAttribute('product.id', item.productId);
          itemSpan.setAttribute('quantity.requested', item.quantity);
          
          const available = await checkInventory(item.productId, item.quantity);
          itemSpan.setAttribute('quantity.available', available);
          
          if (available < item.quantity) {
            itemSpan.setStatus({ code: SpanStatusCode.ERROR, message: 'Insufficient inventory' });
          }
          
          itemSpan.end();
        });
      }
      
      inventorySpan.end();
      
      // Step 2: Calculate pricing
      await tracer.startActiveSpan('order.calculate_pricing', async (pricingSpan) => {
        const pricing = await calculatePricing(orderData);
        pricingSpan.setAttribute('order.subtotal', pricing.subtotal);
        pricingSpan.setAttribute('order.tax', pricing.tax);
        pricingSpan.setAttribute('order.total', pricing.total);
        pricingSpan.end();
      });
      
      // Step 3: Process payment
      await tracer.startActiveSpan('order.payment', {
        kind: SpanKind.CLIENT,
      }, async (paymentSpan) => {
        paymentSpan.setAttribute('payment.provider', 'stripe');
        const transactionId = await processPayment(orderData);
        paymentSpan.setAttribute('payment.transaction_id', transactionId);
        paymentSpan.end();
      });
      
      // Step 4: Create order record
      const order = await tracer.startActiveSpan('order.persist', async (persistSpan) => {
        const created = await createOrderRecord(orderData);
        persistSpan.setAttribute('order.id', created.id);
        persistSpan.end();
        return created;
      });
      
      // Step 5: Trigger fulfillment (async)
      await tracer.startActiveSpan('order.trigger_fulfillment', {
        kind: SpanKind.PRODUCER,
      }, async (fulfillSpan) => {
        await publishFulfillmentEvent(order);
        fulfillSpan.end();
      });
      
      span.setAttribute('order.id', order.id);
      span.setStatus({ code: SpanStatusCode.OK });
      
      return order;
    } catch (error) {
      span.setStatus({ code: SpanStatusCode.ERROR });
      span.recordException(error as Error);
      throw error;
    } finally {
      span.end();
    }
  });
}

// Mock implementations
async function checkInventory(productId: string, quantity: number): Promise<number> {
  return 100;
}

async function calculatePricing(orderData: any): Promise<{ subtotal: number; tax: number; total: number }> {
  return { subtotal: 100, tax: 10, total: 110 };
}

async function processPayment(orderData: any): Promise<string> {
  return 'txn_123';
}

async function createOrderRecord(orderData: any): Promise<Order> {
  return { id: 'order_123', customerId: orderData.customerId, items: orderData.items, total: 110 };
}

async function publishFulfillmentEvent(order: Order): Promise<void> {
  // Publish to queue
}
```

### Scenario 2: Correlating Logs with Traces

```typescript
// log-correlation.ts
import { trace, context } from '@opentelemetry/api';
import pino from 'pino';

// Create logger that automatically includes trace context
const logger = pino({
  mixin() {
    const span = trace.getSpan(context.active());
    if (span) {
      const spanContext = span.spanContext();
      return {
        trace_id: spanContext.traceId,
        span_id: spanContext.spanId,
        trace_flags: spanContext.traceFlags,
      };
    }
    return {};
  },
  formatters: {
    level: (label) => ({ level: label }),
  },
});

// Usage - logs automatically include trace context
async function processRequest(req: any): Promise<void> {
  logger.info({ userId: req.user.id }, 'Processing request');
  
  try {
    await doSomething();
    logger.info('Request processed successfully');
  } catch (error) {
    logger.error({ error }, 'Request processing failed');
    throw error;
  }
}

async function doSomething(): Promise<void> {
  // Work
}

// In your log aggregation system (e.g., Elasticsearch), you can now
// search for all logs with a specific trace_id to see everything
// that happened during that request
```

---

## Common Pitfalls

### 1. Not Propagating Context

```typescript
// ❌ BAD: Context lost in async operations
async function processItems(items: string[]): Promise<void> {
  // Promise.all loses trace context!
  await Promise.all(items.map(item => processItem(item)));
}

// ✅ GOOD: Preserve context
import { context } from '@opentelemetry/api';

async function processItems(items: string[]): Promise<void> {
  const currentContext = context.active();
  
  await Promise.all(items.map(item => 
    context.with(currentContext, () => processItem(item))
  ));
}

async function processItem(item: string): Promise<void> {
  // Processing
}
```

### 2. Creating Too Many Spans

```typescript
// ❌ BAD: Span for every tiny operation
for (const item of items) {
  const span = tracer.startSpan('processItem');
  processItem(item); // 1ms operation
  span.end();
}

// ✅ GOOD: One span for the batch
const span = tracer.startSpan('processItems', {
  attributes: { 'items.count': items.length },
});
for (const item of items) {
  processItem(item);
}
span.end();

const tracer = trace.getTracer('example');
import { trace } from '@opentelemetry/api';
function processItem(item: any): void {}
const items: any[] = [];
```

### 3. Not Ending Spans

```typescript
// ❌ BAD: Span never ended on error
async function doWork(): Promise<void> {
  const span = tracer.startSpan('work');
  await riskyOperation(); // Throws - span never ends!
  span.end();
}

// ✅ GOOD: Always end spans
async function doWork(): Promise<void> {
  const span = tracer.startSpan('work');
  try {
    await riskyOperation();
  } finally {
    span.end(); // Always called
  }
}

// Or use startActiveSpan which handles this
async function doWorkBetter(): Promise<void> {
  await tracer.startActiveSpan('work', async (span) => {
    try {
      await riskyOperation();
    } finally {
      span.end();
    }
  });
}

async function riskyOperation(): Promise<void> {}
```

---

## Interview Questions

### Q1: What's the difference between traces, spans, and logs?

**A:** **Traces** are end-to-end records of requests across services. **Spans** are individual units of work within a trace (like function calls), with start/end times, attributes, and parent-child relationships. **Logs** are timestamped messages. Traces provide the "big picture" of request flow; spans provide timing and relationships; logs provide detailed context. Modern observability correlates all three using trace IDs.

### Q2: How does context propagation work across services?

**A:** Context propagation passes trace information (trace ID, span ID, sampling decision) between services via headers. W3C Trace Context is the standard format. When Service A calls Service B, it injects context into HTTP headers. Service B extracts this context and creates child spans linked to the parent. This creates a connected trace across all services.

### Q3: What are the performance implications of distributed tracing?

**A:** Tracing adds overhead: span creation (~1μs), context propagation, and data export. Mitigate with: sampling (only trace a percentage), async export (don't block requests), batching (send spans in groups), and avoiding excessive spans. Production typically samples 1-10% of requests, always sampling errors and slow requests.

### Q4: How do you debug a slow request using distributed tracing?

**A:** 1) Find the trace by request ID or time range, 2) View the waterfall diagram showing all spans, 3) Identify the slowest spans, 4) Look at span attributes and events for context, 5) Check for gaps between spans (network latency), 6) Correlate with logs using the trace ID, 7) Look at child spans to find the root cause.

---

## Quick Reference Checklist

### Setup
- [ ] Install OpenTelemetry SDK
- [ ] Configure trace exporter (Jaeger/OTLP)
- [ ] Enable auto-instrumentation
- [ ] Set service name and version

### Instrumentation
- [ ] Propagate context between services
- [ ] Add business-relevant attributes
- [ ] Create spans for significant operations
- [ ] Always end spans (use try/finally)

### Best Practices
- [ ] Use sampling in production
- [ ] Correlate logs with trace IDs
- [ ] Don't over-instrument
- [ ] Sanitize sensitive data

### Monitoring
- [ ] Set up trace visualization (Jaeger UI)
- [ ] Create alerts for error rates
- [ ] Monitor trace export health
- [ ] Track instrumentation coverage

---

*Last updated: February 2026*

