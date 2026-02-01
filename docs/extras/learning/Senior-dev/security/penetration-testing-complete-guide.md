# 🔍 Penetration Testing - Complete Guide

> A comprehensive guide to penetration testing - OWASP ZAP, vulnerability scanning, security audits, and finding vulnerabilities before attackers do.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Penetration testing is authorized simulated attacks on your system to find security vulnerabilities - the goal is to think like an attacker, find weaknesses before they do, and provide actionable remediation guidance."

### Penetration Testing Phases
```
PENETRATION TESTING PHASES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. RECONNAISSANCE (Information Gathering)                      │
│     • Passive: DNS, WHOIS, public info, social media           │
│     • Active: Port scanning, service detection                 │
│                                                                  │
│  2. SCANNING (Vulnerability Assessment)                         │
│     • Automated scanners: OWASP ZAP, Nessus, Burp Suite        │
│     • Identify known vulnerabilities, CVEs                     │
│                                                                  │
│  3. EXPLOITATION (Attack)                                       │
│     • Attempt to exploit found vulnerabilities                 │
│     • Prove impact (access data, escalate privileges)          │
│                                                                  │
│  4. POST-EXPLOITATION (Maintain Access)                         │
│     • Persistence, lateral movement                            │
│     • Assess true impact of breach                             │
│                                                                  │
│  5. REPORTING                                                   │
│     • Document findings with severity                          │
│     • Provide remediation recommendations                      │
│     • Executive summary + technical details                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement
> "I run quarterly penetration tests on our application. We use OWASP ZAP in our CI/CD pipeline for automated scanning on every deploy, catching issues early. For deep testing, I use Burp Suite to manually test authentication flows, authorization bypasses, and business logic flaws that scanners miss. Last quarter's pentest found an IDOR vulnerability where users could access other users' invoices by changing the ID - the scanner flagged the endpoint, but manual testing confirmed the authorization bypass. We track all findings in a vulnerability management system, prioritize by CVSS score, and aim for critical fixes within 24 hours."

---

## 📚 Tools & Techniques

### OWASP ZAP (Free)

```bash
# ════════════════════════════════════════════════════════════════
# OWASP ZAP - Zed Attack Proxy
# ════════════════════════════════════════════════════════════════

# Install ZAP (Docker)
docker pull owasp/zap2docker-stable

# Quick scan (baseline)
docker run -t owasp/zap2docker-stable zap-baseline.py \
    -t https://yourapp.com \
    -r report.html

# Full scan (more thorough, takes longer)
docker run -t owasp/zap2docker-stable zap-full-scan.py \
    -t https://yourapp.com \
    -r report.html

# API scan (for REST APIs)
docker run -t owasp/zap2docker-stable zap-api-scan.py \
    -t https://yourapp.com/openapi.json \
    -f openapi \
    -r report.html

# ════════════════════════════════════════════════════════════════
# ZAP IN CI/CD (GitHub Actions)
# ════════════════════════════════════════════════════════════════

# .github/workflows/security.yml
name: Security Scan
on: [push, pull_request]

jobs:
  zap-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Start application
        run: docker-compose up -d
        
      - name: Wait for app
        run: sleep 30
        
      - name: ZAP Scan
        uses: zaproxy/action-baseline@v0.10.0
        with:
          target: 'http://localhost:3000'
          rules_file_name: '.zap/rules.tsv'
          fail_action: true  # Fail if high severity issues found
          
      - name: Upload Report
        uses: actions/upload-artifact@v4
        with:
          name: zap-report
          path: report_html.html
```

### Burp Suite (Professional)

```
BURP SUITE WORKFLOW:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. CONFIGURE PROXY                                             │
│     • Set browser to use 127.0.0.1:8080                        │
│     • Install Burp's CA certificate in browser                 │
│                                                                  │
│  2. SPIDER/CRAWL                                                │
│     • Let Burp discover all endpoints                          │
│     • Manually browse to ensure coverage                       │
│                                                                  │
│  3. PASSIVE SCAN                                                │
│     • Analyzes traffic for issues                              │
│     • Low hanging fruit: missing headers, cookies              │
│                                                                  │
│  4. ACTIVE SCAN                                                 │
│     • Sends attack payloads                                    │
│     • Tests for SQLi, XSS, etc.                                │
│                                                                  │
│  5. MANUAL TESTING                                              │
│     • Repeater: Modify and resend requests                     │
│     • Intruder: Automated payload fuzzing                      │
│     • Test business logic flaws                                │
│                                                                  │
│  KEY TESTS:                                                     │
│  • Change user IDs in requests (IDOR)                          │
│  • Remove/modify authorization headers                         │
│  • Test parameter tampering (price, quantity)                  │
│  • Check for SQL injection in all inputs                       │
│  • Test file upload restrictions                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Common Vulnerability Tests

```typescript
// ════════════════════════════════════════════════════════════════
// IDOR (Insecure Direct Object Reference) TEST
// ════════════════════════════════════════════════════════════════

// Test: Can user A access user B's resources?

// 1. Login as User A
// 2. Get your own resource: GET /api/users/123/profile
// 3. Try accessing User B's resource: GET /api/users/456/profile
// 4. If you get User B's data → IDOR vulnerability!

// Automated test
async function testIDOR() {
    const userAToken = await login('userA@test.com', 'password');
    const userBId = '456';  // Different user's ID
    
    const response = await fetch(`/api/users/${userBId}/profile`, {
        headers: { Authorization: `Bearer ${userAToken}` }
    });
    
    if (response.ok) {
        console.error('IDOR VULNERABILITY: User A can access User B data!');
    }
}

// ════════════════════════════════════════════════════════════════
// BROKEN AUTHENTICATION TEST
// ════════════════════════════════════════════════════════════════

// Test: Brute force protection
async function testBruteForce() {
    const attempts = [];
    
    for (let i = 0; i < 20; i++) {
        const response = await fetch('/api/login', {
            method: 'POST',
            body: JSON.stringify({
                email: 'target@example.com',
                password: `wrong_password_${i}`
            })
        });
        
        attempts.push({
            attempt: i + 1,
            status: response.status,
            rateLimit: response.headers.get('X-RateLimit-Remaining')
        });
    }
    
    // Check if rate limiting kicked in
    const blocked = attempts.some(a => a.status === 429);
    if (!blocked) {
        console.error('VULNERABILITY: No brute force protection!');
    }
}

// ════════════════════════════════════════════════════════════════
// PRIVILEGE ESCALATION TEST
// ════════════════════════════════════════════════════════════════

// Test: Can regular user access admin functions?
async function testPrivilegeEscalation() {
    const userToken = await login('regular@test.com', 'password');
    
    // Try admin-only endpoints
    const adminEndpoints = [
        '/api/admin/users',
        '/api/admin/settings',
        '/api/admin/audit-logs'
    ];
    
    for (const endpoint of adminEndpoints) {
        const response = await fetch(endpoint, {
            headers: { Authorization: `Bearer ${userToken}` }
        });
        
        if (response.ok) {
            console.error(`PRIVILEGE ESCALATION: ${endpoint} accessible to regular user!`);
        }
    }
}
```

### Security Scanners

```bash
# ════════════════════════════════════════════════════════════════
# NMAP - Network scanning
# ════════════════════════════════════════════════════════════════

# Port scan
nmap -sV -sC target.com

# Scan all ports
nmap -p- target.com

# Vulnerability scan
nmap --script vuln target.com

# ════════════════════════════════════════════════════════════════
# NIKTO - Web server scanner
# ════════════════════════════════════════════════════════════════

nikto -h https://target.com

# ════════════════════════════════════════════════════════════════
# SQLMAP - SQL injection testing
# ════════════════════════════════════════════════════════════════

# Test URL parameter
sqlmap -u "https://target.com/page?id=1" --dbs

# Test POST parameter
sqlmap -u "https://target.com/login" --data="username=admin&password=test" --dbs

# ════════════════════════════════════════════════════════════════
# NUCLEI - Fast vulnerability scanner
# ════════════════════════════════════════════════════════════════

# Install
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest

# Scan with all templates
nuclei -u https://target.com

# Scan with specific templates
nuclei -u https://target.com -t cves/
nuclei -u https://target.com -t vulnerabilities/

# ════════════════════════════════════════════════════════════════
# TRIVY - Container vulnerability scanner
# ════════════════════════════════════════════════════════════════

# Scan Docker image
trivy image myapp:latest

# Scan filesystem
trivy fs .

# Scan IaC (Terraform, etc.)
trivy config ./terraform/
```

### Dependency Scanning

```yaml
# ════════════════════════════════════════════════════════════════
# DEPENDENCY VULNERABILITY SCANNING
# ════════════════════════════════════════════════════════════════

# GitHub Dependabot - .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 10
    labels:
      - "dependencies"
      - "security"

# ════════════════════════════════════════════════════════════════
# SNYK - CI/CD Integration
# ════════════════════════════════════════════════════════════════

# GitHub Actions
name: Security
on: push

jobs:
  snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

# ════════════════════════════════════════════════════════════════
# NPM AUDIT
# ════════════════════════════════════════════════════════════════

# Check for vulnerabilities
npm audit

# Auto-fix where possible
npm audit fix

# Check production only
npm audit --production

# CI check (fail on high severity)
npm audit --audit-level=high
```

---

## Vulnerability Severity (CVSS)

```
CVSS SEVERITY LEVELS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  CRITICAL (9.0 - 10.0)                                          │
│  • Remote code execution                                        │
│  • Full system compromise                                       │
│  • Unauthenticated access to sensitive data                    │
│  → Fix: Immediately (within 24 hours)                          │
│                                                                  │
│  HIGH (7.0 - 8.9)                                               │
│  • SQL injection                                                │
│  • Authentication bypass                                        │
│  • Privilege escalation                                        │
│  → Fix: Within 1 week                                          │
│                                                                  │
│  MEDIUM (4.0 - 6.9)                                             │
│  • Stored XSS                                                  │
│  • CSRF                                                        │
│  • Information disclosure                                      │
│  → Fix: Within 1 month                                         │
│                                                                  │
│  LOW (0.1 - 3.9)                                                │
│  • Reflected XSS (requires user interaction)                   │
│  • Missing security headers                                    │
│  • Verbose error messages                                      │
│  → Fix: Next release cycle                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "How do you approach penetration testing?"**
> "I follow a structured methodology: reconnaissance (understand the target), scanning (automated vulnerability detection), exploitation (prove vulnerabilities are real), and reporting (document findings with severity and remediation). I use tools like ZAP for automated scanning and Burp Suite for manual testing. Critical areas: authentication, authorization (especially IDOR), input validation, and business logic."

**Q: "How do you integrate security testing into CI/CD?"**
> "Multiple layers: SAST (static analysis) scans code for vulnerabilities on every commit. Dependency scanning (npm audit, Snyk) checks for known CVEs in packages. DAST (ZAP) runs against deployed staging environment. Container scanning (Trivy) checks Docker images. Pipeline fails on high/critical findings. This catches issues before production."

**Q: "What's the difference between vulnerability scanning and penetration testing?"**
> "Vulnerability scanning is automated, finds known issues (CVEs, misconfigurations), runs regularly, good for baseline. Penetration testing is manual, finds unknown issues (business logic flaws, complex attack chains), done periodically by skilled testers. Scanning finds 'what's wrong', pentesting finds 'how bad can it get'. You need both."

---

## Quick Reference

```
PENETRATION TESTING CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  AUTHENTICATION:                                                │
│  □ Brute force protection                                      │
│  □ Password policy enforcement                                 │
│  □ Session fixation                                            │
│  □ JWT/Token validation                                        │
│                                                                  │
│  AUTHORIZATION:                                                 │
│  □ IDOR (access other users' data)                             │
│  □ Privilege escalation                                        │
│  □ Horizontal access (user → user)                             │
│  □ Vertical access (user → admin)                              │
│                                                                  │
│  INPUT VALIDATION:                                              │
│  □ SQL injection                                               │
│  □ XSS (reflected, stored, DOM)                                │
│  □ Command injection                                           │
│  □ Path traversal                                              │
│                                                                  │
│  CONFIGURATION:                                                 │
│  □ Security headers                                            │
│  □ TLS configuration                                           │
│  □ Error handling (no stack traces)                            │
│  □ Dependency vulnerabilities                                  │
│                                                                  │
│  TOOLS:                                                         │
│  • OWASP ZAP (free DAST)                                       │
│  • Burp Suite (professional DAST)                              │
│  • Nuclei (vulnerability templates)                            │
│  • Snyk/npm audit (dependency scanning)                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


