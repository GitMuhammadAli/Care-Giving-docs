# 🔐 Secrets Management - Complete Guide

> A comprehensive guide to secrets management - Vault, environment variables, rotation, encryption at rest, and keeping credentials secure throughout the development lifecycle.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Secrets management is the practice of securely storing, accessing, distributing, and rotating sensitive credentials like API keys, database passwords, and encryption keys - because hardcoded secrets in code are the #1 way attackers gain access."

### Secrets Management Hierarchy
```
SECRETS MANAGEMENT HIERARCHY:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ❌ LEVEL 0: Hardcoded in source code                          │
│     const API_KEY = "sk_live_abc123";                          │
│     → Visible in git history FOREVER                           │
│                                                                  │
│  ⚠️  LEVEL 1: Environment variables (local .env)               │
│     process.env.API_KEY                                        │
│     → Better, but still plain text on disk                     │
│                                                                  │
│  ✓ LEVEL 2: Environment variables (injected at deploy)         │
│     CI/CD or container orchestrator injects                    │
│     → Not in code, but still plain text in memory              │
│                                                                  │
│  ✓✓ LEVEL 3: Secrets Manager (AWS, GCP, Azure)                 │
│     Encrypted at rest, IAM-based access control                │
│     → Audited, encrypted, but static                           │
│                                                                  │
│  ✓✓✓ LEVEL 4: Vault (HashiCorp) with dynamic secrets           │
│     Short-lived credentials generated on demand                │
│     → Best: encrypted, audited, rotated, dynamic               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement
> "I migrated our secrets from .env files to HashiCorp Vault. Database credentials are now dynamic - each service gets unique credentials that auto-expire in 1 hour. We use AppRole authentication for services, and Vault Agent handles token renewal automatically. Secrets are encrypted at rest with AES-256, and every access is logged. The migration also revealed we had 12 unused API keys and 3 that were shared across environments - now each environment has isolated secrets with automatic rotation every 30 days."

---

## 📚 Core Concepts

### Environment Variables (Basic)

```typescript
// ════════════════════════════════════════════════════════════════
// ENVIRONMENT VARIABLES BASICS
// ════════════════════════════════════════════════════════════════

// .env file (NEVER commit to git!)
/*
DATABASE_URL=postgresql://user:pass@localhost:5432/db
API_KEY=sk_live_abc123
JWT_SECRET=super_secret_key_here
*/

// .gitignore
/*
.env
.env.local
.env.*.local
*/

// .env.example (commit this, no real values)
/*
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
API_KEY=your_api_key_here
JWT_SECRET=generate_a_secure_random_string
*/

// Load environment variables
import dotenv from 'dotenv';
dotenv.config();

// Access with validation
function getEnvVar(name: string): string {
    const value = process.env[name];
    if (!value) {
        throw new Error(`Missing required environment variable: ${name}`);
    }
    return value;
}

const config = {
    database: {
        url: getEnvVar('DATABASE_URL')
    },
    jwt: {
        secret: getEnvVar('JWT_SECRET'),
        expiresIn: process.env.JWT_EXPIRES_IN || '15m'
    }
};
```

### Cloud Secrets Managers

```typescript
// ════════════════════════════════════════════════════════════════
// AWS SECRETS MANAGER
// ════════════════════════════════════════════════════════════════

import { 
    SecretsManagerClient, 
    GetSecretValueCommand 
} from '@aws-sdk/client-secrets-manager';

const client = new SecretsManagerClient({ region: 'us-east-1' });

async function getSecret(secretName: string): Promise<Record<string, string>> {
    const command = new GetSecretValueCommand({ SecretId: secretName });
    const response = await client.send(command);
    
    if (response.SecretString) {
        return JSON.parse(response.SecretString);
    }
    
    throw new Error('Secret not found');
}

// Usage
const dbCredentials = await getSecret('prod/database');
const connection = await pool.connect({
    host: dbCredentials.host,
    user: dbCredentials.username,
    password: dbCredentials.password,
    database: dbCredentials.database
});

// ════════════════════════════════════════════════════════════════
// GCP SECRET MANAGER
// ════════════════════════════════════════════════════════════════

import { SecretManagerServiceClient } from '@google-cloud/secret-manager';

const client = new SecretManagerServiceClient();

async function getGCPSecret(name: string): Promise<string> {
    const [version] = await client.accessSecretVersion({
        name: `projects/my-project/secrets/${name}/versions/latest`
    });
    
    return version.payload?.data?.toString() || '';
}

// ════════════════════════════════════════════════════════════════
// AZURE KEY VAULT
// ════════════════════════════════════════════════════════════════

import { SecretClient } from '@azure/keyvault-secrets';
import { DefaultAzureCredential } from '@azure/identity';

const credential = new DefaultAzureCredential();
const vaultUrl = 'https://my-vault.vault.azure.net';
const client = new SecretClient(vaultUrl, credential);

async function getAzureSecret(name: string): Promise<string> {
    const secret = await client.getSecret(name);
    return secret.value || '';
}
```

### HashiCorp Vault

```typescript
// ════════════════════════════════════════════════════════════════
// HASHICORP VAULT - FULL IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

import Vault from 'node-vault';

// Initialize client
const vault = Vault({
    apiVersion: 'v1',
    endpoint: process.env.VAULT_ADDR || 'http://localhost:8200',
    token: process.env.VAULT_TOKEN  // Or use AppRole below
});

// Read a static secret
async function readSecret(path: string): Promise<any> {
    const response = await vault.read(`secret/data/${path}`);
    return response.data.data;
}

// Write a secret
async function writeSecret(path: string, data: object): Promise<void> {
    await vault.write(`secret/data/${path}`, { data });
}

// ════════════════════════════════════════════════════════════════
// APPROLE AUTHENTICATION (For services)
// ════════════════════════════════════════════════════════════════

async function authenticateWithAppRole(roleId: string, secretId: string) {
    const response = await vault.approleLogin({
        role_id: roleId,
        secret_id: secretId
    });
    
    vault.token = response.auth.client_token;
    
    // Schedule token renewal before expiry
    const ttl = response.auth.lease_duration;
    setTimeout(() => renewToken(), (ttl - 60) * 1000);
    
    return response.auth.client_token;
}

async function renewToken(): Promise<void> {
    const response = await vault.tokenRenewSelf();
    const ttl = response.auth.lease_duration;
    setTimeout(() => renewToken(), (ttl - 60) * 1000);
}

// ════════════════════════════════════════════════════════════════
// DYNAMIC DATABASE CREDENTIALS
// ════════════════════════════════════════════════════════════════

// Vault generates unique, short-lived credentials for each request
async function getDatabaseCredentials(role: string) {
    const response = await vault.read(`database/creds/${role}`);
    
    return {
        username: response.data.username,  // Generated: v-approle-myapp-abc123
        password: response.data.password,  // Random, strong password
        leaseId: response.lease_id,
        leaseDuration: response.lease_duration  // e.g., 3600 seconds
    };
}

// Usage with connection pool
class DynamicPool {
    private pool: Pool | null = null;
    private leaseId: string | null = null;
    
    async getConnection() {
        if (!this.pool) {
            await this.createPool();
        }
        return this.pool.connect();
    }
    
    async createPool() {
        const creds = await getDatabaseCredentials('myapp-role');
        
        this.pool = new Pool({
            user: creds.username,
            password: creds.password,
            // ... other config
        });
        
        this.leaseId = creds.leaseId;
        
        // Renew credentials before expiry
        const renewIn = (creds.leaseDuration - 300) * 1000;
        setTimeout(() => this.renewCredentials(), renewIn);
    }
    
    async renewCredentials() {
        // Renew lease or create new pool with new credentials
        try {
            await vault.write(`sys/leases/renew`, {
                lease_id: this.leaseId
            });
        } catch {
            // Lease expired, recreate pool
            await this.pool?.end();
            await this.createPool();
        }
    }
}
```

### Secret Rotation

```typescript
// ════════════════════════════════════════════════════════════════
// SECRET ROTATION STRATEGY
// ════════════════════════════════════════════════════════════════

// AWS Secrets Manager rotation Lambda
export const handler = async (event: SecretsManagerRotationEvent) => {
    const { SecretId, ClientRequestToken, Step } = event;
    
    switch (Step) {
        case 'createSecret':
            // Generate new secret
            const newPassword = generateSecurePassword();
            await secretsManager.putSecretValue({
                SecretId,
                ClientRequestToken,
                SecretString: JSON.stringify({ password: newPassword }),
                VersionStage: 'AWSPENDING'
            });
            break;
            
        case 'setSecret':
            // Update the actual service (e.g., change DB password)
            const pending = await getSecretVersion(SecretId, 'AWSPENDING');
            await updateDatabasePassword(pending.password);
            break;
            
        case 'testSecret':
            // Verify new credentials work
            const newCreds = await getSecretVersion(SecretId, 'AWSPENDING');
            await testDatabaseConnection(newCreds);
            break;
            
        case 'finishSecret':
            // Mark new version as current
            await secretsManager.updateSecretVersionStage({
                SecretId,
                VersionStage: 'AWSCURRENT',
                MoveToVersionId: ClientRequestToken,
                RemoveFromVersionId: currentVersionId
            });
            break;
    }
};

// ════════════════════════════════════════════════════════════════
// API KEY ROTATION (Application level)
// ════════════════════════════════════════════════════════════════

class ApiKeyRotation {
    // Dual-key strategy: both old and new work during transition
    async rotateApiKey(clientId: string): Promise<{ newKey: string }> {
        const newKey = generateApiKey();
        const newKeyHash = hashApiKey(newKey);
        
        await db.apiKeys.update({
            where: { clientId },
            data: {
                keyHash: newKeyHash,
                previousKeyHash: existingKeyHash,
                previousKeyExpiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000),
                rotatedAt: new Date()
            }
        });
        
        // Return new key (only time it's visible)
        return { newKey };
    }
    
    // Validation accepts both current and previous (if not expired)
    async validateApiKey(key: string): Promise<Client | null> {
        const hash = hashApiKey(key);
        
        const client = await db.apiKeys.findFirst({
            where: {
                OR: [
                    { keyHash: hash },
                    {
                        previousKeyHash: hash,
                        previousKeyExpiresAt: { gt: new Date() }
                    }
                ]
            }
        });
        
        return client;
    }
}
```

---

## Secrets in CI/CD

```yaml
# ════════════════════════════════════════════════════════════════
# GITHUB ACTIONS - Using secrets
# ════════════════════════════════════════════════════════════════

name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Secrets injected as environment variables
      - name: Build and Deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          npm run build
          npm run deploy
      
      # Using OIDC for AWS (no static credentials!)
      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/github-actions
          aws-region: us-east-1

# ════════════════════════════════════════════════════════════════
# DOCKER - Secret handling
# ════════════════════════════════════════════════════════════════

# ❌ DON'T: Put secrets in Dockerfile
# ENV API_KEY=sk_live_abc123

# ❌ DON'T: Pass secrets as build args (visible in history)
# ARG API_KEY
# docker build --build-arg API_KEY=sk_live_abc123

# ✅ DO: Pass at runtime
# docker run -e API_KEY=$API_KEY myapp

# ✅ DO: Use Docker secrets (Swarm/Compose)
# docker secret create api_key ./api_key.txt
```

---

## Interview Questions

**Q: "How do you manage secrets in production?"**
> "I use cloud secrets managers (AWS Secrets Manager, Vault) for encrypted storage with IAM-based access control. Secrets are injected at runtime, never in code or images. For databases, I use dynamic credentials from Vault that auto-expire. All access is audited. Rotation happens automatically every 30 days for static secrets, or continuously for dynamic ones."

**Q: "What if secrets are accidentally committed to git?"**
> "Immediate rotation is critical - assume compromised. Rotate the secret in the actual service first, then update the secrets manager. Use git-filter-repo to remove from history, but assume the secret was seen. Prevent future issues with pre-commit hooks (gitleaks, truffleHog) and CI checks. Keys in git history are indexed by scanners within minutes."

**Q: "How do you handle secrets in development vs production?"**
> "Strict environment separation. Development uses local .env files with non-sensitive test credentials. Production uses cloud secrets manager. Never share secrets across environments. Developers don't have access to production secrets - they use their own test API keys. CI/CD has read-only access to deploy secrets."

---

## Quick Reference

```
SECRETS MANAGEMENT CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  □ Never commit secrets to git                                 │
│  □ Use .gitignore for .env files                               │
│  □ Pre-commit hooks to scan for secrets                        │
│  □ Use secrets manager in production                           │
│  □ Encrypt secrets at rest                                     │
│  □ Rotate secrets regularly (30-90 days)                       │
│  □ Use dynamic/short-lived credentials where possible          │
│  □ Audit secret access                                         │
│  □ Separate secrets per environment                            │
│  □ Least privilege access to secrets                           │
│                                                                  │
│  ROTATION STRATEGY:                                             │
│  • API keys: Generate new, keep old valid briefly              │
│  • DB passwords: Dual-user rotation                            │
│  • Encryption keys: Re-encrypt with new key                    │
│                                                                  │
│  TOOLS:                                                         │
│  • Storage: Vault, AWS Secrets Manager, GCP Secret Manager     │
│  • Scanning: gitleaks, truffleHog, git-secrets                 │
│  • Injection: doppler, chamber, dotenv-vault                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


