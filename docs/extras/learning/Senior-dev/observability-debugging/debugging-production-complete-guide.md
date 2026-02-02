# Debugging Production - Complete Guide

> **MUST REMEMBER**: Production debugging requires non-invasive techniques: feature flags for safe code changes, debug logging that can be enabled dynamically, distributed tracing to follow requests, and metrics to identify anomalies. Never attach debuggers directly in production. Use read-only tools and replay techniques when possible.

---

## How to Explain Like a Senior Developer

"You can't attach a debugger to production and step through code - users are depending on it. Production debugging is detective work: correlate metrics to identify WHEN something changed, traces to see WHERE in the call chain, logs to understand WHAT happened, and error reports for the exact failure. Feature flags let you toggle debug logging for specific users without deploying. Shadow traffic lets you replay production requests in staging. The key mindset is: observe without disturbing. Every debugging action should be safe to do while serving real traffic."

---

## Core Implementation

### Dynamic Debug Logging

```typescript
// debug/dynamic-logging.ts
import { logger } from '../logger';

interface DebugConfig {
  enabled: boolean;
  users: Set<string>;
  paths: Set<string>;
  features: Set<string>;
  sampleRate: number;
  expiresAt?: Date;
}

let debugConfig: DebugConfig = {
  enabled: false,
  users: new Set(),
  paths: new Set(),
  features: new Set(),
  sampleRate: 0,
};

// Update debug config (call from admin endpoint or feature flag service)
export function updateDebugConfig(config: Partial<DebugConfig>): void {
  debugConfig = { ...debugConfig, ...config };
  logger.info({ debugConfig }, 'Debug config updated');
}

// Check if debug logging should be enabled for this context
export function shouldDebug(context: {
  userId?: string;
  path?: string;
  feature?: string;
}): boolean {
  // Check if globally enabled
  if (!debugConfig.enabled) return false;
  
  // Check if expired
  if (debugConfig.expiresAt && new Date() > debugConfig.expiresAt) {
    debugConfig.enabled = false;
    return false;
  }
  
  // Check specific user
  if (context.userId && debugConfig.users.has(context.userId)) {
    return true;
  }
  
  // Check specific path
  if (context.path && debugConfig.paths.has(context.path)) {
    return true;
  }
  
  // Check specific feature
  if (context.feature && debugConfig.features.has(context.feature)) {
    return true;
  }
  
  // Sample rate
  if (debugConfig.sampleRate > 0 && Math.random() < debugConfig.sampleRate) {
    return true;
  }
  
  return false;
}

// Debug logger that checks config
export function debugLog(
  context: { userId?: string; path?: string; feature?: string },
  message: string,
  data?: object
): void {
  if (shouldDebug(context)) {
    logger.debug({
      ...data,
      debug: true,
      context,
      msg: message,
    });
  }
}

// Express middleware for debug context
import { Request, Response, NextFunction } from 'express';

export function debugMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  const context = {
    userId: (req as any).user?.id,
    path: req.path,
  };
  
  if (shouldDebug(context)) {
    debugLog(context, 'Request started', {
      method: req.method,
      query: req.query,
      headers: req.headers,
    });
    
    // Log response
    const originalSend = res.send.bind(res);
    res.send = function(body: any) {
      debugLog(context, 'Response sent', {
        statusCode: res.statusCode,
        bodySize: body?.length,
      });
      return originalSend(body);
    };
  }
  
  next();
}
```

### Feature Flag Based Debugging

```typescript
// debug/feature-flags.ts
import { logger } from '../logger';

interface FeatureFlagService {
  isEnabled(flag: string, userId?: string): Promise<boolean>;
  getVariant(flag: string, userId?: string): Promise<string | null>;
}

// Assume we have a feature flag service
declare const featureFlags: FeatureFlagService;

// Debug wrapper that uses feature flags
export async function debugOperation<T>(
  operationName: string,
  userId: string | undefined,
  operation: () => Promise<T>
): Promise<T> {
  const isDebug = await featureFlags.isEnabled(`debug:${operationName}`, userId);
  
  if (isDebug) {
    const start = Date.now();
    logger.debug({ operation: operationName, userId, phase: 'start' });
    
    try {
      const result = await operation();
      
      logger.debug({
        operation: operationName,
        userId,
        phase: 'complete',
        duration: Date.now() - start,
        // Careful: don't log sensitive data
        resultType: typeof result,
      });
      
      return result;
    } catch (error) {
      logger.debug({
        operation: operationName,
        userId,
        phase: 'error',
        duration: Date.now() - start,
        error: (error as Error).message,
        stack: (error as Error).stack,
      });
      throw error;
    }
  }
  
  return operation();
}

// Usage
async function processPayment(userId: string, amount: number): Promise<void> {
  await debugOperation('payment', userId, async () => {
    // Actual payment processing
  });
}
```

### Request Replay and Shadow Traffic

```typescript
// debug/request-replay.ts
import { Request } from 'express';
import axios from 'axios';

interface RecordedRequest {
  id: string;
  timestamp: Date;
  method: string;
  path: string;
  query: object;
  headers: object;
  body: object;
  userId?: string;
}

// Record requests for replay (store in Redis or S3)
const recordedRequests: RecordedRequest[] = [];

export function recordRequest(req: Request): void {
  const recorded: RecordedRequest = {
    id: `req-${Date.now()}-${Math.random().toString(36).slice(2)}`,
    timestamp: new Date(),
    method: req.method,
    path: req.path,
    query: req.query,
    headers: sanitizeHeaders(req.headers),
    body: req.body,
    userId: (req as any).user?.id,
  };
  
  recordedRequests.push(recorded);
  
  // Keep only last 1000 requests
  if (recordedRequests.length > 1000) {
    recordedRequests.shift();
  }
}

function sanitizeHeaders(headers: object): object {
  const sanitized = { ...headers } as any;
  delete sanitized.authorization;
  delete sanitized.cookie;
  return sanitized;
}

// Replay request to staging environment
export async function replayRequest(
  request: RecordedRequest,
  targetUrl: string
): Promise<any> {
  const response = await axios({
    method: request.method as any,
    url: `${targetUrl}${request.path}`,
    params: request.query,
    headers: {
      ...request.headers,
      'x-replay-request-id': request.id,
      'x-original-timestamp': request.timestamp.toISOString(),
    },
    data: request.body,
    validateStatus: () => true, // Don't throw on error status
  });
  
  return {
    originalRequestId: request.id,
    replayStatus: response.status,
    replayData: response.data,
  };
}

// Replay recent requests to debug
export async function replayRecentRequests(
  filter: { path?: string; userId?: string; since?: Date },
  targetUrl: string
): Promise<any[]> {
  const filtered = recordedRequests.filter((req) => {
    if (filter.path && !req.path.includes(filter.path)) return false;
    if (filter.userId && req.userId !== filter.userId) return false;
    if (filter.since && req.timestamp < filter.since) return false;
    return true;
  });
  
  const results = [];
  for (const req of filtered) {
    const result = await replayRequest(req, targetUrl);
    results.push(result);
  }
  
  return results;
}
```

### Distributed Debugging with Trace Context

```typescript
// debug/trace-debug.ts
import { trace, context, SpanStatusCode } from '@opentelemetry/api';
import { logger } from '../logger';

const tracer = trace.getTracer('debug-tracer');

interface DebugContext {
  traceId: string;
  spanId: string;
  userId?: string;
  debugMode: boolean;
}

// Get debug context from current trace
export function getDebugContext(): DebugContext {
  const span = trace.getSpan(context.active());
  const spanContext = span?.spanContext();
  
  return {
    traceId: spanContext?.traceId || 'no-trace',
    spanId: spanContext?.spanId || 'no-span',
    userId: (context.active() as any).userId,
    debugMode: span?.getAttribute('debug') === true,
  };
}

// Enable debug mode for current trace
export function enableDebugMode(): void {
  const span = trace.getSpan(context.active());
  if (span) {
    span.setAttribute('debug', true);
    span.setAttribute('debug.enabled_at', new Date().toISOString());
  }
}

// Debug point - adds detailed info to trace
export function debugPoint(
  name: string,
  data: object
): void {
  const ctx = getDebugContext();
  
  if (ctx.debugMode) {
    const span = trace.getSpan(context.active());
    span?.addEvent(`debug:${name}`, {
      ...data,
      timestamp: Date.now(),
    });
    
    logger.debug({
      msg: `Debug point: ${name}`,
      traceId: ctx.traceId,
      data,
    });
  }
}

// Usage in code
async function processOrder(orderId: string): Promise<void> {
  debugPoint('order.start', { orderId });
  
  const order = await fetchOrder(orderId);
  debugPoint('order.fetched', { 
    orderId, 
    items: order.items.length,
    total: order.total 
  });
  
  await validateInventory(order);
  debugPoint('order.inventory_validated', { orderId });
  
  await processPayment(order);
  debugPoint('order.payment_processed', { orderId });
}

async function fetchOrder(orderId: string): Promise<any> {
  return { items: [], total: 0 };
}

async function validateInventory(order: any): Promise<void> {}

async function processPayment(order: any): Promise<void> {}
```

### Query Inspector for Database Debugging

```typescript
// debug/query-inspector.ts
import { logger } from '../logger';

interface QueryLog {
  id: string;
  query: string;
  params: any[];
  duration: number;
  rowCount?: number;
  error?: string;
  timestamp: Date;
  stack?: string;
}

const queryLogs: QueryLog[] = [];
let inspectorEnabled = false;

export function enableQueryInspector(): void {
  inspectorEnabled = true;
  logger.info('Query inspector enabled');
}

export function disableQueryInspector(): void {
  inspectorEnabled = false;
  logger.info('Query inspector disabled');
}

export function logQuery(
  query: string,
  params: any[],
  duration: number,
  rowCount?: number,
  error?: string
): void {
  if (!inspectorEnabled) return;
  
  const log: QueryLog = {
    id: `query-${Date.now()}`,
    query,
    params,
    duration,
    rowCount,
    error,
    timestamp: new Date(),
    stack: new Error().stack,
  };
  
  queryLogs.push(log);
  
  // Keep last 100 queries
  if (queryLogs.length > 100) {
    queryLogs.shift();
  }
  
  // Log slow queries immediately
  if (duration > 1000) {
    logger.warn({
      msg: 'Slow query detected',
      query: query.substring(0, 200),
      duration,
    });
  }
}

// Get recent queries for debugging
export function getRecentQueries(filter?: {
  minDuration?: number;
  hasError?: boolean;
}): QueryLog[] {
  return queryLogs.filter((log) => {
    if (filter?.minDuration && log.duration < filter.minDuration) return false;
    if (filter?.hasError !== undefined && !!log.error !== filter.hasError) return false;
    return true;
  });
}

// Analyze query patterns
export function analyzeQueryPatterns(): object {
  const patterns: Map<string, { count: number; totalDuration: number }> = new Map();
  
  for (const log of queryLogs) {
    // Normalize query (remove specific values)
    const pattern = log.query
      .replace(/\$\d+/g, '?')
      .replace(/\d+/g, 'N')
      .replace(/'[^']*'/g, "'X'");
    
    const existing = patterns.get(pattern) || { count: 0, totalDuration: 0 };
    existing.count++;
    existing.totalDuration += log.duration;
    patterns.set(pattern, existing);
  }
  
  return Array.from(patterns.entries())
    .map(([pattern, stats]) => ({
      pattern,
      count: stats.count,
      avgDuration: stats.totalDuration / stats.count,
    }))
    .sort((a, b) => b.count - a.count);
}
```

---

## Real-World Scenarios

### Scenario 1: Debugging Intermittent Errors

```typescript
// debug/intermittent-errors.ts
import { logger } from '../logger';

interface ErrorContext {
  errorId: string;
  timestamp: Date;
  error: Error;
  request: {
    method: string;
    path: string;
    userId?: string;
    body?: object;
  };
  systemState: {
    memoryUsage: number;
    cpuLoad: number;
    activeConnections: number;
  };
  recentEvents: string[];
}

const errorContexts: ErrorContext[] = [];
const recentEvents: string[] = [];

// Track important events
export function trackEvent(event: string): void {
  recentEvents.push(`${new Date().toISOString()}: ${event}`);
  if (recentEvents.length > 100) {
    recentEvents.shift();
  }
}

// Capture rich error context
export function captureErrorContext(
  error: Error,
  request: { method: string; path: string; userId?: string; body?: object }
): ErrorContext {
  const context: ErrorContext = {
    errorId: `err-${Date.now()}-${Math.random().toString(36).slice(2)}`,
    timestamp: new Date(),
    error,
    request,
    systemState: {
      memoryUsage: process.memoryUsage().heapUsed,
      cpuLoad: process.cpuUsage().user,
      activeConnections: getActiveConnections(),
    },
    recentEvents: [...recentEvents],
  };
  
  errorContexts.push(context);
  
  logger.error({
    msg: 'Error captured with context',
    errorId: context.errorId,
    error: error.message,
    systemState: context.systemState,
    recentEvents: context.recentEvents.slice(-10),
  });
  
  return context;
}

function getActiveConnections(): number {
  // Get from your server/pool stats
  return 0;
}

// Find patterns in errors
export function analyzeErrors(): object {
  const byHour: Map<number, number> = new Map();
  const byPath: Map<string, number> = new Map();
  const byMemory: { low: number; medium: number; high: number } = { low: 0, medium: 0, high: 0 };
  
  for (const ctx of errorContexts) {
    // By hour
    const hour = ctx.timestamp.getHours();
    byHour.set(hour, (byHour.get(hour) || 0) + 1);
    
    // By path
    byPath.set(ctx.request.path, (byPath.get(ctx.request.path) || 0) + 1);
    
    // By memory
    const memoryMB = ctx.systemState.memoryUsage / 1024 / 1024;
    if (memoryMB < 500) byMemory.low++;
    else if (memoryMB < 1000) byMemory.medium++;
    else byMemory.high++;
  }
  
  return {
    total: errorContexts.length,
    byHour: Object.fromEntries(byHour),
    byPath: Object.fromEntries(byPath),
    byMemory,
  };
}
```

### Scenario 2: Live Traffic Analysis

```typescript
// debug/traffic-analyzer.ts
interface RequestSample {
  timestamp: Date;
  path: string;
  method: string;
  statusCode: number;
  duration: number;
  userId?: string;
}

const samples: RequestSample[] = [];
const SAMPLE_RATE = 0.1; // Sample 10% of requests

export function sampleRequest(sample: RequestSample): void {
  if (Math.random() > SAMPLE_RATE) return;
  
  samples.push(sample);
  
  // Keep 1 hour of samples
  const oneHourAgo = new Date(Date.now() - 3600000);
  while (samples.length > 0 && samples[0].timestamp < oneHourAgo) {
    samples.shift();
  }
}

// Real-time analysis
export function getTrafficAnalysis(): object {
  const now = Date.now();
  const lastMinute = samples.filter(s => s.timestamp.getTime() > now - 60000);
  const last5Minutes = samples.filter(s => s.timestamp.getTime() > now - 300000);
  
  return {
    lastMinute: {
      requestCount: lastMinute.length / SAMPLE_RATE,
      avgDuration: avg(lastMinute.map(s => s.duration)),
      p99Duration: percentile(lastMinute.map(s => s.duration), 99),
      errorRate: lastMinute.filter(s => s.statusCode >= 500).length / lastMinute.length,
      topPaths: groupByCount(lastMinute, 'path'),
    },
    last5Minutes: {
      requestCount: last5Minutes.length / SAMPLE_RATE,
      avgDuration: avg(last5Minutes.map(s => s.duration)),
      errorRate: last5Minutes.filter(s => s.statusCode >= 500).length / last5Minutes.length,
    },
  };
}

function avg(arr: number[]): number {
  return arr.length ? arr.reduce((a, b) => a + b, 0) / arr.length : 0;
}

function percentile(arr: number[], p: number): number {
  const sorted = [...arr].sort((a, b) => a - b);
  const index = Math.ceil((p / 100) * sorted.length) - 1;
  return sorted[index] || 0;
}

function groupByCount(arr: RequestSample[], key: keyof RequestSample): object {
  const groups: Map<string, number> = new Map();
  arr.forEach(item => {
    const value = String(item[key]);
    groups.set(value, (groups.get(value) || 0) + 1);
  });
  return Object.fromEntries([...groups.entries()].sort((a, b) => b[1] - a[1]).slice(0, 10));
}
```

---

## Common Pitfalls

### 1. Exposing Debug Endpoints Without Auth

```typescript
// ❌ BAD: Debug endpoints open to everyone
app.get('/debug/logs', (req, res) => {
  res.json(getDebugLogs());
});

// ✅ GOOD: Authenticated and rate limited
app.get('/debug/logs', 
  requireAdmin,
  rateLimit({ windowMs: 60000, max: 10 }),
  (req, res) => {
    res.json(getDebugLogs());
  }
);

function getDebugLogs(): object { return {}; }
function requireAdmin(req: any, res: any, next: Function): void { next(); }
function rateLimit(config: object): Function { return (req: any, res: any, next: Function) => next(); }
const app = { get: (path: string, ...handlers: Function[]) => {} };
```

### 2. Logging Sensitive Data in Debug Mode

```typescript
// ❌ BAD: Logging passwords
debugLog(context, 'User login attempt', { email, password });

// ✅ GOOD: Sanitize sensitive fields
debugLog(context, 'User login attempt', { 
  email, 
  passwordProvided: !!password 
});
```

### 3. Debug Code Affecting Production Performance

```typescript
// ❌ BAD: Always computing debug info
const debugInfo = computeExpensiveDebugInfo(); // Runs every time
if (debugEnabled) {
  logger.debug(debugInfo);
}

// ✅ GOOD: Only compute when needed
if (debugEnabled) {
  const debugInfo = computeExpensiveDebugInfo();
  logger.debug(debugInfo);
}

function computeExpensiveDebugInfo(): object { return {}; }
const debugEnabled = false;
const logger = { debug: (data: object) => {} };
```

---

## Interview Questions

### Q1: How do you debug a problem that only occurs in production?

**A:** 1) Use logs and metrics to understand when/where it happens, 2) Enable debug logging for affected users via feature flags, 3) Analyze distributed traces to see the request flow, 4) Capture rich error context (system state, recent events), 5) Replay requests in staging if possible, 6) Add targeted instrumentation, 7) Use A/B testing to isolate the cause.

### Q2: How do you safely add debugging code to production?

**A:** Use feature flags to control debug functionality. Require authentication for debug endpoints. Set automatic expiration on debug modes. Rate limit debug operations. Ensure debug code doesn't affect performance when disabled. Log when debug mode is activated. Have kill switches ready.

### Q3: What information should you capture when an error occurs?

**A:** Error message and stack trace, request details (path, method, user), system state (memory, CPU, connections), correlation/trace ID, recent events leading to error, timestamp, and any relevant business context. Avoid sensitive data like passwords or tokens.

### Q4: How do you debug performance issues in production?

**A:** Use metrics to identify WHEN (time patterns, after deployments). Use traces to identify WHERE (slow spans, blocking calls). Use profiling to identify WHAT (specific functions). Sample real traffic, don't profile everything. Look for patterns: is it specific endpoints, users, or data patterns?

---

## Quick Reference Checklist

### Safe Debug Techniques
- [ ] Feature flag controlled debug logging
- [ ] Authenticated debug endpoints
- [ ] Request sampling and replay
- [ ] Read-only diagnostics
- [ ] Automatic debug mode expiration

### Information to Capture
- [ ] Full error context
- [ ] Request/trace correlation
- [ ] System state at error time
- [ ] Recent events/breadcrumbs
- [ ] User context

### Security
- [ ] Require admin auth for debug endpoints
- [ ] Sanitize sensitive data
- [ ] Rate limit debug operations
- [ ] Audit debug access

### Performance
- [ ] Only compute debug info when needed
- [ ] Use sampling for high-volume debugging
- [ ] Set time limits on debug modes
- [ ] Monitor debug overhead

---

*Last updated: February 2026*

