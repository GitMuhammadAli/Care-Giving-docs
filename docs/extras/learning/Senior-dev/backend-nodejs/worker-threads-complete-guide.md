# Worker Threads - Complete Guide

> **MUST REMEMBER**: Worker threads allow running JavaScript in parallel threads within the same Node.js process, sharing memory via SharedArrayBuffer and transferring data via structured cloning. Unlike clustering (separate processes), worker threads are lightweight and can share memory, making them ideal for CPU-intensive tasks that would otherwise block the event loop.

---

## How to Explain Like a Senior Developer

"Worker threads are Node.js's answer to CPU-bound operations blocking the event loop. Unlike clustering which spawns entire processes, worker threads run within the same process but in separate V8 isolates - each with its own event loop. The killer feature is SharedArrayBuffer for zero-copy memory sharing between threads. Use them for image processing, cryptography, data parsing - anything that would freeze your server. But remember: they're not free. Spawning threads has overhead, so pool them for repeated operations. Comlink makes the API feel like calling regular async functions, abstracting away the postMessage complexity."

---

## Core Implementation

### Basic Worker Thread

```typescript
// main.ts
import { Worker, isMainThread, parentPort, workerData } from 'worker_threads';
import { fileURLToPath } from 'url';
import path from 'path';

if (isMainThread) {
  // Main thread
  const __filename = fileURLToPath(import.meta.url);
  
  const worker = new Worker(__filename, {
    workerData: { number: 42 }
  });
  
  worker.on('message', (result) => {
    console.log('Result from worker:', result);
  });
  
  worker.on('error', (error) => {
    console.error('Worker error:', error);
  });
  
  worker.on('exit', (code) => {
    console.log(`Worker exited with code ${code}`);
  });
  
} else {
  // Worker thread
  const { number } = workerData;
  
  // Simulate CPU-intensive work
  let result = 0;
  for (let i = 0; i < 1e9; i++) {
    result += Math.sqrt(number * i);
  }
  
  parentPort?.postMessage(result);
}
```

### Separate Worker File Pattern

```typescript
// cpu-worker.ts
import { parentPort, workerData } from 'worker_threads';

interface WorkerInput {
  operation: 'fibonacci' | 'prime' | 'hash';
  data: unknown;
}

interface WorkerOutput {
  success: boolean;
  result?: unknown;
  error?: string;
  duration: number;
}

function fibonacci(n: number): bigint {
  if (n <= 1) return BigInt(n);
  let a = BigInt(0), b = BigInt(1);
  for (let i = 2; i <= n; i++) {
    [a, b] = [b, a + b];
  }
  return b;
}

function isPrime(n: number): boolean {
  if (n < 2) return false;
  for (let i = 2; i <= Math.sqrt(n); i++) {
    if (n % i === 0) return false;
  }
  return true;
}

function findPrimes(limit: number): number[] {
  const primes: number[] = [];
  for (let i = 2; i <= limit; i++) {
    if (isPrime(i)) primes.push(i);
  }
  return primes;
}

parentPort?.on('message', (input: WorkerInput) => {
  const start = Date.now();
  
  try {
    let result: unknown;
    
    switch (input.operation) {
      case 'fibonacci':
        result = fibonacci(input.data as number).toString();
        break;
      case 'prime':
        result = findPrimes(input.data as number);
        break;
      default:
        throw new Error(`Unknown operation: ${input.operation}`);
    }
    
    const output: WorkerOutput = {
      success: true,
      result,
      duration: Date.now() - start,
    };
    
    parentPort?.postMessage(output);
  } catch (error) {
    const output: WorkerOutput = {
      success: false,
      error: (error as Error).message,
      duration: Date.now() - start,
    };
    
    parentPort?.postMessage(output);
  }
});

// main-with-worker.ts
import { Worker } from 'worker_threads';
import path from 'path';

function runWorker<T>(operation: string, data: unknown): Promise<T> {
  return new Promise((resolve, reject) => {
    const worker = new Worker(path.join(__dirname, 'cpu-worker.js'));
    
    worker.on('message', (result: { success: boolean; result?: T; error?: string }) => {
      if (result.success) {
        resolve(result.result as T);
      } else {
        reject(new Error(result.error));
      }
      worker.terminate();
    });
    
    worker.on('error', reject);
    
    worker.postMessage({ operation, data });
  });
}

// Usage
async function main() {
  const fib = await runWorker<string>('fibonacci', 1000);
  console.log('Fibonacci(1000):', fib.slice(0, 50) + '...');
  
  const primes = await runWorker<number[]>('prime', 10000);
  console.log('Primes up to 10000:', primes.length);
}
```

### Worker Thread Pool

```typescript
// worker-pool.ts
import { Worker } from 'worker_threads';
import { EventEmitter } from 'events';
import path from 'path';

interface Task<T> {
  data: unknown;
  resolve: (value: T) => void;
  reject: (error: Error) => void;
}

interface PoolOptions {
  workerPath: string;
  minWorkers?: number;
  maxWorkers?: number;
  idleTimeout?: number;
}

class WorkerPool<T = unknown> extends EventEmitter {
  private workers: Worker[] = [];
  private idleWorkers: Worker[] = [];
  private taskQueue: Task<T>[] = [];
  private workerTaskMap = new Map<Worker, Task<T>>();
  
  private readonly workerPath: string;
  private readonly minWorkers: number;
  private readonly maxWorkers: number;
  private readonly idleTimeout: number;
  
  constructor(options: PoolOptions) {
    super();
    this.workerPath = options.workerPath;
    this.minWorkers = options.minWorkers ?? 2;
    this.maxWorkers = options.maxWorkers ?? 4;
    this.idleTimeout = options.idleTimeout ?? 30000;
    
    // Pre-spawn minimum workers
    for (let i = 0; i < this.minWorkers; i++) {
      this.spawnWorker();
    }
  }
  
  private spawnWorker(): Worker {
    const worker = new Worker(this.workerPath);
    
    worker.on('message', (result: T) => {
      const task = this.workerTaskMap.get(worker);
      if (task) {
        this.workerTaskMap.delete(worker);
        task.resolve(result);
        this.onWorkerFree(worker);
      }
    });
    
    worker.on('error', (error) => {
      const task = this.workerTaskMap.get(worker);
      if (task) {
        this.workerTaskMap.delete(worker);
        task.reject(error);
      }
      this.removeWorker(worker);
    });
    
    worker.on('exit', (code) => {
      this.removeWorker(worker);
      
      // Respawn if below minimum
      if (this.workers.length < this.minWorkers) {
        this.spawnWorker();
      }
    });
    
    this.workers.push(worker);
    this.idleWorkers.push(worker);
    
    return worker;
  }
  
  private removeWorker(worker: Worker): void {
    const workerIndex = this.workers.indexOf(worker);
    if (workerIndex > -1) {
      this.workers.splice(workerIndex, 1);
    }
    
    const idleIndex = this.idleWorkers.indexOf(worker);
    if (idleIndex > -1) {
      this.idleWorkers.splice(idleIndex, 1);
    }
  }
  
  private onWorkerFree(worker: Worker): void {
    // Check for pending tasks
    const nextTask = this.taskQueue.shift();
    
    if (nextTask) {
      this.assignTask(worker, nextTask);
    } else {
      this.idleWorkers.push(worker);
      
      // Schedule idle worker cleanup
      if (this.workers.length > this.minWorkers) {
        setTimeout(() => {
          const idleIndex = this.idleWorkers.indexOf(worker);
          if (idleIndex > -1 && this.workers.length > this.minWorkers) {
            this.idleWorkers.splice(idleIndex, 1);
            this.removeWorker(worker);
            worker.terminate();
          }
        }, this.idleTimeout);
      }
    }
  }
  
  private assignTask(worker: Worker, task: Task<T>): void {
    const idleIndex = this.idleWorkers.indexOf(worker);
    if (idleIndex > -1) {
      this.idleWorkers.splice(idleIndex, 1);
    }
    
    this.workerTaskMap.set(worker, task);
    worker.postMessage(task.data);
  }
  
  exec(data: unknown): Promise<T> {
    return new Promise((resolve, reject) => {
      const task: Task<T> = { data, resolve, reject };
      
      // Try to find an idle worker
      const idleWorker = this.idleWorkers.shift();
      
      if (idleWorker) {
        this.assignTask(idleWorker, task);
      } else if (this.workers.length < this.maxWorkers) {
        // Spawn new worker
        const worker = this.spawnWorker();
        this.idleWorkers.pop(); // Remove from idle (we just added it)
        this.assignTask(worker, task);
      } else {
        // Queue the task
        this.taskQueue.push(task);
      }
    });
  }
  
  async destroy(): Promise<void> {
    const terminations = this.workers.map(worker => worker.terminate());
    await Promise.all(terminations);
    this.workers = [];
    this.idleWorkers = [];
    this.taskQueue = [];
  }
  
  getStats(): { total: number; idle: number; queued: number } {
    return {
      total: this.workers.length,
      idle: this.idleWorkers.length,
      queued: this.taskQueue.length,
    };
  }
}

// Usage
const pool = new WorkerPool<number>({
  workerPath: path.join(__dirname, 'compute-worker.js'),
  minWorkers: 2,
  maxWorkers: 8,
});

async function processData(items: number[]): Promise<number[]> {
  const results = await Promise.all(
    items.map(item => pool.exec(item))
  );
  return results;
}
```

### SharedArrayBuffer for Zero-Copy Data Sharing

```typescript
// shared-memory.ts
import { Worker, isMainThread, parentPort, workerData } from 'worker_threads';

if (isMainThread) {
  // Create shared buffer
  const sharedBuffer = new SharedArrayBuffer(1024 * 1024); // 1MB
  const sharedArray = new Int32Array(sharedBuffer);
  
  // Initialize with data
  for (let i = 0; i < 1000; i++) {
    sharedArray[i] = i;
  }
  
  // Create multiple workers sharing the same buffer
  const workers = [];
  
  for (let i = 0; i < 4; i++) {
    const worker = new Worker(__filename, {
      workerData: {
        sharedBuffer,
        startIndex: i * 250,
        endIndex: (i + 1) * 250,
        workerId: i,
      }
    });
    
    workers.push(new Promise<void>((resolve) => {
      worker.on('message', (msg) => {
        console.log(`Worker ${i}: ${msg}`);
        resolve();
      });
    }));
  }
  
  await Promise.all(workers);
  
  // Check results (modified in place by workers)
  console.log('First 10 values:', Array.from(sharedArray.slice(0, 10)));
  
} else {
  const { sharedBuffer, startIndex, endIndex, workerId } = workerData;
  const sharedArray = new Int32Array(sharedBuffer);
  
  // Process our slice of the array
  for (let i = startIndex; i < endIndex; i++) {
    sharedArray[i] = sharedArray[i] * 2; // Double each value
  }
  
  parentPort?.postMessage(`Processed indices ${startIndex}-${endIndex}`);
}

// Using Atomics for thread-safe operations
if (isMainThread) {
  const counterBuffer = new SharedArrayBuffer(4);
  const counter = new Int32Array(counterBuffer);
  
  const workers = [];
  
  for (let i = 0; i < 4; i++) {
    const worker = new Worker(__filename, {
      workerData: { counterBuffer, iterations: 10000 }
    });
    
    workers.push(new Promise<void>((resolve) => {
      worker.on('exit', resolve);
    }));
  }
  
  await Promise.all(workers);
  
  // Should be exactly 40000 (4 workers × 10000 iterations)
  console.log('Final counter:', Atomics.load(counter, 0));
  
} else {
  const { counterBuffer, iterations } = workerData;
  const counter = new Int32Array(counterBuffer);
  
  for (let i = 0; i < iterations; i++) {
    // Thread-safe increment
    Atomics.add(counter, 0, 1);
  }
}
```

### Using Comlink for Simplified API

```typescript
// comlink-worker.ts
import * as Comlink from 'comlink';
import { parentPort } from 'worker_threads';
import nodeEndpoint from 'comlink/dist/umd/node-adapter';

const api = {
  fibonacci(n: number): bigint {
    if (n <= 1) return BigInt(n);
    let a = BigInt(0), b = BigInt(1);
    for (let i = 2; i <= n; i++) {
      [a, b] = [b, a + b];
    }
    return b;
  },
  
  async processImage(imageData: Uint8Array): Promise<Uint8Array> {
    // Image processing logic
    const result = new Uint8Array(imageData.length);
    for (let i = 0; i < imageData.length; i++) {
      result[i] = 255 - imageData[i]; // Invert
    }
    return result;
  },
  
  heavyComputation(data: number[]): number {
    return data.reduce((sum, val) => sum + Math.sqrt(val), 0);
  },
};

Comlink.expose(api, nodeEndpoint(parentPort!));

// main-comlink.ts
import { Worker } from 'worker_threads';
import * as Comlink from 'comlink';
import nodeEndpoint from 'comlink/dist/umd/node-adapter';

async function main(): Promise<void> {
  const worker = new Worker('./comlink-worker.js');
  const api = Comlink.wrap<typeof import('./comlink-worker')['api']>(
    nodeEndpoint(worker)
  );
  
  // Call worker functions like regular async functions!
  const fib = await api.fibonacci(100);
  console.log('Fibonacci(100):', fib.toString());
  
  const data = new Array(1000000).fill(0).map(() => Math.random() * 100);
  const result = await api.heavyComputation(data);
  console.log('Computation result:', result);
  
  // Don't forget to terminate
  worker.terminate();
}
```

### Transferable Objects

```typescript
// transferable.ts
import { Worker, isMainThread, parentPort, workerData } from 'worker_threads';

if (isMainThread) {
  const worker = new Worker(__filename);
  
  // Create a large buffer
  const buffer = new ArrayBuffer(1024 * 1024 * 100); // 100MB
  const view = new Uint8Array(buffer);
  view.fill(42);
  
  console.log('Before transfer, buffer size:', buffer.byteLength);
  
  // Transfer ownership (zero-copy)
  worker.postMessage({ buffer }, [buffer]);
  
  console.log('After transfer, buffer size:', buffer.byteLength); // 0! Detached
  
  worker.on('message', ({ buffer: resultBuffer }) => {
    console.log('Received back, size:', resultBuffer.byteLength);
    const view = new Uint8Array(resultBuffer);
    console.log('First byte:', view[0]); // 84 (doubled)
  });
  
} else {
  parentPort?.on('message', ({ buffer }) => {
    const view = new Uint8Array(buffer);
    
    // Process buffer
    for (let i = 0; i < view.length; i++) {
      view[i] = view[i] * 2;
    }
    
    // Transfer back
    parentPort?.postMessage({ buffer }, [buffer]);
  });
}
```

---

## Real-World Scenarios

### Scenario 1: Image Processing Server

```typescript
// image-processor.ts
import { Worker } from 'worker_threads';
import path from 'path';
import express from 'express';
import multer from 'multer';

const upload = multer({ storage: multer.memoryStorage() });

// Worker pool for image processing
class ImageWorkerPool {
  private workers: Worker[] = [];
  private queue: Array<{
    imageBuffer: Buffer;
    operation: string;
    resolve: (result: Buffer) => void;
    reject: (error: Error) => void;
  }> = [];
  private busyWorkers = new Set<Worker>();
  
  constructor(size: number) {
    for (let i = 0; i < size; i++) {
      const worker = new Worker(path.join(__dirname, 'image-worker.js'));
      
      worker.on('message', ({ buffer, error }) => {
        const task = (worker as any).__currentTask;
        delete (worker as any).__currentTask;
        this.busyWorkers.delete(worker);
        
        if (error) {
          task.reject(new Error(error));
        } else {
          task.resolve(Buffer.from(buffer));
        }
        
        this.processQueue();
      });
      
      this.workers.push(worker);
    }
  }
  
  private processQueue(): void {
    if (this.queue.length === 0) return;
    
    const availableWorker = this.workers.find(w => !this.busyWorkers.has(w));
    if (!availableWorker) return;
    
    const task = this.queue.shift()!;
    this.busyWorkers.add(availableWorker);
    (availableWorker as any).__currentTask = task;
    
    availableWorker.postMessage({
      buffer: task.imageBuffer,
      operation: task.operation,
    });
  }
  
  process(imageBuffer: Buffer, operation: string): Promise<Buffer> {
    return new Promise((resolve, reject) => {
      this.queue.push({ imageBuffer, operation, resolve, reject });
      this.processQueue();
    });
  }
}

// image-worker.js
import { parentPort } from 'worker_threads';
import sharp from 'sharp';

parentPort?.on('message', async ({ buffer, operation }) => {
  try {
    let result: Buffer;
    
    switch (operation) {
      case 'thumbnail':
        result = await sharp(buffer)
          .resize(200, 200, { fit: 'cover' })
          .jpeg({ quality: 80 })
          .toBuffer();
        break;
        
      case 'grayscale':
        result = await sharp(buffer)
          .grayscale()
          .toBuffer();
        break;
        
      case 'blur':
        result = await sharp(buffer)
          .blur(10)
          .toBuffer();
        break;
        
      default:
        throw new Error(`Unknown operation: ${operation}`);
    }
    
    parentPort?.postMessage({ buffer: result });
  } catch (error) {
    parentPort?.postMessage({ error: (error as Error).message });
  }
});

// Express server
const app = express();
const pool = new ImageWorkerPool(4);

app.post('/process', upload.single('image'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No image provided' });
  }
  
  const operation = req.query.operation as string || 'thumbnail';
  
  try {
    const result = await pool.process(req.file.buffer, operation);
    res.contentType('image/jpeg');
    res.send(result);
  } catch (error) {
    res.status(500).json({ error: (error as Error).message });
  }
});

app.listen(3000);
```

### Scenario 2: Parallel Data Processing

```typescript
// data-processor.ts
import { Worker, isMainThread, parentPort, workerData } from 'worker_threads';
import os from 'os';

interface DataChunk {
  id: number;
  data: number[];
}

interface ProcessedChunk {
  id: number;
  result: {
    sum: number;
    avg: number;
    min: number;
    max: number;
    stdDev: number;
  };
}

if (isMainThread) {
  async function processDataInParallel(data: number[]): Promise<ProcessedChunk['result']> {
    const numWorkers = os.cpus().length;
    const chunkSize = Math.ceil(data.length / numWorkers);
    
    const workers: Promise<ProcessedChunk>[] = [];
    
    for (let i = 0; i < numWorkers; i++) {
      const start = i * chunkSize;
      const end = Math.min(start + chunkSize, data.length);
      const chunk = data.slice(start, end);
      
      if (chunk.length === 0) continue;
      
      const workerPromise = new Promise<ProcessedChunk>((resolve, reject) => {
        const worker = new Worker(__filename, {
          workerData: { id: i, data: chunk }
        });
        
        worker.on('message', resolve);
        worker.on('error', reject);
      });
      
      workers.push(workerPromise);
    }
    
    const results = await Promise.all(workers);
    
    // Combine results
    const combined = {
      sum: 0,
      count: 0,
      min: Infinity,
      max: -Infinity,
      sumOfSquares: 0,
    };
    
    for (const { result } of results) {
      combined.sum += result.sum;
      combined.count += result.avg; // Actually chunk length
      combined.min = Math.min(combined.min, result.min);
      combined.max = Math.max(combined.max, result.max);
    }
    
    const avg = combined.sum / data.length;
    
    // Recalculate stdDev (simplified)
    let variance = 0;
    for (const val of data) {
      variance += Math.pow(val - avg, 2);
    }
    const stdDev = Math.sqrt(variance / data.length);
    
    return {
      sum: combined.sum,
      avg,
      min: combined.min,
      max: combined.max,
      stdDev,
    };
  }
  
  // Test with large dataset
  const testData = new Array(10_000_000)
    .fill(0)
    .map(() => Math.random() * 1000);
  
  console.time('parallel');
  const result = await processDataInParallel(testData);
  console.timeEnd('parallel');
  console.log('Result:', result);
  
} else {
  const { id, data }: DataChunk = workerData;
  
  const sum = data.reduce((a, b) => a + b, 0);
  const avg = data.length;
  const min = Math.min(...data);
  const max = Math.max(...data);
  const mean = sum / data.length;
  const variance = data.reduce((acc, val) => acc + Math.pow(val - mean, 2), 0) / data.length;
  const stdDev = Math.sqrt(variance);
  
  const result: ProcessedChunk = {
    id,
    result: { sum, avg, min, max, stdDev },
  };
  
  parentPort?.postMessage(result);
}
```

---

## Common Pitfalls

### 1. Overusing Worker Threads

```typescript
// ❌ BAD: Creating worker for every request
app.get('/compute', async (req, res) => {
  const worker = new Worker('./worker.js'); // Expensive!
  const result = await runWorker(worker, req.query);
  res.json(result);
});

// ✅ GOOD: Use a worker pool
const pool = new WorkerPool({ workerPath: './worker.js', maxWorkers: 4 });

app.get('/compute', async (req, res) => {
  const result = await pool.exec(req.query);
  res.json(result);
});
```

### 2. Not Handling Worker Errors

```typescript
// ❌ BAD: Unhandled worker errors
const worker = new Worker('./worker.js');
worker.postMessage(data);

// ✅ GOOD: Always handle errors
const worker = new Worker('./worker.js');

worker.on('error', (error) => {
  console.error('Worker error:', error);
  // Handle cleanup
});

worker.on('exit', (code) => {
  if (code !== 0) {
    console.error(`Worker exited with code ${code}`);
  }
});

worker.postMessage(data);
```

### 3. SharedArrayBuffer Race Conditions

```typescript
// ❌ BAD: Race condition without Atomics
const shared = new Int32Array(sharedBuffer);
shared[0]++; // Not thread-safe!

// ✅ GOOD: Use Atomics for thread-safety
Atomics.add(shared, 0, 1); // Thread-safe increment
```

---

## Interview Questions

### Q1: When should you use worker threads vs clustering?

**A:** Use **worker threads** for CPU-intensive tasks within an application (image processing, parsing, crypto). They share memory and are lighter weight. Use **clustering** for scaling I/O-bound servers across CPU cores. Clusters are separate processes with isolated memory, ideal for handling more concurrent connections.

### Q2: What is SharedArrayBuffer and when would you use it?

**A:** SharedArrayBuffer is a fixed-size raw binary buffer that can be shared between the main thread and worker threads without copying. Use it when you need zero-copy data sharing for large datasets. Always use Atomics operations for thread-safe access. Note: It requires specific security headers (COOP/COEP) in browsers.

### Q3: What's the overhead of worker threads?

**A:** Each worker thread creates a new V8 isolate (~2-4MB base memory), has its own event loop, and requires message serialization for non-shared data. Spawning takes ~50-100ms. This is why worker pools are important - amortize the startup cost over many operations.

### Q4: How does postMessage data transfer work?

**A:** By default, data is copied using structured cloning (handles most JS types). For ArrayBuffers, you can use transferables to move ownership without copying. SharedArrayBuffer is neither - it's truly shared memory. Transferables are marked by passing them in the second argument array.

---

## Quick Reference Checklist

### When to Use Worker Threads
- [ ] CPU-intensive operations (crypto, compression, parsing)
- [ ] Operations that would block event loop > 100ms
- [ ] Parallel processing of large datasets
- [ ] Background processing that shouldn't affect main thread

### Best Practices
- [ ] Use worker pools instead of spawning per-request
- [ ] Transfer large ArrayBuffers instead of copying
- [ ] Use Atomics with SharedArrayBuffer
- [ ] Handle worker errors and exits
- [ ] Terminate workers when done

### Avoid Worker Threads For
- [ ] Simple I/O operations (use async instead)
- [ ] Quick operations < 10ms
- [ ] Operations that need frequent main thread communication

---

*Last updated: February 2026*

