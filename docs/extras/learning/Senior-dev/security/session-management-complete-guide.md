# 🔄 Session Management - Complete Guide

> A comprehensive guide to session management - secure cookies, session fixation, session hijacking, and maintaining user identity securely.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Session management maintains user state across multiple requests - the session ID is effectively a bearer token for the user's identity, so protecting it is critical: secure cookies, regeneration on login, proper invalidation on logout, and defense against fixation and hijacking attacks."

### Session Security Model
```
SESSION SECURITY:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SESSION LIFECYCLE:                                             │
│                                                                  │
│  1. CREATE (on login)                                          │
│     • Generate cryptographically random session ID             │
│     • Store session data server-side                           │
│     • Send session ID in secure cookie                         │
│                                                                  │
│  2. VALIDATE (each request)                                     │
│     • Receive cookie from browser                              │
│     • Look up session in store                                 │
│     • Verify not expired                                       │
│     • Attach user to request                                   │
│                                                                  │
│  3. REGENERATE (on privilege change)                            │
│     • Generate new session ID                                  │
│     • Transfer session data to new ID                          │
│     • Delete old session                                       │
│     • Prevents session fixation                                │
│                                                                  │
│  4. DESTROY (on logout)                                         │
│     • Delete session from store                                │
│     • Clear session cookie                                     │
│     • Invalidate any tokens                                    │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  COOKIE FLAGS (ALL required for security):                      │
│                                                                  │
│  HttpOnly    → JavaScript cannot read cookie                   │
│  Secure      → Cookie only sent over HTTPS                     │
│  SameSite    → Strict/Lax prevents CSRF                        │
│  Path=/      → Sent for all paths                              │
│  Max-Age     → Expiry time                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement
> "I implemented a defense-in-depth session strategy. Sessions stored in Redis with 24-hour expiry and sliding window renewal. Session IDs are regenerated on every privilege change (login, role change, password change) to prevent fixation attacks. Cookies have HttpOnly, Secure, SameSite=Strict. We bind sessions to IP and user-agent fingerprint - if either changes mid-session, we require re-authentication. For sensitive operations (payment, password change), we use step-up authentication requiring fresh login within 5 minutes. All session events are logged for audit."

---

## 📚 Core Implementation

### Secure Session Configuration

```typescript
// ════════════════════════════════════════════════════════════════
// EXPRESS SESSION SETUP WITH REDIS
// ════════════════════════════════════════════════════════════════

import session from 'express-session';
import RedisStore from 'connect-redis';
import Redis from 'ioredis';
import crypto from 'crypto';

const redis = new Redis({
    host: process.env.REDIS_HOST,
    port: 6379,
    password: process.env.REDIS_PASSWORD
});

app.use(session({
    store: new RedisStore({ client: redis }),
    
    // Session ID settings
    genid: () => crypto.randomUUID(),  // Cryptographically random
    name: 'sessionId',  // Don't use default 'connect.sid' (fingerprinting)
    
    // Security settings
    secret: process.env.SESSION_SECRET,  // Use strong secret
    resave: false,
    saveUninitialized: false,  // Don't create session until needed
    
    // Cookie settings
    cookie: {
        httpOnly: true,     // JavaScript cannot access
        secure: true,       // HTTPS only (set false for local dev)
        sameSite: 'strict', // Prevent CSRF
        maxAge: 24 * 60 * 60 * 1000,  // 24 hours
        path: '/',
        domain: process.env.NODE_ENV === 'production' 
            ? '.myapp.com'  // Share across subdomains if needed
            : undefined
    }
}));

// ════════════════════════════════════════════════════════════════
// DEVELOPMENT VS PRODUCTION SETTINGS
// ════════════════════════════════════════════════════════════════

const isProduction = process.env.NODE_ENV === 'production';

app.use(session({
    // ...
    cookie: {
        httpOnly: true,
        secure: isProduction,  // Allow HTTP in development
        sameSite: isProduction ? 'strict' : 'lax',
        maxAge: 24 * 60 * 60 * 1000
    },
    proxy: isProduction  // Trust proxy for HTTPS detection behind load balancer
}));

// If behind proxy (nginx, load balancer)
app.set('trust proxy', 1);
```

### Session Regeneration (Prevent Fixation)

```typescript
// ════════════════════════════════════════════════════════════════
// SESSION FIXATION PREVENTION
// ════════════════════════════════════════════════════════════════

/*
SESSION FIXATION ATTACK:
1. Attacker gets a valid session ID (visits site)
2. Attacker tricks victim into using that session ID
3. Victim logs in with attacker's session ID
4. Attacker now has authenticated session!

PREVENTION: Regenerate session ID on login
*/

// Login handler with session regeneration
app.post('/login', async (req, res) => {
    const { email, password } = req.body;
    
    // Verify credentials
    const user = await verifyCredentials(email, password);
    if (!user) {
        return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // CRITICAL: Regenerate session ID before storing user data
    req.session.regenerate((err) => {
        if (err) {
            return res.status(500).json({ error: 'Session error' });
        }
        
        // Now safe to store user info in new session
        req.session.userId = user.id;
        req.session.role = user.role;
        req.session.loginTime = Date.now();
        req.session.lastActivity = Date.now();
        
        req.session.save((err) => {
            if (err) {
                return res.status(500).json({ error: 'Session error' });
            }
            res.json({ message: 'Login successful' });
        });
    });
});

// Regenerate on ANY privilege change
async function upgradeToAdmin(req, res) {
    // Regenerate session before upgrading privileges
    req.session.regenerate((err) => {
        if (err) return res.status(500).json({ error: 'Session error' });
        
        req.session.userId = req.user.id;
        req.session.role = 'admin';  // Upgrade
        req.session.save(() => {
            res.json({ message: 'Upgraded to admin' });
        });
    });
}
```

### Secure Logout

```typescript
// ════════════════════════════════════════════════════════════════
// SECURE LOGOUT
// ════════════════════════════════════════════════════════════════

app.post('/logout', (req, res) => {
    // Get session ID for audit logging
    const sessionId = req.sessionID;
    const userId = req.session?.userId;
    
    // Destroy session on server
    req.session.destroy((err) => {
        if (err) {
            console.error('Session destroy error:', err);
        }
        
        // Clear the session cookie
        res.clearCookie('sessionId', {
            httpOnly: true,
            secure: true,
            sameSite: 'strict',
            path: '/'
        });
        
        // Audit log
        console.log('User logged out:', { userId, sessionId });
        
        res.json({ message: 'Logged out' });
    });
});

// ════════════════════════════════════════════════════════════════
// LOGOUT ALL SESSIONS (For password change, security breach)
// ════════════════════════════════════════════════════════════════

async function logoutAllSessions(userId: string) {
    // Approach 1: Track sessions per user in Redis
    const sessionKeys = await redis.keys(`sess:${userId}:*`);
    if (sessionKeys.length > 0) {
        await redis.del(...sessionKeys);
    }
    
    // Approach 2: Use session version
    // Store version number with user
    await db.users.update({
        where: { id: userId },
        data: { sessionVersion: { increment: 1 } }
    });
    
    // In middleware, check session version matches
}

// Middleware to check session version
async function validateSessionVersion(req, res, next) {
    if (!req.session.userId) return next();
    
    const user = await db.users.findUnique({
        where: { id: req.session.userId },
        select: { sessionVersion: true }
    });
    
    if (user?.sessionVersion !== req.session.sessionVersion) {
        // Session was invalidated
        req.session.destroy(() => {
            res.status(401).json({ error: 'Session expired' });
        });
        return;
    }
    
    next();
}
```

### Session Hijacking Prevention

```typescript
// ════════════════════════════════════════════════════════════════
// SESSION BINDING (Prevent hijacking)
// ════════════════════════════════════════════════════════════════

import crypto from 'crypto';

// Create fingerprint from request
function createFingerprint(req: Request): string {
    const components = [
        req.ip,
        req.headers['user-agent'],
        req.headers['accept-language']
    ];
    
    return crypto
        .createHash('sha256')
        .update(components.join('|'))
        .digest('hex');
}

// On login, store fingerprint
app.post('/login', async (req, res) => {
    // ... authentication ...
    
    req.session.regenerate((err) => {
        req.session.userId = user.id;
        req.session.fingerprint = createFingerprint(req);
        req.session.save(() => res.json({ success: true }));
    });
});

// Middleware to validate fingerprint
function validateFingerprint(req, res, next) {
    if (req.session.userId) {
        const currentFingerprint = createFingerprint(req);
        
        if (req.session.fingerprint !== currentFingerprint) {
            // Fingerprint changed - potential hijacking
            console.warn('Session fingerprint mismatch:', {
                userId: req.session.userId,
                expected: req.session.fingerprint,
                actual: currentFingerprint
            });
            
            // Force re-authentication
            req.session.destroy(() => {
                res.status(401).json({ 
                    error: 'Session expired',
                    code: 'FINGERPRINT_MISMATCH'
                });
            });
            return;
        }
    }
    
    next();
}

// ════════════════════════════════════════════════════════════════
// SESSION ACTIVITY MONITORING
// ════════════════════════════════════════════════════════════════

// Middleware to track activity and implement sliding expiration
function sessionActivity(req, res, next) {
    if (req.session.userId) {
        const now = Date.now();
        const lastActivity = req.session.lastActivity || now;
        const INACTIVE_TIMEOUT = 30 * 60 * 1000;  // 30 minutes
        
        if (now - lastActivity > INACTIVE_TIMEOUT) {
            // Session inactive too long
            req.session.destroy(() => {
                res.status(401).json({ error: 'Session expired due to inactivity' });
            });
            return;
        }
        
        // Update last activity
        req.session.lastActivity = now;
    }
    
    next();
}
```

### Step-Up Authentication

```typescript
// ════════════════════════════════════════════════════════════════
// STEP-UP AUTHENTICATION FOR SENSITIVE OPERATIONS
// ════════════════════════════════════════════════════════════════

// Require fresh authentication for sensitive operations
function requireFreshAuth(maxAgeMinutes: number = 5) {
    return (req, res, next) => {
        const loginTime = req.session.lastAuthTime || req.session.loginTime;
        const ageMinutes = (Date.now() - loginTime) / 60000;
        
        if (ageMinutes > maxAgeMinutes) {
            return res.status(401).json({
                error: 'Re-authentication required',
                code: 'STEP_UP_REQUIRED'
            });
        }
        
        next();
    };
}

// Step-up auth endpoint
app.post('/auth/step-up', authenticate, async (req, res) => {
    const { password } = req.body;
    
    // Verify password
    const user = await db.users.findUnique({ where: { id: req.session.userId } });
    const valid = await bcrypt.compare(password, user.passwordHash);
    
    if (!valid) {
        return res.status(401).json({ error: 'Invalid password' });
    }
    
    // Update auth timestamp
    req.session.lastAuthTime = Date.now();
    req.session.save(() => {
        res.json({ message: 'Re-authenticated' });
    });
});

// Protected sensitive routes
app.post('/settings/change-password', 
    authenticate,
    requireFreshAuth(5),  // Must have authed in last 5 minutes
    changePassword
);

app.post('/payment/submit',
    authenticate,
    requireFreshAuth(5),
    submitPayment
);
```

---

## Session Storage Options

```
SESSION STORAGE COMPARISON:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  MEMORY (Default) ❌                                            │
│  • Lost on restart                                             │
│  • Not scalable (single server)                                │
│  • Memory leaks                                                │
│  → Never use in production                                     │
│                                                                  │
│  REDIS ✅ (Recommended)                                         │
│  • Fast, in-memory                                             │
│  • Automatic expiration (TTL)                                  │
│  • Scalable (cluster mode)                                     │
│  • connect-redis package                                       │
│                                                                  │
│  DATABASE                                                       │
│  • Persistent                                                  │
│  • Slower than Redis                                           │
│  • Need cleanup job for expired sessions                       │
│  → Use if Redis not available                                  │
│                                                                  │
│  COOKIES (JWT)                                                  │
│  • Stateless (no server storage)                               │
│  • Can't revoke easily                                         │
│  • Size limitations (4KB)                                      │
│  → Use for APIs, not traditional sessions                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "What is session fixation and how do you prevent it?"**
> "Session fixation is when an attacker sets a victim's session ID before they authenticate. The attacker knows the ID, so after the victim logs in, the attacker has an authenticated session. Prevention: regenerate the session ID on login and any privilege change. The old session ID becomes invalid, so the attacker's known ID is useless."

**Q: "What cookie flags are essential for secure sessions?"**
> "HttpOnly prevents JavaScript access (XSS can't steal the cookie). Secure ensures cookie only sent over HTTPS (prevents network sniffing). SameSite=Strict prevents CSRF (cookie not sent with cross-site requests). Together, they protect against XSS stealing cookies, network interception, and cross-site request forgery."

**Q: "How do you implement 'logout everywhere' functionality?"**
> "Two approaches: 1) Track all session IDs per user in Redis, delete them all on request. 2) Store a session version number with the user. Each session stores its version. On 'logout everywhere', increment the version. Middleware checks if session version matches - if not, session is invalid. The version approach is simpler for distributed systems."

---

## Quick Reference

```
SESSION SECURITY CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  COOKIE FLAGS:                                                  │
│  □ HttpOnly: true                                              │
│  □ Secure: true (production)                                   │
│  □ SameSite: strict                                            │
│  □ Custom name (not 'connect.sid')                             │
│                                                                  │
│  SESSION LIFECYCLE:                                             │
│  □ Regenerate ID on login                                      │
│  □ Regenerate on privilege change                              │
│  □ Proper destroy on logout                                    │
│  □ Clear cookie on logout                                      │
│                                                                  │
│  PROTECTION:                                                    │
│  □ Cryptographically random session IDs                        │
│  □ Server-side storage (Redis)                                 │
│  □ Session timeout (idle + absolute)                           │
│  □ Fingerprint binding (optional)                              │
│  □ Step-up auth for sensitive ops                              │
│                                                                  │
│  MONITORING:                                                    │
│  □ Log session events                                          │
│  □ Alert on anomalies                                          │
│  □ Concurrent session limits                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


