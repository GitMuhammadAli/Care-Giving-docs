# 🔒 SSL/TLS - Complete Guide

> A comprehensive guide to SSL/TLS - certificates, HTTPS, certificate pinning, renewal, and securing communication channels.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "TLS (Transport Layer Security) encrypts data in transit between client and server, authenticates the server via certificates, and ensures data integrity - HTTPS is just HTTP over TLS, and it's non-negotiable for any production application."

### TLS Handshake Overview
```
TLS 1.3 HANDSHAKE (Simplified):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  CLIENT                                SERVER                   │
│                                                                  │
│  1. ClientHello ─────────────────────────────────────>          │
│     • Supported TLS versions                                    │
│     • Supported cipher suites                                   │
│     • Client random                                             │
│     • Key share (TLS 1.3)                                       │
│                                                                  │
│  2. <───────────────────────────────────── ServerHello          │
│     • Chosen TLS version                                        │
│     • Chosen cipher suite                                       │
│     • Server random                                             │
│     • Key share                                                 │
│                                                                  │
│  3. <───────────────────────────────────── Certificate          │
│     • Server's certificate chain                               │
│     • CertificateVerify (signature)                            │
│     • Finished                                                  │
│                                                                  │
│  4. Finished ────────────────────────────────────────>          │
│     • Client's Finished message                                │
│                                                                  │
│  5. <════════════════════════════════════════════════>          │
│     ENCRYPTED APPLICATION DATA                                  │
│                                                                  │
│  TLS 1.3: 1-RTT handshake (vs 2-RTT in TLS 1.2)               │
│  0-RTT resumption available (but has replay risks)             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement
> "I configured our TLS setup for A+ rating on SSL Labs. We use TLS 1.3 only in production, with TLS 1.2 fallback for legacy clients. Certificates are from Let's Encrypt with automatic renewal via certbot 30 days before expiry. We use HSTS with a 2-year max-age to prevent downgrade attacks, CAA records to restrict certificate issuance, and OCSP stapling for faster certificate validation. For internal services, we use mutual TLS (mTLS) where both client and server present certificates - this is our service mesh authentication layer."

---

## 📚 Core Concepts

### Certificate Types

```
CERTIFICATE TYPES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  DV (Domain Validation)                                         │
│  ─────────────────────                                          │
│  • Verifies domain ownership only                              │
│  • Automated (Let's Encrypt)                                   │
│  • Cheapest/free                                               │
│  • Good for: Most websites, APIs                               │
│                                                                  │
│  OV (Organization Validation)                                   │
│  ────────────────────────────                                   │
│  • Verifies organization details                               │
│  • Manual verification process                                 │
│  • Shows company name in certificate                           │
│  • Good for: Business websites                                 │
│                                                                  │
│  EV (Extended Validation)                                       │
│  ─────────────────────────                                      │
│  • Rigorous verification                                       │
│  • Legal entity verification                                   │
│  • Was: Green bar in browser (deprecated)                      │
│  • Good for: Banking, high-trust sites                         │
│                                                                  │
│  WILDCARD                                                       │
│  ────────                                                       │
│  • *.example.com covers all subdomains                         │
│  • Single level only (not sub.sub.example.com)                 │
│  • Can be DV, OV, or EV                                        │
│                                                                  │
│  SAN (Subject Alternative Name)                                 │
│  ──────────────────────────────                                 │
│  • Multiple domains in one certificate                         │
│  • example.com + www.example.com + api.example.com             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Let's Encrypt & Certbot

```bash
# ════════════════════════════════════════════════════════════════
# CERTBOT INSTALLATION & USAGE
# ════════════════════════════════════════════════════════════════

# Install certbot
sudo apt update
sudo apt install certbot python3-certbot-nginx

# Get certificate (Nginx)
sudo certbot --nginx -d example.com -d www.example.com

# Get certificate (standalone - for manual setup)
sudo certbot certonly --standalone -d example.com

# Wildcard certificate (requires DNS challenge)
sudo certbot certonly --manual --preferred-challenges dns \
    -d example.com -d "*.example.com"

# Test renewal
sudo certbot renew --dry-run

# Force renewal
sudo certbot renew --force-renewal

# ════════════════════════════════════════════════════════════════
# AUTOMATIC RENEWAL (cron or systemd timer)
# ════════════════════════════════════════════════════════════════

# /etc/cron.d/certbot (usually auto-installed)
0 */12 * * * root certbot renew --quiet --post-hook "systemctl reload nginx"

# Certificate locations
# /etc/letsencrypt/live/example.com/fullchain.pem  (cert + intermediates)
# /etc/letsencrypt/live/example.com/privkey.pem   (private key)
```

### Nginx TLS Configuration

```nginx
# ════════════════════════════════════════════════════════════════
# NGINX TLS CONFIGURATION (A+ Rating)
# ════════════════════════════════════════════════════════════════

server {
    listen 80;
    server_name example.com www.example.com;
    
    # Redirect all HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;
    
    # Certificate files
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # TLS versions - TLS 1.2 and 1.3 only
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # Cipher suites (TLS 1.3 uses its own)
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers off;  # Let client choose (modern)
    
    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;
    
    # Session cache
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;  # Disable for perfect forward secrecy
    
    # DH parameters (for TLS 1.2)
    ssl_dhparam /etc/ssl/dhparam.pem;  # Generate: openssl dhparam -out dhparam.pem 2048
    
    # HSTS (HTTP Strict Transport Security)
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    
    # Other security headers
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    
    # ... rest of config
}
```

### Node.js HTTPS Server

```typescript
// ════════════════════════════════════════════════════════════════
// NODE.JS HTTPS SERVER
// ════════════════════════════════════════════════════════════════

import https from 'https';
import fs from 'fs';
import express from 'express';

const app = express();

// Load certificates
const options = {
    key: fs.readFileSync('/etc/letsencrypt/live/example.com/privkey.pem'),
    cert: fs.readFileSync('/etc/letsencrypt/live/example.com/fullchain.pem'),
    
    // TLS options
    minVersion: 'TLSv1.2',
    
    // Cipher suites
    ciphers: [
        'ECDHE-ECDSA-AES128-GCM-SHA256',
        'ECDHE-RSA-AES128-GCM-SHA256',
        'ECDHE-ECDSA-AES256-GCM-SHA384',
        'ECDHE-RSA-AES256-GCM-SHA384'
    ].join(':'),
    
    // OCSP stapling
    // (Node.js doesn't support automatic stapling, use nginx)
};

// Create HTTPS server
https.createServer(options, app).listen(443);

// Redirect HTTP to HTTPS
import http from 'http';
http.createServer((req, res) => {
    res.writeHead(301, { Location: `https://${req.headers.host}${req.url}` });
    res.end();
}).listen(80);
```

### Certificate Pinning

```typescript
// ════════════════════════════════════════════════════════════════
// CERTIFICATE PINNING (Mobile apps, high-security clients)
// ════════════════════════════════════════════════════════════════

import https from 'https';
import crypto from 'crypto';

// Pin the public key hash (SPKI)
const PINNED_KEYS = [
    'sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=',  // Current cert
    'sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB='   // Backup cert
];

function makeSecureRequest(url: string) {
    return new Promise((resolve, reject) => {
        const req = https.request(url, {
            checkServerIdentity: (hostname, cert) => {
                // Calculate SPKI hash
                const pubkey = cert.pubkey;
                const hash = crypto.createHash('sha256')
                    .update(pubkey)
                    .digest('base64');
                const pin = `sha256/${hash}`;
                
                // Verify pin
                if (!PINNED_KEYS.includes(pin)) {
                    throw new Error(`Certificate pinning failed for ${hostname}`);
                }
                
                // Also do normal hostname verification
                return undefined;  // No error
            }
        }, (res) => {
            resolve(res);
        });
        
        req.on('error', reject);
        req.end();
    });
}

// ════════════════════════════════════════════════════════════════
// HTTP PUBLIC KEY PINNING (HPKP) - DEPRECATED
// ════════════════════════════════════════════════════════════════

// HPKP was a header-based pinning mechanism but was deprecated
// because misconfiguration could permanently brick a site
// Use Certificate Transparency (CT) logs instead
```

### Mutual TLS (mTLS)

```typescript
// ════════════════════════════════════════════════════════════════
// MUTUAL TLS - CLIENT CERTIFICATES
// ════════════════════════════════════════════════════════════════

import https from 'https';
import fs from 'fs';

// Server that requires client certificates
const server = https.createServer({
    key: fs.readFileSync('server-key.pem'),
    cert: fs.readFileSync('server-cert.pem'),
    
    // Require client certificate
    requestCert: true,
    rejectUnauthorized: true,  // Reject if no valid client cert
    
    // CA that signed client certificates
    ca: fs.readFileSync('client-ca.pem')
}, (req, res) => {
    // Access client certificate
    const clientCert = req.socket.getPeerCertificate();
    console.log('Client:', clientCert.subject.CN);  // Common Name
    
    res.writeHead(200);
    res.end('Authenticated!');
});

// Client making request with certificate
const req = https.request({
    hostname: 'api.example.com',
    port: 443,
    path: '/secure',
    method: 'GET',
    key: fs.readFileSync('client-key.pem'),
    cert: fs.readFileSync('client-cert.pem'),
    ca: fs.readFileSync('server-ca.pem')
}, (res) => {
    // Handle response
});
```

---

## DNS Records for TLS

```
DNS RECORDS FOR TLS SECURITY:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  CAA (Certificate Authority Authorization)                      │
│  ─────────────────────────────────────────                      │
│  Specifies which CAs can issue certificates for your domain    │
│                                                                  │
│  example.com. CAA 0 issue "letsencrypt.org"                    │
│  example.com. CAA 0 issuewild "letsencrypt.org"                │
│  example.com. CAA 0 iodef "mailto:security@example.com"        │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  DANE (DNS-Based Authentication of Named Entities)             │
│  ─────────────────────────────────────────────────              │
│  Publish certificate hash in DNS (requires DNSSEC)             │
│                                                                  │
│  _443._tcp.example.com. TLSA 3 1 1 <certificate-hash>          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "How does TLS work?"**
> "TLS provides encryption, authentication, and integrity. The handshake: client sends supported ciphers, server responds with chosen cipher and certificate, client verifies certificate against trusted CAs, they exchange keys using asymmetric crypto, then use those keys for symmetric encryption of data. TLS 1.3 reduced this to a 1-RTT handshake."

**Q: "How do you handle certificate renewal?"**
> "Automate it. Use Let's Encrypt with certbot, which handles renewal via cron job. Certificates renew 30 days before expiry. I set up monitoring to alert if a cert is within 14 days of expiry - if automation failed. For load balancers, use AWS ACM or similar that handles renewal automatically."

**Q: "What is HSTS and why use it?"**
> "HTTP Strict Transport Security tells browsers to only use HTTPS for your domain, preventing downgrade attacks. Once received, browser refuses HTTP connections for the max-age period. Include includeSubDomains to cover all subdomains. Preload adds you to browser's built-in HSTS list. Start with short max-age (1 day), verify everything works, then increase to 2 years."

---

## Quick Reference

```
TLS CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  □ TLS 1.2+ only (disable TLS 1.0, 1.1, SSL)                   │
│  □ Strong cipher suites (ECDHE, AES-GCM)                       │
│  □ HTTP → HTTPS redirect                                       │
│  □ HSTS header (Strict-Transport-Security)                     │
│  □ Valid certificate (not expired, correct domain)             │
│  □ Full certificate chain (intermediates included)             │
│  □ Automatic renewal (certbot)                                 │
│  □ CAA DNS records                                             │
│  □ OCSP stapling                                               │
│  □ Monitor certificate expiry                                  │
│                                                                  │
│  TEST YOUR CONFIG:                                              │
│  • https://www.ssllabs.com/ssltest/                            │
│  • https://securityheaders.com/                                │
│                                                                  │
│  COMMON ISSUES:                                                 │
│  • Mixed content (HTTP resources on HTTPS page)                │
│  • Missing intermediate certificates                           │
│  • Certificate-hostname mismatch                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


