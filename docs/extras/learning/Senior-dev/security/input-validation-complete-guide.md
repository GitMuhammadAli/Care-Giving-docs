# ✅ Input Validation & Sanitization - Complete Guide

> A comprehensive guide to input validation and sanitization - schema validation, escaping, allowlists, blocklists, and protecting against injection attacks.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Validation checks that input conforms to expected format and rules; sanitization transforms input to make it safe - both are essential because all user input is untrusted, and the boundary between your code and the outside world is your primary defense line."

### Validation vs Sanitization
```
VALIDATION VS SANITIZATION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  VALIDATION (Accept or Reject)                                  │
│  ─────────────────────────────                                  │
│  • Is email format valid?                                      │
│  • Is age a number between 1-150?                              │
│  • Is password at least 8 characters?                          │
│  → REJECT invalid input, return error                          │
│                                                                  │
│  SANITIZATION (Transform to make safe)                          │
│  ─────────────────────────────────────                          │
│  • Escape HTML characters: < → &lt;                            │
│  • Remove script tags from rich text                           │
│  • Trim whitespace                                             │
│  → TRANSFORM input before using                                │
│                                                                  │
│  WHEN TO USE WHICH:                                             │
│  • Structured data (email, number) → Validate                  │
│  • Free-form text displayed as HTML → Sanitize                 │
│  • Both when appropriate                                       │
│                                                                  │
│  ORDER OF OPERATIONS:                                           │
│  1. Decode (handle encoding)                                   │
│  2. Validate (check format/rules)                              │
│  3. Sanitize (if needed)                                       │
│  4. Use (in DB, display, etc.)                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement
> "I implemented a validation strategy based on 'trust boundaries'. All external input (HTTP requests, file uploads, webhooks) passes through a validation layer before entering the system. We use Zod schemas for type-safe runtime validation, DOMPurify for user-generated HTML, parameterized queries to prevent SQL injection, and Content Security Policy as a last-resort defense. The key insight: validation at entry points, encoding at output points, and never trust data just because it's in your database - it came from somewhere."

---

## 📚 Core Concepts

### Schema Validation with Zod

```typescript
// ════════════════════════════════════════════════════════════════
// COMPREHENSIVE VALIDATION WITH ZOD
// ════════════════════════════════════════════════════════════════

import { z } from 'zod';

// String validations
const emailSchema = z.string()
    .email('Invalid email format')
    .max(255, 'Email too long')
    .toLowerCase();  // Normalize

const usernameSchema = z.string()
    .min(3, 'Username too short')
    .max(30, 'Username too long')
    .regex(/^[a-zA-Z0-9_]+$/, 'Only alphanumeric and underscore allowed');

const passwordSchema = z.string()
    .min(8, 'Password must be at least 8 characters')
    .max(100, 'Password too long')
    .regex(/[A-Z]/, 'Must contain uppercase')
    .regex(/[a-z]/, 'Must contain lowercase')
    .regex(/[0-9]/, 'Must contain number')
    .regex(/[^A-Za-z0-9]/, 'Must contain special character');

// Number validations
const ageSchema = z.number()
    .int('Must be whole number')
    .min(13, 'Must be at least 13')
    .max(120, 'Invalid age');

const priceSchema = z.number()
    .positive('Must be positive')
    .multipleOf(0.01, 'Max 2 decimal places');

// Complex object validation
const userSchema = z.object({
    email: emailSchema,
    username: usernameSchema,
    password: passwordSchema,
    age: ageSchema.optional(),
    profile: z.object({
        bio: z.string().max(500).optional(),
        website: z.string().url().optional()
    }).optional(),
    role: z.enum(['user', 'admin', 'moderator']).default('user'),
    tags: z.array(z.string()).max(10).optional()
});

// Array validation
const idsSchema = z.array(z.string().uuid()).min(1).max(100);

// Union types
const idSchema = z.union([
    z.string().uuid(),
    z.number().int().positive()
]);

// Discriminated unions
const notificationSchema = z.discriminatedUnion('type', [
    z.object({ type: z.literal('email'), email: z.string().email() }),
    z.object({ type: z.literal('sms'), phone: z.string() }),
    z.object({ type: z.literal('push'), deviceId: z.string() })
]);

// ════════════════════════════════════════════════════════════════
// VALIDATION MIDDLEWARE
// ════════════════════════════════════════════════════════════════

function validateBody<T extends z.ZodSchema>(schema: T) {
    return (req: Request, res: Response, next: NextFunction) => {
        const result = schema.safeParse(req.body);
        
        if (!result.success) {
            return res.status(400).json({
                error: 'Validation failed',
                details: result.error.issues.map(issue => ({
                    path: issue.path.join('.'),
                    message: issue.message
                }))
            });
        }
        
        req.body = result.data;  // Use parsed/transformed data
        next();
    };
}

function validateQuery<T extends z.ZodSchema>(schema: T) {
    return (req: Request, res: Response, next: NextFunction) => {
        const result = schema.safeParse(req.query);
        if (!result.success) {
            return res.status(400).json({ error: 'Invalid query parameters' });
        }
        req.query = result.data;
        next();
    };
}

// Usage
app.post('/api/users', validateBody(userSchema), createUser);
```

### HTML Sanitization

```typescript
// ════════════════════════════════════════════════════════════════
// HTML SANITIZATION WITH DOMPURIFY
// ════════════════════════════════════════════════════════════════

import DOMPurify from 'dompurify';
import { JSDOM } from 'jsdom';

const window = new JSDOM('').window;
const purify = DOMPurify(window);

// Basic sanitization - removes scripts, event handlers
function sanitizeHTML(dirty: string): string {
    return purify.sanitize(dirty);
}

// Custom allowed tags
function sanitizeRichText(dirty: string): string {
    return purify.sanitize(dirty, {
        ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br', 'ul', 'ol', 'li'],
        ALLOWED_ATTR: ['href', 'target'],
        ALLOW_DATA_ATTR: false
    });
}

// Strict - text only
function stripHTML(dirty: string): string {
    return purify.sanitize(dirty, { ALLOWED_TAGS: [] });
}

// ════════════════════════════════════════════════════════════════
// CONTEXT-AWARE ENCODING
// ════════════════════════════════════════════════════════════════

// HTML context
function escapeHTML(str: string): string {
    return str
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&#39;');
}

// JavaScript string context
function escapeJS(str: string): string {
    return str
        .replace(/\\/g, '\\\\')
        .replace(/'/g, "\\'")
        .replace(/"/g, '\\"')
        .replace(/\n/g, '\\n')
        .replace(/\r/g, '\\r');
}

// URL parameter context
function escapeURL(str: string): string {
    return encodeURIComponent(str);
}

// CSS context
function escapeCSS(str: string): string {
    return str.replace(/[^a-zA-Z0-9]/g, char => 
        '\\' + char.charCodeAt(0).toString(16) + ' '
    );
}
```

### Allowlist vs Blocklist

```typescript
// ════════════════════════════════════════════════════════════════
// ALLOWLIST (WHITELIST) - PREFERRED
// ════════════════════════════════════════════════════════════════

// ✅ ALLOWLIST: Define what IS allowed
const ALLOWED_FILE_TYPES = ['image/jpeg', 'image/png', 'image/gif'];
const ALLOWED_EXTENSIONS = ['.jpg', '.jpeg', '.png', '.gif'];

function validateFileUpload(file: Express.Multer.File) {
    // Check MIME type
    if (!ALLOWED_FILE_TYPES.includes(file.mimetype)) {
        throw new Error('Invalid file type');
    }
    
    // Check extension
    const ext = path.extname(file.originalname).toLowerCase();
    if (!ALLOWED_EXTENSIONS.includes(ext)) {
        throw new Error('Invalid file extension');
    }
    
    // Check magic bytes (file signature)
    const buffer = fs.readFileSync(file.path);
    const fileType = await fileTypeFromBuffer(buffer);
    if (!ALLOWED_FILE_TYPES.includes(fileType?.mime)) {
        throw new Error('File content does not match type');
    }
}

// ❌ BLOCKLIST: Define what is NOT allowed
// Easy to bypass - attacker finds something you didn't block
const BLOCKED_EXTENSIONS = ['.exe', '.bat', '.sh'];  // What about .cmd, .ps1?

// ════════════════════════════════════════════════════════════════
// ALLOWLIST FOR REDIRECTS (Prevent Open Redirect)
// ════════════════════════════════════════════════════════════════

const ALLOWED_REDIRECT_DOMAINS = [
    'myapp.com',
    'www.myapp.com',
    'auth.myapp.com'
];

function validateRedirect(url: string): boolean {
    try {
        const parsed = new URL(url);
        return ALLOWED_REDIRECT_DOMAINS.includes(parsed.hostname);
    } catch {
        return false;  // Invalid URL
    }
}

// Usage
app.get('/redirect', (req, res) => {
    const { url } = req.query;
    
    if (!validateRedirect(url)) {
        return res.status(400).json({ error: 'Invalid redirect URL' });
    }
    
    res.redirect(url);
});

// ════════════════════════════════════════════════════════════════
// ALLOWLIST FOR SQL COLUMN NAMES
// ════════════════════════════════════════════════════════════════

const ALLOWED_SORT_COLUMNS = ['name', 'created_at', 'updated_at', 'email'];
const ALLOWED_SORT_ORDERS = ['asc', 'desc'];

function buildOrderClause(column: string, order: string): string {
    // Validate against allowlist
    if (!ALLOWED_SORT_COLUMNS.includes(column)) {
        throw new Error('Invalid sort column');
    }
    if (!ALLOWED_SORT_ORDERS.includes(order.toLowerCase())) {
        throw new Error('Invalid sort order');
    }
    
    return `ORDER BY ${column} ${order}`;  // Safe because validated
}
```

### Common Validation Patterns

```typescript
// ════════════════════════════════════════════════════════════════
// COMMON VALIDATION PATTERNS
// ════════════════════════════════════════════════════════════════

// UUID validation
const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i;

// Phone number (international)
const phoneRegex = /^\+?[1-9]\d{1,14}$/;  // E.164 format

// Credit card (Luhn check)
function isValidCreditCard(number: string): boolean {
    const digits = number.replace(/\D/g, '');
    let sum = 0;
    let isEven = false;
    
    for (let i = digits.length - 1; i >= 0; i--) {
        let digit = parseInt(digits[i], 10);
        
        if (isEven) {
            digit *= 2;
            if (digit > 9) digit -= 9;
        }
        
        sum += digit;
        isEven = !isEven;
    }
    
    return sum % 10 === 0;
}

// Slug validation
const slugRegex = /^[a-z0-9]+(?:-[a-z0-9]+)*$/;

// Path traversal prevention
function sanitizePath(userPath: string): string {
    // Remove path traversal attempts
    const normalized = path.normalize(userPath);
    
    // Ensure it doesn't escape the base directory
    const basePath = '/uploads';
    const fullPath = path.join(basePath, normalized);
    
    if (!fullPath.startsWith(basePath)) {
        throw new Error('Path traversal detected');
    }
    
    return fullPath;
}

// JSON depth limit (prevent DoS)
function parseJSONSafe(json: string, maxDepth = 10): any {
    let depth = 0;
    
    return JSON.parse(json, (key, value) => {
        if (typeof value === 'object' && value !== null) {
            depth++;
            if (depth > maxDepth) {
                throw new Error('JSON too deeply nested');
            }
        }
        return value;
    });
}
```

---

## Validation Checklist by Input Type

```
INPUT VALIDATION CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  STRINGS:                                                       │
│  □ Min/max length                                              │
│  □ Character allowlist or regex                                │
│  □ Trim whitespace                                             │
│  □ Normalize (lowercase emails)                                │
│                                                                  │
│  NUMBERS:                                                       │
│  □ Type check (parseInt can fail)                              │
│  □ Min/max bounds                                              │
│  □ Integer vs decimal                                          │
│  □ NaN, Infinity checks                                        │
│                                                                  │
│  ARRAYS:                                                        │
│  □ Max length                                                  │
│  □ Validate each item                                          │
│  □ Unique values if needed                                     │
│                                                                  │
│  FILES:                                                         │
│  □ Extension allowlist                                         │
│  □ MIME type check                                             │
│  □ Magic bytes verification                                    │
│  □ Size limit                                                  │
│  □ Filename sanitization                                       │
│                                                                  │
│  URLS:                                                          │
│  □ Protocol allowlist (https only?)                            │
│  □ Domain allowlist for redirects                              │
│  □ No javascript: or data: URLs                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "Validation vs sanitization - what's the difference?"**
> "Validation checks if input meets criteria and rejects if not - 'is this a valid email?'. Sanitization transforms input to make it safe - 'remove scripts from this HTML'. Use validation for structured data (emails, numbers), sanitization for free-form content that will be rendered. Often use both: validate format, then sanitize content."

**Q: "Allowlist vs blocklist?"**
> "Always prefer allowlist - define what IS allowed. Blocklists fail because you can't anticipate all malicious inputs. Example: blocking .exe, .bat extensions but missing .cmd, .ps1. With allowlist: only .jpg, .png allowed - anything else rejected by default. Allowlist is default-deny, blocklist is default-allow."

**Q: "How do you prevent injection attacks?"**
> "Never trust user input. For SQL: parameterized queries, never string concatenation. For XSS: escape output in the right context (HTML, JS, URL), use CSP headers. For command injection: avoid shell execution, use execFile with args array. For path traversal: normalize and validate against base directory. General: validate at entry, encode at output."

---

## Quick Reference

```
VALIDATION CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  TRUST BOUNDARIES:                                              │
│  • HTTP requests (headers, body, query, params)                │
│  • File uploads                                                │
│  • Webhooks from third parties                                 │
│  • Database data (it came from somewhere!)                     │
│                                                                  │
│  VALIDATION STRATEGY:                                           │
│  1. Parse: Convert to expected type                            │
│  2. Validate: Check against rules                              │
│  3. Sanitize: Remove/escape dangerous content                  │
│  4. Use: Now safe to process                                   │
│                                                                  │
│  ENCODING BY CONTEXT:                                           │
│  • HTML body: &lt; &gt; &amp; &quot;                           │
│  • HTML attribute: &quot; &#39;                                │
│  • JavaScript: \' \" \\                                        │
│  • URL parameter: encodeURIComponent                           │
│  • CSS: escape non-alphanumeric                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


