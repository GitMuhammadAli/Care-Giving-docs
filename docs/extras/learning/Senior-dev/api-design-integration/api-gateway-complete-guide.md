# 🚪 API Gateway - Complete Guide

> A comprehensive guide to API Gateways - Kong, AWS API Gateway, routing, rate limiting, transformation, and building the front door to your microservices.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "An API Gateway is a single entry point for all client requests to your microservices, handling cross-cutting concerns like authentication, rate limiting, routing, caching, monitoring, and protocol translation - essentially acting as a reverse proxy with superpowers."

### The 7 Key Concepts (Remember These!)
```
1. ROUTING           → Direct requests to appropriate services
2. AUTHENTICATION    → Verify identity at edge
3. RATE LIMITING     → Protect services from overload
4. TRANSFORMATION    → Modify request/response format
5. CACHING           → Reduce backend load
6. LOAD BALANCING    → Distribute traffic
7. MONITORING        → Centralized observability
```

### API Gateway Pattern
```
┌─────────────────────────────────────────────────────────────────┐
│                   API GATEWAY PATTERN                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CLIENTS                                                        │
│  ───────                                                        │
│    Web App ─────┐                                               │
│    Mobile App ──┼─────>  ┌─────────────────────┐                │
│    Third Party ─┤        │    API GATEWAY      │                │
│    IoT Device ──┘        │                     │                │
│                          │  • Authentication   │                │
│                          │  • Rate Limiting    │                │
│                          │  • Routing          │                │
│                          │  • Caching          │                │
│                          │  • Logging          │                │
│                          │  • Transformation   │                │
│                          └─────────┬───────────┘                │
│                                    │                            │
│  SERVICES                          ▼                            │
│  ────────                                                       │
│    ┌──────────────────────────────────────────┐                │
│    │                                          │                │
│    ▼           ▼           ▼           ▼      │                │
│  ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐       │                │
│  │User │   │Order│   │Prod │   │Pay  │       │                │
│  │Svc  │   │Svc  │   │Svc  │   │Svc  │       │                │
│  └─────┘   └─────┘   └─────┘   └─────┘       │                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### API Gateway Options
```
┌─────────────────────────────────────────────────────────────────┐
│                 API GATEWAY OPTIONS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  KONG                                                          │
│  ────                                                           │
│  • Open source, Lua/Nginx based                                │
│  • Plugin architecture                                         │
│  • Self-hosted or Kong Cloud                                   │
│  • Good for: Customization, on-prem                            │
│                                                                 │
│  AWS API GATEWAY                                               │
│  ────────────────                                               │
│  • Fully managed, serverless                                   │
│  • Deep AWS integration                                        │
│  • Pay per request                                             │
│  • Good for: AWS workloads, Lambda                             │
│                                                                 │
│  NGINX                                                         │
│  ─────                                                          │
│  • High performance                                            │
│  • Reverse proxy first                                         │
│  • Lua scripting for custom logic                              │
│  • Good for: Simple routing, performance                       │
│                                                                 │
│  ENVOY                                                         │
│  ─────                                                          │
│  • Service mesh friendly                                       │
│  • Dynamic configuration                                       │
│  • gRPC native                                                 │
│  • Good for: Kubernetes, Istio                                 │
│                                                                 │
│  TRAEFIK                                                       │
│  ───────                                                        │
│  • Cloud native                                                │
│  • Auto-discovery                                              │
│  • Let's Encrypt built-in                                      │
│  • Good for: Docker, Kubernetes                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Edge service"** | "The API Gateway serves as our edge service" |
| **"BFF pattern"** | "We have separate BFFs for web and mobile" |
| **"Request aggregation"** | "Gateway handles request aggregation for dashboard" |
| **"Protocol translation"** | "Gateway translates REST to gRPC for internal services" |
| **"Circuit breaker"** | "Gateway implements circuit breaker for resilience" |
| **"Service discovery"** | "Dynamic routing via service discovery integration" |

### Key Numbers to Remember
| Metric | Target | Why |
|--------|-------|-----|
| Latency overhead | **< 10ms** | Gateway shouldn't be bottleneck |
| Availability | **99.99%** | Single point of entry |
| Request timeout | **30s default** | Prevent hanging connections |
| Cache hit rate | **> 60%** | Reduce backend load |

### The "Wow" Statement (Memorize This!)
> "We use Kong as our API Gateway, handling all cross-cutting concerns at the edge. Authentication happens at gateway - JWT validation with rate limiting per API key. We have different gateways per client type (BFF pattern) - mobile gets optimized payloads. Request aggregation combines multiple service calls for the dashboard endpoint. Gateway caches GET requests with 60-second TTL, improving response times by 40%. We use circuit breakers to prevent cascade failures. Routing is dynamic via service discovery (Consul). Observability is centralized - all requests logged with correlation IDs for tracing. The gateway adds ~5ms latency, worth it for the centralized control."

---

## 📚 Table of Contents

1. [Core Features](#1-core-features)
2. [Kong Gateway](#2-kong-gateway)
3. [AWS API Gateway](#3-aws-api-gateway)
4. [Custom Gateway](#4-custom-gateway)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Core Features

```typescript
// ════════════════════════════════════════════════════════════════
// API GATEWAY CORE FEATURES
// ════════════════════════════════════════════════════════════════

/*
 * 1. ROUTING
 *    - Path-based: /users → User Service
 *    - Header-based: X-Version: v2 → v2 Service
 *    - Query-based: ?region=eu → EU Service
 *    - Method-based: POST /orders → Order Create Service
 * 
 * 2. AUTHENTICATION
 *    - JWT validation
 *    - API key verification
 *    - OAuth token introspection
 *    - mTLS for service-to-service
 * 
 * 3. RATE LIMITING
 *    - Per client/API key
 *    - Per endpoint
 *    - Per time window
 * 
 * 4. TRANSFORMATION
 *    - Request: Add headers, modify body
 *    - Response: Filter fields, format change
 *    - Protocol: REST → gRPC
 * 
 * 5. CACHING
 *    - Response caching
 *    - Cache invalidation
 *    - Vary headers
 * 
 * 6. LOAD BALANCING
 *    - Round robin
 *    - Weighted
 *    - Least connections
 *    - Health checks
 * 
 * 7. MONITORING
 *    - Request logging
 *    - Metrics collection
 *    - Distributed tracing
 */

// ════════════════════════════════════════════════════════════════
// REQUEST FLOW THROUGH GATEWAY
// ════════════════════════════════════════════════════════════════

/*
 * REQUEST LIFECYCLE
 * 
 * 1. Client Request
 *    │
 * 2. ├── Rate Limit Check
 *    │   └── If exceeded → 429 Response
 *    │
 * 3. ├── Authentication
 *    │   └── If invalid → 401 Response
 *    │
 * 4. ├── Authorization
 *    │   └── If forbidden → 403 Response
 *    │
 * 5. ├── Cache Lookup
 *    │   └── If hit → Return cached response
 *    │
 * 6. ├── Request Transformation
 *    │   └── Add headers, modify body
 *    │
 * 7. ├── Route to Service
 *    │   └── Load balance, circuit break
 *    │
 * 8. ├── Response Transformation
 *    │   └── Filter, format
 *    │
 * 9. ├── Cache Store
 *    │
 * 10.└── Return Response to Client
 */

// ════════════════════════════════════════════════════════════════
// BFF (Backend for Frontend) PATTERN
// ════════════════════════════════════════════════════════════════

/*
 * Instead of one gateway for all clients,
 * create specific gateways per client type:
 * 
 *   Web App ──────> Web BFF ─────┐
 *                                │
 *   Mobile App ───> Mobile BFF ──┼──> Microservices
 *                                │
 *   Third Party ──> Public API ──┘
 * 
 * Benefits:
 * - Optimized responses per client
 * - Different caching strategies
 * - Client-specific aggregation
 * - Easier versioning
 */
```

---

## 2. Kong Gateway

```yaml
# ════════════════════════════════════════════════════════════════
# KONG CONFIGURATION
# ════════════════════════════════════════════════════════════════

# kong.yml - Declarative configuration

_format_version: "3.0"

# ════════════════════════════════════════════════════════════════
# SERVICES (Backend services)
# ════════════════════════════════════════════════════════════════

services:
  - name: user-service
    url: http://user-service:3000
    connect_timeout: 5000
    write_timeout: 60000
    read_timeout: 60000
    retries: 3

  - name: order-service
    url: http://order-service:3000
    connect_timeout: 5000
    retries: 3

  - name: product-service
    url: http://product-service:3000
    connect_timeout: 5000

# ════════════════════════════════════════════════════════════════
# ROUTES (URL patterns to services)
# ════════════════════════════════════════════════════════════════

routes:
  # User routes
  - name: users-route
    service: user-service
    paths:
      - /api/users
      - /api/auth
    methods:
      - GET
      - POST
      - PUT
      - DELETE
    strip_path: false

  # Order routes
  - name: orders-route
    service: order-service
    paths:
      - /api/orders
    methods:
      - GET
      - POST
      - PUT
    headers:
      x-api-version:
        - v1
        - v2

  # Product routes (public, no auth)
  - name: products-public-route
    service: product-service
    paths:
      - /api/public/products

# ════════════════════════════════════════════════════════════════
# PLUGINS (Cross-cutting concerns)
# ════════════════════════════════════════════════════════════════

plugins:
  # Global rate limiting
  - name: rate-limiting
    config:
      minute: 100
      policy: redis
      redis_host: redis
      redis_port: 6379

  # JWT authentication (global)
  - name: jwt
    config:
      key_claim_name: kid
      claims_to_verify:
        - exp

  # CORS
  - name: cors
    config:
      origins:
        - https://app.example.com
      methods:
        - GET
        - POST
        - PUT
        - DELETE
      headers:
        - Accept
        - Authorization
        - Content-Type
      credentials: true
      max_age: 3600

  # Request logging
  - name: http-log
    config:
      http_endpoint: http://logging-service:3000/logs
      method: POST
      content_type: application/json

  # Response transformer (add headers)
  - name: response-transformer
    config:
      add:
        headers:
          - "X-Served-By: Kong"
          - "X-Request-Id: $(request_id)"

# ════════════════════════════════════════════════════════════════
# ROUTE-SPECIFIC PLUGINS
# ════════════════════════════════════════════════════════════════

  # Higher rate limit for products (public)
  - name: rate-limiting
    route: products-public-route
    config:
      minute: 1000
      policy: local

  # Skip auth for public products
  - name: jwt
    route: products-public-route
    enabled: false

  # Request size limit for orders
  - name: request-size-limiting
    route: orders-route
    config:
      allowed_payload_size: 5  # MB

  # Caching for product listings
  - name: proxy-cache
    route: products-public-route
    config:
      response_code:
        - 200
      request_method:
        - GET
      content_type:
        - application/json
      cache_ttl: 300
      strategy: memory

# ════════════════════════════════════════════════════════════════
# CONSUMERS (API clients)
# ════════════════════════════════════════════════════════════════

consumers:
  - username: mobile-app
    custom_id: mobile-app-v1
    
  - username: web-app
    custom_id: web-app-v1
    
  - username: partner-api
    custom_id: partner-acme

# Consumer plugins (per-consumer rate limits)
plugins:
  - name: rate-limiting
    consumer: partner-api
    config:
      minute: 1000  # Higher limit for partner
```

```bash
# ════════════════════════════════════════════════════════════════
# KONG ADMIN API COMMANDS
# ════════════════════════════════════════════════════════════════

# Create service
curl -X POST http://localhost:8001/services \
  -d name=user-service \
  -d url=http://user-service:3000

# Create route
curl -X POST http://localhost:8001/services/user-service/routes \
  -d "paths[]=/api/users" \
  -d name=users-route

# Enable plugin
curl -X POST http://localhost:8001/services/user-service/plugins \
  -d name=rate-limiting \
  -d config.minute=100

# Create consumer
curl -X POST http://localhost:8001/consumers \
  -d username=mobile-app

# Create JWT credential for consumer
curl -X POST http://localhost:8001/consumers/mobile-app/jwt \
  -d key=mobile-app-key \
  -d secret=your-secret

# Health check
curl http://localhost:8001/status
```

---

## 3. AWS API Gateway

```yaml
# ════════════════════════════════════════════════════════════════
# AWS API GATEWAY (OpenAPI + Extensions)
# ════════════════════════════════════════════════════════════════

openapi: "3.0.1"
info:
  title: "User Management API"
  version: "1.0"

# Custom domain
x-amazon-apigateway-endpoint-configuration:
  types:
    - REGIONAL

paths:
  /users:
    get:
      summary: List users
      security:
        - jwt-authorizer: []
      x-amazon-apigateway-integration:
        type: HTTP_PROXY
        uri: http://${stageVariables.userServiceUrl}/users
        httpMethod: GET
        connectionType: VPC_LINK
        connectionId: ${stageVariables.vpcLinkId}
        requestParameters:
          integration.request.header.X-Correlation-Id: context.requestId
        responses:
          default:
            statusCode: "200"
    
    post:
      summary: Create user
      security:
        - jwt-authorizer: []
      x-amazon-apigateway-request-validator: all
      x-amazon-apigateway-integration:
        type: AWS_PROXY
        uri: arn:aws:apigateway:${region}:lambda:path/2015-03-31/functions/${createUserFunctionArn}/invocations
        httpMethod: POST
        passthroughBehavior: WHEN_NO_MATCH

  /users/{userId}:
    get:
      summary: Get user by ID
      parameters:
        - name: userId
          in: path
          required: true
          schema:
            type: string
      x-amazon-apigateway-integration:
        type: HTTP_PROXY
        uri: http://${stageVariables.userServiceUrl}/users/{userId}
        httpMethod: GET
        requestParameters:
          integration.request.path.userId: method.request.path.userId

# ════════════════════════════════════════════════════════════════
# SECURITY (JWT Authorizer)
# ════════════════════════════════════════════════════════════════

components:
  securitySchemes:
    jwt-authorizer:
      type: apiKey
      name: Authorization
      in: header
      x-amazon-apigateway-authtype: COGNITO_USER_POOLS
      x-amazon-apigateway-authorizer:
        type: COGNITO_USER_POOLS
        providerARNs:
          - arn:aws:cognito-idp:${region}:${account}:userpool/${userPoolId}

# Custom Lambda authorizer
    lambda-authorizer:
      type: apiKey
      name: Authorization
      in: header
      x-amazon-apigateway-authtype: CUSTOM
      x-amazon-apigateway-authorizer:
        type: REQUEST
        authorizerUri: arn:aws:apigateway:${region}:lambda:path/2015-03-31/functions/${authorizerFunctionArn}/invocations
        authorizerResultTtlInSeconds: 300
        identitySource: method.request.header.Authorization
```

```typescript
// ════════════════════════════════════════════════════════════════
// AWS CDK API GATEWAY SETUP
// ════════════════════════════════════════════════════════════════

import * as cdk from 'aws-cdk-lib';
import * as apigateway from 'aws-cdk-lib/aws-apigateway';
import * as lambda from 'aws-cdk-lib/aws-lambda';

export class ApiGatewayStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string) {
    super(scope, id);

    // Create API
    const api = new apigateway.RestApi(this, 'UserApi', {
      restApiName: 'User Service',
      description: 'User management API',
      deployOptions: {
        stageName: 'prod',
        throttlingBurstLimit: 100,
        throttlingRateLimit: 50,
        cachingEnabled: true,
        cacheTtl: cdk.Duration.minutes(5),
      },
      defaultCorsPreflightOptions: {
        allowOrigins: apigateway.Cors.ALL_ORIGINS,
        allowMethods: ['GET', 'POST', 'PUT', 'DELETE'],
      },
    });

    // Lambda function
    const getUserHandler = new lambda.Function(this, 'GetUserHandler', {
      runtime: lambda.Runtime.NODEJS_18_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset('lambda/get-user'),
    });

    // Create resource and method
    const users = api.root.addResource('users');
    const user = users.addResource('{userId}');

    // GET /users/{userId}
    user.addMethod('GET', new apigateway.LambdaIntegration(getUserHandler), {
      authorizationType: apigateway.AuthorizationType.IAM,
      requestParameters: {
        'method.request.path.userId': true,
      },
      methodResponses: [
        {
          statusCode: '200',
          responseParameters: {
            'method.response.header.Access-Control-Allow-Origin': true,
          },
        },
      ],
    });

    // Usage plan and API key
    const plan = api.addUsagePlan('UsagePlan', {
      name: 'Standard',
      throttle: {
        rateLimit: 100,
        burstLimit: 200,
      },
      quota: {
        limit: 10000,
        period: apigateway.Period.MONTH,
      },
    });

    const key = api.addApiKey('ApiKey');
    plan.addApiKey(key);
  }
}
```

---

## 4. Custom Gateway

```typescript
// ════════════════════════════════════════════════════════════════
// CUSTOM API GATEWAY (Express-based)
// ════════════════════════════════════════════════════════════════

import express, { Request, Response, NextFunction } from 'express';
import httpProxy from 'http-proxy-middleware';
import rateLimit from 'express-rate-limit';
import jwt from 'jsonwebtoken';
import Redis from 'ioredis';

const app = express();
const redis = new Redis();

// ════════════════════════════════════════════════════════════════
// SERVICE REGISTRY
// ════════════════════════════════════════════════════════════════

const services: Record<string, { url: string; healthPath: string }> = {
  'user-service': {
    url: process.env.USER_SERVICE_URL || 'http://localhost:3001',
    healthPath: '/health',
  },
  'order-service': {
    url: process.env.ORDER_SERVICE_URL || 'http://localhost:3002',
    healthPath: '/health',
  },
  'product-service': {
    url: process.env.PRODUCT_SERVICE_URL || 'http://localhost:3003',
    healthPath: '/health',
  },
};

// ════════════════════════════════════════════════════════════════
// MIDDLEWARE: REQUEST ID
// ════════════════════════════════════════════════════════════════

function requestIdMiddleware(req: Request, res: Response, next: NextFunction) {
  const requestId = req.headers['x-request-id'] || crypto.randomUUID();
  req.requestId = requestId as string;
  res.setHeader('X-Request-Id', requestId);
  next();
}

// ════════════════════════════════════════════════════════════════
// MIDDLEWARE: LOGGING
// ════════════════════════════════════════════════════════════════

function loggingMiddleware(req: Request, res: Response, next: NextFunction) {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(JSON.stringify({
      timestamp: new Date().toISOString(),
      requestId: req.requestId,
      method: req.method,
      path: req.path,
      status: res.statusCode,
      duration,
      userAgent: req.headers['user-agent'],
      ip: req.ip,
    }));
  });
  
  next();
}

// ════════════════════════════════════════════════════════════════
// MIDDLEWARE: AUTHENTICATION
// ════════════════════════════════════════════════════════════════

interface AuthenticatedRequest extends Request {
  user?: { id: string; role: string };
}

async function authMiddleware(
  req: AuthenticatedRequest, 
  res: Response, 
  next: NextFunction
) {
  // Skip auth for public routes
  if (req.path.startsWith('/api/public/')) {
    return next();
  }

  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Missing authentication' });
  }

  const token = authHeader.substring(7);
  
  try {
    // Check token blacklist
    const isBlacklisted = await redis.get(`blacklist:${token}`);
    if (isBlacklisted) {
      return res.status(401).json({ error: 'Token revoked' });
    }

    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as {
      sub: string;
      role: string;
    };
    
    req.user = { id: decoded.sub, role: decoded.role };
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}

// ════════════════════════════════════════════════════════════════
// MIDDLEWARE: RATE LIMITING
// ════════════════════════════════════════════════════════════════

const rateLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 100,
  standardHeaders: true,
  legacyHeaders: false,
  keyGenerator: (req: AuthenticatedRequest) => {
    return req.user?.id || req.ip || 'anonymous';
  },
  handler: (req, res) => {
    res.status(429).json({
      error: 'rate_limit_exceeded',
      retry_after: 60,
    });
  },
});

// ════════════════════════════════════════════════════════════════
// MIDDLEWARE: CACHING
// ════════════════════════════════════════════════════════════════

async function cacheMiddleware(
  req: Request, 
  res: Response, 
  next: NextFunction
) {
  // Only cache GET requests
  if (req.method !== 'GET') {
    return next();
  }

  const cacheKey = `cache:${req.originalUrl}`;
  const cached = await redis.get(cacheKey);

  if (cached) {
    const { body, headers } = JSON.parse(cached);
    res.setHeader('X-Cache', 'HIT');
    Object.entries(headers).forEach(([key, value]) => {
      res.setHeader(key, value as string);
    });
    return res.json(body);
  }

  // Store original json method
  const originalJson = res.json.bind(res);
  
  res.json = (body: any) => {
    // Cache successful responses
    if (res.statusCode === 200) {
      const headers = {
        'Content-Type': res.getHeader('Content-Type'),
      };
      redis.setex(cacheKey, 60, JSON.stringify({ body, headers }));
    }
    res.setHeader('X-Cache', 'MISS');
    return originalJson(body);
  };

  next();
}

// ════════════════════════════════════════════════════════════════
// MIDDLEWARE: CIRCUIT BREAKER
// ════════════════════════════════════════════════════════════════

class CircuitBreaker {
  private failures: Map<string, number> = new Map();
  private lastFailure: Map<string, number> = new Map();
  private readonly threshold = 5;
  private readonly timeout = 30000; // 30 seconds

  isOpen(service: string): boolean {
    const failures = this.failures.get(service) || 0;
    const lastFailure = this.lastFailure.get(service) || 0;
    
    if (failures >= this.threshold) {
      // Check if timeout has passed
      if (Date.now() - lastFailure < this.timeout) {
        return true; // Circuit is open
      }
      // Reset after timeout
      this.failures.set(service, 0);
    }
    return false;
  }

  recordFailure(service: string) {
    const current = this.failures.get(service) || 0;
    this.failures.set(service, current + 1);
    this.lastFailure.set(service, Date.now());
  }

  recordSuccess(service: string) {
    this.failures.set(service, 0);
  }
}

const circuitBreaker = new CircuitBreaker();

// ════════════════════════════════════════════════════════════════
// PROXY CONFIGURATION
// ════════════════════════════════════════════════════════════════

function createProxy(serviceName: string) {
  const service = services[serviceName];
  
  return httpProxy.createProxyMiddleware({
    target: service.url,
    changeOrigin: true,
    pathRewrite: {
      [`^/api/${serviceName}`]: '',
    },
    onProxyReq: (proxyReq, req: AuthenticatedRequest) => {
      // Add headers
      proxyReq.setHeader('X-Request-Id', req.requestId!);
      if (req.user) {
        proxyReq.setHeader('X-User-Id', req.user.id);
        proxyReq.setHeader('X-User-Role', req.user.role);
      }
    },
    onProxyRes: (proxyRes, req, res) => {
      circuitBreaker.recordSuccess(serviceName);
    },
    onError: (err, req, res) => {
      circuitBreaker.recordFailure(serviceName);
      (res as Response).status(503).json({
        error: 'service_unavailable',
        message: `${serviceName} is temporarily unavailable`,
      });
    },
  });
}

// ════════════════════════════════════════════════════════════════
// ROUTE SETUP
// ════════════════════════════════════════════════════════════════

// Apply global middleware
app.use(requestIdMiddleware);
app.use(loggingMiddleware);
app.use(express.json());

// Health check (no auth)
app.get('/health', (req, res) => res.json({ status: 'healthy' }));

// Apply auth and rate limiting to API routes
app.use('/api', authMiddleware);
app.use('/api', rateLimiter);

// Caching for public routes
app.use('/api/public', cacheMiddleware);

// Circuit breaker check
app.use('/api/:service/*', (req, res, next) => {
  const service = req.params.service;
  if (circuitBreaker.isOpen(service)) {
    return res.status(503).json({
      error: 'circuit_open',
      message: `${service} is temporarily unavailable`,
    });
  }
  next();
});

// Service proxies
app.use('/api/users', createProxy('user-service'));
app.use('/api/orders', createProxy('order-service'));
app.use('/api/public/products', createProxy('product-service'));

// Start server
app.listen(8080, () => {
  console.log('API Gateway running on port 8080');
});
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# API GATEWAY BEST PRACTICES
# ════════════════════════════════════════════════════════════════

security:
  authentication:
    - Validate tokens at gateway (not each service)
    - Use short-lived tokens
    - Implement token blacklist for revocation
    - Pass user context via headers to services
    
  rate_limiting:
    - Global limits + per-endpoint limits
    - Different limits per user tier
    - Always return rate limit headers
    
  protection:
    - Enable CORS properly
    - Validate request size
    - Sanitize headers before forwarding
    - DDoS protection at edge

performance:
  caching:
    - Cache at gateway when possible
    - Use appropriate TTLs
    - Cache key should include relevant params
    - Support cache invalidation
    
  timeouts:
    - Set appropriate timeouts (don't wait forever)
    - Different timeouts per endpoint
    - Return gateway timeout (504) gracefully
    
  connection_pooling:
    - Reuse connections to backend services
    - Configure pool size appropriately

resilience:
  circuit_breaker:
    - Fail fast when service is down
    - Auto-recover after timeout
    - Return meaningful error messages
    
  retries:
    - Retry idempotent requests only
    - Use exponential backoff
    - Set max retry limits
    
  health_checks:
    - Check backend service health
    - Remove unhealthy instances from routing

observability:
  logging:
    - Log all requests with correlation ID
    - Include timing information
    - Mask sensitive data
    
  metrics:
    - Request rate, latency, error rate
    - Per-service and per-endpoint
    - Rate limit hits
    
  tracing:
    - Propagate trace context to services
    - Use standard headers (traceparent)
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# API GATEWAY PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Gateway as single point of failure
# Bad - No redundancy
# One gateway instance for all traffic

# Good
# - Multiple gateway instances behind load balancer
# - Auto-scaling based on load
# - Multi-region deployment

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Too much business logic in gateway
# Bad
# Gateway validates business rules
# Gateway transforms complex data
# Gateway makes multiple service calls with logic

# Good
# Gateway handles cross-cutting concerns only:
# - Auth, rate limiting, routing, caching
# Complex logic belongs in services

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: No timeout configuration
# Bad
# Default or no timeouts
# Requests hang when services are slow

# Good
# - Connection timeout: 5 seconds
# - Read timeout: 30 seconds (or appropriate)
# - Return 504 Gateway Timeout when exceeded

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Coupling gateway to service internals
# Bad
# Gateway knows about service implementation
# Gateway changes required for service changes

# Good
# - Gateway only knows routes
# - Services define their own contracts
# - Loose coupling via standard interfaces

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Missing health checks
# Bad
# Route to services without checking health
# Users get errors for down services

# Good
# - Active health checks
# - Remove unhealthy instances
# - Proper circuit breaker pattern

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Auth at each service instead of gateway
# Bad
# Each service validates JWT
# Duplicated auth logic everywhere

# Good
# - Validate at gateway once
# - Pass user info via trusted headers
# - Services trust internal headers
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is an API Gateway?"**
> "An API Gateway is a single entry point for clients to access microservices. Handles cross-cutting concerns: authentication, rate limiting, routing, caching, logging. Acts as reverse proxy with additional features. Examples: Kong, AWS API Gateway, NGINX."

**Q: "Why use an API Gateway?"**
> "Centralized security (auth once, not per service), simplified client interface (single endpoint), cross-cutting concerns handled in one place, protocol translation (REST to gRPC), request aggregation, and centralized monitoring. Clients don't need to know service topology."

**Q: "What's the difference between API Gateway and Load Balancer?"**
> "Load balancer distributes traffic across instances of same service (L4/L7). API Gateway routes to different services based on request, plus adds features like auth, transformation, caching. Gateway operates at application layer with request awareness. Often used together: LB → Gateway → Services."

### Intermediate Questions

**Q: "What is the BFF pattern?"**
> "Backend For Frontend - separate gateway per client type (web, mobile, third-party). Each BFF is optimized for its client: different payload sizes, aggregation patterns, caching strategies. Mobile BFF might aggregate more to reduce round trips. Prevents one-size-fits-all API."

**Q: "How do you handle auth at the gateway?"**
> "Validate JWT tokens at gateway, not each service. Extract user info (ID, roles), pass to services via trusted headers. Gateway handles token revocation (blacklist), refresh. Services trust headers from gateway (internal network only). This centralizes auth, simplifies services."

**Q: "What is request aggregation?"**
> "Gateway combines multiple service calls into single client request. Example: Dashboard needs user + orders + recommendations. Instead of 3 client calls, gateway makes them internally and returns combined response. Reduces latency (parallel calls), simplifies client. Use sparingly - too much logic doesn't belong in gateway."

### Advanced Questions

**Q: "How do you make API Gateway highly available?"**
> "Multiple instances behind load balancer. Auto-scaling based on CPU/requests. Stateless design (no session state, use Redis for distributed state). Health checks for instances. Multi-region deployment for disaster recovery. Gateway shouldn't be single point of failure."

**Q: "How do you handle versioning at the gateway?"**
> "Multiple approaches: 1) Path: /v1/users, /v2/users - route to different services. 2) Header: X-API-Version, route based on header. 3) Same service handles multiple versions. Gateway can transform requests/responses between versions. Deprecation: return headers about upcoming changes."

**Q: "What's the difference between API Gateway and Service Mesh?"**
> "API Gateway: North-south traffic (external to internal), client-facing. Service Mesh (Istio, Linkerd): East-west traffic (service to service), with sidecar pattern. Mesh provides: mTLS between services, traffic management, observability. They complement each other - gateway at edge, mesh internally."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  API GATEWAY CHECKLIST                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CORE FEATURES:                                                 │
│  □ Request routing to services                                 │
│  □ Authentication/Authorization                                │
│  □ Rate limiting                                               │
│  □ Response caching                                            │
│  □ Request/Response logging                                    │
│                                                                 │
│  RESILIENCE:                                                    │
│  □ Circuit breaker                                             │
│  □ Timeouts configured                                         │
│  □ Health checks                                               │
│  □ Retry logic (idempotent only)                               │
│                                                                 │
│  OBSERVABILITY:                                                 │
│  □ Correlation IDs                                             │
│  □ Metrics collection                                          │
│  □ Distributed tracing                                         │
│  □ Error tracking                                              │
│                                                                 │
│  HIGH AVAILABILITY:                                             │
│  □ Multiple instances                                          │
│  □ Auto-scaling                                                │
│  □ No single point of failure                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

GATEWAY OPTIONS:
┌────────────────────────────────────────────────────────────────┐
│ Kong:         Open source, plugin-based, self-hosted          │
│ AWS API GW:   Managed, serverless, AWS integrated             │
│ NGINX:        High performance, simple routing                │
│ Envoy:        Service mesh, dynamic config, gRPC native       │
│ Traefik:      Cloud native, auto-discovery                    │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

