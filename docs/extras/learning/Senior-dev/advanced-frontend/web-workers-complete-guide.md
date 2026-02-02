# 🔧 Web Workers - Complete Guide

> A comprehensive guide to Web Workers - background processing, SharedArrayBuffer, Comlink, and offloading heavy computations from the main thread.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Web Workers run JavaScript in background threads, enabling CPU-intensive operations without blocking the main thread UI, communicating via message passing with postMessage/onmessage."

### The 7 Key Concepts (Remember These!)
```
1. DEDICATED WORKER   → One-to-one with main thread
2. SHARED WORKER      → Shared between tabs/windows
3. SERVICE WORKER     → Proxy for network requests
4. MESSAGE PASSING    → postMessage/onmessage communication
5. TRANSFERABLE       → Zero-copy transfer (ArrayBuffer)
6. SHAREDARRAYBUFFER  → Shared memory between threads
7. COMLINK            → RPC-style worker communication
```

### Worker Types
```
┌─────────────────────────────────────────────────────────────────┐
│              WEB WORKER TYPES                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DEDICATED WORKER                                              │
│  ─────────────────                                              │
│  • One-to-one relationship with creator                        │
│  • Cannot be shared between tabs                               │
│  • Most common type                                            │
│  Use: Heavy computation, data processing                       │
│                                                                 │
│  SHARED WORKER                                                 │
│  ─────────────                                                  │
│  • Shared between multiple tabs/windows                        │
│  • Uses ports for communication                                │
│  • Same origin only                                            │
│  Use: Shared state, connection pooling                         │
│                                                                 │
│  SERVICE WORKER                                                │
│  ──────────────                                                 │
│  • Proxy between app and network                               │
│  • Lifecycle: install → activate → fetch                       │
│  • Persists across page loads                                  │
│  Use: Offline, caching, push notifications                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Worker Limitations
```
┌─────────────────────────────────────────────────────────────────┐
│              WORKER LIMITATIONS                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ❌ NO ACCESS TO:                                               │
│  • DOM (document, window)                                      │
│  • Parent scope variables                                      │
│  • document.cookie                                             │
│  • localStorage (but IndexedDB works)                          │
│  • Most BOM APIs                                               │
│                                                                 │
│  ✅ CAN ACCESS:                                                │
│  • navigator (partial)                                         │
│  • location (read-only)                                        │
│  • XMLHttpRequest / fetch                                      │
│  • setTimeout / setInterval                                    │
│  • IndexedDB                                                   │
│  • WebSockets                                                  │
│  • crypto                                                      │
│  • importScripts()                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Main thread"** | "Heavy computation on main thread causes jank" |
| **"Transferable"** | "We transfer ArrayBuffers to avoid copying" |
| **"Structured clone"** | "Messages use structured clone algorithm" |
| **"Worker pool"** | "We use a worker pool for parallel processing" |
| **"Off-main-thread"** | "Image processing runs off-main-thread" |
| **"Comlink"** | "Comlink makes workers feel like async functions" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Frame budget | **16ms** | 60fps = 16.67ms/frame |
| Long task | **> 50ms** | Blocks main thread noticeably |
| Worker count | **navigator.hardwareConcurrency** | CPU cores |
| Message overhead | **~1ms** | postMessage serialization |

### The "Wow" Statement (Memorize This!)
> "We offload heavy computation to Web Workers to keep the UI responsive. Image processing, JSON parsing of large datasets, and crypto operations run in dedicated workers. We use Comlink so worker communication feels like calling async functions instead of raw postMessage. For large data, we transfer ArrayBuffers instead of copying - zero-copy transfer. Our worker pool matches navigator.hardwareConcurrency so we use all CPU cores for parallel processing. The result: main thread stays under 50ms tasks, UI stays at 60fps even during heavy computation."

---

## 📚 Table of Contents

1. [Dedicated Workers](#1-dedicated-workers)
2. [Message Passing](#2-message-passing)
3. [Transferables & SharedArrayBuffer](#3-transferables--sharedarraybuffer)
4. [Comlink](#4-comlink)
5. [Worker Pools](#5-worker-pools)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Dedicated Workers

```typescript
// ════════════════════════════════════════════════════════════════
// BASIC WORKER SETUP
// ════════════════════════════════════════════════════════════════

// worker.ts
self.onmessage = (event: MessageEvent) => {
  const { type, payload } = event.data;

  switch (type) {
    case 'PROCESS_DATA':
      const result = heavyComputation(payload);
      self.postMessage({ type: 'RESULT', payload: result });
      break;
      
    case 'CANCEL':
      // Handle cancellation
      break;
  }
};

function heavyComputation(data: number[]): number {
  // Simulate heavy work
  let sum = 0;
  for (let i = 0; i < data.length; i++) {
    sum += Math.sqrt(data[i]) * Math.sin(data[i]);
  }
  return sum;
}

// main.ts
const worker = new Worker(new URL('./worker.ts', import.meta.url), {
  type: 'module',
});

worker.onmessage = (event: MessageEvent) => {
  const { type, payload } = event.data;
  if (type === 'RESULT') {
    console.log('Result:', payload);
  }
};

worker.onerror = (error) => {
  console.error('Worker error:', error.message);
};

// Send work to worker
worker.postMessage({
  type: 'PROCESS_DATA',
  payload: Array.from({ length: 1000000 }, (_, i) => i),
});

// Terminate when done
// worker.terminate();

// ════════════════════════════════════════════════════════════════
// REACT HOOK FOR WORKERS
// ════════════════════════════════════════════════════════════════

import { useEffect, useRef, useCallback, useState } from 'react';

function useWorker<T, R>(
  workerFactory: () => Worker
): {
  postMessage: (data: T) => void;
  result: R | null;
  error: Error | null;
  isProcessing: boolean;
} {
  const workerRef = useRef<Worker | null>(null);
  const [result, setResult] = useState<R | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [isProcessing, setIsProcessing] = useState(false);

  useEffect(() => {
    workerRef.current = workerFactory();

    workerRef.current.onmessage = (event: MessageEvent<R>) => {
      setResult(event.data);
      setIsProcessing(false);
    };

    workerRef.current.onerror = (e) => {
      setError(new Error(e.message));
      setIsProcessing(false);
    };

    return () => {
      workerRef.current?.terminate();
    };
  }, [workerFactory]);

  const postMessage = useCallback((data: T) => {
    setIsProcessing(true);
    setError(null);
    workerRef.current?.postMessage(data);
  }, []);

  return { postMessage, result, error, isProcessing };
}

// Usage
function DataProcessor() {
  const { postMessage, result, isProcessing } = useWorker<number[], number>(
    () => new Worker(new URL('./worker.ts', import.meta.url), { type: 'module' })
  );

  const handleProcess = () => {
    postMessage(Array.from({ length: 1000000 }, (_, i) => i));
  };

  return (
    <div>
      <button onClick={handleProcess} disabled={isProcessing}>
        {isProcessing ? 'Processing...' : 'Process Data'}
      </button>
      {result !== null && <p>Result: {result}</p>}
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// INLINE WORKER (No separate file)
// ════════════════════════════════════════════════════════════════

function createInlineWorker(fn: Function): Worker {
  const blob = new Blob(
    [`self.onmessage = ${fn.toString()}`],
    { type: 'application/javascript' }
  );
  const url = URL.createObjectURL(blob);
  const worker = new Worker(url);
  URL.revokeObjectURL(url);
  return worker;
}

// Usage
const worker = createInlineWorker((event: MessageEvent) => {
  const numbers = event.data;
  const sum = numbers.reduce((a: number, b: number) => a + b, 0);
  self.postMessage(sum);
});

worker.postMessage([1, 2, 3, 4, 5]);
worker.onmessage = (e) => console.log('Sum:', e.data); // 15
```

---

## 2. Message Passing

```typescript
// ════════════════════════════════════════════════════════════════
// TYPED MESSAGE PROTOCOL
// ════════════════════════════════════════════════════════════════

// types.ts - Shared between main and worker
interface ProcessDataMessage {
  type: 'PROCESS_DATA';
  payload: {
    data: number[];
    options: { method: 'sum' | 'average' | 'max' };
  };
}

interface CancelMessage {
  type: 'CANCEL';
}

interface ProgressMessage {
  type: 'PROGRESS';
  payload: number; // 0-100
}

interface ResultMessage {
  type: 'RESULT';
  payload: number;
}

interface ErrorMessage {
  type: 'ERROR';
  payload: string;
}

type MainToWorkerMessage = ProcessDataMessage | CancelMessage;
type WorkerToMainMessage = ProgressMessage | ResultMessage | ErrorMessage;

// worker.ts
let cancelled = false;

self.onmessage = (event: MessageEvent<MainToWorkerMessage>) => {
  const message = event.data;

  switch (message.type) {
    case 'PROCESS_DATA':
      cancelled = false;
      processData(message.payload.data, message.payload.options);
      break;
      
    case 'CANCEL':
      cancelled = true;
      break;
  }
};

function processData(
  data: number[],
  options: { method: 'sum' | 'average' | 'max' }
) {
  try {
    let result = 0;
    const total = data.length;

    for (let i = 0; i < total; i++) {
      if (cancelled) {
        return;
      }

      // Report progress every 10%
      if (i % Math.floor(total / 10) === 0) {
        const progress = Math.round((i / total) * 100);
        self.postMessage({ type: 'PROGRESS', payload: progress } as ProgressMessage);
      }

      // Process
      switch (options.method) {
        case 'sum':
          result += data[i];
          break;
        case 'max':
          result = Math.max(result, data[i]);
          break;
        case 'average':
          result += data[i] / total;
          break;
      }
    }

    self.postMessage({ type: 'RESULT', payload: result } as ResultMessage);
  } catch (error) {
    self.postMessage({ 
      type: 'ERROR', 
      payload: error instanceof Error ? error.message : 'Unknown error' 
    } as ErrorMessage);
  }
}

// main.ts
const worker = new Worker(new URL('./worker.ts', import.meta.url), {
  type: 'module',
});

worker.onmessage = (event: MessageEvent<WorkerToMainMessage>) => {
  const message = event.data;

  switch (message.type) {
    case 'PROGRESS':
      updateProgressBar(message.payload);
      break;
    case 'RESULT':
      handleResult(message.payload);
      break;
    case 'ERROR':
      handleError(message.payload);
      break;
  }
};

// Send typed message
function sendToWorker(message: MainToWorkerMessage) {
  worker.postMessage(message);
}

sendToWorker({
  type: 'PROCESS_DATA',
  payload: {
    data: [1, 2, 3, 4, 5],
    options: { method: 'sum' },
  },
});

// Cancel
sendToWorker({ type: 'CANCEL' });
```

---

## 3. Transferables & SharedArrayBuffer

```typescript
// ════════════════════════════════════════════════════════════════
// TRANSFERABLE OBJECTS
// ════════════════════════════════════════════════════════════════

// Transferables: ArrayBuffer, MessagePort, ImageBitmap, OffscreenCanvas

// ❌ SLOW: Copying large ArrayBuffer
const buffer = new ArrayBuffer(100_000_000); // 100MB
worker.postMessage(buffer); // Copies entire buffer!

// ✅ FAST: Transfer ownership (zero-copy)
const buffer = new ArrayBuffer(100_000_000);
worker.postMessage(buffer, [buffer]); // Transfer, not copy
// Note: buffer is now detached (unusable) in main thread!

// Example: Image processing
async function processImage(imageData: ImageData) {
  const worker = new Worker(new URL('./image-worker.ts', import.meta.url));

  return new Promise<ImageData>((resolve) => {
    worker.onmessage = (e) => {
      resolve(e.data);
      worker.terminate();
    };

    // Transfer the underlying buffer
    worker.postMessage(imageData, [imageData.data.buffer]);
  });
}

// image-worker.ts
self.onmessage = (event: MessageEvent<ImageData>) => {
  const imageData = event.data;
  const data = imageData.data;

  // Apply grayscale filter
  for (let i = 0; i < data.length; i += 4) {
    const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
    data[i] = avg;     // R
    data[i + 1] = avg; // G
    data[i + 2] = avg; // B
    // data[i + 3] is Alpha, leave unchanged
  }

  // Transfer back
  self.postMessage(imageData, [imageData.data.buffer]);
};

// ════════════════════════════════════════════════════════════════
// SHAREDARRAYBUFFER (Shared Memory)
// ════════════════════════════════════════════════════════════════

// Requires Cross-Origin-Isolated headers:
// Cross-Origin-Opener-Policy: same-origin
// Cross-Origin-Embedder-Policy: require-corp

// main.ts
const sharedBuffer = new SharedArrayBuffer(1024);
const sharedArray = new Int32Array(sharedBuffer);

// Initialize
sharedArray[0] = 0; // Counter

// Share with worker
worker.postMessage({ sharedBuffer });

// Worker can read/write same memory!

// worker.ts
self.onmessage = (event) => {
  const { sharedBuffer } = event.data;
  const sharedArray = new Int32Array(sharedBuffer);

  // Atomics for thread-safe operations
  for (let i = 0; i < 1000; i++) {
    Atomics.add(sharedArray, 0, 1); // Thread-safe increment
  }

  // Notify main thread
  Atomics.notify(sharedArray, 0);
};

// main.ts - Wait for worker
Atomics.wait(sharedArray, 0, 0); // Wait while value is 0
console.log('Counter:', sharedArray[0]); // 1000

// ════════════════════════════════════════════════════════════════
// OFFSCREENCANVAS
// ════════════════════════════════════════════════════════════════

// main.ts
const canvas = document.getElementById('canvas') as HTMLCanvasElement;
const offscreen = canvas.transferControlToOffscreen();

worker.postMessage({ canvas: offscreen }, [offscreen]);

// worker.ts
let canvas: OffscreenCanvas;
let ctx: OffscreenCanvasRenderingContext2D;

self.onmessage = (event) => {
  if (event.data.canvas) {
    canvas = event.data.canvas;
    ctx = canvas.getContext('2d')!;
    
    // Render loop in worker
    function render() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.fillStyle = 'blue';
      ctx.fillRect(Math.random() * 100, Math.random() * 100, 50, 50);
      requestAnimationFrame(render);
    }
    render();
  }
};
```

---

## 4. Comlink

```typescript
// ════════════════════════════════════════════════════════════════
// COMLINK - RPC FOR WORKERS
// ════════════════════════════════════════════════════════════════

// npm install comlink

// worker.ts
import * as Comlink from 'comlink';

const api = {
  // Methods exposed to main thread
  async processData(data: number[]): Promise<number> {
    // Heavy computation
    return data.reduce((a, b) => a + b, 0);
  },

  async fetchAndProcess(url: string): Promise<any> {
    const response = await fetch(url);
    const data = await response.json();
    // Process data...
    return data;
  },

  // Can expose callbacks
  async processWithProgress(
    data: number[],
    onProgress: (progress: number) => void
  ): Promise<number> {
    let result = 0;
    for (let i = 0; i < data.length; i++) {
      result += data[i];
      if (i % 1000 === 0) {
        onProgress(Math.round((i / data.length) * 100));
      }
    }
    return result;
  },
};

Comlink.expose(api);

export type WorkerAPI = typeof api;

// main.ts
import * as Comlink from 'comlink';
import type { WorkerAPI } from './worker';

async function main() {
  const worker = new Worker(
    new URL('./worker.ts', import.meta.url),
    { type: 'module' }
  );

  // Wrap worker with Comlink
  const api = Comlink.wrap<WorkerAPI>(worker);

  // Call worker methods like regular async functions!
  const sum = await api.processData([1, 2, 3, 4, 5]);
  console.log('Sum:', sum);

  // With progress callback
  const result = await api.processWithProgress(
    Array.from({ length: 10000 }, (_, i) => i),
    Comlink.proxy((progress) => {
      console.log(`Progress: ${progress}%`);
    })
  );
  console.log('Result:', result);
}

main();

// ════════════════════════════════════════════════════════════════
// COMLINK REACT HOOK
// ════════════════════════════════════════════════════════════════

import { useEffect, useRef } from 'react';
import * as Comlink from 'comlink';

function useComlinkWorker<T>(
  workerFactory: () => Worker
): Comlink.Remote<T> | null {
  const workerRef = useRef<Worker | null>(null);
  const apiRef = useRef<Comlink.Remote<T> | null>(null);

  useEffect(() => {
    workerRef.current = workerFactory();
    apiRef.current = Comlink.wrap<T>(workerRef.current);

    return () => {
      workerRef.current?.terminate();
    };
  }, [workerFactory]);

  return apiRef.current;
}

// Usage
function DataProcessor() {
  const api = useComlinkWorker<WorkerAPI>(
    () => new Worker(new URL('./worker.ts', import.meta.url), { type: 'module' })
  );

  const handleProcess = async () => {
    if (!api) return;
    
    const result = await api.processData([1, 2, 3, 4, 5]);
    console.log('Result:', result);
  };

  return <button onClick={handleProcess}>Process</button>;
}
```

---

## 5. Worker Pools

```typescript
// ════════════════════════════════════════════════════════════════
// WORKER POOL IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

interface Task<T, R> {
  id: number;
  data: T;
  resolve: (result: R) => void;
  reject: (error: Error) => void;
}

class WorkerPool<T, R> {
  private workers: Worker[] = [];
  private availableWorkers: Worker[] = [];
  private taskQueue: Task<T, R>[] = [];
  private taskId = 0;

  constructor(
    workerFactory: () => Worker,
    poolSize: number = navigator.hardwareConcurrency || 4
  ) {
    for (let i = 0; i < poolSize; i++) {
      const worker = workerFactory();
      this.workers.push(worker);
      this.availableWorkers.push(worker);

      worker.onmessage = (event: MessageEvent<R>) => {
        this.availableWorkers.push(worker);
        this.processQueue();
      };
    }
  }

  exec(data: T): Promise<R> {
    return new Promise((resolve, reject) => {
      const task: Task<T, R> = {
        id: this.taskId++,
        data,
        resolve,
        reject,
      };

      this.taskQueue.push(task);
      this.processQueue();
    });
  }

  private processQueue() {
    while (this.taskQueue.length > 0 && this.availableWorkers.length > 0) {
      const task = this.taskQueue.shift()!;
      const worker = this.availableWorkers.shift()!;

      // Store task reference for this worker
      const handler = (event: MessageEvent<R>) => {
        worker.removeEventListener('message', handler);
        task.resolve(event.data);
      };
      worker.addEventListener('message', handler);

      worker.postMessage(task.data);
    }
  }

  terminate() {
    this.workers.forEach((worker) => worker.terminate());
    this.workers = [];
    this.availableWorkers = [];
    this.taskQueue = [];
  }
}

// Usage
const pool = new WorkerPool<number[], number>(
  () => new Worker(new URL('./worker.ts', import.meta.url), { type: 'module' }),
  4 // 4 workers
);

// Process multiple tasks in parallel
async function processAllData(datasets: number[][]) {
  const results = await Promise.all(
    datasets.map((data) => pool.exec(data))
  );
  return results;
}

// Process 100 datasets using 4 workers
const datasets = Array.from({ length: 100 }, () =>
  Array.from({ length: 10000 }, () => Math.random())
);

processAllData(datasets).then((results) => {
  console.log('All processed:', results.length);
  pool.terminate();
});

// ════════════════════════════════════════════════════════════════
// WORKERPOOL LIBRARY
// ════════════════════════════════════════════════════════════════

// npm install workerpool
import workerpool from 'workerpool';

// Create pool
const pool = workerpool.pool('./worker.js');

// Execute tasks
pool.exec('heavyTask', [data])
  .then((result) => console.log('Result:', result))
  .catch((err) => console.error('Error:', err));

// Proxy pattern (like Comlink)
const workerFunctions = await pool.proxy();
const result = await workerFunctions.heavyTask(data);

// Terminate
pool.terminate();
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# WEB WORKER PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Creating too many workers
# Bad
data.forEach(item => {
  const worker = new Worker('./worker.js');
  worker.postMessage(item);
});  # 1000 workers = bad!

# Good
# Use worker pool with limited workers
const pool = new WorkerPool(4);  # Match CPU cores

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Not transferring large data
# Bad
const hugeArray = new Float64Array(10_000_000);
worker.postMessage(hugeArray);  # Copies 80MB!

# Good
worker.postMessage(hugeArray.buffer, [hugeArray.buffer]);  # Transfer

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Trying to access DOM
# Bad (in worker)
document.getElementById('foo');  # Error! No DOM access

# Good
# Send results to main thread to update DOM
self.postMessage({ type: 'UPDATE_DOM', html: '<p>Result</p>' });

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Not handling errors
# Bad
worker.postMessage(data);  # What if worker throws?

# Good
worker.onerror = (e) => console.error('Worker error:', e);
# Or wrap in try-catch in worker and send error messages

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Not terminating workers
# Bad
function processData() {
  const worker = new Worker('./worker.js');
  worker.postMessage(data);
}  # Memory leak - workers never terminated

# Good
worker.onmessage = (e) => {
  handleResult(e.data);
  worker.terminate();  # Clean up
};
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What are Web Workers?"**
> "Web Workers run JavaScript in background threads, separate from the main thread. They enable CPU-intensive operations without blocking the UI. Communication is via message passing (postMessage). They can't access the DOM but can use fetch, IndexedDB, and other APIs."

**Q: "What's the difference between dedicated and shared workers?"**
> "Dedicated workers have a one-to-one relationship with their creator. Shared workers can be shared between multiple tabs/windows/iframes of the same origin - useful for shared state or connection pooling. Service workers are a third type, specialized for network proxying."

**Q: "How do you communicate with workers?"**
> "Using postMessage to send data and onmessage to receive. Messages are copied via structured clone algorithm. For large data, use Transferable objects (ArrayBuffer) for zero-copy transfer - but original becomes unusable. Libraries like Comlink make it feel like calling async functions."

### Intermediate Questions

**Q: "What are Transferable objects?"**
> "Transferables (ArrayBuffer, MessagePort, ImageBitmap, OffscreenCanvas) can be transferred to a worker with zero-copy - ownership moves to the worker. The original becomes detached and unusable. Much faster than copying for large data: worker.postMessage(buffer, [buffer])."

**Q: "What is SharedArrayBuffer?"**
> "SharedArrayBuffer allows true shared memory between threads - both main thread and workers can read/write the same memory. Requires Atomics for thread-safe operations. Needs Cross-Origin-Isolated headers (COOP/COEP) for security. Useful for high-performance parallel computing."

**Q: "How would you implement a worker pool?"**
> "Create fixed number of workers (match CPU cores). Queue tasks when all workers busy. When worker completes, assign next queued task. Benefits: avoid creating too many workers, reuse workers, parallel processing. Libraries like workerpool or Comlink simplify this."

### Advanced Questions

**Q: "When should you use workers vs. stay on main thread?"**
> "Workers for: long-running computation (> 50ms), parsing large JSON, image/video processing, crypto operations. Main thread for: simple calculations, DOM updates, small data. Overhead of message passing means workers aren't worth it for trivial tasks."

**Q: "How do you handle cancellation in workers?"**
> "Options: 1) Set a flag that worker checks periodically. 2) Use AbortController with fetch. 3) Terminate the worker (drastic). 4) With SharedArrayBuffer, use Atomics for coordination. Worker should check cancellation flag in loops."

**Q: "How would you debug workers?"**
> "Chrome DevTools: Sources panel shows worker scripts, can set breakpoints. Console logs from workers appear in main console. Network panel shows worker fetch requests. Performance panel shows worker activity. For complex debugging, add logging messages back to main thread."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              WEB WORKERS CHECKLIST                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  WHEN TO USE:                                                   │
│  □ CPU-intensive computation (> 50ms)                          │
│  □ Parsing large JSON/data                                     │
│  □ Image/video processing                                      │
│  □ Cryptographic operations                                    │
│                                                                 │
│  OPTIMIZATION:                                                  │
│  □ Use Transferables for large ArrayBuffers                    │
│  □ Worker pool instead of many workers                         │
│  □ Match pool size to CPU cores                                │
│                                                                 │
│  PATTERNS:                                                      │
│  □ Typed message protocol                                      │
│  □ Progress reporting                                          │
│  □ Error handling                                              │
│  □ Cancellation support                                        │
│                                                                 │
│  LIBRARIES:                                                     │
│  □ Comlink for RPC-style API                                   │
│  □ workerpool for pool management                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

WORKER TYPES:
┌────────────────────────────────────────────────────────────────┐
│ Dedicated:  One-to-one, most common, heavy computation         │
│ Shared:     Shared between tabs, connection pooling            │
│ Service:    Network proxy, offline, push notifications         │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

