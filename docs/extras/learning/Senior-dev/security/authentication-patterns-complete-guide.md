# 🔐 Authentication Patterns - Complete Guide

> The definitive guide to authentication patterns - JWT vs sessions, OAuth 2.0, OIDC, SAML, refresh tokens, token storage, and implementing secure authentication from scratch.

---

## Part 1: Must Remember + Core Concepts

### 🧠 MUST REMEMBER TO IMPRESS

#### 1-Liner Definition
> "Authentication verifies WHO you are - typically through something you know (password), something you have (phone/hardware key), or something you are (biometrics). The pattern you choose (sessions vs JWT, OAuth vs SAML) depends on your architecture, but the security principles remain constant: never store plain passwords, always use HTTPS, implement proper session management, and plan for token revocation."

#### The "Wow" Statement
> "We migrated from session-based auth to JWTs for our microservices architecture. Sessions required a shared Redis cluster that became a single point of failure and added 5ms latency to every request. With JWTs, each service verifies the signature independently using a shared public key - no network call needed. We use short-lived access tokens (15 minutes) stored in memory, with refresh tokens (7 days) in httpOnly cookies. For logout, we maintain a minimal token blacklist in Redis that only tracks tokens until their natural expiry. The key insight was the hybrid approach: we get stateless scalability for reads but can still revoke tokens when needed. For our OAuth integration, we use the Authorization Code flow with PKCE even for our web app - it protects against code interception attacks that affect the implicit flow."

### Authentication Models Comparison

```
AUTHENTICATION MODELS:
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  SESSION-BASED AUTHENTICATION (Stateful)                                    │
│  ═══════════════════════════════════════                                    │
│                                                                              │
│  ┌──────────┐     1. Login            ┌──────────┐                         │
│  │  Client  │ ────────────────────────▶│  Server  │                         │
│  │          │     credentials          │          │                         │
│  │          │                          │          │                         │
│  │          │     2. Create session    │    ┌─────┴─────┐                   │
│  │          │                          │    │  Session  │                   │
│  │          │                          │    │   Store   │                   │
│  │          │                          │    │  (Redis)  │                   │
│  │          │ ◀────────────────────────│    └─────┬─────┘                   │
│  │          │     Set-Cookie: sid=abc  │          │                         │
│  │          │                          │          │                         │
│  │          │     3. Subsequent request│          │                         │
│  │          │ ────────────────────────▶│  Lookup  │                         │
│  │          │     Cookie: sid=abc      │  session │                         │
│  └──────────┘                          └──────────┘                         │
│                                                                              │
│  ✓ Easy to invalidate (delete session)                                     │
│  ✓ Server controls all session data                                        │
│  ✗ Requires shared session store for scaling                               │
│  ✗ Session lookup on every request                                         │
│                                                                              │
│  ════════════════════════════════════════════════════════════════════════   │
│                                                                              │
│  JWT-BASED AUTHENTICATION (Stateless)                                       │
│  ════════════════════════════════════                                       │
│                                                                              │
│  ┌──────────┐     1. Login            ┌──────────┐                         │
│  │  Client  │ ────────────────────────▶│  Server  │                         │
│  │          │     credentials          │          │                         │
│  │          │                          │          │                         │
│  │          │     2. Generate JWT      │          │                         │
│  │          │ ◀────────────────────────│  Sign    │                         │
│  │          │     { accessToken }      │  with    │                         │
│  │          │                          │  secret  │                         │
│  │   Store  │                          │          │                         │
│  │   token  │     3. Subsequent request│          │                         │
│  │          │ ────────────────────────▶│  Verify  │                         │
│  │          │     Authorization:       │  signature│                        │
│  │          │     Bearer <token>       │  only    │                         │
│  └──────────┘                          └──────────┘                         │
│                                                                              │
│  ✓ No server-side storage needed                                           │
│  ✓ Works across domains/services                                           │
│  ✗ Can't invalidate until expiry (without blacklist)                       │
│  ✗ Token size larger than session ID                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### OAuth 2.0 Flows

```
OAUTH 2.0 FLOWS:
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  1. AUTHORIZATION CODE FLOW (Web apps with backend)                         │
│  ══════════════════════════════════════════════════                         │
│                                                                              │
│  User    Browser    Your Server    Auth Server    Resource Server           │
│   │         │            │              │               │                   │
│   │──Click──▶│            │              │               │                  │
│   │  Login   │──Redirect─▶│              │               │                  │
│   │         │  /authorize?             │               │                   │
│   │         │  client_id=X&           │               │                   │
│   │         │  redirect_uri=Y&        │               │                   │
│   │         │  scope=Z&               │               │                   │
│   │         │  state=random           │               │                   │
│   │         │            │──────────────▶│               │                  │
│   │         │◀───Login Page────────────│               │                   │
│   │──Creds──▶│            │              │               │                  │
│   │         │────────────────────────────▶│               │                  │
│   │         │◀──Redirect with code────────│               │                  │
│   │         │  /callback?code=ABC&       │               │                   │
│   │         │  state=random             │               │                   │
│   │         │            │◀──────────────│               │                  │
│   │         │            │  Exchange    │               │                   │
│   │         │            │──code + ─────▶│               │                  │
│   │         │            │  secret      │               │                   │
│   │         │            │◀─────────────│               │                  │
│   │         │            │  access_token               │                   │
│   │         │            │  refresh_token              │                   │
│   │         │            │              │               │                   │
│   │         │            │──────────────────────────────▶│                  │
│   │         │            │  Authorization: Bearer token  │                  │
│   │         │            │◀──────────────────────────────│                  │
│   │◀────────│◀───────────│  User data                   │                  │
│                                                                              │
│  ✓ Most secure for web apps                                                │
│  ✓ Tokens never exposed to browser                                         │
│  ✓ Can use refresh tokens                                                  │
│                                                                              │
│  ════════════════════════════════════════════════════════════════════════   │
│                                                                              │
│  2. AUTHORIZATION CODE WITH PKCE (SPAs, Mobile)                            │
│  ═══════════════════════════════════════════════                            │
│                                                                              │
│  Same as above, but:                                                        │
│  • Generate code_verifier (random string)                                  │
│  • Create code_challenge = SHA256(code_verifier)                           │
│  • Send code_challenge with /authorize                                     │
│  • Send code_verifier with token exchange                                  │
│  • Server verifies SHA256(verifier) == challenge                           │
│                                                                              │
│  ✓ Protects against code interception                                      │
│  ✓ No client_secret needed                                                 │
│  ✓ RECOMMENDED for ALL clients now (RFC 7636)                              │
│                                                                              │
│  ════════════════════════════════════════════════════════════════════════   │
│                                                                              │
│  3. CLIENT CREDENTIALS (Server-to-Server)                                   │
│  ════════════════════════════════════════                                   │
│                                                                              │
│  Your Server ──client_id + client_secret──▶ Auth Server                    │
│              ◀─────── access_token ────────                                │
│                                                                              │
│  ✓ Simple for machine-to-machine                                           │
│  ✓ No user involved                                                        │
│                                                                              │
│  ════════════════════════════════════════════════════════════════════════   │
│                                                                              │
│  ❌ IMPLICIT FLOW (DEPRECATED)                                              │
│  • Token returned directly in URL fragment                                 │
│  • Vulnerable to token leakage                                             │
│  • Use PKCE instead                                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Part 2: Session-Based Authentication Implementation

### Complete Session Implementation

```typescript
// ════════════════════════════════════════════════════════════════════════════
// SESSION-BASED AUTHENTICATION - COMPLETE IMPLEMENTATION
// ════════════════════════════════════════════════════════════════════════════

import express from 'express';
import session from 'express-session';
import RedisStore from 'connect-redis';
import Redis from 'ioredis';
import crypto from 'crypto';
import argon2 from 'argon2';

const app = express();
const redis = new Redis({
    host: process.env.REDIS_HOST,
    port: 6379,
    password: process.env.REDIS_PASSWORD,
    tls: process.env.NODE_ENV === 'production' ? {} : undefined
});

// ──────────────────────────────────────────────────────────────────────────
// SESSION CONFIGURATION
// ──────────────────────────────────────────────────────────────────────────

const isProduction = process.env.NODE_ENV === 'production';

app.use(session({
    store: new RedisStore({ 
        client: redis,
        prefix: 'sess:',
        ttl: 86400  // 24 hours
    }),
    
    // Session ID generation
    genid: () => crypto.randomUUID(),
    name: 'sid',  // Cookie name (don't use default 'connect.sid')
    
    // Secret for signing session ID cookie
    secret: process.env.SESSION_SECRET!,
    
    // Session saving behavior
    resave: false,              // Don't save if unmodified
    saveUninitialized: false,   // Don't create until data exists
    rolling: true,              // Reset expiry on each request
    
    // Cookie configuration
    cookie: {
        httpOnly: true,         // JavaScript can't access
        secure: isProduction,   // HTTPS only in production
        sameSite: 'lax',        // CSRF protection
        maxAge: 24 * 60 * 60 * 1000,  // 24 hours
        domain: isProduction ? '.myapp.com' : undefined,
        path: '/'
    }
}));

// Trust proxy if behind load balancer
if (isProduction) {
    app.set('trust proxy', 1);
}

// ──────────────────────────────────────────────────────────────────────────
// TYPE DEFINITIONS
// ──────────────────────────────────────────────────────────────────────────

declare module 'express-session' {
    interface SessionData {
        userId: string;
        email: string;
        role: string;
        loginTime: number;
        lastActivity: number;
        fingerprint: string;
        mfaVerified: boolean;
    }
}

// ──────────────────────────────────────────────────────────────────────────
// LOGIN ENDPOINT
// ──────────────────────────────────────────────────────────────────────────

interface LoginRequest {
    email: string;
    password: string;
}

app.post('/api/auth/login', async (req, res) => {
    const { email, password }: LoginRequest = req.body;
    
    // Input validation
    if (!email || !password) {
        return res.status(400).json({ error: 'Email and password required' });
    }
    
    try {
        // Find user
        const user = await db.users.findUnique({ where: { email } });
        
        if (!user) {
            // Constant-time comparison to prevent timing attacks
            await argon2.hash(password);
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Check if account is locked
        if (user.lockedUntil && user.lockedUntil > new Date()) {
            const waitMinutes = Math.ceil((user.lockedUntil.getTime() - Date.now()) / 60000);
            return res.status(423).json({ 
                error: `Account locked. Try again in ${waitMinutes} minutes` 
            });
        }
        
        // Verify password
        const validPassword = await argon2.verify(user.passwordHash, password);
        
        if (!validPassword) {
            // Increment failed attempts
            const attempts = (user.failedLoginAttempts || 0) + 1;
            const lockAccount = attempts >= 5;
            
            await db.users.update({
                where: { id: user.id },
                data: {
                    failedLoginAttempts: attempts,
                    lockedUntil: lockAccount 
                        ? new Date(Date.now() + 30 * 60 * 1000)  // 30 min lock
                        : null
                }
            });
            
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Reset failed attempts on successful login
        await db.users.update({
            where: { id: user.id },
            data: {
                failedLoginAttempts: 0,
                lockedUntil: null,
                lastLoginAt: new Date()
            }
        });
        
        // Check if MFA is enabled
        if (user.mfaEnabled) {
            // Create pending MFA session
            req.session.userId = user.id;
            req.session.mfaVerified = false;
            
            return res.json({
                status: 'mfa_required',
                mfaMethods: user.mfaMethods  // ['totp', 'backup']
            });
        }
        
        // CRITICAL: Regenerate session ID to prevent session fixation
        req.session.regenerate((err) => {
            if (err) {
                console.error('Session regeneration failed:', err);
                return res.status(500).json({ error: 'Authentication failed' });
            }
            
            // Set session data
            req.session.userId = user.id;
            req.session.email = user.email;
            req.session.role = user.role;
            req.session.loginTime = Date.now();
            req.session.lastActivity = Date.now();
            req.session.fingerprint = createFingerprint(req);
            req.session.mfaVerified = true;
            
            // Save and respond
            req.session.save((err) => {
                if (err) {
                    console.error('Session save failed:', err);
                    return res.status(500).json({ error: 'Authentication failed' });
                }
                
                res.json({
                    status: 'success',
                    user: {
                        id: user.id,
                        email: user.email,
                        name: user.name,
                        role: user.role
                    }
                });
            });
        });
        
    } catch (error) {
        console.error('Login error:', error);
        res.status(500).json({ error: 'Authentication failed' });
    }
});

// ──────────────────────────────────────────────────────────────────────────
// FINGERPRINT CREATION (Session Binding)
// ──────────────────────────────────────────────────────────────────────────

function createFingerprint(req: express.Request): string {
    const components = [
        req.ip,
        req.headers['user-agent'] || '',
        req.headers['accept-language'] || ''
    ];
    
    return crypto
        .createHash('sha256')
        .update(components.join('|'))
        .digest('hex')
        .substring(0, 32);
}

// ──────────────────────────────────────────────────────────────────────────
// AUTHENTICATION MIDDLEWARE
// ──────────────────────────────────────────────────────────────────────────

interface AuthenticatedRequest extends express.Request {
    user: {
        id: string;
        email: string;
        role: string;
    };
}

const authenticate = (
    req: express.Request, 
    res: express.Response, 
    next: express.NextFunction
) => {
    // Check session exists
    if (!req.session.userId) {
        return res.status(401).json({ error: 'Authentication required' });
    }
    
    // Check MFA verification
    if (!req.session.mfaVerified) {
        return res.status(401).json({ 
            error: 'MFA verification required',
            code: 'MFA_REQUIRED'
        });
    }
    
    // Validate fingerprint (optional - can cause issues with dynamic IPs)
    const currentFingerprint = createFingerprint(req);
    if (req.session.fingerprint !== currentFingerprint) {
        console.warn('Fingerprint mismatch', {
            sessionId: req.sessionID,
            userId: req.session.userId,
            expected: req.session.fingerprint,
            actual: currentFingerprint
        });
        
        // Destroy suspicious session
        return req.session.destroy(() => {
            res.status(401).json({ 
                error: 'Session expired',
                code: 'FINGERPRINT_MISMATCH'
            });
        });
    }
    
    // Check inactivity timeout (30 minutes)
    const inactiveTime = Date.now() - (req.session.lastActivity || 0);
    if (inactiveTime > 30 * 60 * 1000) {
        return req.session.destroy(() => {
            res.status(401).json({ 
                error: 'Session expired due to inactivity',
                code: 'INACTIVE_TIMEOUT'
            });
        });
    }
    
    // Check absolute timeout (24 hours from login)
    const sessionAge = Date.now() - (req.session.loginTime || 0);
    if (sessionAge > 24 * 60 * 60 * 1000) {
        return req.session.destroy(() => {
            res.status(401).json({ 
                error: 'Session expired',
                code: 'ABSOLUTE_TIMEOUT'
            });
        });
    }
    
    // Update last activity
    req.session.lastActivity = Date.now();
    
    // Attach user to request
    (req as AuthenticatedRequest).user = {
        id: req.session.userId,
        email: req.session.email!,
        role: req.session.role!
    };
    
    next();
};

// ──────────────────────────────────────────────────────────────────────────
// LOGOUT ENDPOINT
// ──────────────────────────────────────────────────────────────────────────

app.post('/api/auth/logout', authenticate, (req, res) => {
    const userId = req.session.userId;
    const sessionId = req.sessionID;
    
    req.session.destroy((err) => {
        if (err) {
            console.error('Logout error:', err);
        }
        
        // Clear cookie
        res.clearCookie('sid', {
            httpOnly: true,
            secure: isProduction,
            sameSite: 'lax',
            path: '/'
        });
        
        // Audit log
        console.log('User logged out', { userId, sessionId });
        
        res.json({ message: 'Logged out successfully' });
    });
});

// ──────────────────────────────────────────────────────────────────────────
// LOGOUT ALL SESSIONS
// ──────────────────────────────────────────────────────────────────────────

app.post('/api/auth/logout-all', authenticate, async (req, res) => {
    const userId = (req as AuthenticatedRequest).user.id;
    
    // Method 1: Delete all session keys for user
    // Requires tracking session IDs per user
    const sessionKeys = await redis.keys(`sess:*`);
    
    for (const key of sessionKeys) {
        const sessionData = await redis.get(key);
        if (sessionData) {
            try {
                const parsed = JSON.parse(sessionData);
                if (parsed.userId === userId) {
                    await redis.del(key);
                }
            } catch {}
        }
    }
    
    // Method 2: Increment session version (more scalable)
    await db.users.update({
        where: { id: userId },
        data: { sessionVersion: { increment: 1 } }
    });
    
    res.json({ message: 'All sessions invalidated' });
});
```

---

## Part 3: JWT-Based Authentication Implementation

### Complete JWT Implementation

```typescript
// ════════════════════════════════════════════════════════════════════════════
// JWT-BASED AUTHENTICATION - COMPLETE IMPLEMENTATION
// ════════════════════════════════════════════════════════════════════════════

import jwt, { JwtPayload, SignOptions } from 'jsonwebtoken';
import crypto from 'crypto';

// ──────────────────────────────────────────────────────────────────────────
// CONFIGURATION
// ──────────────────────────────────────────────────────────────────────────

const JWT_CONFIG = {
    accessToken: {
        secret: process.env.JWT_ACCESS_SECRET!,
        expiresIn: '15m'
    },
    refreshToken: {
        secret: process.env.JWT_REFRESH_SECRET!,
        expiresIn: '7d'
    },
    issuer: 'myapp',
    audience: 'myapp-users'
};

interface TokenPayload extends JwtPayload {
    sub: string;      // User ID
    email: string;
    role: string;
    type: 'access' | 'refresh';
    jti?: string;     // JWT ID for refresh tokens
}

// ──────────────────────────────────────────────────────────────────────────
// TOKEN GENERATION
// ──────────────────────────────────────────────────────────────────────────

function generateAccessToken(user: User): string {
    const payload: Omit<TokenPayload, 'iat' | 'exp'> = {
        sub: user.id,
        email: user.email,
        role: user.role,
        type: 'access'
    };
    
    return jwt.sign(payload, JWT_CONFIG.accessToken.secret, {
        expiresIn: JWT_CONFIG.accessToken.expiresIn,
        issuer: JWT_CONFIG.issuer,
        audience: JWT_CONFIG.audience
    });
}

function generateRefreshToken(user: User): { token: string; jti: string } {
    const jti = crypto.randomUUID();  // Unique ID for this refresh token
    
    const payload: Omit<TokenPayload, 'iat' | 'exp'> = {
        sub: user.id,
        email: user.email,
        role: user.role,
        type: 'refresh',
        jti
    };
    
    const token = jwt.sign(payload, JWT_CONFIG.refreshToken.secret, {
        expiresIn: JWT_CONFIG.refreshToken.expiresIn,
        issuer: JWT_CONFIG.issuer,
        audience: JWT_CONFIG.audience
    });
    
    return { token, jti };
}

interface TokenPair {
    accessToken: string;
    refreshToken: string;
    expiresIn: number;
}

async function generateTokenPair(user: User): Promise<TokenPair> {
    const accessToken = generateAccessToken(user);
    const { token: refreshToken, jti } = generateRefreshToken(user);
    
    // Store refresh token info in database for revocation
    await db.refreshTokens.create({
        data: {
            id: jti,
            userId: user.id,
            expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
            userAgent: '', // Can store device info
            ipAddress: ''
        }
    });
    
    return {
        accessToken,
        refreshToken,
        expiresIn: 15 * 60  // 15 minutes in seconds
    };
}

// ──────────────────────────────────────────────────────────────────────────
// TOKEN VERIFICATION
// ──────────────────────────────────────────────────────────────────────────

function verifyAccessToken(token: string): TokenPayload {
    try {
        const payload = jwt.verify(token, JWT_CONFIG.accessToken.secret, {
            algorithms: ['HS256'],  // Explicit algorithm
            issuer: JWT_CONFIG.issuer,
            audience: JWT_CONFIG.audience
        }) as TokenPayload;
        
        if (payload.type !== 'access') {
            throw new Error('Invalid token type');
        }
        
        return payload;
    } catch (error) {
        if (error instanceof jwt.TokenExpiredError) {
            throw new AuthError('Token expired', 'TOKEN_EXPIRED');
        }
        if (error instanceof jwt.JsonWebTokenError) {
            throw new AuthError('Invalid token', 'INVALID_TOKEN');
        }
        throw error;
    }
}

async function verifyRefreshToken(token: string): Promise<TokenPayload> {
    const payload = jwt.verify(token, JWT_CONFIG.refreshToken.secret, {
        algorithms: ['HS256'],
        issuer: JWT_CONFIG.issuer,
        audience: JWT_CONFIG.audience
    }) as TokenPayload;
    
    if (payload.type !== 'refresh') {
        throw new AuthError('Invalid token type', 'INVALID_TOKEN');
    }
    
    // Check if refresh token is still valid in database
    const storedToken = await db.refreshTokens.findUnique({
        where: { id: payload.jti }
    });
    
    if (!storedToken) {
        throw new AuthError('Token revoked', 'TOKEN_REVOKED');
    }
    
    if (storedToken.revokedAt) {
        // Token was revoked - possible token theft!
        // Revoke all tokens for this user
        await revokeAllUserTokens(payload.sub);
        throw new AuthError('Token revoked - all sessions invalidated', 'TOKEN_REVOKED');
    }
    
    return payload;
}

// ──────────────────────────────────────────────────────────────────────────
// LOGIN ENDPOINT (JWT)
// ──────────────────────────────────────────────────────────────────────────

app.post('/api/auth/login', async (req, res) => {
    const { email, password } = req.body;
    
    // ... validation and password verification (same as session-based) ...
    
    const user = await verifyCredentials(email, password);
    if (!user) {
        return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Check MFA
    if (user.mfaEnabled) {
        // Return temporary token for MFA flow
        const mfaToken = jwt.sign(
            { sub: user.id, type: 'mfa_pending' },
            JWT_CONFIG.accessToken.secret,
            { expiresIn: '5m' }
        );
        
        return res.json({
            status: 'mfa_required',
            mfaToken,
            mfaMethods: user.mfaMethods
        });
    }
    
    // Generate tokens
    const tokens = await generateTokenPair(user);
    
    // Set refresh token in httpOnly cookie
    res.cookie('refreshToken', tokens.refreshToken, {
        httpOnly: true,
        secure: process.env.NODE_ENV === 'production',
        sameSite: 'strict',
        maxAge: 7 * 24 * 60 * 60 * 1000,  // 7 days
        path: '/api/auth'  // Only sent to auth endpoints
    });
    
    // Return access token in response (frontend stores in memory)
    res.json({
        status: 'success',
        accessToken: tokens.accessToken,
        expiresIn: tokens.expiresIn,
        user: {
            id: user.id,
            email: user.email,
            name: user.name,
            role: user.role
        }
    });
});

// ──────────────────────────────────────────────────────────────────────────
// TOKEN REFRESH ENDPOINT
// ──────────────────────────────────────────────────────────────────────────

app.post('/api/auth/refresh', async (req, res) => {
    const refreshToken = req.cookies.refreshToken;
    
    if (!refreshToken) {
        return res.status(401).json({ error: 'Refresh token required' });
    }
    
    try {
        // Verify refresh token
        const payload = await verifyRefreshToken(refreshToken);
        
        // Get fresh user data
        const user = await db.users.findUnique({ 
            where: { id: payload.sub }
        });
        
        if (!user || user.disabled) {
            throw new AuthError('User not found or disabled', 'USER_INVALID');
        }
        
        // Rotate refresh token (issue new one, invalidate old)
        const oldJti = payload.jti!;
        
        // Invalidate old refresh token
        await db.refreshTokens.update({
            where: { id: oldJti },
            data: { revokedAt: new Date() }
        });
        
        // Generate new token pair
        const tokens = await generateTokenPair(user);
        
        // Set new refresh token
        res.cookie('refreshToken', tokens.refreshToken, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            maxAge: 7 * 24 * 60 * 60 * 1000,
            path: '/api/auth'
        });
        
        res.json({
            accessToken: tokens.accessToken,
            expiresIn: tokens.expiresIn
        });
        
    } catch (error) {
        // Clear invalid refresh token
        res.clearCookie('refreshToken', { path: '/api/auth' });
        
        if (error instanceof AuthError) {
            return res.status(401).json({ 
                error: error.message,
                code: error.code
            });
        }
        
        return res.status(401).json({ error: 'Invalid refresh token' });
    }
});

// ──────────────────────────────────────────────────────────────────────────
// JWT AUTHENTICATION MIDDLEWARE
// ──────────────────────────────────────────────────────────────────────────

const authenticateJWT = async (
    req: express.Request,
    res: express.Response,
    next: express.NextFunction
) => {
    const authHeader = req.headers.authorization;
    
    if (!authHeader?.startsWith('Bearer ')) {
        return res.status(401).json({ error: 'Authorization header required' });
    }
    
    const token = authHeader.split(' ')[1];
    
    try {
        const payload = verifyAccessToken(token);
        
        // Optional: Check blacklist for revoked tokens
        const isBlacklisted = await redis.get(`blacklist:${token}`);
        if (isBlacklisted) {
            return res.status(401).json({ error: 'Token revoked' });
        }
        
        // Attach user to request
        (req as AuthenticatedRequest).user = {
            id: payload.sub,
            email: payload.email,
            role: payload.role
        };
        
        next();
    } catch (error) {
        if (error instanceof AuthError) {
            return res.status(401).json({
                error: error.message,
                code: error.code
            });
        }
        return res.status(401).json({ error: 'Invalid token' });
    }
};

// ──────────────────────────────────────────────────────────────────────────
// LOGOUT (JWT)
// ──────────────────────────────────────────────────────────────────────────

app.post('/api/auth/logout', authenticateJWT, async (req, res) => {
    const refreshToken = req.cookies.refreshToken;
    
    // Revoke refresh token
    if (refreshToken) {
        try {
            const payload = jwt.decode(refreshToken) as TokenPayload;
            if (payload?.jti) {
                await db.refreshTokens.update({
                    where: { id: payload.jti },
                    data: { revokedAt: new Date() }
                });
            }
        } catch {}
    }
    
    // Blacklist current access token until it expires
    const authHeader = req.headers.authorization;
    const accessToken = authHeader?.split(' ')[1];
    
    if (accessToken) {
        const payload = jwt.decode(accessToken) as TokenPayload;
        if (payload?.exp) {
            const ttl = payload.exp - Math.floor(Date.now() / 1000);
            if (ttl > 0) {
                await redis.setex(`blacklist:${accessToken}`, ttl, '1');
            }
        }
    }
    
    // Clear refresh token cookie
    res.clearCookie('refreshToken', { path: '/api/auth' });
    
    res.json({ message: 'Logged out successfully' });
});

// ──────────────────────────────────────────────────────────────────────────
// REVOKE ALL USER TOKENS
// ──────────────────────────────────────────────────────────────────────────

async function revokeAllUserTokens(userId: string): Promise<void> {
    // Revoke all refresh tokens
    await db.refreshTokens.updateMany({
        where: { 
            userId,
            revokedAt: null
        },
        data: { revokedAt: new Date() }
    });
    
    // Increment token version (invalidates all access tokens)
    await db.users.update({
        where: { id: userId },
        data: { tokenVersion: { increment: 1 } }
    });
}
```

---

## Part 4: OAuth 2.0 / OpenID Connect Implementation

### OAuth with Passport.js

```typescript
// ════════════════════════════════════════════════════════════════════════════
// OAUTH 2.0 / OIDC IMPLEMENTATION
// ════════════════════════════════════════════════════════════════════════════

import passport from 'passport';
import { Strategy as GoogleStrategy } from 'passport-google-oauth20';
import { Strategy as GitHubStrategy } from 'passport-github2';

// ──────────────────────────────────────────────────────────────────────────
// GOOGLE OAUTH STRATEGY
// ──────────────────────────────────────────────────────────────────────────

passport.use(new GoogleStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID!,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    callbackURL: `${process.env.APP_URL}/api/auth/google/callback`,
    scope: ['profile', 'email']
}, async (accessToken, refreshToken, profile, done) => {
    try {
        // Find or create user
        let user = await db.users.findFirst({
            where: {
                OR: [
                    { googleId: profile.id },
                    { email: profile.emails?.[0]?.value }
                ]
            }
        });
        
        if (user) {
            // Link Google account if not already linked
            if (!user.googleId) {
                user = await db.users.update({
                    where: { id: user.id },
                    data: { 
                        googleId: profile.id,
                        emailVerified: true  // Google verified the email
                    }
                });
            }
        } else {
            // Create new user
            user = await db.users.create({
                data: {
                    googleId: profile.id,
                    email: profile.emails?.[0]?.value!,
                    name: profile.displayName,
                    avatar: profile.photos?.[0]?.value,
                    emailVerified: true,
                    authProvider: 'google'
                }
            });
        }
        
        done(null, user);
    } catch (error) {
        done(error as Error);
    }
}));

// ──────────────────────────────────────────────────────────────────────────
// GITHUB OAUTH STRATEGY
// ──────────────────────────────────────────────────────────────────────────

passport.use(new GitHubStrategy({
    clientID: process.env.GITHUB_CLIENT_ID!,
    clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    callbackURL: `${process.env.APP_URL}/api/auth/github/callback`,
    scope: ['user:email']
}, async (accessToken, refreshToken, profile, done) => {
    try {
        // GitHub may not return email in profile, need separate API call
        let email = profile.emails?.[0]?.value;
        
        if (!email) {
            // Fetch email from GitHub API
            const response = await fetch('https://api.github.com/user/emails', {
                headers: { Authorization: `Bearer ${accessToken}` }
            });
            const emails = await response.json();
            email = emails.find((e: any) => e.primary)?.email;
        }
        
        let user = await db.users.findFirst({
            where: {
                OR: [
                    { githubId: profile.id },
                    { email }
                ]
            }
        });
        
        if (user) {
            if (!user.githubId) {
                user = await db.users.update({
                    where: { id: user.id },
                    data: { githubId: profile.id }
                });
            }
        } else {
            user = await db.users.create({
                data: {
                    githubId: profile.id,
                    email: email!,
                    name: profile.displayName || profile.username,
                    avatar: profile.photos?.[0]?.value,
                    authProvider: 'github'
                }
            });
        }
        
        done(null, user);
    } catch (error) {
        done(error as Error);
    }
}));

// ──────────────────────────────────────────────────────────────────────────
// OAUTH ROUTES
// ──────────────────────────────────────────────────────────────────────────

// Initiate Google OAuth
app.get('/api/auth/google', (req, res, next) => {
    // Store return URL in session for post-auth redirect
    const returnTo = req.query.returnTo as string;
    if (returnTo) {
        req.session.returnTo = returnTo;
    }
    
    passport.authenticate('google', {
        scope: ['profile', 'email'],
        prompt: 'select_account'  // Always show account picker
    })(req, res, next);
});

// Google OAuth callback
app.get('/api/auth/google/callback',
    passport.authenticate('google', { 
        failureRedirect: '/login?error=oauth_failed',
        session: false  // We'll handle session/JWT ourselves
    }),
    async (req, res) => {
        const user = req.user as User;
        
        // Generate our own tokens
        const tokens = await generateTokenPair(user);
        
        // Set refresh token cookie
        res.cookie('refreshToken', tokens.refreshToken, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'lax',  // Lax for OAuth redirect
            maxAge: 7 * 24 * 60 * 60 * 1000,
            path: '/api/auth'
        });
        
        // Redirect to frontend with access token
        const returnTo = req.session.returnTo || '/dashboard';
        delete req.session.returnTo;
        
        // Option 1: Redirect with token in URL (less secure but simpler)
        res.redirect(`${returnTo}?token=${tokens.accessToken}`);
        
        // Option 2: Redirect to page that reads token from cookie
        // res.redirect(`/auth/callback?returnTo=${encodeURIComponent(returnTo)}`);
    }
);

// ──────────────────────────────────────────────────────────────────────────
// PKCE IMPLEMENTATION FOR SPA
// ──────────────────────────────────────────────────────────────────────────

// Generate PKCE challenge
app.get('/api/auth/pkce/challenge', (req, res) => {
    // Generate code verifier (random string)
    const codeVerifier = crypto.randomBytes(32)
        .toString('base64url');
    
    // Generate code challenge (SHA256 hash of verifier)
    const codeChallenge = crypto
        .createHash('sha256')
        .update(codeVerifier)
        .digest('base64url');
    
    // Store verifier in session (or send to client to store)
    req.session.codeVerifier = codeVerifier;
    
    res.json({
        codeChallenge,
        codeChallengeMethod: 'S256'
    });
});

// OAuth callback with PKCE verification
app.post('/api/auth/oauth/callback', async (req, res) => {
    const { code, codeVerifier, provider } = req.body;
    
    // Verify code verifier matches challenge
    const storedVerifier = req.session.codeVerifier;
    if (!storedVerifier || storedVerifier !== codeVerifier) {
        return res.status(400).json({ error: 'Invalid PKCE verifier' });
    }
    
    // Exchange code for tokens with provider
    // ... provider-specific token exchange
});
```

---

## Part 5: Security Best Practices & Common Mistakes

### Token Storage Security

```typescript
// ════════════════════════════════════════════════════════════════════════════
// TOKEN STORAGE SECURITY
// ════════════════════════════════════════════════════════════════════════════

/*
TOKEN STORAGE OPTIONS:
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  STORAGE LOCATION       XSS RISK    CSRF RISK    RECOMMENDATION            │
│  ═══════════════════════════════════════════════════════════════════════   │
│                                                                              │
│  localStorage           HIGH        NONE         ❌ Never for tokens        │
│  • JavaScript can read                           • XSS steals tokens        │
│  • Persists indefinitely                                                    │
│                                                                              │
│  sessionStorage         HIGH        NONE         ❌ Never for tokens        │
│  • JavaScript can read                           • XSS steals tokens        │
│  • Cleared on tab close                                                     │
│                                                                              │
│  Cookie (httpOnly)      NONE        HIGH         ⚠️ With CSRF protection    │
│  • JS cannot read                                • Use SameSite=Strict      │
│  • Sent automatically                            • Good for refresh token   │
│                                                                              │
│  Memory (JS variable)   LOW         NONE         ✅ Best for access token   │
│  • Lost on refresh                               • Combine with httpOnly    │
│  • XSS can't persist                             • refresh token cookie     │
│                                                                              │
│  ═══════════════════════════════════════════════════════════════════════   │
│                                                                              │
│  RECOMMENDED PATTERN:                                                        │
│  • Access token: In-memory (JS variable)                                    │
│  • Refresh token: httpOnly cookie (SameSite=Strict, path=/api/auth)        │
│  • On page load: Call /refresh to get new access token                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
*/

// ──────────────────────────────────────────────────────────────────────────
// FRONTEND TOKEN MANAGEMENT (React Example)
// ──────────────────────────────────────────────────────────────────────────

// auth-context.tsx
import React, { createContext, useContext, useEffect, useState, useCallback } from 'react';

interface AuthState {
    accessToken: string | null;
    user: User | null;
    isLoading: boolean;
}

const AuthContext = createContext<AuthState | null>(null);

export function AuthProvider({ children }: { children: React.ReactNode }) {
    const [state, setState] = useState<AuthState>({
        accessToken: null,
        user: null,
        isLoading: true
    });
    
    // Refresh token on mount and before expiry
    const refreshAccessToken = useCallback(async () => {
        try {
            const response = await fetch('/api/auth/refresh', {
                method: 'POST',
                credentials: 'include'  // Send cookies
            });
            
            if (response.ok) {
                const { accessToken, user, expiresIn } = await response.json();
                
                setState({
                    accessToken,
                    user,
                    isLoading: false
                });
                
                // Schedule next refresh before expiry
                const refreshIn = (expiresIn - 60) * 1000;  // 1 minute before
                setTimeout(refreshAccessToken, refreshIn);
                
                return accessToken;
            } else {
                // Refresh failed - user needs to login
                setState({ accessToken: null, user: null, isLoading: false });
                return null;
            }
        } catch (error) {
            setState({ accessToken: null, user: null, isLoading: false });
            return null;
        }
    }, []);
    
    // Initial refresh on mount
    useEffect(() => {
        refreshAccessToken();
    }, [refreshAccessToken]);
    
    return (
        <AuthContext.Provider value={state}>
            {children}
        </AuthContext.Provider>
    );
}

// Axios interceptor for authenticated requests
import axios from 'axios';

const api = axios.create({
    baseURL: '/api'
});

api.interceptors.request.use(async (config) => {
    const { accessToken } = useAuth();  // Get from context
    
    if (accessToken) {
        config.headers.Authorization = `Bearer ${accessToken}`;
    }
    
    return config;
});

api.interceptors.response.use(
    (response) => response,
    async (error) => {
        if (error.response?.status === 401) {
            // Try to refresh token
            const newToken = await refreshAccessToken();
            
            if (newToken) {
                // Retry original request
                error.config.headers.Authorization = `Bearer ${newToken}`;
                return api.request(error.config);
            }
        }
        return Promise.reject(error);
    }
);
```

### Common Authentication Mistakes

```typescript
// ════════════════════════════════════════════════════════════════════════════
// COMMON AUTHENTICATION MISTAKES
// ════════════════════════════════════════════════════════════════════════════

// ──────────────────────────────────────────────────────────────────────────
// MISTAKE 1: Storing Tokens in localStorage
// ──────────────────────────────────────────────────────────────────────────

// ❌ WRONG
localStorage.setItem('accessToken', token);
// Any XSS vulnerability exposes the token permanently

// ✅ CORRECT
// Store access token in memory variable
let accessToken = response.accessToken;
// Store refresh token in httpOnly cookie (server sets it)

// ──────────────────────────────────────────────────────────────────────────
// MISTAKE 2: Long-Lived Access Tokens
// ──────────────────────────────────────────────────────────────────────────

// ❌ WRONG
jwt.sign(payload, secret, { expiresIn: '30d' });
// If token is stolen, attacker has access for 30 days

// ✅ CORRECT
jwt.sign(payload, secret, { expiresIn: '15m' });
// Short expiry limits damage, use refresh tokens for persistence

// ──────────────────────────────────────────────────────────────────────────
// MISTAKE 3: Not Validating JWT Algorithm
// ──────────────────────────────────────────────────────────────────────────

// ❌ WRONG
jwt.verify(token, secret);  // Accepts any algorithm!
// Attacker can change alg to 'none' or switch HS256 to RS256

// ✅ CORRECT
jwt.verify(token, secret, { algorithms: ['HS256'] });

// ──────────────────────────────────────────────────────────────────────────
// MISTAKE 4: Sensitive Data in JWT Payload
// ──────────────────────────────────────────────────────────────────────────

// ❌ WRONG
jwt.sign({
    userId: '123',
    email: 'user@example.com',
    passwordHash: 'abc...',  // NEVER!
    ssn: '123-45-6789'       // NEVER!
}, secret);
// JWT payload is just base64, anyone can decode it

// ✅ CORRECT
jwt.sign({
    sub: '123',    // User ID only
    role: 'user'   // Minimal claims
}, secret);
// Fetch sensitive data server-side when needed

// ──────────────────────────────────────────────────────────────────────────
// MISTAKE 5: No Session Regeneration
// ──────────────────────────────────────────────────────────────────────────

// ❌ WRONG - Session fixation vulnerability
app.post('/login', (req, res) => {
    req.session.userId = user.id;  // Using existing session ID!
});

// ✅ CORRECT
app.post('/login', (req, res) => {
    req.session.regenerate(() => {
        req.session.userId = user.id;  // New session ID
    });
});

// ──────────────────────────────────────────────────────────────────────────
// MISTAKE 6: Timing Attacks in Password Comparison
// ──────────────────────────────────────────────────────────────────────────

// ❌ WRONG
if (user.password === inputPassword) {
    // String comparison exits early on mismatch
    // Attacker can deduce characters from timing
}

// ✅ CORRECT
await argon2.verify(user.passwordHash, inputPassword);
// Constant-time comparison

// ──────────────────────────────────────────────────────────────────────────
// MISTAKE 7: Information Leakage in Error Messages
// ──────────────────────────────────────────────────────────────────────────

// ❌ WRONG
if (!user) {
    return res.status(401).json({ error: 'User not found' });
}
if (!validPassword) {
    return res.status(401).json({ error: 'Wrong password' });
}
// Attacker can enumerate valid emails

// ✅ CORRECT
if (!user || !await argon2.verify(user.passwordHash, password)) {
    // Always hash even if user not found (constant time)
    if (!user) await argon2.hash(password);
    return res.status(401).json({ error: 'Invalid credentials' });
}
```

---

## Part 6: Comprehensive Interview Q&A

### Fundamental Questions

**Q1: "Explain the difference between sessions and JWTs"**

> "Sessions are stateful - a session ID stored in a cookie maps to session data stored on the server (typically Redis). The server looks up the session on every request. JWTs are stateless - the token itself contains claims (user ID, role, expiry) signed by the server. The server verifies the signature without any lookup.

> Sessions are easier to revoke (delete the session), but require shared storage for scaling. JWTs scale better (no shared state) but can't be truly revoked until expiry. In practice, I use a hybrid: short-lived JWTs (15 min) for API auth, with refresh tokens stored in httpOnly cookies. For logout, I maintain a minimal blacklist in Redis that only tracks tokens until their natural expiry."

**Q2: "How would you implement secure authentication for a SPA?"**

> "For a single-page application: Use the Authorization Code flow with PKCE - even without a backend secret, PKCE prevents code interception attacks. Store the access token in memory only (JavaScript variable) - never localStorage. Store the refresh token in an httpOnly, Secure, SameSite=Strict cookie so JavaScript can't access it.

> On page load, silently call the refresh endpoint to get a new access token. Set up an Axios/fetch interceptor to add the token to requests and handle 401s by refreshing. Schedule token refresh before expiry. On logout, call the server to revoke the refresh token and clear it from cookies."

**Q3: "What is OAuth 2.0 and when would you use it?"**

> "OAuth 2.0 is an authorization framework for delegated access - it lets users grant third-party apps limited access to their resources without sharing credentials. The key flows are: Authorization Code (web apps with backend), Authorization Code with PKCE (SPAs, mobile), and Client Credentials (server-to-server).

> Use OAuth when: you want 'Login with Google/GitHub' (user convenience, no password to manage), when building a platform where third parties access user data (like Slack apps), or for machine-to-machine API access. OpenID Connect (OIDC) extends OAuth for authentication, providing standardized user identity claims."

### Advanced Questions

**Q4: "How do you handle token revocation in a stateless JWT system?"**

> "JWTs are inherently unrevocable until expiry - that's the trade-off for statelessness. Strategies to handle this:

> 1. **Short expiry + refresh tokens**: Access tokens expire in 15 minutes. Even if stolen, limited window. Refresh tokens can be revoked in database.

> 2. **Token blacklist**: Store revoked tokens in Redis with TTL matching token expiry. Check blacklist on each request. Adds a lookup but still faster than session lookup.

> 3. **Token versioning**: Store a version number with the user. Include version in JWT claims. Increment version to invalidate all tokens. Check version on each request.

> 4. **Refresh token rotation**: Issue new refresh token on each use, invalidate old one. If old token is used, all tokens are compromised - revoke everything.

> I typically use approach 1 + 2 for logout, and approach 4 to detect token theft."

**Q5: "Explain the OAuth Authorization Code flow with PKCE"**

> "PKCE (Proof Key for Code Exchange) prevents authorization code interception attacks, especially important for public clients like SPAs and mobile apps.

> Flow: 1) Client generates random `code_verifier` and creates `code_challenge` = SHA256(code_verifier). 2) Client redirects to auth server with code_challenge. 3) User authenticates, auth server redirects back with authorization code. 4) Client exchanges code + code_verifier for tokens. 5) Auth server verifies SHA256(code_verifier) matches stored code_challenge before issuing tokens.

> Even if an attacker intercepts the authorization code, they can't exchange it without the code_verifier, which never left the client. This makes PKCE the recommended flow for all OAuth clients, replacing the deprecated Implicit flow."

**Q6: "How would you implement 'Remember Me' functionality securely?"**

> "Two approaches, both involving separate token handling:

> **Extended refresh token**: When 'Remember Me' is checked, issue a refresh token with longer expiry (30 days vs 7 days). Store in httpOnly cookie with extended maxAge. The access token stays short-lived regardless.

> **Separate remember-me token**: Issue a second, long-lived token specifically for re-authentication. Store hashed in database with device info. When user returns, exchange remember-me token for fresh session/JWT, but require password for sensitive operations.

> Either way: bind tokens to device fingerprint, allow users to view/revoke active sessions, and require re-authentication for security-critical actions. Never extend access token lifetime just because of 'Remember Me'."

### Scenario Questions

**Q7: "A user reports their account was accessed from an unknown location. How do you respond?"**

> "Immediate steps: 1) Revoke all sessions/refresh tokens for the user - implement 'logout everywhere' if not already available. 2) Force password reset on next login. 3) Check audit logs for what actions the attacker took. 4) If we have location data, notify user of suspicious login details.

> Investigation: Look for patterns - is this a single user or multiple? Check if credentials were stuffed (try against breach databases). Review authentication logs for the compromised session - how did attacker get access? Password? Session hijacking? OAuth?

> Prevention: Implement/require MFA. Add suspicious login detection (new device, new location, impossible travel). Send login notification emails. Consider step-up authentication for sensitive actions."

**Q8: "Design the authentication system for a microservices architecture"**

> "I'd use a dedicated Auth Service that handles all authentication, issuing JWTs that other services verify independently.

> **Auth Service**: Handles login, registration, MFA, OAuth, password reset. Issues short-lived JWTs (15 min) and manages refresh tokens. Stores user credentials, sessions, refresh token metadata.

> **Service-to-service**: Use mutual TLS (mTLS) or signed JWTs with service identity. Each service has its own credentials, issued by the auth service or a service mesh.

> **JWT verification**: Each service has the public key to verify JWT signatures - no network call needed. Can optionally cache user permissions from a central permissions service with short TTL.

> **API Gateway**: Can handle JWT verification at the edge, passing verified user claims to internal services. Implements rate limiting and initial auth checks.

> **Token revocation**: Minimal blacklist in Redis, replicated to all services. Or use token versioning where version is checked against user service."

### Quick Reference

```
AUTHENTICATION CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  SESSION-BASED:                                                             │
│  • Cookie: httpOnly, Secure, SameSite=Strict                               │
│  • Regenerate session ID on login                                          │
│  • Store in Redis for scaling                                              │
│  • Easy revocation (delete session)                                        │
│                                                                              │
│  JWT-BASED:                                                                 │
│  • Short access token (15 min) in memory                                   │
│  • Refresh token (7 days) in httpOnly cookie                               │
│  • Rotate refresh tokens on use                                            │
│  • Blacklist for immediate revocation                                      │
│                                                                              │
│  OAUTH 2.0:                                                                 │
│  • Authorization Code + PKCE for all clients                               │
│  • Client Credentials for server-to-server                                 │
│  • Never use Implicit flow (deprecated)                                    │
│                                                                              │
│  PASSWORD HANDLING:                                                         │
│  • Argon2id for hashing                                                    │
│  • Rate limit login attempts                                               │
│  • Account lockout after failures                                          │
│  • Constant-time comparison                                                │
│                                                                              │
│  TOKEN STORAGE:                                                             │
│  • NEVER localStorage                                                      │
│  • Access token: Memory only                                               │
│  • Refresh token: httpOnly cookie                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

