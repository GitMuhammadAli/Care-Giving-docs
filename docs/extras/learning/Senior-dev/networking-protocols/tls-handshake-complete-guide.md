# TLS Handshake - Complete Guide

> **MUST REMEMBER**: TLS handshake establishes encrypted connection: client sends supported ciphers, server sends certificate, they exchange keys, then communicate encrypted. TLS 1.3 reduced handshake from 2 round-trips to 1 (and supports 0-RTT). Certificate chain: your cert → intermediate CA → root CA. Certificate pinning prevents MITM but complicates rotation. Always use TLS 1.2+ and disable older versions.

---

## How to Explain Like a Senior Developer

"TLS is how we encrypt web traffic. The handshake negotiates encryption before any data is sent. Client says 'I support these ciphers', server picks one and sends its certificate. Client verifies the certificate chain (your cert signed by intermediate CA, signed by root CA in their trust store). They do a key exchange (Diffie-Hellman) so both sides have a shared secret, then all traffic is encrypted. TLS 1.3 streamlined this to one round-trip and removed insecure options. Certificate pinning locks your app to specific certificates, preventing MITM even if a CA is compromised - but you need to plan for rotation. Modern best practice: TLS 1.2+, strong ciphers only, HSTS header, and certificate transparency monitoring."

---

## Core Implementation

### TLS Server in Node.js

```typescript
// tls/server.ts
import * as tls from 'tls';
import * as fs from 'fs';

interface TlsServerOptions {
  keyPath: string;
  certPath: string;
  caPath?: string; // For client certificate auth
  port: number;
}

function createTlsServer(options: TlsServerOptions): tls.Server {
  const serverOptions: tls.TlsOptions = {
    key: fs.readFileSync(options.keyPath),
    cert: fs.readFileSync(options.certPath),
    
    // Minimum TLS version (disable TLS 1.0, 1.1)
    minVersion: 'TLSv1.2',
    
    // Preferred cipher suites (TLS 1.3 ciphers auto-selected)
    ciphers: [
      'TLS_AES_256_GCM_SHA384',         // TLS 1.3
      'TLS_CHACHA20_POLY1305_SHA256',   // TLS 1.3
      'TLS_AES_128_GCM_SHA256',         // TLS 1.3
      'ECDHE-ECDSA-AES256-GCM-SHA384',  // TLS 1.2
      'ECDHE-RSA-AES256-GCM-SHA384',    // TLS 1.2
      'ECDHE-ECDSA-CHACHA20-POLY1305',  // TLS 1.2
      'ECDHE-RSA-CHACHA20-POLY1305',    // TLS 1.2
    ].join(':'),
    
    // Honor server's cipher preference
    honorCipherOrder: true,
    
    // Session resumption
    sessionTimeout: 300,
    
    // ECDH curve
    ecdhCurve: 'auto',
  };
  
  // Optional: Mutual TLS (client certificate)
  if (options.caPath) {
    serverOptions.ca = fs.readFileSync(options.caPath);
    serverOptions.requestCert = true;
    serverOptions.rejectUnauthorized = true;
  }
  
  const server = tls.createServer(serverOptions, (socket) => {
    console.log('TLS connection established');
    console.log('Protocol:', socket.getProtocol());
    console.log('Cipher:', socket.getCipher());
    
    // Client certificate info (if mutual TLS)
    if (socket.authorized) {
      const cert = socket.getPeerCertificate();
      console.log('Client CN:', cert.subject.CN);
    }
    
    socket.on('data', (data) => {
      console.log('Received:', data.toString());
      socket.write(`Echo: ${data.toString()}`);
    });
  });
  
  server.on('tlsClientError', (err, socket) => {
    console.error('TLS client error:', err.message);
  });
  
  server.listen(options.port, () => {
    console.log(`TLS server listening on port ${options.port}`);
  });
  
  return server;
}

// Usage
createTlsServer({
  keyPath: './server.key',
  certPath: './server.crt',
  port: 8443,
});
```

### TLS Client with Certificate Verification

```typescript
// tls/client.ts
import * as tls from 'tls';
import * as fs from 'fs';
import * as crypto from 'crypto';

interface TlsClientOptions {
  host: string;
  port: number;
  ca?: string; // Custom CA certificate
  rejectUnauthorized?: boolean;
  clientCert?: string; // For mutual TLS
  clientKey?: string;
  checkServerIdentity?: (host: string, cert: tls.PeerCertificate) => Error | undefined;
}

async function connectTls(options: TlsClientOptions): Promise<tls.TLSSocket> {
  return new Promise((resolve, reject) => {
    const tlsOptions: tls.ConnectionOptions = {
      host: options.host,
      port: options.port,
      minVersion: 'TLSv1.2',
      rejectUnauthorized: options.rejectUnauthorized ?? true,
    };
    
    // Custom CA (for self-signed or private CA)
    if (options.ca) {
      tlsOptions.ca = fs.readFileSync(options.ca);
    }
    
    // Client certificate (mutual TLS)
    if (options.clientCert && options.clientKey) {
      tlsOptions.cert = fs.readFileSync(options.clientCert);
      tlsOptions.key = fs.readFileSync(options.clientKey);
    }
    
    // Custom server identity check (for pinning)
    if (options.checkServerIdentity) {
      tlsOptions.checkServerIdentity = options.checkServerIdentity;
    }
    
    const socket = tls.connect(tlsOptions, () => {
      if (!socket.authorized && options.rejectUnauthorized !== false) {
        reject(new Error(`TLS authorization failed: ${socket.authorizationError}`));
        return;
      }
      
      console.log('Connected securely');
      console.log('Protocol:', socket.getProtocol());
      console.log('Cipher:', socket.getCipher());
      
      resolve(socket);
    });
    
    socket.on('error', reject);
  });
}

// Usage
async function main() {
  const socket = await connectTls({
    host: 'example.com',
    port: 443,
  });
  
  socket.write('GET / HTTP/1.1\r\nHost: example.com\r\n\r\n');
  
  socket.on('data', (data) => {
    console.log(data.toString());
  });
}
```

### Certificate Pinning

```typescript
// tls/certificate-pinning.ts
import * as tls from 'tls';
import * as crypto from 'crypto';
import * as https from 'https';

/**
 * Certificate Pinning Strategies:
 * 1. Pin the certificate itself (most secure, hardest to rotate)
 * 2. Pin the public key (survives cert renewal if key unchanged)
 * 3. Pin the CA (allows any cert from that CA)
 */

// Calculate public key hash (SPKI - Subject Public Key Info)
function getPublicKeyHash(cert: tls.PeerCertificate): string {
  const pubkey = cert.pubkey;
  if (!pubkey) {
    throw new Error('No public key in certificate');
  }
  
  const hash = crypto.createHash('sha256');
  hash.update(pubkey);
  return hash.digest('base64');
}

// Pin public key hashes
const pinnedKeys = new Set([
  'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=', // Current key
  'BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=', // Backup key
]);

function checkPinning(
  hostname: string,
  cert: tls.PeerCertificate
): Error | undefined {
  // First, do normal hostname verification
  const err = tls.checkServerIdentity(hostname, cert);
  if (err) return err;
  
  // Then check pinning
  const keyHash = getPublicKeyHash(cert);
  
  if (!pinnedKeys.has(keyHash)) {
    return new Error(
      `Certificate pinning failed. Key hash: ${keyHash}`
    );
  }
  
  return undefined;
}

// HTTPS agent with pinning
const pinnedAgent = new https.Agent({
  checkServerIdentity: checkPinning,
});

// Usage with fetch (Node 18+)
async function fetchWithPinning(url: string): Promise<Response> {
  return fetch(url, {
    // @ts-ignore - dispatcher option
    dispatcher: pinnedAgent,
  });
}

// Usage with https module
function httpsWithPinning(url: string): Promise<string> {
  return new Promise((resolve, reject) => {
    const req = https.get(url, { agent: pinnedAgent }, (res) => {
      let data = '';
      res.on('data', chunk => data += chunk);
      res.on('end', () => resolve(data));
    });
    
    req.on('error', reject);
  });
}

// Certificate transparency check
async function checkCertificateTransparency(domain: string): Promise<boolean> {
  // In practice, use a CT log API
  // This is a simplified example
  const response = await fetch(
    `https://crt.sh/?q=${domain}&output=json`
  );
  const certs = await response.json();
  
  // Check for unexpected certificates
  console.log(`Found ${certs.length} certificates for ${domain}`);
  
  return true;
}
```

### TLS 1.3 Specific Features

```typescript
// tls/tls13-features.ts
import * as tls from 'tls';
import * as https from 'https';
import * as fs from 'fs';

/**
 * TLS 1.3 Improvements:
 * 1. 1-RTT handshake (vs 2-RTT in TLS 1.2)
 * 2. 0-RTT resumption (early data)
 * 3. Removed insecure ciphers
 * 4. Encrypted handshake (hides more metadata)
 * 5. Simplified cipher suites
 */

// TLS 1.3 server with 0-RTT
function createTls13Server(): https.Server {
  const options: https.ServerOptions = {
    key: fs.readFileSync('./server.key'),
    cert: fs.readFileSync('./server.crt'),
    minVersion: 'TLSv1.3',
    maxVersion: 'TLSv1.3',
    
    // Enable session tickets for resumption
    ticketKeys: crypto.randomBytes(48),
  };
  
  return https.createServer(options, (req, res) => {
    // Check if this was 0-RTT early data
    const socket = req.socket as tls.TLSSocket;
    
    // Note: Node.js doesn't directly expose early data info
    // This is conceptual
    
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      protocol: socket.getProtocol(),
      cipher: socket.getCipher(),
    }));
  });
}

// Session resumption client
class TlsSessionClient {
  private session: Buffer | null = null;
  
  async connect(host: string, port: number): Promise<tls.TLSSocket> {
    return new Promise((resolve, reject) => {
      const options: tls.ConnectionOptions = {
        host,
        port,
        minVersion: 'TLSv1.3',
        session: this.session || undefined, // Reuse session
      };
      
      const socket = tls.connect(options, () => {
        console.log('Session reused:', socket.isSessionReused());
        resolve(socket);
      });
      
      // Save session for next connection
      socket.on('session', (session) => {
        this.session = session;
        console.log('Session ticket received');
      });
      
      socket.on('error', reject);
    });
  }
}

import * as crypto from 'crypto';
```

### Certificate Chain Verification

```typescript
// tls/certificate-chain.ts
import * as tls from 'tls';
import * as https from 'https';
import * as crypto from 'crypto';

interface CertificateInfo {
  subject: string;
  issuer: string;
  validFrom: Date;
  validTo: Date;
  serialNumber: string;
  fingerprint: string;
  isCA: boolean;
}

// Get full certificate chain
async function getCertificateChain(hostname: string): Promise<CertificateInfo[]> {
  return new Promise((resolve, reject) => {
    const options = {
      host: hostname,
      port: 443,
      method: 'GET',
      rejectUnauthorized: false, // Accept to inspect chain
    };
    
    const req = https.request(options, (res) => {
      const socket = res.socket as tls.TLSSocket;
      const chain: CertificateInfo[] = [];
      
      // Get leaf certificate
      let cert = socket.getPeerCertificate(true);
      
      while (cert && Object.keys(cert).length > 0) {
        chain.push({
          subject: cert.subject?.CN || 'Unknown',
          issuer: cert.issuer?.CN || 'Unknown',
          validFrom: new Date(cert.valid_from),
          validTo: new Date(cert.valid_to),
          serialNumber: cert.serialNumber,
          fingerprint: cert.fingerprint256,
          isCA: cert.issuer?.CN !== cert.subject?.CN,
        });
        
        // Move to issuer certificate
        cert = (cert as any).issuerCertificate;
        
        // Prevent infinite loop at root
        if (cert === (cert as any).issuerCertificate) break;
      }
      
      resolve(chain);
    });
    
    req.on('error', reject);
    req.end();
  });
}

// Verify certificate chain
function verifyCertificateChain(chain: CertificateInfo[]): {
  valid: boolean;
  errors: string[];
} {
  const errors: string[] = [];
  const now = new Date();
  
  for (let i = 0; i < chain.length; i++) {
    const cert = chain[i];
    
    // Check validity period
    if (now < cert.validFrom) {
      errors.push(`Certificate "${cert.subject}" not yet valid`);
    }
    if (now > cert.validTo) {
      errors.push(`Certificate "${cert.subject}" has expired`);
    }
    
    // Check chain linkage (issuer matches next cert's subject)
    if (i < chain.length - 1) {
      const issuerCert = chain[i + 1];
      if (cert.issuer !== issuerCert.subject) {
        errors.push(`Chain break: "${cert.subject}" issued by "${cert.issuer}" but next cert is "${issuerCert.subject}"`);
      }
    }
  }
  
  return {
    valid: errors.length === 0,
    errors,
  };
}

// Check certificate expiration
async function checkExpirationDays(hostname: string): Promise<number> {
  const chain = await getCertificateChain(hostname);
  const leafCert = chain[0];
  
  const now = new Date();
  const daysUntilExpiry = Math.floor(
    (leafCert.validTo.getTime() - now.getTime()) / (1000 * 60 * 60 * 24)
  );
  
  return daysUntilExpiry;
}

// Usage
async function main() {
  const chain = await getCertificateChain('google.com');
  
  console.log('Certificate Chain:');
  chain.forEach((cert, i) => {
    console.log(`  ${i + 1}. ${cert.subject} (issued by: ${cert.issuer})`);
    console.log(`     Valid: ${cert.validFrom.toISOString()} to ${cert.validTo.toISOString()}`);
  });
  
  const verification = verifyCertificateChain(chain);
  console.log('Chain valid:', verification.valid);
  
  if (!verification.valid) {
    console.log('Errors:', verification.errors);
  }
}
```

---

## Real-World Scenarios

### Scenario 1: Mutual TLS (mTLS) for Service-to-Service

```typescript
// mtls/service-auth.ts
import * as https from 'https';
import * as fs from 'fs';

/**
 * Mutual TLS: Both client and server authenticate with certificates
 * Common in microservices for zero-trust security
 */

// mTLS Server
function createMtlsServer(options: {
  port: number;
  serverCert: string;
  serverKey: string;
  clientCA: string; // CA that signs client certs
}): https.Server {
  const server = https.createServer({
    key: fs.readFileSync(options.serverKey),
    cert: fs.readFileSync(options.serverCert),
    ca: fs.readFileSync(options.clientCA),
    requestCert: true,       // Request client certificate
    rejectUnauthorized: true, // Reject if no valid cert
  }, (req, res) => {
    const socket = req.socket as tls.TLSSocket;
    const clientCert = socket.getPeerCertificate();
    
    // Verify client identity
    const clientCN = clientCert.subject?.CN;
    console.log(`Authenticated client: ${clientCN}`);
    
    // Authorization based on client certificate
    const allowedServices = ['payment-service', 'order-service'];
    if (!allowedServices.includes(clientCN)) {
      res.writeHead(403);
      res.end('Forbidden');
      return;
    }
    
    res.writeHead(200);
    res.end(`Hello, ${clientCN}!`);
  });
  
  return server.listen(options.port);
}

// mTLS Client
async function mtlsRequest(options: {
  url: string;
  clientCert: string;
  clientKey: string;
  serverCA: string;
}): Promise<string> {
  return new Promise((resolve, reject) => {
    const urlObj = new URL(options.url);
    
    const req = https.request({
      hostname: urlObj.hostname,
      port: urlObj.port || 443,
      path: urlObj.pathname,
      method: 'GET',
      cert: fs.readFileSync(options.clientCert),
      key: fs.readFileSync(options.clientKey),
      ca: fs.readFileSync(options.serverCA),
    }, (res) => {
      let data = '';
      res.on('data', chunk => data += chunk);
      res.on('end', () => resolve(data));
    });
    
    req.on('error', reject);
    req.end();
  });
}

import * as tls from 'tls';
```

### Scenario 2: TLS Configuration Audit

```typescript
// tls/audit.ts
import * as tls from 'tls';

interface TlsAuditResult {
  host: string;
  port: number;
  protocol: string;
  cipher: string;
  certificateExpiry: Date;
  issues: string[];
  recommendations: string[];
}

async function auditTlsConfiguration(
  host: string,
  port: number = 443
): Promise<TlsAuditResult> {
  const issues: string[] = [];
  const recommendations: string[] = [];
  
  return new Promise((resolve, reject) => {
    const socket = tls.connect({
      host,
      port,
      rejectUnauthorized: false, // Accept to audit
    }, () => {
      const protocol = socket.getProtocol();
      const cipher = socket.getCipher();
      const cert = socket.getPeerCertificate();
      
      // Check protocol version
      if (protocol === 'TLSv1' || protocol === 'TLSv1.1') {
        issues.push(`Insecure protocol: ${protocol}`);
        recommendations.push('Upgrade to TLS 1.2 or 1.3');
      }
      
      if (protocol !== 'TLSv1.3') {
        recommendations.push('Consider enabling TLS 1.3');
      }
      
      // Check cipher strength
      const weakCiphers = ['RC4', 'DES', '3DES', 'MD5'];
      if (weakCiphers.some(weak => cipher.name.includes(weak))) {
        issues.push(`Weak cipher: ${cipher.name}`);
        recommendations.push('Use AES-GCM or ChaCha20-Poly1305');
      }
      
      if (!cipher.name.includes('GCM') && !cipher.name.includes('CHACHA')) {
        recommendations.push('Prefer AEAD ciphers (GCM, ChaCha20-Poly1305)');
      }
      
      // Check key exchange
      if (!cipher.name.includes('ECDHE') && !cipher.name.includes('DHE')) {
        issues.push('No forward secrecy');
        recommendations.push('Use ECDHE or DHE key exchange');
      }
      
      // Check certificate
      const now = new Date();
      const validTo = new Date(cert.valid_to);
      const daysUntilExpiry = Math.floor(
        (validTo.getTime() - now.getTime()) / (1000 * 60 * 60 * 24)
      );
      
      if (daysUntilExpiry < 0) {
        issues.push('Certificate has expired');
      } else if (daysUntilExpiry < 30) {
        issues.push(`Certificate expires in ${daysUntilExpiry} days`);
        recommendations.push('Renew certificate soon');
      }
      
      // Check key size
      if (cert.bits && cert.bits < 2048) {
        issues.push(`Weak key size: ${cert.bits} bits`);
        recommendations.push('Use at least 2048-bit RSA or 256-bit ECDSA');
      }
      
      socket.end();
      
      resolve({
        host,
        port,
        protocol: protocol || 'unknown',
        cipher: cipher.name,
        certificateExpiry: validTo,
        issues,
        recommendations,
      });
    });
    
    socket.on('error', reject);
  });
}

// Usage
async function runAudit() {
  const result = await auditTlsConfiguration('example.com');
  
  console.log('TLS Audit Results:');
  console.log(`Protocol: ${result.protocol}`);
  console.log(`Cipher: ${result.cipher}`);
  console.log(`Cert expires: ${result.certificateExpiry}`);
  
  if (result.issues.length > 0) {
    console.log('\nIssues:');
    result.issues.forEach(i => console.log(`  ⚠️ ${i}`));
  }
  
  if (result.recommendations.length > 0) {
    console.log('\nRecommendations:');
    result.recommendations.forEach(r => console.log(`  💡 ${r}`));
  }
}
```

---

## Common Pitfalls

### 1. Disabling Certificate Verification

```typescript
// ❌ BAD: Disabling verification in production
const agent = new https.Agent({
  rejectUnauthorized: false, // NEVER in production!
});

// ✅ GOOD: Proper certificate handling
const agent = new https.Agent({
  rejectUnauthorized: true,
  ca: fs.readFileSync('./custom-ca.crt'), // If using private CA
});
```

### 2. Not Handling Certificate Expiration

```typescript
// ❌ BAD: No expiration monitoring
// Cert expires, site goes down!

// ✅ GOOD: Monitor and alert before expiration
async function checkCertExpiry(domain: string): Promise<void> {
  const days = await checkExpirationDays(domain);
  
  if (days < 7) {
    await sendAlert(`CRITICAL: ${domain} cert expires in ${days} days`);
  } else if (days < 30) {
    await sendAlert(`WARNING: ${domain} cert expires in ${days} days`);
  }
}

// Run daily
setInterval(() => {
  checkCertExpiry('example.com');
}, 24 * 60 * 60 * 1000);

async function checkExpirationDays(domain: string): Promise<number> { return 0; }
async function sendAlert(msg: string): Promise<void> {}
```

### 3. Using Outdated TLS Versions

```typescript
// ❌ BAD: Allowing TLS 1.0/1.1
const server = https.createServer({
  key: fs.readFileSync('./key.pem'),
  cert: fs.readFileSync('./cert.pem'),
  // No minVersion - allows TLS 1.0!
});

// ✅ GOOD: Enforce TLS 1.2+
const server = https.createServer({
  key: fs.readFileSync('./key.pem'),
  cert: fs.readFileSync('./cert.pem'),
  minVersion: 'TLSv1.2',
});
```

---

## Interview Questions

### Q1: Explain the TLS 1.2 handshake process.

**A:** 1) Client Hello: client sends supported versions, ciphers, random bytes. 2) Server Hello: server picks cipher, sends its random bytes. 3) Certificate: server sends certificate chain. 4) Server Key Exchange: server sends DH parameters (for forward secrecy). 5) Client Key Exchange: client sends its DH value, both derive shared secret. 6) Change Cipher Spec + Finished: both switch to encrypted communication. Takes 2 round-trips.

### Q2: How does TLS 1.3 improve on TLS 1.2?

**A:** 1) 1-RTT handshake (vs 2-RTT) - faster connection. 2) 0-RTT resumption for returning clients (with replay protection). 3) Removed insecure algorithms (RSA key exchange, CBC mode). 4) Encrypted more of the handshake (hides certificate, etc.). 5) Simplified cipher suites - only 5 approved. 6) Mandatory forward secrecy.

### Q3: What is certificate pinning and when should you use it?

**A:** Pinning hardcodes expected certificate or public key in your application, rejecting any other even if CA-signed. Use when: you control both client and server, high-security applications, can manage rotation. Avoid for: public websites (breaks if cert changes), long-lived mobile apps (hard to update). Consider pinning intermediate CA as compromise.

### Q4: What is forward secrecy and why is it important?

**A:** Forward secrecy means if the server's private key is compromised later, past recorded sessions can't be decrypted. Achieved via ephemeral Diffie-Hellman (DHE/ECDHE) - each session generates unique keys. Important because attackers may record traffic now and decrypt later when they get the key. RSA key exchange doesn't have forward secrecy.

---

## Quick Reference Checklist

### Server Configuration
- [ ] Minimum TLS 1.2 (prefer 1.3)
- [ ] Strong cipher suites only (AES-GCM, ChaCha20)
- [ ] Forward secrecy enabled (ECDHE)
- [ ] Certificate chain complete
- [ ] HSTS header enabled

### Certificate Management
- [ ] Monitor expiration dates
- [ ] Automate renewal (Let's Encrypt)
- [ ] Use strong key (2048-bit RSA or 256-bit ECDSA)
- [ ] CAA records configured
- [ ] Certificate Transparency monitoring

### Client Security
- [ ] Verify certificates (don't disable)
- [ ] Pin certificates for high-security apps
- [ ] Handle certificate errors gracefully
- [ ] Update root CA store

### Compliance
- [ ] PCI DSS: TLS 1.2+
- [ ] Disable weak ciphers
- [ ] Regular security audits
- [ ] Document TLS configuration

---

*Last updated: February 2026*

