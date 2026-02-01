# 🛡️ Security Headers - Complete Guide

> A comprehensive guide to security headers - CSP, HSTS, X-Frame-Options, X-Content-Type-Options, and protecting your application through HTTP response headers.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Security headers are HTTP response headers that instruct browsers to enable security features - they're a critical defense-in-depth layer that costs nothing to implement and protects against XSS, clickjacking, MIME sniffing, and protocol downgrade attacks."

### Essential Security Headers
```
SECURITY HEADERS OVERVIEW:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  MUST HAVE:                                                     │
│  ─────────                                                      │
│  Content-Security-Policy (CSP)                                  │
│      → Controls what resources can load (XSS prevention)       │
│                                                                  │
│  Strict-Transport-Security (HSTS)                               │
│      → Forces HTTPS (prevents downgrade attacks)               │
│                                                                  │
│  X-Content-Type-Options                                         │
│      → Prevents MIME type sniffing                             │
│                                                                  │
│  X-Frame-Options                                                │
│      → Prevents clickjacking (iframe embedding)                │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  SHOULD HAVE:                                                   │
│  ───────────                                                    │
│  Referrer-Policy                                                │
│      → Controls referrer information sent                      │
│                                                                  │
│  Permissions-Policy                                             │
│      → Controls browser features (camera, mic, etc.)           │
│                                                                  │
│  X-XSS-Protection                                               │
│      → Legacy XSS filter (mostly obsolete)                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement
> "I implemented a comprehensive security headers strategy that achieved an A+ on securityheaders.com. CSP in report-only mode first to catch violations without breaking the site, then enforced with nonces for inline scripts. HSTS with 2-year max-age and preload submission to browser lists. We blocked framing completely with X-Frame-Options DENY since we don't need iframe embedding. The key was iterating: start permissive, monitor CSP reports, tighten gradually. One CSP change blocked a third-party tracking script we didn't know was injected - that's defense-in-depth working."

---

## 📚 Core Headers

### Content-Security-Policy (CSP)

```typescript
// ════════════════════════════════════════════════════════════════
// CONTENT SECURITY POLICY - THE BIG ONE
// ════════════════════════════════════════════════════════════════

import helmet from 'helmet';

// Basic CSP
app.use(helmet.contentSecurityPolicy({
    directives: {
        defaultSrc: ["'self'"],                    // Default for all resources
        scriptSrc: ["'self'", "https://cdn.example.com"],
        styleSrc: ["'self'", "'unsafe-inline'"],   // Allow inline styles
        imgSrc: ["'self'", "data:", "https:"],     // Images from HTTPS
        connectSrc: ["'self'", "https://api.example.com"],
        fontSrc: ["'self'", "https://fonts.gstatic.com"],
        objectSrc: ["'none'"],                     // No Flash, etc.
        mediaSrc: ["'self'"],
        frameSrc: ["'none'"],                      // No iframes
        baseUri: ["'self'"],                       // Restrict <base> tag
        formAction: ["'self'"],                    // Form submissions
        frameAncestors: ["'none'"],                // Can't be framed
        upgradeInsecureRequests: [],               // Upgrade HTTP to HTTPS
    }
}));

// ════════════════════════════════════════════════════════════════
// CSP WITH NONCES (Secure inline scripts)
// ════════════════════════════════════════════════════════════════

import crypto from 'crypto';

// Generate nonce per request
app.use((req, res, next) => {
    res.locals.nonce = crypto.randomBytes(16).toString('base64');
    next();
});

app.use(helmet.contentSecurityPolicy({
    directives: {
        defaultSrc: ["'self'"],
        scriptSrc: [
            "'self'",
            (req, res) => `'nonce-${res.locals.nonce}'`
        ],
        styleSrc: [
            "'self'",
            (req, res) => `'nonce-${res.locals.nonce}'`
        ]
    }
}));

// In template/HTML
// <script nonce="{{nonce}}">
//     // This inline script will execute
// </script>

// ════════════════════════════════════════════════════════════════
// CSP REPORT-ONLY MODE (Test before enforcing)
// ════════════════════════════════════════════════════════════════

app.use(helmet.contentSecurityPolicy({
    directives: {
        // ... same directives
        reportUri: ['/csp-report']  // Deprecated but still works
        // Or use: reportTo: ['csp-endpoint']
    },
    reportOnly: true  // Don't block, just report violations
}));

// Receive violation reports
app.post('/csp-report', express.json({ type: 'application/csp-report' }), (req, res) => {
    console.log('CSP Violation:', req.body);
    // Log to your monitoring system
    res.status(204).end();
});

// ════════════════════════════════════════════════════════════════
// NEXT.JS CSP
// ════════════════════════════════════════════════════════════════

// next.config.js
const cspHeader = `
    default-src 'self';
    script-src 'self' 'nonce-{nonce}' 'strict-dynamic';
    style-src 'self' 'unsafe-inline';
    img-src 'self' blob: data:;
    font-src 'self';
    object-src 'none';
    base-uri 'self';
    form-action 'self';
    frame-ancestors 'none';
    upgrade-insecure-requests;
`;

module.exports = {
    async headers() {
        return [{
            source: '/(.*)',
            headers: [
                {
                    key: 'Content-Security-Policy',
                    value: cspHeader.replace(/\s{2,}/g, ' ').trim()
                }
            ]
        }];
    }
};
```

### HSTS (HTTP Strict Transport Security)

```typescript
// ════════════════════════════════════════════════════════════════
// HSTS - FORCE HTTPS
// ════════════════════════════════════════════════════════════════

app.use(helmet.hsts({
    maxAge: 63072000,      // 2 years in seconds
    includeSubDomains: true,
    preload: true          // For browser preload list submission
}));

// Or manual header
app.use((req, res, next) => {
    res.setHeader(
        'Strict-Transport-Security',
        'max-age=63072000; includeSubDomains; preload'
    );
    next();
});

/*
HSTS DEPLOYMENT STRATEGY:
1. Start with short max-age (5 minutes): max-age=300
2. Test everything works over HTTPS
3. Increase to 1 week: max-age=604800
4. Increase to 1 month: max-age=2592000
5. Add includeSubDomains (make sure ALL subdomains support HTTPS)
6. Increase to 2 years: max-age=63072000
7. Add preload and submit to https://hstspreload.org/
*/
```

### X-Frame-Options

```typescript
// ════════════════════════════════════════════════════════════════
// X-FRAME-OPTIONS - PREVENT CLICKJACKING
// ════════════════════════════════════════════════════════════════

// Prevent any framing
app.use(helmet.frameguard({ action: 'deny' }));
// Header: X-Frame-Options: DENY

// Allow framing from same origin only
app.use(helmet.frameguard({ action: 'sameorigin' }));
// Header: X-Frame-Options: SAMEORIGIN

// Modern alternative: CSP frame-ancestors
// Content-Security-Policy: frame-ancestors 'none';
// Content-Security-Policy: frame-ancestors 'self';
// Content-Security-Policy: frame-ancestors https://allowed.com;

// Note: CSP frame-ancestors is more flexible and preferred
// But X-Frame-Options still needed for older browsers
```

### X-Content-Type-Options

```typescript
// ════════════════════════════════════════════════════════════════
// X-CONTENT-TYPE-OPTIONS - PREVENT MIME SNIFFING
// ════════════════════════════════════════════════════════════════

app.use(helmet.noSniff());
// Header: X-Content-Type-Options: nosniff

/*
WHY THIS MATTERS:
Without this header, browsers might "sniff" file content and 
interpret a .txt file as JavaScript if it contains JS-like content.

Attack scenario:
1. Attacker uploads malicious.txt containing <script>alert('xss')</script>
2. Without nosniff, browser might execute it as script
3. With nosniff, browser respects Content-Type header
*/
```

### Referrer-Policy

```typescript
// ════════════════════════════════════════════════════════════════
// REFERRER-POLICY - CONTROL REFERRER INFORMATION
// ════════════════════════════════════════════════════════════════

app.use(helmet.referrerPolicy({
    policy: 'strict-origin-when-cross-origin'
}));

/*
REFERRER POLICY OPTIONS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  no-referrer                                                    │
│      → Never send referrer                                     │
│                                                                  │
│  no-referrer-when-downgrade (default)                          │
│      → Send for HTTPS→HTTPS, not HTTPS→HTTP                    │
│                                                                  │
│  same-origin                                                    │
│      → Only send for same-origin requests                      │
│                                                                  │
│  origin                                                         │
│      → Send origin only (not full path)                        │
│      → https://example.com (not /secret/page)                  │
│                                                                  │
│  strict-origin                                                  │
│      → Origin only, not on downgrade                           │
│                                                                  │
│  strict-origin-when-cross-origin (RECOMMENDED)                 │
│      → Full URL for same-origin                                │
│      → Origin only for cross-origin                            │
│      → Nothing on downgrade                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
*/
```

### Permissions-Policy

```typescript
// ════════════════════════════════════════════════════════════════
// PERMISSIONS-POLICY - CONTROL BROWSER FEATURES
// ════════════════════════════════════════════════════════════════

app.use(helmet.permittedCrossDomainPolicies());

// Custom permissions policy
app.use((req, res, next) => {
    res.setHeader('Permissions-Policy', [
        'camera=()',           // Disable camera
        'microphone=()',       // Disable microphone
        'geolocation=(self)',  // Allow geolocation only for self
        'payment=()',          // Disable payment API
        'usb=()',              // Disable USB
        'accelerometer=()',    // Disable motion sensors
        'gyroscope=()'
    ].join(', '));
    next();
});

/*
PERMISSIONS POLICY VALUES:
*           → Allow all
()          → Disable completely
(self)      → Allow only same origin
("https://example.com") → Allow specific origin
*/
```

### Complete Helmet Configuration

```typescript
// ════════════════════════════════════════════════════════════════
// COMPLETE HELMET SETUP
// ════════════════════════════════════════════════════════════════

import helmet from 'helmet';

app.use(helmet({
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            scriptSrc: ["'self'"],
            styleSrc: ["'self'", "'unsafe-inline'"],
            imgSrc: ["'self'", "data:", "https:"],
            connectSrc: ["'self'"],
            fontSrc: ["'self'"],
            objectSrc: ["'none'"],
            frameSrc: ["'none'"],
            frameAncestors: ["'none'"],
            baseUri: ["'self'"],
            formAction: ["'self'"],
            upgradeInsecureRequests: []
        }
    },
    hsts: {
        maxAge: 63072000,
        includeSubDomains: true,
        preload: true
    },
    frameguard: { action: 'deny' },
    referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
    noSniff: true,
    xssFilter: true,  // Legacy X-XSS-Protection
    hidePoweredBy: true
}));

// Remove X-Powered-By header (done by hidePoweredBy above, or:)
app.disable('x-powered-by');
```

---

## Nginx Configuration

```nginx
# ════════════════════════════════════════════════════════════════
# NGINX SECURITY HEADERS
# ════════════════════════════════════════════════════════════════

server {
    # ... server config ...
    
    # HSTS
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    
    # CSP
    add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; object-src 'none'; frame-ancestors 'none'; base-uri 'self'; form-action 'self';" always;
    
    # Prevent clickjacking
    add_header X-Frame-Options "DENY" always;
    
    # Prevent MIME type sniffing
    add_header X-Content-Type-Options "nosniff" always;
    
    # XSS protection (legacy)
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Referrer policy
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    
    # Permissions policy
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=(self)" always;
    
    # Hide server version
    server_tokens off;
}
```

---

## Interview Questions

**Q: "What is CSP and how does it prevent XSS?"**
> "Content-Security-Policy tells the browser what sources are allowed to load scripts, styles, images, etc. By default, only 'self' (same origin) is allowed. Even if an attacker injects a script tag, if the source isn't allowed by CSP, the browser blocks it. For inline scripts, we use nonces or hashes - the browser only executes scripts with matching nonce/hash."

**Q: "How do you implement CSP without breaking the site?"**
> "Start with report-only mode - same policy but violations are reported, not blocked. Set up a reporting endpoint. Run for a week or two, collect violation reports, identify legitimate resources being blocked, add them to the policy. Once clean, switch from report-only to enforced. Keep the reporting for ongoing monitoring."

**Q: "What is HSTS preloading?"**
> "HSTS preloading is submitting your domain to browser vendors' built-in HSTS list. Once on the list, browsers will never make an HTTP request to your domain - HTTPS is enforced before any request. Requirements: valid HTTPS on root and all subdomains, HSTS header with max-age ≥ 1 year, includeSubDomains, and preload directive. Submit at hstspreload.org. Be careful - removal from the list takes months."

---

## Quick Reference

```
SECURITY HEADERS CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  MUST HAVE:                                                     │
│  □ Content-Security-Policy (start with report-only)            │
│  □ Strict-Transport-Security (max-age=63072000)                │
│  □ X-Content-Type-Options: nosniff                             │
│  □ X-Frame-Options: DENY (or SAMEORIGIN if framing needed)     │
│                                                                  │
│  SHOULD HAVE:                                                   │
│  □ Referrer-Policy: strict-origin-when-cross-origin            │
│  □ Permissions-Policy (disable unused features)                │
│  □ Remove X-Powered-By header                                  │
│  □ Remove Server version                                       │
│                                                                  │
│  TEST YOUR HEADERS:                                             │
│  • https://securityheaders.com/                                │
│  • https://observatory.mozilla.org/                            │
│                                                                  │
│  CSP GOTCHAS:                                                   │
│  • 'unsafe-inline' defeats XSS protection                      │
│  • 'unsafe-eval' allows eval() - avoid if possible             │
│  • Third-party scripts may break - audit them                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


