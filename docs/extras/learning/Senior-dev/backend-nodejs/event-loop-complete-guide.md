# Event Loop Deep Dive - Complete Guide

> **MUST REMEMBER**: The Node.js event loop is what allows non-blocking I/O operations despite JavaScript being single-threaded. It offloads operations to the system kernel whenever possible and processes callbacks in specific phases with a defined order.

---

## How to Explain Like a Senior Developer

"The event loop is Node.js's secret sauce for handling thousands of concurrent connections with a single thread. Think of it as a really efficient restaurant host - it doesn't cook the food (blocking operations go to the kitchen/OS), but it manages the flow of orders and serves dishes when they're ready. Understanding its phases - timers, pending callbacks, poll, check, close - helps you write performant code and debug mysterious timing issues. The key insight is that `process.nextTick()` and `Promise.then()` (microtasks) run between phases, which is why they can starve the event loop if misused."

---

## Core Implementation

### Event Loop Phases Visualization

```typescript
/**
 * Event Loop Phases (in order):
 * 
 * ┌───────────────────────────┐
 * │         timers            │  ← setTimeout, setInterval callbacks
 * └─────────────┬─────────────┘
 *               │
 * ┌─────────────┴─────────────┐
 * │     pending callbacks     │  ← I/O callbacks deferred to next iteration
 * └─────────────┬─────────────┘
 *               │
 * ┌─────────────┴─────────────┐
 * │       idle, prepare       │  ← internal use only
 * └─────────────┬─────────────┘
 *               │
 * ┌─────────────┴─────────────┐  ← incoming connections, data, etc.
 * │          poll             │  ← retrieve new I/O events
 * └─────────────┬─────────────┘
 *               │
 * ┌─────────────┴─────────────┐
 * │          check            │  ← setImmediate callbacks
 * └─────────────┬─────────────┘
 *               │
 * ┌─────────────┴─────────────┐
 * │     close callbacks       │  ← socket.on('close', ...)
 * └───────────────────────────┘
 * 
 * Between each phase: process.nextTick() and Promise microtasks
 */
```

### Understanding Timer Phase

```typescript
// timers-phase.ts
import { performance } from 'perf_hooks';

// setTimeout is NOT guaranteed to execute at exact time
// It's the MINIMUM time before callback can execute
function demonstrateTimerBehavior(): void {
  const start = performance.now();
  
  // Schedule timer for 100ms
  setTimeout(() => {
    const actual = performance.now() - start;
    console.log(`Timer scheduled: 100ms, Actual: ${actual.toFixed(2)}ms`);
  }, 100);
  
  // Simulate blocking operation
  const blockUntil = performance.now() + 200;
  while (performance.now() < blockUntil) {
    // Blocking the event loop for 200ms
  }
  
  console.log('Blocking complete');
  // Timer callback will run after ~200ms, not 100ms!
}

// Timer coalescing and ordering
function timerOrdering(): void {
  setTimeout(() => console.log('timeout 1'), 0);
  setTimeout(() => console.log('timeout 2'), 0);
  setTimeout(() => console.log('timeout 3'), 0);
  
  // All three will run in order during the same timer phase
  // if they become ready at the same time
}

// Understanding timer minimum delay
function timerMinimumDelay(): void {
  const iterations = 1000;
  let count = 0;
  const start = performance.now();
  
  function scheduleNext(): void {
    if (count++ < iterations) {
      setTimeout(scheduleNext, 0);
    } else {
      const elapsed = performance.now() - start;
      console.log(`${iterations} setTimeout(0) took ${elapsed.toFixed(2)}ms`);
      console.log(`Average per timer: ${(elapsed / iterations).toFixed(4)}ms`);
      // Will be ~1ms per timer due to minimum delay
    }
  }
  
  scheduleNext();
}
```

### process.nextTick vs setImmediate

```typescript
// nextTick-vs-setImmediate.ts

/**
 * process.nextTick():
 * - Runs BEFORE the event loop continues
 * - Part of the "microtask queue"
 * - Can starve the event loop if called recursively
 * 
 * setImmediate():
 * - Runs in the "check" phase of the event loop
 * - Allows I/O to happen between iterations
 * - Safer for recursive operations
 */

function demonstrateDifference(): void {
  // This order is predictable when called from main module
  setImmediate(() => console.log('1. setImmediate'));
  process.nextTick(() => console.log('2. nextTick'));
  Promise.resolve().then(() => console.log('3. Promise.then'));
  
  // Output order:
  // 2. nextTick (microtask queue, runs first)
  // 3. Promise.then (microtask queue, after nextTick)
  // 1. setImmediate (check phase)
}

// Inside I/O callback, order can be different
import * as fs from 'fs';

function demonstrateIOContext(): void {
  fs.readFile(__filename, () => {
    // Inside I/O callback, we're in the poll phase
    // setImmediate is in check phase (next)
    // setTimeout is in timers phase (next iteration)
    
    setTimeout(() => console.log('1. setTimeout'), 0);
    setImmediate(() => console.log('2. setImmediate'));
    process.nextTick(() => console.log('3. nextTick'));
    
    // Output order:
    // 3. nextTick (always first - microtask)
    // 2. setImmediate (check phase is next after poll)
    // 1. setTimeout (timers phase in next iteration)
  });
}

// Starving the event loop with nextTick (DON'T DO THIS)
function starvingExample(): void {
  let count = 0;
  
  function recursiveNextTick(): void {
    count++;
    if (count < 1000000) {
      process.nextTick(recursiveNextTick);
    }
  }
  
  // This will block all I/O until complete!
  recursiveNextTick();
  
  // setTimeout will NEVER run until nextTick queue is empty
  setTimeout(() => console.log('This waits a long time'), 0);
}

// Safe recursive pattern with setImmediate
function safeRecursion(): void {
  let count = 0;
  
  function recursiveImmediate(): void {
    count++;
    if (count < 1000000) {
      setImmediate(recursiveImmediate);
    }
  }
  
  recursiveImmediate();
  
  // I/O can happen between iterations
  setTimeout(() => console.log('This runs promptly'), 0);
}
```

### Microtask Queue Deep Dive

```typescript
// microtasks.ts

/**
 * Microtask queue order:
 * 1. process.nextTick callbacks
 * 2. Promise.then/catch/finally callbacks
 * 3. queueMicrotask callbacks
 * 
 * Microtasks are processed between EVERY phase of the event loop
 * and after EVERY callback in the current phase
 */

function microtaskOrder(): void {
  console.log('1. Script start');
  
  setTimeout(() => {
    console.log('6. setTimeout');
  }, 0);
  
  Promise.resolve()
    .then(() => {
      console.log('3. Promise 1');
    })
    .then(() => {
      console.log('4. Promise 2');
    });
  
  process.nextTick(() => {
    console.log('2. nextTick');
  });
  
  queueMicrotask(() => {
    console.log('5. queueMicrotask');
  });
  
  console.log('Script end (sync)');
  
  // Output:
  // 1. Script start
  // Script end (sync)
  // 2. nextTick (nextTick queue processed first)
  // 3. Promise 1 (promise microtasks next)
  // 4. Promise 2
  // 5. queueMicrotask (after promises)
  // 6. setTimeout (timer phase)
}

// Nested microtasks
function nestedMicrotasks(): void {
  Promise.resolve().then(() => {
    console.log('1. Promise 1');
    
    process.nextTick(() => {
      console.log('2. Nested nextTick');
    });
    
    Promise.resolve().then(() => {
      console.log('3. Nested Promise');
    });
  });
  
  setTimeout(() => {
    console.log('4. setTimeout');
  }, 0);
  
  // Output:
  // 1. Promise 1
  // 2. Nested nextTick (nextTick has priority)
  // 3. Nested Promise
  // 4. setTimeout
}
```

### Poll Phase and I/O

```typescript
// poll-phase.ts
import * as fs from 'fs';
import * as net from 'net';

/**
 * Poll phase responsibilities:
 * 1. Calculate how long to block and poll for I/O
 * 2. Process events in the poll queue
 * 
 * If poll queue is NOT empty:
 *   - Execute callbacks synchronously until queue is empty or limit reached
 * 
 * If poll queue IS empty:
 *   - If setImmediate callbacks scheduled → move to check phase
 *   - If timers are ready → wrap back to timers phase
 *   - Otherwise, wait for callbacks to be added to queue
 */

function demonstratePollPhase(): void {
  const server = net.createServer((socket) => {
    // This callback runs in the poll phase
    socket.on('data', (data) => {
      // Data events also run in poll phase
      console.log('Data received in poll phase');
      
      // Schedule something for check phase
      setImmediate(() => {
        console.log('Processing in check phase');
      });
    });
  });
  
  server.listen(3000);
}

// File I/O in poll phase
function fileIOExample(): void {
  const start = Date.now();
  
  // fs.readFile callback runs in poll phase
  fs.readFile('large-file.txt', (err, data) => {
    const elapsed = Date.now() - start;
    console.log(`File read complete in ${elapsed}ms (poll phase)`);
    
    // Heavy processing here blocks the event loop
    // Better to use setImmediate or worker threads
    setImmediate(() => {
      // Process data without blocking poll phase
      processData(data);
    });
  });
  
  // Multiple parallel reads
  const files = ['file1.txt', 'file2.txt', 'file3.txt'];
  files.forEach(file => {
    fs.readFile(file, (err, data) => {
      // These may complete in any order
      // All callbacks queue in poll phase
    });
  });
}

function processData(data: Buffer): void {
  // Data processing
}
```

### Measuring Event Loop Lag

```typescript
// event-loop-lag.ts
import { monitorEventLoopDelay, PerformanceObserver, performance } from 'perf_hooks';

// Method 1: Using monitorEventLoopDelay (Node.js 11+)
function monitorWithBuiltin(): void {
  const histogram = monitorEventLoopDelay({ resolution: 20 });
  histogram.enable();
  
  setInterval(() => {
    console.log({
      min: histogram.min / 1e6,      // Convert nanoseconds to ms
      max: histogram.max / 1e6,
      mean: histogram.mean / 1e6,
      stddev: histogram.stddev / 1e6,
      percentile99: histogram.percentile(99) / 1e6,
    });
    histogram.reset();
  }, 5000);
}

// Method 2: Manual measurement
function manualLagMeasurement(): void {
  const interval = 100; // Expected interval
  let lastTime = Date.now();
  
  setInterval(() => {
    const now = Date.now();
    const delta = now - lastTime;
    const lag = delta - interval;
    
    if (lag > 10) { // More than 10ms lag
      console.warn(`Event loop lag: ${lag}ms`);
    }
    
    lastTime = now;
  }, interval);
}

// Method 3: Using setImmediate for more accuracy
class EventLoopMonitor {
  private samples: number[] = [];
  private readonly maxSamples = 100;
  
  start(): void {
    this.measure();
  }
  
  private measure(): void {
    const start = process.hrtime.bigint();
    
    setImmediate(() => {
      const end = process.hrtime.bigint();
      const lagNs = Number(end - start);
      const lagMs = lagNs / 1e6;
      
      this.samples.push(lagMs);
      if (this.samples.length > this.maxSamples) {
        this.samples.shift();
      }
      
      // Continue measuring
      setImmediate(() => this.measure());
    });
  }
  
  getStats(): { avg: number; max: number; p99: number } {
    const sorted = [...this.samples].sort((a, b) => a - b);
    const avg = this.samples.reduce((a, b) => a + b, 0) / this.samples.length;
    const max = Math.max(...this.samples);
    const p99Index = Math.floor(sorted.length * 0.99);
    const p99 = sorted[p99Index] || 0;
    
    return { avg, max, p99 };
  }
}
```

### Blocking vs Non-Blocking Operations

```typescript
// blocking-operations.ts
import * as fs from 'fs';
import * as crypto from 'crypto';

// BLOCKING - Avoid in production
function blockingExamples(): void {
  // Synchronous file operations
  const data = fs.readFileSync('file.txt'); // BLOCKS
  
  // CPU-intensive operations
  const hash = crypto.pbkdf2Sync(
    'password',
    'salt',
    100000,
    64,
    'sha512'
  ); // BLOCKS for ~100ms
  
  // JSON parsing large data
  const largeJson = JSON.parse(hugeJsonString); // BLOCKS
  
  // RegExp on large strings
  const result = largeString.match(/complex.*pattern/g); // BLOCKS
}

// NON-BLOCKING - Preferred patterns
async function nonBlockingExamples(): Promise<void> {
  // Async file operations
  const data = await fs.promises.readFile('file.txt');
  
  // Async crypto operations
  const hash = await new Promise<Buffer>((resolve, reject) => {
    crypto.pbkdf2(
      'password',
      'salt',
      100000,
      64,
      'sha512',
      (err, derivedKey) => {
        if (err) reject(err);
        else resolve(derivedKey);
      }
    );
  });
}

// Chunking CPU-intensive work
async function processLargeArray<T>(
  items: T[],
  processor: (item: T) => void,
  chunkSize = 100
): Promise<void> {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    
    // Process chunk synchronously
    chunk.forEach(processor);
    
    // Yield to event loop between chunks
    await new Promise(resolve => setImmediate(resolve));
  }
}

// Breaking up JSON parsing
async function parseJsonInChunks(jsonString: string): Promise<unknown> {
  // For truly large JSON, consider streaming parsers like JSONStream
  return new Promise((resolve, reject) => {
    setImmediate(() => {
      try {
        resolve(JSON.parse(jsonString));
      } catch (e) {
        reject(e);
      }
    });
  });
}
```

### Async/Await and the Event Loop

```typescript
// async-await-eventloop.ts

/**
 * async/await is syntactic sugar over Promises
 * Each await creates a microtask for the continuation
 */

async function asyncBehavior(): Promise<void> {
  console.log('1. Before await');
  
  await Promise.resolve();
  // Everything after await is a microtask
  
  console.log('2. After await');
}

// This is equivalent to:
function equivalentPromise(): Promise<void> {
  console.log('1. Before await');
  
  return Promise.resolve().then(() => {
    console.log('2. After await');
  });
}

// Multiple awaits create multiple microtasks
async function multipleAwaits(): Promise<void> {
  console.log('A');
  await Promise.resolve();
  console.log('B'); // Microtask 1
  await Promise.resolve();
  console.log('C'); // Microtask 2
  await Promise.resolve();
  console.log('D'); // Microtask 3
}

// Interleaving with other async functions
async function interleaving(): Promise<void> {
  async function task1(): Promise<void> {
    console.log('task1: start');
    await Promise.resolve();
    console.log('task1: after await 1');
    await Promise.resolve();
    console.log('task1: after await 2');
  }
  
  async function task2(): Promise<void> {
    console.log('task2: start');
    await Promise.resolve();
    console.log('task2: after await 1');
    await Promise.resolve();
    console.log('task2: after await 2');
  }
  
  // Start both without awaiting
  task1();
  task2();
  
  // Output:
  // task1: start
  // task2: start
  // task1: after await 1
  // task2: after await 1
  // task1: after await 2
  // task2: after await 2
  
  // They interleave at each await point!
}

// Parallel vs Sequential execution
async function executionPatterns(): Promise<void> {
  const delay = (ms: number) => new Promise(r => setTimeout(r, ms));
  
  // SEQUENTIAL - Each waits for previous
  console.time('sequential');
  await delay(100);
  await delay(100);
  await delay(100);
  console.timeEnd('sequential'); // ~300ms
  
  // PARALLEL - All start immediately
  console.time('parallel');
  await Promise.all([
    delay(100),
    delay(100),
    delay(100)
  ]);
  console.timeEnd('parallel'); // ~100ms
}
```

### Event Loop in Different Environments

```typescript
// environments.ts

// Node.js specific
function nodeSpecific(): void {
  // process.nextTick - Node.js only
  process.nextTick(() => {
    console.log('nextTick');
  });
  
  // setImmediate - Node.js only (also in some browsers)
  setImmediate(() => {
    console.log('immediate');
  });
}

// Cross-platform microtask scheduling
function crossPlatform(): void {
  // queueMicrotask - Standard, works in browsers and Node.js
  queueMicrotask(() => {
    console.log('microtask');
  });
  
  // Promise.resolve().then - Works everywhere
  Promise.resolve().then(() => {
    console.log('promise microtask');
  });
}

// Polyfill setImmediate for browsers
const setImmediatePolyfill = 
  typeof setImmediate !== 'undefined' 
    ? setImmediate 
    : (fn: () => void) => setTimeout(fn, 0);

// Better browser alternative using MessageChannel
function browserSetImmediate(callback: () => void): void {
  const channel = new MessageChannel();
  channel.port1.onmessage = callback;
  channel.port2.postMessage(undefined);
}
```

---

## Real-World Scenarios

### Scenario 1: Debugging Unexpected Execution Order

```typescript
// Problem: Code executing in unexpected order
async function orderingProblem(): Promise<void> {
  console.log('1');
  
  setTimeout(() => console.log('2'), 0);
  
  Promise.resolve().then(() => console.log('3'));
  
  process.nextTick(() => console.log('4'));
  
  setImmediate(() => console.log('5'));
  
  await Promise.resolve();
  console.log('6');
  
  console.log('7');
  
  // Output: 1, 7, 4, 3, 6, 5, 2
  // Why?
  // 1, 7 - synchronous
  // 4 - nextTick (highest priority microtask)
  // 3 - Promise (microtask)
  // 6 - continuation after await (microtask)
  // 5 - setImmediate (check phase)
  // 2 - setTimeout (timers phase, next iteration)
}
```

### Scenario 2: High-Performance Server

```typescript
// server-performance.ts
import * as http from 'http';
import { monitorEventLoopDelay } from 'perf_hooks';

const histogram = monitorEventLoopDelay({ resolution: 20 });
histogram.enable();

const server = http.createServer(async (req, res) => {
  // Check event loop health
  const lagMs = histogram.mean / 1e6;
  
  if (lagMs > 100) {
    // Event loop is lagging, shed load
    res.writeHead(503, { 'Retry-After': '5' });
    res.end('Service temporarily unavailable');
    return;
  }
  
  // Use setImmediate to not starve connections
  setImmediate(async () => {
    try {
      const result = await handleRequest(req);
      res.writeHead(200);
      res.end(JSON.stringify(result));
    } catch (error) {
      res.writeHead(500);
      res.end('Internal error');
    }
  });
});

async function handleRequest(req: http.IncomingMessage): Promise<unknown> {
  // Request handling
  return { success: true };
}

// Expose metrics endpoint
setInterval(() => {
  console.log('Event loop stats:', {
    meanLag: (histogram.mean / 1e6).toFixed(2) + 'ms',
    maxLag: (histogram.max / 1e6).toFixed(2) + 'ms',
    p99Lag: (histogram.percentile(99) / 1e6).toFixed(2) + 'ms',
  });
}, 10000);
```

### Scenario 3: Batch Processing Without Blocking

```typescript
// batch-processing.ts

interface Job {
  id: string;
  data: unknown;
}

class NonBlockingBatchProcessor {
  private queue: Job[] = [];
  private processing = false;
  private readonly batchSize = 100;
  private readonly yieldInterval = 10; // Yield every 10 items
  
  add(job: Job): void {
    this.queue.push(job);
    if (!this.processing) {
      this.startProcessing();
    }
  }
  
  private startProcessing(): void {
    this.processing = true;
    this.processNext();
  }
  
  private processNext(): void {
    if (this.queue.length === 0) {
      this.processing = false;
      return;
    }
    
    // Process a small batch
    const batch = this.queue.splice(0, this.yieldInterval);
    
    for (const job of batch) {
      this.processJob(job);
    }
    
    // Yield to event loop
    if (this.queue.length > 0) {
      setImmediate(() => this.processNext());
    } else {
      this.processing = false;
    }
  }
  
  private processJob(job: Job): void {
    // Synchronous job processing
    console.log(`Processing job ${job.id}`);
  }
}

// Usage
const processor = new NonBlockingBatchProcessor();

// Add thousands of jobs
for (let i = 0; i < 10000; i++) {
  processor.add({ id: `job-${i}`, data: {} });
}

// Server continues to respond while processing
http.createServer((req, res) => {
  res.end('OK'); // Responds immediately
}).listen(3000);
```

---

## Common Pitfalls

### 1. Starving the Event Loop

```typescript
// ❌ BAD: Recursive nextTick blocks I/O
function badRecursive(): void {
  process.nextTick(() => {
    doWork();
    badRecursive(); // I/O NEVER runs
  });
}

// ✅ GOOD: Use setImmediate for recursive operations
function goodRecursive(): void {
  setImmediate(() => {
    doWork();
    goodRecursive(); // I/O can run between iterations
  });
}

function doWork(): void {}
```

### 2. Assuming Timer Precision

```typescript
// ❌ BAD: Assuming exact timing
setTimeout(() => {
  // This will NOT run at exactly 100ms
  console.log('Expected: 100ms');
}, 100);

// ✅ GOOD: Use timers for minimum delay, not precise timing
function scheduleWithMinimumDelay(
  callback: () => void,
  minimumDelay: number
): void {
  const start = Date.now();
  
  setTimeout(() => {
    const actual = Date.now() - start;
    console.log(`Scheduled: ${minimumDelay}ms, Actual: ${actual}ms`);
    callback();
  }, minimumDelay);
}
```

### 3. Mixing Sync and Async in APIs

```typescript
// ❌ BAD: Sometimes sync, sometimes async (Zalgo)
function inconsistentAPI(cached: boolean, callback: (data: string) => void): void {
  if (cached) {
    callback('cached data'); // Sync!
  } else {
    fs.readFile('file.txt', 'utf8', (err, data) => {
      callback(data); // Async!
    });
  }
}

// ✅ GOOD: Always async
function consistentAPI(cached: boolean, callback: (data: string) => void): void {
  if (cached) {
    process.nextTick(() => callback('cached data'));
  } else {
    fs.readFile('file.txt', 'utf8', (err, data) => {
      callback(data!);
    });
  }
}
```

### 4. Blocking in Callbacks

```typescript
// ❌ BAD: Heavy computation in callback
server.on('request', (req, res) => {
  const result = heavyComputation(); // Blocks all connections!
  res.end(result);
});

// ✅ GOOD: Offload heavy work
server.on('request', async (req, res) => {
  // Option 1: Worker thread
  const result = await runInWorker(heavyComputation);
  
  // Option 2: Break into chunks
  const result2 = await computeInChunks(data);
  
  res.end(result);
});

function heavyComputation(): string {
  return 'result';
}

async function runInWorker(fn: () => string): Promise<string> {
  // Worker thread implementation
  return fn();
}

async function computeInChunks(data: unknown): Promise<string> {
  return 'result';
}
```

---

## Interview Questions

### Q1: What's the difference between process.nextTick() and setImmediate()?

**A:** `process.nextTick()` callbacks execute before the event loop continues to the next phase - they're processed after the current operation, before any I/O. `setImmediate()` callbacks execute in the "check" phase of the event loop, after I/O events. Key differences:
- nextTick is faster but can starve I/O if used recursively
- setImmediate allows I/O to happen between iterations
- nextTick callbacks have priority over Promise microtasks

### Q2: Explain the order of execution in this code:

```typescript
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
process.nextTick(() => console.log('4'));
setImmediate(() => console.log('5'));
console.log('6');
```

**A:** Output: 1, 6, 4, 3, 5, 2
- 1, 6: Synchronous code runs first
- 4: nextTick queue (highest priority microtask)
- 3: Promise microtask queue
- 5: setImmediate (check phase)
- 2: setTimeout (timers phase, but check phase comes first when in I/O cycle)

### Q3: How can you prevent blocking the event loop during CPU-intensive operations?

**A:** Several strategies:
1. **Worker Threads**: Offload to separate threads for true parallelism
2. **Chunking**: Break work into small pieces, yield between chunks with setImmediate
3. **Child Processes**: Fork separate processes for heavy work
4. **External Services**: Use specialized services for heavy computation
5. **Streaming**: Process data in streams instead of loading entirely into memory

### Q4: What happens if you call process.nextTick() recursively?

**A:** The event loop will be blocked because nextTick callbacks are processed before the event loop can continue. All I/O operations will be starved until the recursive nextTick calls complete. This is called "starving" the event loop. Always use setImmediate for recursive operations to allow I/O to proceed.

### Q5: Why might a setTimeout with 0ms delay not execute immediately?

**A:** Several reasons:
1. **Minimum delay**: Browsers enforce 4ms minimum; Node.js has ~1ms minimum
2. **Event loop busy**: Other phases/callbacks must complete first
3. **I/O callbacks**: Poll phase may be processing I/O
4. **Microtasks**: All microtasks run before timer callbacks
5. **System load**: OS scheduling affects timer accuracy

---

## Quick Reference Checklist

### Event Loop Phases
- [ ] Timers → Pending I/O → Idle/Prepare → Poll → Check → Close
- [ ] Microtasks run between EVERY phase
- [ ] nextTick has priority over Promise microtasks

### Best Practices
- [ ] Use setImmediate for recursive operations
- [ ] Never block in callbacks - chunk heavy work
- [ ] Monitor event loop lag in production
- [ ] Keep APIs consistently async (avoid Zalgo)
- [ ] Use worker threads for CPU-intensive tasks

### Debugging
- [ ] Use `monitorEventLoopDelay()` for lag measurement
- [ ] Check if blocking operations are synchronous
- [ ] Verify timer expectations vs reality
- [ ] Profile with `--inspect` flag for flame graphs

### Performance
- [ ] Process data in chunks with setImmediate yields
- [ ] Use streaming for large data
- [ ] Implement load shedding when event loop lags
- [ ] Avoid synchronous file operations

---

*Last updated: February 2026*

