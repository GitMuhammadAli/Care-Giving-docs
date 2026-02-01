# 🚪 API Gateway Patterns Complete Guide

> A comprehensive guide to API Gateway patterns - routing, authentication, rate limiting, aggregation, and building resilient API infrastructures.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "An API Gateway is a single entry point for all clients that handles cross-cutting concerns like authentication, rate limiting, and routing - decoupling clients from the complexity of multiple backend services."

### The 7 Core Responsibilities (Memorize!)
```
1. ROUTING           → Direct requests to appropriate backend services
2. AUTHENTICATION    → Verify identity (JWT, API keys, OAuth)
3. AUTHORIZATION     → Check permissions (RBAC, ABAC, scopes)
4. RATE LIMITING     → Prevent abuse (per user, per IP, per endpoint)
5. AGGREGATION       → Combine multiple service calls into one response
6. TRANSFORMATION    → Convert protocols/formats (REST↔gRPC, XML↔JSON)
7. OBSERVABILITY     → Logging, metrics, tracing for all requests
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Edge layer"** | "Auth happens at the edge layer, services don't need to re-validate" |
| **"BFF pattern"** | "We use BFF - one gateway per client type (web, mobile, IoT)" |
| **"Token introspection"** | "Gateway does token introspection against the auth server" |
| **"Fan-out"** | "Gateway does fan-out to 5 services and aggregates responses" |
| **"Circuit breaker"** | "Gateway has circuit breaker to prevent cascade failures" |
| **"Backpressure"** | "Rate limiting provides backpressure to protect backends" |
| **"Request coalescing"** | "We coalesce duplicate requests to reduce backend load" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Rate limit response | **429** | Too Many Requests |
| Token validation | **<5ms** | Don't add latency (use local validation) |
| Circuit breaker threshold | **50%** | Trip at 50% failure rate |
| Gateway timeout | **30s** | Don't hold connections forever |
| Retry attempts | **3** | Max retries before failing |

### Gateway vs Service Mesh
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  API GATEWAY (North-South Traffic)                              │
│  └── Client → Gateway → Services                               │
│  └── Handles external requests                                 │
│  └── Single entry point                                        │
│  └── Auth, rate limiting, routing                              │
│                                                                  │
│  SERVICE MESH (East-West Traffic)                              │
│  └── Service ↔ Service                                         │
│  └── Handles internal communication                            │
│  └── Sidecar proxies (Envoy)                                  │
│  └── mTLS, retries, observability                             │
│                                                                  │
│  Together: Gateway for clients, Mesh for services              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement (Memorize This!)
> "Without an API gateway, every microservice needs to implement auth, rate limiting, and logging - that's duplicated code and inconsistent security. Our gateway centralizes these concerns: JWT validation happens once at the edge, rate limits protect all backends uniformly, and we get a single place for observability. For mobile clients, we use the BFF pattern - one gateway optimized for mobile that aggregates 5 backend calls into one. The circuit breaker prevents cascade failures when a service is down. We've reduced average latency by 40% through response caching at the gateway level."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                    API GATEWAY ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   CLIENTS                                                       │
│   ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐                       │
│   │ Web  │  │Mobile│  │ IoT  │  │ 3rd  │                       │
│   │ App  │  │ App  │  │Device│  │Party │                       │
│   └──┬───┘  └──┬───┘  └──┬───┘  └──┬───┘                       │
│      │         │         │         │                            │
│      └─────────┴────┬────┴─────────┘                            │
│                     ▼                                            │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │                    API GATEWAY                           │  │
│   │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │  │
│   │  │  Auth   │ │  Rate   │ │ Routing │ │ Circuit │       │  │
│   │  │Validate │ │ Limit   │ │         │ │ Breaker │       │  │
│   │  └─────────┘ └─────────┘ └─────────┘ └─────────┘       │  │
│   │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │  │
│   │  │ Logging │ │ Caching │ │Transform│ │Aggregate│       │  │
│   │  └─────────┘ └─────────┘ └─────────┘ └─────────┘       │  │
│   └─────────────────────────────────────────────────────────┘  │
│                     │                                            │
│      ┌──────────────┼──────────────┐                            │
│      ▼              ▼              ▼                            │
│   ┌──────┐      ┌──────┐      ┌──────┐                         │
│   │User  │      │Order │      │Payment│                         │
│   │Service│     │Service│     │Service│                         │
│   └──────┘      └──────┘      └──────┘                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is an API Gateway?"**
> "Single entry point for all clients. Handles cross-cutting concerns: auth, rate limiting, routing, logging. Decouples clients from backend complexity."

**Q: "Gateway vs Load Balancer?"**
> "Load balancer distributes traffic (L4/L7), gateway handles application logic (auth, rate limits, transformation). Often used together: LB in front of gateway cluster."

**Q: "What is the BFF pattern?"**
> "Backend for Frontend - one gateway per client type. Mobile BFF aggregates calls, Web BFF serves different data. Each optimized for its client's needs."

**Q: "How do you handle auth at the gateway?"**
> "Validate JWT locally (no auth server call per request), attach user context to downstream requests, terminate SSL, and pass identity headers to services."

**Q: "What about rate limiting?"**
> "Multiple levels: per user, per IP, per endpoint. Use token bucket or sliding window algorithms. Return 429 with Retry-After header. Store counters in Redis."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "Why use an API Gateway?"

**Junior Answer:**
> "It routes requests to different services."

**Senior Answer:**
> "An API Gateway solves several problems in distributed systems:

**1. Cross-Cutting Concerns**
- Auth/authz once at the edge, not in every service
- Consistent rate limiting across all APIs
- Centralized logging and metrics

**2. Client Simplification**
- Single endpoint instead of multiple service URLs
- Protocol translation (gRPC internally, REST externally)
- Response aggregation (one call instead of many)

**3. Security**
- Single point for SSL termination
- Attack surface reduction (services not directly exposed)
- Request validation before hitting backends

**4. Operational Benefits**
- A/B testing and canary deployments
- Request/response transformation
- Caching at the edge

Trade-offs:
- Single point of failure (mitigate with clustering)
- Additional latency (minimize with local processing)
- Operational complexity (managed solutions help)"

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "Single point of failure?" | "Deploy multiple gateway instances behind a load balancer. Use health checks, auto-scaling, and multi-AZ deployment." |
| "Doesn't it add latency?" | "Yes, 1-5ms typically. Offset by reducing client round-trips (aggregation), caching, and moving auth out of services." |
| "Build vs buy?" | "For simple needs: Kong, AWS API Gateway, Cloudflare. For custom logic: build with Express/Fastify or use Envoy." |
| "How does it scale?" | "Stateless design, scale horizontally. Externalize state (rate limit counters in Redis). Auto-scale based on request rate." |

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Routing Patterns](#2-routing-patterns)
3. [Authentication & Authorization](#3-authentication--authorization)
4. [Rate Limiting](#4-rate-limiting)
5. [Aggregation & BFF](#5-aggregation--bff)
6. [Resilience Patterns](#6-resilience-patterns)
7. [Implementation](#7-implementation)
8. [When to Use / Not Use](#8-when-to-use--not-use)
9. [Interview Questions](#9-interview-questions)

---

## 1. Core Concepts

### Request Flow Through Gateway

```
REQUEST LIFECYCLE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. CLIENT REQUEST                                              │
│     └── HTTPS request arrives at gateway                       │
│                                                                  │
│  2. SSL TERMINATION                                             │
│     └── Decrypt HTTPS → HTTP internally                        │
│                                                                  │
│  3. RATE LIMITING CHECK                                         │
│     └── Check counters (Redis)                                 │
│     └── If exceeded → 429 Too Many Requests                    │
│                                                                  │
│  4. AUTHENTICATION                                              │
│     └── Validate JWT signature (local, no network call)        │
│     └── Check token expiration                                 │
│     └── If invalid → 401 Unauthorized                          │
│                                                                  │
│  5. AUTHORIZATION                                               │
│     └── Check scopes/permissions                               │
│     └── If forbidden → 403 Forbidden                           │
│                                                                  │
│  6. REQUEST TRANSFORMATION                                      │
│     └── Add headers (X-User-ID, X-Request-ID)                 │
│     └── Transform body if needed                               │
│                                                                  │
│  7. ROUTING                                                     │
│     └── Match route to backend service                         │
│     └── Check circuit breaker state                            │
│                                                                  │
│  8. PROXY TO BACKEND                                            │
│     └── Forward request to service                             │
│     └── Apply timeout                                          │
│                                                                  │
│  9. RESPONSE HANDLING                                           │
│     └── Transform response if needed                           │
│     └── Cache if cacheable                                     │
│     └── Log request/response                                   │
│                                                                  │
│  10. RETURN TO CLIENT                                          │
│      └── Re-encrypt (HTTPS) if needed                          │
│      └── Return response                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Gateway Types

```
GATEWAY PATTERNS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. SINGLE GATEWAY (Simple)                                     │
│     └── One gateway for all clients                            │
│     └── Simple but can become bottleneck                       │
│                                                                  │
│     [Web] [Mobile] [IoT]                                       │
│            │                                                    │
│         [Gateway]                                               │
│            │                                                    │
│     [Service A] [Service B]                                    │
│                                                                  │
│  2. BFF - BACKEND FOR FRONTEND (Per-Client)                    │
│     └── One gateway per client type                            │
│     └── Optimized for each client's needs                      │
│                                                                  │
│     [Web]     [Mobile]     [IoT]                               │
│       │          │           │                                  │
│   [Web BFF] [Mobile BFF] [IoT BFF]                            │
│       │          │           │                                  │
│     [Service A] [Service B] [Service C]                        │
│                                                                  │
│  3. FEDERATED GATEWAY (GraphQL)                                │
│     └── Single schema, federated execution                     │
│     └── Each service owns part of schema                       │
│                                                                  │
│     [Clients]                                                   │
│         │                                                       │
│   [Apollo Gateway]                                              │
│         │                                                       │
│   [Users Subgraph] [Orders Subgraph]                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Popular Gateway Solutions

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    API GATEWAY SOLUTIONS                                  │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  MANAGED (Cloud)                                                         │
│  ├── AWS API Gateway    │ Lambda integration, managed, pay-per-request  │
│  ├── Azure API Mgmt     │ Enterprise features, policy engine           │
│  ├── Google Cloud EP    │ GCP integration, OpenAPI support             │
│  └── Cloudflare Workers │ Edge execution, global, low latency          │
│                                                                           │
│  SELF-HOSTED (Open Source)                                               │
│  ├── Kong               │ Plugin ecosystem, Lua-based, popular         │
│  ├── NGINX              │ High performance, reverse proxy, config-heavy│
│  ├── Envoy              │ Service mesh ready, L7 proxy, gRPC support  │
│  ├── Traefik            │ Docker/K8s native, auto-discovery           │
│  └── Express Gateway    │ Node.js, simple, good for small teams       │
│                                                                           │
│  WHEN TO USE WHAT:                                                       │
│  └── Small team, AWS  → AWS API Gateway                                 │
│  └── Need plugins     → Kong                                            │
│  └── K8s native       → Traefik or Envoy                               │
│  └── Custom logic     → Build with Express/Fastify                     │
│  └── Edge computing   → Cloudflare Workers                             │
│                                                                           │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Routing Patterns

### Basic Routing

```typescript
// ════════════════════════════════════════════════════════════════
// EXPRESS-BASED API GATEWAY: Basic Routing
// ════════════════════════════════════════════════════════════════

import express from 'express';
import { createProxyMiddleware } from 'http-proxy-middleware';

const app = express();

// Service registry (in production, use service discovery)
const services = {
  users: 'http://users-service:3001',
  orders: 'http://orders-service:3002',
  products: 'http://products-service:3003',
  payments: 'http://payments-service:3004',
};

// Route: /api/users/* → users-service
app.use('/api/users', createProxyMiddleware({
  target: services.users,
  changeOrigin: true,
  pathRewrite: { '^/api/users': '' },
}));

// Route: /api/orders/* → orders-service
app.use('/api/orders', createProxyMiddleware({
  target: services.orders,
  changeOrigin: true,
  pathRewrite: { '^/api/orders': '' },
}));

// Route: /api/products/* → products-service
app.use('/api/products', createProxyMiddleware({
  target: services.products,
  changeOrigin: true,
  pathRewrite: { '^/api/products': '' },
}));

app.listen(3000, () => console.log('Gateway running on :3000'));
```

### Advanced Routing Patterns

```typescript
// ════════════════════════════════════════════════════════════════
// HEADER-BASED ROUTING (A/B Testing, Canary)
// ════════════════════════════════════════════════════════════════

const routeByHeader = (req: Request): string => {
  // Route based on feature flag header
  if (req.headers['x-feature-flag'] === 'new-checkout') {
    return services.ordersV2;
  }
  
  // Route based on user group (A/B testing)
  if (req.headers['x-ab-group'] === 'B') {
    return services.ordersExperimental;
  }
  
  return services.orders;
};

app.use('/api/orders', (req, res, next) => {
  const target = routeByHeader(req);
  createProxyMiddleware({ target, changeOrigin: true })(req, res, next);
});

// ════════════════════════════════════════════════════════════════
// WEIGHTED ROUTING (Canary Deployments)
// ════════════════════════════════════════════════════════════════

const weightedRoute = (): string => {
  const random = Math.random() * 100;
  
  // 90% to stable, 10% to canary
  if (random < 90) {
    return services.ordersStable;
  }
  return services.ordersCanary;
};

// ════════════════════════════════════════════════════════════════
// PATH-BASED VERSIONING
// ════════════════════════════════════════════════════════════════

// /api/v1/users → users-service-v1
app.use('/api/v1/users', createProxyMiddleware({
  target: services.usersV1,
  pathRewrite: { '^/api/v1/users': '' },
}));

// /api/v2/users → users-service-v2
app.use('/api/v2/users', createProxyMiddleware({
  target: services.usersV2,
  pathRewrite: { '^/api/v2/users': '' },
}));

// ════════════════════════════════════════════════════════════════
// HEADER-BASED VERSIONING
// ════════════════════════════════════════════════════════════════

app.use('/api/users', (req, res, next) => {
  const version = req.headers['api-version'] || 'v1';
  const target = version === 'v2' ? services.usersV2 : services.usersV1;
  
  createProxyMiddleware({ target, changeOrigin: true })(req, res, next);
});
```

### Request Transformation

```typescript
// ════════════════════════════════════════════════════════════════
// REQUEST TRANSFORMATION: Add headers, modify body
// ════════════════════════════════════════════════════════════════

import { v4 as uuidv4 } from 'uuid';

// Add correlation ID for distributed tracing
const addCorrelationId = (req: Request, res: Response, next: NextFunction) => {
  const correlationId = req.headers['x-correlation-id'] || uuidv4();
  req.headers['x-correlation-id'] = correlationId;
  res.setHeader('x-correlation-id', correlationId);
  next();
};

// Add user context from JWT
const addUserContext = (req: Request, res: Response, next: NextFunction) => {
  if (req.user) {
    req.headers['x-user-id'] = req.user.id;
    req.headers['x-user-email'] = req.user.email;
    req.headers['x-user-roles'] = req.user.roles.join(',');
  }
  next();
};

// Transform request body
const transformRequest = (req: Request, res: Response, next: NextFunction) => {
  if (req.body && req.path.startsWith('/api/orders')) {
    // Add server-side timestamp
    req.body.processedAt = new Date().toISOString();
    // Add client info
    req.body.clientIp = req.ip;
    req.body.userAgent = req.headers['user-agent'];
  }
  next();
};

app.use(addCorrelationId);
app.use(addUserContext);
app.use(express.json());
app.use(transformRequest);

// ════════════════════════════════════════════════════════════════
// RESPONSE TRANSFORMATION
// ════════════════════════════════════════════════════════════════

const transformResponse = (proxyRes: any, req: Request, res: Response) => {
  // Modify response headers
  proxyRes.headers['x-gateway'] = 'my-gateway';
  proxyRes.headers['x-response-time'] = Date.now() - req.startTime;
  
  // Remove internal headers
  delete proxyRes.headers['x-internal-service'];
};

app.use('/api/users', createProxyMiddleware({
  target: services.users,
  onProxyRes: transformResponse,
}));
```

### Service Discovery Integration

```typescript
// ════════════════════════════════════════════════════════════════
// DYNAMIC SERVICE DISCOVERY
// ════════════════════════════════════════════════════════════════

import Consul from 'consul';

const consul = new Consul({ host: 'consul.local', port: 8500 });

// Cache service locations
const serviceCache = new Map<string, string[]>();

// Refresh service locations periodically
async function refreshServices() {
  const services = ['users', 'orders', 'products'];
  
  for (const service of services) {
    const result = await consul.health.service({
      service,
      passing: true,  // Only healthy instances
    });
    
    const addresses = result.map(
      (entry: any) => `http://${entry.Service.Address}:${entry.Service.Port}`
    );
    
    serviceCache.set(service, addresses);
  }
}

// Refresh every 30 seconds
setInterval(refreshServices, 30000);
refreshServices();

// Round-robin load balancing
const roundRobinIndex = new Map<string, number>();

function getServiceUrl(serviceName: string): string {
  const addresses = serviceCache.get(serviceName) || [];
  
  if (addresses.length === 0) {
    throw new Error(`No healthy instances for ${serviceName}`);
  }
  
  const index = (roundRobinIndex.get(serviceName) || 0) % addresses.length;
  roundRobinIndex.set(serviceName, index + 1);
  
  return addresses[index];
}

// Use in routing
app.use('/api/users', (req, res, next) => {
  const target = getServiceUrl('users');
  createProxyMiddleware({ target, changeOrigin: true })(req, res, next);
});
```

---

## 3. Authentication & Authorization

### JWT Validation at Gateway

```typescript
// ════════════════════════════════════════════════════════════════
// JWT AUTHENTICATION: Local validation (no auth server call)
// ════════════════════════════════════════════════════════════════

import jwt from 'jsonwebtoken';
import jwksClient from 'jwks-rsa';

// JWKS client for key rotation support
const jwks = jwksClient({
  jwksUri: 'https://auth.myapp.com/.well-known/jwks.json',
  cache: true,
  cacheMaxAge: 600000,  // 10 minutes
  rateLimit: true,
});

// Get signing key from JWKS
function getKey(header: jwt.JwtHeader, callback: jwt.SigningKeyCallback) {
  jwks.getSigningKey(header.kid, (err, key) => {
    if (err) return callback(err);
    const signingKey = key?.getPublicKey();
    callback(null, signingKey);
  });
}

// Authentication middleware
const authenticate = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const authHeader = req.headers.authorization;
  
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Missing authorization header' });
  }
  
  const token = authHeader.substring(7);
  
  try {
    // Verify JWT locally (no network call to auth server!)
    const decoded = await new Promise<JwtPayload>((resolve, reject) => {
      jwt.verify(
        token,
        getKey,
        {
          algorithms: ['RS256'],
          issuer: 'https://auth.myapp.com',
          audience: 'my-api',
        },
        (err, decoded) => {
          if (err) reject(err);
          else resolve(decoded as JwtPayload);
        }
      );
    });
    
    // Attach user to request
    req.user = {
      id: decoded.sub,
      email: decoded.email,
      roles: decoded.roles || [],
      permissions: decoded.permissions || [],
    };
    
    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired' });
    }
    return res.status(401).json({ error: 'Invalid token' });
  }
};

// ════════════════════════════════════════════════════════════════
// API KEY AUTHENTICATION
// ════════════════════════════════════════════════════════════════

interface ApiKeyRecord {
  key: string;
  clientId: string;
  permissions: string[];
  rateLimit: number;
}

// In production, use Redis or database
const apiKeys = new Map<string, ApiKeyRecord>();

const authenticateApiKey = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const apiKey = req.headers['x-api-key'] as string;
  
  if (!apiKey) {
    return res.status(401).json({ error: 'Missing API key' });
  }
  
  const keyRecord = await getApiKeyFromCache(apiKey);
  
  if (!keyRecord) {
    return res.status(401).json({ error: 'Invalid API key' });
  }
  
  req.client = {
    id: keyRecord.clientId,
    permissions: keyRecord.permissions,
    rateLimit: keyRecord.rateLimit,
  };
  
  next();
};
```

### Authorization Patterns

```typescript
// ════════════════════════════════════════════════════════════════
// ROLE-BASED ACCESS CONTROL (RBAC)
// ════════════════════════════════════════════════════════════════

const requireRole = (...roles: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Not authenticated' });
    }
    
    const hasRole = roles.some(role => req.user.roles.includes(role));
    
    if (!hasRole) {
      return res.status(403).json({ 
        error: 'Forbidden',
        required: roles,
        actual: req.user.roles,
      });
    }
    
    next();
  };
};

// Usage
app.delete('/api/users/:id', 
  authenticate, 
  requireRole('admin'), 
  proxyToUsersService
);

// ════════════════════════════════════════════════════════════════
// PERMISSION-BASED (Fine-grained)
// ════════════════════════════════════════════════════════════════

const requirePermission = (...permissions: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Not authenticated' });
    }
    
    const hasPermission = permissions.every(
      perm => req.user.permissions.includes(perm)
    );
    
    if (!hasPermission) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    
    next();
  };
};

// Usage
app.post('/api/orders', 
  authenticate, 
  requirePermission('orders:create'), 
  proxyToOrdersService
);

// ════════════════════════════════════════════════════════════════
// SCOPE-BASED (OAuth2)
// ════════════════════════════════════════════════════════════════

const requireScope = (...scopes: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const tokenScopes = req.user?.scopes || [];
    
    const hasScopes = scopes.every(scope => tokenScopes.includes(scope));
    
    if (!hasScopes) {
      return res.status(403).json({
        error: 'insufficient_scope',
        required: scopes,
        actual: tokenScopes,
      });
    }
    
    next();
  };
};

// Usage
app.get('/api/users/me', 
  authenticate, 
  requireScope('profile:read'), 
  proxyToUsersService
);

// ════════════════════════════════════════════════════════════════
// RESOURCE-BASED (ABAC - Attribute-Based)
// ════════════════════════════════════════════════════════════════

const authorizeResource = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const resourceId = req.params.id;
  const userId = req.user?.id;
  
  // Fetch resource ownership from service
  const resource = await fetch(
    `${services.resources}/${resourceId}/owner`
  ).then(r => r.json());
  
  // Check ownership or admin role
  const isOwner = resource.ownerId === userId;
  const isAdmin = req.user?.roles.includes('admin');
  
  if (!isOwner && !isAdmin) {
    return res.status(403).json({ error: 'Not authorized for this resource' });
  }
  
  next();
};
```

### Passing Identity Downstream

```typescript
// ════════════════════════════════════════════════════════════════
// IDENTITY PROPAGATION TO SERVICES
// ════════════════════════════════════════════════════════════════

const propagateIdentity = (req: Request, res: Response, next: NextFunction) => {
  if (req.user) {
    // Add user info as headers (services trust gateway)
    req.headers['x-user-id'] = req.user.id;
    req.headers['x-user-email'] = req.user.email;
    req.headers['x-user-roles'] = JSON.stringify(req.user.roles);
    
    // Forward original token if services need to validate
    // (for service-to-service calls)
    req.headers['x-original-token'] = req.headers.authorization;
  }
  
  if (req.client) {
    // For API key auth
    req.headers['x-client-id'] = req.client.id;
    req.headers['x-client-permissions'] = JSON.stringify(req.client.permissions);
  }
  
  // Always add request metadata
  req.headers['x-request-id'] = req.id;
  req.headers['x-gateway-timestamp'] = Date.now().toString();
  
  next();
};

// Service can trust these headers because only gateway can set them
// (services should reject direct requests without going through gateway)
```

### Auth Patterns Summary

```
AUTHENTICATION PATTERNS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  PATTERN           │ USE WHEN                                   │
│  ─────────────────────────────────────────────────────────────  │
│  JWT (Local)       │ User auth, stateless, scalable            │
│  JWT (Introspect)  │ Need real-time revocation                 │
│  API Key           │ Third-party integrations, M2M             │
│  OAuth2            │ Third-party apps, delegated auth          │
│  mTLS              │ Service-to-service (internal)             │
│                                                                  │
│  WHERE TO VALIDATE:                                             │
│  ─────────────────────────────────────────────────────────────  │
│  Gateway           │ All requests (centralized)                │
│  Service           │ Fine-grained, resource-specific           │
│  Both              │ Gateway validates token, service checks   │
│                    │ resource ownership                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Rate Limiting

### Rate Limiting Algorithms

```
RATE LIMITING ALGORITHMS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. TOKEN BUCKET                                                │
│     └── Bucket fills with tokens at fixed rate                 │
│     └── Each request consumes a token                          │
│     └── Allows bursts (up to bucket size)                      │
│     └── Most popular algorithm                                 │
│                                                                  │
│     [████████░░]  Bucket (8/10 tokens)                        │
│     Request → Remove token → [███████░░░]                      │
│     Refill → Add token every 100ms                             │
│                                                                  │
│  2. SLIDING WINDOW                                              │
│     └── Count requests in rolling time window                  │
│     └── More accurate than fixed window                        │
│     └── Uses more memory (stores timestamps)                   │
│                                                                  │
│     Window: [─────────60s─────────]                            │
│             [req][req][req]  Count = 3                         │
│                                                                  │
│  3. FIXED WINDOW                                                │
│     └── Count requests per fixed time period                   │
│     └── Simple but can allow 2x burst at boundary             │
│                                                                  │
│     |────Minute 1────|────Minute 2────|                        │
│     [req][req][req]   [req][req][req]                          │
│                  └─6 requests in 2 seconds─┘                   │
│                                                                  │
│  4. LEAKY BUCKET                                                │
│     └── Requests enter bucket, processed at fixed rate        │
│     └── Smooths out bursts                                     │
│     └── Good for backends that can't handle spikes            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Token Bucket Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// TOKEN BUCKET with Redis
// ════════════════════════════════════════════════════════════════

import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

interface RateLimitResult {
  allowed: boolean;
  remaining: number;
  resetAt: number;
}

class TokenBucketLimiter {
  constructor(
    private redis: Redis,
    private maxTokens: number,
    private refillRate: number,  // tokens per second
    private keyPrefix: string
  ) {}

  async check(key: string): Promise<RateLimitResult> {
    const now = Date.now();
    const bucketKey = `${this.keyPrefix}:${key}`;
    
    // Lua script for atomic operation
    const script = `
      local key = KEYS[1]
      local maxTokens = tonumber(ARGV[1])
      local refillRate = tonumber(ARGV[2])
      local now = tonumber(ARGV[3])
      
      local bucket = redis.call('HMGET', key, 'tokens', 'lastRefill')
      local tokens = tonumber(bucket[1]) or maxTokens
      local lastRefill = tonumber(bucket[2]) or now
      
      -- Refill tokens based on time passed
      local timePassed = (now - lastRefill) / 1000
      local newTokens = math.min(maxTokens, tokens + (timePassed * refillRate))
      
      if newTokens >= 1 then
        -- Consume a token
        newTokens = newTokens - 1
        redis.call('HMSET', key, 'tokens', newTokens, 'lastRefill', now)
        redis.call('EXPIRE', key, 60)
        return {1, newTokens, 0}  -- allowed
      else
        -- Calculate when next token available
        local waitTime = (1 - newTokens) / refillRate * 1000
        return {0, 0, now + waitTime}  -- not allowed
      end
    `;
    
    const result = await this.redis.eval(
      script,
      1,
      bucketKey,
      this.maxTokens,
      this.refillRate,
      now
    ) as [number, number, number];
    
    return {
      allowed: result[0] === 1,
      remaining: result[1],
      resetAt: result[2],
    };
  }
}

// Usage
const limiter = new TokenBucketLimiter(redis, 100, 10, 'ratelimit');
// 100 requests max, refills 10 per second
```

### Rate Limiting Middleware

```typescript
// ════════════════════════════════════════════════════════════════
// RATE LIMITING MIDDLEWARE
// ════════════════════════════════════════════════════════════════

interface RateLimitConfig {
  windowMs: number;      // Time window in milliseconds
  max: number;           // Max requests per window
  keyGenerator?: (req: Request) => string;
  skip?: (req: Request) => boolean;
}

const createRateLimiter = (config: RateLimitConfig) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    // Skip if configured
    if (config.skip?.(req)) {
      return next();
    }
    
    // Generate key (default: IP address)
    const key = config.keyGenerator?.(req) || req.ip;
    
    const result = await limiter.check(key);
    
    // Set rate limit headers
    res.setHeader('X-RateLimit-Limit', config.max);
    res.setHeader('X-RateLimit-Remaining', result.remaining);
    res.setHeader('X-RateLimit-Reset', Math.ceil(result.resetAt / 1000));
    
    if (!result.allowed) {
      res.setHeader('Retry-After', Math.ceil((result.resetAt - Date.now()) / 1000));
      
      return res.status(429).json({
        error: 'Too Many Requests',
        message: 'Rate limit exceeded',
        retryAfter: Math.ceil((result.resetAt - Date.now()) / 1000),
      });
    }
    
    next();
  };
};

// ════════════════════════════════════════════════════════════════
// DIFFERENT RATE LIMITS FOR DIFFERENT SCENARIOS
// ════════════════════════════════════════════════════════════════

// Global rate limit (per IP)
const globalLimiter = createRateLimiter({
  windowMs: 60000,  // 1 minute
  max: 100,
  keyGenerator: (req) => req.ip,
});

// Per-user rate limit
const userLimiter = createRateLimiter({
  windowMs: 60000,
  max: 1000,  // Higher limit for authenticated users
  keyGenerator: (req) => req.user?.id || req.ip,
  skip: (req) => !req.user,  // Only for authenticated users
});

// Per-endpoint rate limit (stricter for expensive operations)
const expensiveOpLimiter = createRateLimiter({
  windowMs: 60000,
  max: 10,  // Only 10 per minute
  keyGenerator: (req) => `${req.user?.id}:${req.path}`,
});

// Tier-based rate limits
const tierLimiter = createRateLimiter({
  windowMs: 60000,
  max: 100,  // Default
  keyGenerator: (req) => {
    const tier = req.user?.tier || 'free';
    const limits = { free: 100, pro: 1000, enterprise: 10000 };
    // Store tier-specific limit in request for middleware
    req.rateLimit = limits[tier];
    return req.user?.id || req.ip;
  },
});

// ════════════════════════════════════════════════════════════════
// APPLY RATE LIMITERS
// ════════════════════════════════════════════════════════════════

// Global limit first
app.use(globalLimiter);

// Then user-specific limits
app.use('/api', authenticate, userLimiter);

// Stricter limits for specific endpoints
app.post('/api/reports/generate', expensiveOpLimiter, proxyToReportsService);
app.post('/api/export', expensiveOpLimiter, proxyToExportService);
```

### Distributed Rate Limiting (Backpressure)

```typescript
// ════════════════════════════════════════════════════════════════
// BACKPRESSURE: Slow responses when overloaded
// ════════════════════════════════════════════════════════════════

class AdaptiveRateLimiter {
  private currentLoad = 0;
  private maxLoad = 1000;  // Max concurrent requests
  
  async middleware(req: Request, res: Response, next: NextFunction) {
    // Track concurrent requests
    this.currentLoad++;
    
    try {
      // Calculate load percentage
      const loadPercentage = this.currentLoad / this.maxLoad;
      
      // Adaptive delay based on load
      if (loadPercentage > 0.8) {
        // At 80%+ load, add delay (backpressure)
        const delay = Math.min(5000, (loadPercentage - 0.8) * 25000);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
      
      // At 95%+ load, start rejecting (load shedding)
      if (loadPercentage > 0.95) {
        return res.status(503).json({
          error: 'Service Overloaded',
          retryAfter: 5,
        });
      }
      
      next();
    } finally {
      this.currentLoad--;
    }
  }
}

// ════════════════════════════════════════════════════════════════
// REDIS-BASED SLIDING WINDOW (Distributed)
// ════════════════════════════════════════════════════════════════

async function slidingWindowRateLimit(
  redis: Redis,
  key: string,
  windowMs: number,
  maxRequests: number
): Promise<{ allowed: boolean; current: number }> {
  const now = Date.now();
  const windowStart = now - windowMs;
  
  // Lua script for atomic sliding window
  const script = `
    local key = KEYS[1]
    local now = tonumber(ARGV[1])
    local windowStart = tonumber(ARGV[2])
    local maxRequests = tonumber(ARGV[3])
    local windowMs = tonumber(ARGV[4])
    
    -- Remove old entries
    redis.call('ZREMRANGEBYSCORE', key, '-inf', windowStart)
    
    -- Count current requests
    local current = redis.call('ZCARD', key)
    
    if current < maxRequests then
      -- Add new request
      redis.call('ZADD', key, now, now .. ':' .. math.random())
      redis.call('PEXPIRE', key, windowMs)
      return {1, current + 1}
    else
      return {0, current}
    end
  `;
  
  const result = await redis.eval(
    script, 1, key, now, windowStart, maxRequests, windowMs
  ) as [number, number];
  
  return { allowed: result[0] === 1, current: result[1] };
}
```

---

## 5. Aggregation & BFF

### API Aggregation Pattern

```typescript
// ════════════════════════════════════════════════════════════════
// AGGREGATION: Combine multiple service calls into one response
// ════════════════════════════════════════════════════════════════

// Without aggregation: Mobile app makes 5 requests
// GET /api/users/123
// GET /api/users/123/orders
// GET /api/users/123/recommendations
// GET /api/users/123/notifications
// GET /api/cart

// With aggregation: Mobile app makes 1 request
// GET /api/dashboard → Returns all data in one response

app.get('/api/dashboard', authenticate, async (req, res) => {
  const userId = req.user.id;
  
  try {
    // Parallel requests to all services
    const [user, orders, recommendations, notifications, cart] = await Promise.all([
      fetch(`${services.users}/${userId}`).then(r => r.json()),
      fetch(`${services.orders}?userId=${userId}&limit=5`).then(r => r.json()),
      fetch(`${services.recommendations}/${userId}`).then(r => r.json()),
      fetch(`${services.notifications}/${userId}?unread=true`).then(r => r.json()),
      fetch(`${services.cart}/${userId}`).then(r => r.json()),
    ]);
    
    // Aggregate into single response
    res.json({
      user: {
        id: user.id,
        name: user.name,
        avatar: user.avatar,
      },
      recentOrders: orders.slice(0, 3),
      recommendations: recommendations.slice(0, 5),
      unreadNotifications: notifications.count,
      cartItemCount: cart.items.length,
    });
    
  } catch (error) {
    // Handle partial failures gracefully
    res.status(500).json({ error: 'Failed to load dashboard' });
  }
});

// ════════════════════════════════════════════════════════════════
// AGGREGATION WITH PARTIAL FAILURE HANDLING
// ════════════════════════════════════════════════════════════════

interface ServiceResult<T> {
  data?: T;
  error?: string;
  source: string;
}

async function fetchWithFallback<T>(
  url: string,
  source: string,
  fallback: T
): Promise<ServiceResult<T>> {
  try {
    const response = await fetch(url, { timeout: 3000 });
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    const data = await response.json();
    return { data, source };
  } catch (error) {
    console.error(`${source} failed:`, error.message);
    return { data: fallback, error: error.message, source };
  }
}

app.get('/api/dashboard', authenticate, async (req, res) => {
  const userId = req.user.id;
  
  const results = await Promise.all([
    fetchWithFallback(`${services.users}/${userId}`, 'users', null),
    fetchWithFallback(`${services.orders}?userId=${userId}`, 'orders', []),
    fetchWithFallback(`${services.recommendations}/${userId}`, 'recommendations', []),
    fetchWithFallback(`${services.notifications}/${userId}`, 'notifications', { count: 0 }),
  ]);
  
  const [user, orders, recommendations, notifications] = results;
  const errors = results.filter(r => r.error).map(r => r.source);
  
  res.json({
    user: user.data,
    recentOrders: orders.data,
    recommendations: recommendations.data,
    unreadNotifications: notifications.data?.count || 0,
    _partial: errors.length > 0,
    _errors: errors,
  });
});
```

### Backend for Frontend (BFF) Pattern

```typescript
// ════════════════════════════════════════════════════════════════
// BFF: Separate gateways optimized for each client type
// ════════════════════════════════════════════════════════════════

// mobile-bff.ts - Optimized for mobile
const mobileBFF = express();

mobileBFF.get('/api/home', authenticate, async (req, res) => {
  // Mobile needs minimal data, optimized for bandwidth
  const [user, feed] = await Promise.all([
    fetchUser(req.user.id, ['id', 'name', 'avatarUrl']),  // Only essential fields
    fetchFeed(req.user.id, 10),  // Smaller page size
  ]);
  
  res.json({
    user,
    feed: feed.map(item => ({
      id: item.id,
      title: item.title,
      thumbnailUrl: item.images.thumbnail,  // Small images only
      summary: item.content.substring(0, 100),  // Truncated content
    })),
  });
});

// web-bff.ts - Optimized for web
const webBFF = express();

webBFF.get('/api/home', authenticate, async (req, res) => {
  // Web can handle more data
  const [user, feed, sidebar, analytics] = await Promise.all([
    fetchUser(req.user.id),  // Full user object
    fetchFeed(req.user.id, 25),  // Larger page
    fetchSidebarWidgets(req.user.id),
    fetchUserAnalytics(req.user.id),
  ]);
  
  res.json({
    user,
    feed: feed.map(item => ({
      ...item,
      images: item.images,  // Full resolution
      content: item.content,  // Full content
    })),
    sidebar,
    analytics,
  });
});

// admin-bff.ts - Optimized for admin dashboard
const adminBFF = express();

adminBFF.get('/api/overview', requireRole('admin'), async (req, res) => {
  const [metrics, recentUsers, alerts, systemHealth] = await Promise.all([
    fetchAdminMetrics(),
    fetchRecentUsers(20),
    fetchActiveAlerts(),
    fetchSystemHealth(),
  ]);
  
  res.json({ metrics, recentUsers, alerts, systemHealth });
});
```

---

## 6. Resilience Patterns

### Circuit Breaker

```typescript
// ════════════════════════════════════════════════════════════════
// CIRCUIT BREAKER: Prevent cascade failures
// ════════════════════════════════════════════════════════════════

enum CircuitState {
  CLOSED = 'CLOSED',      // Normal operation
  OPEN = 'OPEN',          // Failing, reject requests
  HALF_OPEN = 'HALF_OPEN' // Testing if service recovered
}

class CircuitBreaker {
  private state: CircuitState = CircuitState.CLOSED;
  private failures = 0;
  private successes = 0;
  private lastFailureTime = 0;
  
  constructor(
    private failureThreshold: number = 5,      // Open after 5 failures
    private successThreshold: number = 2,       // Close after 2 successes
    private timeout: number = 30000,            // Try again after 30s
  ) {}
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    // Check if circuit should transition
    this.checkState();
    
    if (this.state === CircuitState.OPEN) {
      throw new CircuitOpenError('Circuit is open');
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  private checkState(): void {
    if (this.state === CircuitState.OPEN) {
      // Check if timeout has passed
      if (Date.now() - this.lastFailureTime >= this.timeout) {
        this.state = CircuitState.HALF_OPEN;
        this.successes = 0;
      }
    }
  }
  
  private onSuccess(): void {
    if (this.state === CircuitState.HALF_OPEN) {
      this.successes++;
      if (this.successes >= this.successThreshold) {
        this.state = CircuitState.CLOSED;
        this.failures = 0;
      }
    }
    this.failures = 0;  // Reset failures on success
  }
  
  private onFailure(): void {
    this.failures++;
    this.lastFailureTime = Date.now();
    
    if (this.failures >= this.failureThreshold) {
      this.state = CircuitState.OPEN;
    }
  }
  
  getState(): CircuitState {
    this.checkState();
    return this.state;
  }
}

// Circuit breaker per service
const circuits = new Map<string, CircuitBreaker>();

function getCircuit(serviceName: string): CircuitBreaker {
  if (!circuits.has(serviceName)) {
    circuits.set(serviceName, new CircuitBreaker());
  }
  return circuits.get(serviceName)!;
}

// Usage in gateway
app.get('/api/users/:id', async (req, res) => {
  const circuit = getCircuit('users-service');
  
  try {
    const user = await circuit.execute(async () => {
      const response = await fetch(`${services.users}/${req.params.id}`);
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return response.json();
    });
    
    res.json(user);
  } catch (error) {
    if (error instanceof CircuitOpenError) {
      // Return cached data or fallback
      res.status(503).json({
        error: 'Service temporarily unavailable',
        fallback: await getCachedUser(req.params.id),
      });
    } else {
      res.status(500).json({ error: error.message });
    }
  }
});
```

### Retry with Exponential Backoff

```typescript
// ════════════════════════════════════════════════════════════════
// RETRY WITH EXPONENTIAL BACKOFF
// ════════════════════════════════════════════════════════════════

interface RetryConfig {
  maxRetries: number;
  baseDelay: number;
  maxDelay: number;
  retryableErrors: number[];
}

async function fetchWithRetry<T>(
  url: string,
  options: RequestInit = {},
  config: RetryConfig = {
    maxRetries: 3,
    baseDelay: 100,
    maxDelay: 5000,
    retryableErrors: [408, 429, 500, 502, 503, 504],
  }
): Promise<T> {
  let lastError: Error;
  
  for (let attempt = 0; attempt <= config.maxRetries; attempt++) {
    try {
      const response = await fetch(url, {
        ...options,
        signal: AbortSignal.timeout(10000),  // 10s timeout
      });
      
      if (!response.ok) {
        if (config.retryableErrors.includes(response.status)) {
          throw new RetryableError(`HTTP ${response.status}`);
        }
        throw new Error(`HTTP ${response.status}`);
      }
      
      return await response.json();
      
    } catch (error) {
      lastError = error;
      
      if (!(error instanceof RetryableError) || attempt === config.maxRetries) {
        throw error;
      }
      
      // Exponential backoff with jitter
      const delay = Math.min(
        config.maxDelay,
        config.baseDelay * Math.pow(2, attempt) + Math.random() * 100
      );
      
      console.log(`Retry ${attempt + 1}/${config.maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  
  throw lastError!;
}

// ════════════════════════════════════════════════════════════════
// TIMEOUT HANDLING
// ════════════════════════════════════════════════════════════════

const withTimeout = <T>(
  promise: Promise<T>,
  timeoutMs: number,
  errorMessage = 'Operation timed out'
): Promise<T> => {
  return Promise.race([
    promise,
    new Promise<never>((_, reject) =>
      setTimeout(() => reject(new TimeoutError(errorMessage)), timeoutMs)
    ),
  ]);
};

// Usage
const user = await withTimeout(
  fetch(`${services.users}/${userId}`).then(r => r.json()),
  5000,
  'Users service timed out'
);
```

### Response Caching

```typescript
// ════════════════════════════════════════════════════════════════
// RESPONSE CACHING AT GATEWAY
// ════════════════════════════════════════════════════════════════

import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

interface CacheConfig {
  ttl: number;           // Time to live in seconds
  staleWhileRevalidate?: number;  // Serve stale while fetching fresh
  keyGenerator?: (req: Request) => string;
  shouldCache?: (req: Request, res: any) => boolean;
}

const cacheMiddleware = (config: CacheConfig) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    // Only cache GET requests
    if (req.method !== 'GET') return next();
    
    const cacheKey = config.keyGenerator?.(req) || `cache:${req.originalUrl}`;
    
    // Try to get from cache
    const cached = await redis.get(cacheKey);
    
    if (cached) {
      const { data, timestamp } = JSON.parse(cached);
      const age = (Date.now() - timestamp) / 1000;
      
      res.setHeader('X-Cache', 'HIT');
      res.setHeader('Age', Math.floor(age));
      
      // Check if stale
      if (age > config.ttl && config.staleWhileRevalidate) {
        // Serve stale, revalidate in background
        res.setHeader('X-Cache', 'STALE');
        setImmediate(() => refreshCache(req, cacheKey, config));
      }
      
      return res.json(data);
    }
    
    // Cache miss - intercept response
    const originalJson = res.json.bind(res);
    res.json = (data: any) => {
      // Check if should cache
      if (config.shouldCache?.(req, data) !== false) {
        const cacheData = JSON.stringify({ data, timestamp: Date.now() });
        redis.setex(cacheKey, config.ttl + (config.staleWhileRevalidate || 0), cacheData);
      }
      
      res.setHeader('X-Cache', 'MISS');
      return originalJson(data);
    };
    
    next();
  };
};

// Usage
app.get('/api/products', 
  cacheMiddleware({ 
    ttl: 300,  // 5 minutes
    staleWhileRevalidate: 60,  // Serve stale for 1 more minute
    keyGenerator: (req) => `products:${req.query.category || 'all'}`,
  }),
  proxyToProductsService
);
```

---

## 7. Implementation

### Kong Configuration Example

```yaml
# kong.yml - Declarative Kong configuration
_format_version: "3.0"

services:
  - name: users-service
    url: http://users-service:3001
    routes:
      - name: users-route
        paths:
          - /api/users
        strip_path: true
    plugins:
      - name: rate-limiting
        config:
          minute: 100
          policy: redis
          redis_host: redis
      - name: jwt
        config:
          secret_is_base64: false
      - name: cors
        config:
          origins:
            - "*"
          methods:
            - GET
            - POST
            - PUT
            - DELETE

  - name: orders-service
    url: http://orders-service:3002
    routes:
      - name: orders-route
        paths:
          - /api/orders
    plugins:
      - name: rate-limiting
        config:
          minute: 50
      - name: jwt
      - name: request-transformer
        config:
          add:
            headers:
              - "X-Gateway: kong"

plugins:
  - name: prometheus  # Global metrics
  - name: correlation-id
    config:
      header_name: X-Request-ID
      generator: uuid
```

### AWS API Gateway (Serverless)

```yaml
# serverless.yml
service: my-api

provider:
  name: aws
  runtime: nodejs18.x
  
  # API Gateway configuration
  httpApi:
    cors: true
    authorizers:
      jwtAuthorizer:
        type: jwt
        identitySource: $request.header.Authorization
        issuerUrl: https://auth.myapp.com/
        audience:
          - my-api

functions:
  getUsers:
    handler: src/handlers/users.get
    events:
      - httpApi:
          path: /users
          method: get
          authorizer:
            name: jwtAuthorizer
  
  createUser:
    handler: src/handlers/users.create
    events:
      - httpApi:
          path: /users
          method: post
          authorizer:
            name: jwtAuthorizer
          throttling:
            maxRequestsPerSecond: 10
            burstLimit: 20

  # Rate limiting per user via custom authorizer
  customAuth:
    handler: src/auth/authorizer.handler

resources:
  Resources:
    # Usage plan for API keys
    UsagePlan:
      Type: AWS::ApiGateway::UsagePlan
      Properties:
        UsagePlanName: standard
        Throttle:
          BurstLimit: 100
          RateLimit: 50
        Quota:
          Limit: 10000
          Period: MONTH
```

---

## 8. When to Use / Not Use

### When TO Use an API Gateway

```
✅ USE API GATEWAY WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. MICROSERVICES ARCHITECTURE                                  │
│     └── Multiple backend services                              │
│     └── Need single entry point for clients                    │
│     └── Services should not be directly exposed                │
│                                                                  │
│  2. CROSS-CUTTING CONCERNS                                      │
│     └── Auth needs to happen once, not in every service       │
│     └── Consistent rate limiting across APIs                   │
│     └── Centralized logging and monitoring                     │
│                                                                  │
│  3. CLIENT SIMPLIFICATION                                       │
│     └── Mobile apps need aggregated responses                  │
│     └── Different APIs for different clients (BFF)            │
│     └── Protocol translation needed                            │
│                                                                  │
│  4. API VERSIONING & MANAGEMENT                                 │
│     └── Multiple API versions                                  │
│     └── A/B testing, canary deployments                        │
│     └── API analytics and monetization                         │
│                                                                  │
│  5. SECURITY REQUIREMENTS                                       │
│     └── SSL termination                                        │
│     └── DDoS protection at edge                                │
│     └── Request validation                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When NOT to Use an API Gateway

```
❌ DON'T USE API GATEWAY WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. SIMPLE MONOLITH                                             │
│     └── Single backend service                                 │
│     └── No need for aggregation                                │
│     └── Gateway adds unnecessary complexity                    │
│                                                                  │
│  2. INTERNAL SERVICE-TO-SERVICE                                │
│     └── Use service mesh instead (Istio, Linkerd)             │
│     └── Gateway is for external (north-south) traffic         │
│     └── Mesh handles internal (east-west) traffic             │
│                                                                  │
│  3. REAL-TIME APPLICATIONS                                      │
│     └── WebSocket-heavy apps may need direct connections      │
│     └── Gaming, video streaming                                │
│     └── Gateway adds latency                                   │
│                                                                  │
│  4. HIGH-PERFORMANCE REQUIREMENTS                               │
│     └── Every millisecond counts                               │
│     └── Direct service access is faster                        │
│     └── Consider edge computing instead                        │
│                                                                  │
│  5. SMALL TEAM / EARLY STAGE                                   │
│     └── Operational overhead may not be worth it              │
│     └── Start simple, add gateway when needed                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Interview Questions & Answers

### Basic Questions

**Q1: What is an API Gateway?**
> **A:** An API Gateway is a single entry point for all client requests. It handles cross-cutting concerns like authentication, rate limiting, routing, and logging - decoupling clients from the complexity of multiple backend services. Think of it as a reverse proxy with application-level intelligence.

**Q2: Gateway vs Load Balancer?**
> **A:**
> - **Load Balancer**: Distributes traffic across servers (L4/L7), health checks, simple routing. Doesn't understand application logic.
> - **API Gateway**: Application-aware - handles auth, rate limiting, request transformation, aggregation. Often sits behind a load balancer.
>
> In production: Load Balancer → API Gateway cluster → Backend services.

**Q3: What is the BFF pattern?**
> **A:** Backend for Frontend - one gateway per client type (web, mobile, IoT). Each BFF is optimized for its client's needs: mobile BFF returns minimal data to save bandwidth, web BFF includes richer responses. Avoids one-size-fits-all API that serves no one well.

**Q4: How do you handle auth at the gateway?**
> **A:** Validate JWT locally using public keys (JWKS) - no network call to auth server per request. Check signature, expiration, and claims. If valid, attach user context as headers for downstream services. Services trust the gateway, don't re-validate.

### Intermediate Questions

**Q5: How do you implement rate limiting?**
> **A:** 
> - **Algorithm**: Token bucket (allows bursts) or sliding window (more accurate)
> - **Storage**: Redis for distributed counters
> - **Levels**: Per IP (global), per user (authenticated), per endpoint (expensive operations)
> - **Response**: 429 status with `Retry-After` header
> - **Headers**: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`

**Q6: What is a circuit breaker?**
> **A:** Pattern to prevent cascade failures. Three states:
> - **Closed**: Normal operation, requests pass through
> - **Open**: Service failing, reject requests immediately (fail fast)
> - **Half-Open**: Test if service recovered with limited requests
>
> Opens after threshold failures (e.g., 5), closes after threshold successes (e.g., 2) in half-open state. Prevents hammering a failing service.

**Q7: How do you handle partial failures in aggregation?**
> **A:** 
> - Make parallel requests with individual timeouts
> - Use fallback values for failed services (cached data, defaults)
> - Return partial response with `_partial: true` flag
> - Include error details so client knows what's missing
> - Consider which services are critical vs optional

**Q8: Gateway vs Service Mesh?**
> **A:**
> - **Gateway**: External traffic (north-south), client-facing, handles auth/rate limiting
> - **Service Mesh**: Internal traffic (east-west), service-to-service, handles mTLS/retries/observability
>
> Use both: Gateway for external clients, mesh for internal communication. They complement each other.

### Advanced Questions

**Q9: How do you scale an API Gateway?**
> **A:**
> - **Stateless design**: No local state, scale horizontally
> - **External state**: Rate limit counters in Redis, session data externalized
> - **Load balancer**: Multiple gateway instances behind LB
> - **Auto-scaling**: Based on request rate or CPU
> - **Multi-AZ**: Deploy across availability zones
> - **Caching**: Response caching reduces backend load

**Q10: How do you implement canary deployments through the gateway?**
> **A:**
> - **Weighted routing**: 95% to stable, 5% to canary
> - **Header-based**: Route users with specific header to canary
> - **User-based**: Route specific user IDs or groups to canary
> - **Gradual increase**: Monitor metrics, increase canary traffic if healthy
> - **Automatic rollback**: Circuit breaker trips if canary has high error rate

**Q11: What about backpressure at the gateway?**
> **A:**
> - **Rate limiting**: First line of defense, reject excess requests
> - **Adaptive throttling**: Slow responses under load (add delay)
> - **Load shedding**: At extreme load, reject some requests (503)
> - **Queue-based**: Buffer requests with bounded queue, fail fast when full
> - **Circuit breaker**: Stop calling failing backends
> - **Return 429/503** with `Retry-After` header to signal clients to back off

**Q12: How do you secure an API Gateway?**
> **A:**
> - **SSL/TLS termination**: Decrypt at gateway, HTTP internally
> - **JWT validation**: Verify signatures locally (JWKS)
> - **API key rotation**: Support multiple active keys during rotation
> - **Request validation**: Validate payloads before forwarding
> - **IP allowlisting**: For admin APIs or known partners
> - **DDoS protection**: Rate limiting, WAF integration
> - **Secrets management**: Vault/KMS for sensitive config

### Scenario Questions

**Q13: Design a gateway for a mobile app with 5 backend services**
> **A:**
> 1. **BFF pattern**: Create mobile-specific gateway
> 2. **Aggregation endpoint**: `/api/mobile/home` combines 5 calls
> 3. **Optimized responses**: Return only needed fields, compressed images
> 4. **Auth**: JWT validated once at gateway
> 5. **Rate limiting**: Per-user limits, higher for premium users
> 6. **Resilience**: Circuit breaker per service, partial failure handling
> 7. **Caching**: Cache stable data (products), short TTL for dynamic (feed)

**Q14: Your gateway is adding 200ms latency. How do you diagnose and fix?**
> **A:** 
> **Diagnose**:
> - Add timing headers at each stage (auth, routing, proxy)
> - Check if auth is making network calls (should be local JWT validation)
> - Check connection pooling to backends
> - Look for DNS resolution delays
>
> **Fix**:
> - Local JWT validation (no auth server call)
> - Connection pooling/keep-alive to backends
> - Response caching for static data
> - Move expensive logic to async (logging, analytics)
> - Consider edge deployment closer to users

---

## 🎓 Key Takeaways

1. **API Gateway = single entry point** for all client requests
2. **Cross-cutting concerns** centralized: auth, rate limiting, logging
3. **BFF pattern** - optimize gateway per client type
4. **JWT validation locally** - no network call per request
5. **Rate limiting** with Redis - per user, IP, endpoint
6. **Circuit breaker** prevents cascade failures
7. **Aggregation** reduces client round-trips
8. **Backpressure** through rate limiting and load shedding
9. **Gateway for external traffic**, service mesh for internal
10. **Don't over-engineer** - start simple, add gateway when needed

---

## 📚 Resources

### Documentation
- [Kong Gateway](https://docs.konghq.com/)
- [AWS API Gateway](https://docs.aws.amazon.com/apigateway/)
- [NGINX as API Gateway](https://www.nginx.com/learn/api-gateway/)

### Tools
- [Kong](https://konghq.com/)
- [Traefik](https://traefik.io/)
- [Envoy Proxy](https://www.envoyproxy.io/)
- [Express Gateway](https://www.express-gateway.io/)

### Patterns
- [Microservices.io - API Gateway](https://microservices.io/patterns/apigateway.html)
- [BFF Pattern - Sam Newman](https://samnewman.io/patterns/architectural/bff/)


