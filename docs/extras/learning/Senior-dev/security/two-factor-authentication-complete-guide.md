# 🔐 Two-Factor Authentication - Complete Guide

> A comprehensive guide to two-factor authentication - TOTP, WebAuthn, backup codes, and implementing MFA correctly.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Two-factor authentication (2FA) requires two different types of proof: something you know (password), something you have (phone/hardware key), or something you are (biometrics) - it means that even if an attacker steals your password, they can't access your account without the second factor."

### 2FA Methods Comparison
```
2FA METHODS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SMS OTP ⚠️                                                     │
│  • Code sent via text message                                  │
│  • Vulnerable to SIM swapping                                  │
│  • Better than nothing, but not recommended                    │
│                                                                  │
│  TOTP (Time-based OTP) ✅                                       │
│  • Google Authenticator, Authy, 1Password                      │
│  • 6-digit code changes every 30 seconds                       │
│  • Secret stored on user's device                              │
│  • Most common, good security                                  │
│                                                                  │
│  WebAuthn / Passkeys ✅✅                                        │
│  • Hardware keys (YubiKey), biometrics                         │
│  • Phishing-resistant                                          │
│  • Best security, best UX                                      │
│                                                                  │
│  Email OTP ⚠️                                                   │
│  • Code sent via email                                         │
│  • Email often already compromised                             │
│  • Acceptable for low-risk apps                                │
│                                                                  │
│  Push Notification ✅                                           │
│  • Approve login on registered device                          │
│  • Good UX, good security                                      │
│  • Requires dedicated app                                      │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  BACKUP CODES (Recovery)                                        │
│  • One-time use codes for account recovery                     │
│  • Essential for when device is lost                           │
│  • Usually 8-10 codes, single use each                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement
> "I implemented a multi-layered 2FA system. Primary method is TOTP with Google Authenticator compatibility - we generate a secret, show a QR code, and verify setup with a confirmation code. We also support WebAuthn for hardware keys, which is phishing-resistant because the browser verifies the origin. Every user gets 10 backup codes stored hashed (like passwords). For recovery, we require identity verification through support. We track 2FA adoption in metrics and gently prompt users who haven't enabled it, especially after security events. Admin accounts require hardware keys (WebAuthn) - TOTP alone isn't sufficient for high-privilege access."

---

## 📚 Core Implementation

### TOTP (Time-based One-Time Password)

```typescript
// ════════════════════════════════════════════════════════════════
// TOTP IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

import { authenticator } from 'otplib';
import qrcode from 'qrcode';

// Configure TOTP
authenticator.options = {
    digits: 6,        // 6-digit code
    step: 30,         // 30-second window
    window: 1         // Allow 1 window before/after (drift tolerance)
};

// ════════════════════════════════════════════════════════════════
// SETUP 2FA
// ════════════════════════════════════════════════════════════════

interface TwoFactorSetup {
    secret: string;
    otpauthUrl: string;
    qrCodeDataUrl: string;
    backupCodes: string[];
}

async function initiate2FASetup(userId: string, email: string): Promise<TwoFactorSetup> {
    // Generate secret
    const secret = authenticator.generateSecret();
    
    // Create otpauth URL (for QR code)
    const otpauthUrl = authenticator.keyuri(
        email,
        'MyApp',  // App name shown in authenticator
        secret
    );
    
    // Generate QR code
    const qrCodeDataUrl = await qrcode.toDataURL(otpauthUrl);
    
    // Generate backup codes
    const backupCodes = await generateBackupCodes(10);
    
    // Store secret temporarily (not yet verified)
    await db.twoFactorSetup.upsert({
        where: { userId },
        create: {
            userId,
            secret,
            backupCodes: await hashBackupCodes(backupCodes)
        },
        update: {
            secret,
            backupCodes: await hashBackupCodes(backupCodes)
        }
    });
    
    return {
        secret,
        otpauthUrl,
        qrCodeDataUrl,
        backupCodes  // Show once to user, they must save them
    };
}

// ════════════════════════════════════════════════════════════════
// VERIFY AND ENABLE 2FA
// ════════════════════════════════════════════════════════════════

async function verify2FASetup(userId: string, code: string): Promise<boolean> {
    const setup = await db.twoFactorSetup.findUnique({ where: { userId } });
    if (!setup) {
        throw new Error('2FA setup not initiated');
    }
    
    // Verify the code
    const isValid = authenticator.verify({
        token: code,
        secret: setup.secret
    });
    
    if (!isValid) {
        return false;
    }
    
    // Enable 2FA on user account
    await db.$transaction([
        db.users.update({
            where: { id: userId },
            data: {
                twoFactorEnabled: true,
                twoFactorSecret: setup.secret,
                backupCodes: setup.backupCodes
            }
        }),
        db.twoFactorSetup.delete({ where: { userId } })
    ]);
    
    return true;
}

// ════════════════════════════════════════════════════════════════
// VERIFY 2FA DURING LOGIN
// ════════════════════════════════════════════════════════════════

async function verifyTOTP(userId: string, code: string): Promise<boolean> {
    const user = await db.users.findUnique({
        where: { id: userId },
        select: { twoFactorSecret: true }
    });
    
    if (!user?.twoFactorSecret) {
        throw new Error('2FA not enabled');
    }
    
    // Verify TOTP code
    return authenticator.verify({
        token: code,
        secret: user.twoFactorSecret
    });
}
```

### Backup Codes

```typescript
// ════════════════════════════════════════════════════════════════
// BACKUP CODES
// ════════════════════════════════════════════════════════════════

import crypto from 'crypto';
import bcrypt from 'bcrypt';

// Generate backup codes
async function generateBackupCodes(count: number = 10): Promise<string[]> {
    const codes: string[] = [];
    
    for (let i = 0; i < count; i++) {
        // Format: XXXX-XXXX (8 alphanumeric characters)
        const code = crypto.randomBytes(4).toString('hex').toUpperCase();
        codes.push(`${code.slice(0, 4)}-${code.slice(4)}`);
    }
    
    return codes;
}

// Hash backup codes for storage (like passwords)
async function hashBackupCodes(codes: string[]): Promise<string[]> {
    return Promise.all(codes.map(code => bcrypt.hash(code, 10)));
}

// Verify and consume a backup code
async function verifyBackupCode(userId: string, code: string): Promise<boolean> {
    const user = await db.users.findUnique({
        where: { id: userId },
        select: { backupCodes: true }
    });
    
    if (!user?.backupCodes || user.backupCodes.length === 0) {
        return false;
    }
    
    // Normalize input (remove dashes, uppercase)
    const normalizedCode = code.replace(/-/g, '').toUpperCase();
    
    // Check each stored hash
    for (let i = 0; i < user.backupCodes.length; i++) {
        const hashedCode = user.backupCodes[i];
        const normalizedStored = `${normalizedCode.slice(0, 4)}-${normalizedCode.slice(4)}`;
        
        if (await bcrypt.compare(normalizedStored, hashedCode)) {
            // Remove used code
            const remainingCodes = [...user.backupCodes];
            remainingCodes.splice(i, 1);
            
            await db.users.update({
                where: { id: userId },
                data: { backupCodes: remainingCodes }
            });
            
            // Log backup code usage
            console.log('Backup code used:', { userId, remainingCount: remainingCodes.length });
            
            return true;
        }
    }
    
    return false;
}

// Regenerate backup codes
async function regenerateBackupCodes(userId: string): Promise<string[]> {
    const codes = await generateBackupCodes(10);
    
    await db.users.update({
        where: { id: userId },
        data: { backupCodes: await hashBackupCodes(codes) }
    });
    
    return codes;  // Show to user once
}
```

### WebAuthn / Passkeys

```typescript
// ════════════════════════════════════════════════════════════════
// WEBAUTHN IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

import {
    generateRegistrationOptions,
    verifyRegistrationResponse,
    generateAuthenticationOptions,
    verifyAuthenticationResponse
} from '@simplewebauthn/server';

const rpName = 'MyApp';
const rpID = 'myapp.com';  // Your domain
const origin = `https://${rpID}`;

// ════════════════════════════════════════════════════════════════
// REGISTRATION (Adding a security key)
// ════════════════════════════════════════════════════════════════

// Step 1: Generate registration options
app.post('/api/webauthn/register/options', authenticate, async (req, res) => {
    const user = req.user;
    
    // Get existing authenticators to exclude
    const existingAuthenticators = await db.authenticators.findMany({
        where: { userId: user.id }
    });
    
    const options = await generateRegistrationOptions({
        rpName,
        rpID,
        userID: user.id,
        userName: user.email,
        userDisplayName: user.name,
        attestationType: 'none',  // Don't need attestation for most apps
        excludeCredentials: existingAuthenticators.map(auth => ({
            id: Buffer.from(auth.credentialID, 'base64'),
            type: 'public-key',
            transports: auth.transports
        })),
        authenticatorSelection: {
            residentKey: 'preferred',
            userVerification: 'preferred'
        }
    });
    
    // Store challenge for verification
    await db.webauthnChallenges.upsert({
        where: { userId: user.id },
        create: { userId: user.id, challenge: options.challenge },
        update: { challenge: options.challenge }
    });
    
    res.json(options);
});

// Step 2: Verify registration response
app.post('/api/webauthn/register/verify', authenticate, async (req, res) => {
    const { body } = req;
    const user = req.user;
    
    // Get expected challenge
    const challenge = await db.webauthnChallenges.findUnique({
        where: { userId: user.id }
    });
    
    if (!challenge) {
        return res.status(400).json({ error: 'Challenge not found' });
    }
    
    const verification = await verifyRegistrationResponse({
        response: body,
        expectedChallenge: challenge.challenge,
        expectedOrigin: origin,
        expectedRPID: rpID
    });
    
    if (!verification.verified || !verification.registrationInfo) {
        return res.status(400).json({ error: 'Verification failed' });
    }
    
    const { credentialPublicKey, credentialID, counter } = verification.registrationInfo;
    
    // Store authenticator
    await db.authenticators.create({
        data: {
            userId: user.id,
            credentialID: Buffer.from(credentialID).toString('base64'),
            credentialPublicKey: Buffer.from(credentialPublicKey).toString('base64'),
            counter,
            transports: body.response.transports
        }
    });
    
    // Clean up challenge
    await db.webauthnChallenges.delete({ where: { userId: user.id } });
    
    res.json({ verified: true });
});

// ════════════════════════════════════════════════════════════════
// AUTHENTICATION (Using a security key)
// ════════════════════════════════════════════════════════════════

// Step 1: Generate authentication options
app.post('/api/webauthn/authenticate/options', async (req, res) => {
    const { userId } = req.body;
    
    const authenticators = await db.authenticators.findMany({
        where: { userId }
    });
    
    const options = await generateAuthenticationOptions({
        rpID,
        allowCredentials: authenticators.map(auth => ({
            id: Buffer.from(auth.credentialID, 'base64'),
            type: 'public-key',
            transports: auth.transports
        })),
        userVerification: 'preferred'
    });
    
    // Store challenge
    await db.webauthnChallenges.upsert({
        where: { tempId: req.body.tempId },  // Use temp ID for unauthenticated requests
        create: { tempId: req.body.tempId, challenge: options.challenge, userId },
        update: { challenge: options.challenge, userId }
    });
    
    res.json(options);
});

// Step 2: Verify authentication response
app.post('/api/webauthn/authenticate/verify', async (req, res) => {
    const { body } = req;
    
    const challenge = await db.webauthnChallenges.findUnique({
        where: { tempId: req.body.tempId }
    });
    
    if (!challenge) {
        return res.status(400).json({ error: 'Challenge not found' });
    }
    
    const authenticator = await db.authenticators.findUnique({
        where: {
            credentialID: Buffer.from(body.id, 'base64url').toString('base64')
        }
    });
    
    if (!authenticator) {
        return res.status(400).json({ error: 'Authenticator not found' });
    }
    
    const verification = await verifyAuthenticationResponse({
        response: body,
        expectedChallenge: challenge.challenge,
        expectedOrigin: origin,
        expectedRPID: rpID,
        authenticator: {
            credentialID: Buffer.from(authenticator.credentialID, 'base64'),
            credentialPublicKey: Buffer.from(authenticator.credentialPublicKey, 'base64'),
            counter: authenticator.counter
        }
    });
    
    if (!verification.verified) {
        return res.status(400).json({ error: 'Verification failed' });
    }
    
    // Update counter to prevent replay attacks
    await db.authenticators.update({
        where: { id: authenticator.id },
        data: { counter: verification.authenticationInfo.newCounter }
    });
    
    // Clean up and create session
    await db.webauthnChallenges.delete({ where: { tempId: req.body.tempId } });
    
    // ... create session for user
    
    res.json({ verified: true });
});
```

### Complete Login Flow with 2FA

```typescript
// ════════════════════════════════════════════════════════════════
// COMPLETE LOGIN FLOW WITH 2FA
// ════════════════════════════════════════════════════════════════

interface LoginState {
    step: 'password' | '2fa' | 'complete';
    userId?: string;
    tempToken?: string;
}

// Step 1: Verify password
app.post('/api/login', async (req, res) => {
    const { email, password } = req.body;
    
    const user = await db.users.findUnique({ where: { email } });
    if (!user) {
        // Constant time comparison
        await bcrypt.hash(password, 10);
        return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    const valid = await bcrypt.compare(password, user.passwordHash);
    if (!valid) {
        return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Check if 2FA is enabled
    if (user.twoFactorEnabled) {
        // Generate temporary token for 2FA step
        const tempToken = crypto.randomBytes(32).toString('hex');
        
        await db.twoFactorPending.create({
            data: {
                tempToken,
                userId: user.id,
                expiresAt: new Date(Date.now() + 5 * 60 * 1000)  // 5 minutes
            }
        });
        
        return res.json({
            step: '2fa',
            tempToken,
            methods: user.webauthnEnabled 
                ? ['totp', 'webauthn', 'backup']
                : ['totp', 'backup']
        });
    }
    
    // No 2FA, create session
    req.session.userId = user.id;
    req.session.regenerate(() => {
        res.json({ step: 'complete' });
    });
});

// Step 2: Verify 2FA code
app.post('/api/login/2fa', async (req, res) => {
    const { tempToken, code, method } = req.body;
    
    // Find pending 2FA
    const pending = await db.twoFactorPending.findFirst({
        where: {
            tempToken,
            expiresAt: { gt: new Date() }
        }
    });
    
    if (!pending) {
        return res.status(400).json({ error: 'Session expired, please login again' });
    }
    
    let verified = false;
    
    switch (method) {
        case 'totp':
            verified = await verifyTOTP(pending.userId, code);
            break;
        case 'backup':
            verified = await verifyBackupCode(pending.userId, code);
            break;
        // WebAuthn handled separately with challenge-response
    }
    
    if (!verified) {
        return res.status(401).json({ error: 'Invalid code' });
    }
    
    // Clean up and create session
    await db.twoFactorPending.delete({ where: { id: pending.id } });
    
    req.session.userId = pending.userId;
    req.session.regenerate(() => {
        res.json({ step: 'complete' });
    });
});
```

---

## Interview Questions

**Q: "Why is TOTP better than SMS for 2FA?"**
> "SMS is vulnerable to SIM swapping - attackers can convince carriers to transfer your number. SMS can also be intercepted. TOTP secrets are stored on your device, not transmitted each time. The code is generated offline, so there's no interception risk. TOTP apps also work without cell service."

**Q: "What is WebAuthn and why is it phishing-resistant?"**
> "WebAuthn uses public key cryptography with hardware authenticators or biometrics. It's phishing-resistant because the browser includes the origin (domain) in the cryptographic challenge - a phishing site at fake-myapp.com can't generate a valid response for myapp.com. The credential is bound to the specific domain."

**Q: "How do you handle account recovery when 2FA device is lost?"**
> "Multiple layers: 1) Backup codes given at setup (hashed like passwords, single use). 2) Secondary 2FA method if enabled (TOTP + WebAuthn). 3) Identity verification through support (ID verification, security questions from signup). Never allow recovery that bypasses 2FA completely - that defeats the purpose. The goal is to make recovery possible but difficult for attackers."

---

## Quick Reference

```
2FA IMPLEMENTATION CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  TOTP:                                                          │
│  □ Generate 160-bit secret                                     │
│  □ Show QR code for setup                                      │
│  □ Verify code before enabling                                 │
│  □ Allow 1 window drift (30 sec before/after)                  │
│                                                                  │
│  BACKUP CODES:                                                  │
│  □ Generate 10 codes at setup                                  │
│  □ Hash codes (like passwords)                                 │
│  □ Single use, delete after use                                │
│  □ Allow regeneration (invalidates old)                        │
│                                                                  │
│  WEBAUTHN:                                                      │
│  □ Store credential ID and public key                          │
│  □ Update counter after each use                               │
│  □ Support multiple authenticators                             │
│                                                                  │
│  UX:                                                            │
│  □ Don't require 2FA immediately (grace period)                │
│  □ Prompt to enable after suspicious activity                  │
│  □ Remember device option (30 days, separate cookie)           │
│  □ Clear 2FA for high-privilege ops (fresh auth)               │
│                                                                  │
│  SECURITY:                                                      │
│  □ Rate limit 2FA attempts                                     │
│  □ Lock after failed attempts                                  │
│  □ Log all 2FA events                                          │
│  □ Require 2FA for admin accounts                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


