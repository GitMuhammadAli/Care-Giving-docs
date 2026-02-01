# OWASP Security Concepts

> Understanding web security for CareCircle.

---

## 1. What Is OWASP?

### Plain English Explanation

OWASP (Open Web Application Security Project) is a **community that documents common web security vulnerabilities**.

Think of it like a **burglar's playbook in reverse**:
- OWASP Top 10 = The 10 most common ways apps get hacked
- Know the attacks → Build defenses
- Updated regularly as threats evolve

### Why It Matters for CareCircle

CareCircle handles **sensitive health data**:
- Medical records
- Medication schedules
- Personal information
- Family relationships

A security breach could harm vulnerable people. Security is not optional.

---

## 2. OWASP Top 10 (2021)

### The List

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         OWASP TOP 10 (2021)                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  A01: Broken Access Control         ← Most critical for CareCircle         │
│  A02: Cryptographic Failures                                                │
│  A03: Injection                     ← SQL injection, XSS                   │
│  A04: Insecure Design                                                       │
│  A05: Security Misconfiguration                                             │
│  A06: Vulnerable Components                                                 │
│  A07: Authentication Failures       ← Critical for any app                 │
│  A08: Software & Data Integrity                                             │
│  A09: Logging & Monitoring Failures                                         │
│  A10: Server-Side Request Forgery                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Key Vulnerabilities & Defenses

### A01: Broken Access Control

**What:** Users accessing data/actions they shouldn't.

```
ATTACK:
User A changes URL from /families/123 to /families/456
→ Accesses another family's data

DEFENSE (CareCircle):
• FamilyAccessGuard on every endpoint
• Family-scoped database queries
• Role-based access control (RBAC)
```

**Checklist:**
- [ ] Every endpoint has authorization guard
- [ ] Database queries filter by family membership
- [ ] Admin actions require ADMIN role
- [ ] Direct object references validated

---

### A02: Cryptographic Failures

**What:** Sensitive data exposed due to weak/missing encryption.

```
ATTACK:
• Passwords stored in plain text
• Data transmitted over HTTP
• Weak encryption algorithms

DEFENSE (CareCircle):
• Passwords hashed with bcrypt (cost 10+)
• HTTPS enforced in production
• JWT signed with strong secret
• Sensitive data encrypted at rest (future)
```

**Checklist:**
- [ ] All passwords hashed with bcrypt
- [ ] HTTPS only in production
- [ ] Strong JWT secrets (256-bit+)
- [ ] No sensitive data in logs

---

### A03: Injection

**What:** Malicious input interpreted as code.

```
SQL INJECTION:
Input: ' OR '1'='1
Query: SELECT * FROM users WHERE email = '' OR '1'='1'
Result: Returns all users!

DEFENSE (CareCircle):
• Prisma/TypeORM parameterized queries (automatic)
• Input validation with class-validator
• Never concatenate user input into queries
```

```
XSS (Cross-Site Scripting):
Input: <script>steal(document.cookie)</script>
Result: Executes in victim's browser

DEFENSE (CareCircle):
• React auto-escapes JSX (automatic)
• HTTP-only cookies (JS can't read tokens)
• Content Security Policy headers
```

**Checklist:**
- [ ] Using ORM (no raw SQL with user input)
- [ ] All input validated with DTOs
- [ ] React rendering (auto-escapes)
- [ ] HTTP-only cookies for tokens

---

### A07: Authentication Failures

**What:** Weak authentication allows account takeover.

```
ATTACKS:
• Brute force passwords
• Session hijacking
• Credential stuffing

DEFENSE (CareCircle):
• Rate limiting on login (5 attempts/15 min)
• Strong password requirements
• JWT + refresh token rotation
• Session tracking (IP, user agent)
• Account lockout after failures
```

**Checklist:**
- [ ] Rate limiting on auth endpoints
- [ ] Password complexity enforced
- [ ] Refresh tokens rotated
- [ ] Sessions tracked and revocable

---

## 4. CareCircle Security Measures

### Authentication Security

```
✅ Implemented:
• JWT access tokens (15 min expiry)
• Refresh token rotation
• HTTP-only, Secure, SameSite cookies
• bcrypt password hashing
• Email verification required
• Rate limiting on login

🔜 Future:
• Two-factor authentication
• Password breach checking
• Login anomaly detection
```

### Authorization Security

```
✅ Implemented:
• Family membership verification
• Role-based access (ADMIN/CAREGIVER/VIEWER)
• Guards on all protected endpoints
• Family-scoped database queries

🔜 Future:
• Audit logging for sensitive actions
• Fine-grained permissions
```

### Data Security

```
✅ Implemented:
• HTTPS in production
• Passwords hashed
• Tokens signed
• Input validation

🔜 Future:
• Field-level encryption
• Data anonymization
• Backup encryption
```

---

## 5. Security Best Practices

### Input Validation

```typescript
// Always validate with DTOs
class CreateMedicationDto {
  @IsString()
  @IsNotEmpty()
  @MaxLength(100)
  name: string;

  @IsString()
  @Matches(/^\d+(\.\d+)?(mg|ml|g)$/)  // e.g., "100mg"
  dosage: string;

  @IsUUID()
  careRecipientId: string;
}
```

### Output Encoding

```typescript
// Never trust user input in responses
// React auto-escapes, but be careful with:

// ❌ DANGEROUS:
<div dangerouslySetInnerHTML={{ __html: userContent }} />

// ✅ SAFE:
<div>{userContent}</div>  // Auto-escaped
```

### Error Handling

```typescript
// ❌ BAD: Expose internal details
catch (error) {
  return { error: error.stack };  // Shows internals!
}

// ✅ GOOD: Generic message
catch (error) {
  logger.error('Database error', error);  // Log internally
  return { error: 'An error occurred' };  // Generic to user
}
```

---

## 6. Security Checklist

### Development

- [ ] Dependencies updated regularly
- [ ] No secrets in code (use .env)
- [ ] Input validation on all endpoints
- [ ] Authorization on all protected routes
- [ ] Error messages don't leak details

### Deployment

- [ ] HTTPS enabled
- [ ] Security headers configured
- [ ] Rate limiting enabled
- [ ] Logging and monitoring active
- [ ] Secrets in environment variables

### Ongoing

- [ ] Dependency audit (`npm audit`)
- [ ] Security testing in CI/CD
- [ ] Incident response plan
- [ ] Regular security reviews

---

## 7. Quick Reference

### Security Headers

```typescript
// Helmet.js middleware
app.use(helmet());

// Sets these headers:
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000
```

### Secure Cookie Settings

```typescript
response.cookie('token', value, {
  httpOnly: true,     // No JS access
  secure: true,       // HTTPS only
  sameSite: 'strict', // No cross-site
  maxAge: 15 * 60 * 1000,
});
```

### Rate Limiting

```typescript
@UseGuards(ThrottlerGuard)
@Throttle(5, 900)  // 5 requests per 15 minutes
@Post('login')
async login() { }
```

---

## 8. Common Attacks & Defenses Summary

| Attack | Defense |
|--------|---------|
| SQL Injection | Use ORM, parameterized queries |
| XSS | React auto-escape, CSP headers |
| CSRF | SameSite cookies, CSRF tokens |
| Brute Force | Rate limiting, account lockout |
| Session Hijacking | HTTP-only cookies, token rotation |
| Broken Access | Guards, family-scoped queries |
| Data Exposure | HTTPS, encryption, secure storage |

---

## 9. Resources

- [OWASP Top 10](https://owasp.org/Top10/)
- [OWASP Cheat Sheets](https://cheatsheetseries.owasp.org/)
- [Node.js Security Checklist](https://blog.risingstack.com/node-js-security-checklist/)
- [NestJS Security](https://docs.nestjs.com/security/helmet)

---

*Next: [Security Principles](principles.md) | [Authentication](../backend/authentication.md)*

