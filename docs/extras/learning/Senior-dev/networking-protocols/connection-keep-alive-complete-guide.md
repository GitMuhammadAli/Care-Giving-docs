# Connection Keep-Alive - Complete Guide

> **MUST REMEMBER**: Keep-Alive reuses TCP connections for multiple HTTP requests, avoiding the overhead of new connections (DNS, TCP handshake, TLS handshake). HTTP/1.1 has Keep-Alive by default. Key settings: timeout (how long to keep idle connection), max requests per connection. Connection pooling on clients prevents exhaustion. HTTP/2 multiplexes many requests on one connection, making Keep-Alive less relevant.

---

## How to Explain Like a Senior Developer

"Every new TCP connection has overhead: DNS lookup, TCP three-way handshake, TLS handshake for HTTPS - that's 2-4 round trips before any data. Keep-Alive lets you reuse connections for multiple requests, which is huge for performance. HTTP/1.1 defaults to Keep-Alive; you have to explicitly say 'Connection: close' to disable it. The server controls timeout (how long it keeps idle connections) and max requests. On the client side, connection pooling manages a set of reusable connections. With HTTP/2, it's even better - one connection handles hundreds of parallel requests via multiplexing. The gotcha is connection exhaustion - if you don't pool or close connections, you can run out of file descriptors or overwhelm backends."

---

## Core Implementation

### HTTP Keep-Alive in Node.js

```typescript
// keep-alive/server.ts
import http from 'http';
import https from 'https';
import fs from 'fs';

// HTTP Server with Keep-Alive settings
const server = http.createServer((req, res) => {
  // Log connection reuse
  const socket = req.socket;
  console.log(`Request on connection ${(socket as any)._connectionId || 'new'}`);
  
  // Keep-Alive is enabled by default in HTTP/1.1
  // You can explicitly set headers
  res.setHeader('Connection', 'keep-alive');
  res.setHeader('Keep-Alive', 'timeout=5, max=100');
  
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ message: 'Hello!' }));
});

// Server keep-alive settings
server.keepAliveTimeout = 5000; // Close idle connections after 5s
server.headersTimeout = 6000;   // Must be > keepAliveTimeout
server.maxRequestsPerSocket = 100; // Max requests per connection (Node 16+)

// Track connection reuse
let connectionId = 0;
server.on('connection', (socket) => {
  (socket as any)._connectionId = ++connectionId;
  console.log(`New connection: ${connectionId}`);
  
  socket.on('close', () => {
    console.log(`Connection closed: ${(socket as any)._connectionId}`);
  });
});

server.listen(3000, () => {
  console.log('Server running with Keep-Alive enabled');
});
```

### HTTP Client with Keep-Alive Agent

```typescript
// keep-alive/client.ts
import http from 'http';
import https from 'https';

// Create Keep-Alive agent for connection pooling
const httpAgent = new http.Agent({
  keepAlive: true,           // Enable keep-alive
  keepAliveMsecs: 1000,      // Initial delay for keep-alive probes
  maxSockets: 25,            // Max concurrent connections per host
  maxFreeSockets: 5,         // Max idle connections to keep
  timeout: 60000,            // Socket timeout
  scheduling: 'fifo',        // 'fifo' or 'lifo'
});

const httpsAgent = new https.Agent({
  keepAlive: true,
  maxSockets: 25,
  maxFreeSockets: 5,
  // TLS session resumption
  maxCachedSessions: 100,
});

// Make request with agent
function makeRequest(url: string): Promise<string> {
  return new Promise((resolve, reject) => {
    const urlObj = new URL(url);
    const agent = urlObj.protocol === 'https:' ? httpsAgent : httpAgent;
    
    const req = (urlObj.protocol === 'https:' ? https : http).request(
      {
        hostname: urlObj.hostname,
        port: urlObj.port,
        path: urlObj.pathname + urlObj.search,
        method: 'GET',
        agent, // Use keep-alive agent
      },
      (res) => {
        let data = '';
        res.on('data', chunk => data += chunk);
        res.on('end', () => resolve(data));
      }
    );
    
    req.on('error', reject);
    req.end();
  });
}

// Compare performance with and without Keep-Alive
async function benchmark(): Promise<void> {
  const url = 'http://localhost:3000/api';
  const iterations = 100;
  
  // Without Keep-Alive
  const noKeepAliveAgent = new http.Agent({ keepAlive: false });
  const startNoKA = Date.now();
  
  for (let i = 0; i < iterations; i++) {
    await new Promise<void>((resolve, reject) => {
      http.get(url, { agent: noKeepAliveAgent }, (res) => {
        res.on('data', () => {});
        res.on('end', resolve);
      }).on('error', reject);
    });
  }
  
  const noKATime = Date.now() - startNoKA;
  
  // With Keep-Alive
  const startKA = Date.now();
  
  for (let i = 0; i < iterations; i++) {
    await makeRequest(url);
  }
  
  const kaTime = Date.now() - startKA;
  
  console.log(`Without Keep-Alive: ${noKATime}ms`);
  console.log(`With Keep-Alive: ${kaTime}ms`);
  console.log(`Improvement: ${((noKATime - kaTime) / noKATime * 100).toFixed(1)}%`);
}
```

### Axios with Keep-Alive

```typescript
// keep-alive/axios-client.ts
import axios, { AxiosInstance } from 'axios';
import http from 'http';
import https from 'https';

// Create Axios instance with Keep-Alive agents
function createApiClient(baseURL: string): AxiosInstance {
  const httpAgent = new http.Agent({
    keepAlive: true,
    maxSockets: 25,
    maxFreeSockets: 5,
  });
  
  const httpsAgent = new https.Agent({
    keepAlive: true,
    maxSockets: 25,
    maxFreeSockets: 5,
  });
  
  return axios.create({
    baseURL,
    httpAgent,
    httpsAgent,
    timeout: 30000,
    headers: {
      'Connection': 'keep-alive',
    },
  });
}

const api = createApiClient('https://api.example.com');

// All requests reuse connections
async function fetchData(): Promise<void> {
  // These requests will reuse the same connection if made quickly
  const [users, products, orders] = await Promise.all([
    api.get('/users'),
    api.get('/products'),
    api.get('/orders'),
  ]);
  
  console.log({ users: users.data, products: products.data, orders: orders.data });
}

// Monitor connection pool status
function getPoolStatus(agent: http.Agent): object {
  return {
    freeSockets: Object.keys(agent.freeSockets).length,
    sockets: Object.keys(agent.sockets).length,
    requests: Object.keys(agent.requests).length,
  };
}
```

### Connection Pooling for Databases

```typescript
// keep-alive/db-pool.ts
import { Pool, PoolConfig } from 'pg';

/**
 * Database connection pooling follows similar principles:
 * - Reuse connections instead of creating new ones
 * - Set appropriate min/max pool sizes
 * - Configure idle timeout
 */

const poolConfig: PoolConfig = {
  host: 'localhost',
  port: 5432,
  database: 'myapp',
  user: 'postgres',
  password: 'password',
  
  // Pool settings
  min: 2,                    // Minimum connections to maintain
  max: 20,                   // Maximum connections
  idleTimeoutMillis: 30000,  // Close idle connections after 30s
  connectionTimeoutMillis: 5000, // Timeout waiting for connection
  
  // Keep-alive settings
  keepAlive: true,
  keepAliveInitialDelayMillis: 10000,
};

const pool = new Pool(poolConfig);

// Monitor pool
pool.on('connect', (client) => {
  console.log('New database connection established');
});

pool.on('remove', (client) => {
  console.log('Database connection removed from pool');
});

pool.on('error', (err, client) => {
  console.error('Unexpected database pool error:', err);
});

// Use pool
async function query(sql: string, params?: any[]): Promise<any> {
  const client = await pool.connect();
  
  try {
    const result = await client.query(sql, params);
    return result.rows;
  } finally {
    client.release(); // Return connection to pool
  }
}

// Pool statistics
async function getPoolStats(): Promise<object> {
  return {
    totalCount: pool.totalCount,
    idleCount: pool.idleCount,
    waitingCount: pool.waitingCount,
  };
}
```

### Nginx Keep-Alive Configuration

```nginx
# nginx/keep-alive.conf

# Upstream (backend) keep-alive
upstream backend {
    server app1:3000;
    server app2:3000;
    
    # Keep connections to backend alive
    keepalive 32;                    # Number of idle keepalive connections
    keepalive_requests 1000;         # Max requests per connection
    keepalive_timeout 60s;           # Idle timeout
}

server {
    listen 80;
    
    # Client-facing keep-alive
    keepalive_timeout 65;            # Keep client connections open
    keepalive_requests 100;          # Max requests per client connection
    
    # Send timeout for slow clients
    send_timeout 60;
    
    location /api {
        proxy_pass http://backend;
        
        # Enable keepalive to backend
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        
        # Timeouts
        proxy_connect_timeout 10s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}

# HTTP/2 settings (implicit multiplexing)
server {
    listen 443 ssl http2;
    
    # HTTP/2 specific
    http2_max_concurrent_streams 128;
    http2_idle_timeout 3m;
}
```

---

## Real-World Scenarios

### Scenario 1: Microservices with Connection Reuse

```typescript
// microservices/http-client.ts
import http from 'http';
import https from 'https';

interface ServiceConfig {
  name: string;
  baseUrl: string;
  maxConnections: number;
}

class ServiceClient {
  private agents: Map<string, http.Agent | https.Agent> = new Map();
  
  constructor(private services: ServiceConfig[]) {
    for (const service of services) {
      const url = new URL(service.baseUrl);
      const isHttps = url.protocol === 'https:';
      
      const agent = isHttps
        ? new https.Agent({
            keepAlive: true,
            maxSockets: service.maxConnections,
            maxFreeSockets: Math.floor(service.maxConnections / 4),
          })
        : new http.Agent({
            keepAlive: true,
            maxSockets: service.maxConnections,
            maxFreeSockets: Math.floor(service.maxConnections / 4),
          });
      
      this.agents.set(service.name, agent);
    }
  }
  
  async request(
    serviceName: string,
    path: string,
    options: { method?: string; body?: any } = {}
  ): Promise<any> {
    const service = this.services.find(s => s.name === serviceName);
    if (!service) throw new Error(`Unknown service: ${serviceName}`);
    
    const agent = this.agents.get(serviceName)!;
    const url = new URL(path, service.baseUrl);
    
    return new Promise((resolve, reject) => {
      const protocol = url.protocol === 'https:' ? https : http;
      
      const req = protocol.request({
        hostname: url.hostname,
        port: url.port,
        path: url.pathname + url.search,
        method: options.method || 'GET',
        agent,
        headers: {
          'Content-Type': 'application/json',
        },
      }, (res) => {
        let data = '';
        res.on('data', chunk => data += chunk);
        res.on('end', () => {
          try {
            resolve(JSON.parse(data));
          } catch {
            resolve(data);
          }
        });
      });
      
      req.on('error', reject);
      
      if (options.body) {
        req.write(JSON.stringify(options.body));
      }
      req.end();
    });
  }
  
  getStats(): Record<string, object> {
    const stats: Record<string, object> = {};
    
    for (const [name, agent] of this.agents) {
      stats[name] = {
        sockets: Object.keys((agent as any).sockets || {}).length,
        freeSockets: Object.keys((agent as any).freeSockets || {}).length,
        requests: Object.keys((agent as any).requests || {}).length,
      };
    }
    
    return stats;
  }
  
  destroy(): void {
    for (const agent of this.agents.values()) {
      agent.destroy();
    }
  }
}

// Usage
const client = new ServiceClient([
  { name: 'users', baseUrl: 'http://user-service:3001', maxConnections: 10 },
  { name: 'orders', baseUrl: 'http://order-service:3002', maxConnections: 20 },
  { name: 'products', baseUrl: 'http://product-service:3003', maxConnections: 15 },
]);

async function handleRequest(): Promise<void> {
  const [user, orders] = await Promise.all([
    client.request('users', '/users/123'),
    client.request('orders', '/orders?userId=123'),
  ]);
  
  console.log({ user, orders });
  console.log('Connection stats:', client.getStats());
}
```

### Scenario 2: Graceful Connection Draining

```typescript
// graceful/connection-drain.ts
import http from 'http';

const server = http.createServer((req, res) => {
  // Simulate work
  setTimeout(() => {
    res.writeHead(200);
    res.end('OK');
  }, 100);
});

// Track active connections
const connections = new Set<any>();

server.on('connection', (socket) => {
  connections.add(socket);
  socket.on('close', () => connections.delete(socket));
});

// Graceful shutdown
async function gracefulShutdown(): Promise<void> {
  console.log('Starting graceful shutdown...');
  
  // 1. Stop accepting new connections
  server.close(() => {
    console.log('Server closed to new connections');
  });
  
  // 2. Set a shorter keep-alive timeout
  server.keepAliveTimeout = 1;
  
  // 3. Mark existing connections to close after current request
  for (const socket of connections) {
    // Send Connection: close on next response
    socket._httpMessage?.setHeader('Connection', 'close');
  }
  
  // 4. Wait for connections to drain or force close after timeout
  const drainTimeout = 30000; // 30 seconds
  const start = Date.now();
  
  while (connections.size > 0 && Date.now() - start < drainTimeout) {
    console.log(`Waiting for ${connections.size} connections to close...`);
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  
  // 5. Force close remaining connections
  if (connections.size > 0) {
    console.log(`Force closing ${connections.size} connections`);
    for (const socket of connections) {
      socket.destroy();
    }
  }
  
  console.log('Shutdown complete');
  process.exit(0);
}

process.on('SIGTERM', gracefulShutdown);
process.on('SIGINT', gracefulShutdown);

server.listen(3000);
```

---

## Common Pitfalls

### 1. Not Using Keep-Alive Agent

```typescript
// ❌ BAD: New connection for every request
for (let i = 0; i < 100; i++) {
  await fetch('http://api.example.com/data');
  // Creates 100 new TCP connections!
}

// ✅ GOOD: Reuse connections with agent
import http from 'http';
const agent = new http.Agent({ keepAlive: true });

for (let i = 0; i < 100; i++) {
  await new Promise((resolve) => {
    http.get('http://api.example.com/data', { agent }, resolve);
  });
  // Reuses connections!
}
```

### 2. Connection Exhaustion

```typescript
// ❌ BAD: Unlimited connections
const agent = new http.Agent({ keepAlive: true });
// maxSockets defaults to Infinity!

// ✅ GOOD: Limit connections
const agent = new http.Agent({
  keepAlive: true,
  maxSockets: 25,     // Limit per host
  maxTotalSockets: 100, // Limit total (Node 16+)
});
```

### 3. Forgetting to Configure Nginx Upstream Keep-Alive

```nginx
# ❌ BAD: New connection to backend for each request
upstream backend {
    server app:3000;
}

location / {
    proxy_pass http://backend;
    # No keepalive = new connection each time!
}

# ✅ GOOD: Keep connections to backend
upstream backend {
    server app:3000;
    keepalive 32;
}

location / {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Connection "";
}
```

---

## Interview Questions

### Q1: What is HTTP Keep-Alive and why is it important?

**A:** Keep-Alive allows reusing TCP connections for multiple HTTP requests. Without it, every request requires: DNS lookup, TCP handshake (1 RTT), TLS handshake (1-2 RTT for HTTPS). With Keep-Alive, subsequent requests skip these steps. HTTP/1.1 enables it by default. It significantly reduces latency and server load.

### Q2: How does HTTP/2 handle connections differently?

**A:** HTTP/2 uses a single connection for all requests via multiplexing - many parallel streams on one connection. Keep-Alive becomes implicit and more efficient. No head-of-line blocking at HTTP level (though still at TCP level). One connection can handle hundreds of concurrent requests.

### Q3: What is connection pooling and how should you configure it?

**A:** Connection pooling maintains a set of reusable connections. Key settings: maxSockets (concurrent connections per host), maxFreeSockets (idle connections to keep), timeout (when to close idle), keepAlive (enable reuse). Set based on expected concurrency and backend capacity. Too few = queuing, too many = backend overwhelm.

### Q4: How do you handle graceful shutdown with Keep-Alive?

**A:** 1) Stop accepting new connections. 2) Set keep-alive timeout very low. 3) Send "Connection: close" header on pending responses. 4) Wait for connections to drain (with timeout). 5) Force close remaining connections. This ensures in-flight requests complete while preventing new long-lived connections.

---

## Quick Reference Checklist

### Server Configuration
- [ ] Set appropriate keepAliveTimeout
- [ ] Set headersTimeout > keepAliveTimeout
- [ ] Configure maxRequestsPerSocket
- [ ] Handle graceful shutdown

### Client Configuration
- [ ] Use Keep-Alive agent
- [ ] Set appropriate maxSockets
- [ ] Set maxFreeSockets for idle connections
- [ ] Configure timeout

### Nginx/Proxy
- [ ] Enable upstream keepalive
- [ ] Set proxy_http_version 1.1
- [ ] Clear Connection header
- [ ] Configure keepalive_timeout

### Monitoring
- [ ] Track active connections
- [ ] Monitor connection reuse rate
- [ ] Alert on connection exhaustion
- [ ] Log connection errors

---

*Last updated: February 2026*

