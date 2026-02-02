# Health Checks - Complete Guide

> **MUST REMEMBER**: Health checks verify your application is running correctly and ready to handle traffic. There are three main types: liveness (is it running?), readiness (can it handle requests?), and startup (has it finished initializing?). Deep health checks verify dependencies like databases and external services, but must be fast to avoid false positives.

---

## How to Explain Like a Senior Developer

"Health checks are how your infrastructure knows if your app is okay. A liveness probe answers 'should I restart this?', a readiness probe answers 'should I send traffic here?', and a startup probe answers 'has it finished booting?'. The trick is making them fast and meaningful - a slow database query in your health check can cause cascading failures when the load balancer marks healthy instances as unhealthy. Deep checks that verify dependencies are valuable but risky; shallow checks are safer but might not catch real problems. The best approach is layered: a fast liveness check, a moderate readiness check that verifies critical deps, and separate endpoints for detailed diagnostics."

---

## Core Implementation

### Basic Health Check Endpoints

```typescript
// health-routes.ts
import { Router } from 'express';

const router = Router();

// Simple liveness - is the process running?
router.get('/health/live', (req, res) => {
  res.status(200).json({
    status: 'alive',
    timestamp: new Date().toISOString(),
  });
});

// Readiness - can it handle requests?
router.get('/health/ready', async (req, res) => {
  try {
    // Check critical dependencies
    await checkDatabase();
    await checkCache();
    
    res.status(200).json({
      status: 'ready',
      timestamp: new Date().toISOString(),
    });
  } catch (error) {
    res.status(503).json({
      status: 'not ready',
      error: (error as Error).message,
      timestamp: new Date().toISOString(),
    });
  }
});

// Deep health check - full system status
router.get('/health', async (req, res) => {
  const checks = await runHealthChecks();
  const allHealthy = checks.every(c => c.status === 'healthy');
  
  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? 'healthy' : 'unhealthy',
    timestamp: new Date().toISOString(),
    checks,
  });
});

export { router as healthRouter };

async function checkDatabase(): Promise<void> {
  // Implementation
}

async function checkCache(): Promise<void> {
  // Implementation
}

async function runHealthChecks(): Promise<Array<{ name: string; status: string }>> {
  return [];
}
```

### Comprehensive Health Check Service

```typescript
// health-check-service.ts
import { EventEmitter } from 'events';

type HealthStatus = 'healthy' | 'unhealthy' | 'degraded';

interface HealthCheckResult {
  name: string;
  status: HealthStatus;
  latency: number;
  message?: string;
  details?: Record<string, unknown>;
}

interface HealthCheckConfig {
  name: string;
  critical: boolean;
  timeout: number;
  check: () => Promise<void>;
}

interface HealthReport {
  status: HealthStatus;
  timestamp: string;
  uptime: number;
  version: string;
  checks: HealthCheckResult[];
  memory: NodeJS.MemoryUsage;
}

class HealthCheckService extends EventEmitter {
  private checks: HealthCheckConfig[] = [];
  private cachedResults: Map<string, HealthCheckResult> = new Map();
  private cacheInterval: NodeJS.Timeout | null = null;
  
  constructor(
    private readonly version: string = process.env.npm_package_version || '0.0.0'
  ) {
    super();
  }
  
  /**
   * Register a health check
   */
  register(config: HealthCheckConfig): void {
    this.checks.push(config);
  }
  
  /**
   * Start background health checking
   */
  startBackgroundChecks(intervalMs: number = 30000): void {
    this.runChecks(); // Initial check
    
    this.cacheInterval = setInterval(() => {
      this.runChecks();
    }, intervalMs);
  }
  
  /**
   * Stop background checks
   */
  stopBackgroundChecks(): void {
    if (this.cacheInterval) {
      clearInterval(this.cacheInterval);
      this.cacheInterval = null;
    }
  }
  
  /**
   * Run all health checks
   */
  async runChecks(): Promise<HealthCheckResult[]> {
    const results = await Promise.all(
      this.checks.map(check => this.executeCheck(check))
    );
    
    // Update cache
    results.forEach(result => {
      this.cachedResults.set(result.name, result);
    });
    
    // Emit events for status changes
    results.forEach(result => {
      const previous = this.cachedResults.get(result.name);
      if (previous && previous.status !== result.status) {
        this.emit('status-change', {
          name: result.name,
          previous: previous.status,
          current: result.status,
        });
      }
    });
    
    return results;
  }
  
  /**
   * Execute a single health check with timeout
   */
  private async executeCheck(config: HealthCheckConfig): Promise<HealthCheckResult> {
    const start = Date.now();
    
    try {
      await Promise.race([
        config.check(),
        new Promise((_, reject) => 
          setTimeout(() => reject(new Error('Timeout')), config.timeout)
        ),
      ]);
      
      return {
        name: config.name,
        status: 'healthy',
        latency: Date.now() - start,
      };
    } catch (error) {
      return {
        name: config.name,
        status: 'unhealthy',
        latency: Date.now() - start,
        message: (error as Error).message,
      };
    }
  }
  
  /**
   * Get liveness status (simple alive check)
   */
  getLiveness(): { status: 'alive'; timestamp: string } {
    return {
      status: 'alive',
      timestamp: new Date().toISOString(),
    };
  }
  
  /**
   * Get readiness status (based on critical checks)
   */
  async getReadiness(): Promise<{ status: 'ready' | 'not ready'; timestamp: string }> {
    const criticalChecks = this.checks.filter(c => c.critical);
    const results = await Promise.all(
      criticalChecks.map(check => this.executeCheck(check))
    );
    
    const allHealthy = results.every(r => r.status === 'healthy');
    
    return {
      status: allHealthy ? 'ready' : 'not ready',
      timestamp: new Date().toISOString(),
    };
  }
  
  /**
   * Get full health report
   */
  async getHealthReport(): Promise<HealthReport> {
    const checks = await this.runChecks();
    
    const criticalUnhealthy = checks.some(
      c => this.checks.find(cfg => cfg.name === c.name)?.critical && 
           c.status === 'unhealthy'
    );
    
    const anyUnhealthy = checks.some(c => c.status === 'unhealthy');
    
    let status: HealthStatus;
    if (criticalUnhealthy) {
      status = 'unhealthy';
    } else if (anyUnhealthy) {
      status = 'degraded';
    } else {
      status = 'healthy';
    }
    
    return {
      status,
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      version: this.version,
      checks,
      memory: process.memoryUsage(),
    };
  }
  
  /**
   * Get cached results (fast, for frequent polling)
   */
  getCachedResults(): HealthCheckResult[] {
    return Array.from(this.cachedResults.values());
  }
}

export const healthService = new HealthCheckService();
```

### Dependency Health Checks

```typescript
// health-checks/index.ts
import { healthService } from './health-check-service';
import { Pool } from 'pg';
import Redis from 'ioredis';
import axios from 'axios';

// PostgreSQL check
export function registerDatabaseCheck(pool: Pool): void {
  healthService.register({
    name: 'database',
    critical: true,
    timeout: 5000,
    check: async () => {
      const client = await pool.connect();
      try {
        await client.query('SELECT 1');
      } finally {
        client.release();
      }
    },
  });
}

// Redis check
export function registerRedisCheck(redis: Redis): void {
  healthService.register({
    name: 'redis',
    critical: true,
    timeout: 3000,
    check: async () => {
      const result = await redis.ping();
      if (result !== 'PONG') {
        throw new Error('Unexpected response');
      }
    },
  });
}

// External API check
export function registerExternalAPICheck(
  name: string,
  url: string,
  critical: boolean = false
): void {
  healthService.register({
    name: `external-${name}`,
    critical,
    timeout: 5000,
    check: async () => {
      const response = await axios.get(url, { timeout: 4000 });
      if (response.status !== 200) {
        throw new Error(`Status ${response.status}`);
      }
    },
  });
}

// Disk space check
export function registerDiskSpaceCheck(
  path: string = '/',
  minFreePercent: number = 10
): void {
  healthService.register({
    name: 'disk-space',
    critical: false,
    timeout: 2000,
    check: async () => {
      const { execSync } = await import('child_process');
      const output = execSync(`df -P ${path} | tail -1 | awk '{print $5}'`);
      const usedPercent = parseInt(output.toString().replace('%', ''));
      const freePercent = 100 - usedPercent;
      
      if (freePercent < minFreePercent) {
        throw new Error(`Only ${freePercent}% disk space free`);
      }
    },
  });
}

// Memory check
export function registerMemoryCheck(maxHeapUsagePercent: number = 90): void {
  healthService.register({
    name: 'memory',
    critical: false,
    timeout: 1000,
    check: async () => {
      const usage = process.memoryUsage();
      const heapUsedPercent = (usage.heapUsed / usage.heapTotal) * 100;
      
      if (heapUsedPercent > maxHeapUsagePercent) {
        throw new Error(`Heap usage at ${heapUsedPercent.toFixed(1)}%`);
      }
    },
  });
}

// Event loop lag check
export function registerEventLoopCheck(maxLagMs: number = 100): void {
  let lastCheck = Date.now();
  let lag = 0;
  
  // Background measurement
  setInterval(() => {
    const now = Date.now();
    const expected = 1000;
    lag = Math.max(0, now - lastCheck - expected);
    lastCheck = now;
  }, 1000).unref();
  
  healthService.register({
    name: 'event-loop',
    critical: false,
    timeout: 1000,
    check: async () => {
      if (lag > maxLagMs) {
        throw new Error(`Event loop lag: ${lag}ms`);
      }
    },
  });
}
```

### Health Check Routes with Caching

```typescript
// health-routes-advanced.ts
import { Router, Request, Response } from 'express';
import { healthService } from './health-check-service';

const router = Router();

// Cache control for health endpoints
const setCacheHeaders = (res: Response, maxAge: number): void => {
  res.setHeader('Cache-Control', `public, max-age=${maxAge}`);
};

// Liveness - very fast, no dependencies
router.get('/health/live', (req, res) => {
  setCacheHeaders(res, 5);
  res.status(200).json(healthService.getLiveness());
});

// Readiness - check critical dependencies
router.get('/health/ready', async (req, res) => {
  try {
    const result = await healthService.getReadiness();
    const statusCode = result.status === 'ready' ? 200 : 503;
    
    setCacheHeaders(res, 5);
    res.status(statusCode).json(result);
  } catch (error) {
    res.status(503).json({
      status: 'not ready',
      error: 'Health check failed',
      timestamp: new Date().toISOString(),
    });
  }
});

// Full health - detailed status of all checks
router.get('/health', async (req, res) => {
  // Use cached results for speed if available
  const useCache = req.query.cache !== 'false';
  
  let report;
  if (useCache) {
    const cached = healthService.getCachedResults();
    if (cached.length > 0) {
      const allHealthy = cached.every(c => c.status === 'healthy');
      report = {
        status: allHealthy ? 'healthy' : 'unhealthy',
        timestamp: new Date().toISOString(),
        uptime: process.uptime(),
        checks: cached,
        cached: true,
      };
    }
  }
  
  if (!report) {
    report = await healthService.getHealthReport();
  }
  
  const statusCode = report.status === 'healthy' ? 200 : 
                     report.status === 'degraded' ? 200 : 503;
  
  setCacheHeaders(res, 10);
  res.status(statusCode).json(report);
});

// Startup probe endpoint (K8s 1.16+)
let startupComplete = false;

router.get('/health/startup', (req, res) => {
  if (startupComplete) {
    res.status(200).json({ status: 'started' });
  } else {
    res.status(503).json({ status: 'starting' });
  }
});

export function markStartupComplete(): void {
  startupComplete = true;
}

export { router as healthRouter };
```

### Kubernetes Probe Configuration

```yaml
# kubernetes-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: api
          image: api-server:latest
          ports:
            - containerPort: 3000
          
          # Startup probe - gives app time to initialize
          startupProbe:
            httpGet:
              path: /health/startup
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 30  # 30 * 5 = 150 seconds max startup
          
          # Liveness probe - restart if unhealthy
          livenessProbe:
            httpGet:
              path: /health/live
              port: 3000
            initialDelaySeconds: 0
            periodSeconds: 10
            timeoutSeconds: 3
            failureThreshold: 3
          
          # Readiness probe - remove from service if not ready
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 3000
            initialDelaySeconds: 0
            periodSeconds: 5
            timeoutSeconds: 5
            failureThreshold: 3
```

---

## Real-World Scenarios

### Scenario 1: Circuit Breaker for External Dependencies

```typescript
// circuit-breaker-health.ts
import { healthService } from './health-check-service';

enum CircuitState {
  CLOSED = 'closed',
  OPEN = 'open',
  HALF_OPEN = 'half_open',
}

class CircuitBreaker {
  private state: CircuitState = CircuitState.CLOSED;
  private failures = 0;
  private lastFailure: Date | null = null;
  
  constructor(
    private readonly name: string,
    private readonly threshold: number,
    private readonly resetTimeout: number
  ) {}
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === CircuitState.OPEN) {
      if (this.shouldAttemptReset()) {
        this.state = CircuitState.HALF_OPEN;
      } else {
        throw new Error(`Circuit breaker ${this.name} is OPEN`);
      }
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  private shouldAttemptReset(): boolean {
    if (!this.lastFailure) return true;
    return Date.now() - this.lastFailure.getTime() > this.resetTimeout;
  }
  
  private onSuccess(): void {
    this.failures = 0;
    this.state = CircuitState.CLOSED;
  }
  
  private onFailure(): void {
    this.failures++;
    this.lastFailure = new Date();
    
    if (this.failures >= this.threshold) {
      this.state = CircuitState.OPEN;
    }
  }
  
  getStatus(): { state: CircuitState; failures: number } {
    return { state: this.state, failures: this.failures };
  }
}

// Usage with health checks
const externalApiBreaker = new CircuitBreaker('external-api', 5, 30000);

healthService.register({
  name: 'external-api',
  critical: false,
  timeout: 5000,
  check: async () => {
    const status = externalApiBreaker.getStatus();
    
    if (status.state === CircuitState.OPEN) {
      throw new Error('Circuit breaker open');
    }
    
    // Only actually check if circuit is closed or half-open
    await externalApiBreaker.execute(async () => {
      const response = await fetch('https://api.example.com/health');
      if (!response.ok) throw new Error('API unhealthy');
    });
  },
});
```

### Scenario 2: Multi-Level Health Checks

```typescript
// multi-level-health.ts
import { Router } from 'express';

const router = Router();

// L0: Process alive (instant)
router.get('/health/l0', (req, res) => {
  res.status(200).send('OK');
});

// L1: App initialized (instant, cached)
let appInitialized = false;
router.get('/health/l1', (req, res) => {
  if (appInitialized) {
    res.status(200).json({ initialized: true });
  } else {
    res.status(503).json({ initialized: false });
  }
});

// L2: Critical dependencies (fast, < 1s)
router.get('/health/l2', async (req, res) => {
  const start = Date.now();
  const checks = await Promise.allSettled([
    checkDatabase(),
    checkRedis(),
  ]);
  
  const results = checks.map((result, i) => ({
    name: ['database', 'redis'][i],
    status: result.status === 'fulfilled' ? 'healthy' : 'unhealthy',
    error: result.status === 'rejected' ? (result.reason as Error).message : undefined,
  }));
  
  const allHealthy = results.every(r => r.status === 'healthy');
  
  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? 'healthy' : 'unhealthy',
    latency: Date.now() - start,
    checks: results,
  });
});

// L3: All dependencies including non-critical (slower, < 10s)
router.get('/health/l3', async (req, res) => {
  const start = Date.now();
  const checks = await Promise.allSettled([
    checkDatabase(),
    checkRedis(),
    checkElasticsearch(),
    checkS3(),
    checkExternalAPI(),
  ]);
  
  const names = ['database', 'redis', 'elasticsearch', 's3', 'external-api'];
  const critical = ['database', 'redis'];
  
  const results = checks.map((result, i) => ({
    name: names[i],
    critical: critical.includes(names[i]),
    status: result.status === 'fulfilled' ? 'healthy' : 'unhealthy',
    error: result.status === 'rejected' ? (result.reason as Error).message : undefined,
  }));
  
  const criticalHealthy = results
    .filter(r => r.critical)
    .every(r => r.status === 'healthy');
  
  const allHealthy = results.every(r => r.status === 'healthy');
  
  res.status(criticalHealthy ? 200 : 503).json({
    status: allHealthy ? 'healthy' : criticalHealthy ? 'degraded' : 'unhealthy',
    latency: Date.now() - start,
    checks: results,
  });
});

export function setAppInitialized(): void {
  appInitialized = true;
}

export { router as healthRouter };

// Mock implementations
async function checkDatabase(): Promise<void> {}
async function checkRedis(): Promise<void> {}
async function checkElasticsearch(): Promise<void> {}
async function checkS3(): Promise<void> {}
async function checkExternalAPI(): Promise<void> {}
```

---

## Common Pitfalls

### 1. Slow Health Checks Causing Cascading Failures

```typescript
// ❌ BAD: Slow health check with no timeout
router.get('/health', async (req, res) => {
  await db.query('SELECT COUNT(*) FROM large_table'); // Could take 30s!
  res.status(200).send('OK');
});

// ✅ GOOD: Fast health check with timeout
router.get('/health', async (req, res) => {
  try {
    await Promise.race([
      db.query('SELECT 1'), // Simple query
      new Promise((_, reject) => 
        setTimeout(() => reject(new Error('Timeout')), 3000)
      ),
    ]);
    res.status(200).send('OK');
  } catch {
    res.status(503).send('Unhealthy');
  }
});
```

### 2. Health Check Authentication

```typescript
// ❌ BAD: Health checks require authentication
app.use(authMiddleware); // Applied to ALL routes including health
app.get('/health', healthHandler);

// ✅ GOOD: Health checks bypass authentication
app.get('/health', healthHandler); // BEFORE auth middleware
app.use(authMiddleware);
// ... other routes
```

### 3. Not Differentiating Liveness and Readiness

```typescript
// ❌ BAD: Single health check for everything
router.get('/health', async (req, res) => {
  await checkDatabase();
  await checkRedis();
  await checkExternalAPI();
  res.status(200).send('OK');
});

// ✅ GOOD: Separate probes for different purposes
router.get('/health/live', (req, res) => {
  // Just check if process is running
  res.status(200).send('OK');
});

router.get('/health/ready', async (req, res) => {
  // Check critical dependencies
  try {
    await checkDatabase();
    await checkRedis();
    res.status(200).send('OK');
  } catch {
    res.status(503).send('Not Ready');
  }
});
```

---

## Interview Questions

### Q1: What's the difference between liveness, readiness, and startup probes?

**A:** 
- **Liveness**: Checks if the process is running and not deadlocked. Failure triggers container restart.
- **Readiness**: Checks if the app can handle requests. Failure removes the pod from the service but doesn't restart.
- **Startup**: Checks if the app has finished initializing. Disables liveness/readiness probes until successful. Useful for slow-starting apps.

### Q2: Why shouldn't health checks include heavy database queries?

**A:** Slow health checks can cause cascading failures. If the database is slow, health checks timeout, instances are marked unhealthy and removed, remaining instances get more load, their health checks also timeout, and the entire service can go down. Health checks should be fast (< 5s) with simple queries like `SELECT 1`.

### Q3: Should readiness probes check all external dependencies?

**A:** Only check critical dependencies that would make the app unable to serve requests. Don't check non-critical services in readiness probes - if your recommendation service is down, you can still serve the main functionality. Use separate monitoring for non-critical dependencies.

### Q4: How do you handle flapping health checks?

**A:** 
1. Use `failureThreshold` > 1 to require multiple consecutive failures
2. Implement circuit breakers for external dependencies
3. Cache health check results for short periods
4. Add jitter to check intervals to prevent thundering herd
5. Use exponential backoff for recovery checks

---

## Quick Reference Checklist

### Probe Types
- [ ] Liveness: Is the process alive? (fast, no dependencies)
- [ ] Readiness: Can it handle traffic? (check critical deps)
- [ ] Startup: Has it finished initializing? (slow-starting apps)

### Best Practices
- [ ] Keep health checks fast (< 5s)
- [ ] Use timeouts on all checks
- [ ] Separate critical and non-critical dependencies
- [ ] Cache results for frequently polled endpoints
- [ ] Skip authentication for health endpoints

### Kubernetes Configuration
- [ ] Set appropriate `failureThreshold`
- [ ] Configure `timeoutSeconds` for each probe
- [ ] Use startup probes for slow-starting apps
- [ ] Match probe intervals to app behavior

### Monitoring
- [ ] Alert on health check failures
- [ ] Track health check latency
- [ ] Monitor dependency health separately
- [ ] Log health check results

---

*Last updated: February 2026*

