# 🔑 Password Security - Complete Guide

> A comprehensive guide to password security - hashing algorithms, bcrypt, Argon2, password policies, and protecting user credentials.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Password security means NEVER storing passwords in plain text - use slow, memory-hard hashing algorithms like Argon2id or bcrypt with unique salts per password, combined with sensible policies that encourage length over complexity."

### Password Hashing Overview
```
PASSWORD HASHING EVOLUTION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ❌ PLAIN TEXT                                                   │
│     password → password                                        │
│     Never. Just never.                                         │
│                                                                  │
│  ❌ FAST HASH (MD5, SHA1, SHA256)                               │
│     password → 5f4dcc3b5aa765d61d8327deb882cf99                │
│     Too fast! Billions of guesses per second.                  │
│                                                                  │
│  ⚠️  SLOW HASH WITHOUT SALT                                     │
│     password → $2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/X    │
│     Vulnerable to rainbow tables.                              │
│                                                                  │
│  ✅ BCRYPT (Good)                                                │
│     password + random_salt → $2b$12$abc...                     │
│     Intentionally slow, unique salt per password.              │
│                                                                  │
│  ✅✅ ARGON2ID (Best)                                            │
│     password + salt → $argon2id$v=19$m=65536,t=3,p=4$...       │
│     Memory-hard, resistant to GPU/ASIC attacks.                │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  WHY SLOW MATTERS:                                              │
│                                                                  │
│  SHA256 (fast):     10 billion hashes/second on GPU            │
│  bcrypt (slow):     ~10,000 hashes/second on GPU               │
│  Argon2id (memory): ~100 hashes/second (memory-bound)          │
│                                                                  │
│  For 8-char password: SHA256 = seconds, Argon2 = years         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement
> "I migrated our password storage from bcrypt to Argon2id with 64MB memory cost, 3 iterations, and 4 parallel threads. This makes GPU-based attacks ~1000x more expensive than bcrypt alone because attackers need dedicated memory per parallel attempt. We implemented transparent re-hashing: on login, if the hash is bcrypt, we verify then re-hash with Argon2. For password policy, we dropped complex character requirements (users just add '1!' to weak passwords) and require 12+ characters instead - NIST guidelines show length beats complexity. We also check against HaveIBeenPwned's breached password database."

---

## 📚 Core Implementation

### Argon2 (Recommended)

```typescript
// ════════════════════════════════════════════════════════════════
// ARGON2ID - BEST CHOICE FOR PASSWORD HASHING
// ════════════════════════════════════════════════════════════════

import argon2 from 'argon2';

// Configuration (adjust based on your server's resources)
const ARGON2_CONFIG = {
    type: argon2.argon2id,  // Hybrid: resistant to side-channel + GPU attacks
    memoryCost: 65536,      // 64 MB (2^16 KB)
    timeCost: 3,            // Number of iterations
    parallelism: 4,         // Threads to use
    hashLength: 32          // Output hash length
};

// Hash a password
async function hashPassword(password: string): Promise<string> {
    return argon2.hash(password, ARGON2_CONFIG);
}
// Returns: $argon2id$v=19$m=65536,t=3,p=4$Zm9vYmFy$...

// Verify a password
async function verifyPassword(password: string, hash: string): Promise<boolean> {
    try {
        return await argon2.verify(hash, password);
    } catch (error) {
        // Invalid hash format or other error
        console.error('Password verification error:', error);
        return false;
    }
}

// Check if hash needs upgrade (after config change)
async function needsRehash(hash: string): Promise<boolean> {
    return argon2.needsRehash(hash, ARGON2_CONFIG);
}

// ════════════════════════════════════════════════════════════════
// LOGIN WITH TRANSPARENT REHASH
// ════════════════════════════════════════════════════════════════

async function login(email: string, password: string) {
    const user = await db.users.findUnique({ where: { email } });
    if (!user) {
        // Constant time comparison to prevent timing attacks
        await argon2.hash(password, ARGON2_CONFIG);
        return null;
    }
    
    // Verify password
    const valid = await verifyPassword(password, user.passwordHash);
    if (!valid) {
        return null;
    }
    
    // Check if hash needs upgrade
    if (await needsRehash(user.passwordHash)) {
        const newHash = await hashPassword(password);
        await db.users.update({
            where: { id: user.id },
            data: { passwordHash: newHash }
        });
    }
    
    return user;
}
```

### Bcrypt (Alternative)

```typescript
// ════════════════════════════════════════════════════════════════
// BCRYPT - STILL WIDELY USED AND SECURE
// ════════════════════════════════════════════════════════════════

import bcrypt from 'bcrypt';

// Work factor (cost) - adjust for ~250ms hash time
const SALT_ROUNDS = 12;  // 2^12 iterations

// Hash a password
async function hashPassword(password: string): Promise<string> {
    return bcrypt.hash(password, SALT_ROUNDS);
}
// Returns: $2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/X.OS4aDLJ5JLQzD2

// Verify a password
async function verifyPassword(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
}

// ════════════════════════════════════════════════════════════════
// CHOOSING WORK FACTOR
// ════════════════════════════════════════════════════════════════

// Target: ~250ms on your production hardware
async function calibrateWorkFactor() {
    const testPassword = 'test_password_123';
    
    for (let rounds = 10; rounds <= 16; rounds++) {
        const start = Date.now();
        await bcrypt.hash(testPassword, rounds);
        const duration = Date.now() - start;
        
        console.log(`Rounds: ${rounds}, Time: ${duration}ms`);
        // Use the highest rounds that stays under 500ms
    }
}
// Typical results (2024 hardware):
// Rounds: 10, Time: 65ms
// Rounds: 11, Time: 130ms
// Rounds: 12, Time: 260ms  ← Good choice
// Rounds: 13, Time: 520ms
```

### Password Policies (NIST Guidelines)

```typescript
// ════════════════════════════════════════════════════════════════
// MODERN PASSWORD POLICY (NIST SP 800-63B)
// ════════════════════════════════════════════════════════════════

import { z } from 'zod';

/*
NIST GUIDELINES (simplified):
✓ Minimum 8 characters (12+ recommended)
✓ Maximum 64+ characters (allow passphrases)
✓ Allow all printable characters, spaces, Unicode
✗ DON'T require complexity (upper, lower, number, special)
✗ DON'T use password hints
✗ DON'T force periodic rotation (causes worse passwords)
✓ Check against breached password lists
✓ Check against common/dictionary passwords
*/

const passwordSchema = z.string()
    .min(12, 'Password must be at least 12 characters')
    .max(128, 'Password must be less than 128 characters')
    // Optional: check for all spaces (usually a mistake)
    .refine(pw => pw.trim().length > 0, 'Password cannot be only spaces');

// ════════════════════════════════════════════════════════════════
// BREACHED PASSWORD CHECK (HaveIBeenPwned)
// ════════════════════════════════════════════════════════════════

import crypto from 'crypto';

async function isPasswordBreached(password: string): Promise<boolean> {
    // Hash password with SHA1
    const hash = crypto.createHash('sha1')
        .update(password)
        .digest('hex')
        .toUpperCase();
    
    // k-anonymity: only send first 5 characters
    const prefix = hash.substring(0, 5);
    const suffix = hash.substring(5);
    
    // Query HaveIBeenPwned API
    const response = await fetch(
        `https://api.pwnedpasswords.com/range/${prefix}`
    );
    const text = await response.text();
    
    // Check if our suffix is in the results
    const lines = text.split('\n');
    for (const line of lines) {
        const [hashSuffix, count] = line.split(':');
        if (hashSuffix === suffix) {
            console.log(`Password found in ${count} breaches`);
            return true;
        }
    }
    
    return false;
}

// ════════════════════════════════════════════════════════════════
// COMMON PASSWORD CHECK
// ════════════════════════════════════════════════════════════════

// Load common passwords list (10k most common)
const COMMON_PASSWORDS = new Set([
    'password', '123456', 'qwerty', 'letmein', 
    'welcome', 'admin', 'iloveyou', 'monkey',
    // ... load from file
]);

function isCommonPassword(password: string): boolean {
    return COMMON_PASSWORDS.has(password.toLowerCase());
}

// ════════════════════════════════════════════════════════════════
// COMPLETE PASSWORD VALIDATION
// ════════════════════════════════════════════════════════════════

interface PasswordValidationResult {
    valid: boolean;
    errors: string[];
}

async function validatePassword(password: string): Promise<PasswordValidationResult> {
    const errors: string[] = [];
    
    // Length check
    if (password.length < 12) {
        errors.push('Password must be at least 12 characters');
    }
    
    if (password.length > 128) {
        errors.push('Password must be less than 128 characters');
    }
    
    // Common password check
    if (isCommonPassword(password)) {
        errors.push('This password is too common');
    }
    
    // Breached password check
    if (errors.length === 0) {  // Only check if other validations pass
        const breached = await isPasswordBreached(password);
        if (breached) {
            errors.push('This password has been found in a data breach');
        }
    }
    
    return {
        valid: errors.length === 0,
        errors
    };
}
```

### Password Reset Flow

```typescript
// ════════════════════════════════════════════════════════════════
// SECURE PASSWORD RESET
// ════════════════════════════════════════════════════════════════

import crypto from 'crypto';

// Generate reset token
function generateResetToken(): { token: string; hash: string; expires: Date } {
    const token = crypto.randomBytes(32).toString('hex');
    const hash = crypto.createHash('sha256').update(token).digest('hex');
    const expires = new Date(Date.now() + 60 * 60 * 1000);  // 1 hour
    
    return { token, hash, expires };
}

// Request password reset
app.post('/api/forgot-password', async (req, res) => {
    const { email } = req.body;
    
    // Always return same response (prevent email enumeration)
    const genericResponse = { message: 'If email exists, reset link was sent' };
    
    const user = await db.users.findUnique({ where: { email } });
    if (!user) {
        // Simulate delay to prevent timing attacks
        await new Promise(r => setTimeout(r, 100 + Math.random() * 100));
        return res.json(genericResponse);
    }
    
    // Generate token
    const { token, hash, expires } = generateResetToken();
    
    // Store hash (not token!) in database
    await db.passwordResets.create({
        data: {
            userId: user.id,
            tokenHash: hash,
            expiresAt: expires
        }
    });
    
    // Send email with token
    await sendEmail(email, 'Password Reset', `
        Click to reset: https://myapp.com/reset-password?token=${token}
    `);
    
    res.json(genericResponse);
});

// Reset password with token
app.post('/api/reset-password', async (req, res) => {
    const { token, newPassword } = req.body;
    
    // Validate new password
    const validation = await validatePassword(newPassword);
    if (!validation.valid) {
        return res.status(400).json({ errors: validation.errors });
    }
    
    // Hash token and look up
    const tokenHash = crypto.createHash('sha256').update(token).digest('hex');
    
    const reset = await db.passwordResets.findFirst({
        where: {
            tokenHash,
            expiresAt: { gt: new Date() },
            usedAt: null
        }
    });
    
    if (!reset) {
        return res.status(400).json({ error: 'Invalid or expired reset token' });
    }
    
    // Hash new password
    const passwordHash = await hashPassword(newPassword);
    
    // Update password and mark token as used
    await db.$transaction([
        db.users.update({
            where: { id: reset.userId },
            data: { passwordHash }
        }),
        db.passwordResets.update({
            where: { id: reset.id },
            data: { usedAt: new Date() }
        })
    ]);
    
    // Invalidate all sessions
    await invalidateAllSessions(reset.userId);
    
    res.json({ message: 'Password updated successfully' });
});
```

---

## Password Strength Estimation

```typescript
// ════════════════════════════════════════════════════════════════
// PASSWORD STRENGTH METER (Client-side)
// ════════════════════════════════════════════════════════════════

import zxcvbn from 'zxcvbn';  // Dropbox's password strength estimator

function checkPasswordStrength(password: string, userInputs: string[] = []) {
    // userInputs: email, username, etc. to check for
    const result = zxcvbn(password, userInputs);
    
    return {
        score: result.score,  // 0-4 (0=very weak, 4=strong)
        crackTime: result.crack_times_display.offline_fast_hashing_1e10_per_second,
        feedback: result.feedback.suggestions,
        warning: result.feedback.warning
    };
}

/*
ZXCVBN SCORES:
0 - Very weak (instant to crack)
1 - Weak (less than 1 hour)
2 - Fair (less than 1 day)
3 - Strong (less than 1 month)
4 - Very strong (centuries)
*/

// Usage in React
function PasswordInput() {
    const [password, setPassword] = useState('');
    const [strength, setStrength] = useState(null);
    
    const handleChange = (e) => {
        const value = e.target.value;
        setPassword(value);
        
        if (value.length > 0) {
            setStrength(checkPasswordStrength(value));
        }
    };
    
    return (
        <div>
            <input 
                type="password" 
                value={password} 
                onChange={handleChange}
            />
            {strength && (
                <div>
                    <progress value={strength.score} max={4} />
                    <p>Time to crack: {strength.crackTime}</p>
                    {strength.feedback.map(tip => <p key={tip}>{tip}</p>)}
                </div>
            )}
        </div>
    );
}
```

---

## Interview Questions

**Q: "Why bcrypt/Argon2 instead of SHA256 for passwords?"**
> "SHA256 is fast - that's bad for passwords. Attackers with GPUs can try billions of SHA256 hashes per second. Bcrypt and Argon2 are intentionally slow (configurable work factor), making brute force attacks impractical. Argon2 is even better because it's memory-hard - attackers need dedicated RAM for each parallel attempt, making GPU/ASIC attacks much more expensive."

**Q: "What's the purpose of a salt?"**
> "A salt is random data added to each password before hashing. Without salt, identical passwords produce identical hashes - attackers can use precomputed 'rainbow tables'. With unique salts, each password must be cracked individually. Both bcrypt and Argon2 generate and embed salts automatically - you don't need to manage them separately."

**Q: "What password policy do you recommend?"**
> "Follow NIST guidelines: minimum 12 characters, allow long passphrases (64+ chars), check against breached password lists (HaveIBeenPwned), reject common passwords. Avoid forced complexity rules (users just add '1!' to bad passwords) and periodic rotation (causes weaker passwords). Encourage password managers and offer 2FA."

---

## Quick Reference

```
PASSWORD SECURITY CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  HASHING:                                                       │
│  □ Use Argon2id (preferred) or bcrypt                          │
│  □ Never use MD5, SHA1, SHA256 for passwords                   │
│  □ Configure work factor for ~250ms hash time                  │
│  □ Re-hash passwords when upgrading algorithm                  │
│                                                                  │
│  POLICY:                                                        │
│  □ Minimum 12 characters                                       │
│  □ Maximum 64+ characters                                      │
│  □ Check against breached passwords (HIBP)                     │
│  □ Block common passwords                                      │
│  □ Show strength meter                                         │
│  □ Don't force complexity rules                                │
│                                                                  │
│  RESET:                                                         │
│  □ Cryptographically random token                              │
│  □ Store hash, not token                                       │
│  □ Short expiry (1 hour)                                       │
│  □ Single use                                                  │
│  □ Invalidate sessions after change                            │
│                                                                  │
│  NEVER:                                                         │
│  ✗ Store plain text passwords                                  │
│  ✗ Use fast hashing algorithms                                 │
│  ✗ Email passwords in plain text                               │
│  ✗ Use password hints                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


