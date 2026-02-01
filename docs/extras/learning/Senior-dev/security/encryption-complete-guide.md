# 🔐 Encryption - Complete Guide

> A comprehensive guide to encryption - at rest, in transit, hashing, salting, key management, and implementing cryptography correctly.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Encryption transforms readable data (plaintext) into unreadable data (ciphertext) using an algorithm and key - 'at rest' protects stored data, 'in transit' protects data moving over networks, and the hardest part isn't the crypto itself, it's managing the keys."

### Encryption Types Overview
```
ENCRYPTION OVERVIEW:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SYMMETRIC ENCRYPTION (Same key for encrypt/decrypt)            │
│  ─────────────────────────────────────────────────              │
│  • AES-256-GCM (recommended)                                   │
│  • Fast, used for bulk data                                    │
│  • Challenge: How to share the key securely?                   │
│                                                                  │
│  plaintext → [AES + key] → ciphertext                          │
│  ciphertext → [AES + key] → plaintext                          │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  ASYMMETRIC ENCRYPTION (Public/private key pair)               │
│  ───────────────────────────────────────────────                │
│  • RSA, ECC                                                    │
│  • Slower, used for key exchange and signatures                │
│  • Public key encrypts, private key decrypts                   │
│                                                                  │
│  plaintext → [RSA + public key] → ciphertext                   │
│  ciphertext → [RSA + private key] → plaintext                  │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  HASHING (One-way, no decryption)                              │
│  ────────────────────────────────                               │
│  • SHA-256 (general), bcrypt/Argon2 (passwords)                │
│  • Cannot reverse - verify by hashing input and comparing      │
│                                                                  │
│  password → [bcrypt] → hash                                    │
│  (can't get password back from hash)                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement
> "I implemented a field-level encryption system for PII data. Sensitive fields (SSN, bank account) are encrypted at the application layer with AES-256-GCM before hitting the database - so even DBA access doesn't expose plaintext. Each record has a unique IV (initialization vector), and we use envelope encryption: data encrypted with DEK (data encryption key), DEK encrypted with KEK (key encryption key) stored in AWS KMS. Key rotation rotates the KEK without re-encrypting all data - we just re-wrap the DEKs. For passwords, we use Argon2id with 64MB memory cost - hardware attacks are prohibitively expensive."

---

## 📚 Core Concepts

### Symmetric Encryption (AES)

```typescript
// ════════════════════════════════════════════════════════════════
// AES-256-GCM ENCRYPTION
// ════════════════════════════════════════════════════════════════

import crypto from 'crypto';

const ALGORITHM = 'aes-256-gcm';
const IV_LENGTH = 12;  // 96 bits for GCM
const AUTH_TAG_LENGTH = 16;  // 128 bits

// Generate a secure key (do this once, store securely)
function generateKey(): Buffer {
    return crypto.randomBytes(32);  // 256 bits
}

// Encrypt data
function encrypt(plaintext: string, key: Buffer): string {
    const iv = crypto.randomBytes(IV_LENGTH);
    const cipher = crypto.createCipheriv(ALGORITHM, key, iv, {
        authTagLength: AUTH_TAG_LENGTH
    });
    
    const encrypted = Buffer.concat([
        cipher.update(plaintext, 'utf8'),
        cipher.final()
    ]);
    
    const authTag = cipher.getAuthTag();
    
    // Combine: IV + encrypted data + auth tag
    const combined = Buffer.concat([iv, encrypted, authTag]);
    return combined.toString('base64');
}

// Decrypt data
function decrypt(ciphertext: string, key: Buffer): string {
    const combined = Buffer.from(ciphertext, 'base64');
    
    // Extract components
    const iv = combined.subarray(0, IV_LENGTH);
    const authTag = combined.subarray(-AUTH_TAG_LENGTH);
    const encrypted = combined.subarray(IV_LENGTH, -AUTH_TAG_LENGTH);
    
    const decipher = crypto.createDecipheriv(ALGORITHM, key, iv, {
        authTagLength: AUTH_TAG_LENGTH
    });
    decipher.setAuthTag(authTag);
    
    const decrypted = Buffer.concat([
        decipher.update(encrypted),
        decipher.final()
    ]);
    
    return decrypted.toString('utf8');
}

// Usage
const key = generateKey();  // Store this securely!
const encrypted = encrypt('sensitive data', key);
const decrypted = decrypt(encrypted, key);

// ════════════════════════════════════════════════════════════════
// FIELD-LEVEL ENCRYPTION SERVICE
// ════════════════════════════════════════════════════════════════

class FieldEncryptionService {
    private key: Buffer;
    
    constructor(keyBase64: string) {
        this.key = Buffer.from(keyBase64, 'base64');
    }
    
    encryptField(value: string): string {
        return encrypt(value, this.key);
    }
    
    decryptField(encrypted: string): string {
        return decrypt(encrypted, this.key);
    }
    
    // Encrypt specific fields in an object
    encryptObject<T extends Record<string, any>>(
        obj: T, 
        fieldsToEncrypt: (keyof T)[]
    ): T {
        const result = { ...obj };
        for (const field of fieldsToEncrypt) {
            if (result[field]) {
                result[field] = this.encryptField(String(result[field]));
            }
        }
        return result;
    }
}

// Usage with Prisma middleware
prisma.$use(async (params, next) => {
    const encryptedFields = ['ssn', 'bankAccount', 'taxId'];
    
    // Encrypt on write
    if (['create', 'update'].includes(params.action)) {
        if (params.args.data) {
            for (const field of encryptedFields) {
                if (params.args.data[field]) {
                    params.args.data[field] = encryptionService.encryptField(
                        params.args.data[field]
                    );
                }
            }
        }
    }
    
    const result = await next(params);
    
    // Decrypt on read
    if (result && params.action === 'findUnique') {
        for (const field of encryptedFields) {
            if (result[field]) {
                result[field] = encryptionService.decryptField(result[field]);
            }
        }
    }
    
    return result;
});
```

### Envelope Encryption

```typescript
// ════════════════════════════════════════════════════════════════
// ENVELOPE ENCRYPTION (Using AWS KMS)
// ════════════════════════════════════════════════════════════════

import { KMSClient, GenerateDataKeyCommand, DecryptCommand } from '@aws-sdk/client-kms';

const kms = new KMSClient({ region: 'us-east-1' });
const KEY_ID = 'arn:aws:kms:us-east-1:123456789:key/abc-123';

// Generate a data key for encrypting data
async function generateDataKey() {
    const command = new GenerateDataKeyCommand({
        KeyId: KEY_ID,
        KeySpec: 'AES_256'
    });
    
    const response = await kms.send(command);
    
    return {
        // Plaintext key - use this to encrypt data, then discard
        plaintext: response.Plaintext,
        // Encrypted key - store this alongside encrypted data
        encryptedKey: response.CiphertextBlob
    };
}

// Decrypt the data key using KMS
async function decryptDataKey(encryptedKey: Buffer): Promise<Buffer> {
    const command = new DecryptCommand({
        CiphertextBlob: encryptedKey,
        KeyId: KEY_ID
    });
    
    const response = await kms.send(command);
    return Buffer.from(response.Plaintext!);
}

// Envelope encryption flow
class EnvelopeEncryption {
    // Encrypt data with envelope encryption
    async encrypt(plaintext: string): Promise<{
        encryptedData: string;
        encryptedKey: string;
    }> {
        // 1. Generate a new data key from KMS
        const { plaintext: dataKey, encryptedKey } = await generateDataKey();
        
        // 2. Encrypt data with the plaintext data key
        const encryptedData = encrypt(plaintext, Buffer.from(dataKey!));
        
        // 3. Return encrypted data + encrypted key
        // (plaintext data key is never stored)
        return {
            encryptedData,
            encryptedKey: Buffer.from(encryptedKey!).toString('base64')
        };
    }
    
    // Decrypt data
    async decrypt(encryptedData: string, encryptedKey: string): Promise<string> {
        // 1. Decrypt the data key using KMS
        const dataKey = await decryptDataKey(
            Buffer.from(encryptedKey, 'base64')
        );
        
        // 2. Decrypt the data with the plaintext data key
        return decrypt(encryptedData, dataKey);
    }
}

/*
WHY ENVELOPE ENCRYPTION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. Performance: KMS is slow. With envelope encryption, we     │
│     only call KMS once to get the data key, then encrypt       │
│     locally (fast) with AES.                                   │
│                                                                  │
│  2. Key Rotation: To rotate the master key (KEK), we just      │
│     re-encrypt the data keys. We don't need to re-encrypt      │
│     all the actual data.                                       │
│                                                                  │
│  3. Key Limits: KMS has limits on data size (4KB).             │
│     Envelope encryption lets us encrypt any size data.         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
*/
```

### Hashing (Passwords)

```typescript
// ════════════════════════════════════════════════════════════════
// PASSWORD HASHING WITH ARGON2
// ════════════════════════════════════════════════════════════════

import argon2 from 'argon2';

// Hash a password
async function hashPassword(password: string): Promise<string> {
    return argon2.hash(password, {
        type: argon2.argon2id,  // Recommended variant
        memoryCost: 65536,      // 64 MB
        timeCost: 3,            // 3 iterations
        parallelism: 4          // 4 threads
    });
}

// Verify a password
async function verifyPassword(password: string, hash: string): Promise<boolean> {
    try {
        return await argon2.verify(hash, password);
    } catch {
        return false;
    }
}

// Check if re-hash is needed (after config change)
async function needsRehash(hash: string): Promise<boolean> {
    return argon2.needsRehash(hash, {
        memoryCost: 65536,
        timeCost: 3
    });
}

// ════════════════════════════════════════════════════════════════
// PASSWORD HASHING WITH BCRYPT (Alternative)
// ════════════════════════════════════════════════════════════════

import bcrypt from 'bcrypt';

const SALT_ROUNDS = 12;  // ~250ms on modern hardware

async function hashPasswordBcrypt(password: string): Promise<string> {
    return bcrypt.hash(password, SALT_ROUNDS);
}

async function verifyPasswordBcrypt(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
}

// ════════════════════════════════════════════════════════════════
// GENERAL HASHING (NOT FOR PASSWORDS)
// ════════════════════════════════════════════════════════════════

import crypto from 'crypto';

// SHA-256 for integrity checks, tokens, etc.
function sha256(data: string): string {
    return crypto.createHash('sha256').update(data).digest('hex');
}

// HMAC for authentication codes
function hmac(data: string, secret: string): string {
    return crypto.createHmac('sha256', secret).update(data).digest('hex');
}
```

### Encryption at Rest

```typescript
// ════════════════════════════════════════════════════════════════
// DATABASE ENCRYPTION AT REST
// ════════════════════════════════════════════════════════════════

/*
DATABASE-LEVEL ENCRYPTION:

PostgreSQL:
- pg_crypto extension for column-level
- Transparent Data Encryption (TDE) at file level
- AWS RDS: Enable encryption when creating instance

MongoDB:
- Encrypted Storage Engine
- Client-Side Field Level Encryption (CSFLE)

AWS S3:
- SSE-S3: S3-managed keys
- SSE-KMS: KMS-managed keys (recommended)
- SSE-C: Customer-provided keys
*/

// S3 with server-side encryption
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';

const s3 = new S3Client({ region: 'us-east-1' });

async function uploadEncrypted(bucket: string, key: string, data: Buffer) {
    await s3.send(new PutObjectCommand({
        Bucket: bucket,
        Key: key,
        Body: data,
        ServerSideEncryption: 'aws:kms',  // Use KMS
        SSEKMSKeyId: 'arn:aws:kms:...'    // Your KMS key
    }));
}
```

---

## Key Management Best Practices

```
KEY MANAGEMENT:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  KEY HIERARCHY:                                                 │
│                                                                  │
│  Master Key (KEK) ── stored in HSM/KMS, never exported         │
│       │                                                         │
│       ├── Data Encryption Key (DEK) 1                          │
│       │       └── Encrypts user data batch 1                   │
│       │                                                         │
│       ├── Data Encryption Key (DEK) 2                          │
│       │       └── Encrypts user data batch 2                   │
│       │                                                         │
│       └── Data Encryption Key (DEK) N                          │
│               └── Encrypts user data batch N                   │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  KEY ROTATION:                                                  │
│                                                                  │
│  1. Generate new KEK                                           │
│  2. Re-encrypt all DEKs with new KEK                          │
│  3. Keep old KEK for decrypting old data                      │
│  4. Eventually re-encrypt all data with new DEKs              │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  STORAGE:                                                       │
│  • KEK: Hardware Security Module (HSM) or cloud KMS           │
│  • DEK: Encrypted in database alongside data                  │
│  • Never store keys with the data they encrypt                │
│  • Never log keys                                             │
│  • Never commit keys to source control                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "Symmetric vs asymmetric encryption?"**
> "Symmetric uses the same key for encrypt/decrypt - fast, used for bulk data (AES-256-GCM). Asymmetric uses public/private key pair - slower, used for key exchange and signatures (RSA, ECC). In practice, we combine them: asymmetric to securely exchange a symmetric key, then symmetric for the actual data. That's how TLS works."

**Q: "How do you handle encryption key rotation?"**
> "With envelope encryption. Data is encrypted with a DEK, DEK is encrypted with KEK stored in KMS. To rotate: generate new KEK, re-encrypt all DEKs with new KEK (fast, DEKs are small), keep old KEK to decrypt old data. We can optionally re-encrypt data with new DEKs gradually. This way rotation doesn't require re-encrypting terabytes of data at once."

**Q: "Why bcrypt/Argon2 for passwords instead of SHA-256?"**
> "Password hashes need to be slow to prevent brute force. SHA-256 is fast - attackers can try billions of guesses per second. Bcrypt has a configurable work factor, Argon2 also has memory cost. They're intentionally slow (~250ms) and use salts to prevent rainbow tables. The slowness is the feature."

---

## Quick Reference

```
ENCRYPTION CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  USE CASES:                                                     │
│  • Bulk data: AES-256-GCM (symmetric)                          │
│  • Passwords: Argon2id or bcrypt (hash, not encrypt)           │
│  • Key exchange: RSA or ECDH (asymmetric)                      │
│  • Signatures: RSA or ECDSA                                    │
│  • Integrity: HMAC-SHA256                                      │
│  • Random IDs: crypto.randomBytes                              │
│                                                                  │
│  DON'T:                                                         │
│  ✗ Roll your own crypto                                        │
│  ✗ Use MD5 or SHA1 for security                                │
│  ✗ Reuse IVs/nonces                                            │
│  ✗ Use ECB mode                                                │
│  ✗ Store keys with encrypted data                              │
│                                                                  │
│  DO:                                                            │
│  ✓ Use authenticated encryption (GCM mode)                     │
│  ✓ Generate random IV per encryption                           │
│  ✓ Use envelope encryption for scalability                     │
│  ✓ Store keys in HSM/KMS                                       │
│  ✓ Rotate keys regularly                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


