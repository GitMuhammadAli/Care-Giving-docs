# 🔐 API Authentication - Complete Guide

> A comprehensive guide to API authentication - Bearer tokens, API keys, OAuth flows, and securing your APIs.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "API authentication is the process of verifying the identity of clients making requests to your API, typically using tokens (JWT), API keys, or OAuth, to ensure only authorized clients can access protected resources."

### The 7 Key Concepts (Remember These!)
```
1. BEARER TOKEN      → JWT in Authorization header
2. API KEY           → Static key for server-to-server
3. OAUTH 2.0         → Authorization framework with flows
4. JWT               → JSON Web Token (header.payload.signature)
5. REFRESH TOKEN     → Long-lived token to get new access tokens
6. SCOPE             → Permissions granted to token
7. TOKEN REVOCATION  → Invalidate tokens before expiry
```

### Authentication Methods Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│              AUTHENTICATION METHODS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BEARER TOKEN (JWT)                                            │
│  ──────────────────                                             │
│  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...                 │
│  ✅ Stateless (no server storage)                              │
│  ✅ Contains user info (claims)                                │
│  ✅ Works across services                                      │
│  ❌ Can't be revoked easily                                    │
│  ❌ Larger than API key                                        │
│  Use: User authentication, SPAs, mobile apps                   │
│                                                                 │
│  API KEY                                                       │
│  ───────                                                        │
│  X-API-Key: sk_live_abc123...                                  │
│  ✅ Simple to implement                                        │
│  ✅ Easy to rotate                                             │
│  ✅ Per-client tracking                                        │
│  ❌ Stateful (must store keys)                                 │
│  ❌ No user context                                            │
│  Use: Server-to-server, third-party integrations               │
│                                                                 │
│  OAUTH 2.0                                                     │
│  ─────────                                                      │
│  Multiple flows for different scenarios                        │
│  ✅ Industry standard                                          │
│  ✅ Delegated authorization                                    │
│  ✅ Scoped permissions                                         │
│  ❌ Complex to implement                                       │
│  Use: Third-party access, social login                         │
│                                                                 │
│  BASIC AUTH                                                    │
│  ──────────                                                     │
│  Authorization: Basic base64(username:password)                │
│  ✅ Simple                                                     │
│  ❌ Sends credentials every request                            │
│  ❌ Must use HTTPS                                             │
│  Use: Internal APIs, simple auth (not recommended)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### JWT Structure
```
┌─────────────────────────────────────────────────────────────────┐
│                    JWT STRUCTURE                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.                         │
│  eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4ifQ.                │
│  SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c                   │
│                                                                 │
│  ┌─────────────────┐                                           │
│  │     HEADER      │  { "alg": "HS256", "typ": "JWT" }         │
│  └─────────────────┘                                           │
│           │                                                    │
│           ▼                                                    │
│  ┌─────────────────┐  {                                        │
│  │     PAYLOAD     │    "sub": "user_123",                     │
│  │    (Claims)     │    "name": "John Doe",                    │
│  └─────────────────┘    "role": "admin",                       │
│           │             "iat": 1640000000,                     │
│           │             "exp": 1640003600                      │
│           ▼           }                                        │
│  ┌─────────────────┐                                           │
│  │   SIGNATURE     │  HMACSHA256(                              │
│  └─────────────────┘    base64(header) + "." + base64(payload),│
│                         secret                                 │
│                       )                                        │
│                                                                 │
│  STANDARD CLAIMS:                                              │
│  • sub: Subject (user ID)                                      │
│  • iat: Issued at (timestamp)                                  │
│  • exp: Expiration (timestamp)                                 │
│  • iss: Issuer                                                 │
│  • aud: Audience                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Bearer token"** | "We use Bearer tokens in the Authorization header" |
| **"Access/refresh token"** | "Short-lived access tokens with refresh rotation" |
| **"Token introspection"** | "OAuth token introspection for validation" |
| **"PKCE"** | "We use PKCE for public clients like SPAs" |
| **"Opaque vs JWT"** | "JWTs are self-contained, opaque tokens need lookup" |
| **"Token binding"** | "Consider token binding for enhanced security" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Access token expiry | **15-60 minutes** | Limit exposure |
| Refresh token expiry | **7-30 days** | User convenience |
| API key length | **32-64 chars** | Sufficient entropy |
| JWT max size | **< 8KB** | Header size limits |

### The "Wow" Statement (Memorize This!)
> "We use JWT Bearer tokens for user authentication. Access tokens are short-lived (15 minutes), containing user ID, role, and scopes. Refresh tokens (7 days) are stored securely and rotated on each use. For third-party integrations, we provide API keys with rate limits per key. OAuth 2.0 with PKCE for our mobile and SPA clients. JWTs are validated by checking signature, expiration, issuer, and audience. We maintain a token revocation list for logout and security events. API keys are hashed before storage, like passwords. All auth happens over HTTPS only."

---

## 📚 Table of Contents

1. [Bearer Tokens (JWT)](#1-bearer-tokens-jwt)
2. [API Keys](#2-api-keys)
3. [OAuth 2.0](#3-oauth-20)
4. [Token Management](#4-token-management)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Bearer Tokens (JWT)

```typescript
// ════════════════════════════════════════════════════════════════
// JWT GENERATION AND VALIDATION
// ════════════════════════════════════════════════════════════════

import jwt from 'jsonwebtoken';

interface TokenPayload {
  sub: string;        // User ID
  email: string;
  role: string;
  permissions: string[];
}

interface TokenPair {
  accessToken: string;
  refreshToken: string;
  expiresIn: number;
}

const JWT_SECRET = process.env.JWT_SECRET!;
const JWT_REFRESH_SECRET = process.env.JWT_REFRESH_SECRET!;
const ACCESS_TOKEN_EXPIRY = '15m';
const REFRESH_TOKEN_EXPIRY = '7d';

// Generate token pair
function generateTokens(user: User): TokenPair {
  const payload: TokenPayload = {
    sub: user.id,
    email: user.email,
    role: user.role,
    permissions: user.permissions,
  };

  const accessToken = jwt.sign(payload, JWT_SECRET, {
    expiresIn: ACCESS_TOKEN_EXPIRY,
    issuer: 'api.example.com',
    audience: 'example.com',
  });

  const refreshToken = jwt.sign(
    { sub: user.id, tokenFamily: crypto.randomUUID() },
    JWT_REFRESH_SECRET,
    { expiresIn: REFRESH_TOKEN_EXPIRY }
  );

  return {
    accessToken,
    refreshToken,
    expiresIn: 15 * 60, // 15 minutes in seconds
  };
}

// Validate access token
function validateAccessToken(token: string): TokenPayload {
  try {
    return jwt.verify(token, JWT_SECRET, {
      issuer: 'api.example.com',
      audience: 'example.com',
    }) as TokenPayload;
  } catch (error) {
    if (error instanceof jwt.TokenExpiredError) {
      throw new UnauthorizedError('Token expired');
    }
    if (error instanceof jwt.JsonWebTokenError) {
      throw new UnauthorizedError('Invalid token');
    }
    throw error;
  }
}

// ════════════════════════════════════════════════════════════════
// AUTH MIDDLEWARE
// ════════════════════════════════════════════════════════════════

async function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;

  if (!authHeader) {
    return res.status(401).json({
      type: 'https://api.example.com/errors/unauthorized',
      title: 'Unauthorized',
      status: 401,
      detail: 'Authorization header is required',
    });
  }

  if (!authHeader.startsWith('Bearer ')) {
    return res.status(401).json({
      type: 'https://api.example.com/errors/unauthorized',
      title: 'Unauthorized',
      status: 401,
      detail: 'Invalid authorization format. Use: Bearer <token>',
    });
  }

  const token = authHeader.substring(7);

  try {
    // Check token blacklist (for revoked tokens)
    const isRevoked = await redis.get(`revoked:${token}`);
    if (isRevoked) {
      return res.status(401).json({
        detail: 'Token has been revoked',
      });
    }

    const payload = validateAccessToken(token);
    req.user = payload;
    next();
  } catch (error) {
    return res.status(401).json({
      type: 'https://api.example.com/errors/unauthorized',
      title: 'Unauthorized',
      status: 401,
      detail: error.message,
    });
  }
}

// ════════════════════════════════════════════════════════════════
// AUTH ENDPOINTS
// ════════════════════════════════════════════════════════════════

// Login
app.post('/auth/login', async (req, res) => {
  const { email, password } = req.body;

  const user = await userService.findByEmail(email);
  if (!user || !await bcrypt.compare(password, user.passwordHash)) {
    return res.status(401).json({
      detail: 'Invalid email or password',
    });
  }

  const tokens = generateTokens(user);
  
  // Store refresh token hash
  await redis.setex(
    `refresh:${user.id}:${tokens.refreshToken.substring(0, 16)}`,
    7 * 24 * 60 * 60,  // 7 days
    tokens.refreshToken
  );

  res.json({
    access_token: tokens.accessToken,
    refresh_token: tokens.refreshToken,
    token_type: 'Bearer',
    expires_in: tokens.expiresIn,
  });
});

// Refresh token
app.post('/auth/refresh', async (req, res) => {
  const { refresh_token } = req.body;

  try {
    const payload = jwt.verify(refresh_token, JWT_REFRESH_SECRET) as {
      sub: string;
      tokenFamily: string;
    };

    // Validate refresh token exists
    const storedToken = await redis.get(
      `refresh:${payload.sub}:${refresh_token.substring(0, 16)}`
    );
    
    if (!storedToken || storedToken !== refresh_token) {
      // Possible token reuse attack - revoke all tokens for user
      await revokeAllUserTokens(payload.sub);
      return res.status(401).json({ detail: 'Invalid refresh token' });
    }

    // Get user
    const user = await userService.findById(payload.sub);
    if (!user) {
      return res.status(401).json({ detail: 'User not found' });
    }

    // Generate new tokens (rotation)
    const tokens = generateTokens(user);

    // Delete old refresh token
    await redis.del(`refresh:${payload.sub}:${refresh_token.substring(0, 16)}`);

    // Store new refresh token
    await redis.setex(
      `refresh:${user.id}:${tokens.refreshToken.substring(0, 16)}`,
      7 * 24 * 60 * 60,
      tokens.refreshToken
    );

    res.json({
      access_token: tokens.accessToken,
      refresh_token: tokens.refreshToken,
      token_type: 'Bearer',
      expires_in: tokens.expiresIn,
    });
  } catch (error) {
    return res.status(401).json({ detail: 'Invalid refresh token' });
  }
});

// Logout
app.post('/auth/logout', authMiddleware, async (req, res) => {
  const token = req.headers.authorization!.substring(7);
  const { refresh_token } = req.body;

  // Revoke access token (add to blacklist until expiry)
  const decoded = jwt.decode(token) as { exp: number };
  const ttl = decoded.exp - Math.floor(Date.now() / 1000);
  if (ttl > 0) {
    await redis.setex(`revoked:${token}`, ttl, '1');
  }

  // Delete refresh token
  if (refresh_token) {
    await redis.del(`refresh:${req.user.sub}:${refresh_token.substring(0, 16)}`);
  }

  res.status(204).send();
});
```

---

## 2. API Keys

```typescript
// ════════════════════════════════════════════════════════════════
// API KEY MANAGEMENT
// ════════════════════════════════════════════════════════════════

import crypto from 'crypto';

interface ApiKey {
  id: string;
  name: string;
  keyPrefix: string;      // First 8 chars for identification
  keyHash: string;        // Hashed full key
  userId: string;
  scopes: string[];
  rateLimit: number;      // Requests per minute
  createdAt: Date;
  lastUsedAt: Date | null;
  expiresAt: Date | null;
}

// Generate API key
function generateApiKey(): { key: string; keyPrefix: string; keyHash: string } {
  // Format: sk_live_<random> or sk_test_<random>
  const environment = process.env.NODE_ENV === 'production' ? 'live' : 'test';
  const random = crypto.randomBytes(24).toString('base64url');
  const key = `sk_${environment}_${random}`;
  
  return {
    key,
    keyPrefix: key.substring(0, 12),
    keyHash: crypto.createHash('sha256').update(key).digest('hex'),
  };
}

// Create API key for user
async function createApiKey(userId: string, name: string, scopes: string[]): Promise<{
  apiKey: ApiKey;
  secretKey: string;
}> {
  const { key, keyPrefix, keyHash } = generateApiKey();

  const apiKey = await prisma.apiKey.create({
    data: {
      name,
      keyPrefix,
      keyHash,
      userId,
      scopes,
      rateLimit: 100,
      expiresAt: null,  // Or set expiration
    },
  });

  // Return full key only once (won't be stored)
  return {
    apiKey,
    secretKey: key,  // Show to user only at creation
  };
}

// Validate API key
async function validateApiKey(key: string): Promise<ApiKey | null> {
  const keyHash = crypto.createHash('sha256').update(key).digest('hex');

  const apiKey = await prisma.apiKey.findFirst({
    where: {
      keyHash,
      OR: [
        { expiresAt: null },
        { expiresAt: { gt: new Date() } },
      ],
    },
  });

  if (apiKey) {
    // Update last used
    await prisma.apiKey.update({
      where: { id: apiKey.id },
      data: { lastUsedAt: new Date() },
    });
  }

  return apiKey;
}

// ════════════════════════════════════════════════════════════════
// API KEY MIDDLEWARE
// ════════════════════════════════════════════════════════════════

async function apiKeyMiddleware(req: Request, res: Response, next: NextFunction) {
  const apiKey = req.headers['x-api-key'] as string ||
                 req.headers['authorization']?.replace('Bearer ', '');

  if (!apiKey) {
    return res.status(401).json({
      type: 'https://api.example.com/errors/unauthorized',
      title: 'Unauthorized',
      status: 401,
      detail: 'API key is required',
    });
  }

  const validKey = await validateApiKey(apiKey);

  if (!validKey) {
    return res.status(401).json({
      type: 'https://api.example.com/errors/unauthorized',
      title: 'Unauthorized',
      status: 401,
      detail: 'Invalid API key',
    });
  }

  // Check rate limit
  const rateLimitKey = `ratelimit:${validKey.id}`;
  const currentCount = await redis.incr(rateLimitKey);
  
  if (currentCount === 1) {
    await redis.expire(rateLimitKey, 60);  // 1 minute window
  }

  if (currentCount > validKey.rateLimit) {
    return res.status(429).json({
      type: 'https://api.example.com/errors/rate-limit',
      title: 'Too Many Requests',
      status: 429,
      detail: `Rate limit exceeded (${validKey.rateLimit}/minute)`,
    });
  }

  req.apiKey = validKey;
  next();
}

// ════════════════════════════════════════════════════════════════
// SCOPE-BASED AUTHORIZATION
// ════════════════════════════════════════════════════════════════

function requireScopes(...requiredScopes: string[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    const keyScopes = req.apiKey?.scopes || [];

    const hasAllScopes = requiredScopes.every(
      scope => keyScopes.includes(scope) || keyScopes.includes('*')
    );

    if (!hasAllScopes) {
      return res.status(403).json({
        type: 'https://api.example.com/errors/forbidden',
        title: 'Forbidden',
        status: 403,
        detail: `Required scopes: ${requiredScopes.join(', ')}`,
      });
    }

    next();
  };
}

// Usage
app.get('/api/users', apiKeyMiddleware, requireScopes('users:read'), getUsers);
app.post('/api/users', apiKeyMiddleware, requireScopes('users:write'), createUser);
app.delete('/api/users/:id', apiKeyMiddleware, requireScopes('users:delete'), deleteUser);
```

---

## 3. OAuth 2.0

```typescript
// ════════════════════════════════════════════════════════════════
// OAUTH 2.0 AUTHORIZATION CODE FLOW WITH PKCE
// ════════════════════════════════════════════════════════════════

import crypto from 'crypto';

// Authorization endpoint
app.get('/oauth/authorize', async (req, res) => {
  const {
    client_id,
    redirect_uri,
    response_type,
    scope,
    state,
    code_challenge,
    code_challenge_method,
  } = req.query;

  // Validate client
  const client = await oauthClientService.findById(client_id as string);
  if (!client) {
    return res.status(400).json({ error: 'invalid_client' });
  }

  // Validate redirect URI
  if (!client.redirectUris.includes(redirect_uri as string)) {
    return res.status(400).json({ error: 'invalid_redirect_uri' });
  }

  // Validate response type
  if (response_type !== 'code') {
    return res.status(400).json({ error: 'unsupported_response_type' });
  }

  // Require PKCE for public clients
  if (client.type === 'public' && !code_challenge) {
    return res.status(400).json({ error: 'code_challenge_required' });
  }

  // If user not authenticated, redirect to login
  if (!req.user) {
    const returnUrl = encodeURIComponent(req.originalUrl);
    return res.redirect(`/login?return=${returnUrl}`);
  }

  // Show consent screen (or auto-approve for first-party apps)
  if (!client.firstParty) {
    return res.render('consent', {
      client,
      scopes: (scope as string).split(' '),
      state,
    });
  }

  // Generate authorization code
  const code = crypto.randomBytes(32).toString('base64url');
  
  // Store code with metadata (expires in 10 minutes)
  await redis.setex(`oauth:code:${code}`, 600, JSON.stringify({
    clientId: client_id,
    userId: req.user.id,
    redirectUri: redirect_uri,
    scope,
    codeChallenge: code_challenge,
    codeChallengeMethod: code_challenge_method,
  }));

  // Redirect to client with code
  const redirectUrl = new URL(redirect_uri as string);
  redirectUrl.searchParams.set('code', code);
  if (state) redirectUrl.searchParams.set('state', state as string);
  
  res.redirect(redirectUrl.toString());
});

// Token endpoint
app.post('/oauth/token', async (req, res) => {
  const { grant_type } = req.body;

  if (grant_type === 'authorization_code') {
    return handleAuthorizationCodeGrant(req, res);
  }

  if (grant_type === 'refresh_token') {
    return handleRefreshTokenGrant(req, res);
  }

  if (grant_type === 'client_credentials') {
    return handleClientCredentialsGrant(req, res);
  }

  return res.status(400).json({ error: 'unsupported_grant_type' });
});

async function handleAuthorizationCodeGrant(req: Request, res: Response) {
  const { code, redirect_uri, client_id, client_secret, code_verifier } = req.body;

  // Get stored code data
  const codeData = await redis.get(`oauth:code:${code}`);
  if (!codeData) {
    return res.status(400).json({ error: 'invalid_grant' });
  }

  const data = JSON.parse(codeData);

  // Validate client
  if (data.clientId !== client_id) {
    return res.status(400).json({ error: 'invalid_client' });
  }

  // Validate redirect URI
  if (data.redirectUri !== redirect_uri) {
    return res.status(400).json({ error: 'invalid_redirect_uri' });
  }

  // Verify PKCE
  if (data.codeChallenge) {
    const expectedChallenge = data.codeChallengeMethod === 'S256'
      ? crypto.createHash('sha256').update(code_verifier).digest('base64url')
      : code_verifier;

    if (expectedChallenge !== data.codeChallenge) {
      return res.status(400).json({ error: 'invalid_grant' });
    }
  }

  // Delete used code
  await redis.del(`oauth:code:${code}`);

  // Generate tokens
  const user = await userService.findById(data.userId);
  const tokens = generateTokens(user, data.scope.split(' '));

  res.json({
    access_token: tokens.accessToken,
    refresh_token: tokens.refreshToken,
    token_type: 'Bearer',
    expires_in: tokens.expiresIn,
    scope: data.scope,
  });
}

// ════════════════════════════════════════════════════════════════
// OAUTH CLIENT REGISTRATION
// ════════════════════════════════════════════════════════════════

interface OAuthClient {
  id: string;
  name: string;
  secret?: string;           // Hashed, only for confidential clients
  type: 'public' | 'confidential';
  redirectUris: string[];
  scopes: string[];
  firstParty: boolean;
}

async function registerOAuthClient(data: {
  name: string;
  type: 'public' | 'confidential';
  redirectUris: string[];
}): Promise<{ client: OAuthClient; secret?: string }> {
  const clientId = crypto.randomUUID();
  let secret: string | undefined;
  let secretHash: string | undefined;

  if (data.type === 'confidential') {
    secret = crypto.randomBytes(32).toString('base64url');
    secretHash = await bcrypt.hash(secret, 10);
  }

  const client = await prisma.oauthClient.create({
    data: {
      id: clientId,
      name: data.name,
      secret: secretHash,
      type: data.type,
      redirectUris: data.redirectUris,
      scopes: ['read', 'write'],
      firstParty: false,
    },
  });

  return { client, secret };
}
```

---

## 4. Token Management

```typescript
// ════════════════════════════════════════════════════════════════
// TOKEN REVOCATION
// ════════════════════════════════════════════════════════════════

// Revoke single token
async function revokeToken(token: string): Promise<void> {
  try {
    const decoded = jwt.decode(token) as { exp: number; sub: string };
    if (!decoded) return;

    const ttl = decoded.exp - Math.floor(Date.now() / 1000);
    if (ttl > 0) {
      await redis.setex(`revoked:${token}`, ttl, '1');
    }
  } catch {
    // Invalid token, nothing to revoke
  }
}

// Revoke all tokens for user (e.g., password change, security event)
async function revokeAllUserTokens(userId: string): Promise<void> {
  // Option 1: Store user token version
  await redis.incr(`token_version:${userId}`);

  // Option 2: Delete all refresh tokens
  const keys = await redis.keys(`refresh:${userId}:*`);
  if (keys.length > 0) {
    await redis.del(...keys);
  }
}

// Check if token is revoked
async function isTokenRevoked(token: string): Promise<boolean> {
  const revoked = await redis.get(`revoked:${token}`);
  return !!revoked;
}

// Token version check (include version in JWT, compare on validation)
async function validateTokenVersion(userId: string, tokenVersion: number): Promise<boolean> {
  const currentVersion = await redis.get(`token_version:${userId}`);
  return !currentVersion || parseInt(currentVersion) <= tokenVersion;
}

// ════════════════════════════════════════════════════════════════
// REFRESH TOKEN ROTATION
// ════════════════════════════════════════════════════════════════

/*
 * REFRESH TOKEN ROTATION
 * 
 * Why: If refresh token is stolen, attacker can get new access tokens
 * 
 * Solution: Issue new refresh token on each refresh
 * - Old refresh token becomes invalid
 * - If old token is used again (attacker), revoke all tokens (reuse detection)
 * 
 * Flow:
 * 1. Client uses refresh token
 * 2. Server validates token
 * 3. Server issues NEW access + refresh tokens
 * 4. Server invalidates old refresh token
 * 5. If old token is used again → token theft detected → revoke all
 */

interface RefreshTokenFamily {
  familyId: string;
  userId: string;
  currentToken: string;
  usedTokens: string[];
  createdAt: Date;
}

async function refreshWithRotation(refreshToken: string): Promise<TokenPair | null> {
  const payload = jwt.verify(refreshToken, JWT_REFRESH_SECRET) as {
    sub: string;
    familyId: string;
  };

  const familyKey = `refresh_family:${payload.familyId}`;
  const family = await redis.hgetall(familyKey);

  if (!family || !family.currentToken) {
    // Family doesn't exist
    return null;
  }

  // Check if this is the current token
  if (family.currentToken !== refreshToken) {
    // Token was already rotated - this is a reuse attack!
    console.warn(`Refresh token reuse detected for user ${payload.sub}`);
    await revokeAllUserTokens(payload.sub);
    await redis.del(familyKey);
    return null;
  }

  // Generate new tokens
  const user = await userService.findById(payload.sub);
  if (!user) return null;

  const newTokens = generateTokens(user);
  const newRefreshPayload = jwt.decode(newTokens.refreshToken) as { familyId: string };

  // Update family with new current token
  await redis.hset(familyKey, {
    currentToken: newTokens.refreshToken,
    lastRotatedAt: new Date().toISOString(),
  });

  return newTokens;
}
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# API AUTHENTICATION BEST PRACTICES
# ════════════════════════════════════════════════════════════════

token_management:
  access_tokens:
    - Short-lived (15-60 minutes)
    - Stateless (JWT) for scalability
    - Include minimal claims
    - Sign with strong algorithm (RS256 or HS256)
    
  refresh_tokens:
    - Long-lived (7-30 days)
    - Stored securely (server-side)
    - Rotate on each use
    - Revoke on logout/password change
    
  api_keys:
    - Hash before storing (like passwords)
    - Prefix for identification (sk_live_, sk_test_)
    - Per-key rate limits
    - Scoped permissions

security:
  transport:
    - HTTPS only
    - HSTS enabled
    - Secure cookie flags
    
  storage_client_side:
    - Access token: memory only
    - Refresh token: httpOnly cookie or secure storage
    - Never localStorage for sensitive tokens
    
  validation:
    - Check signature
    - Check expiration
    - Check issuer and audience
    - Check revocation

oauth:
  flows:
    - Authorization Code + PKCE for SPAs/mobile
    - Client Credentials for server-to-server
    - Never use Implicit flow
    
  security:
    - Require PKCE for public clients
    - Validate redirect URIs exactly
    - Short authorization code lifetime (10 min)
    - Bind tokens to client

monitoring:
  - Log authentication events
  - Alert on anomalies (brute force, token reuse)
  - Track API key usage
  - Audit trail for sensitive operations
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# AUTHENTICATION PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Storing tokens in localStorage
# Bad
localStorage.setItem('access_token', token);
# XSS can steal it!

# Good
# Access token in memory
# Refresh token in httpOnly cookie

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Long-lived access tokens
# Bad
const token = jwt.sign(payload, secret, { expiresIn: '30d' });
# Stolen token valid for a month!

# Good
const accessToken = jwt.sign(payload, secret, { expiresIn: '15m' });
const refreshToken = ...;  // Use refresh flow

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Not validating JWT properly
# Bad
const payload = jwt.decode(token);  // No signature validation!

# Good
const payload = jwt.verify(token, secret, {
  issuer: 'expected-issuer',
  audience: 'expected-audience',
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Storing API keys in plain text
# Bad
await db.apiKey.create({ data: { key: apiKey } });

# Good
const keyHash = crypto.createHash('sha256').update(apiKey).digest('hex');
await db.apiKey.create({ data: { keyHash } });

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: No token revocation
# Bad
# User logs out, token still valid until expiry
# No way to invalidate on password change

# Good
# Maintain revocation list (blacklist)
# Or use short-lived tokens + refresh

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Mixing authentication and authorization errors
# Bad
401 for "You don't have permission"

# Good
401: Not authenticated (no token, invalid token)
403: Not authorized (valid token, no permission)
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What's the difference between authentication and authorization?"**
> "Authentication: Who are you? (verify identity). Authorization: What can you do? (check permissions). Auth happens first (401 if failed), then authz (403 if failed). JWT can contain both: identity in claims, roles/permissions for authz."

**Q: "What is a JWT?"**
> "JSON Web Token - self-contained token with header, payload, signature. Header has algorithm. Payload has claims (sub, exp, iat, custom). Signature verifies integrity. Base64-encoded, not encrypted. Stateless - server doesn't need to store it."

**Q: "Why use short-lived access tokens with refresh tokens?"**
> "Short access tokens (15min) limit damage if stolen. Refresh tokens (7d) provide convenience without repeated login. Refresh can be revoked server-side. Rotation on each refresh detects theft. Balance security and UX."

### Intermediate Questions

**Q: "How do you handle token revocation with JWTs?"**
> "JWTs are stateless, can't be revoked directly. Options: 1) Short expiry - limit window. 2) Blacklist in Redis - check on validation. 3) Token versioning - increment on logout. 4) Refresh token revocation - can't get new access tokens."

**Q: "What is PKCE and why is it needed?"**
> "Proof Key for Code Exchange - protects OAuth authorization code flow for public clients (SPAs, mobile). Client generates code_verifier, sends code_challenge. On token exchange, proves possession of verifier. Prevents authorization code interception attacks."

**Q: "How do you secure API keys?"**
> "Hash before storage (like passwords). Prefix for identification (sk_live_). Show full key only at creation. Rate limit per key. Scope permissions. Monitor usage patterns. Allow rotation. Set expiration if applicable."

### Advanced Questions

**Q: "How do you implement refresh token rotation?"**
> "Issue new refresh token on each use, invalidate old one. Track 'token family' - tokens from same login. If old token is reused (attacker), revoke entire family. Detects theft while maintaining UX. Store token state server-side."

**Q: "How would you design multi-tenant authentication?"**
> "Options: 1) Tenant ID in JWT claims. 2) Separate token endpoints per tenant. 3) API key prefix with tenant ID. Validate tenant in every request. Consider: tenant isolation, cross-tenant access, tenant-specific secrets."

**Q: "How do you handle authentication in microservices?"**
> "Options: 1) API Gateway validates, passes user context. 2) Each service validates JWT (shared secret or public key). 3) Service mesh with mTLS. Gateway preferred for edge, internal services trust gateway-added headers."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│               AUTHENTICATION CHECKLIST                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  JWT:                                                           │
│  □ Short expiry (15-60 min)                                    │
│  □ Verify signature, exp, iss, aud                             │
│  □ Strong algorithm (RS256/HS256)                              │
│  □ Revocation strategy                                         │
│                                                                 │
│  REFRESH TOKENS:                                                │
│  □ Longer expiry (7-30 days)                                   │
│  □ Rotate on each use                                          │
│  □ Store securely (httpOnly cookie)                            │
│  □ Reuse detection                                             │
│                                                                 │
│  API KEYS:                                                      │
│  □ Hash before storage                                         │
│  □ Rate limit per key                                          │
│  □ Scoped permissions                                          │
│  □ Show full key once only                                     │
│                                                                 │
│  SECURITY:                                                      │
│  □ HTTPS only                                                  │
│  □ No tokens in localStorage                                   │
│  □ 401 vs 403 correct                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

HEADER FORMAT:
┌────────────────────────────────────────────────────────────────┐
│ Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9... │
│ X-API-Key: sk_live_abc123...                                   │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

