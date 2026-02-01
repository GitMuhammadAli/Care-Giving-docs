# Security Principles

> How CareCircle protects user data and maintains trust.

---

## Why Security Matters More Here

CareCircle handles **health-adjacent data**:

- Medication schedules (could cause harm if wrong)
- Emergency contacts (life-critical)
- Health conditions (private)
- Family relationships (sensitive)

A breach doesn't just expose data—it could **endanger** someone.

---

## The Defense in Depth Philosophy

### The Mental Model: Castle Security

```
                          ┌─────────────────────────┐
                          │      THE DATA           │
                          │   (Family health info)  │
                          └───────────┬─────────────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    │                                   │
                    │    LAYER 5: DATA ENCRYPTION       │
                    │    • At rest (database)           │
                    │    • In transit (HTTPS)           │
                    │                                   │
            ┌───────┼───────────────────────────────────┼───────┐
            │       │                                   │       │
            │       │    LAYER 4: INPUT VALIDATION      │       │
            │       │    • DTO validation               │       │
            │       │    • SQL injection prevention     │       │
            │       │    • XSS sanitization             │       │
            │       │                                   │       │
        ┌───┼───────┼───────────────────────────────────┼───────┼───┐
        │   │       │                                   │       │   │
        │   │       │    LAYER 3: AUTHORIZATION         │       │   │
        │   │       │    • Family membership            │       │   │
        │   │       │    • Role-based access            │       │   │
        │   │       │    • Resource ownership           │       │   │
        │   │       │                                   │       │   │
    ┌───┼───┼───────┼───────────────────────────────────┼───────┼───┼───┐
    │   │   │       │                                   │       │   │   │
    │   │   │       │    LAYER 2: AUTHENTICATION        │       │   │   │
    │   │   │       │    • JWT validation               │       │   │   │
    │   │   │       │    • Token expiration             │       │   │   │
    │   │   │       │    • Session management           │       │   │   │
    │   │   │       │                                   │       │   │   │
┌───┼───┼───┼───────┼───────────────────────────────────┼───────┼───┼───┼───┐
│   │   │   │       │                                   │       │   │   │   │
│   │   │   │       │    LAYER 1: NETWORK               │       │   │   │   │
│   │   │   │       │    • HTTPS/TLS                    │       │   │   │   │
│   │   │   │       │    • CORS policy                  │       │   │   │   │
│   │   │   │       │    • Rate limiting                │       │   │   │   │
│   │   │   │       │    • Firewall rules               │       │   │   │   │
│   │   │   │       │                                   │       │   │   │   │
└───┴───┴───┴───────┴───────────────────────────────────┴───────┴───┴───┴───┘
                                      │
                          ┌───────────▼───────────────┐
                          │       ATTACKER            │
                          └───────────────────────────┘
```

### The Key Principle

**No single layer provides complete protection.** Each layer assumes the previous layer might fail.

---

## Principle 1: Authentication (Who Are You?)

### Token-Based Auth with JWTs

```
THE FLOW:
─────────

1. USER LOGS IN
   POST /auth/login { email, password }
   
   Server:
   • Verifies credentials
   • Creates access token (short-lived: 15 min)
   • Creates refresh token (longer-lived: 7 days)
   • Sends both in HTTP-only cookies

2. USER MAKES REQUESTS
   GET /medications
   
   Server:
   • Extracts token from cookie
   • Validates signature (not tampered)
   • Checks expiration (not expired)
   • Extracts user ID from payload
   • Proceeds with request

3. TOKEN EXPIRES
   Access token hits 15 min limit
   
   Server:
   • Returns 401 Unauthorized
   • Client automatically calls /auth/refresh
   • New tokens issued
   • Original request retried
```

### Why HTTP-only Cookies (Not localStorage)?

```
localStorage:
─────────────
  ❌ Accessible by JavaScript
  ❌ XSS attack can steal token
  ❌ Token visible in browser DevTools
  
  Attacker injects script → reads localStorage → steals token → game over


HTTP-only Cookies:
──────────────────
  ✅ NOT accessible by JavaScript
  ✅ XSS cannot read the token
  ✅ Automatically sent with requests
  
  Attacker injects script → cannot access cookie → blocked!
```

### Token Security Measures

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         TOKEN SECURITY CHECKLIST                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ✅ Short access token lifetime (15 min)                                    │
│     → Limits damage if token is compromised                                 │
│                                                                              │
│  ✅ Refresh token rotation                                                  │
│     → Each refresh invalidates old token                                    │
│     → Stolen refresh token only works once                                  │
│                                                                              │
│  ✅ Secure cookie flags                                                     │
│     → HttpOnly: JS can't read                                               │
│     → Secure: HTTPS only                                                    │
│     → SameSite=Strict: No cross-site requests                              │
│                                                                              │
│  ✅ Strong secret keys                                                      │
│     → 256-bit minimum                                                       │
│     → Different for access vs refresh                                       │
│     → Stored in environment variables                                       │
│                                                                              │
│  ✅ Token blacklist for logout                                              │
│     → Invalidate tokens before natural expiration                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Principle 2: Authorization (What Can You Do?)

### The Family-Scoped Model

```
USER cannot access data unless:
───────────────────────────────

1. They are a MEMBER of the FAMILY
2. The data BELONGS to that FAMILY
3. Their ROLE allows the OPERATION

EXAMPLE:
────────

Alice (ADMIN of Family A)
  → Can view Family A's medications     ✅
  → Can edit Family A's medications     ✅
  → Can view Family B's medications     ❌ (not a member)

Bob (VIEWER of Family A)
  → Can view Family A's medications     ✅
  → Can edit Family A's medications     ❌ (viewer can't edit)
```

### Role Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ROLE PERMISSIONS MATRIX                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Role          │ View │ Create │ Edit │ Delete │ Invite │ Manage Members    │
│  ──────────────┼──────┼────────┼──────┼────────┼────────┼───────────────    │
│  VIEWER        │  ✅  │   ❌   │  ❌  │   ❌   │   ❌   │      ❌           │
│  CAREGIVER     │  ✅  │   ✅   │  ✅  │   ❌   │   ❌   │      ❌           │
│  ADMIN         │  ✅  │   ✅   │  ✅  │   ✅   │   ✅   │      ✅           │
│                                                                              │
│  VIEWER: Family members who just need to see info (relatives)               │
│  CAREGIVER: Active caregivers who log medications, shifts                   │
│  ADMIN: Family managers who can add/remove people                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### How Guards Work

```typescript
// 1. JwtAuthGuard (global) - Applied to all routes
@UseGuards(JwtAuthGuard)  // In AppModule

// Effect: All routes require valid JWT by default
// Opt-out: @Public() decorator for login, register, etc.


// 2. FamilyAccessGuard - Applied per route
@UseGuards(FamilyAccessGuard)
@FamilyAccess({ param: 'familyId', roles: [FamilyRole.ADMIN] })
async deleteMedication(@Param('familyId') familyId: string) {
  // Guard already verified:
  // - User is member of this family
  // - User has ADMIN role
  // Safe to proceed
}
```

### The Authorization Check Pattern

```typescript
// ALWAYS verify ownership, don't trust IDs from request

❌ WRONG: Trust the input

async getMedication(@Param('id') id: string) {
  return this.prisma.medication.findUnique({ where: { id } });
  // Anyone who guesses the ID can access!
}


✅ RIGHT: Verify ownership

async getMedication(
  @Param('id') id: string,
  @CurrentUser() user: User  // From JWT
) {
  const medication = await this.prisma.medication.findUnique({
    where: { id },
    include: { careRecipient: { include: { family: true } } }
  });
  
  // Verify user can access this medication's family
  const isMember = await this.isFamilyMember(
    user.id, 
    medication.careRecipient.familyId
  );
  
  if (!isMember) {
    throw new ForbiddenException('Not a family member');
  }
  
  return medication;
}
```

---

## Principle 3: Input Validation (Never Trust User Input)

### The Validation Pipeline

```
USER INPUT
    │
    ▼
┌───────────────────┐
│   TYPE COERCION   │  "123" → 123 (if number expected)
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│   SANITIZATION    │  Remove/escape dangerous characters
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  SCHEMA VALIDATION│  Check against DTO rules
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ BUSINESS VALIDATION│  Domain-specific rules
└─────────┬─────────┘
          │
          ▼
    SAFE TO USE
```

### DTO Validation Example

```typescript
// DTOs are the first line of defense

class CreateMedicationDto {
  @IsString()
  @Length(1, 100)
  @Transform(({ value }) => sanitize(value))  // Sanitize input
  name: string;

  @IsString()
  @IsNotEmpty()
  dosage: string;

  @IsEnum(MedicationFrequency)
  frequency: MedicationFrequency;

  @IsUUID()
  careRecipientId: string;  // Must be valid UUID format

  @IsOptional()
  @IsArray()
  @MaxLength(500, { each: true })
  notes?: string[];
}

// Validation is automatic via ValidationPipe
// Invalid input → 400 Bad Request with details
```

### SQL Injection Prevention

```typescript
❌ DANGER: String interpolation

const query = `SELECT * FROM users WHERE email = '${email}'`;
// If email = "'; DROP TABLE users; --"
// Query becomes: SELECT * FROM users WHERE email = ''; DROP TABLE users; --'


✅ SAFE: Parameterized queries (Prisma does this automatically)

const user = await prisma.user.findUnique({
  where: { email }  // Prisma escapes automatically
});
// email is never part of the SQL string
```

---

## Principle 4: Rate Limiting (Prevent Abuse)

### Why Rate Limit?

```
WITHOUT RATE LIMITING:
──────────────────────

Attacker scripts:
  for i in 1..1000000:
    POST /auth/login { email: "victim@email.com", password: passwords[i] }

Result: Password cracked by brute force


WITH RATE LIMITING:
───────────────────

After 5 failed attempts:
  429 Too Many Requests
  "Try again in 15 minutes"

Result: Brute force infeasible
```

### CareCircle's Rate Limiting Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         RATE LIMITS BY ENDPOINT                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ENDPOINT                    │ LIMIT           │ WINDOW   │ WHY            │
│  ────────────────────────────┼─────────────────┼──────────┼───────────────  │
│  POST /auth/login            │ 5 attempts      │ 15 min   │ Brute force    │
│  POST /auth/register         │ 3 attempts      │ 1 hour   │ Spam accounts  │
│  POST /auth/forgot-password  │ 3 attempts      │ 1 hour   │ Email bombing  │
│  POST /auth/verify-otp       │ 3 attempts      │ 15 min   │ OTP brute force│
│                              │                 │          │                │
│  General API                 │ 100 requests    │ 1 min    │ Fair usage     │
│  File uploads                │ 10 uploads      │ 1 hour   │ Storage abuse  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Principle 5: Secure Defaults

### The Philosophy

```
DEFAULT: SECURE
───────────────

Everything is locked down by default.
Security is OPT-OUT, not OPT-IN.

Examples:
• All routes require auth (opt-out with @Public())
• All inputs are validated (opt-out with @SkipValidation())
• All responses exclude sensitive fields (opt-in with @Expose())
```

### Configuration Checklist

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      SECURE DEFAULTS CHECKLIST                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  HELMET (HTTP headers)                                                       │
│  ✅ X-Content-Type-Options: nosniff                                         │
│  ✅ X-Frame-Options: DENY                                                   │
│  ✅ X-XSS-Protection: 1; mode=block                                         │
│  ✅ Strict-Transport-Security (HSTS)                                        │
│                                                                              │
│  CORS                                                                        │
│  ✅ Whitelist specific origins (not *)                                      │
│  ✅ Credentials: true (for cookies)                                         │
│  ✅ Allowed methods explicitly listed                                       │
│                                                                              │
│  COOKIES                                                                     │
│  ✅ HttpOnly: true                                                          │
│  ✅ Secure: true (production)                                               │
│  ✅ SameSite: Strict                                                        │
│                                                                              │
│  PASSWORDS                                                                   │
│  ✅ Bcrypt/Argon2 hashing (not SHA/MD5)                                     │
│  ✅ Minimum 8 characters                                                    │
│  ✅ Never logged or exposed in errors                                       │
│                                                                              │
│  LOGGING                                                                     │
│  ✅ No passwords in logs                                                    │
│  ✅ No tokens in logs                                                       │
│  ✅ PII is masked                                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Principle 6: Secrets Management

### The Rules

```
NEVER IN CODE:
──────────────
❌ const API_KEY = "sk_live_abc123";
❌ // Password: admin123

NEVER IN GIT:
─────────────
❌ .env file committed
❌ config.json with credentials
❌ Hardcoded connection strings

ALWAYS:
───────
✅ Environment variables
✅ .env.example with placeholder values
✅ Secrets manager in production (Vault, AWS Secrets Manager)
✅ Different secrets per environment
```

### Environment Variable Validation

```typescript
// Don't trust environment variables exist - validate them

// In @carecircle/config package
const ConfigSchema = z.object({
  JWT_SECRET: z.string().min(32, 'JWT_SECRET must be at least 32 characters'),
  DATABASE_URL: z.string().url('DATABASE_URL must be a valid URL'),
  REDIS_URL: z.string().url('REDIS_URL must be a valid URL'),
  // ...
});

// App startup
const config = ConfigSchema.safeParse(process.env);
if (!config.success) {
  console.error('Invalid configuration:', config.error);
  process.exit(1);  // Fail fast if misconfigured
}
```

---

## Common Attacks & Defenses

### Attack Reference Table

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ATTACK → DEFENSE MATRIX                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ATTACK                │  HOW IT WORKS           │  OUR DEFENSE             │
│  ──────────────────────┼─────────────────────────┼─────────────────────────  │
│  SQL Injection         │  Malicious SQL in input │  Prisma (parameterized)  │
│  XSS                   │  Script in user content │  Sanitization + CSP      │
│  CSRF                  │  Forged cross-site req  │  SameSite cookies        │
│  Brute Force           │  Password guessing      │  Rate limiting + lockout │
│  Session Hijacking     │  Steal session token    │  HttpOnly cookies, HTTPS │
│  Insecure Direct Obj   │  Guess resource IDs     │  UUID + ownership check  │
│  Mass Assignment       │  Extra fields in input  │  DTO whitelisting        │
│  Token Theft           │  Steal JWT              │  Short expiry, rotation  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Security Checklist for New Features

### Before Writing Code

```
□ Does this feature handle sensitive data?
□ Who should be able to access this?
□ What's the worst that could happen if compromised?
□ What input does this accept?
```

### While Writing Code

```
□ Is auth required? (If yes, guards applied?)
□ Is authorization checked? (Family membership, role)
□ Is input validated? (DTO with decorators)
□ Are errors generic? (No stack traces to client)
□ Is logging safe? (No passwords, tokens, PII)
```

### Before Deploying

```
□ Have you tested with invalid input?
□ Have you tested unauthorized access?
□ Are environment variables set correctly?
□ Is HTTPS configured?
```

---

## Quick Reference

### Security Headers Checklist

```nginx
# In Nginx or via Helmet.js
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
Referrer-Policy: strict-origin-when-cross-origin
```

### Password Requirements

```
• Minimum 8 characters
• At least one uppercase letter
• At least one lowercase letter
• At least one number
• Hashed with bcrypt (cost factor 12) or Argon2
• Never stored in plain text
• Never logged
```

---

*Next: [Authentication Deep Dive](auth-security.md) | [Data Protection](data-protection.md)*


