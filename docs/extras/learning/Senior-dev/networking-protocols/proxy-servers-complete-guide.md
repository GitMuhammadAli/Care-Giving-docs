# Proxy Servers - Complete Guide

> **MUST REMEMBER**: Forward proxy sits between client and internet (client knows about it) - used for caching, filtering, anonymity. Reverse proxy sits in front of servers (client doesn't know) - used for load balancing, SSL termination, caching, security. Nginx is the most common reverse proxy. Key concepts: upstream servers, health checks, connection pooling, header forwarding (X-Forwarded-For).

---

## How to Explain Like a Senior Developer

"A proxy is a middleman for network requests. Forward proxy: you configure your browser to use it, it makes requests on your behalf - useful for corporate filtering, VPNs, or caching. Reverse proxy: sits in front of your servers, clients think they're talking to it directly - this is what Nginx does. Reverse proxies handle SSL termination (decrypt HTTPS once), load balancing (distribute to backend servers), caching (reduce backend load), and add a security layer. In production, you almost always have a reverse proxy - it's your first line of defense and optimization."

---

## Core Implementation

### Basic Nginx Reverse Proxy

```nginx
# nginx/basic-proxy.conf

# Define upstream servers (backend)
upstream api_servers {
    # Load balancing methods:
    # (default) round-robin
    # least_conn - send to least busy
    # ip_hash - sticky sessions by client IP
    # hash $request_uri - consistent hashing
    
    least_conn;
    
    server backend1.local:3000 weight=5;  # Receives more traffic
    server backend2.local:3000 weight=3;
    server backend3.local:3000 weight=2;
    
    # Health check (passive)
    # max_fails - number of failures before marked down
    # fail_timeout - time to wait before retrying
    server backend4.local:3000 max_fails=3 fail_timeout=30s;
    
    # Backup server - only used when others are down
    server backup.local:3000 backup;
    
    # Keep connections to backend alive
    keepalive 32;
}

server {
    listen 80;
    server_name api.example.com;
    
    # Redirect to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;
    
    # SSL configuration
    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # Proxy settings
    location / {
        proxy_pass http://api_servers;
        
        # Pass original host and IP
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        # Buffer settings
        proxy_buffer_size 128k;
        proxy_buffers 4 256k;
        proxy_busy_buffers_size 256k;
        
        # Don't pass server errors to client
        proxy_intercept_errors on;
        error_page 502 503 504 /50x.html;
    }
    
    location /50x.html {
        root /usr/share/nginx/html;
        internal;
    }
}
```

### Nginx Caching Reverse Proxy

```nginx
# nginx/caching-proxy.conf

# Define cache zone
proxy_cache_path /var/cache/nginx/api 
    levels=1:2 
    keys_zone=api_cache:10m    # 10MB for keys
    max_size=1g                 # 1GB for cached content
    inactive=60m                # Remove if not accessed in 60min
    use_temp_path=off;

upstream api_servers {
    server backend:3000;
    keepalive 32;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;
    
    # SSL config...
    
    # Cache settings
    location /api/products {
        proxy_pass http://api_servers;
        
        # Enable caching
        proxy_cache api_cache;
        proxy_cache_valid 200 10m;      # Cache 200 responses for 10 min
        proxy_cache_valid 404 1m;        # Cache 404 for 1 min
        proxy_cache_valid any 5m;        # Cache everything else for 5 min
        
        # Cache key
        proxy_cache_key "$scheme$request_method$host$request_uri";
        
        # Add cache status header
        add_header X-Cache-Status $upstream_cache_status;
        
        # Bypass cache for specific conditions
        proxy_cache_bypass $http_cache_control;
        
        # Use stale cache while revalidating
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503;
        proxy_cache_background_update on;
        proxy_cache_lock on;
        
        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # Don't cache authenticated requests
    location /api/user {
        proxy_pass http://api_servers;
        proxy_cache off;
        
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Node.js HTTP Proxy

```typescript
// proxy/http-proxy.ts
import http from 'http';
import https from 'https';
import { URL } from 'url';

interface ProxyOptions {
  target: string;
  changeOrigin?: boolean;
  pathRewrite?: Record<string, string>;
  onProxyReq?: (proxyReq: http.ClientRequest, req: http.IncomingMessage) => void;
  onProxyRes?: (proxyRes: http.IncomingMessage, req: http.IncomingMessage, res: http.ServerResponse) => void;
}

function createProxyServer(options: ProxyOptions): http.Server {
  const targetUrl = new URL(options.target);
  
  return http.createServer((req, res) => {
    let path = req.url || '/';
    
    // Path rewriting
    if (options.pathRewrite) {
      for (const [pattern, replacement] of Object.entries(options.pathRewrite)) {
        path = path.replace(new RegExp(pattern), replacement);
      }
    }
    
    // Determine protocol
    const protocol = targetUrl.protocol === 'https:' ? https : http;
    
    const proxyOptions: http.RequestOptions = {
      hostname: targetUrl.hostname,
      port: targetUrl.port || (targetUrl.protocol === 'https:' ? 443 : 80),
      path: path,
      method: req.method,
      headers: { ...req.headers },
    };
    
    // Change origin if requested
    if (options.changeOrigin) {
      proxyOptions.headers!['host'] = targetUrl.host;
    }
    
    // Add X-Forwarded headers
    const clientIp = req.socket.remoteAddress || '';
    proxyOptions.headers!['x-forwarded-for'] = 
      (req.headers['x-forwarded-for'] as string || '') + 
      (req.headers['x-forwarded-for'] ? ', ' : '') + 
      clientIp;
    proxyOptions.headers!['x-forwarded-proto'] = 'http';
    proxyOptions.headers!['x-forwarded-host'] = req.headers.host || '';
    
    const proxyReq = protocol.request(proxyOptions, (proxyRes) => {
      // Call hook
      if (options.onProxyRes) {
        options.onProxyRes(proxyRes, req, res);
      }
      
      // Copy status and headers
      res.writeHead(proxyRes.statusCode || 500, proxyRes.headers);
      
      // Pipe response
      proxyRes.pipe(res, { end: true });
    });
    
    // Call hook
    if (options.onProxyReq) {
      options.onProxyReq(proxyReq, req);
    }
    
    // Handle errors
    proxyReq.on('error', (err) => {
      console.error('Proxy error:', err);
      res.writeHead(502);
      res.end('Bad Gateway');
    });
    
    // Pipe request body
    req.pipe(proxyReq, { end: true });
  });
}

// Usage
const proxy = createProxyServer({
  target: 'http://backend:3000',
  changeOrigin: true,
  pathRewrite: {
    '^/api': '', // Remove /api prefix
  },
  onProxyReq: (proxyReq, req) => {
    console.log(`Proxying: ${req.method} ${req.url}`);
  },
  onProxyRes: (proxyRes, req, res) => {
    // Add custom header
    proxyRes.headers['x-proxy-by'] = 'my-proxy';
  },
});

proxy.listen(8080, () => {
  console.log('Proxy server running on port 8080');
});
```

### Express Proxy Middleware

```typescript
// proxy/express-proxy.ts
import express, { Request, Response, NextFunction } from 'express';
import { createProxyMiddleware, Options } from 'http-proxy-middleware';

const app = express();

// Basic proxy
app.use('/api', createProxyMiddleware({
  target: 'http://backend:3000',
  changeOrigin: true,
  pathRewrite: {
    '^/api': '', // Remove /api prefix
  },
}));

// Multiple backends with routing
const apiProxy = createProxyMiddleware({
  target: 'http://api-service:3000',
  changeOrigin: true,
});

const authProxy = createProxyMiddleware({
  target: 'http://auth-service:3001',
  changeOrigin: true,
});

const mediaProxy = createProxyMiddleware({
  target: 'http://media-service:3002',
  changeOrigin: true,
  onProxyReq: (proxyReq, req, res) => {
    // Increase timeout for uploads
    proxyReq.setTimeout(300000); // 5 minutes
  },
});

app.use('/api/auth', authProxy);
app.use('/api/media', mediaProxy);
app.use('/api', apiProxy);

// WebSocket proxy
const wsProxy = createProxyMiddleware({
  target: 'http://ws-service:3003',
  changeOrigin: true,
  ws: true, // Enable WebSocket proxying
});

app.use('/ws', wsProxy);

// Custom routing logic
app.use('/api/v2', (req: Request, res: Response, next: NextFunction) => {
  // Route based on header
  const region = req.headers['x-region'] as string;
  
  const targets: Record<string, string> = {
    'us': 'http://us-api:3000',
    'eu': 'http://eu-api:3000',
    'asia': 'http://asia-api:3000',
  };
  
  const target = targets[region] || targets['us'];
  
  createProxyMiddleware({
    target,
    changeOrigin: true,
  })(req, res, next);
});

app.listen(8080);
```

### Load Balancing Proxy

```typescript
// proxy/load-balancer.ts
import http from 'http';
import https from 'https';

interface Backend {
  host: string;
  port: number;
  weight: number;
  healthy: boolean;
  activeConnections: number;
}

type LoadBalancingStrategy = 'round-robin' | 'least-connections' | 'weighted' | 'ip-hash';

class LoadBalancer {
  private backends: Backend[];
  private currentIndex = 0;
  private strategy: LoadBalancingStrategy;
  
  constructor(
    backends: Array<{ host: string; port: number; weight?: number }>,
    strategy: LoadBalancingStrategy = 'round-robin'
  ) {
    this.backends = backends.map(b => ({
      ...b,
      weight: b.weight || 1,
      healthy: true,
      activeConnections: 0,
    }));
    this.strategy = strategy;
    
    // Start health checks
    this.startHealthChecks();
  }
  
  getBackend(clientIp?: string): Backend | null {
    const healthy = this.backends.filter(b => b.healthy);
    if (healthy.length === 0) return null;
    
    switch (this.strategy) {
      case 'round-robin':
        return this.roundRobin(healthy);
      case 'least-connections':
        return this.leastConnections(healthy);
      case 'weighted':
        return this.weighted(healthy);
      case 'ip-hash':
        return this.ipHash(healthy, clientIp || '');
      default:
        return healthy[0];
    }
  }
  
  private roundRobin(backends: Backend[]): Backend {
    const backend = backends[this.currentIndex % backends.length];
    this.currentIndex++;
    return backend;
  }
  
  private leastConnections(backends: Backend[]): Backend {
    return backends.reduce((min, b) => 
      b.activeConnections < min.activeConnections ? b : min
    );
  }
  
  private weighted(backends: Backend[]): Backend {
    const totalWeight = backends.reduce((sum, b) => sum + b.weight, 0);
    let random = Math.random() * totalWeight;
    
    for (const backend of backends) {
      random -= backend.weight;
      if (random <= 0) return backend;
    }
    
    return backends[0];
  }
  
  private ipHash(backends: Backend[], clientIp: string): Backend {
    // Simple hash based on IP
    let hash = 0;
    for (let i = 0; i < clientIp.length; i++) {
      hash = ((hash << 5) - hash) + clientIp.charCodeAt(i);
      hash = hash & hash; // Convert to 32bit integer
    }
    return backends[Math.abs(hash) % backends.length];
  }
  
  private async startHealthChecks(): Promise<void> {
    setInterval(async () => {
      for (const backend of this.backends) {
        backend.healthy = await this.checkHealth(backend);
      }
    }, 10000); // Check every 10 seconds
  }
  
  private async checkHealth(backend: Backend): Promise<boolean> {
    return new Promise((resolve) => {
      const req = http.request({
        hostname: backend.host,
        port: backend.port,
        path: '/health',
        method: 'GET',
        timeout: 5000,
      }, (res) => {
        resolve(res.statusCode === 200);
      });
      
      req.on('error', () => resolve(false));
      req.on('timeout', () => {
        req.destroy();
        resolve(false);
      });
      
      req.end();
    });
  }
  
  incrementConnections(backend: Backend): void {
    backend.activeConnections++;
  }
  
  decrementConnections(backend: Backend): void {
    backend.activeConnections = Math.max(0, backend.activeConnections - 1);
  }
}

// Create load balancing proxy server
function createLoadBalancingProxy(
  balancer: LoadBalancer,
  port: number
): http.Server {
  return http.createServer((req, res) => {
    const clientIp = req.socket.remoteAddress || '';
    const backend = balancer.getBackend(clientIp);
    
    if (!backend) {
      res.writeHead(503);
      res.end('Service Unavailable');
      return;
    }
    
    balancer.incrementConnections(backend);
    
    const proxyReq = http.request({
      hostname: backend.host,
      port: backend.port,
      path: req.url,
      method: req.method,
      headers: {
        ...req.headers,
        'X-Forwarded-For': clientIp,
        'X-Real-IP': clientIp,
      },
    }, (proxyRes) => {
      res.writeHead(proxyRes.statusCode || 500, proxyRes.headers);
      proxyRes.pipe(res);
      
      proxyRes.on('end', () => {
        balancer.decrementConnections(backend);
      });
    });
    
    proxyReq.on('error', () => {
      balancer.decrementConnections(backend);
      res.writeHead(502);
      res.end('Bad Gateway');
    });
    
    req.pipe(proxyReq);
  }).listen(port);
}

// Usage
const balancer = new LoadBalancer([
  { host: 'backend1', port: 3000, weight: 5 },
  { host: 'backend2', port: 3000, weight: 3 },
  { host: 'backend3', port: 3000, weight: 2 },
], 'weighted');

createLoadBalancingProxy(balancer, 8080);
```

---

## Real-World Scenarios

### Scenario 1: API Gateway with Rate Limiting

```nginx
# nginx/api-gateway.conf

# Rate limiting zones
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;
limit_req_zone $http_authorization zone=auth_limit:10m rate=100r/s;

# Connection limiting
limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

upstream api_backend {
    server backend:3000;
    keepalive 32;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;
    
    # SSL config...
    
    # Global rate limit
    limit_req zone=api_limit burst=20 nodelay;
    limit_conn conn_limit 10;
    
    # API versioning
    location /v1 {
        proxy_pass http://api_backend;
        proxy_set_header X-API-Version "1";
    }
    
    location /v2 {
        proxy_pass http://api_backend;
        proxy_set_header X-API-Version "2";
    }
    
    # Public endpoints - lower rate limit
    location /api/public {
        limit_req zone=api_limit burst=5;
        proxy_pass http://api_backend;
    }
    
    # Authenticated endpoints - higher rate limit
    location /api/private {
        limit_req zone=auth_limit burst=50;
        
        # Verify auth header exists
        if ($http_authorization = "") {
            return 401;
        }
        
        proxy_pass http://api_backend;
    }
    
    # Health check endpoint - no rate limit
    location /health {
        limit_req off;
        proxy_pass http://api_backend;
    }
}
```

### Scenario 2: Blue-Green Deployment with Proxy

```nginx
# nginx/blue-green.conf

# Blue environment (current production)
upstream blue {
    server blue1:3000;
    server blue2:3000;
}

# Green environment (new version)
upstream green {
    server green1:3000;
    server green2:3000;
}

# Map to control traffic routing
map $cookie_deployment $backend {
    default blue;
    "green" green;
}

# Or percentage-based canary
split_clients "${remote_addr}${uri}" $backend_canary {
    10%  green;  # 10% to green
    *    blue;   # 90% to blue
}

server {
    listen 443 ssl http2;
    server_name app.example.com;
    
    location / {
        # Use cookie-based routing
        proxy_pass http://$backend;
        
        # Or use canary routing
        # proxy_pass http://$backend_canary;
        
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # Admin endpoint to switch deployment
    location /admin/deploy {
        # Secure this in production!
        allow 10.0.0.0/8;
        deny all;
        
        # Set cookie to route to green
        add_header Set-Cookie "deployment=green; Path=/; Max-Age=3600";
        return 200 "Switched to green deployment";
    }
    
    location /admin/rollback {
        allow 10.0.0.0/8;
        deny all;
        
        add_header Set-Cookie "deployment=; Path=/; Max-Age=0";
        return 200 "Rolled back to blue deployment";
    }
}
```

---

## Common Pitfalls

### 1. Not Forwarding Client IP

```nginx
# ❌ BAD: Backend sees proxy IP
location / {
    proxy_pass http://backend;
}

# ✅ GOOD: Forward real client IP
location / {
    proxy_pass http://backend;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

### 2. WebSocket Proxying Without Upgrade Headers

```nginx
# ❌ BAD: WebSocket won't work
location /ws {
    proxy_pass http://backend;
}

# ✅ GOOD: Enable WebSocket upgrade
location /ws {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### 3. Timeout Too Short for Uploads

```nginx
# ❌ BAD: Upload fails on large files
location /upload {
    proxy_pass http://backend;
    # Default 60s timeout
}

# ✅ GOOD: Extended timeout for uploads
location /upload {
    proxy_pass http://backend;
    proxy_read_timeout 300s;
    proxy_send_timeout 300s;
    client_max_body_size 100M;
}
```

---

## Interview Questions

### Q1: What's the difference between forward and reverse proxy?

**A:** **Forward proxy** sits between clients and the internet - the client knows about it and configures it. Used for: caching, filtering, anonymity. **Reverse proxy** sits in front of servers - clients don't know it exists, they think they're talking to the actual server. Used for: load balancing, SSL termination, caching, security.

### Q2: Why would you use Nginx as a reverse proxy instead of serving directly from Node.js?

**A:** 1) SSL termination - Nginx handles HTTPS efficiently. 2) Static file serving - much faster than Node.js. 3) Load balancing across multiple Node processes. 4) Connection pooling and keep-alive. 5) Request buffering protects backend from slow clients. 6) Security features (rate limiting, request filtering). 7) Caching reduces backend load.

### Q3: How does a proxy handle WebSocket connections?

**A:** WebSocket requires HTTP upgrade mechanism. Proxy must: 1) Pass Upgrade and Connection headers. 2) Use HTTP/1.1 (HTTP/2 handles it differently). 3) Keep connection open (no buffering). 4) Not modify the bidirectional stream. In Nginx: set `proxy_http_version 1.1` and pass upgrade headers.

### Q4: What is X-Forwarded-For and why is it important?

**A:** X-Forwarded-For header contains the original client IP when requests pass through proxies. Without it, the backend only sees the proxy's IP. The header is a comma-separated list of IPs (client, proxy1, proxy2). Important for: logging, analytics, rate limiting, geolocation. Be careful: it can be spoofed, so only trust it from known proxies.

---

## Quick Reference Checklist

### Basic Setup
- [ ] Forward client IP (X-Real-IP, X-Forwarded-For)
- [ ] Forward protocol (X-Forwarded-Proto)
- [ ] Set appropriate timeouts
- [ ] Configure buffer sizes

### Load Balancing
- [ ] Define upstream servers
- [ ] Choose balancing algorithm
- [ ] Configure health checks
- [ ] Set up failover/backup servers

### Security
- [ ] SSL termination configured
- [ ] Rate limiting enabled
- [ ] Request size limits set
- [ ] Hide upstream errors

### Performance
- [ ] Enable keep-alive to backends
- [ ] Configure caching where appropriate
- [ ] Use gzip compression
- [ ] Tune worker processes and connections

---

*Last updated: February 2026*

