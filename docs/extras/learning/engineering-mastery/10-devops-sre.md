# Chapter 10: DevOps & SRE

> "SRE is what happens when you ask a software engineer to design an operations team."

---

## 🔄 DevOps Culture

```
Traditional:
┌──────────────┐                    ┌──────────────┐
│ Development  │ ──── Wall ────── │  Operations  │
│ "Ship fast"  │  "Your problem"  │ "Keep stable"│
└──────────────┘                    └──────────────┘

DevOps:
┌─────────────────────────────────────────────────┐
│              Shared Responsibility               │
│                                                 │
│  Build → Test → Deploy → Monitor → Improve      │
│         ↑                              │        │
│         └──────── Feedback Loop ───────┘        │
└─────────────────────────────────────────────────┘
```

---

## 🚀 CI/CD Pipeline

### Continuous Integration

```
┌─────────────────────────────────────────────────────────────────┐
│                    CI Pipeline                                   │
│                                                                 │
│  git push → Build → Lint → Test → Security Scan → Artifact     │
│     │         │       │      │         │            │          │
│     │         │       │      │         │            │          │
│     ▼         ▼       ▼      ▼         ▼            ▼          │
│  Trigger   Compile   ESLint  Unit    Snyk/Trivy   Docker       │
│            TypeScript        Jest                  Image        │
│                              E2E                                │
└─────────────────────────────────────────────────────────────────┘
```

### Continuous Deployment

```
┌─────────────────────────────────────────────────────────────────┐
│                    CD Pipeline                                   │
│                                                                 │
│                         ┌─────────────┐                         │
│                         │   Staging   │                         │
│                         │   Deploy    │                         │
│  Artifact ──────────────┤     ↓       │                         │
│                         │  Smoke Test │                         │
│                         │     ↓       │                         │
│                         │  Approval   │                         │
│                         └──────┬──────┘                         │
│                                │                                │
│                         ┌──────┴──────┐                         │
│                         │ Production  │                         │
│                         │   Deploy    │                         │
│                         │     ↓       │                         │
│                         │  Canary 5%  │                         │
│                         │     ↓       │                         │
│                         │  Full 100%  │                         │
│                         └─────────────┘                         │
└─────────────────────────────────────────────────────────────────┘
```

### GitHub Actions Example

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run lint
      - run: npm run test
      - run: npm run build

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  deploy-staging:
    needs: [test, security]
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Staging
        run: |
          # Deploy to staging environment
          
  deploy-production:
    needs: [test, security]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production  # Requires approval
    steps:
      - name: Deploy to Production
        run: |
          # Deploy to production
```

---

## 🐳 Containers & Kubernetes

### Docker Best Practices

```dockerfile
# Multi-stage build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine AS runner
WORKDIR /app

# Don't run as root
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Copy only what's needed
COPY --from=builder /app/node_modules ./node_modules
COPY --chown=nextjs:nodejs . .

USER nextjs

EXPOSE 3000
CMD ["node", "server.js"]
```

### Kubernetes Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                            │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    Control Plane                          │  │
│  │  ┌──────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐  │  │
│  │  │API Server│ │ Scheduler │ │Controller │ │   etcd    │  │  │
│  │  └──────────┘ └───────────┘ │  Manager  │ │           │  │  │
│  │                             └───────────┘ └───────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                               │                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                      Worker Nodes                         │  │
│  │                                                           │  │
│  │  Node 1              Node 2              Node 3           │  │
│  │  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐  │  │
│  │  │ ┌────┐ ┌────┐│   │ ┌────┐ ┌────┐│   │ ┌────┐ ┌────┐│  │  │
│  │  │ │Pod1│ │Pod2││   │ │Pod3│ │Pod4││   │ │Pod5│ │Pod6││  │  │
│  │  │ └────┘ └────┘│   │ └────┘ └────┘│   │ └────┘ └────┘│  │  │
│  │  │  kubelet     │   │  kubelet     │   │  kubelet     │  │  │
│  │  └──────────────┘   └──────────────┘   └──────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Kubernetes Resources

```yaml
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: myapp/api:v1.2.3
          ports:
            - containerPort: 3000
          resources:
            requests:
              memory: "256Mi"
              cpu: "100m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 30
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
---
# Service
apiVersion: v1
kind: Service
metadata:
  name: api
spec:
  selector:
    app: api
  ports:
    - port: 80
      targetPort: 3000
---
# HorizontalPodAutoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## 🏗️ Infrastructure as Code

### Terraform Example

```hcl
# AWS Infrastructure
provider "aws" {
  region = "us-east-1"
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  
  tags = {
    Name = "production"
  }
}

# RDS Database
resource "aws_db_instance" "postgres" {
  identifier           = "production-db"
  allocated_storage    = 100
  engine              = "postgres"
  engine_version      = "15.4"
  instance_class      = "db.r6g.large"
  db_name             = "carecircle"
  username            = var.db_username
  password            = var.db_password
  
  multi_az            = true
  storage_encrypted   = true
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  
  vpc_security_group_ids = [aws_security_group.db.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
}

# EKS Cluster
resource "aws_eks_cluster" "main" {
  name     = "production"
  role_arn = aws_iam_role.eks.arn
  version  = "1.28"
  
  vpc_config {
    subnet_ids = aws_subnet.private[*].id
  }
}

# ElastiCache Redis
resource "aws_elasticache_cluster" "redis" {
  cluster_id           = "production-cache"
  engine               = "redis"
  node_type            = "cache.r6g.large"
  num_cache_nodes      = 1
  parameter_group_name = "default.redis7"
  port                 = 6379
  
  subnet_group_name    = aws_elasticache_subnet_group.main.name
  security_group_ids   = [aws_security_group.redis.id]
}
```

---

## 📊 SRE Practices

### SLOs and Error Budgets

```
SLI (Service Level Indicator):
  Actual measurement
  Example: Latency p99 = 250ms

SLO (Service Level Objective):
  Target for the SLI
  Example: p99 latency < 300ms

SLA (Service Level Agreement):
  Promise to customers (with consequences)
  Example: 99.9% uptime or credit

Error Budget:
┌──────────────────────────────────────────────────────────────┐
│ SLO: 99.9% availability                                      │
│ Error budget: 0.1% = 43.2 minutes/month                      │
│                                                              │
│ Month Progress:                                              │
│ ████████████████░░░░░░░░░░░░░░░░ 50% through month           │
│                                                              │
│ Budget Used:                                                 │
│ ████████░░░░░░░░░░░░░░░░░░░░░░░░ 20 minutes (46% of budget) │
│                                                              │
│ Status: GREEN - Can take risks, deploy features              │
└──────────────────────────────────────────────────────────────┘

Budget actions:
- >50% remaining: Ship features, take risks
- 25-50% remaining: Normal operations
- <25% remaining: Focus on reliability
- 0% remaining: Freeze deployments
```

### Incident Management

```
Incident Lifecycle:
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Detection → Triage → Mitigation → Resolution → Postmortem │
│                                                             │
└─────────────────────────────────────────────────────────────┘

Severity Levels:
┌─────────┬─────────────────────────────────────────┬─────────┐
│ Level   │ Description                             │ Response│
├─────────┼─────────────────────────────────────────┼─────────┤
│ SEV1    │ Complete outage, data loss              │ <5 min  │
│ SEV2    │ Major feature broken, significant users │ <15 min │
│ SEV3    │ Minor feature broken, workaround exists │ <1 hour │
│ SEV4    │ Cosmetic issue, low impact              │ Next day│
└─────────┴─────────────────────────────────────────┴─────────┘

On-Call Rotation:
- Primary + Secondary on-call
- Escalation path defined
- Runbooks for common issues
- Post-incident review (blameless)
```

### Postmortem Template

```markdown
# Incident Postmortem: [Title]

**Date:** 2024-01-15
**Duration:** 2 hours 15 minutes
**Severity:** SEV2
**Author:** Jane Doe

## Summary
Brief description of what happened.

## Impact
- 15% of users affected
- $50,000 estimated revenue impact
- 2.5 hours of partial outage

## Timeline
- 14:00 - Deployment started
- 14:15 - First alerts fired
- 14:20 - On-call paged
- 14:35 - Root cause identified
- 15:00 - Mitigation deployed
- 16:15 - Full recovery

## Root Cause
Database connection pool exhausted due to missing
connection timeout configuration.

## What Went Well
- Alerts fired quickly
- Team responded within SLO
- Good communication

## What Went Wrong
- Deployment wasn't canary tested
- Monitoring didn't catch connection leak
- Runbook was outdated

## Action Items
| Action | Owner | Due Date |
|--------|-------|----------|
| Add connection pool monitoring | Alice | 2024-01-20 |
| Update deployment checklist | Bob | 2024-01-18 |
| Implement canary deployments | Carol | 2024-02-01 |

## Lessons Learned
Connection pool configuration should be explicitly
set and monitored for all database connections.
```

---

## 🔄 Deployment Strategies

```
Blue-Green:
┌──────────┐                    ┌──────────┐
│  Blue    │  ← Current         │  Green   │  ← New
│  (v1.0)  │     Traffic        │  (v1.1)  │
└──────────┘                    └──────────┘
       ↕ Switch instantly

Canary:
┌──────────────────────────────────────────┐
│ 95% ────────────────────► Old (v1.0)     │
│  5% ────────────────────► New (v1.1)     │
└──────────────────────────────────────────┘
       Gradually increase new version

Rolling:
┌────────────────────────────────────────────┐
│ Instance 1: v1.0 → v1.1 ✓                  │
│ Instance 2: v1.0 → v1.1 ✓                  │
│ Instance 3: v1.0 → v1.1 (in progress)      │
│ Instance 4: v1.0 (waiting)                 │
└────────────────────────────────────────────┘

Feature Flags:
┌────────────────────────────────────────────┐
│ if (featureFlags.newCheckout) {            │
│   return <NewCheckout />                   │
│ } else {                                   │
│   return <OldCheckout />                   │
│ }                                          │
└────────────────────────────────────────────┘
Deploy code, enable for % of users
```

---

## 📖 Further Reading

- "Site Reliability Engineering" by Google
- "The Phoenix Project"
- "Accelerate" by Forsgren
- "The DevOps Handbook"

---

**Next:** [Chapter 11: Performance Engineering →](./11-performance.md)


