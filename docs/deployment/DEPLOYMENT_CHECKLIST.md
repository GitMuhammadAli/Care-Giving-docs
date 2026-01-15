# 🚀 Production Deployment Checklist - Missing DevOps Items

## What's Missing from the Security Engineering Document

After reviewing the security engineering documentation, here are the **critical DevOps and deployment items that are MISSING** and need to be added:

---

## 🔴 **CRITICAL MISSING ITEMS**

### 1. **CI/CD Pipeline** ⚠️ **MISSING**

**What you need**:

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: npm test
      - name: Security scan
        run: npm audit

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      - name: Push to registry
        run: docker push myapp:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to production
        run: |
          ssh production-server "docker pull myapp:${{ github.sha }}"
          ssh production-server "docker-compose up -d"
```

**Why it's critical**:
- ❌ No automated testing before deployment
- ❌ No automated security scans
- ❌ Manual deployment is error-prone
- ❌ No rollback mechanism
- ❌ No deployment history

---

### 2. **Container Orchestration (Docker/Kubernetes)** ⚠️ **MISSING**

**Current**: Manual PM2 deployment
**What you need**: Docker + Kubernetes or Docker Swarm

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    image: carecircle-api:latest
    restart: always
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure
        max_attempts: 3

  web:
    image: carecircle-web:latest
    restart: always
    depends_on:
      - api

  postgres:
    image: postgres:16
    restart: always
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=${DB_PASSWORD}

  redis:
    image: redis:7
    restart: always
    command: redis-server --requirepass ${REDIS_PASSWORD}
```

**Kubernetes alternative**:
```yaml
# k8s/deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: carecircle-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: carecircle-api
  template:
    metadata:
      labels:
        app: carecircle-api
    spec:
      containers:
      - name: api
        image: carecircle-api:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

**Why it's critical**:
- ❌ No blue-green deployment
- ❌ No zero-downtime updates
- ❌ No automatic scaling
- ❌ No health check-based restart
- ❌ Inconsistent environments (dev vs prod)

---

### 3. **Monitoring & Observability** ⚠️ **MISSING**

**What you need**:

#### **Application Monitoring (APM)**:
```javascript
// Add to your app
const Sentry = require('@sentry/node');

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
});

// Error tracking
app.use(Sentry.Handlers.errorHandler());

// Performance monitoring
Sentry.startTransaction({
  op: "http.server",
  name: "User Login",
});
```

#### **Metrics Collection**:
```javascript
// Prometheus metrics
const promClient = require('prom-client');

const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
});

const activeConnections = new promClient.Gauge({
  name: 'active_connections',
  help: 'Number of active connections',
});

// Expose metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', promClient.register.contentType);
  res.end(await promClient.register.metrics());
});
```

#### **Logging Stack**:
```yaml
# docker-compose.yml for ELK stack
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports:
      - "5601:5601"
```

**What's missing**:
- ❌ No centralized logging (ELK/Loki)
- ❌ No metrics dashboard (Grafana)
- ❌ No error tracking (Sentry)
- ❌ No APM (Application Performance Monitoring)
- ❌ No uptime monitoring
- ❌ No alerting system

---

### 4. **Secrets Management** ⚠️ **PARTIALLY COVERED**

**Current**: Environment variables only
**What you need**: Vault, AWS Secrets Manager, or similar

```javascript
// Using HashiCorp Vault
const vault = require('node-vault')({
  endpoint: process.env.VAULT_ADDR,
  token: process.env.VAULT_TOKEN,
});

async function getSecrets() {
  const result = await vault.read('secret/data/carecircle/prod');
  return result.data.data;
}

// Automatic secret rotation
async function rotateSecret() {
  const newPassword = generateSecurePassword();

  // Update in Vault
  await vault.write('secret/data/carecircle/prod', {
    data: { db_password: newPassword }
  });

  // Update in database
  await updateDatabasePassword(newPassword);

  // Restart applications
  await rollingRestart();
}
```

**What's missing**:
- ❌ No automatic secrets rotation
- ❌ No secrets versioning
- ❌ No audit trail for secret access
- ❌ Secrets in plain .env files

---

### 5. **Backup & Disaster Recovery** ⚠️ **PARTIALLY COVERED**

**Current**: Manual backup command shown
**What you need**: Automated backup strategy

```bash
#!/bin/bash
# backup.sh - Run daily via cron

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/postgres"
S3_BUCKET="s3://carecircle-backups"

# Full database backup
pg_dump -h $DB_HOST -U $DB_USER $DB_NAME | \
  gzip > "$BACKUP_DIR/db_backup_$DATE.sql.gz"

# Upload to S3 with encryption
aws s3 cp "$BACKUP_DIR/db_backup_$DATE.sql.gz" \
  "$S3_BUCKET/postgres/" \
  --storage-class GLACIER \
  --server-side-encryption AES256

# Verify backup
gunzip -t "$BACKUP_DIR/db_backup_$DATE.sql.gz"

# Cleanup old local backups (keep 7 days)
find $BACKUP_DIR -name "db_backup_*.sql.gz" -mtime +7 -delete

# Send success notification
curl -X POST $SLACK_WEBHOOK_URL \
  -d "{\"text\": \"✅ Database backup completed: $DATE\"}"
```

**Disaster Recovery Plan**:
```bash
#!/bin/bash
# disaster-recovery.sh

# 1. Stop applications
docker-compose down

# 2. Restore database from latest backup
LATEST_BACKUP=$(aws s3 ls $S3_BUCKET/postgres/ | sort | tail -1)
aws s3 cp "$S3_BUCKET/postgres/$LATEST_BACKUP" /tmp/restore.sql.gz
gunzip /tmp/restore.sql.gz

# 3. Drop and recreate database
psql -h $DB_HOST -U postgres -c "DROP DATABASE carecircle;"
psql -h $DB_HOST -U postgres -c "CREATE DATABASE carecircle;"

# 4. Restore data
psql -h $DB_HOST -U $DB_USER carecircle < /tmp/restore.sql

# 5. Verify data integrity
psql -h $DB_HOST -U $DB_USER carecircle -c "SELECT COUNT(*) FROM users;"

# 6. Restart applications
docker-compose up -d

# 7. Run smoke tests
./smoke-tests.sh

echo "Recovery completed. RTO: $(date)" >> /var/log/recovery.log
```

**What's missing**:
- ❌ No automated daily backups
- ❌ No backup verification
- ❌ No off-site backup storage
- ❌ No documented Recovery Time Objective (RTO)
- ❌ No documented Recovery Point Objective (RPO)
- ❌ No disaster recovery drills

---

### 6. **Auto-Scaling** ⚠️ **MISSING**

**What you need**:

```yaml
# Kubernetes HPA (Horizontal Pod Autoscaler)
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: carecircle-api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: carecircle-api
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
```

**What's missing**:
- ❌ No automatic scaling based on load
- ❌ No metric-based scaling
- ❌ Manual capacity planning only

---

### 7. **Health Checks & Self-Healing** ⚠️ **PARTIALLY COVERED**

**Current**: Basic /health endpoint
**What you need**: Comprehensive health checks

```javascript
// Enhanced health check
app.get('/health', async (req, res) => {
  const checks = {
    uptime: process.uptime(),
    timestamp: Date.now(),
    status: 'healthy',
    checks: {}
  };

  try {
    // Database check
    const dbStart = Date.now();
    await db.query('SELECT 1');
    checks.checks.database = {
      status: 'up',
      responseTime: Date.now() - dbStart
    };

    // Redis check
    const redisStart = Date.now();
    await redis.ping();
    checks.checks.redis = {
      status: 'up',
      responseTime: Date.now() - redisStart
    };

    // Disk space check
    const diskSpace = await checkDiskSpace('/');
    checks.checks.disk = {
      status: diskSpace.free > 1024 * 1024 * 1024 ? 'up' : 'warning',
      freeSpace: diskSpace.free,
      percentage: diskSpace.percent
    };

    // Memory check
    const memoryUsage = process.memoryUsage();
    checks.checks.memory = {
      status: memoryUsage.heapUsed < memoryUsage.heapTotal * 0.9 ? 'up' : 'warning',
      heapUsed: memoryUsage.heapUsed,
      heapTotal: memoryUsage.heapTotal
    };

    res.json(checks);
  } catch (error) {
    checks.status = 'unhealthy';
    checks.error = error.message;
    res.status(503).json(checks);
  }
});

// Readiness check (for load balancer)
app.get('/ready', async (req, res) => {
  try {
    await db.query('SELECT 1');
    res.status(200).send('Ready');
  } catch (error) {
    res.status(503).send('Not ready');
  }
});

// Liveness check (for restart decision)
app.get('/live', (req, res) => {
  res.status(200).send('Alive');
});
```

**What's missing**:
- ❌ No comprehensive dependency checks
- ❌ No graceful degradation
- ❌ No circuit breakers

---

### 8. **Infrastructure as Code (IaC)** ⚠️ **PARTIALLY COVERED**

**Current**: Manual OCI CLI commands
**What you need**: Terraform or Pulumi

```hcl
# terraform/main.tf
terraform {
  required_providers {
    oci = {
      source  = "oracle/oci"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket = "terraform-state"
    key    = "carecircle/prod/terraform.tfstate"
    region = "us-east-1"
  }
}

# VCN
resource "oci_core_vcn" "main" {
  compartment_id = var.compartment_id
  display_name   = "carecircle-vcn"
  cidr_blocks    = ["10.0.0.0/16"]
  dns_label      = "carecircle"
}

# Public Subnet
resource "oci_core_subnet" "public" {
  vcn_id         = oci_core_vcn.main.id
  cidr_block     = "10.0.1.0/24"
  display_name   = "public-subnet"
  route_table_id = oci_core_route_table.public.id
}

# Load Balancer
resource "oci_load_balancer_load_balancer" "main" {
  compartment_id = var.compartment_id
  display_name   = "carecircle-lb"
  shape          = "flexible"
  subnet_ids     = [oci_core_subnet.public.id]

  shape_details {
    minimum_bandwidth_in_mbps = 10
    maximum_bandwidth_in_mbps = 100
  }
}
```

**What's missing**:
- ❌ No version-controlled infrastructure
- ❌ No infrastructure testing
- ❌ No infrastructure change approval process
- ❌ Manual infrastructure changes

---

### 9. **Database Migration Strategy** ⚠️ **MISSING**

**What you need**:

```javascript
// Using knex.js or typeorm migrations
// migrations/20240115_add_user_roles.js
exports.up = async function(knex) {
  await knex.schema.alterTable('users', (table) => {
    table.string('role').defaultTo('USER');
    table.index('role');
  });

  // Data migration
  await knex('users')
    .where('is_admin', true)
    .update({ role: 'ADMIN' });
};

exports.down = async function(knex) {
  await knex.schema.alterTable('users', (table) => {
    table.dropColumn('role');
  });
};
```

**Migration deployment script**:
```bash
#!/bin/bash
# deploy-with-migration.sh

# 1. Backup database
./backup.sh

# 2. Put app in maintenance mode
curl -X POST $API_URL/admin/maintenance -H "X-Admin-Token: $TOKEN"

# 3. Run migrations
npm run migrate:up

# 4. Verify migration
if [ $? -eq 0 ]; then
  echo "Migration successful"
else
  echo "Migration failed, rolling back"
  npm run migrate:down
  exit 1
fi

# 5. Deploy new application version
./deploy.sh

# 6. Run smoke tests
./smoke-tests.sh

# 7. Exit maintenance mode
curl -X DELETE $API_URL/admin/maintenance -H "X-Admin-Token: $TOKEN"
```

**What's missing**:
- ❌ No automated database migrations
- ❌ No migration rollback strategy
- ❌ No zero-downtime migration strategy
- ❌ No migration testing in staging

---

### 10. **Rate Limiting & DDoS Protection** ⚠️ **PARTIALLY COVERED**

**Current**: Basic rate limiting
**What you need**: Multi-layer protection

```javascript
// Redis-backed distributed rate limiting
const Redis = require('ioredis');
const { RateLimiterRedis } = require('rate-limiter-flexible');

const redis = new Redis({
  host: process.env.REDIS_HOST,
  port: 6379,
  enableOfflineQueue: false,
});

const rateLimiter = new RateLimiterRedis({
  storeClient: redis,
  keyPrefix: 'ratelimit',
  points: 100, // requests
  duration: 60, // per 60 seconds
  blockDuration: 300, // block for 5 minutes if exceeded
});

// Per-IP rate limiting
const rateLimiterMiddleware = async (req, res, next) => {
  try {
    const ip = req.ip;
    await rateLimiter.consume(ip);
    next();
  } catch (error) {
    res.status(429).json({
      error: 'Too Many Requests',
      retryAfter: error.msBeforeNext / 1000
    });
  }
};

// Per-user rate limiting (authenticated)
const userRateLimiter = new RateLimiterRedis({
  storeClient: redis,
  keyPrefix: 'ratelimit:user',
  points: 1000,
  duration: 3600,
});
```

**Cloudflare WAF rules**:
```javascript
// Cloudflare Workers script for advanced protection
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request));
});

async function handleRequest(request) {
  const url = new URL(request.url);

  // Block known bad IPs
  const clientIP = request.headers.get('CF-Connecting-IP');
  const isBlacklisted = await checkBlacklist(clientIP);
  if (isBlacklisted) {
    return new Response('Forbidden', { status: 403 });
  }

  // Challenge suspicious traffic
  const threatScore = request.cf.threatScore;
  if (threatScore > 50) {
    return new Response('Please complete CAPTCHA', {
      status: 403,
      headers: { 'Content-Type': 'text/html' }
    });
  }

  // Rate limit by country
  const country = request.cf.country;
  if (country === 'CN' || country === 'RU') {
    // Stricter limits for high-risk countries
    // ...
  }

  return fetch(request);
}
```

**What's missing**:
- ❌ No distributed rate limiting
- ❌ No IP reputation checking
- ❌ No geographic rate limiting
- ❌ No bot detection
- ❌ No CAPTCHA for suspicious traffic

---

## 🟡 **IMPORTANT MISSING ITEMS**

### 11. **SSL Certificate Auto-Renewal** ⚠️ **PARTIALLY COVERED**

**Current**: Manual certbot
**What you need**: Automated renewal with monitoring

```bash
# /etc/cron.d/certbot-renew
0 0,12 * * * root certbot renew --quiet --deploy-hook "systemctl reload nginx && curl -X POST $SLACK_WEBHOOK -d '{\"text\":\"SSL certificates renewed\"}'"

# Certificate expiry monitoring
# monitor-certs.sh
#!/bin/bash
DOMAIN="api.carecircle.com"
EXPIRY=$(echo | openssl s_client -servername $DOMAIN -connect $DOMAIN:443 2>/dev/null | openssl x509 -noout -enddate | cut -d= -f2)
EXPIRY_EPOCH=$(date -d "$EXPIRY" +%s)
NOW_EPOCH=$(date +%s)
DAYS_LEFT=$(( ($EXPIRY_EPOCH - $NOW_EPOCH) / 86400 ))

if [ $DAYS_LEFT -lt 30 ]; then
  curl -X POST $SLACK_WEBHOOK -d "{\"text\":\"⚠️ SSL certificate expires in $DAYS_LEFT days!\"}"
fi
```

---

### 12. **Database Connection Pooling** ⚠️ **PARTIALLY COVERED**

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  host: process.env.DB_HOST,
  port: 5432,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,

  // Connection pooling config
  max: 20, // max connections
  min: 5, // min connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,

  // Health checks
  query_timeout: 5000,
  statement_timeout: 10000,

  // SSL in production
  ssl: process.env.NODE_ENV === 'production' ? {
    rejectUnauthorized: false
  } : false
});

// Monitor pool
pool.on('error', (err, client) => {
  console.error('Unexpected error on idle client', err);
  // Send to monitoring
});

pool.on('connect', () => {
  console.log('New database connection established');
});
```

---

### 13. **Graceful Shutdown** ⚠️ **MISSING**

```javascript
// server.js
const server = app.listen(PORT);

// Graceful shutdown handler
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, closing server gracefully');

  // Stop accepting new connections
  server.close(async () => {
    console.log('HTTP server closed');

    // Close database connections
    await pool.end();
    console.log('Database pool closed');

    // Close Redis connections
    await redis.quit();
    console.log('Redis connection closed');

    // Close other resources
    await closeQueues();

    console.log('Graceful shutdown complete');
    process.exit(0);
  });

  // Force shutdown after 30 seconds
  setTimeout(() => {
    console.error('Forced shutdown after timeout');
    process.exit(1);
  }, 30000);
});
```

---

### 14. **Security Scanning in CI** ⚠️ **MISSING**

```yaml
# .github/workflows/security.yml
name: Security Scan

on: [push, pull_request]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      # Dependency vulnerability scanning
      - name: npm audit
        run: npm audit --audit-level=high

      # SAST (Static Application Security Testing)
      - name: SonarCloud Scan
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

      # Secret scanning
      - name: Gitleaks
        uses: gitleaks/gitleaks-action@v2

      # Container scanning
      - name: Trivy scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'carecircle:latest'
          format: 'sarif'
          output: 'trivy-results.sarif'

      # License compliance
      - name: License check
        run: npx license-checker --failOn 'GPL;AGPL'
```

---

### 15. **Performance Testing** ⚠️ **MISSING**

**What you need**: Automated performance regression tests

```yaml
# .github/workflows/performance.yml
name: Performance Test

on:
  pull_request:
    branches: [main]

jobs:
  performance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run k6 performance test
        uses: grafana/k6-action@v0.3.0
        with:
          filename: tests/performance.js
          flags: --out json=results.json

      - name: Check performance regression
        run: |
          CURRENT_P95=$(jq '.metrics.http_req_duration.values."p(95)"' results.json)
          THRESHOLD=500
          if (( $(echo "$CURRENT_P95 > $THRESHOLD" | bc -l) )); then
            echo "Performance regression detected: p95 = ${CURRENT_P95}ms (threshold: ${THRESHOLD}ms)"
            exit 1
          fi
```

---

## 📊 **Priority Matrix**

| Priority | Item | Impact | Effort | Timeline |
|----------|------|--------|--------|----------|
| 🔴 P0 | CI/CD Pipeline | High | Medium | Week 1 |
| 🔴 P0 | Monitoring & Alerting | High | Medium | Week 1-2 |
| 🔴 P0 | Automated Backups | High | Low | Week 1 |
| 🔴 P0 | Health Checks | High | Low | Week 1 |
| 🟡 P1 | Docker/Kubernetes | High | High | Week 2-3 |
| 🟡 P1 | Secrets Management | Medium | Medium | Week 2 |
| 🟡 P1 | Auto-Scaling | Medium | Medium | Week 3 |
| 🟢 P2 | Infrastructure as Code | Medium | High | Week 4 |
| 🟢 P2 | Performance Testing | Low | Low | Week 4 |

---

## ✅ **Quick Start Implementation Order**

### **Week 1: Critical Foundation**
1. Set up CI/CD pipeline (GitHub Actions)
2. Add comprehensive health checks
3. Implement automated daily backups
4. Set up basic monitoring (Sentry + Uptime Robot)
5. Configure graceful shutdown

### **Week 2: Production Readiness**
1. Containerize with Docker
2. Set up secrets management (Vault or AWS Secrets Manager)
3. Implement distributed rate limiting
4. Add comprehensive logging (ELK or CloudWatch)
5. Set up alerting (PagerDuty or Opsgenie)

### **Week 3: Scalability**
1. Deploy to Kubernetes or Docker Swarm
2. Configure auto-scaling
3. Set up load testing in CI
4. Implement circuit breakers
5. Add APM (Application Performance Monitoring)

### **Week 4: Advanced Operations**
1. Convert infrastructure to Terraform
2. Set up disaster recovery drills
3. Implement blue-green deployment
4. Add canary releases
5. Set up cost monitoring

---

## 🎯 **Summary**

The security engineering document is **excellent for security** but **missing critical DevOps operational practices**:

### **What's Covered Well** ✅:
- Network security (VCN, NSGs, firewalls)
- Application security (auth, input validation, SQL injection)
- TLS/SSL configuration
- Basic rate limiting
- Security headers

### **What's MISSING** ❌:
- CI/CD pipelines
- Container orchestration
- Comprehensive monitoring & observability
- Automated backup & disaster recovery
- Auto-scaling
- Infrastructure as Code
- Secrets rotation
- Database migration strategy
- Performance testing
- Security scanning in CI

### **Recommendation**:
Start with **P0 items in Week 1**, then progressively add operational maturity. Don't launch to production without at least monitoring, backups, and health checks!

---

**Next Steps**:
1. Review this checklist with your DevOps team
2. Prioritize based on your risk tolerance
3. Implement P0 items before production launch
4. Schedule P1 and P2 items for post-launch

**Remember**: Security without proper DevOps operational practices is incomplete. You need both for a production-ready system!
