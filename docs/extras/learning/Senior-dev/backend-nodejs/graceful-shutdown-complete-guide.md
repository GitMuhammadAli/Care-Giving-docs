# Graceful Shutdown - Complete Guide

> **MUST REMEMBER**: Graceful shutdown means stopping your application without losing data or dropping active connections. When receiving SIGTERM/SIGINT, stop accepting new requests, finish processing existing ones, close database connections, flush buffers, and only then exit. This is essential for zero-downtime deployments and container orchestration.

---

## How to Explain Like a Senior Developer

"Graceful shutdown is how your application handles being told to stop - think of it as cleaning up your desk before leaving work. When Kubernetes sends SIGTERM or you hit Ctrl+C, your app should: 1) Stop accepting new connections, 2) Wait for in-flight requests to complete, 3) Close database pools and message queues, 4) Flush logs and metrics. If you just call process.exit(), you'll drop requests mid-response, leave database transactions hanging, and lose unflushed data. The challenge is implementing timeouts - you can't wait forever, so after a reasonable period, force shutdown even if work remains."

---

## Core Implementation

### Basic Graceful Shutdown

```typescript
// graceful-shutdown-basic.ts
import http from 'http';
import { promisify } from 'util';

const server = http.createServer((req, res) => {
  // Simulate some work
  setTimeout(() => {
    res.writeHead(200);
    res.end('OK');
  }, 1000);
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});

// Graceful shutdown handler
async function shutdown(signal: string): Promise<void> {
  console.log(`\nReceived ${signal}. Starting graceful shutdown...`);
  
  // Stop accepting new connections
  server.close(() => {
    console.log('HTTP server closed');
    process.exit(0);
  });
  
  // Force exit after timeout
  setTimeout(() => {
    console.error('Forcing shutdown after timeout');
    process.exit(1);
  }, 10000);
}

process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));
```

### Complete Shutdown Manager

```typescript
// shutdown-manager.ts
import { EventEmitter } from 'events';

type ShutdownHandler = () => Promise<void>;
type ShutdownPhase = 'pre-shutdown' | 'connections' | 'cleanup' | 'post-cleanup';

interface ShutdownOptions {
  timeout?: number;
  signals?: NodeJS.Signals[];
  logger?: ShutdownLogger;
}

interface ShutdownLogger {
  info: (message: string, meta?: object) => void;
  warn: (message: string, meta?: object) => void;
  error: (message: string, meta?: object) => void;
}

class ShutdownManager extends EventEmitter {
  private handlers: Map<ShutdownPhase, ShutdownHandler[]> = new Map();
  private isShuttingDown = false;
  private timeout: number;
  private logger: ShutdownLogger;
  
  constructor(options: ShutdownOptions = {}) {
    super();
    
    this.timeout = options.timeout ?? 30000;
    this.logger = options.logger ?? console;
    
    // Initialize phases
    const phases: ShutdownPhase[] = ['pre-shutdown', 'connections', 'cleanup', 'post-cleanup'];
    phases.forEach(phase => this.handlers.set(phase, []));
    
    // Register signal handlers
    const signals = options.signals ?? ['SIGTERM', 'SIGINT'];
    signals.forEach(signal => {
      process.on(signal, () => this.shutdown(signal));
    });
    
    // Handle uncaught errors
    process.on('uncaughtException', (error) => {
      this.logger.error('Uncaught exception', { error });
      this.shutdown('uncaughtException');
    });
    
    process.on('unhandledRejection', (reason) => {
      this.logger.error('Unhandled rejection', { reason });
      this.shutdown('unhandledRejection');
    });
  }
  
  /**
   * Register a shutdown handler for a specific phase
   */
  register(phase: ShutdownPhase, handler: ShutdownHandler): void {
    const handlers = this.handlers.get(phase);
    if (handlers) {
      handlers.push(handler);
    }
  }
  
  /**
   * Convenience method to register HTTP server shutdown
   */
  registerServer(server: import('http').Server, name = 'HTTP Server'): void {
    // Pre-shutdown: mark server as draining
    this.register('pre-shutdown', async () => {
      this.logger.info(`${name}: Stopping new connections`);
    });
    
    // Connections phase: close server and wait for connections
    this.register('connections', async () => {
      return new Promise((resolve, reject) => {
        // Track active connections
        let activeConnections = 0;
        
        server.on('connection', (socket) => {
          activeConnections++;
          socket.on('close', () => activeConnections--);
        });
        
        server.close((err) => {
          if (err) {
            this.logger.error(`${name}: Error closing`, { error: err });
            reject(err);
          } else {
            this.logger.info(`${name}: Closed (${activeConnections} connections remaining)`);
            resolve();
          }
        });
      });
    });
  }
  
  /**
   * Execute shutdown sequence
   */
  async shutdown(reason: string): Promise<void> {
    if (this.isShuttingDown) {
      this.logger.warn('Shutdown already in progress');
      return;
    }
    
    this.isShuttingDown = true;
    this.emit('shutdown-start', reason);
    
    this.logger.info(`Starting graceful shutdown`, { reason, timeout: this.timeout });
    
    // Set up forced exit timeout
    const forceExitTimer = setTimeout(() => {
      this.logger.error('Forced shutdown after timeout');
      process.exit(1);
    }, this.timeout);
    
    // Don't let the timer keep the process alive
    forceExitTimer.unref();
    
    try {
      const phases: ShutdownPhase[] = ['pre-shutdown', 'connections', 'cleanup', 'post-cleanup'];
      
      for (const phase of phases) {
        this.logger.info(`Executing ${phase} handlers`);
        const handlers = this.handlers.get(phase) || [];
        
        await Promise.allSettled(
          handlers.map(async (handler) => {
            try {
              await handler();
            } catch (error) {
              this.logger.error(`Handler error in ${phase}`, { error });
            }
          })
        );
      }
      
      this.emit('shutdown-complete');
      this.logger.info('Graceful shutdown complete');
      
      clearTimeout(forceExitTimer);
      process.exit(0);
      
    } catch (error) {
      this.logger.error('Shutdown error', { error });
      process.exit(1);
    }
  }
  
  /**
   * Check if shutdown is in progress
   */
  get shuttingDown(): boolean {
    return this.isShuttingDown;
  }
}

export const shutdownManager = new ShutdownManager();
export { ShutdownManager, ShutdownPhase };
```

### Express Application with Graceful Shutdown

```typescript
// express-graceful.ts
import express from 'express';
import http from 'http';
import { shutdownManager } from './shutdown-manager';

const app = express();

// Track active requests
let activeRequests = 0;
let isShuttingDown = false;

// Middleware to track requests and handle draining
app.use((req, res, next) => {
  if (isShuttingDown) {
    res.setHeader('Connection', 'close');
    res.status(503).json({
      error: 'Server is shutting down',
      retryAfter: 5,
    });
    return;
  }
  
  activeRequests++;
  
  res.on('finish', () => {
    activeRequests--;
  });
  
  next();
});

// Routes
app.get('/health', (req, res) => {
  res.json({
    status: isShuttingDown ? 'draining' : 'healthy',
    activeRequests,
  });
});

app.get('/slow', async (req, res) => {
  // Simulate slow operation
  await new Promise(resolve => setTimeout(resolve, 5000));
  res.json({ message: 'Done' });
});

// Create HTTP server
const server = http.createServer(app);

// Configure keep-alive timeout (important for graceful shutdown)
server.keepAliveTimeout = 65000; // Slightly higher than ALB/nginx
server.headersTimeout = 66000;

server.listen(3000, () => {
  console.log('Server running on port 3000');
});

// Register shutdown handlers
shutdownManager.register('pre-shutdown', async () => {
  isShuttingDown = true;
  console.log('Server marked as draining');
});

shutdownManager.registerServer(server);

shutdownManager.register('cleanup', async () => {
  // Wait for active requests to complete
  const maxWait = 25000;
  const checkInterval = 500;
  let waited = 0;
  
  while (activeRequests > 0 && waited < maxWait) {
    console.log(`Waiting for ${activeRequests} active requests...`);
    await new Promise(resolve => setTimeout(resolve, checkInterval));
    waited += checkInterval;
  }
  
  if (activeRequests > 0) {
    console.warn(`Proceeding with ${activeRequests} active requests`);
  }
});
```

### Database Connection Graceful Shutdown

```typescript
// database-shutdown.ts
import { Pool, PoolClient } from 'pg';
import { shutdownManager } from './shutdown-manager';

class DatabasePool {
  private pool: Pool;
  private activeQueries = 0;
  
  constructor() {
    this.pool = new Pool({
      host: process.env.DB_HOST,
      port: parseInt(process.env.DB_PORT || '5432'),
      database: process.env.DB_NAME,
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      max: 20,
      idleTimeoutMillis: 30000,
      connectionTimeoutMillis: 2000,
    });
    
    this.pool.on('error', (err) => {
      console.error('Unexpected database error', err);
    });
    
    // Register shutdown handler
    shutdownManager.register('cleanup', () => this.shutdown());
  }
  
  async query<T>(text: string, params?: unknown[]): Promise<T[]> {
    if (shutdownManager.shuttingDown) {
      throw new Error('Database is shutting down');
    }
    
    this.activeQueries++;
    try {
      const result = await this.pool.query(text, params);
      return result.rows as T[];
    } finally {
      this.activeQueries--;
    }
  }
  
  async transaction<T>(
    fn: (client: PoolClient) => Promise<T>
  ): Promise<T> {
    if (shutdownManager.shuttingDown) {
      throw new Error('Database is shutting down');
    }
    
    const client = await this.pool.connect();
    this.activeQueries++;
    
    try {
      await client.query('BEGIN');
      const result = await fn(client);
      await client.query('COMMIT');
      return result;
    } catch (error) {
      await client.query('ROLLBACK');
      throw error;
    } finally {
      client.release();
      this.activeQueries--;
    }
  }
  
  private async shutdown(): Promise<void> {
    console.log('Closing database connections...');
    
    // Wait for active queries
    const maxWait = 10000;
    const startTime = Date.now();
    
    while (this.activeQueries > 0) {
      if (Date.now() - startTime > maxWait) {
        console.warn(`Forcing DB shutdown with ${this.activeQueries} active queries`);
        break;
      }
      console.log(`Waiting for ${this.activeQueries} database queries...`);
      await new Promise(resolve => setTimeout(resolve, 500));
    }
    
    await this.pool.end();
    console.log('Database pool closed');
  }
}

export const db = new DatabasePool();
```

### Message Queue Consumer Graceful Shutdown

```typescript
// queue-consumer-shutdown.ts
import amqp, { Channel, Connection } from 'amqplib';
import { shutdownManager } from './shutdown-manager';

class QueueConsumer {
  private connection: Connection | null = null;
  private channel: Channel | null = null;
  private consumerTag: string | null = null;
  private processingCount = 0;
  
  async connect(url: string): Promise<void> {
    this.connection = await amqp.connect(url);
    this.channel = await this.connection.createChannel();
    
    this.connection.on('error', (err) => {
      console.error('RabbitMQ connection error', err);
    });
    
    // Prefetch to control parallel processing
    await this.channel.prefetch(10);
    
    // Register shutdown handler
    shutdownManager.register('pre-shutdown', () => this.stopConsuming());
    shutdownManager.register('cleanup', () => this.close());
  }
  
  async startConsuming(
    queue: string,
    handler: (message: amqp.ConsumeMessage) => Promise<void>
  ): Promise<void> {
    if (!this.channel) throw new Error('Not connected');
    
    const { consumerTag } = await this.channel.consume(queue, async (msg) => {
      if (!msg) return;
      
      this.processingCount++;
      
      try {
        await handler(msg);
        this.channel?.ack(msg);
      } catch (error) {
        console.error('Message processing error', error);
        // Nack and requeue on error
        this.channel?.nack(msg, false, true);
      } finally {
        this.processingCount--;
      }
    });
    
    this.consumerTag = consumerTag;
    console.log(`Consuming from ${queue} with tag ${consumerTag}`);
  }
  
  private async stopConsuming(): Promise<void> {
    if (!this.channel || !this.consumerTag) return;
    
    console.log('Stopping message consumption...');
    await this.channel.cancel(this.consumerTag);
    this.consumerTag = null;
    console.log('Stopped consuming messages');
  }
  
  private async close(): Promise<void> {
    // Wait for in-flight messages
    const maxWait = 30000;
    const startTime = Date.now();
    
    while (this.processingCount > 0) {
      if (Date.now() - startTime > maxWait) {
        console.warn(`Forcing close with ${this.processingCount} messages processing`);
        break;
      }
      console.log(`Waiting for ${this.processingCount} messages to complete...`);
      await new Promise(resolve => setTimeout(resolve, 500));
    }
    
    if (this.channel) {
      await this.channel.close();
      console.log('Channel closed');
    }
    
    if (this.connection) {
      await this.connection.close();
      console.log('RabbitMQ connection closed');
    }
  }
}

export const queueConsumer = new QueueConsumer();
```

### Background Job Processor Graceful Shutdown

```typescript
// job-processor-shutdown.ts
import { Worker, Queue, Job } from 'bullmq';
import { shutdownManager } from './shutdown-manager';
import Redis from 'ioredis';

class JobProcessor {
  private workers: Worker[] = [];
  private connection: Redis;
  
  constructor() {
    this.connection = new Redis({
      host: process.env.REDIS_HOST,
      port: parseInt(process.env.REDIS_PORT || '6379'),
      maxRetriesPerRequest: null, // Required for BullMQ
    });
    
    // Register shutdown handler
    shutdownManager.register('cleanup', () => this.shutdown());
  }
  
  createWorker(
    queueName: string,
    processor: (job: Job) => Promise<void>
  ): Worker {
    const worker = new Worker(
      queueName,
      async (job) => {
        console.log(`Processing job ${job.id}`);
        await processor(job);
        console.log(`Completed job ${job.id}`);
      },
      {
        connection: this.connection,
        concurrency: 5,
      }
    );
    
    worker.on('failed', (job, err) => {
      console.error(`Job ${job?.id} failed:`, err);
    });
    
    this.workers.push(worker);
    return worker;
  }
  
  private async shutdown(): Promise<void> {
    console.log('Shutting down job workers...');
    
    // Close all workers (waits for current jobs to complete)
    await Promise.all(
      this.workers.map(async (worker) => {
        console.log(`Closing worker for ${worker.name}...`);
        await worker.close();
        console.log(`Worker ${worker.name} closed`);
      })
    );
    
    // Close Redis connection
    await this.connection.quit();
    console.log('Redis connection closed');
  }
}

export const jobProcessor = new JobProcessor();
```

---

## Real-World Scenarios

### Scenario 1: Kubernetes Pod Termination

```typescript
// kubernetes-shutdown.ts
import http from 'http';
import express from 'express';

const app = express();

// Health endpoints for Kubernetes
let isReady = false;
let isHealthy = true;
let isDraining = false;

// Liveness probe - basic health
app.get('/health/live', (req, res) => {
  if (isHealthy) {
    res.status(200).json({ status: 'alive' });
  } else {
    res.status(503).json({ status: 'unhealthy' });
  }
});

// Readiness probe - ready to accept traffic
app.get('/health/ready', (req, res) => {
  if (isReady && !isDraining) {
    res.status(200).json({ status: 'ready' });
  } else {
    res.status(503).json({ 
      status: isDraining ? 'draining' : 'not ready',
    });
  }
});

const server = http.createServer(app);

// Start server
async function start(): Promise<void> {
  // Initialize dependencies
  await initializeDependencies();
  
  server.listen(3000, () => {
    console.log('Server started');
    isReady = true; // Mark ready for traffic
  });
}

async function initializeDependencies(): Promise<void> {
  // Database, cache, etc.
}

// Graceful shutdown for Kubernetes
async function shutdown(): Promise<void> {
  console.log('Received termination signal');
  
  // 1. Mark as not ready (Kubernetes stops sending traffic)
  isDraining = true;
  isReady = false;
  
  // 2. Wait for Kubernetes to update endpoints
  // This gives time for load balancers to drain
  console.log('Waiting for Kubernetes endpoint update...');
  await new Promise(resolve => setTimeout(resolve, 5000));
  
  // 3. Stop accepting new connections
  console.log('Closing HTTP server...');
  await new Promise<void>((resolve) => {
    server.close(() => resolve());
  });
  
  // 4. Cleanup resources
  console.log('Cleaning up resources...');
  await cleanupResources();
  
  console.log('Shutdown complete');
  process.exit(0);
}

async function cleanupResources(): Promise<void> {
  // Close DB connections, flush buffers, etc.
}

// Handle SIGTERM (Kubernetes sends this)
process.on('SIGTERM', shutdown);

// Handle SIGINT (Ctrl+C for local development)
process.on('SIGINT', shutdown);

start().catch((err) => {
  console.error('Startup failed', err);
  process.exit(1);
});
```

### Scenario 2: WebSocket Server Graceful Shutdown

```typescript
// websocket-shutdown.ts
import { WebSocketServer, WebSocket } from 'ws';
import http from 'http';
import { shutdownManager } from './shutdown-manager';

const server = http.createServer();
const wss = new WebSocketServer({ server });

// Track active connections
const connections = new Set<WebSocket>();

wss.on('connection', (ws) => {
  connections.add(ws);
  console.log(`Client connected. Total: ${connections.size}`);
  
  ws.on('close', () => {
    connections.delete(ws);
    console.log(`Client disconnected. Total: ${connections.size}`);
  });
  
  ws.on('message', (data) => {
    // Handle messages
    ws.send(`Echo: ${data}`);
  });
});

server.listen(3000, () => {
  console.log('WebSocket server running on port 3000');
});

// Register shutdown handlers
shutdownManager.register('pre-shutdown', async () => {
  console.log('Notifying WebSocket clients of shutdown...');
  
  // Notify all clients
  connections.forEach((ws) => {
    if (ws.readyState === WebSocket.OPEN) {
      ws.send(JSON.stringify({
        type: 'server-shutdown',
        message: 'Server is shutting down. Please reconnect.',
      }));
    }
  });
  
  // Give clients time to receive the message
  await new Promise(resolve => setTimeout(resolve, 1000));
});

shutdownManager.register('connections', async () => {
  console.log(`Closing ${connections.size} WebSocket connections...`);
  
  // Close all connections gracefully
  connections.forEach((ws) => {
    if (ws.readyState === WebSocket.OPEN) {
      ws.close(1001, 'Server shutting down');
    }
  });
  
  // Wait for connections to close
  const maxWait = 5000;
  const startTime = Date.now();
  
  while (connections.size > 0 && Date.now() - startTime < maxWait) {
    await new Promise(resolve => setTimeout(resolve, 100));
  }
  
  if (connections.size > 0) {
    console.warn(`Force closing ${connections.size} connections`);
    connections.forEach((ws) => ws.terminate());
  }
});

shutdownManager.registerServer(server);
```

---

## Common Pitfalls

### 1. Not Handling In-Flight Requests

```typescript
// ❌ BAD: Drops in-flight requests
process.on('SIGTERM', () => {
  server.close();
  process.exit(0); // Exits immediately!
});

// ✅ GOOD: Wait for requests to complete
process.on('SIGTERM', async () => {
  // Stop accepting new connections
  server.close();
  
  // Wait for existing requests (with timeout)
  await waitForRequests(30000);
  
  process.exit(0);
});

async function waitForRequests(timeout: number): Promise<void> {
  // Implementation
}
```

### 2. Keep-Alive Connections Blocking Shutdown

```typescript
// ❌ BAD: Keep-alive connections prevent server.close() from completing
server.listen(3000);
// Default keepAliveTimeout is 5000ms

// ✅ GOOD: Configure appropriate timeouts
server.keepAliveTimeout = 65000; // Slightly higher than load balancer
server.headersTimeout = 66000; // Must be higher than keepAliveTimeout

// Or destroy idle connections on shutdown
const connections = new Set<import('net').Socket>();

server.on('connection', (socket) => {
  connections.add(socket);
  socket.on('close', () => connections.delete(socket));
});

// In shutdown:
connections.forEach((socket) => {
  // Destroy idle connections
  if (!socket.destroyed) {
    socket.destroy();
  }
});
```

### 3. Not Setting Shutdown Timeout

```typescript
// ❌ BAD: Waits forever
async function shutdown(): Promise<void> {
  await closeAllConnections(); // What if this hangs?
  process.exit(0);
}

// ✅ GOOD: Always have a timeout
async function shutdown(): Promise<void> {
  const shutdownTimer = setTimeout(() => {
    console.error('Shutdown timeout - forcing exit');
    process.exit(1);
  }, 30000);
  
  shutdownTimer.unref(); // Don't let timer keep process alive
  
  try {
    await closeAllConnections();
    process.exit(0);
  } catch (error) {
    process.exit(1);
  }
}

async function closeAllConnections(): Promise<void> {
  // Implementation
}
```

---

## Interview Questions

### Q1: What's the difference between SIGTERM and SIGKILL?

**A:** SIGTERM is a polite request to terminate - your process can catch it and perform cleanup. SIGKILL forcefully terminates immediately - it cannot be caught or ignored. In Kubernetes, SIGTERM is sent first, then SIGKILL after `terminationGracePeriodSeconds` (default 30s) if the process is still running.

### Q2: Why is the order of shutdown operations important?

**A:** You must shutdown in the correct order to prevent errors:
1. Stop accepting new connections (health check returns 503)
2. Wait for load balancer to drain traffic
3. Complete in-flight requests
4. Close database connections
5. Flush logs/metrics
If you close the database before requests complete, those requests will fail.

### Q3: How do you handle graceful shutdown in a clustered Node.js application?

**A:** Send SIGTERM to all workers, each worker stops accepting connections and drains existing requests. The master process waits for workers to exit gracefully, with a timeout for forced termination. PM2 handles this automatically with `pm2 reload` for zero-downtime restarts.

### Q4: What's the purpose of waiting before closing the server in Kubernetes?

**A:** When a pod is terminating, Kubernetes updates endpoints asynchronously. There's a brief period where the pod is terminating but still receiving traffic from load balancers that haven't updated. Waiting 5-10 seconds allows endpoints to update, preventing dropped connections.

---

## Quick Reference Checklist

### Signal Handling
- [ ] Handle SIGTERM (container orchestrators)
- [ ] Handle SIGINT (Ctrl+C / local dev)
- [ ] Set shutdown timeout to prevent hanging

### Shutdown Order
- [ ] 1. Mark service as unhealthy/draining
- [ ] 2. Wait for load balancer drain (5-10s in K8s)
- [ ] 3. Stop accepting new connections
- [ ] 4. Wait for in-flight requests
- [ ] 5. Close database connections
- [ ] 6. Close message queues
- [ ] 7. Flush logs and metrics
- [ ] 8. Exit process

### Keep-Alive Handling
- [ ] Set `keepAliveTimeout` appropriately
- [ ] Set `headersTimeout` > `keepAliveTimeout`
- [ ] Track and destroy idle connections on shutdown

### Kubernetes Specific
- [ ] Implement `/health/ready` endpoint
- [ ] Return 503 when draining
- [ ] Configure `terminationGracePeriodSeconds`

---

*Last updated: February 2026*

