# Log Analysis - Complete Guide

> **MUST REMEMBER**: Effective log analysis requires structured logs (JSON), consistent fields (timestamp, level, traceId), and powerful query tools (ELK, Loki). Index strategically - not everything. Use log levels correctly: ERROR for failures needing attention, WARN for recoverable issues, INFO for significant events, DEBUG for troubleshooting. Set up log-based alerts for critical patterns.

---

## How to Explain Like a Senior Developer

"Logs are your application's diary - but only useful if you can read them. First, structure your logs as JSON with consistent fields: timestamp, level, service name, trace ID, and the actual message/data. This lets you query programmatically. Use log aggregation (ELK, Loki, CloudWatch) to centralize logs from all services. The key is balance: log enough to debug issues but not so much you're drowning in noise or paying huge storage costs. Set up alerts on log patterns - 'ERROR' count spikes or specific exception messages. Learn your query language well - being able to quickly find relevant logs during an incident is a critical skill."

---

## Core Implementation

### Structured Logging Setup

```typescript
// logging/structured-logger.ts
import pino from 'pino';

interface LogContext {
  traceId?: string;
  spanId?: string;
  userId?: string;
  requestId?: string;
  service?: string;
}

// Create base logger
const baseLogger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label }),
    bindings: () => ({
      service: process.env.SERVICE_NAME || 'api',
      version: process.env.APP_VERSION || 'unknown',
      environment: process.env.NODE_ENV || 'development',
    }),
  },
  timestamp: () => `,"timestamp":"${new Date().toISOString()}"`,
  // Production: JSON output for log aggregation
  // Development: Pretty print
  transport: process.env.NODE_ENV === 'development' 
    ? { target: 'pino-pretty' }
    : undefined,
});

// Context-aware logger
export class Logger {
  private context: LogContext;
  
  constructor(context: LogContext = {}) {
    this.context = context;
  }
  
  child(context: LogContext): Logger {
    return new Logger({ ...this.context, ...context });
  }
  
  private log(level: string, msg: string, data?: object): void {
    const logData = {
      ...this.context,
      ...data,
      msg,
    };
    
    (baseLogger as any)[level](logData);
  }
  
  debug(msg: string, data?: object): void {
    this.log('debug', msg, data);
  }
  
  info(msg: string, data?: object): void {
    this.log('info', msg, data);
  }
  
  warn(msg: string, data?: object): void {
    this.log('warn', msg, data);
  }
  
  error(msg: string, data?: object): void {
    this.log('error', msg, data);
  }
  
  // Log with duration tracking
  timed<T>(msg: string, fn: () => T | Promise<T>, data?: object): T | Promise<T> {
    const start = Date.now();
    const result = fn();
    
    if (result instanceof Promise) {
      return result.finally(() => {
        this.info(msg, { ...data, durationMs: Date.now() - start });
      });
    }
    
    this.info(msg, { ...data, durationMs: Date.now() - start });
    return result;
  }
}

export const logger = new Logger();

// Example output:
// {
//   "level": "info",
//   "timestamp": "2026-02-02T10:30:00.000Z",
//   "service": "api",
//   "version": "1.2.3",
//   "environment": "production",
//   "traceId": "abc123",
//   "userId": "user-456",
//   "msg": "Order processed",
//   "orderId": "order-789",
//   "amount": 99.99,
//   "durationMs": 234
// }
```

### Log Level Guidelines

```typescript
// logging/log-levels.ts

/**
 * Log Level Guidelines:
 * 
 * ERROR - Something failed that needs attention
 *   - Unhandled exceptions
 *   - Failed external calls that affect the user
 *   - Data corruption or integrity issues
 *   - Security violations
 *   
 * WARN - Something unexpected but handled
 *   - Deprecated API usage
 *   - Retry attempts
 *   - Fallback to default behavior
 *   - Rate limiting triggered
 *   
 * INFO - Significant business events
 *   - Request started/completed
 *   - User actions (login, purchase)
 *   - Configuration changes
 *   - Service startup/shutdown
 *   
 * DEBUG - Detailed troubleshooting info
 *   - Function entry/exit
 *   - Variable values
 *   - Query details
 *   - Cache hits/misses
 */

import { Logger } from './structured-logger';

const logger = new Logger();

// ERROR examples
function handlePaymentError(error: Error, orderId: string): void {
  logger.error('Payment processing failed', {
    orderId,
    error: error.message,
    stack: error.stack,
    // Include context needed to debug
  });
}

// WARN examples
function handleRateLimitHit(userId: string, endpoint: string): void {
  logger.warn('Rate limit exceeded', {
    userId,
    endpoint,
    action: 'request_rejected',
  });
}

// INFO examples
function logOrderComplete(orderId: string, userId: string, amount: number): void {
  logger.info('Order completed', {
    orderId,
    userId,
    amount,
    currency: 'USD',
  });
}

// DEBUG examples
function logCacheOperation(key: string, hit: boolean): void {
  logger.debug('Cache operation', {
    key,
    hit,
    action: hit ? 'cache_hit' : 'cache_miss',
  });
}
```

### Express Request Logging Middleware

```typescript
// logging/request-logger.ts
import { Request, Response, NextFunction } from 'express';
import { Logger } from './structured-logger';
import { v4 as uuidv4 } from 'uuid';

export function requestLogger(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  // Generate request ID
  const requestId = req.headers['x-request-id'] as string || uuidv4();
  const traceId = req.headers['x-trace-id'] as string || requestId;
  
  // Attach to request for use in handlers
  (req as any).requestId = requestId;
  (req as any).logger = new Logger({
    requestId,
    traceId,
    userId: (req as any).user?.id,
  });
  
  // Add to response headers
  res.setHeader('x-request-id', requestId);
  
  const start = Date.now();
  
  // Log request start
  (req as any).logger.info('Request started', {
    method: req.method,
    path: req.path,
    query: req.query,
    userAgent: req.headers['user-agent'],
    ip: req.ip,
  });
  
  // Capture response
  const originalSend = res.send.bind(res);
  res.send = function(body: any) {
    const duration = Date.now() - start;
    
    // Log request completion
    (req as any).logger.info('Request completed', {
      method: req.method,
      path: req.path,
      statusCode: res.statusCode,
      durationMs: duration,
      responseSize: body?.length,
    });
    
    // Log slow requests as warnings
    if (duration > 5000) {
      (req as any).logger.warn('Slow request detected', {
        method: req.method,
        path: req.path,
        durationMs: duration,
      });
    }
    
    return originalSend(body);
  };
  
  next();
}
```

### ELK Stack Queries

```typescript
// logging/elk-queries.ts

/**
 * Common Elasticsearch/Kibana queries for log analysis
 */

// Find all errors in the last hour
const errorQuery = {
  query: {
    bool: {
      must: [
        { match: { level: 'error' } },
        { range: { timestamp: { gte: 'now-1h' } } },
      ],
    },
  },
  sort: [{ timestamp: { order: 'desc' } }],
};

// Find errors for a specific trace
const traceQuery = (traceId: string) => ({
  query: {
    bool: {
      must: [
        { match: { traceId } },
      ],
    },
  },
  sort: [{ timestamp: { order: 'asc' } }],
});

// Find slow requests
const slowRequestQuery = {
  query: {
    bool: {
      must: [
        { match: { msg: 'Request completed' } },
        { range: { durationMs: { gte: 5000 } } },
        { range: { timestamp: { gte: 'now-24h' } } },
      ],
    },
  },
  aggs: {
    by_path: {
      terms: { field: 'path.keyword', size: 20 },
      aggs: {
        avg_duration: { avg: { field: 'durationMs' } },
      },
    },
  },
};

// Error rate aggregation
const errorRateQuery = {
  query: {
    range: { timestamp: { gte: 'now-1h' } },
  },
  aggs: {
    by_minute: {
      date_histogram: {
        field: 'timestamp',
        fixed_interval: '1m',
      },
      aggs: {
        errors: {
          filter: { match: { level: 'error' } },
        },
        error_rate: {
          bucket_script: {
            buckets_path: {
              errors: 'errors._count',
              total: '_count',
            },
            script: 'params.errors / params.total * 100',
          },
        },
      },
    },
  },
};

// Find specific error pattern
const errorPatternQuery = (pattern: string) => ({
  query: {
    bool: {
      must: [
        { match: { level: 'error' } },
        { match_phrase: { msg: pattern } },
        { range: { timestamp: { gte: 'now-24h' } } },
      ],
    },
  },
});
```

### Loki/LogQL Queries

```typescript
// logging/loki-queries.ts

/**
 * Grafana Loki LogQL queries for log analysis
 */

const lokiQueries = {
  // All errors from api service
  errors: `{service="api"} |= "level=error"`,
  
  // Errors with JSON parsing
  errorsJson: `{service="api"} | json | level="error"`,
  
  // Filter by trace ID
  byTrace: (traceId: string) => 
    `{service="api"} | json | traceId="${traceId}"`,
  
  // Slow requests (> 5s)
  slowRequests: `{service="api"} | json | durationMs > 5000`,
  
  // Error rate (for alerting)
  errorRate: `
    sum(rate({service="api"} | json | level="error" [5m]))
    /
    sum(rate({service="api"} [5m]))
    * 100
  `,
  
  // Requests by status code
  byStatusCode: `
    sum by (statusCode) (
      count_over_time({service="api"} | json | statusCode != "" [1h])
    )
  `,
  
  // Top error messages
  topErrors: `
    topk(10, 
      sum by (msg) (
        count_over_time({service="api"} | json | level="error" [1h])
      )
    )
  `,
  
  // Specific user's activity
  userActivity: (userId: string) =>
    `{service="api"} | json | userId="${userId}" | line_format "{{.timestamp}} {{.msg}}"`,
};
```

### Log-Based Alerting

```typescript
// logging/log-alerts.ts
import { Logger } from './structured-logger';

const logger = new Logger();

interface LogAlertRule {
  name: string;
  query: string; // LogQL or similar
  threshold: number;
  window: string;
  severity: 'warning' | 'critical';
  message: string;
}

const alertRules: LogAlertRule[] = [
  {
    name: 'HighErrorRate',
    query: `sum(rate({service="api"} | json | level="error" [5m])) / sum(rate({service="api"} [5m])) * 100`,
    threshold: 5, // 5% error rate
    window: '5m',
    severity: 'critical',
    message: 'API error rate exceeds 5%',
  },
  {
    name: 'AuthenticationFailures',
    query: `sum(rate({service="api"} |= "authentication failed" [5m]))`,
    threshold: 100, // 100 failures per 5 min
    window: '5m',
    severity: 'warning',
    message: 'High rate of authentication failures',
  },
  {
    name: 'DatabaseConnectionErrors',
    query: `count_over_time({service="api"} |= "database connection" |= "error" [1m])`,
    threshold: 5,
    window: '1m',
    severity: 'critical',
    message: 'Database connection errors detected',
  },
  {
    name: 'PaymentProcessingErrors',
    query: `count_over_time({service="api"} | json | msg="Payment processing failed" [5m])`,
    threshold: 1,
    window: '5m',
    severity: 'critical',
    message: 'Payment processing failures detected',
  },
];

// Prometheus alerting rules (YAML format for reference)
const prometheusAlerts = `
groups:
  - name: log-based-alerts
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate({service="api"} | json | level="error" [5m]))
          /
          sum(rate({service="api"} [5m]))
          * 100 > 5
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate in API"
          description: "Error rate is {{ $value }}%"
          
      - alert: NoLogs
        expr: |
          absent(count_over_time({service="api"} [5m]))
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "No logs from API"
          description: "API service may be down"
`;
```

### Log Retention and Cost Management

```typescript
// logging/retention.ts

/**
 * Log Retention Strategy:
 * 
 * Hot Storage (fast queries, expensive):
 *   - Last 7 days of all logs
 *   - Indexed and searchable
 *   
 * Warm Storage (slower queries, cheaper):
 *   - 8-30 days
 *   - May have reduced indexing
 *   
 * Cold Storage (archive, very cheap):
 *   - 31-90 days
 *   - Compressed, minimal indexing
 *   
 * Glacier/Archive (compliance):
 *   - 90+ days
 *   - For audit requirements
 */

interface RetentionPolicy {
  tier: 'hot' | 'warm' | 'cold' | 'archive';
  maxAgeDays: number;
  indexFields: string[];
  compression: boolean;
  estimatedCostPerGBMonth: number;
}

const retentionPolicies: RetentionPolicy[] = [
  {
    tier: 'hot',
    maxAgeDays: 7,
    indexFields: ['timestamp', 'level', 'service', 'traceId', 'userId', 'msg'],
    compression: false,
    estimatedCostPerGBMonth: 0.50,
  },
  {
    tier: 'warm',
    maxAgeDays: 30,
    indexFields: ['timestamp', 'level', 'service', 'traceId'],
    compression: true,
    estimatedCostPerGBMonth: 0.10,
  },
  {
    tier: 'cold',
    maxAgeDays: 90,
    indexFields: ['timestamp', 'level'],
    compression: true,
    estimatedCostPerGBMonth: 0.03,
  },
  {
    tier: 'archive',
    maxAgeDays: 365,
    indexFields: [],
    compression: true,
    estimatedCostPerGBMonth: 0.01,
  },
];

// What to log and what NOT to log
const loggingGuidelines = {
  alwaysLog: [
    'Errors and exceptions',
    'Security events (login, permission denied)',
    'Business transactions (orders, payments)',
    'System events (startup, shutdown, config changes)',
    'External service calls (with timing)',
  ],
  
  conditionalLog: [
    'Debug information (only when DEBUG level enabled)',
    'Request/response bodies (careful with PII)',
    'Cache operations (useful for debugging, verbose)',
  ],
  
  neverLog: [
    'Passwords or secrets',
    'Full credit card numbers',
    'Personal health information',
    'Raw PII without redaction',
    'High-frequency events without sampling',
  ],
};
```

---

## Real-World Scenarios

### Scenario 1: Debugging a Production Issue

```typescript
// Investigation workflow using logs

/*
1. Start with the symptom:
   Query: {service="api"} | json | level="error" | last 1h

2. Find affected trace ID from error:
   Result: traceId="abc-123-xyz"

3. Get full trace timeline:
   Query: {service=~".+"} | json | traceId="abc-123-xyz"

4. Identify the root cause service/operation:
   Found: Database timeout in user-service

5. Get more context around the failure:
   Query: {service="user-service"} | json | traceId="abc-123-xyz"
*/

// Automated investigation helper
async function investigateError(errorLog: {
  traceId: string;
  timestamp: Date;
  service: string;
}): Promise<object> {
  // Query logs before and after the error
  const timeRange = {
    start: new Date(errorLog.timestamp.getTime() - 60000), // 1 min before
    end: new Date(errorLog.timestamp.getTime() + 60000),   // 1 min after
  };
  
  // Get all logs for this trace
  const traceLogs = await queryLogs({
    filter: `traceId="${errorLog.traceId}"`,
    start: timeRange.start,
    end: timeRange.end,
  });
  
  // Get similar errors
  const similarErrors = await queryLogs({
    filter: `level="error" AND service="${errorLog.service}"`,
    start: new Date(Date.now() - 3600000), // last hour
  });
  
  return {
    traceTimeline: traceLogs,
    errorCount: similarErrors.length,
    affectedServices: [...new Set(traceLogs.map((l: any) => l.service))],
  };
}

async function queryLogs(params: { filter: string; start: Date; end?: Date }): Promise<any[]> {
  return [];
}
```

### Scenario 2: Setting Up Log Dashboards

```typescript
// Dashboard panels configuration
const dashboardPanels = [
  {
    title: 'Request Rate',
    query: `sum(rate({service="api"} | json | msg="Request completed" [1m]))`,
    visualization: 'time_series',
  },
  {
    title: 'Error Rate %',
    query: `
      sum(rate({service="api"} | json | level="error" [5m]))
      /
      sum(rate({service="api"} [5m]))
      * 100
    `,
    visualization: 'gauge',
    thresholds: [
      { value: 0, color: 'green' },
      { value: 1, color: 'yellow' },
      { value: 5, color: 'red' },
    ],
  },
  {
    title: 'P95 Latency',
    query: `quantile_over_time(0.95, {service="api"} | json | unwrap durationMs [5m])`,
    visualization: 'time_series',
  },
  {
    title: 'Top Errors',
    query: `topk(5, sum by (msg) (count_over_time({service="api"} | json | level="error" [1h])))`,
    visualization: 'table',
  },
  {
    title: 'Logs Stream',
    query: `{service="api"} | json | level=~"error|warn"`,
    visualization: 'logs',
  },
];
```

---

## Common Pitfalls

### 1. Unstructured Logs

```typescript
// ❌ BAD: Unstructured, hard to query
console.log(`User ${userId} created order ${orderId} for $${amount}`);

// ✅ GOOD: Structured JSON
logger.info('Order created', { userId, orderId, amount });
```

### 2. Missing Correlation IDs

```typescript
// ❌ BAD: No way to correlate logs
logger.info('Processing started');
// ... in another service ...
logger.info('Processing completed');

// ✅ GOOD: Trace ID connects all logs
logger.info('Processing started', { traceId });
// ... in another service ...
logger.info('Processing completed', { traceId });
```

### 3. Logging Sensitive Data

```typescript
// ❌ BAD: Logs password
logger.info('User login', { email, password });

// ✅ GOOD: Redact sensitive fields
logger.info('User login', { 
  email, 
  passwordProvided: !!password,
  // Or use a redaction library
});
```

### 4. Over-Logging

```typescript
// ❌ BAD: Logging in a tight loop
for (const item of items) {
  logger.debug('Processing item', { itemId: item.id }); // Millions of logs!
  process(item);
}

// ✅ GOOD: Log summary
logger.info('Processing batch', { itemCount: items.length });
for (const item of items) {
  process(item);
}
logger.info('Batch complete', { itemCount: items.length, durationMs: Date.now() - start });

function process(item: any): void {}
```

---

## Interview Questions

### Q1: What makes a good log message?

**A:** Structured format (JSON), consistent fields (timestamp, level, service, traceId), meaningful message describing what happened, relevant context data (IDs, durations, counts), appropriate log level, and no sensitive data. Should answer: what happened, when, where, and enough context to debug.

### Q2: How do you debug an issue using logs?

**A:** 1) Start with error logs around the incident time, 2) Find a trace/correlation ID from the error, 3) Query all logs with that trace ID across services, 4) Build a timeline of events, 5) Identify where things went wrong, 6) Look for patterns - is this one occurrence or many?

### Q3: How do you manage log storage costs?

**A:** Tiered retention (hot/warm/cold), sampling high-volume logs, not logging in tight loops, appropriate log levels (DEBUG off in production unless needed), indexing only necessary fields, compression for older logs, and regular review of what's being logged.

### Q4: What should you alert on from logs?

**A:** Error rate exceeding threshold, specific critical error messages (payment failures, security events), absence of expected logs (service might be down), unusual patterns (sudden spike in 404s), and security-related events (failed auth attempts).

---

## Quick Reference Checklist

### Logging Best Practices
- [ ] Use structured JSON format
- [ ] Include trace/correlation IDs
- [ ] Use appropriate log levels
- [ ] Don't log sensitive data
- [ ] Include timing information

### Log Aggregation
- [ ] Centralize logs from all services
- [ ] Set up retention policies
- [ ] Index important fields
- [ ] Create useful dashboards

### Alerting
- [ ] Alert on error rate thresholds
- [ ] Alert on critical error patterns
- [ ] Alert on missing logs
- [ ] Set up escalation paths

### Cost Management
- [ ] Review what you're logging
- [ ] Use tiered storage
- [ ] Sample high-volume logs
- [ ] Compress older logs

---

*Last updated: February 2026*

