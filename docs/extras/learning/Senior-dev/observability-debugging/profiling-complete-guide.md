# Profiling - Complete Guide

> **MUST REMEMBER**: Profiling measures where your application spends time (CPU) and memory. CPU profiling shows which functions consume the most time; memory profiling shows allocation patterns and leaks. Use flame graphs to visualize call stacks. Profile in production with sampling to minimize overhead. Always profile before optimizing.

---

## How to Explain Like a Senior Developer

"Profiling answers 'where is my code slow?' or 'why is memory growing?'. CPU profilers sample the call stack periodically - if a function appears in 30% of samples, it uses ~30% of CPU. Memory profilers track allocations to find leaks. Flame graphs are the key visualization - the wider a bar, the more time spent there. The important thing is profiling in production because local benchmarks often miss real-world patterns. Use sampling profilers (low overhead) rather than instrumentation profilers. And the golden rule: profile first, optimize second. Don't guess where the bottleneck is."

---

## Core Implementation

### CPU Profiling with Node.js Inspector

```typescript
// profiling/cpu-profiler.ts
import * as inspector from 'inspector';
import * as fs from 'fs';
import * as path from 'path';

const session = new inspector.Session();

export class CPUProfiler {
  private isEnabled = false;
  
  start(): void {
    if (this.isEnabled) return;
    
    session.connect();
    session.post('Profiler.enable');
    session.post('Profiler.start');
    
    this.isEnabled = true;
    console.log('CPU profiler started');
  }
  
  async stop(): Promise<string> {
    if (!this.isEnabled) {
      throw new Error('Profiler not started');
    }
    
    return new Promise((resolve, reject) => {
      session.post('Profiler.stop', (err, { profile }) => {
        if (err) {
          reject(err);
          return;
        }
        
        // Save profile
        const filename = `cpu-profile-${Date.now()}.cpuprofile`;
        const filepath = path.join('/tmp', filename);
        
        fs.writeFileSync(filepath, JSON.stringify(profile));
        
        session.post('Profiler.disable');
        session.disconnect();
        
        this.isEnabled = false;
        console.log(`CPU profile saved to ${filepath}`);
        
        resolve(filepath);
      });
    });
  }
}

// Express endpoints for on-demand profiling
import { Router } from 'express';

const router = Router();
const profiler = new CPUProfiler();

router.post('/debug/profile/start', (req, res) => {
  try {
    profiler.start();
    res.json({ status: 'profiling started' });
  } catch (error) {
    res.status(400).json({ error: (error as Error).message });
  }
});

router.post('/debug/profile/stop', async (req, res) => {
  try {
    const filepath = await profiler.stop();
    res.json({ status: 'profiling stopped', file: filepath });
  } catch (error) {
    res.status(400).json({ error: (error as Error).message });
  }
});

export { router as profilingRouter };
```

### Automatic Profile Capture on High CPU

```typescript
// profiling/auto-profiler.ts
import * as inspector from 'inspector';
import * as fs from 'fs';
import * as os from 'os';

const session = new inspector.Session();
let isProfileRunning = false;

interface AutoProfilerConfig {
  cpuThreshold: number;    // CPU % to trigger (e.g., 80)
  durationMs: number;      // How long to profile
  cooldownMs: number;      // Minimum time between profiles
  outputDir: string;
}

export class AutoCPUProfiler {
  private lastProfileTime = 0;
  private checkInterval: NodeJS.Timeout | null = null;
  
  constructor(private config: AutoProfilerConfig) {}
  
  start(): void {
    // Check CPU every 5 seconds
    this.checkInterval = setInterval(() => this.checkCPU(), 5000);
    console.log('Auto CPU profiler started');
  }
  
  stop(): void {
    if (this.checkInterval) {
      clearInterval(this.checkInterval);
      this.checkInterval = null;
    }
  }
  
  private getCPUUsage(): number {
    const cpus = os.cpus();
    let totalIdle = 0;
    let totalTick = 0;
    
    cpus.forEach(cpu => {
      for (const type in cpu.times) {
        totalTick += cpu.times[type as keyof typeof cpu.times];
      }
      totalIdle += cpu.times.idle;
    });
    
    return 100 - (100 * totalIdle / totalTick);
  }
  
  private async checkCPU(): Promise<void> {
    if (isProfileRunning) return;
    
    const cpuUsage = this.getCPUUsage();
    const now = Date.now();
    
    // Check if CPU is high and cooldown has passed
    if (
      cpuUsage > this.config.cpuThreshold &&
      now - this.lastProfileTime > this.config.cooldownMs
    ) {
      console.log(`High CPU detected (${cpuUsage.toFixed(1)}%), starting profile`);
      await this.captureProfile();
      this.lastProfileTime = now;
    }
  }
  
  private async captureProfile(): Promise<void> {
    isProfileRunning = true;
    
    try {
      session.connect();
      session.post('Profiler.enable');
      session.post('Profiler.start');
      
      // Profile for configured duration
      await new Promise(resolve => setTimeout(resolve, this.config.durationMs));
      
      session.post('Profiler.stop', (err, { profile }) => {
        if (err) {
          console.error('Profile error:', err);
          return;
        }
        
        const filename = `auto-cpu-${Date.now()}.cpuprofile`;
        const filepath = `${this.config.outputDir}/${filename}`;
        
        fs.writeFileSync(filepath, JSON.stringify(profile));
        console.log(`Auto profile saved: ${filepath}`);
      });
      
      session.post('Profiler.disable');
      session.disconnect();
    } finally {
      isProfileRunning = false;
    }
  }
}

// Usage
const autoProfiler = new AutoCPUProfiler({
  cpuThreshold: 80,
  durationMs: 30000,
  cooldownMs: 300000, // 5 minutes
  outputDir: '/var/log/profiles',
});

autoProfiler.start();
```

### Memory Profiling

```typescript
// profiling/memory-profiler.ts
import * as inspector from 'inspector';
import * as v8 from 'v8';
import * as fs from 'fs';

const session = new inspector.Session();

export class MemoryProfiler {
  // Take heap snapshot
  async takeHeapSnapshot(): Promise<string> {
    return new Promise((resolve, reject) => {
      session.connect();
      
      const chunks: string[] = [];
      
      session.on('HeapProfiler.addHeapSnapshotChunk', (message) => {
        chunks.push(message.params.chunk);
      });
      
      session.post('HeapProfiler.takeHeapSnapshot', undefined, (err) => {
        session.disconnect();
        
        if (err) {
          reject(err);
          return;
        }
        
        const filename = `heap-${Date.now()}.heapsnapshot`;
        const filepath = `/tmp/${filename}`;
        
        fs.writeFileSync(filepath, chunks.join(''));
        console.log(`Heap snapshot saved to ${filepath}`);
        
        resolve(filepath);
      });
    });
  }
  
  // Get heap statistics
  getHeapStats(): v8.HeapInfo {
    return v8.getHeapStatistics();
  }
  
  // Get detailed heap space info
  getHeapSpaceStats(): v8.HeapSpaceInfo[] {
    return v8.getHeapSpaceStatistics();
  }
  
  // Track allocations (start sampling)
  startAllocationTracking(): void {
    session.connect();
    session.post('HeapProfiler.enable');
    session.post('HeapProfiler.startSampling', {
      samplingInterval: 32768, // bytes
    });
    console.log('Allocation tracking started');
  }
  
  // Stop tracking and get allocation profile
  async stopAllocationTracking(): Promise<string> {
    return new Promise((resolve, reject) => {
      session.post('HeapProfiler.stopSampling', (err, result) => {
        session.post('HeapProfiler.disable');
        session.disconnect();
        
        if (err) {
          reject(err);
          return;
        }
        
        const filename = `allocation-${Date.now()}.heapprofile`;
        const filepath = `/tmp/${filename}`;
        
        fs.writeFileSync(filepath, JSON.stringify(result.profile));
        console.log(`Allocation profile saved to ${filepath}`);
        
        resolve(filepath);
      });
    });
  }
}

// Memory monitoring
export function startMemoryMonitoring(
  intervalMs: number = 60000,
  warnThreshold: number = 0.8 // 80% of heap
): void {
  setInterval(() => {
    const stats = v8.getHeapStatistics();
    const usedRatio = stats.used_heap_size / stats.heap_size_limit;
    
    console.log('Memory stats:', {
      used: `${(stats.used_heap_size / 1024 / 1024).toFixed(2)} MB`,
      total: `${(stats.total_heap_size / 1024 / 1024).toFixed(2)} MB`,
      limit: `${(stats.heap_size_limit / 1024 / 1024).toFixed(2)} MB`,
      usedPercent: `${(usedRatio * 100).toFixed(1)}%`,
    });
    
    if (usedRatio > warnThreshold) {
      console.warn('HIGH MEMORY USAGE:', {
        usedPercent: `${(usedRatio * 100).toFixed(1)}%`,
      });
    }
  }, intervalMs);
}
```

### Flame Graph Generation

```typescript
// profiling/flamegraph.ts
import { execSync } from 'child_process';

/**
 * Convert Node.js CPU profile to flame graph
 * 
 * Prerequisites:
 * npm install -g 0x
 * or
 * npm install -g flamebearer
 */

export function generateFlameGraph(
  profilePath: string,
  outputPath: string
): void {
  // Using flamebearer
  try {
    execSync(`flamebearer ${profilePath}`, {
      cwd: outputPath,
    });
    console.log(`Flame graph generated at ${outputPath}/flamegraph.html`);
  } catch (error) {
    console.error('Failed to generate flame graph:', error);
  }
}

// Using 0x for automatic profiling with flame graphs
/*
npx 0x -- node server.js

This automatically:
1. Profiles your application
2. Generates an interactive flame graph
3. Opens it in browser when you stop the server
*/

// Programmatic flame graph with clinic.js
/*
npx clinic flame -- node server.js

Or for specific profiling:
npx clinic doctor -- node server.js
npx clinic bubbleprof -- node server.js  (for async issues)
*/
```

### Production Profiling with pprof

```typescript
// profiling/pprof.ts
import * as pprof from 'pprof';

// Start CPU profiling with sampling
let profiling = false;

export async function startProfiling(): Promise<void> {
  if (profiling) return;
  profiling = true;
  
  // Start collecting samples
  await pprof.time.start({
    durationMillis: 30000, // 30 seconds
    intervalMicros: 1000,  // Sample every 1ms
  });
}

export async function stopProfiling(): Promise<Buffer> {
  if (!profiling) throw new Error('Not profiling');
  
  const profile = await pprof.time.stop();
  profiling = false;
  
  // Return pprof-formatted profile
  return pprof.encode(profile);
}

// Express endpoint for pprof-compatible tools
import { Router, Response } from 'express';

const pprofRouter = Router();

pprofRouter.get('/debug/pprof/profile', async (req, res: Response) => {
  const seconds = parseInt(req.query.seconds as string) || 30;
  
  try {
    const profile = await pprof.time.profile({
      durationMillis: seconds * 1000,
    });
    
    const encoded = await pprof.encode(profile);
    
    res.set('Content-Type', 'application/octet-stream');
    res.send(encoded);
  } catch (error) {
    res.status(500).json({ error: (error as Error).message });
  }
});

pprofRouter.get('/debug/pprof/heap', async (req, res: Response) => {
  try {
    const profile = await pprof.heap.profile();
    const encoded = await pprof.encode(profile);
    
    res.set('Content-Type', 'application/octet-stream');
    res.send(encoded);
  } catch (error) {
    res.status(500).json({ error: (error as Error).message });
  }
});

export { pprofRouter };
```

---

## Real-World Scenarios

### Scenario 1: Finding Memory Leaks

```typescript
// leak-detection.ts
import * as v8 from 'v8';

interface MemorySample {
  timestamp: Date;
  heapUsed: number;
  heapTotal: number;
  external: number;
}

class MemoryLeakDetector {
  private samples: MemorySample[] = [];
  private readonly maxSamples = 60; // 1 hour at 1-minute intervals
  private interval: NodeJS.Timeout | null = null;
  
  start(intervalMs: number = 60000): void {
    this.interval = setInterval(() => {
      this.collectSample();
      this.analyzeForLeak();
    }, intervalMs);
  }
  
  stop(): void {
    if (this.interval) {
      clearInterval(this.interval);
      this.interval = null;
    }
  }
  
  private collectSample(): void {
    const stats = v8.getHeapStatistics();
    
    this.samples.push({
      timestamp: new Date(),
      heapUsed: stats.used_heap_size,
      heapTotal: stats.total_heap_size,
      external: stats.external_memory,
    });
    
    // Keep only recent samples
    if (this.samples.length > this.maxSamples) {
      this.samples.shift();
    }
  }
  
  private analyzeForLeak(): void {
    if (this.samples.length < 10) return;
    
    // Calculate trend using linear regression
    const trend = this.calculateTrend();
    
    // If heap is growing > 1MB per minute on average, likely leak
    const growthPerMinute = trend * 60000;
    const growthMB = growthPerMinute / (1024 * 1024);
    
    if (growthMB > 1) {
      console.warn('POTENTIAL MEMORY LEAK DETECTED', {
        growthPerMinute: `${growthMB.toFixed(2)} MB/min`,
        currentHeap: `${(this.samples[this.samples.length - 1].heapUsed / 1024 / 1024).toFixed(2)} MB`,
      });
      
      // Could trigger heap snapshot here
    }
  }
  
  private calculateTrend(): number {
    const n = this.samples.length;
    let sumX = 0, sumY = 0, sumXY = 0, sumX2 = 0;
    
    this.samples.forEach((sample, i) => {
      const x = i;
      const y = sample.heapUsed;
      sumX += x;
      sumY += y;
      sumXY += x * y;
      sumX2 += x * x;
    });
    
    // Slope of linear regression (bytes per sample)
    return (n * sumXY - sumX * sumY) / (n * sumX2 - sumX * sumX);
  }
  
  getReport(): object {
    const latest = this.samples[this.samples.length - 1];
    const oldest = this.samples[0];
    
    return {
      samples: this.samples.length,
      duration: latest ? latest.timestamp.getTime() - oldest.timestamp.getTime() : 0,
      currentHeapMB: latest ? (latest.heapUsed / 1024 / 1024).toFixed(2) : 0,
      growthMB: latest && oldest 
        ? ((latest.heapUsed - oldest.heapUsed) / 1024 / 1024).toFixed(2) 
        : 0,
    };
  }
}

export const leakDetector = new MemoryLeakDetector();
```

### Scenario 2: Performance Regression Testing

```typescript
// performance-test.ts
import { performance, PerformanceObserver } from 'perf_hooks';

interface PerformanceBaseline {
  name: string;
  p50: number;
  p95: number;
  p99: number;
}

const baselines: Map<string, PerformanceBaseline> = new Map();
const measurements: Map<string, number[]> = new Map();

// Measure function execution time
export function measurePerformance<T>(
  name: string,
  fn: () => T | Promise<T>
): T | Promise<T> {
  const start = performance.now();
  
  const result = fn();
  
  if (result instanceof Promise) {
    return result.finally(() => {
      recordMeasurement(name, performance.now() - start);
    });
  }
  
  recordMeasurement(name, performance.now() - start);
  return result;
}

function recordMeasurement(name: string, duration: number): void {
  if (!measurements.has(name)) {
    measurements.set(name, []);
  }
  
  const samples = measurements.get(name)!;
  samples.push(duration);
  
  // Keep last 1000 samples
  if (samples.length > 1000) {
    samples.shift();
  }
  
  // Check against baseline
  const baseline = baselines.get(name);
  if (baseline) {
    checkRegression(name, duration, baseline);
  }
}

function calculatePercentile(samples: number[], percentile: number): number {
  const sorted = [...samples].sort((a, b) => a - b);
  const index = Math.ceil((percentile / 100) * sorted.length) - 1;
  return sorted[index];
}

function checkRegression(
  name: string,
  duration: number,
  baseline: PerformanceBaseline
): void {
  // Alert if more than 50% slower than p95 baseline
  if (duration > baseline.p95 * 1.5) {
    console.warn(`PERFORMANCE REGRESSION: ${name}`, {
      current: `${duration.toFixed(2)}ms`,
      baselineP95: `${baseline.p95.toFixed(2)}ms`,
      regression: `${((duration / baseline.p95 - 1) * 100).toFixed(1)}%`,
    });
  }
}

// Set baseline from current measurements
export function setBaseline(name: string): void {
  const samples = measurements.get(name);
  if (!samples || samples.length < 100) {
    throw new Error(`Need at least 100 samples, have ${samples?.length || 0}`);
  }
  
  baselines.set(name, {
    name,
    p50: calculatePercentile(samples, 50),
    p95: calculatePercentile(samples, 95),
    p99: calculatePercentile(samples, 99),
  });
}

// Get current stats
export function getStats(name: string): object | null {
  const samples = measurements.get(name);
  if (!samples || samples.length === 0) return null;
  
  return {
    name,
    samples: samples.length,
    p50: calculatePercentile(samples, 50).toFixed(2),
    p95: calculatePercentile(samples, 95).toFixed(2),
    p99: calculatePercentile(samples, 99).toFixed(2),
    baseline: baselines.get(name),
  };
}
```

---

## Common Pitfalls

### 1. Profiling Only in Development

```typescript
// ❌ BAD: Only local profiling
if (process.env.NODE_ENV === 'development') {
  startProfiler();
}

// ✅ GOOD: Production profiling with safeguards
if (process.env.ENABLE_PROFILING === 'true') {
  startSamplingProfiler({
    sampleRate: 0.01, // 1% of requests
    maxDuration: 30000, // Max 30 seconds
    authRequired: true, // Require admin auth
  });
}

function startProfiler(): void {}
function startSamplingProfiler(config: object): void {}
```

### 2. High-Overhead Profiling in Production

```typescript
// ❌ BAD: Full instrumentation profiler
const profiler = new InstrumentationProfiler(); // 50%+ overhead

// ✅ GOOD: Sampling profiler
const profiler = new SamplingProfiler({
  intervalMicros: 10000, // Sample every 10ms, ~5% overhead
});

class InstrumentationProfiler {}
class SamplingProfiler { constructor(config: object) {} }
```

### 3. Not Correlating Profiles with User Actions

```typescript
// ❌ BAD: Profile without context
startProfile();
// ... traffic ...
const profile = stopProfile(); // What request caused the issue?

// ✅ GOOD: Profile specific operations
router.post('/slow-endpoint', async (req, res) => {
  const profileId = startProfile(`request-${req.id}`);
  try {
    await slowOperation();
    res.json({ success: true });
  } finally {
    stopProfile(profileId);
  }
});

function startProfile(id?: string): string { return ''; }
function stopProfile(id: string): void {}
async function slowOperation(): Promise<void> {}
const router = { post: (path: string, handler: Function) => {} };
```

---

## Interview Questions

### Q1: What's the difference between CPU profiling and memory profiling?

**A:** CPU profiling measures where time is spent executing code - which functions are called and how long they take. Memory profiling measures allocations - how much memory is used, what objects exist, and identifies leaks. Use CPU profiling for slow code, memory profiling for memory issues.

### Q2: How do you read a flame graph?

**A:** The x-axis shows the call stack (wider = more time). The y-axis shows stack depth (callee above caller). Look for wide bars - they indicate functions that consume the most time. The root is at the bottom. Colors are usually random for readability. Focus on the "plateaus" - functions that take significant time themselves rather than just calling other functions.

### Q3: How do you profile in production safely?

**A:** Use sampling profilers (low overhead, ~5%). Profile for short durations (30s). Trigger based on signals (high CPU, specific endpoint). Sample a small percentage of requests. Protect profiling endpoints with authentication. Have automatic shutoff after time limit.

### Q4: How do you identify a memory leak?

**A:** Monitor heap size over time - continuous growth indicates a leak. Take heap snapshots at different times and compare. Look for objects that increase in count between snapshots. Common causes: event listeners not removed, closures holding references, caches without eviction, timers not cleared.

---

## Quick Reference Checklist

### CPU Profiling
- [ ] Use sampling profiler (low overhead)
- [ ] Profile for short durations
- [ ] Generate flame graphs for visualization
- [ ] Correlate with specific requests

### Memory Profiling
- [ ] Monitor heap size over time
- [ ] Take snapshots for comparison
- [ ] Look for growing object counts
- [ ] Check for common leak patterns

### Production Safety
- [ ] Authenticate profiling endpoints
- [ ] Set time limits
- [ ] Use sampling, not instrumentation
- [ ] Monitor profiler overhead

### Tools
- [ ] Chrome DevTools for local profiling
- [ ] 0x / clinic.js for flame graphs
- [ ] pprof for production profiling
- [ ] heapdump for memory snapshots

---

*Last updated: February 2026*

