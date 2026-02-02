# Structured Logging - Complete Guide

> **MUST REMEMBER**: Structured logging outputs logs as JSON objects instead of plain text, making them parseable by machines. Include context (user ID, request ID, trace ID), use consistent log levels, and aggregate logs centrally. Key fields: timestamp, level, message, service, trace_id, and business context.

---

## How to Explain Like a Senior Developer

"Stop writing logs like console.log('User ' + userId + ' did something'). That's impossible to parse at scale. Structured logging means every log is a JSON object with consistent fields - timestamp, level, message, plus any context you need. When you have 100 services producing millions of logs, you need to search 'show me all errors for user X in the last hour'. That's only possible with structured data. Use Pino or Winston, always include request/trace IDs, and set up log aggregation (ELK, Loki) from day one. The investment pays off the first time you debug a production issue."

---

## Core Implementation

### Pino Logger Setup

```typescript
// logger/index.ts
import pino, { Logger, LoggerOptions } from 'pino';

interface LogContext {
  requestId?: string;
  userId?: string;
  traceId?: string;
  spanId?: string;
  [key: string]: unknown;
}

// Base logger configuration
const baseOptions: LoggerOptions = {
  level: process.env.LOG_LEVEL || 'info',
  
  // Custom log levels
  customLevels: {
    audit: 35, // Between info (30) and warn (40)
  },
  
  // Base context included in every log
  base: {
    service: process.env.SERVICE_NAME || 'unknown-service',
    version: process.env.SERVICE_VERSION || '0.0.0',
    environment: process.env.NODE_ENV || 'development',
  },
  
  // Timestamp format
  timestamp: pino.stdTimeFunctions.isoTime,
  
  // Format log level as string
  formatters: {
    level: (label) => ({ level: label }),
  },
  
  // Redact sensitive fields
  redact: {
    paths: [
      'password',
      'token',
      'authorization',
      'creditCard',
      '*.password',
      '*.token',
      'req.headers.authorization',
    ],
    censor: '[REDACTED]',
  },
};

// Development: pretty print
// Production: JSON output
const devOptions: LoggerOptions = {
  ...baseOptions,
  transport: {
    target: 'pino-pretty',
    options: {
      colorize: true,
      translateTime: 'SYS:standard',
      ignore: 'pid,hostname',
    },
  },
};

const prodOptions: LoggerOptions = {
  ...baseOptions,
};

// Create logger instance
export const logger: Logger = pino(
  process.env.NODE_ENV === 'production' ? prodOptions : devOptions
);

// Child logger factory with context
export function createLogger(context: LogContext): Logger {
  return logger.child(context);
}

// Request-scoped logger
export function createRequestLogger(req: {
  id: string;
  user?: { id: string };
  headers?: { 'x-trace-id'?: string };
}): Logger {
  return logger.child({
    requestId: req.id,
    userId: req.user?.id,
    traceId: req.headers?.['x-trace-id'],
  });
}
```

### Express Middleware for Request Logging

```typescript
// middleware/request-logger.ts
import { Request, Response, NextFunction } from 'express';
import { logger, createRequestLogger } from '../logger';
import { v4 as uuidv4 } from 'uuid';

// Extend Express Request type
declare global {
  namespace Express {
    interface Request {
      id: string;
      log: typeof logger;
      startTime: number;
    }
  }
}

export function requestLoggingMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  // Assign request ID
  req.id = (req.headers['x-request-id'] as string) || uuidv4();
  req.startTime = Date.now();
  
  // Create request-scoped logger
  req.log = createRequestLogger({
    id: req.id,
    user: (req as any).user,
    headers: req.headers as any,
  });
  
  // Set response header
  res.setHeader('x-request-id', req.id);
  
  // Log incoming request
  req.log.info({
    msg: 'Request started',
    http: {
      method: req.method,
      url: req.originalUrl,
      userAgent: req.headers['user-agent'],
      ip: req.ip,
    },
  });
  
  // Log response when finished
  res.on('finish', () => {
    const duration = Date.now() - req.startTime;
    const logData = {
      msg: 'Request completed',
      http: {
        method: req.method,
        url: req.originalUrl,
        statusCode: res.statusCode,
        duration,
      },
    };
    
    if (res.statusCode >= 500) {
      req.log.error(logData);
    } else if (res.statusCode >= 400) {
      req.log.warn(logData);
    } else {
      req.log.info(logData);
    }
  });
  
  next();
}
```

### Log Levels and When to Use Them

```typescript
// log-levels.ts
import { logger } from './logger';

/**
 * Log Level Guidelines:
 * 
 * FATAL (60): Application cannot continue - immediate shutdown
 * ERROR (50): Operation failed, needs attention
 * WARN  (40): Unexpected but recoverable, potential issues
 * INFO  (30): Normal operations, key business events
 * DEBUG (20): Detailed diagnostic information
 * TRACE (10): Very detailed, step-by-step execution
 */

// FATAL: Unrecoverable errors
function handleFatalError(error: Error): void {
  logger.fatal({ error }, 'Application cannot continue, shutting down');
  process.exit(1);
}

// ERROR: Operation failures that need attention
async function handlePayment(paymentId: string): Promise<void> {
  try {
    await processPayment(paymentId);
  } catch (error) {
    logger.error({
      error,
      paymentId,
      msg: 'Payment processing failed',
    });
    throw error;
  }
}

// WARN: Potential issues, degraded functionality
function handleRateLimitNear(userId: string, current: number, limit: number): void {
  if (current > limit * 0.8) {
    logger.warn({
      userId,
      current,
      limit,
      msg: 'User approaching rate limit',
    });
  }
}

// INFO: Normal operations, business events
function logUserRegistered(userId: string, email: string): void {
  logger.info({
    userId,
    email,
    event: 'user.registered',
    msg: 'New user registered',
  });
}

// DEBUG: Diagnostic information
function logQueryExecution(query: string, params: unknown[], duration: number): void {
  logger.debug({
    query,
    params,
    duration,
    msg: 'Database query executed',
  });
}

// TRACE: Very detailed execution flow
function logCacheOperation(operation: string, key: string, hit: boolean): void {
  logger.trace({
    operation,
    key,
    hit,
    msg: 'Cache operation',
  });
}

async function processPayment(paymentId: string): Promise<void> {}
```

### Contextual Logging with AsyncLocalStorage

```typescript
// context/async-context.ts
import { AsyncLocalStorage } from 'async_hooks';
import { Logger } from 'pino';
import { logger as baseLogger } from '../logger';

interface RequestContext {
  requestId: string;
  userId?: string;
  traceId?: string;
  logger: Logger;
}

// Store for request context
const asyncLocalStorage = new AsyncLocalStorage<RequestContext>();

// Get current context
export function getContext(): RequestContext | undefined {
  return asyncLocalStorage.getStore();
}

// Get logger from context (or base logger)
export function getLogger(): Logger {
  const context = getContext();
  return context?.logger || baseLogger;
}

// Run function with context
export function runWithContext<T>(
  context: Omit<RequestContext, 'logger'>,
  fn: () => T
): T {
  const contextLogger = baseLogger.child({
    requestId: context.requestId,
    userId: context.userId,
    traceId: context.traceId,
  });
  
  return asyncLocalStorage.run(
    { ...context, logger: contextLogger },
    fn
  );
}

// Express middleware
import { Request, Response, NextFunction } from 'express';
import { v4 as uuidv4 } from 'uuid';

export function contextMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  const requestId = (req.headers['x-request-id'] as string) || uuidv4();
  const traceId = req.headers['x-trace-id'] as string;
  const userId = (req as any).user?.id;
  
  runWithContext({ requestId, userId, traceId }, () => {
    next();
  });
}

// Usage in any file - context automatically included
export function doSomething(): void {
  const log = getLogger();
  log.info('Doing something'); // Includes requestId, userId, traceId
}
```

### Error Logging with Stack Traces

```typescript
// error-logging.ts
import { logger } from './logger';

interface ErrorLogData {
  error: Error;
  context?: Record<string, unknown>;
  userId?: string;
  requestId?: string;
}

function logError({ error, context, userId, requestId }: ErrorLogData): void {
  logger.error({
    msg: error.message,
    error: {
      name: error.name,
      message: error.message,
      stack: error.stack,
      // Include custom error properties
      ...(error as any).code && { code: (error as any).code },
      ...(error as any).statusCode && { statusCode: (error as any).statusCode },
    },
    context,
    userId,
    requestId,
  });
}

// Serializer for errors
import pino from 'pino';

const errorSerializer = pino.stdSerializers.err;

// Custom error serializer
function customErrorSerializer(error: Error): Record<string, unknown> {
  const serialized: Record<string, unknown> = {
    type: error.constructor.name,
    message: error.message,
    stack: error.stack,
  };
  
  // Include all enumerable properties
  for (const [key, value] of Object.entries(error)) {
    if (!['name', 'message', 'stack'].includes(key)) {
      serialized[key] = value;
    }
  }
  
  // Handle cause chain (ES2022)
  if (error.cause) {
    serialized.cause = customErrorSerializer(error.cause as Error);
  }
  
  return serialized;
}
```

### Audit Logging

```typescript
// audit-logger.ts
import { logger } from './logger';

interface AuditEvent {
  action: string;
  actor: {
    type: 'user' | 'system' | 'api';
    id: string;
    ip?: string;
  };
  resource: {
    type: string;
    id: string;
  };
  changes?: {
    before?: Record<string, unknown>;
    after?: Record<string, unknown>;
  };
  result: 'success' | 'failure';
  reason?: string;
}

const auditLogger = logger.child({ type: 'audit' });

export function logAuditEvent(event: AuditEvent): void {
  auditLogger.info({
    msg: `${event.action} on ${event.resource.type}`,
    audit: event,
  });
}

// Usage examples
function logUserCreated(adminId: string, userId: string, ip: string): void {
  logAuditEvent({
    action: 'user.create',
    actor: { type: 'user', id: adminId, ip },
    resource: { type: 'user', id: userId },
    result: 'success',
  });
}

function logPermissionDenied(userId: string, resource: string, action: string): void {
  logAuditEvent({
    action: `${resource}.${action}`,
    actor: { type: 'user', id: userId },
    resource: { type: resource, id: 'N/A' },
    result: 'failure',
    reason: 'Permission denied',
  });
}

function logDataModified(
  userId: string,
  resourceType: string,
  resourceId: string,
  before: Record<string, unknown>,
  after: Record<string, unknown>
): void {
  logAuditEvent({
    action: `${resourceType}.update`,
    actor: { type: 'user', id: userId },
    resource: { type: resourceType, id: resourceId },
    changes: { before, after },
    result: 'success',
  });
}
```

---

## Real-World Scenarios

### Scenario 1: Correlation Across Services

```typescript
// correlation.ts
import { logger } from './logger';
import axios from 'axios';

interface RequestHeaders {
  'x-request-id': string;
  'x-trace-id': string;
  'x-correlation-id': string;
}

// Generate correlation headers
function getCorrelationHeaders(requestId: string): RequestHeaders {
  return {
    'x-request-id': requestId,
    'x-trace-id': requestId, // Can be different if using distributed tracing
    'x-correlation-id': requestId,
  };
}

// Service A: Handles incoming request
async function handleIncomingRequest(requestId: string): Promise<void> {
  const log = logger.child({ requestId });
  
  log.info({ msg: 'Processing order', orderId: '123' });
  
  // Call Service B with correlation headers
  const headers = getCorrelationHeaders(requestId);
  
  try {
    const response = await axios.post(
      'http://service-b/validate',
      { orderId: '123' },
      { headers }
    );
    
    log.info({ 
      msg: 'Validation complete', 
      response: response.data,
    });
  } catch (error) {
    log.error({ 
      msg: 'Validation failed', 
      error,
    });
    throw error;
  }
}

// Service B: Extracts correlation ID
import { Request, Response, NextFunction } from 'express';

function correlationMiddleware(req: Request, res: Response, next: NextFunction): void {
  const requestId = req.headers['x-request-id'] as string || 
                    req.headers['x-correlation-id'] as string;
  
  // Add to request
  (req as any).correlationId = requestId;
  
  // Add to response
  if (requestId) {
    res.setHeader('x-correlation-id', requestId);
  }
  
  next();
}
```

### Scenario 2: Log Aggregation Setup

```typescript
// log-shipping.ts
import pino from 'pino';

// For shipping to different destinations
const transports = pino.transport({
  targets: [
    // Console output (development)
    {
      target: 'pino-pretty',
      options: { colorize: true },
      level: 'debug',
    },
    // File output
    {
      target: 'pino/file',
      options: { destination: '/var/log/app/app.log' },
      level: 'info',
    },
    // Loki (via pino-loki)
    {
      target: 'pino-loki',
      options: {
        host: process.env.LOKI_HOST || 'http://localhost:3100',
        labels: { app: 'my-service' },
      },
      level: 'info',
    },
  ],
});

const logger = pino(transports);

// For Elasticsearch (via Filebeat + Logstash)
// Just output JSON to stdout/file, Filebeat picks it up

// For CloudWatch (AWS)
// pino outputs JSON, CloudWatch agent collects from stdout
```

---

## Common Pitfalls

### 1. Logging Sensitive Data

```typescript
// ❌ BAD: Logging passwords
logger.info({ user: { email, password } }, 'User logged in');

// ✅ GOOD: Redact sensitive fields
const logger = pino({
  redact: ['password', '*.password', 'token', 'creditCard'],
});

// Or manually exclude
logger.info({ 
  user: { email }, // Exclude password
  msg: 'User logged in' 
});
```

### 2. Missing Context

```typescript
// ❌ BAD: No context
logger.error('Payment failed');

// ✅ GOOD: Include context
logger.error({
  msg: 'Payment failed',
  paymentId: 'pay_123',
  userId: 'user_456',
  amount: 99.99,
  error: error.message,
});
```

### 3. String Concatenation

```typescript
// ❌ BAD: String concatenation (not searchable)
logger.info(`User ${userId} purchased ${product}`);

// ✅ GOOD: Structured fields
logger.info({
  msg: 'Purchase completed',
  userId,
  product,
});
```

---

## Interview Questions

### Q1: Why use structured logging over plain text?

**A:** Structured logs (JSON) are machine-parseable, enabling: 1) Searching by any field (user ID, error type), 2) Aggregations and analytics, 3) Alerting on specific patterns, 4) Correlation across services via trace IDs. Plain text requires regex parsing which is slow, fragile, and loses type information.

### Q2: What should always be included in production logs?

**A:** Timestamp (ISO 8601), log level, service name, message, request/correlation ID, error details (if applicable), and business context. Optionally: trace ID (for distributed tracing), user ID (if authenticated), duration (for performance).

### Q3: How do you handle high-volume logging without performance impact?

**A:** 1) Use async logging (Pino is async by default), 2) Appropriate log levels (don't DEBUG in prod), 3) Sample verbose logs, 4) Buffer and batch ship to aggregators, 5) Use dedicated log shipping (Filebeat) instead of in-app HTTP calls.

### Q4: How do you correlate logs across microservices?

**A:** Generate a correlation/request ID at the edge (API gateway), propagate it via headers (X-Request-ID, X-Correlation-ID), include it in every log entry. In your log aggregation system, search by correlation ID to see all logs from a single request across all services.

---

## Quick Reference Checklist

### Log Entry Requirements
- [ ] Timestamp in ISO 8601
- [ ] Log level as string
- [ ] Service name
- [ ] Request/correlation ID
- [ ] Descriptive message

### Configuration
- [ ] Redact sensitive fields
- [ ] Set appropriate log levels
- [ ] Configure output format (JSON for prod)
- [ ] Set up log rotation

### Best Practices
- [ ] Use structured fields, not string concatenation
- [ ] Include error stack traces
- [ ] Add business context
- [ ] Correlate with trace IDs

### Operations
- [ ] Set up log aggregation
- [ ] Create log-based alerts
- [ ] Define retention policies
- [ ] Monitor log volume/costs

---

*Last updated: February 2026*

