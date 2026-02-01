# 🔒 API Security - Complete Guide

> A comprehensive guide to API security - rate limiting, API keys, CORS, input validation, and protecting your APIs from abuse and attacks.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "API security is defense in depth - authentication (who), authorization (what), rate limiting (how much), input validation (what data), and transport security (how) - because every exposed endpoint is an attack surface."

### API Security Layers
```
API SECURITY LAYERS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  LAYER 1: Transport Security                                    │
│  • HTTPS/TLS only                                              │
│  • Certificate validation                                       │
│  • HSTS header                                                 │
│                                                                  │
│  LAYER 2: Authentication                                        │
│  • API keys (identify client)                                  │
│  • JWT/OAuth (identify user)                                   │
│  • mTLS (mutual authentication)                                │
│                                                                  │
│  LAYER 3: Authorization                                         │
│  • Verify user can access resource                             │
│  • Scope validation (OAuth scopes)                             │
│  • Rate limiting per user/client                               │
│                                                                  │
│  LAYER 4: Input Validation                                      │
│  • Schema validation                                           │
│  • Type checking                                               │
│  • Sanitization                                                │
│                                                                  │
│  LAYER 5: Output Protection                                     │
│  • Don't leak sensitive data                                   │
│  • Proper error messages                                       │
│  • Response headers                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement
> "I implemented a comprehensive API security strategy: API keys for client identification with separate keys per environment, JWT for user authentication with 15-minute expiry, rate limiting at 100 req/min per user and 1000 req/min per API key, input validation using Zod schemas on every endpoint, and CORS configured to only allow our domains. We also implemented request signing for sensitive operations - client signs request with timestamp, server verifies within 5-minute window to prevent replay attacks. Monitoring flagged unusual patterns like sudden traffic spikes or requests from new geolocations."

---

## 📚 Core Concepts

### Rate Limiting

```typescript
// ════════════════════════════════════════════════════════════════
// RATE LIMITING IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import Redis from 'ioredis';

const redis = new Redis();

// Basic rate limiter
const basicLimiter = rateLimit({
    windowMs: 60 * 1000,  // 1 minute
    max: 100,              // 100 requests per window
    message: { error: 'Too many requests' },
    standardHeaders: true, // Return rate limit info in headers
    legacyHeaders: false,
});

// Redis-backed for distributed systems
const distributedLimiter = rateLimit({
    windowMs: 60 * 1000,
    max: 100,
    store: new RedisStore({
        sendCommand: (...args) => redis.call(...args)
    })
});

// Different limits for different endpoints
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,  // 15 minutes
    max: 5,                     // 5 login attempts
    message: { error: 'Too many login attempts' }
});

const apiLimiter = rateLimit({
    windowMs: 60 * 1000,
    max: 100
});

app.post('/api/login', authLimiter, loginHandler);
app.use('/api/', apiLimiter, apiRouter);

// ════════════════════════════════════════════════════════════════
// USER-BASED RATE LIMITING
// ════════════════════════════════════════════════════════════════

const userLimiter = rateLimit({
    windowMs: 60 * 1000,
    max: 100,
    keyGenerator: (req) => req.user?.id || req.ip,  // Per user or IP
    skip: (req) => req.user?.isPremium  // Skip for premium users
});

// Different limits by tier
const tierLimits = {
    free: 100,
    pro: 1000,
    enterprise: 10000
};

const tieredLimiter = rateLimit({
    windowMs: 60 * 1000,
    max: (req) => tierLimits[req.user?.tier || 'free'],
    keyGenerator: (req) => req.user?.id || req.ip
});
```

### API Keys

```typescript
// ════════════════════════════════════════════════════════════════
// API KEY MANAGEMENT
// ════════════════════════════════════════════════════════════════

import crypto from 'crypto';

// Generate API key
function generateApiKey(): { key: string; hash: string } {
    const key = `sk_live_${crypto.randomBytes(24).toString('hex')}`;
    const hash = crypto.createHash('sha256').update(key).digest('hex');
    return { key, hash };  // Store hash, return key once to user
}

// API key middleware
async function validateApiKey(req, res, next) {
    const apiKey = req.headers['x-api-key'];
    
    if (!apiKey) {
        return res.status(401).json({ error: 'API key required' });
    }
    
    // Hash incoming key and compare
    const hash = crypto.createHash('sha256').update(apiKey).digest('hex');
    const client = await db.apiKeys.findUnique({
        where: { hash },
        include: { permissions: true }
    });
    
    if (!client) {
        return res.status(401).json({ error: 'Invalid API key' });
    }
    
    if (client.expiresAt && client.expiresAt < new Date()) {
        return res.status(401).json({ error: 'API key expired' });
    }
    
    req.client = client;
    next();
}

// Key rotation
async function rotateApiKey(clientId: string) {
    const { key, hash } = generateApiKey();
    
    await db.apiKeys.update({
        where: { clientId },
        data: {
            hash,
            rotatedAt: new Date(),
            previousHash: existingHash,  // Keep old key valid briefly
            previousExpiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000)
        }
    });
    
    return key;
}
```

### CORS Configuration

```typescript
// ════════════════════════════════════════════════════════════════
// CORS CONFIGURATION
// ════════════════════════════════════════════════════════════════

import cors from 'cors';

// ❌ INSECURE: Allow all origins
app.use(cors());  // Don't do this in production!

// ✅ SECURE: Explicit allowed origins
const allowedOrigins = [
    'https://myapp.com',
    'https://www.myapp.com',
    'https://admin.myapp.com'
];

app.use(cors({
    origin: (origin, callback) => {
        // Allow requests with no origin (mobile apps, curl)
        if (!origin) return callback(null, true);
        
        if (allowedOrigins.includes(origin)) {
            callback(null, true);
        } else {
            callback(new Error('Not allowed by CORS'));
        }
    },
    credentials: true,  // Allow cookies
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
    allowedHeaders: ['Content-Type', 'Authorization', 'X-API-Key'],
    exposedHeaders: ['X-RateLimit-Remaining', 'X-RateLimit-Reset'],
    maxAge: 86400  // Cache preflight for 24 hours
}));

// Different CORS for different routes
app.use('/api/public', cors());  // Public API, open CORS
app.use('/api/v1', cors({ origin: allowedOrigins }));  // Private API

// ════════════════════════════════════════════════════════════════
// CORS PREFLIGHT
// ════════════════════════════════════════════════════════════════

// Browser sends OPTIONS request before actual request
// for non-simple requests (custom headers, methods other than GET/POST)

// Manual preflight handling (if not using cors middleware)
app.options('/api/*', (req, res) => {
    res.setHeader('Access-Control-Allow-Origin', req.headers.origin);
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    res.setHeader('Access-Control-Max-Age', '86400');
    res.sendStatus(204);
});
```

### Request Signing

```typescript
// ════════════════════════════════════════════════════════════════
// REQUEST SIGNING (Prevent tampering & replay attacks)
// ════════════════════════════════════════════════════════════════

import crypto from 'crypto';

// Client side: Sign request
function signRequest(apiSecret: string, method: string, path: string, body: any) {
    const timestamp = Date.now().toString();
    const bodyString = body ? JSON.stringify(body) : '';
    const message = `${timestamp}:${method}:${path}:${bodyString}`;
    
    const signature = crypto
        .createHmac('sha256', apiSecret)
        .update(message)
        .digest('hex');
    
    return {
        'X-Timestamp': timestamp,
        'X-Signature': signature
    };
}

// Server side: Verify signature
function verifySignature(req, res, next) {
    const timestamp = req.headers['x-timestamp'];
    const signature = req.headers['x-signature'];
    
    if (!timestamp || !signature) {
        return res.status(401).json({ error: 'Missing signature' });
    }
    
    // Prevent replay attacks - reject if older than 5 minutes
    const age = Date.now() - parseInt(timestamp);
    if (age > 5 * 60 * 1000) {
        return res.status(401).json({ error: 'Request expired' });
    }
    
    // Reconstruct and verify signature
    const apiSecret = req.client.secret;  // From API key lookup
    const bodyString = req.body ? JSON.stringify(req.body) : '';
    const message = `${timestamp}:${req.method}:${req.path}:${bodyString}`;
    
    const expectedSignature = crypto
        .createHmac('sha256', apiSecret)
        .update(message)
        .digest('hex');
    
    if (!crypto.timingSafeEqual(
        Buffer.from(signature),
        Buffer.from(expectedSignature)
    )) {
        return res.status(401).json({ error: 'Invalid signature' });
    }
    
    next();
}
```

### Input Validation

```typescript
// ════════════════════════════════════════════════════════════════
// INPUT VALIDATION WITH ZOD
// ════════════════════════════════════════════════════════════════

import { z } from 'zod';

// Define schemas
const createUserSchema = z.object({
    email: z.string().email().max(255),
    password: z.string().min(8).max(100),
    name: z.string().min(1).max(100).regex(/^[a-zA-Z\s]+$/),
    age: z.number().int().min(13).max(120).optional(),
    role: z.enum(['user', 'admin']).default('user')
});

// Validation middleware
function validate(schema: z.ZodSchema) {
    return (req, res, next) => {
        try {
            req.validated = schema.parse(req.body);
            next();
        } catch (err) {
            if (err instanceof z.ZodError) {
                return res.status(400).json({
                    error: 'Validation failed',
                    details: err.errors.map(e => ({
                        field: e.path.join('.'),
                        message: e.message
                    }))
                });
            }
            next(err);
        }
    };
}

// Usage
app.post('/api/users', 
    validate(createUserSchema),
    async (req, res) => {
        const userData = req.validated;  // Type-safe, validated
        // ...
    }
);

// ════════════════════════════════════════════════════════════════
// PREVENT PARAMETER POLLUTION
// ════════════════════════════════════════════════════════════════

import hpp from 'hpp';

// Prevent: GET /search?sort=name&sort=date (array injection)
app.use(hpp());

// Or manually
function sanitizeQuery(req, res, next) {
    for (const key of Object.keys(req.query)) {
        if (Array.isArray(req.query[key])) {
            req.query[key] = req.query[key][0];  // Take first value
        }
    }
    next();
}
```

---

## Security Headers for APIs

```typescript
// ════════════════════════════════════════════════════════════════
// SECURITY HEADERS
// ════════════════════════════════════════════════════════════════

import helmet from 'helmet';

app.use(helmet());

// Or manually for APIs
app.use((req, res, next) => {
    // Prevent MIME type sniffing
    res.setHeader('X-Content-Type-Options', 'nosniff');
    
    // Prevent clickjacking
    res.setHeader('X-Frame-Options', 'DENY');
    
    // XSS protection (legacy browsers)
    res.setHeader('X-XSS-Protection', '1; mode=block');
    
    // Don't cache sensitive responses
    res.setHeader('Cache-Control', 'no-store');
    res.setHeader('Pragma', 'no-cache');
    
    // Hide server info
    res.removeHeader('X-Powered-By');
    
    next();
});
```

---

## Interview Questions

**Q: "How do you secure a public API?"**
> "Defense in depth: HTTPS only, authentication (API keys or OAuth), rate limiting per key/user/IP, input validation on all endpoints, authorization checks, request size limits, proper error handling that doesn't leak info, monitoring for abuse patterns, and security headers."

**Q: "API keys vs OAuth - when to use each?"**
> "API keys identify the client application - simple, good for server-to-server. OAuth identifies the user - complex but allows delegated access with scopes. Use API keys for your own services or simple integrations. Use OAuth when third parties need to access user data with user consent."

**Q: "How do you prevent API abuse?"**
> "Layered approach: Rate limiting at multiple levels (IP, user, API key), request signing to prevent tampering, CAPTCHA for suspicious activity, IP reputation checking, anomaly detection (unusual patterns), and financial limits (charge for overages). Also regular security audits and penetration testing."

---

## Quick Reference

```
API SECURITY CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  □ HTTPS only (redirect HTTP, HSTS header)                     │
│  □ Authentication on all endpoints                             │
│  □ Authorization checks (user can access resource?)            │
│  □ Rate limiting (IP, user, API key levels)                    │
│  □ Input validation (schema, types, size limits)               │
│  □ Output sanitization (no sensitive data in errors)           │
│  □ CORS configured (explicit origins)                          │
│  □ Security headers (helmet)                                   │
│  □ Request size limits                                         │
│  □ Logging and monitoring                                      │
│  □ API versioning strategy                                     │
│  □ Deprecation policy                                          │
│                                                                  │
│  RATE LIMIT HEADERS:                                            │
│  X-RateLimit-Limit: 100                                        │
│  X-RateLimit-Remaining: 95                                     │
│  X-RateLimit-Reset: 1640000000                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


