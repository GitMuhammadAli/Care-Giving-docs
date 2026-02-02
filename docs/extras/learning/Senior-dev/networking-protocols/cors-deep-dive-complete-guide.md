# CORS Deep Dive - Complete Guide

> **MUST REMEMBER**: CORS (Cross-Origin Resource Sharing) is a browser security feature that blocks requests from different origins. Simple requests (GET/POST with basic headers) go directly; complex requests trigger a preflight OPTIONS request first. Server must respond with Access-Control-Allow-Origin header. Common issues: missing preflight handling, credentials with wildcards, wrong headers. CORS is browser-enforced, not server-side security.

---

## How to Explain Like a Senior Developer

"CORS is the browser's way of asking 'is this cross-origin request allowed?'. If your frontend at app.com calls api.com, that's cross-origin - different protocol, domain, or port counts. Simple requests (GET, POST with standard content types) go directly with Origin header, and the browser checks the response headers. Complex requests (PUT, DELETE, custom headers, JSON content-type) trigger a preflight OPTIONS request first. The server must respond with appropriate Access-Control headers. Key gotcha: CORS is ONLY enforced by browsers, not by curl or server-to-server calls. It's not security against attackers - it protects users from malicious sites making requests with their cookies."

---

## Core Implementation

### Express CORS Middleware

```typescript
// cors/express-middleware.ts
import { Request, Response, NextFunction } from 'express';

interface CorsOptions {
  allowedOrigins: string[] | '*';
  allowedMethods: string[];
  allowedHeaders: string[];
  exposedHeaders?: string[];
  credentials: boolean;
  maxAge?: number; // Preflight cache in seconds
}

const defaultOptions: CorsOptions = {
  allowedOrigins: '*',
  allowedMethods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With'],
  exposedHeaders: ['X-Total-Count', 'X-Page'],
  credentials: false,
  maxAge: 86400, // 24 hours
};

export function createCorsMiddleware(options: Partial<CorsOptions> = {}) {
  const config = { ...defaultOptions, ...options };
  
  return (req: Request, res: Response, next: NextFunction): void => {
    const origin = req.headers.origin;
    
    // Check if origin is allowed
    let allowedOrigin: string | null = null;
    
    if (config.allowedOrigins === '*') {
      // Wildcard - but can't use with credentials!
      if (config.credentials && origin) {
        allowedOrigin = origin; // Reflect origin instead
      } else {
        allowedOrigin = '*';
      }
    } else if (origin && config.allowedOrigins.includes(origin)) {
      allowedOrigin = origin;
    }
    
    if (allowedOrigin) {
      res.setHeader('Access-Control-Allow-Origin', allowedOrigin);
    }
    
    // Handle credentials
    if (config.credentials) {
      res.setHeader('Access-Control-Allow-Credentials', 'true');
    }
    
    // Expose custom headers to JavaScript
    if (config.exposedHeaders && config.exposedHeaders.length > 0) {
      res.setHeader('Access-Control-Expose-Headers', config.exposedHeaders.join(', '));
    }
    
    // Handle preflight request
    if (req.method === 'OPTIONS') {
      res.setHeader('Access-Control-Allow-Methods', config.allowedMethods.join(', '));
      res.setHeader('Access-Control-Allow-Headers', config.allowedHeaders.join(', '));
      
      if (config.maxAge) {
        res.setHeader('Access-Control-Max-Age', config.maxAge.toString());
      }
      
      // Preflight doesn't need a body
      res.status(204).end();
      return;
    }
    
    next();
  };
}

// Usage
import express from 'express';

const app = express();

// Apply CORS globally
app.use(createCorsMiddleware({
  allowedOrigins: ['https://app.example.com', 'https://admin.example.com'],
  credentials: true,
}));

// Or per-route
app.get('/api/public', 
  createCorsMiddleware({ allowedOrigins: '*' }),
  (req, res) => {
    res.json({ public: true });
  }
);
```

### Dynamic Origin Handling

```typescript
// cors/dynamic-origin.ts
import { Request, Response, NextFunction } from 'express';

interface DynamicCorsConfig {
  // Function to determine if origin is allowed
  originValidator: (origin: string, req: Request) => boolean | Promise<boolean>;
  credentials?: boolean;
}

// Allow origins dynamically (e.g., from database)
export function dynamicCorsMiddleware(config: DynamicCorsConfig) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const origin = req.headers.origin;
    
    if (!origin) {
      // No origin header = same origin or non-browser
      return next();
    }
    
    try {
      const isAllowed = await config.originValidator(origin, req);
      
      if (isAllowed) {
        res.setHeader('Access-Control-Allow-Origin', origin);
        
        if (config.credentials) {
          res.setHeader('Access-Control-Allow-Credentials', 'true');
        }
        
        if (req.method === 'OPTIONS') {
          res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
          res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
          res.setHeader('Access-Control-Max-Age', '86400');
          return res.status(204).end();
        }
      }
      
      next();
    } catch (error) {
      next(error);
    }
  };
}

// Example: Allow any subdomain of example.com
function subdomainValidator(origin: string, req: Request): boolean {
  const pattern = /^https:\/\/([a-z0-9-]+\.)?example\.com$/;
  return pattern.test(origin);
}

// Example: Allow origins from database
async function databaseValidator(origin: string, req: Request): Promise<boolean> {
  // In real app, query database
  const allowedOrigins = await getAllowedOriginsFromDb();
  return allowedOrigins.includes(origin);
}

async function getAllowedOriginsFromDb(): Promise<string[]> {
  // Simulated database query
  return ['https://app.example.com', 'https://partner.com'];
}

// Example: Tenant-based CORS
async function tenantValidator(origin: string, req: Request): Promise<boolean> {
  // Get tenant from subdomain or header
  const tenant = extractTenant(req);
  const tenantConfig = await getTenantConfig(tenant);
  
  return tenantConfig.allowedOrigins.includes(origin);
}

function extractTenant(req: Request): string { return 'default'; }
async function getTenantConfig(tenant: string): Promise<{ allowedOrigins: string[] }> {
  return { allowedOrigins: [] };
}
```

### Handling Preflight Requests

```typescript
// cors/preflight.ts

/**
 * Preflight requests occur when:
 * 1. Method is not GET, HEAD, or POST
 * 2. POST with Content-Type other than:
 *    - application/x-www-form-urlencoded
 *    - multipart/form-data
 *    - text/plain
 * 3. Custom headers are included (Authorization, X-Custom-Header)
 */

// Example of what triggers preflight
const requestsThatTriggerPreflight = [
  // PUT/DELETE always trigger preflight
  { method: 'PUT', headers: {} },
  { method: 'DELETE', headers: {} },
  
  // POST with JSON content-type
  { method: 'POST', headers: { 'Content-Type': 'application/json' } },
  
  // Any request with custom header
  { method: 'GET', headers: { 'Authorization': 'Bearer token' } },
  { method: 'GET', headers: { 'X-Custom-Header': 'value' } },
];

// Preflight request example
const preflightRequest = {
  method: 'OPTIONS',
  headers: {
    'Origin': 'https://app.example.com',
    'Access-Control-Request-Method': 'PUT',
    'Access-Control-Request-Headers': 'Content-Type, Authorization',
  },
};

// Server should respond with
const preflightResponse = {
  status: 204,
  headers: {
    'Access-Control-Allow-Origin': 'https://app.example.com',
    'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
    'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    'Access-Control-Max-Age': '86400',
    'Access-Control-Allow-Credentials': 'true',
  },
};

// Optimizing preflight with caching
import { Request, Response, NextFunction } from 'express';

function optimizedPreflightHandler(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  if (req.method === 'OPTIONS') {
    // Cache preflight for 24 hours
    res.setHeader('Access-Control-Max-Age', '86400');
    
    // Vary header for proper caching
    res.setHeader('Vary', 'Origin, Access-Control-Request-Method, Access-Control-Request-Headers');
    
    res.status(204).end();
    return;
  }
  
  next();
}
```

### Credentials and Cookies

```typescript
// cors/credentials.ts

/**
 * To send cookies/auth with CORS requests:
 * 
 * Client: credentials: 'include' in fetch
 * Server: Access-Control-Allow-Credentials: true
 *         Access-Control-Allow-Origin: specific origin (NOT *)
 */

// Server configuration for cookies
import { Request, Response, NextFunction } from 'express';

function corsWithCredentials(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  const origin = req.headers.origin;
  const allowedOrigins = ['https://app.example.com', 'https://admin.example.com'];
  
  if (origin && allowedOrigins.includes(origin)) {
    // MUST be specific origin, not *
    res.setHeader('Access-Control-Allow-Origin', origin);
    
    // Enable credentials
    res.setHeader('Access-Control-Allow-Credentials', 'true');
    
    // Cookies need SameSite=None and Secure in modern browsers
    // Set when creating cookies:
    // res.cookie('session', token, {
    //   httpOnly: true,
    //   secure: true,
    //   sameSite: 'none',
    // });
  }
  
  if (req.method === 'OPTIONS') {
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    return res.status(204).end();
  }
  
  next();
}

// Client-side fetch with credentials
async function fetchWithCredentials(url: string): Promise<Response> {
  return fetch(url, {
    credentials: 'include', // Send cookies
    headers: {
      'Content-Type': 'application/json',
    },
  });
}

// Axios with credentials
import axios from 'axios';

const apiClient = axios.create({
  baseURL: 'https://api.example.com',
  withCredentials: true, // Send cookies
});
```

### Nginx CORS Configuration

```nginx
# cors/nginx.conf

# Global CORS configuration
map $http_origin $cors_origin {
    default "";
    "https://app.example.com" "$http_origin";
    "https://admin.example.com" "$http_origin";
    "~^https://.*\.example\.com$" "$http_origin";
}

server {
    listen 443 ssl;
    server_name api.example.com;
    
    # Add CORS headers to all responses
    add_header 'Access-Control-Allow-Origin' $cors_origin always;
    add_header 'Access-Control-Allow-Credentials' 'true' always;
    add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range' always;
    
    # Handle preflight
    if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' $cors_origin;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,Authorization';
        add_header 'Access-Control-Max-Age' 86400;
        add_header 'Content-Type' 'text/plain charset=UTF-8';
        add_header 'Content-Length' 0;
        return 204;
    }
    
    location /api {
        proxy_pass http://backend:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## Real-World Scenarios

### Scenario 1: Debugging CORS Errors

```typescript
// cors/debugging.ts

/**
 * Common CORS error messages and solutions
 */

const corsErrors = {
  // Error: No 'Access-Control-Allow-Origin' header
  noHeader: {
    message: "No 'Access-Control-Allow-Origin' header is present on the requested resource",
    causes: [
      'Server not sending CORS headers',
      'Server error before CORS middleware runs',
      'OPTIONS request returning error',
    ],
    solutions: [
      'Add CORS middleware before other routes',
      'Handle OPTIONS requests explicitly',
      'Check server logs for errors',
      'Verify the endpoint exists',
    ],
  },
  
  // Error: Wildcard with credentials
  wildcardCredentials: {
    message: "The value of the 'Access-Control-Allow-Origin' header must not be '*' when credentials mode is 'include'",
    causes: [
      "Using '*' with credentials: true",
    ],
    solutions: [
      'Use specific origin instead of wildcard',
      'Reflect the Origin header when credentials needed',
    ],
  },
  
  // Error: Method not allowed
  methodNotAllowed: {
    message: "Method PUT is not allowed by Access-Control-Allow-Methods",
    causes: [
      'Preflight response missing the method',
    ],
    solutions: [
      'Add method to Access-Control-Allow-Methods',
      'Ensure OPTIONS handler returns correct methods',
    ],
  },
  
  // Error: Header not allowed
  headerNotAllowed: {
    message: "Request header field authorization is not allowed by Access-Control-Allow-Headers",
    causes: [
      'Custom header not listed in allowed headers',
    ],
    solutions: [
      'Add header to Access-Control-Allow-Headers',
      'Check for typos in header name',
    ],
  },
  
  // Error: Preflight failure
  preflightFailed: {
    message: "Response to preflight request doesn't pass access control check",
    causes: [
      'OPTIONS request returns error status',
      'Server not handling OPTIONS method',
      'Middleware blocking preflight',
    ],
    solutions: [
      'Add explicit OPTIONS route handler',
      'Ensure CORS middleware runs for OPTIONS',
      'Return 204 for successful preflight',
    ],
  },
};

// Debug middleware to log CORS issues
import { Request, Response, NextFunction } from 'express';

function corsDebugMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  const origin = req.headers.origin;
  
  console.log('CORS Debug:', {
    method: req.method,
    origin,
    path: req.path,
    headers: {
      'access-control-request-method': req.headers['access-control-request-method'],
      'access-control-request-headers': req.headers['access-control-request-headers'],
    },
  });
  
  // Log response headers after response
  const originalSend = res.send.bind(res);
  res.send = function(body) {
    console.log('CORS Response Headers:', {
      'access-control-allow-origin': res.getHeader('access-control-allow-origin'),
      'access-control-allow-credentials': res.getHeader('access-control-allow-credentials'),
      'access-control-allow-methods': res.getHeader('access-control-allow-methods'),
      'access-control-allow-headers': res.getHeader('access-control-allow-headers'),
    });
    return originalSend(body);
  };
  
  next();
}
```

### Scenario 2: Multi-Environment CORS Setup

```typescript
// cors/multi-env.ts
import { Request, Response, NextFunction } from 'express';

interface EnvironmentCorsConfig {
  development: string[];
  staging: string[];
  production: string[];
}

const corsConfig: EnvironmentCorsConfig = {
  development: [
    'http://localhost:3000',
    'http://localhost:5173', // Vite
    'http://127.0.0.1:3000',
  ],
  staging: [
    'https://staging.example.com',
    'https://preview-*.example.com', // Preview deployments
  ],
  production: [
    'https://app.example.com',
    'https://www.example.com',
  ],
};

function createEnvironmentAwareCors() {
  const env = process.env.NODE_ENV || 'development';
  const allowedOrigins = corsConfig[env as keyof EnvironmentCorsConfig] || [];
  
  return (req: Request, res: Response, next: NextFunction) => {
    const origin = req.headers.origin;
    
    if (!origin) {
      return next();
    }
    
    // Check if origin matches (including wildcards in staging)
    const isAllowed = allowedOrigins.some(pattern => {
      if (pattern.includes('*')) {
        // Convert wildcard to regex
        const regex = new RegExp('^' + pattern.replace('*', '.*') + '$');
        return regex.test(origin);
      }
      return pattern === origin;
    });
    
    if (isAllowed) {
      res.setHeader('Access-Control-Allow-Origin', origin);
      res.setHeader('Access-Control-Allow-Credentials', 'true');
      
      if (req.method === 'OPTIONS') {
        res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS');
        res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
        res.setHeader('Access-Control-Max-Age', '86400');
        return res.status(204).end();
      }
    } else if (env === 'development') {
      // In dev, log unallowed origins for debugging
      console.warn(`CORS blocked origin: ${origin}`);
    }
    
    next();
  };
}
```

---

## Common Pitfalls

### 1. Wildcard with Credentials

```typescript
// ❌ BAD: Wildcard doesn't work with credentials
app.use(cors({
  origin: '*',
  credentials: true, // Browser will reject this!
}));

// ✅ GOOD: Specific origin with credentials
app.use(cors({
  origin: 'https://app.example.com',
  credentials: true,
}));

// ✅ GOOD: Dynamic origin reflection
app.use(cors({
  origin: (origin, callback) => {
    const allowed = ['https://app.example.com', 'https://admin.example.com'];
    if (!origin || allowed.includes(origin)) {
      callback(null, origin);
    } else {
      callback(new Error('Not allowed'));
    }
  },
  credentials: true,
}));

import cors from 'cors';
import express from 'express';
const app = express();
```

### 2. Forgetting OPTIONS Handling

```typescript
// ❌ BAD: OPTIONS not handled
app.put('/api/data', authMiddleware, handler);
// Preflight fails because authMiddleware runs!

// ✅ GOOD: Handle OPTIONS before auth
app.options('/api/data', cors()); // CORS handles OPTIONS
app.put('/api/data', cors(), authMiddleware, handler);

// Or better: Global CORS before routes
app.use(cors());
app.put('/api/data', authMiddleware, handler);

function authMiddleware(req: Request, res: Response, next: NextFunction): void { next(); }
function handler(req: Request, res: Response): void {}

import { Request, Response, NextFunction } from 'express';
```

### 3. Not Varying on Origin

```typescript
// ❌ BAD: Caching without Vary header
// CDN caches response for origin A, serves to origin B

// ✅ GOOD: Include Vary header
res.setHeader('Vary', 'Origin');
res.setHeader('Access-Control-Allow-Origin', origin);

// Also vary on other access-control headers
res.setHeader('Vary', 'Origin, Access-Control-Request-Method, Access-Control-Request-Headers');

const origin = '';
const res = { setHeader: (name: string, value: string) => {} };
```

---

## Interview Questions

### Q1: What is CORS and why does it exist?

**A:** CORS (Cross-Origin Resource Sharing) is a browser security mechanism that restricts web pages from making requests to different origins. It exists to protect users - without it, a malicious site could make requests to your bank using your cookies. The Same-Origin Policy blocks cross-origin requests by default; CORS is the safe way to explicitly allow specific cross-origin access.

### Q2: What triggers a preflight request?

**A:** Preflight (OPTIONS) occurs for "non-simple" requests: 1) Methods other than GET, HEAD, POST. 2) POST with Content-Type other than form-urlencoded, multipart, or text/plain. 3) Requests with custom headers (Authorization, X-Custom-*). The browser asks "is this allowed?" before sending the actual request.

### Q3: Why can't you use '*' with credentials?

**A:** Security reason: if `*` worked with credentials, any malicious site could make authenticated requests. When credentials are involved, the server must explicitly name the allowed origin. This forces developers to consciously decide which origins can access authenticated resources.

### Q4: Is CORS server-side security?

**A:** No! CORS is browser-enforced. Server-to-server requests, curl, and Postman ignore CORS completely. CORS protects users from malicious websites, not your API from attackers. You still need authentication, authorization, and rate limiting for actual security. CORS just controls which websites can use browser features (cookies, fetch) to access your API.

---

## Quick Reference Checklist

### Basic Setup
- [ ] Add CORS middleware before routes
- [ ] Handle OPTIONS requests
- [ ] Set appropriate Access-Control-Allow-Origin
- [ ] Configure allowed methods and headers

### Credentials
- [ ] Use specific origin (not wildcard)
- [ ] Set Access-Control-Allow-Credentials: true
- [ ] Configure cookies with SameSite=None, Secure

### Performance
- [ ] Set Access-Control-Max-Age for caching
- [ ] Add Vary header for proper caching
- [ ] Minimize preflight by using simple requests

### Troubleshooting
- [ ] Check browser console for specific error
- [ ] Verify OPTIONS returns 204 with headers
- [ ] Confirm origin is in allowed list
- [ ] Check for errors before CORS middleware

---

*Last updated: February 2026*

