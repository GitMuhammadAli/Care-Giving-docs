# 🌪️ Chaos Engineering - Complete Guide

> A comprehensive guide to chaos engineering - failure injection, resilience testing, game days, and building systems that thrive under adverse conditions.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Chaos engineering is the discipline of experimenting on a distributed system to build confidence in its ability to withstand turbulent conditions in production - proactively finding weaknesses before they cause outages."

### The 7 Key Concepts (Remember These!)
```
1. HYPOTHESIS        → Expected behavior under failure
2. BLAST RADIUS      → Scope of impact from experiment
3. STEADY STATE      → Normal system behavior metrics
4. FAILURE INJECTION → Deliberately causing failures
5. GAME DAY          → Scheduled chaos experiments
6. CIRCUIT BREAKER   → Protection against cascading failure
7. GRACEFUL DEGRADATION → System continues partially during failures
```

### The Chaos Engineering Cycle
```
┌─────────────────────────────────────────────────────────────────┐
│                CHAOS ENGINEERING CYCLE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. DEFINE STEADY STATE                                        │
│     ┌──────────────────────────────────────────────┐           │
│     │ What does "working normally" look like?      │           │
│     │ • Response time < 200ms                      │           │
│     │ • Error rate < 0.1%                          │           │
│     │ • Orders processing successfully             │           │
│     └──────────────────────────────────────────────┘           │
│                            │                                    │
│                            ▼                                    │
│  2. HYPOTHESIZE                                                │
│     ┌──────────────────────────────────────────────┐           │
│     │ "If database replica fails, system should    │           │
│     │  failover within 30 seconds without errors"  │           │
│     └──────────────────────────────────────────────┘           │
│                            │                                    │
│                            ▼                                    │
│  3. DESIGN EXPERIMENT                                          │
│     ┌──────────────────────────────────────────────┐           │
│     │ • Kill database replica                      │           │
│     │ • Monitor: response time, errors, failover   │           │
│     │ • Blast radius: read queries only            │           │
│     │ • Abort conditions: >5% error rate           │           │
│     └──────────────────────────────────────────────┘           │
│                            │                                    │
│                            ▼                                    │
│  4. RUN EXPERIMENT                                             │
│     ┌──────────────────────────────────────────────┐           │
│     │ Execute in production (with safeguards)      │           │
│     │ Or staging that mirrors production           │           │
│     └──────────────────────────────────────────────┘           │
│                            │                                    │
│                            ▼                                    │
│  5. ANALYZE & IMPROVE                                          │
│     ┌──────────────────────────────────────────────┐           │
│     │ • Did hypothesis hold?                       │           │
│     │ • What broke? What worked?                   │           │
│     │ • Fix weaknesses found                       │           │
│     │ • Document learnings                         │           │
│     └──────────────────────────────────────────────┘           │
│                            │                                    │
│                            └─────────► Repeat                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Types of Chaos Experiments
```
┌─────────────────────────────────────────────────────────────────┐
│                  CHAOS EXPERIMENT TYPES                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  INFRASTRUCTURE FAILURES                                       │
│  • Kill server/container/pod                                   │
│  • Network partition between services                          │
│  • Disk full / IO errors                                       │
│  • DNS failures                                                │
│  • Region/zone outage                                          │
│                                                                 │
│  APPLICATION FAILURES                                          │
│  • Service crashes                                             │
│  • Memory leaks / OOM                                          │
│  • Thread pool exhaustion                                      │
│  • Connection pool exhaustion                                  │
│                                                                 │
│  DEPENDENCY FAILURES                                           │
│  • Database unavailable                                        │
│  • Cache miss (Redis down)                                     │
│  • Third-party API timeout                                     │
│  • Message queue unavailable                                   │
│                                                                 │
│  NETWORK CHAOS                                                 │
│  • Latency injection (100ms, 500ms, 2s)                        │
│  • Packet loss (1%, 5%, 20%)                                   │
│  • Bandwidth throttling                                        │
│  • Connection timeouts                                         │
│                                                                 │
│  RESOURCE EXHAUSTION                                           │
│  • CPU stress (100% usage)                                     │
│  • Memory pressure                                             │
│  • File descriptor exhaustion                                  │
│  • Port exhaustion                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Blast radius"** | "We limit blast radius to 5% of traffic initially" |
| **"Steady state"** | "Monitor steady state metrics during experiments" |
| **"Game day"** | "We run quarterly game days to test resilience" |
| **"Chaos Monkey"** | "We use Chaos Monkey to randomly kill instances" |
| **"Failure injection"** | "Failure injection revealed gaps in our retry logic" |
| **"Graceful degradation"** | "System should degrade gracefully, not fail completely" |

### Key Numbers to Remember
| Metric | Guideline | Notes |
|--------|-----------|-------|
| Initial blast radius | **1-5%** | Start small |
| Experiment duration | **5-30 minutes** | Time-boxed |
| Game day frequency | **Monthly/Quarterly** | Regular practice |
| Recovery time | **< 30 seconds** | Automatic recovery |
| Error budget | **Define per service** | Max acceptable errors |

### The "Wow" Statement (Memorize This!)
> "We practice chaos engineering with monthly game days and continuous experiments. We use Chaos Monkey to randomly terminate instances and Gremlin for controlled experiments - network latency, CPU stress, dependency failures. Before running in production, we define hypotheses like 'If payment service fails, orders queue for retry within 30 seconds.' We start with 1% blast radius, monitor steady-state metrics, and have automatic abort conditions. Last quarter's game day found our circuit breaker timeout was too aggressive - we fixed it before it caused a real outage. Chaos engineering has shifted our mindset from 'hope it works' to 'prove it works under failure.' Our MTTR improved by 60% because teams now understand failure modes."

---

## 📚 Table of Contents

1. [Chaos Tools](#1-chaos-tools)
2. [Designing Experiments](#2-designing-experiments)
3. [Game Days](#3-game-days)
4. [Implementation Patterns](#4-implementation-patterns)
5. [Monitoring & Observability](#5-monitoring--observability)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Chaos Tools

```yaml
# ════════════════════════════════════════════════════════════════
# CHAOS TOOLS OVERVIEW
# ════════════════════════════════════════════════════════════════

tools:
  chaos_monkey:
    platform: Netflix / AWS
    purpose: Randomly terminate instances
    scope: EC2, containers
    
  gremlin:
    platform: SaaS
    purpose: Comprehensive chaos platform
    features:
      - Network attacks (latency, packet loss)
      - Resource attacks (CPU, memory, disk)
      - State attacks (process kill, shutdown)
    
  litmus:
    platform: Kubernetes
    purpose: Cloud-native chaos engineering
    features:
      - Pod failures
      - Node failures
      - Network chaos
      
  chaos_mesh:
    platform: Kubernetes
    purpose: Chaos orchestration
    features:
      - Pod chaos
      - Network chaos
      - Stress testing
      
  toxiproxy:
    platform: Application level
    purpose: Network condition simulation
    features:
      - Latency
      - Bandwidth
      - Slow close
      - Timeout

# ════════════════════════════════════════════════════════════════
# CHAOS MESH CONFIGURATION (KUBERNETES)
# ════════════════════════════════════════════════════════════════
```

```yaml
# chaos-mesh/pod-failure.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: pod-failure-experiment
  namespace: chaos-testing
spec:
  action: pod-failure
  mode: one  # Kill one pod
  selector:
    namespaces:
      - production
    labelSelectors:
      app: order-service
  duration: "60s"
  scheduler:
    cron: "@every 1h"  # Run every hour

---
# Network latency injection
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: network-delay
spec:
  action: delay
  mode: all
  selector:
    namespaces:
      - production
    labelSelectors:
      app: payment-service
  delay:
    latency: "200ms"
    jitter: "50ms"
    correlation: "25"
  duration: "5m"
  direction: to
  target:
    selector:
      namespaces:
        - production
      labelSelectors:
        app: database

---
# CPU stress test
apiVersion: chaos-mesh.org/v1alpha1
kind: StressChaos
metadata:
  name: cpu-stress
spec:
  mode: one
  selector:
    labelSelectors:
      app: api-gateway
  stressors:
    cpu:
      workers: 4
      load: 80  # 80% CPU
  duration: "10m"
```

```typescript
// ════════════════════════════════════════════════════════════════
// TOXIPROXY FOR LOCAL/INTEGRATION TESTING
// ════════════════════════════════════════════════════════════════

import { Toxiproxy, Proxy, Toxic } from 'toxiproxy-node-client';

const toxiproxy = new Toxiproxy('http://localhost:8474');

async function setupChaosProxy() {
  // Create proxy for database connection
  const dbProxy = await toxiproxy.createProxy({
    name: 'postgres',
    listen: '0.0.0.0:15432',
    upstream: 'postgres:5432',
  });

  return dbProxy;
}

describe('Resilience Tests', () => {
  let dbProxy: Proxy;

  beforeAll(async () => {
    dbProxy = await setupChaosProxy();
  });

  it('handles database latency gracefully', async () => {
    // Add 500ms latency
    await dbProxy.addToxic(new Toxic({
      type: 'latency',
      attributes: { latency: 500, jitter: 100 },
    }));

    const startTime = Date.now();
    const result = await orderService.getOrders();
    const duration = Date.now() - startTime;

    expect(result).toBeDefined();
    expect(duration).toBeGreaterThan(500);
    expect(duration).toBeLessThan(2000); // Timeout not exceeded

    // Remove toxic
    await dbProxy.removeToxic('latency');
  });

  it('handles database connection failure', async () => {
    // Simulate connection timeout
    await dbProxy.addToxic(new Toxic({
      type: 'timeout',
      attributes: { timeout: 0 }, // Immediate timeout
    }));

    // Should fall back to cache or return error gracefully
    const result = await orderService.getOrders();
    
    expect(result.source).toBe('cache'); // Fallback worked
    // Or expect graceful error
    
    await dbProxy.removeToxic('timeout');
  });

  it('handles packet loss', async () => {
    await dbProxy.addToxic(new Toxic({
      type: 'bandwidth',
      attributes: { rate: 1000 }, // 1KB/s
    }));

    // Service should handle slow responses
    const result = await orderService.createOrder(orderData);
    expect(result.id).toBeDefined();

    await dbProxy.removeToxic('bandwidth');
  });
});
```

---

## 2. Designing Experiments

```yaml
# ════════════════════════════════════════════════════════════════
# EXPERIMENT DESIGN TEMPLATE
# ════════════════════════════════════════════════════════════════

experiment:
  name: "Payment Service Dependency Failure"
  date: "2024-01-15"
  owner: "Platform Team"
  
  # 1. STEADY STATE
  steady_state:
    description: "Normal operating conditions"
    metrics:
      - metric: "Order success rate"
        expected: ">= 99.9%"
      - metric: "P99 response time"
        expected: "<= 500ms"
      - metric: "Payment service error rate"
        expected: "<= 0.1%"
  
  # 2. HYPOTHESIS
  hypothesis: |
    When the payment service becomes unavailable for 60 seconds,
    the order service should:
    1. Queue pending orders for retry
    2. Maintain < 1% error rate for order creation
    3. Resume processing within 30 seconds of recovery
    4. No data loss occurs
  
  # 3. EXPERIMENT DESIGN
  method:
    action: "Block network traffic to payment service"
    duration: "60 seconds"
    blast_radius: "5% of order traffic"
    target: "order-service pods"
    
  # 4. ABORT CONDITIONS
  abort_conditions:
    - "Error rate exceeds 5%"
    - "Data loss detected"
    - "System-wide degradation"
    - "Manual abort requested"
  
  # 5. MONITORING
  monitoring:
    dashboards:
      - "Order Service Dashboard"
      - "Payment Integration Dashboard"
    alerts:
      - "Order failure rate > 1%"
      - "Payment queue depth > 1000"
    
  # 6. ROLLBACK
  rollback:
    automatic: true
    trigger: "Any abort condition met"
    action: "Restore network connectivity immediately"
    
  # 7. RESULTS (filled after experiment)
  results:
    hypothesis_validated: true/false
    observations:
      - "Circuit breaker opened after 5 seconds"
      - "Orders queued successfully"
      - "Recovery completed in 25 seconds"
    issues_found:
      - "Retry logic had exponential backoff bug"
      - "Dead letter queue not properly configured"
    action_items:
      - "Fix retry backoff calculation"
      - "Configure DLQ alerts"
      - "Update runbook with findings"
```

```typescript
// ════════════════════════════════════════════════════════════════
// AUTOMATED CHAOS EXPERIMENT
// ════════════════════════════════════════════════════════════════

interface ChaosExperiment {
  name: string;
  hypothesis: string;
  steadyState: SteadyStateCheck[];
  action: ChaosAction;
  duration: number;
  abortConditions: AbortCondition[];
}

class ChaosRunner {
  async runExperiment(experiment: ChaosExperiment): Promise<ExperimentResult> {
    console.log(`Starting experiment: ${experiment.name}`);
    
    // 1. Verify steady state before experiment
    const preCheck = await this.checkSteadyState(experiment.steadyState);
    if (!preCheck.healthy) {
      return { 
        success: false, 
        reason: 'System not in steady state before experiment',
        details: preCheck 
      };
    }
    
    // 2. Start monitoring
    const monitor = this.startMonitoring(experiment);
    
    // 3. Execute chaos action
    console.log('Injecting chaos...');
    await experiment.action.execute();
    
    // 4. Monitor during experiment
    const startTime = Date.now();
    while (Date.now() - startTime < experiment.duration) {
      // Check abort conditions
      const shouldAbort = await this.checkAbortConditions(
        experiment.abortConditions
      );
      
      if (shouldAbort) {
        console.log('Abort condition triggered! Rolling back...');
        await experiment.action.rollback();
        return { 
          success: false, 
          reason: 'Abort condition triggered',
          metrics: monitor.getMetrics() 
        };
      }
      
      await sleep(1000);
    }
    
    // 5. Stop chaos action
    console.log('Stopping chaos...');
    await experiment.action.rollback();
    
    // 6. Verify recovery
    await sleep(30000); // Wait for recovery
    const postCheck = await this.checkSteadyState(experiment.steadyState);
    
    // 7. Collect results
    return {
      success: postCheck.healthy,
      preCheck,
      postCheck,
      metrics: monitor.getMetrics(),
      hypothesis: experiment.hypothesis,
      validated: postCheck.healthy,
    };
  }
  
  private async checkSteadyState(checks: SteadyStateCheck[]): Promise<HealthCheck> {
    const results = await Promise.all(
      checks.map(async (check) => ({
        metric: check.metric,
        expected: check.expected,
        actual: await this.getMetric(check.metric),
        passed: await this.evaluateCheck(check),
      }))
    );
    
    return {
      healthy: results.every(r => r.passed),
      checks: results,
    };
  }
}

// Usage
const experiment: ChaosExperiment = {
  name: 'Redis Failure Resilience',
  hypothesis: 'Service continues with degraded caching when Redis fails',
  steadyState: [
    { metric: 'error_rate', operator: '<', value: 0.01 },
    { metric: 'p99_latency', operator: '<', value: 500 },
  ],
  action: new KillServiceAction('redis', { namespace: 'production' }),
  duration: 60000, // 60 seconds
  abortConditions: [
    { metric: 'error_rate', operator: '>', value: 0.05 },
  ],
};

const runner = new ChaosRunner();
const result = await runner.runExperiment(experiment);
console.log('Experiment result:', result);
```

---

## 3. Game Days

```yaml
# ════════════════════════════════════════════════════════════════
# GAME DAY PLANNING
# ════════════════════════════════════════════════════════════════

game_day:
  name: "Q1 2024 Resilience Game Day"
  date: "2024-03-15"
  duration: "4 hours"
  
  participants:
    facilitators:
      - "SRE Team Lead"
      - "Platform Engineer"
    observers:
      - "Engineering Manager"
      - "On-call engineers"
    support:
      - "Database Admin"
      - "Network Team"
      
  preparation:
    1_week_before:
      - Announce game day to all stakeholders
      - Review and update runbooks
      - Verify monitoring dashboards
      - Prepare rollback procedures
      - Notify customer support
      
    1_day_before:
      - Final review of experiments
      - Verify all participants available
      - Check communication channels
      - Prepare incident bridge
      
  schedule:
    "09:00":
      activity: "Kickoff meeting"
      description: "Review objectives, experiments, abort procedures"
      
    "09:30":
      activity: "Experiment 1: Database failover"
      description: "Kill primary database, verify automatic failover"
      duration: "30 min"
      
    "10:00":
      activity: "Debrief Experiment 1"
      duration: "15 min"
      
    "10:15":
      activity: "Experiment 2: Service mesh failure"
      description: "Inject latency in service mesh"
      duration: "30 min"
      
    "10:45":
      activity: "Debrief Experiment 2"
      duration: "15 min"
      
    "11:00":
      activity: "Experiment 3: Region failover"
      description: "Simulate us-east-1 outage"
      duration: "45 min"
      
    "11:45":
      activity: "Break"
      duration: "15 min"
      
    "12:00":
      activity: "Final debrief & action items"
      duration: "30 min"
      
  communication:
    primary_channel: "#game-day-2024-q1"
    incident_bridge: "zoom.us/gameday"
    escalation: "PagerDuty - Game Day policy"
    
  success_criteria:
    - "All experiments complete without customer impact"
    - "Failovers occur within defined SLOs"
    - "Runbooks accurately reflect procedures"
    - "Monitoring captures all relevant metrics"
    
  abort_criteria:
    - "Any customer-impacting incident"
    - "Unable to rollback within 5 minutes"
    - "Critical system instability"
```

```markdown
# ════════════════════════════════════════════════════════════════
# GAME DAY REPORT TEMPLATE
# ════════════════════════════════════════════════════════════════

# Game Day Report: Q1 2024 Resilience Testing

## Executive Summary
- **Date**: March 15, 2024
- **Duration**: 4 hours
- **Participants**: 12 engineers
- **Experiments**: 3 completed
- **Customer Impact**: None

## Experiment Results

### Experiment 1: Database Failover
- **Hypothesis**: Primary DB failure results in < 30s failover
- **Result**: ✅ PASSED
- **Actual Failover Time**: 18 seconds
- **Observations**:
  - Automatic failover worked as expected
  - Application reconnected without manual intervention
  - 2 queries failed during transition (acceptable)

### Experiment 2: Service Mesh Latency
- **Hypothesis**: 500ms latency doesn't cause cascading failures
- **Result**: ⚠️ PARTIAL
- **Observations**:
  - Circuit breaker opened correctly
  - However, timeout configuration was too aggressive
  - Some requests failed unnecessarily
- **Action Items**:
  - [ ] Increase timeout from 1s to 3s for order service
  - [ ] Add retry logic for idempotent operations

### Experiment 3: Region Failover
- **Hypothesis**: us-east-1 outage results in automatic failover to us-west-2
- **Result**: ✅ PASSED
- **Actual Failover Time**: 45 seconds
- **Observations**:
  - DNS failover worked correctly
  - Some cached DNS caused 2-minute delay for some clients
- **Action Items**:
  - [ ] Reduce DNS TTL from 300s to 60s
  - [ ] Update runbook with DNS propagation notes

## Key Findings

### What Worked Well
1. Automated failover mechanisms performed as designed
2. Alerting fired correctly within defined thresholds
3. Team communication was effective throughout

### Areas for Improvement
1. Service mesh timeout configuration needs tuning
2. DNS TTL too high for rapid failover
3. Runbook missing steps for partial failures

## Action Items

| Item | Owner | Due Date | Priority |
|------|-------|----------|----------|
| Increase service timeout | @john | March 22 | High |
| Reduce DNS TTL | @sarah | March 20 | High |
| Update failover runbook | @team | March 25 | Medium |
| Add retry logic | @mike | March 29 | Medium |

## Next Game Day
- **Date**: June 2024
- **Focus**: Multi-service cascade failure scenarios
```

---

## 4. Implementation Patterns

```typescript
// ════════════════════════════════════════════════════════════════
// RESILIENCE PATTERNS FOR CHAOS
// ════════════════════════════════════════════════════════════════

// Circuit Breaker Pattern
class CircuitBreaker {
  private failures = 0;
  private lastFailureTime = 0;
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  
  constructor(
    private threshold: number = 5,
    private timeout: number = 30000,
    private halfOpenRequests: number = 3
  ) {}

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.timeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }

    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }

  private onFailure() {
    this.failures++;
    this.lastFailureTime = Date.now();
    
    if (this.failures >= this.threshold) {
      this.state = 'OPEN';
    }
  }
}

// Retry with Exponential Backoff
async function withRetry<T>(
  operation: () => Promise<T>,
  options: {
    maxRetries: number;
    baseDelay: number;
    maxDelay: number;
    retryableErrors?: string[];
  }
): Promise<T> {
  let lastError: Error;
  
  for (let attempt = 0; attempt <= options.maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error;
      
      if (!isRetryable(error, options.retryableErrors)) {
        throw error;
      }
      
      if (attempt < options.maxRetries) {
        const delay = Math.min(
          options.baseDelay * Math.pow(2, attempt),
          options.maxDelay
        );
        await sleep(delay + Math.random() * 100); // Jitter
      }
    }
  }
  
  throw lastError!;
}

// Bulkhead Pattern (Isolation)
class Bulkhead {
  private activeRequests = 0;
  private queue: Array<() => void> = [];

  constructor(
    private maxConcurrent: number = 10,
    private maxQueue: number = 100
  ) {}

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.activeRequests >= this.maxConcurrent) {
      if (this.queue.length >= this.maxQueue) {
        throw new Error('Bulkhead queue full - rejecting request');
      }
      
      await new Promise<void>((resolve) => {
        this.queue.push(resolve);
      });
    }

    this.activeRequests++;
    
    try {
      return await operation();
    } finally {
      this.activeRequests--;
      const next = this.queue.shift();
      if (next) next();
    }
  }
}

// Fallback Pattern
class FallbackService {
  async getUser(id: string): Promise<User> {
    try {
      // Primary: Database
      return await this.userRepository.findById(id);
    } catch (error) {
      console.warn('Primary failed, trying cache fallback');
      
      try {
        // Fallback 1: Cache
        return await this.cache.get(`user:${id}`);
      } catch (cacheError) {
        console.warn('Cache failed, trying stale data');
        
        // Fallback 2: Stale data
        const staleData = await this.getStaleUser(id);
        if (staleData) return staleData;
        
        // Fallback 3: Default response
        return this.getDefaultUser(id);
      }
    }
  }
}

// Timeout Pattern
async function withTimeout<T>(
  operation: () => Promise<T>,
  timeoutMs: number
): Promise<T> {
  const timeoutPromise = new Promise<never>((_, reject) => {
    setTimeout(() => reject(new Error('Operation timed out')), timeoutMs);
  });
  
  return Promise.race([operation(), timeoutPromise]);
}

// Health Check for Chaos Detection
class HealthChecker {
  async check(): Promise<HealthStatus> {
    const checks = await Promise.allSettled([
      this.checkDatabase(),
      this.checkCache(),
      this.checkMessageQueue(),
      this.checkExternalApis(),
    ]);

    const results = checks.map((result, index) => ({
      service: ['database', 'cache', 'messageQueue', 'externalApis'][index],
      healthy: result.status === 'fulfilled' && result.value,
      error: result.status === 'rejected' ? result.reason.message : undefined,
    }));

    return {
      healthy: results.every(r => r.healthy),
      degraded: results.some(r => !r.healthy),
      checks: results,
    };
  }
}
```

---

## 5. Monitoring & Observability

```yaml
# ════════════════════════════════════════════════════════════════
# CHAOS MONITORING DASHBOARD
# ════════════════════════════════════════════════════════════════

# Grafana Dashboard for Chaos Experiments
dashboard:
  title: "Chaos Engineering Dashboard"
  
  rows:
    - title: "Steady State Metrics"
      panels:
        - title: "Request Success Rate"
          query: |
            sum(rate(http_requests_total{status=~"2.."}[5m])) /
            sum(rate(http_requests_total[5m])) * 100
          threshold:
            warning: 99.5
            critical: 99.0
            
        - title: "P99 Latency"
          query: |
            histogram_quantile(0.99, 
              rate(http_request_duration_seconds_bucket[5m]))
          threshold:
            warning: 0.5
            critical: 1.0
            
        - title: "Error Rate"
          query: |
            sum(rate(http_requests_total{status=~"5.."}[5m])) /
            sum(rate(http_requests_total[5m])) * 100
          threshold:
            warning: 0.5
            critical: 1.0

    - title: "Resilience Indicators"
      panels:
        - title: "Circuit Breaker State"
          query: "circuit_breaker_state"
          states: ["CLOSED", "OPEN", "HALF_OPEN"]
          
        - title: "Retry Rate"
          query: "sum(rate(retry_attempts_total[5m]))"
          
        - title: "Fallback Activations"
          query: "sum(rate(fallback_activations_total[5m]))"
          
        - title: "Queue Depth"
          query: "dead_letter_queue_depth"

    - title: "Experiment Status"
      panels:
        - title: "Active Experiments"
          query: "chaos_experiment_active"
          
        - title: "Experiment Duration"
          query: "chaos_experiment_duration_seconds"
          
        - title: "Blast Radius"
          query: "chaos_experiment_affected_percentage"
```

```typescript
// ════════════════════════════════════════════════════════════════
// CHAOS METRICS COLLECTION
// ════════════════════════════════════════════════════════════════

import { Counter, Histogram, Gauge } from 'prom-client';

// Circuit breaker metrics
const circuitBreakerState = new Gauge({
  name: 'circuit_breaker_state',
  help: 'Current circuit breaker state (0=closed, 1=open, 2=half-open)',
  labelNames: ['service'],
});

const circuitBreakerTrips = new Counter({
  name: 'circuit_breaker_trips_total',
  help: 'Total number of circuit breaker trips',
  labelNames: ['service'],
});

// Retry metrics
const retryAttempts = new Counter({
  name: 'retry_attempts_total',
  help: 'Total retry attempts',
  labelNames: ['service', 'operation'],
});

// Fallback metrics
const fallbackActivations = new Counter({
  name: 'fallback_activations_total',
  help: 'Total fallback activations',
  labelNames: ['service', 'fallback_type'],
});

// Chaos experiment metrics
const chaosExperimentActive = new Gauge({
  name: 'chaos_experiment_active',
  help: 'Whether a chaos experiment is currently active',
  labelNames: ['experiment_name'],
});

const chaosExperimentDuration = new Histogram({
  name: 'chaos_experiment_duration_seconds',
  help: 'Duration of chaos experiments',
  labelNames: ['experiment_name', 'result'],
  buckets: [60, 300, 600, 1800, 3600],
});

// Instrument your code
class InstrumentedCircuitBreaker extends CircuitBreaker {
  constructor(private serviceName: string) {
    super();
  }

  protected onStateChange(newState: string) {
    circuitBreakerState.set(
      { service: this.serviceName },
      this.stateToNumber(newState)
    );
    
    if (newState === 'OPEN') {
      circuitBreakerTrips.inc({ service: this.serviceName });
    }
  }
}
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# CHAOS ENGINEERING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Starting in production without staging
# Bad
# First chaos experiment directly in production
# No understanding of failure modes

# Good
# Start in development/staging
# Understand system behavior
# Gradually move to production with small blast radius

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: No abort conditions
# Bad
experiment:
  action: "Kill 50% of database connections"
  duration: "30 minutes"
  abort: none  # Just let it run!

# Good
experiment:
  action: "Kill 5% of database connections"
  duration: "5 minutes"
  abort_conditions:
    - "Error rate > 5%"
    - "P99 latency > 2s"
    - "Manual abort requested"

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Too large blast radius
# Bad
# Start with 100% traffic affected
# Major outage results

# Good
# Start with 1-5% traffic
# Gradually increase if hypothesis holds
# Always have rollback ready

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: No hypothesis
# Bad
# "Let's kill some servers and see what happens"
# No clear success/failure criteria

# Good
# "When we kill 1 API server, load balancer should route
#  traffic to healthy servers within 10 seconds with
#  zero failed requests"

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Not fixing what you find
# Bad
# Run experiment, find issues
# "We'll fix it later"
# Never fix it

# Good
# Every finding becomes an action item
# Track to completion
# Verify fix with follow-up experiment

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Chaos without observability
# Bad
# Inject failure
# No metrics, no alerts
# "I think it worked?"

# Good
# Comprehensive monitoring before chaos
# Track steady state metrics
# Alert on deviations
# Record everything for analysis
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is chaos engineering?"**
> "Chaos engineering proactively tests system resilience by intentionally injecting failures - killing servers, adding latency, breaking dependencies. It's about finding weaknesses before they cause outages. Key principle: If you haven't tested a failure mode, you don't know if you can handle it."

**Q: "What is steady state?"**
> "The normal operating behavior of your system, defined by metrics like request success rate (>99.9%), latency (p99 < 500ms), error rate (<0.1%). Before chaos experiments, verify system is in steady state. After experiments, verify it returns to steady state."

**Q: "What is blast radius?"**
> "The scope of impact from a chaos experiment. Start small: 1% of traffic, one server, one region. Limit blast radius to minimize customer impact while still validating hypotheses. Increase gradually as confidence grows."

### Intermediate Questions

**Q: "How do you design a chaos experiment?"**
> "1) Define steady state metrics. 2) Form hypothesis: 'If X fails, Y should happen.' 3) Design experiment: what to break, duration, blast radius. 4) Define abort conditions. 5) Run experiment with monitoring. 6) Analyze results vs hypothesis. 7) Fix findings and repeat."

**Q: "What is a game day?"**
> "Scheduled chaos engineering exercises where teams practice incident response. Run multiple experiments in controlled setting. Practice: failovers, communication, runbooks. Usually monthly or quarterly. Purpose: build confidence, find gaps, train teams. Safer than random production chaos."

**Q: "What tools do you use for chaos engineering?"**
> "Chaos Monkey: Random instance termination. Gremlin: Comprehensive platform for network, resource, state attacks. Chaos Mesh: Kubernetes-native chaos. Toxiproxy: Network condition simulation for testing. LitmusChaos: Cloud-native experiments. Choice depends on infrastructure."

### Advanced Questions

**Q: "How do you do chaos in production safely?"**
> "Start with extensive staging testing. Small blast radius (1-5%). Automatic abort conditions. Real-time monitoring. Rollback ready. Run during low-traffic periods initially. Team on standby. Inform stakeholders. Gradually increase scope as confidence builds. Never chaos without observability."

**Q: "How does chaos engineering relate to resilience patterns?"**
> "Chaos engineering validates resilience patterns work. Circuit breakers: Test they open under failure. Retries: Test they don't amplify problems. Timeouts: Test they're configured correctly. Fallbacks: Test they activate properly. Bulkheads: Test isolation holds. Chaos proves patterns work in practice."

**Q: "How do you measure chaos engineering success?"**
> "Track: MTTR improvement, incidents prevented (found in chaos vs production), time to detect failures, runbook accuracy, team confidence. Success isn't 'nothing broke' - it's 'we found and fixed weakness X before it caused outage Y.' Measure findings per experiment and fix rate."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│               CHAOS ENGINEERING CHECKLIST                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BEFORE EXPERIMENT:                                             │
│  □ Define steady state metrics                                 │
│  □ Form clear hypothesis                                       │
│  □ Set small blast radius (1-5%)                               │
│  □ Define abort conditions                                     │
│  □ Prepare rollback plan                                       │
│  □ Verify monitoring/alerting                                  │
│                                                                 │
│  DURING EXPERIMENT:                                             │
│  □ Monitor steady state metrics                                │
│  □ Watch for abort conditions                                  │
│  □ Document observations                                       │
│  □ Be ready to rollback                                        │
│                                                                 │
│  AFTER EXPERIMENT:                                              │
│  □ Verify return to steady state                               │
│  □ Analyze results vs hypothesis                               │
│  □ Document findings                                           │
│  □ Create action items                                         │
│  □ Fix and verify fixes                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

CHAOS TYPES:
┌────────────────────────────────────────────────────────────────┐
│ Infrastructure: Kill servers, containers, pods                 │
│ Network:        Latency, packet loss, partition               │
│ Application:    OOM, thread exhaustion, crashes                │
│ Dependency:     Database, cache, API failures                  │
│ Resource:       CPU stress, disk full, memory pressure         │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

