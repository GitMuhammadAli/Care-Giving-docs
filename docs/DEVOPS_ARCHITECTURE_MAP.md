# 🏗️ CareCircle DevOps Architecture Map

**Complete Reference to All Advanced DevOps Implementations**

---

## 📍 Quick Navigation

- [Background Workers](#-background-workers-implementation)
- [Kubernetes Orchestration](#-kubernetes-orchestration)
- [Nginx Load Balancing](#-nginx-load-balancing--reverse-proxy)
- [Docker Containerization](#-docker-containerization)
- [CI/CD Pipeline](#-cicd-pipeline)
- [Monitoring & Health Checks](#-monitoring--health-checks)
- [Database & Caching](#-database--caching-layer)
- [Message Queues](#-message-queues--job-processing)

---

## 🔧 Background Workers Implementation

### Location: `apps/workers/`

**Main Worker Orchestrator:**
- **File**: [apps/workers/src/index.ts](apps/workers/src/index.ts)
- **Features**:
  - Health check HTTP server (port 4001)
  - Graceful shutdown handling (SIGTERM, SIGINT, SIGBREAK)
  - Startup self-tests (Redis, Database connectivity)
  - Automatic recovery from failures
  - Windows-compatible signal handling

**Individual Workers:**

1. **Medication Reminder Worker**
   - **File**: [apps/workers/src/workers/medication-reminder.worker.ts](apps/workers/src/workers/medication-reminder.worker.ts)
   - **Features**:
     - Zod payload validation
     - Timezone-aware scheduling
     - Idempotent notification creation
     - Dead Letter Queue (DLQ) support
     - Error classification (transient vs permanent)
     - BullMQ concurrency: 10 concurrent jobs
   - **Queue**: `medication-reminders`

2. **Appointment Reminder Worker**
   - **File**: [apps/workers/src/workers/appointment-reminder.worker.ts](apps/workers/src/workers/appointment-reminder.worker.ts)
   - **Features**: Similar to medication worker, appointment-specific logic
   - **Queue**: `appointment-reminders`

3. **Shift Reminder Worker**
   - **File**: [apps/workers/src/workers/shift-reminder.worker.ts](apps/workers/src/workers/shift-reminder.worker.ts)
   - **Features**: Caregiver shift notifications
   - **Queue**: `shift-reminders`

4. **Notification Worker**
   - **File**: [apps/workers/src/workers/notification.worker.ts](apps/workers/src/workers/notification.worker.ts)
   - **Features**:
     - Push notifications (Web Push API)
     - Email sending (via MailService)
     - SMS sending (Twilio integration ready)
   - **Queue**: `notifications`

5. **Refill Alert Worker**
   - **File**: [apps/workers/src/workers/refill-alert.worker.ts](apps/workers/src/workers/refill-alert.worker.ts)
   - **Features**: Medication refill alerting
   - **Queue**: `refill-alerts`

6. **Dead Letter Queue Worker**
   - **File**: [apps/workers/src/workers/dead-letter.worker.ts](apps/workers/src/workers/dead-letter.worker.ts)
   - **Features**:
     - Failed job recovery
     - Slack alerting integration (ready)
     - Automatic retry logic
   - **Queue**: `dead-letter`

**Scheduler:**
- **File**: [apps/workers/src/scheduler.ts](apps/workers/src/scheduler.ts)
- **Purpose**: Scans database every 60 seconds and queues upcoming reminders

---

## ☸️ Kubernetes Orchestration

### Location: `k8s/`

**Namespace Configuration:**
- **File**: [k8s/namespace.yaml](k8s/namespace.yaml)
- Isolates CareCircle resources

**API Deployment:**
- **File**: [k8s/api-deployment.yaml](k8s/api-deployment.yaml)
- **Features**:
  ```yaml
  - Replicas: 2 (minimum)
  - Auto-scaling: 2-10 pods based on CPU (70%) and Memory (80%)
  - Health Checks: Liveness & Readiness probes
  - Resources:
      Requests: 100m CPU, 256Mi RAM
      Limits: 500m CPU, 512Mi RAM
  - Security: Non-root user (UID 1001), read-only filesystem
  - Anti-affinity: Pods spread across nodes
  ```

**Workers Deployment:**
- **File**: [k8s/workers-deployment.yaml](k8s/workers-deployment.yaml)
- **Features**: Similar to API, optimized for background jobs

**Web Deployment:**
- **File**: [k8s/web-deployment.yaml](k8s/web-deployment.yaml)
- **Features**: Next.js frontend with CDN-ready configuration

**Ingress Controller:**
- **File**: [k8s/ingress.yaml](k8s/ingress.yaml)
- **Features**:
  ```yaml
  - TLS/SSL: Let's Encrypt auto-provisioning (cert-manager)
  - Rate Limiting: 100 requests/minute
  - Domains: app.carecircle.com, api.carecircle.com
  - Load Balancing: NGINX Ingress Controller
  - Max Upload: 50MB
  ```

**Backup CronJob:**
- **File**: [k8s/backup-cronjob.yaml](k8s/backup-cronjob.yaml)
- **Schedule**: Daily at 2 AM
- **Retention**: 7 days

**ConfigMap & Secrets:**
- **File**: [k8s/config.yaml](k8s/config.yaml)
- Centralized environment variable management

---

## 🌐 Nginx Load Balancing & Reverse Proxy

### Location: `nginx/`

**Main Configuration:**
- **File**: [nginx/nginx.conf](nginx/nginx.conf)
- **Features**:
  ```nginx
  Performance:
  - Worker processes: auto-scaled
  - Worker connections: 4096
  - HTTP/2 enabled
  - Gzip compression (level 6)
  - Brotli compression ready
  - Sendfile, TCP optimizations

  Security:
  - TLS 1.2/1.3 only
  - HSTS enabled (1 year)
  - Security headers (X-Frame-Options, CSP, etc.)
  - Rate limiting zones:
      * API: 100 req/min
      * Login: 5 req/min

  Load Balancing:
  - Algorithm: Least Connections
  - Health checks: max_fails=3, timeout=30s
  - Keepalive: 32 connections, 100 requests
  - Upstream servers: api:3001, web:3000
  ```

**Virtual Host Configurations:**
- API routing
- Web frontend routing
- WebSocket proxy support
- Static asset caching (1 year)
- Custom error pages

---

## 🐳 Docker Containerization

### API Dockerfile
- **File**: [apps/api/Dockerfile](apps/api/Dockerfile)
- **Multi-stage build**:
  ```dockerfile
  Stage 1 (Builder):
  - Base: node:20-alpine
  - Install pnpm 9.0.0
  - Install dependencies with frozen lockfile
  - Generate Prisma client
  - Build TypeScript to dist/

  Stage 2 (Runner):
  - Base: node:20-alpine (minimal)
  - Non-root user (nestjs:1001)
  - Copy only production artifacts
  - Health check: wget localhost:3001/health every 30s
  - Expose port: 3001
  - CMD: node dist/main.js
  ```

### Workers Dockerfile
- **File**: [apps/workers/Dockerfile](apps/workers/Dockerfile)
- Similar multi-stage build optimized for workers

### Web Dockerfile
- **File**: [apps/web/Dockerfile](apps/web/Dockerfile)
- Next.js standalone build for minimal image size

### Docker Compose Configurations

**Development:**
- **File**: [docker-compose.yml](docker-compose.yml)
- **Services**: PostgreSQL, Redis, RabbitMQ
- **Volumes**: Persistent data storage
- **Networks**: Internal bridge network

**Production:**
- **File**: [docker-compose.prod.yml](docker-compose.prod.yml)
- **Additional**: Nginx reverse proxy, Let's Encrypt companion

**Backup:**
- **File**: [docker-compose.backup.yml](docker-compose.backup.yml)
- Automated backup container with cron

---

## 🔄 CI/CD Pipeline

### Location: `.github/workflows/`

**Main CI/CD Pipeline:**
- **File**: [.github/workflows/ci.yml](.github/workflows/ci.yml)
- **Triggers**: Push to main/develop, Pull Requests
- **Jobs**:
  ```yaml
  1. Test:
     - Runs on: ubuntu-latest
     - Services: PostgreSQL 16, Redis 7
     - Steps:
       * Install pnpm 9.0.0
       * Install dependencies (frozen lockfile)
       * Run test suite
       * Upload coverage

  2. Build:
     - Runs after: Test passes
     - Steps:
       * Build all apps
       * Type check (0 errors)
       * Lint check

  3. Security Scan:
     - npm audit
     - Snyk vulnerability scanning
     - OWASP dependency check

  4. Docker Build:
     - Multi-arch (amd64, arm64)
     - Push to GitHub Container Registry
     - Tag: latest, git SHA, semantic version
  ```

**Staging Deployment:**
- **File**: [.github/workflows/deploy-staging.yml](.github/workflows/deploy-staging.yml)
- Auto-deploys to staging environment on develop branch

**Production Deployment:**
- **File**: [.github/workflows/deploy-production.yml](.github/workflows/deploy-production.yml)
- Manual approval required
- Blue-green deployment strategy
- Automated rollback on health check failure

---

## 📊 Monitoring & Health Checks

### Health Check System

**API Health Controller:**
- **File**: [apps/api/src/health/health.controller.ts](apps/api/src/health/health.controller.ts)
- **Endpoints**:
  ```typescript
  GET /health
  - Database connectivity (Prisma)
  - Redis connectivity
  - Memory usage (Heap < 300MB, RSS < 500MB)
  - Disk usage (< 90% full)

  GET /health/ready
  - Kubernetes readiness probe
  - Database + Redis only

  GET /health/live
  - Kubernetes liveness probe
  - Simple timestamp response
  ```

**Custom Health Indicators:**
- **Prisma**: [apps/api/src/health/prisma.health.ts](apps/api/src/health/prisma.health.ts)
- **Redis**: [apps/api/src/health/redis.health.ts](apps/api/src/health/redis.health.ts)

**Metrics Controller:**
- **File**: [apps/api/src/metrics/metrics.controller.ts](apps/api/src/metrics/metrics.controller.ts)
- **Endpoint**: `GET /metrics`
- **Format**: Prometheus-compatible
- **Metrics**:
  ```
  - http_requests_total (by route, method, status)
  - http_request_duration_seconds (histogram)
  - database_query_duration_seconds
  - cache_hits_total / cache_misses_total
  - background_jobs_processed_total
  - background_jobs_failed_total
  - active_websocket_connections
  ```

**Metrics Service:**
- **File**: [apps/api/src/metrics/metrics.service.ts](apps/api/src/metrics/metrics.service.ts)
- Uses `prom-client` library
- Custom business metrics

**Worker Health Checks:**
- **File**: [apps/workers/src/index.ts](apps/workers/src/index.ts)
- **Endpoint**: `GET http://localhost:4001/health` (or `/healthz`)
- **Checks**:
  - Redis connection
  - Database connection
  - Individual worker status
  - Uptime

---

## 🗄️ Database & Caching Layer

### Prisma ORM
- **Schema**: [packages/database/prisma/schema.prisma](packages/database/prisma/schema.prisma)
- **Features**:
  - 19 database models
  - Foreign key constraints
  - Indexes on frequently queried fields
  - Soft deletes where applicable
  - Audit trail (createdAt, updatedAt)

### Database Migrations
- **Directory**: [packages/database/prisma/migrations/](packages/database/prisma/migrations/)
- **Current**: `20260116191155_init`
- **Management**: Prisma Migrate

### Connection Pooling
- **Configuration**: Environment-based
- **Production**:
  - Min connections: 5
  - Max connections: 20
  - Connection timeout: 10s
  - Idle timeout: 300s

### Redis Caching
- **Implementation**: [apps/api/src/cache/](apps/api/src/cache/) (if exists)
- **Strategy**: Cache-aside pattern
- **TTL**: Configurable per cache key
- **Invalidation**: Event-driven

---

## 📨 Message Queues & Job Processing

### BullMQ Configuration

**Queue Definitions:**
- **File**: [apps/workers/src/queues/index.ts](apps/workers/src/queues/index.ts) (or similar)
- **Queues**:
  ```typescript
  - medication-reminders
  - appointment-reminders
  - shift-reminders
  - notifications
  - refill-alerts
  - dead-letter
  ```

**Queue Features:**
- **Redis-backed**: Persistent job storage
- **Retry Logic**:
  - Exponential backoff
  - Max attempts: 3-5 (configurable per queue)
  - Backoff: 1000ms, 5000ms, 15000ms
- **Priority**: High/Normal/Low
- **Rate Limiting**: Configurable per queue
- **Job TTL**: Auto-cleanup after completion

**Dead Letter Queue (DLQ):**
- **Purpose**: Failed jobs that exceeded retry limit
- **Features**:
  - Manual inspection
  - Replay capability
  - Slack alerting (configured)
  - Permanent failure logging

---

## 🔐 Security Implementation

### Authentication Layer
- **JWT Strategy**: [apps/api/src/auth/strategies/jwt.strategy.ts](apps/api/src/auth/strategies/jwt.strategy.ts)
- **Guards**: [apps/api/src/system/guards/](apps/api/src/system/guards/)
- **Decorators**: [apps/api/src/system/decorator/](apps/api/src/system/decorator/)

### Rate Limiting
- **Nginx Level**: Configured in [nginx/nginx.conf](nginx/nginx.conf)
- **Application Level**: NestJS ThrottlerGuard
- **Zones**:
  - General API: 100 req/min
  - Authentication: 5 req/min
  - Password reset: 3 req/hour

### CORS Configuration
- **File**: [apps/api/src/main.ts](apps/api/src/main.ts)
- Configurable origins
- Credentials support
- Preflight caching

### Helmet Security Headers
- CSP (Content Security Policy)
- HSTS (HTTP Strict Transport Security)
- X-Frame-Options
- X-Content-Type-Options
- Referrer-Policy

---

## 📈 Performance Optimizations

### Load Testing
- **K6 Script**: [scripts/load-test.k6.js](scripts/load-test.k6.js)
- **Artillery**: [apps/api/test/load-testing/artillery-load-test.yml](apps/api/test/load-testing/artillery-load-test.yml)
- **Scenarios**: Authentication, CRUD operations, concurrent users

### Database Optimizations
- **Indexes**: On all foreign keys and frequently queried fields
- **Query Optimization**: Prisma select/include optimization
- **Connection Pooling**: PgBouncer-ready
- **Read Replicas**: Configuration ready

### Caching Strategy
- **Redis**: Session storage, rate limiting
- **HTTP Caching**: ETag, Last-Modified headers
- **CDN**: CloudFlare-ready configuration

---

## 🚀 Deployment Options

### 1. Railway / Render
- **Guides**: [docs/deployment/FREE_DEPLOYMENT_GUIDE.md](docs/deployment/FREE_DEPLOYMENT_GUIDE.md)
- One-click deployment
- Auto-scaling included
- Cost: $5-20/month

### 2. Oracle Cloud (Free Tier)
- **Guide**: [docs/deployment/ORACLE_CLOUD_FREE_TIER_GUIDE.md](docs/deployment/ORACLE_CLOUD_FREE_TIER_GUIDE.md)
- Forever FREE
- 4 ARM CPUs + 24GB RAM
- Kubernetes ready

### 3. AWS Enterprise
- **Guide**: [docs/deployment/PRODUCTION_DEPLOYMENT_GUIDE.md](docs/deployment/PRODUCTION_DEPLOYMENT_GUIDE.md)
- ECS/EKS
- Multi-region HA
- Auto-scaling groups

### 4. Kubernetes (Any Cloud)
- Use `k8s/` manifests
- Apply with: `kubectl apply -f k8s/`
- Supports GKE, EKS, AKS, DigitalOcean

---

## 📁 Complete File Structure

```
CareCircle/
│
├── apps/
│   ├── api/                          # NestJS Backend API
│   │   ├── src/
│   │   │   ├── health/              # ✅ Health check endpoints
│   │   │   ├── metrics/             # ✅ Prometheus metrics
│   │   │   ├── auth/                # Authentication & JWT
│   │   │   └── ...                  # Business logic modules
│   │   └── Dockerfile               # ✅ Multi-stage Docker build
│   │
│   ├── web/                          # Next.js Frontend
│   │   └── Dockerfile               # ✅ Next.js optimized build
│   │
│   └── workers/                      # ✅ Background job processors
│       ├── src/
│       │   ├── index.ts             # ✅ Worker orchestrator + health server
│       │   ├── scheduler.ts         # ✅ Job scheduler
│       │   ├── queues/              # BullMQ queue definitions
│       │   └── workers/             # ✅ 6 individual workers
│       └── Dockerfile
│
├── k8s/                              # ✅ Kubernetes manifests
│   ├── namespace.yaml
│   ├── api-deployment.yaml          # ✅ API with HPA
│   ├── workers-deployment.yaml      # ✅ Workers deployment
│   ├── web-deployment.yaml          # Frontend deployment
│   ├── ingress.yaml                 # ✅ Nginx Ingress + TLS
│   ├── config.yaml                  # ConfigMaps
│   └── backup-cronjob.yaml          # ✅ Automated backups
│
├── nginx/
│   └── nginx.conf                   # ✅ Load balancer + reverse proxy
│
├── .github/workflows/               # ✅ CI/CD Pipelines
│   ├── ci.yml                       # ✅ Test, Build, Security
│   ├── deploy-staging.yml           # Staging deployment
│   └── deploy-production.yml        # Production deployment
│
├── docker-compose.yml               # ✅ Local development
├── docker-compose.prod.yml          # ✅ Production setup
└── scripts/
    ├── load-test.k6.js              # ✅ Performance testing
    └── monitor-neon-backups.sh      # Backup monitoring
```

---

## 🎯 Key Takeaways

### What Makes This Advanced?

1. **Production-Grade Workers**:
   - 6 specialized background workers
   - Idempotent job processing
   - Error classification and DLQ
   - Health monitoring
   - Graceful shutdown

2. **Kubernetes-Native**:
   - Auto-scaling (HPA)
   - Health probes (liveness/readiness)
   - Resource limits and requests
   - Pod anti-affinity
   - ConfigMaps and Secrets

3. **Nginx Expertise**:
   - Load balancing with least connections
   - Rate limiting zones
   - TLS/SSL termination
   - HTTP/2 support
   - Compression (Gzip/Brotli)

4. **Docker Optimization**:
   - Multi-stage builds (smaller images)
   - Non-root user security
   - Health checks in Dockerfile
   - Layer caching optimization

5. **CI/CD Automation**:
   - Automated testing
   - Security scanning
   - Multi-arch Docker builds
   - Automated deployment
   - Rollback capabilities

6. **Observability**:
   - Prometheus metrics
   - Health check endpoints
   - Structured logging
   - Performance monitoring

7. **High Availability**:
   - Multiple replicas
   - Auto-scaling based on load
   - Graceful degradation
   - Zero-downtime deployments

---

## 📚 Related Documentation

- [Production Infrastructure Complete](docs/PRODUCTION_INFRASTRUCTURE_COMPLETE.md)
- [Final Status](docs/project-status/FINAL_STATUS.md)
- [Deployment Guides](docs/deployment/)
- [Operations Runbook](docs/operations/)
- [Backup Procedures](docs/operations/BACKUP_PROCEDURES.md)

---

**Last Updated**: January 17, 2026
**Architecture Version**: 5.0.0
