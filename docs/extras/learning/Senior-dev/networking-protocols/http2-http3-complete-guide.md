# HTTP/2 & HTTP/3 - Complete Guide

> **MUST REMEMBER**: HTTP/2 uses multiplexing (multiple requests over one TCP connection), header compression (HPACK), and server push. HTTP/3 replaces TCP with QUIC (UDP-based) eliminating head-of-line blocking and enabling 0-RTT connections. HTTP/2 requires HTTPS in browsers. Use HTTP/2 for most apps today; HTTP/3 for latency-critical applications.

---

## How to Explain Like a Senior Developer

"HTTP/1.1 opens a new TCP connection for each request or uses keep-alive with head-of-line blocking - one slow response blocks everything behind it. HTTP/2 fixes this with multiplexing: many requests share one connection, interleaved as streams. It also compresses headers (HPACK) since headers repeat constantly. Server push lets you send resources before they're requested. HTTP/3 goes further - it replaces TCP with QUIC, a UDP-based protocol. This eliminates TCP's head-of-line blocking entirely because each stream is independent. QUIC also enables 0-RTT connection resumption. The practical impact: faster page loads, especially on high-latency networks like mobile."

---

## Core Implementation

### HTTP/2 Server with Node.js

```typescript
// http2/server.ts
import * as http2 from 'http2';
import * as fs from 'fs';
import * as path from 'path';

// HTTP/2 requires TLS in browsers
const server = http2.createSecureServer({
  key: fs.readFileSync('server.key'),
  cert: fs.readFileSync('server.crt'),
  allowHTTP1: true, // Fallback for HTTP/1.1 clients
});

server.on('error', (err) => console.error(err));

server.on('stream', (stream, headers) => {
  const requestPath = headers[':path'] || '/';
  const method = headers[':method'];
  
  console.log(`${method} ${requestPath}`);
  
  // Handle different routes
  if (requestPath === '/') {
    // Send HTML response
    stream.respond({
      'content-type': 'text/html; charset=utf-8',
      ':status': 200,
    });
    
    // Server Push - push CSS before browser requests it
    if (stream.pushAllowed) {
      stream.pushStream({ ':path': '/styles.css' }, (err, pushStream) => {
        if (err) {
          console.error('Push error:', err);
          return;
        }
        
        pushStream.respond({
          'content-type': 'text/css',
          ':status': 200,
        });
        pushStream.end('body { font-family: sans-serif; }');
      });
    }
    
    stream.end(`
      <!DOCTYPE html>
      <html>
        <head>
          <link rel="stylesheet" href="/styles.css">
        </head>
        <body>
          <h1>HTTP/2 Server</h1>
        </body>
      </html>
    `);
  } else if (requestPath === '/api/data') {
    // JSON API response
    stream.respond({
      'content-type': 'application/json',
      ':status': 200,
    });
    stream.end(JSON.stringify({ message: 'Hello from HTTP/2!' }));
  } else {
    // 404 for other paths
    stream.respond({ ':status': 404 });
    stream.end('Not Found');
  }
});

server.listen(8443, () => {
  console.log('HTTP/2 server running on https://localhost:8443');
});
```

### HTTP/2 Client

```typescript
// http2/client.ts
import * as http2 from 'http2';

async function makeHttp2Request(
  url: string,
  options: {
    method?: string;
    headers?: Record<string, string>;
    body?: string;
  } = {}
): Promise<{ status: number; headers: Record<string, string>; body: string }> {
  const urlObj = new URL(url);
  
  return new Promise((resolve, reject) => {
    const client = http2.connect(`${urlObj.protocol}//${urlObj.host}`, {
      rejectUnauthorized: false, // For self-signed certs in dev
    });
    
    client.on('error', reject);
    
    const req = client.request({
      ':method': options.method || 'GET',
      ':path': urlObj.pathname + urlObj.search,
      ...options.headers,
    });
    
    if (options.body) {
      req.write(options.body);
    }
    
    let responseHeaders: Record<string, string> = {};
    let body = '';
    
    req.on('response', (headers) => {
      responseHeaders = headers as Record<string, string>;
    });
    
    req.on('data', (chunk) => {
      body += chunk;
    });
    
    req.on('end', () => {
      client.close();
      resolve({
        status: parseInt(responseHeaders[':status'] || '0'),
        headers: responseHeaders,
        body,
      });
    });
    
    req.on('error', reject);
    req.end();
  });
}

// Multiplexed requests - send multiple requests on same connection
async function multiplexedRequests(): Promise<void> {
  const client = http2.connect('https://localhost:8443', {
    rejectUnauthorized: false,
  });
  
  // Send multiple requests simultaneously
  const requests = ['/api/users', '/api/products', '/api/orders'].map(
    (path) =>
      new Promise<string>((resolve, reject) => {
        const req = client.request({ ':path': path });
        let data = '';
        
        req.on('data', (chunk) => (data += chunk));
        req.on('end', () => resolve(data));
        req.on('error', reject);
        req.end();
      })
  );
  
  // All requests sent on same connection!
  const results = await Promise.all(requests);
  console.log('Results:', results);
  
  client.close();
}
```

### Express with HTTP/2

```typescript
// http2/express-http2.ts
import express from 'express';
import * as http2 from 'http2';
import * as fs from 'fs';

const app = express();

app.get('/', (req, res) => {
  res.send('Hello from Express on HTTP/2!');
});

app.get('/api/data', (req, res) => {
  res.json({ message: 'API response' });
});

// Create HTTP/2 server with Express
const server = http2.createSecureServer(
  {
    key: fs.readFileSync('server.key'),
    cert: fs.readFileSync('server.crt'),
    allowHTTP1: true,
  },
  // Express handles both HTTP/1.1 and HTTP/2
  app as any
);

server.listen(8443, () => {
  console.log('Express HTTP/2 server on https://localhost:8443');
});
```

### Nginx HTTP/2 Configuration

```nginx
# nginx/http2.conf
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com;
    
    # SSL configuration
    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # HTTP/2 specific settings
    http2_push_preload on;  # Enable push based on Link headers
    http2_max_concurrent_streams 128;
    http2_max_field_size 16k;
    http2_max_header_size 32k;
    
    location / {
        root /var/www/html;
        
        # Server push via Link header
        http2_push /styles.css;
        http2_push /app.js;
    }
    
    location /api {
        proxy_pass http://backend:3000;
        proxy_http_version 1.1;  # Backend might not support HTTP/2
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# HTTP/3 (QUIC) - requires nginx compiled with quic
server {
    listen 443 quic reuseport;
    listen 443 ssl http2;
    
    server_name example.com;
    
    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
    
    # Advertise HTTP/3 availability
    add_header Alt-Svc 'h3=":443"; ma=86400';
    
    location / {
        root /var/www/html;
    }
}
```

### HTTP/3 with Fastify

```typescript
// http3/fastify-http3.ts
// Note: HTTP/3 support is still experimental in Node.js
// Using @fastify/http2 for HTTP/2, HTTP/3 requires native QUIC

import Fastify from 'fastify';
import * as fs from 'fs';

const fastify = Fastify({
  http2: true,
  https: {
    key: fs.readFileSync('server.key'),
    cert: fs.readFileSync('server.crt'),
  },
  logger: true,
});

fastify.get('/', async (request, reply) => {
  // Check HTTP version
  const httpVersion = request.raw.httpVersion;
  
  return {
    message: 'Hello!',
    httpVersion: httpVersion,
    protocol: httpVersion === '2.0' ? 'HTTP/2' : 'HTTP/1.1',
  };
});

// Stream response for large data
fastify.get('/stream', async (request, reply) => {
  reply.header('Content-Type', 'application/json');
  
  // Stream chunks
  const stream = reply.raw;
  stream.write('{"items":[');
  
  for (let i = 0; i < 1000; i++) {
    stream.write(JSON.stringify({ id: i, name: `Item ${i}` }));
    if (i < 999) stream.write(',');
    
    // Add small delay to demonstrate streaming
    await new Promise(resolve => setTimeout(resolve, 1));
  }
  
  stream.end(']}');
});

fastify.listen({ port: 8443 }, (err) => {
  if (err) throw err;
  console.log('Fastify HTTP/2 server running');
});
```

---

## Real-World Scenarios

### Scenario 1: Comparing HTTP Versions Performance

```typescript
// performance/http-comparison.ts

interface PerformanceResult {
  httpVersion: string;
  totalTime: number;
  requests: number;
  avgLatency: number;
  throughput: number;
}

// Measure HTTP/1.1 vs HTTP/2 for multiple requests
async function compareHttpVersions(
  http1Url: string,
  http2Url: string,
  requestCount: number
): Promise<{ http1: PerformanceResult; http2: PerformanceResult }> {
  // HTTP/1.1 - sequential or limited parallel (browser limits to ~6)
  const http1Start = Date.now();
  const http1Results = await Promise.all(
    Array(requestCount).fill(null).map(() => 
      fetch(http1Url).then(r => r.json())
    )
  );
  const http1Time = Date.now() - http1Start;
  
  // HTTP/2 - true multiplexing
  const http2Start = Date.now();
  const http2Results = await Promise.all(
    Array(requestCount).fill(null).map(() =>
      fetch(http2Url).then(r => r.json())
    )
  );
  const http2Time = Date.now() - http2Start;
  
  return {
    http1: {
      httpVersion: '1.1',
      totalTime: http1Time,
      requests: requestCount,
      avgLatency: http1Time / requestCount,
      throughput: requestCount / (http1Time / 1000),
    },
    http2: {
      httpVersion: '2',
      totalTime: http2Time,
      requests: requestCount,
      avgLatency: http2Time / requestCount,
      throughput: requestCount / (http2Time / 1000),
    },
  };
}

// Expected result: HTTP/2 significantly faster for many parallel requests
// HTTP/1.1: 100 requests at 50ms each with 6 parallel = ~850ms
// HTTP/2: 100 requests at 50ms each, all parallel = ~50-100ms
```

### Scenario 2: Server Push Strategy

```typescript
// http2/push-strategy.ts
import * as http2 from 'http2';
import * as fs from 'fs';
import * as path from 'path';

interface PushManifest {
  [page: string]: string[];  // Page path -> resources to push
}

// Define which resources to push for each page
const pushManifest: PushManifest = {
  '/': ['/styles/main.css', '/js/app.js', '/images/logo.png'],
  '/dashboard': ['/styles/dashboard.css', '/js/dashboard.js', '/api/user'],
  '/products': ['/styles/products.css', '/js/products.js', '/api/products'],
};

function handleStream(
  stream: http2.ServerHttp2Stream,
  headers: http2.IncomingHttpHeaders
): void {
  const requestPath = headers[':path'] as string;
  
  // Check if we should push resources for this path
  const resourcesToPush = pushManifest[requestPath];
  
  if (resourcesToPush && stream.pushAllowed) {
    // Push all associated resources
    for (const resource of resourcesToPush) {
      pushResource(stream, resource);
    }
  }
  
  // Send main response
  serveFile(stream, requestPath);
}

function pushResource(
  stream: http2.ServerHttp2Stream,
  resourcePath: string
): void {
  stream.pushStream({ ':path': resourcePath }, (err, pushStream) => {
    if (err) {
      console.error('Push error:', err);
      return;
    }
    
    const contentType = getContentType(resourcePath);
    
    // Check if it's an API call
    if (resourcePath.startsWith('/api/')) {
      // Push API data
      pushStream.respond({
        ':status': 200,
        'content-type': 'application/json',
        'cache-control': 'no-cache',
      });
      
      // In real app, fetch data from database
      pushStream.end(JSON.stringify({ pushed: true, path: resourcePath }));
    } else {
      // Push static file
      const filePath = path.join('./public', resourcePath);
      
      if (fs.existsSync(filePath)) {
        pushStream.respond({
          ':status': 200,
          'content-type': contentType,
          'cache-control': 'max-age=86400',
        });
        
        fs.createReadStream(filePath).pipe(pushStream);
      } else {
        pushStream.respond({ ':status': 404 });
        pushStream.end();
      }
    }
  });
}

function serveFile(
  stream: http2.ServerHttp2Stream,
  requestPath: string
): void {
  const filePath = requestPath === '/' 
    ? './public/index.html'
    : path.join('./public', requestPath);
  
  if (fs.existsSync(filePath)) {
    stream.respondWithFile(filePath, {
      'content-type': getContentType(requestPath),
    });
  } else {
    stream.respond({ ':status': 404 });
    stream.end('Not Found');
  }
}

function getContentType(filePath: string): string {
  const ext = path.extname(filePath);
  const types: Record<string, string> = {
    '.html': 'text/html',
    '.css': 'text/css',
    '.js': 'application/javascript',
    '.json': 'application/json',
    '.png': 'image/png',
    '.jpg': 'image/jpeg',
  };
  return types[ext] || 'application/octet-stream';
}
```

---

## Common Pitfalls

### 1. Not Using HTTPS with HTTP/2

```typescript
// ❌ BAD: HTTP/2 without TLS (browsers won't support it)
import * as http2 from 'http2';
const server = http2.createServer(); // h2c - cleartext HTTP/2

// ✅ GOOD: HTTP/2 with TLS (required for browsers)
const server = http2.createSecureServer({
  key: fs.readFileSync('server.key'),
  cert: fs.readFileSync('server.crt'),
});
```

### 2. Over-using Server Push

```typescript
// ❌ BAD: Pushing everything
const pushAll = [
  '/style1.css', '/style2.css', '/style3.css',
  '/script1.js', '/script2.js', '/script3.js',
  '/image1.png', '/image2.png', // ... 50 more files
];
// Wastes bandwidth if client has cached resources

// ✅ GOOD: Push critical resources only, use cache digest
const pushCritical = ['/critical.css', '/app.js'];
// Or use 103 Early Hints instead
```

### 3. Ignoring HTTP/1.1 Fallback

```typescript
// ❌ BAD: No fallback for older clients
const server = http2.createSecureServer(options);

// ✅ GOOD: Allow HTTP/1.1 fallback
const server = http2.createSecureServer({
  ...options,
  allowHTTP1: true, // Fallback for HTTP/1.1 clients
});
```

---

## Interview Questions

### Q1: What are the main differences between HTTP/1.1, HTTP/2, and HTTP/3?

**A:** 
- **HTTP/1.1**: One request per connection (or limited pipelining), text-based headers, no multiplexing
- **HTTP/2**: Binary framing, multiplexing (many requests on one TCP connection), header compression (HPACK), server push, requires TLS in browsers
- **HTTP/3**: Uses QUIC (UDP-based) instead of TCP, eliminates head-of-line blocking at transport layer, 0-RTT connection resumption, built-in encryption

### Q2: What is head-of-line blocking and how do HTTP/2 and HTTP/3 address it?

**A:** Head-of-line blocking occurs when one slow/blocked request prevents others from completing. HTTP/1.1 has this at the application layer. HTTP/2 fixes it at application layer via multiplexing but still has it at TCP layer (one lost packet blocks all streams). HTTP/3/QUIC eliminates it entirely since each stream is independent at the transport layer.

### Q3: When would you NOT use server push?

**A:** Don't push when: 1) Resources are likely cached by the client, 2) You're pushing too many resources (bandwidth waste), 3) Resources aren't critical for initial render, 4) Client is on slow/metered connection. Consider 103 Early Hints as alternative - gives hints without committing to push.

### Q4: Why does HTTP/2 require TLS in browsers?

**A:** Browsers (not the spec) require TLS for HTTP/2 to: 1) Avoid middleboxes breaking the binary protocol, 2) Ensure upgrade negotiation via ALPN works reliably, 3) Improve security posture of the web. HTTP/2 cleartext (h2c) exists but isn't supported by browsers.

---

## Quick Reference Checklist

### HTTP/2 Setup
- [ ] Enable TLS/HTTPS (required for browsers)
- [ ] Configure allowHTTP1 fallback
- [ ] Set appropriate stream limits
- [ ] Configure header size limits

### Server Push
- [ ] Only push critical resources
- [ ] Check pushAllowed before pushing
- [ ] Set appropriate cache headers
- [ ] Consider 103 Early Hints alternative

### Performance
- [ ] Reduce domain sharding (not needed with HTTP/2)
- [ ] Stop concatenating files (multiplexing handles it)
- [ ] Remove image sprites (individual files OK now)
- [ ] Monitor HTTP/2 adoption in analytics

### HTTP/3 Readiness
- [ ] Use CDN with HTTP/3 support
- [ ] Add Alt-Svc header for HTTP/3
- [ ] Test on QUIC-enabled browsers
- [ ] Monitor connection migration on mobile

---

*Last updated: February 2026*

