# Authentication Concepts

> Understanding how CareCircle verifies user identity.

---

## 1. What Is Authentication?

### Plain English Explanation

Authentication answers: **"Who are you?"**

Think of it like a **nightclub entrance**:
- You show your ID (credentials)
- Bouncer verifies it's real (validation)
- You get a wristband (token)
- Wristband lets you enter all night (session)

### Authentication vs Authorization

```
AUTHENTICATION: "Are you who you claim to be?"
  → Login with email/password
  → Verify identity

AUTHORIZATION: "Are you allowed to do this?"
  → Check if user can edit medication
  → Verify permissions (separate topic)
```

---

## 2. Core Concepts & Terminology

### JWT (JSON Web Token)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         JWT ANATOMY                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0In0.signature                         │
│  ─────────────────────  ──────────────────  ─────────                       │
│       HEADER                PAYLOAD          SIGNATURE                       │
│                                                                              │
│  HEADER:                                                                     │
│  { "alg": "HS256", "typ": "JWT" }                                           │
│                                                                              │
│  PAYLOAD:                                                                    │
│  { "sub": "user-123", "email": "user@email.com", "exp": 1234567890 }        │
│                                                                              │
│  SIGNATURE:                                                                  │
│  HMAC-SHA256(base64(header) + "." + base64(payload), SECRET_KEY)            │
│                                                                              │
│  The signature PROVES the token wasn't tampered with                        │
│  Anyone can READ the payload, but only the server can CREATE valid tokens   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Terminology

| Term | Definition |
|------|------------|
| **Credentials** | Email + password (proof of identity) |
| **Token** | Proof of successful authentication |
| **Access Token** | Short-lived token for API requests |
| **Refresh Token** | Long-lived token to get new access tokens |
| **Session** | Server-side record of logged-in user |
| **Hashing** | One-way transformation (password → hash) |
| **Salt** | Random data added before hashing |

---

## 3. How CareCircle's Auth Works

### The Login Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         LOGIN FLOW                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. USER SUBMITS CREDENTIALS                                                 │
│     POST /auth/login { email: "...", password: "..." }                      │
│                                                                              │
│  2. SERVER VALIDATES                                                         │
│     • Find user by email                                                    │
│     • Compare password hash (bcrypt)                                        │
│     • Check account status (not locked, email verified)                     │
│                                                                              │
│  3. SERVER CREATES TOKENS                                                    │
│     • Access Token (15 min lifetime)                                        │
│     • Refresh Token (7 days lifetime)                                       │
│     • Both signed with server secret                                        │
│                                                                              │
│  4. SERVER SENDS RESPONSE                                                    │
│     • Tokens in HTTP-only cookies                                           │
│     • User data in response body                                            │
│                                                                              │
│  5. CLIENT STORES                                                            │
│     • Browser automatically stores cookies                                  │
│     • Cookies sent with every request                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Token Refresh Flow

```
Access token expires every 15 minutes.
Instead of forcing re-login:

1. Request fails with 401
2. Client calls POST /auth/refresh
3. Server validates refresh token
4. Server issues NEW access + refresh tokens
5. Original request is retried
6. User never notices!

REFRESH TOKEN ROTATION:
Each refresh invalidates the old refresh token.
If an attacker steals a refresh token, it only works ONCE.
```

---

## 4. Why This Approach

### Why JWT (Not Sessions)?

| Sessions | JWTs |
|----------|------|
| State on server | Stateless |
| Database lookup each request | Signature verification only |
| Hard to scale | Easy horizontal scaling |
| Server controls invalidation | Needs refresh token strategy |

CareCircle uses JWTs for scalability but adds refresh token rotation for security.

### Why HTTP-only Cookies (Not localStorage)?

```
localStorage:
❌ Accessible by JavaScript
❌ XSS attack can steal tokens
❌ You must manually attach to requests

HTTP-only Cookies:
✅ NOT accessible by JavaScript
✅ XSS cannot read tokens
✅ Automatically sent with requests
✅ Can set SameSite for CSRF protection
```

---

## 5. When to Use Auth Patterns ✅

### Use JWTs When:
- Building APIs consumed by multiple clients
- Need stateless authentication
- Horizontal scaling is important

### Use Access + Refresh When:
- Want short-lived access for security
- Want to avoid frequent re-login

### Use HTTP-only Cookies When:
- Web application (browser client)
- Want maximum XSS protection

---

## 6. When to AVOID Patterns ❌

### DON'T Store Sensitive Data in JWT Payload

```
❌ BAD: Password, credit card, etc. in JWT
{ "password": "secret123" }
// Anyone can base64 decode and read!

✅ GOOD: Only non-sensitive identifiers
{ "sub": "user-123", "email": "user@email.com" }
```

### DON'T Trust the Client

```
❌ BAD: Accept user ID from request body
POST /medications { userId: "123" }
// User can send anyone's ID!

✅ GOOD: Get user ID from verified token
const userId = req.user.id;  // From JWT, verified by server
```

---

## 7. Best Practices & Recommendations

### Password Handling

```
STORAGE:
  ❌ Plain text: password123
  ❌ Simple hash: md5(password123)
  ✅ Salted bcrypt: $2b$12$...

HASHING:
  bcrypt with cost factor 12+
  Or Argon2 (newer, memory-hard)

NEVER:
  • Log passwords
  • Send passwords in responses
  • Store passwords in plain text
  • Use reversible encryption
```

### Token Best Practices

```
ACCESS TOKEN:
  • Short lifetime (15 minutes)
  • Contains user ID, email, roles
  • Signed with strong secret (256+ bits)

REFRESH TOKEN:
  • Longer lifetime (7 days)
  • Rotate on each use
  • Store hash in database for revocation

SECRETS:
  • Different secrets for access/refresh
  • Stored in environment variables
  • Never committed to code
```

---

## 8. Common Mistakes & How to Avoid Them

### Mistake 1: Weak Secrets

```
❌ BAD:
JWT_SECRET=secret123

✅ GOOD:
JWT_SECRET=a7f3b2c1d4e5f6...  (32+ random bytes)
```

### Mistake 2: No Expiration

```
❌ BAD: Token never expires
{ "sub": "user-123" }  // No exp claim

✅ GOOD: Always include expiration
{ "sub": "user-123", "exp": 1706659200 }
```

### Mistake 3: Not Validating Tokens Properly

```typescript
❌ BAD: Just decode without verify
const payload = jwt.decode(token);

✅ GOOD: Verify signature and expiration
const payload = jwt.verify(token, SECRET);
```

---

## 9. Security Considerations

### Brute Force Protection

```
• Rate limit login attempts (5 per 15 min)
• Account lockout after failures
• CAPTCHA after failed attempts
• Delay responses on failure
```

### Token Security

```
• Short access token lifetime
• Refresh token rotation
• Blacklist compromised tokens
• Secure cookie settings (HttpOnly, Secure, SameSite)
```

### Password Requirements

```
• Minimum 8 characters
• At least one uppercase
• At least one lowercase
• At least one number
• Check against common passwords
```

---

## 10. Quick Reference

### Cookie Settings

```typescript
response.cookie('access_token', token, {
  httpOnly: true,     // No JS access
  secure: true,       // HTTPS only (production)
  sameSite: 'strict', // No cross-site requests
  maxAge: 15 * 60 * 1000,  // 15 minutes
});
```

### Token Payload Structure

```typescript
interface AccessTokenPayload {
  sub: string;        // User ID
  email: string;
  role: string;
  iat: number;        // Issued at
  exp: number;        // Expires at
}

interface RefreshTokenPayload {
  sub: string;        // User ID
  tokenId: string;    // For revocation
  iat: number;
  exp: number;
}
```

### Auth Endpoints

| Endpoint | Purpose |
|----------|---------|
| `POST /auth/register` | Create account |
| `POST /auth/login` | Get tokens |
| `POST /auth/logout` | Invalidate tokens |
| `POST /auth/refresh` | Get new access token |
| `POST /auth/forgot-password` | Request reset |
| `POST /auth/reset-password` | Set new password |

---

## 11. Learning Resources

- [JWT.io](https://jwt.io) - Debugger and introduction
- OWASP Authentication Cheat Sheet
- [Auth0 Blog](https://auth0.com/blog) - In-depth articles

---

*Next: [Authorization](authorization.md) | [Security Principles](../security/principles.md)*
