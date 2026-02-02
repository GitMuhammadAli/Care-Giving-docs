# Clustering - Complete Guide

> **MUST REMEMBER**: Node.js runs single-threaded by default. Clustering creates multiple worker processes that share the same server port, enabling you to utilize all CPU cores. The master process manages workers and distributes incoming connections. Use PM2 or the built-in cluster module to achieve horizontal scaling on a single machine.

---

## How to Explain Like a Senior Developer

"Node.js is single-threaded, which means out of the box, your 8-core server is running at 12.5% capacity. Clustering fixes this by forking multiple identical processes - one per CPU core - that all share the same port. The OS handles distributing incoming connections. Think of it like a restaurant with one host (master) directing customers to multiple servers (workers). The key is that workers are independent processes, so if one crashes, the others keep running. PM2 abstracts this complexity and adds zero-downtime reloads, but understanding the cluster module helps you debug production issues and implement custom behaviors."

---

## Core Implementation

### Basic Cluster Setup

```typescript
// cluster-basic.ts
import cluster from 'cluster';
import http from 'http';
import os from 'os';

const numCPUs = os.cpus().length;

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} is running`);
  console.log(`Forking ${numCPUs} workers...`);
  
  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  
  // Handle worker exit
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died (${signal || code})`);
    
    // Optionally restart worker
    if (!signal) { // Don't restart on SIGTERM
      console.log('Starting new worker...');
      cluster.fork();
    }
  });
  
  // Handle worker online
  cluster.on('online', (worker) => {
    console.log(`Worker ${worker.process.pid} is online`);
  });
  
} else {
  // Workers share TCP connection
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Hello from worker ${process.pid}\n`);
  }).listen(3000);
  
  console.log(`Worker ${process.pid} started`);
}
```

### Advanced Cluster with Graceful Shutdown

```typescript
// cluster-advanced.ts
import cluster, { Worker } from 'cluster';
import http from 'http';
import os from 'os';

interface WorkerStatus {
  pid: number;
  startTime: Date;
  requestCount: number;
  isShuttingDown: boolean;
}

const workerStatuses = new Map<number, WorkerStatus>();

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;
  
  console.log(`Primary ${process.pid} starting ${numCPUs} workers`);
  
  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    forkWorker();
  }
  
  function forkWorker(): Worker {
    const worker = cluster.fork();
    
    workerStatuses.set(worker.id, {
      pid: worker.process.pid!,
      startTime: new Date(),
      requestCount: 0,
      isShuttingDown: false,
    });
    
    return worker;
  }
  
  // Listen for messages from workers
  cluster.on('message', (worker, message) => {
    if (message.type === 'request') {
      const status = workerStatuses.get(worker.id);
      if (status) {
        status.requestCount++;
      }
    }
  });
  
  // Handle worker exit
  cluster.on('exit', (worker, code, signal) => {
    const status = workerStatuses.get(worker.id);
    workerStatuses.delete(worker.id);
    
    console.log(`Worker ${worker.process.pid} exited`, {
      code,
      signal,
      uptime: status ? Date.now() - status.startTime.getTime() : 0,
      requests: status?.requestCount || 0,
    });
    
    // Restart unless shutting down
    if (!status?.isShuttingDown) {
      console.log('Starting replacement worker...');
      forkWorker();
    }
  });
  
  // Graceful shutdown
  function gracefulShutdown(): void {
    console.log('Initiating graceful shutdown...');
    
    for (const [id, status] of workerStatuses) {
      status.isShuttingDown = true;
      const worker = cluster.workers?.[id];
      
      if (worker) {
        worker.send({ type: 'shutdown' });
        
        // Force kill after timeout
        setTimeout(() => {
          if (!worker.isDead()) {
            console.log(`Force killing worker ${worker.process.pid}`);
            worker.kill('SIGKILL');
          }
        }, 10000);
      }
    }
  }
  
  process.on('SIGTERM', gracefulShutdown);
  process.on('SIGINT', gracefulShutdown);
  
  // Rolling restart
  function rollingRestart(): void {
    const workerIds = Object.keys(cluster.workers || {});
    let index = 0;
    
    function restartNext(): void {
      if (index >= workerIds.length) {
        console.log('Rolling restart complete');
        return;
      }
      
      const workerId = parseInt(workerIds[index++]);
      const worker = cluster.workers?.[workerId];
      
      if (worker) {
        const newWorker = forkWorker();
        
        // Wait for new worker to be ready
        newWorker.once('listening', () => {
          console.log(`New worker ${newWorker.process.pid} ready, killing old`);
          worker.send({ type: 'shutdown' });
          
          setTimeout(restartNext, 1000);
        });
      } else {
        restartNext();
      }
    }
    
    restartNext();
  }
  
  // Expose rolling restart via signal
  process.on('SIGHUP', rollingRestart);
  
} else {
  // Worker process
  const server = http.createServer((req, res) => {
    // Notify primary of request
    process.send?.({ type: 'request' });
    
    res.writeHead(200);
    res.end(`Worker ${process.pid}\n`);
  });
  
  server.listen(3000);
  
  // Handle shutdown message
  process.on('message', (message: { type: string }) => {
    if (message.type === 'shutdown') {
      console.log(`Worker ${process.pid} shutting down...`);
      
      server.close(() => {
        console.log(`Worker ${process.pid} closed all connections`);
        process.exit(0);
      });
      
      // Stop accepting new connections immediately
      setTimeout(() => {
        process.exit(0);
      }, 5000);
    }
  });
}
```

### Sticky Sessions for WebSockets

```typescript
// sticky-sessions.ts
import cluster from 'cluster';
import http from 'http';
import net from 'net';
import os from 'os';
import crypto from 'crypto';

const numCPUs = os.cpus().length;

if (cluster.isPrimary) {
  const workers: cluster.Worker[] = [];
  
  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    workers.push(cluster.fork());
  }
  
  // Create the main server that distributes connections
  const server = net.createServer({ pauseOnConnect: true }, (connection) => {
    // Get client IP for consistent hashing
    const remoteAddress = connection.remoteAddress || '';
    const hash = crypto.createHash('md5').update(remoteAddress).digest();
    const workerIndex = hash.readUInt32LE(0) % workers.length;
    const worker = workers[workerIndex];
    
    // Send connection to specific worker
    worker.send('sticky-session:connection', connection);
  });
  
  server.listen(3000);
  console.log(`Primary listening on port 3000`);
  
  cluster.on('exit', (worker, code, signal) => {
    const index = workers.indexOf(worker);
    if (index > -1) {
      workers[index] = cluster.fork();
    }
  });
  
} else {
  // Worker creates HTTP server
  const server = http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`Worker ${process.pid}\n`);
  });
  
  // Don't bind to port - connections come from primary
  server.listen(0, 'localhost');
  
  // Handle incoming connections from primary
  process.on('message', (message, connection: net.Socket) => {
    if (message === 'sticky-session:connection') {
      // Emit connection to server
      server.emit('connection', connection);
      connection.resume();
    }
  });
}
```

### PM2 Configuration

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'api-server',
    script: './dist/server.js',
    instances: 'max', // Use all CPUs
    exec_mode: 'cluster',
    
    // Environment variables
    env: {
      NODE_ENV: 'development',
      PORT: 3000,
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000,
    },
    
    // Restart behavior
    max_memory_restart: '1G',
    restart_delay: 1000,
    max_restarts: 10,
    min_uptime: '10s',
    
    // Logging
    log_file: './logs/combined.log',
    error_file: './logs/error.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
    merge_logs: true,
    
    // Graceful shutdown
    kill_timeout: 5000,
    wait_ready: true,
    listen_timeout: 10000,
    
    // Watching (development)
    watch: false,
    ignore_watch: ['node_modules', 'logs'],
    
    // Advanced
    node_args: '--max-old-space-size=4096',
    source_map_support: true,
  }],
};
```

### PM2 Process Ready Signal

```typescript
// server-pm2.ts
import http from 'http';

const server = http.createServer((req, res) => {
  res.writeHead(200);
  res.end('OK');
});

server.listen(process.env.PORT || 3000, () => {
  console.log(`Server running on port ${process.env.PORT || 3000}`);
  
  // Signal PM2 that process is ready
  if (process.send) {
    process.send('ready');
  }
});

// Graceful shutdown
process.on('SIGINT', () => {
  console.log('Received SIGINT, shutting down gracefully...');
  
  server.close((err) => {
    if (err) {
      console.error('Error during shutdown:', err);
      process.exit(1);
    }
    
    console.log('Server closed');
    process.exit(0);
  });
  
  // Force close after 10s
  setTimeout(() => {
    console.error('Forcing shutdown after timeout');
    process.exit(1);
  }, 10000);
});
```

### Cluster with Express

```typescript
// express-cluster.ts
import cluster from 'cluster';
import express from 'express';
import os from 'os';

function createApp(): express.Application {
  const app = express();
  
  // Middleware to add worker info
  app.use((req, res, next) => {
    res.setHeader('X-Worker-ID', `${process.pid}`);
    next();
  });
  
  app.get('/health', (req, res) => {
    res.json({
      status: 'healthy',
      worker: process.pid,
      uptime: process.uptime(),
      memory: process.memoryUsage(),
    });
  });
  
  app.get('/heavy', async (req, res) => {
    // Simulate heavy computation
    const start = Date.now();
    let result = 0;
    for (let i = 0; i < 1e8; i++) {
      result += Math.sqrt(i);
    }
    
    res.json({
      worker: process.pid,
      duration: Date.now() - start,
      result,
    });
  });
  
  return app;
}

if (cluster.isPrimary) {
  const numWorkers = os.cpus().length;
  
  console.log(`Primary ${process.pid} starting ${numWorkers} workers`);
  
  for (let i = 0; i < numWorkers; i++) {
    cluster.fork();
  }
  
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork();
  });
  
} else {
  const app = createApp();
  
  app.listen(3000, () => {
    console.log(`Worker ${process.pid} listening on port 3000`);
  });
}
```

### Inter-Process Communication (IPC)

```typescript
// ipc-example.ts
import cluster from 'cluster';
import os from 'os';

interface IPCMessage {
  type: string;
  payload?: unknown;
  from?: number;
}

if (cluster.isPrimary) {
  const workers = new Map<number, cluster.Worker>();
  
  // Shared state (simplified - use Redis in production)
  let sharedCounter = 0;
  
  // Fork workers
  for (let i = 0; i < os.cpus().length; i++) {
    const worker = cluster.fork();
    workers.set(worker.id, worker);
  }
  
  // Handle messages from workers
  cluster.on('message', (worker, message: IPCMessage) => {
    switch (message.type) {
      case 'increment':
        sharedCounter++;
        // Broadcast new value to all workers
        workers.forEach(w => {
          w.send({ type: 'counter-update', payload: sharedCounter });
        });
        break;
        
      case 'get-counter':
        worker.send({ type: 'counter-value', payload: sharedCounter });
        break;
        
      case 'broadcast':
        // Forward message to all other workers
        workers.forEach(w => {
          if (w.id !== worker.id) {
            w.send({
              type: 'broadcast-message',
              payload: message.payload,
              from: worker.id,
            });
          }
        });
        break;
    }
  });
  
  cluster.on('exit', (worker) => {
    workers.delete(worker.id);
    const newWorker = cluster.fork();
    workers.set(newWorker.id, newWorker);
  });
  
} else {
  // Worker process
  let localCounter = 0;
  
  // Handle messages from primary
  process.on('message', (message: IPCMessage) => {
    switch (message.type) {
      case 'counter-update':
        localCounter = message.payload as number;
        console.log(`Worker ${process.pid}: counter updated to ${localCounter}`);
        break;
        
      case 'counter-value':
        console.log(`Worker ${process.pid}: counter is ${message.payload}`);
        break;
        
      case 'broadcast-message':
        console.log(`Worker ${process.pid}: received broadcast from ${message.from}:`, message.payload);
        break;
    }
  });
  
  // Example: increment counter every 5 seconds
  setInterval(() => {
    process.send?.({ type: 'increment' });
  }, 5000);
}
```

---

## Real-World Scenarios

### Scenario 1: Zero-Downtime Deployment

```typescript
// zero-downtime-cluster.ts
import cluster from 'cluster';
import http from 'http';
import os from 'os';

if (cluster.isPrimary) {
  const workers = new Set<cluster.Worker>();
  const pendingShutdown = new Set<cluster.Worker>();
  
  function spawnWorker(): cluster.Worker {
    const worker = cluster.fork();
    workers.add(worker);
    
    worker.on('exit', () => {
      workers.delete(worker);
      pendingShutdown.delete(worker);
    });
    
    return worker;
  }
  
  // Initial workers
  for (let i = 0; i < os.cpus().length; i++) {
    spawnWorker();
  }
  
  // Zero-downtime restart
  async function zeroDowntimeRestart(): Promise<void> {
    console.log('Starting zero-downtime restart...');
    
    const oldWorkers = [...workers];
    
    for (const oldWorker of oldWorkers) {
      // Spawn new worker
      const newWorker = spawnWorker();
      
      // Wait for new worker to be ready
      await new Promise<void>((resolve) => {
        newWorker.once('listening', resolve);
      });
      
      console.log(`New worker ${newWorker.process.pid} ready`);
      
      // Gracefully shutdown old worker
      pendingShutdown.add(oldWorker);
      oldWorker.send({ type: 'shutdown' });
      
      // Wait before processing next worker
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
    
    console.log('Zero-downtime restart complete');
  }
  
  // Trigger restart via SIGHUP
  process.on('SIGHUP', () => {
    zeroDowntimeRestart().catch(console.error);
  });
  
  // Keep workers alive
  cluster.on('exit', (worker, code) => {
    if (!pendingShutdown.has(worker) && code !== 0) {
      console.log(`Worker ${worker.process.pid} died unexpectedly, restarting...`);
      spawnWorker();
    }
  });
  
} else {
  let isShuttingDown = false;
  const connections = new Set<http.IncomingMessage['socket']>();
  
  const server = http.createServer((req, res) => {
    if (isShuttingDown) {
      res.writeHead(503);
      res.end('Server is shutting down');
      return;
    }
    
    connections.add(req.socket);
    req.socket.once('close', () => connections.delete(req.socket));
    
    res.writeHead(200);
    res.end(`Worker ${process.pid}\n`);
  });
  
  server.listen(3000, () => {
    console.log(`Worker ${process.pid} listening`);
  });
  
  process.on('message', (msg: { type: string }) => {
    if (msg.type === 'shutdown') {
      isShuttingDown = true;
      console.log(`Worker ${process.pid} shutting down...`);
      
      server.close(() => {
        console.log(`Worker ${process.pid} closed`);
        process.exit(0);
      });
      
      // Close existing connections gracefully
      connections.forEach(socket => {
        socket.end();
      });
      
      // Force exit after timeout
      setTimeout(() => process.exit(0), 5000);
    }
  });
}
```

### Scenario 2: Load-Based Auto-Scaling

```typescript
// auto-scaling-cluster.ts
import cluster from 'cluster';
import http from 'http';
import os from 'os';

if (cluster.isPrimary) {
  const minWorkers = 2;
  const maxWorkers = os.cpus().length;
  const workers = new Map<number, { worker: cluster.Worker; load: number }>();
  
  let totalRequests = 0;
  let lastCheckRequests = 0;
  
  function spawnWorker(): void {
    if (workers.size >= maxWorkers) return;
    
    const worker = cluster.fork();
    workers.set(worker.id, { worker, load: 0 });
    
    console.log(`Spawned worker ${worker.process.pid}. Total: ${workers.size}`);
  }
  
  function removeWorker(): void {
    if (workers.size <= minWorkers) return;
    
    // Find worker with lowest load
    let minLoad = Infinity;
    let targetId: number | null = null;
    
    workers.forEach(({ load }, id) => {
      if (load < minLoad) {
        minLoad = load;
        targetId = id;
      }
    });
    
    if (targetId !== null) {
      const { worker } = workers.get(targetId)!;
      workers.delete(targetId);
      worker.send({ type: 'shutdown' });
      console.log(`Removing worker ${worker.process.pid}. Total: ${workers.size}`);
    }
  }
  
  // Initial workers
  for (let i = 0; i < minWorkers; i++) {
    spawnWorker();
  }
  
  // Handle worker messages
  cluster.on('message', (worker, msg: { type: string; count?: number }) => {
    if (msg.type === 'request-complete') {
      totalRequests++;
      const workerData = workers.get(worker.id);
      if (workerData) {
        workerData.load = msg.count || 0;
      }
    }
  });
  
  // Auto-scaling logic
  setInterval(() => {
    const requestsPerSecond = (totalRequests - lastCheckRequests) / 5;
    lastCheckRequests = totalRequests;
    
    const avgLoad = [...workers.values()].reduce((sum, w) => sum + w.load, 0) / workers.size;
    
    console.log(`RPS: ${requestsPerSecond.toFixed(2)}, Avg Load: ${avgLoad.toFixed(2)}, Workers: ${workers.size}`);
    
    // Scale up if high load
    if (avgLoad > 100 && workers.size < maxWorkers) {
      spawnWorker();
    }
    
    // Scale down if low load
    if (avgLoad < 20 && workers.size > minWorkers) {
      removeWorker();
    }
  }, 5000);
  
  cluster.on('exit', (worker, code) => {
    workers.delete(worker.id);
    if (code !== 0) {
      spawnWorker();
    }
  });
  
} else {
  let activeRequests = 0;
  
  const server = http.createServer((req, res) => {
    activeRequests++;
    
    // Simulate work
    setTimeout(() => {
      activeRequests--;
      process.send?.({ type: 'request-complete', count: activeRequests });
      res.writeHead(200);
      res.end('OK');
    }, Math.random() * 100);
  });
  
  server.listen(3000);
  
  process.on('message', (msg: { type: string }) => {
    if (msg.type === 'shutdown') {
      server.close(() => process.exit(0));
    }
  });
}
```

---

## Common Pitfalls

### 1. Sharing State Between Workers

```typescript
// ❌ BAD: Assuming state is shared
// Each worker has its own memory!
let counter = 0; // This is NOT shared!

app.post('/increment', (req, res) => {
  counter++; // Each worker has different value
  res.json({ counter });
});

// ✅ GOOD: Use external store for shared state
import Redis from 'ioredis';
const redis = new Redis();

app.post('/increment', async (req, res) => {
  const counter = await redis.incr('counter');
  res.json({ counter });
});
```

### 2. Not Handling Worker Crashes

```typescript
// ❌ BAD: No worker restart
cluster.on('exit', (worker) => {
  console.log(`Worker ${worker.process.pid} died`);
  // Worker gone forever!
});

// ✅ GOOD: Restart crashed workers
cluster.on('exit', (worker, code, signal) => {
  if (signal !== 'SIGTERM') {
    console.log(`Worker ${worker.process.pid} crashed, restarting...`);
    cluster.fork();
  }
});
```

### 3. Memory Leaks Across Workers

```typescript
// ❌ BAD: Memory leak affects single worker, then it gets restarted
// but leak continues in new worker

// ✅ GOOD: Monitor memory and restart proactively
setInterval(() => {
  const used = process.memoryUsage().heapUsed;
  const limit = 500 * 1024 * 1024; // 500MB
  
  if (used > limit) {
    console.log('Memory limit exceeded, requesting graceful shutdown');
    process.send?.({ type: 'memory-exceeded' });
  }
}, 30000);
```

---

## Interview Questions

### Q1: How does Node.js clustering achieve load balancing?

**A:** The primary process accepts incoming connections and distributes them to workers using one of two approaches:
1. **Round-robin** (default on non-Windows): Primary accepts connections and distributes evenly
2. **OS-level**: Workers compete for connections (Windows default)
The primary doesn't process requests; it only manages connection distribution. Each worker is a separate process with its own memory and event loop.

### Q2: What's the difference between cluster workers and worker threads?

**A:** Cluster workers are separate processes with isolated memory, ideal for scaling across CPU cores. Worker threads share memory within a single process, better for CPU-intensive tasks within one application. Use clustering for horizontal scaling of servers; use worker threads for parallel computation without forking entire processes.

### Q3: How do you handle sessions in a clustered environment?

**A:** Since workers don't share memory:
1. **External session store**: Use Redis or database for session storage
2. **Sticky sessions**: Route same client to same worker using IP hash or cookie
3. **Stateless authentication**: Use JWTs that don't require server-side state
Sticky sessions are needed for WebSocket connections since the connection must stay with one worker.

### Q4: How does PM2 achieve zero-downtime restarts?

**A:** PM2 performs rolling restarts:
1. Spawns new worker
2. Waits for new worker's 'ready' signal
3. Sends SIGINT to old worker
4. Old worker stops accepting connections and finishes existing requests
5. Repeats for each worker
This ensures at least one worker is always handling requests.

---

## Quick Reference Checklist

### Cluster Setup
- [ ] Use `cluster.isPrimary` to detect primary process
- [ ] Fork one worker per CPU core
- [ ] Handle worker exit and restart
- [ ] Implement graceful shutdown

### PM2 Best Practices
- [ ] Use `ecosystem.config.js` for configuration
- [ ] Set `max_memory_restart` to prevent memory leaks
- [ ] Send 'ready' signal when server is listening
- [ ] Handle SIGINT for graceful shutdown

### Shared State
- [ ] Use Redis or database for shared state
- [ ] Implement sticky sessions for WebSockets
- [ ] Use IPC for primary-worker communication

### Monitoring
- [ ] Track worker health and restart counts
- [ ] Monitor memory usage per worker
- [ ] Log worker lifecycle events

---

*Last updated: February 2026*

