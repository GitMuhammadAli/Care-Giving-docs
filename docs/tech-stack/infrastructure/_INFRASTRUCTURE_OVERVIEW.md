# Infrastructure Overview

> Understanding how CareCircle runs in production.

---

## The Mental Model

Think of infrastructure like **building a restaurant**:

- **Docker** = Prefab kitchen units (consistent, portable, isolated)
- **Docker Compose** = The building layout (how units connect)
- **CI/CD** = Automated opening procedure (same steps, every day)
- **Environment Variables** = Staff instructions (different for each shift)
- **Nginx** = The host/maître d' (routes guests to right tables)

---

## Environment Architecture

### The Three Environments

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ENVIRONMENT PROGRESSION                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  DEVELOPMENT (Local)                                                         │
│  ───────────────────                                                         │
│  Purpose: Build and test features                                            │
│  Database: Local Docker PostgreSQL                                           │
│  Services: Local Docker (Redis, RabbitMQ)                                    │
│  Debugging: Full access, hot reload                                          │
│                                                                              │
│         │                                                                    │
│         │  git push                                                          │
│         ▼                                                                    │
│                                                                              │
│  STAGING (Optional)                                                          │
│  ─────────────────                                                           │
│  Purpose: Pre-production testing                                             │
│  Database: Staging Neon instance                                             │
│  Services: Staging Upstash/CloudAMQP                                         │
│  Testing: Full feature testing before prod                                   │
│                                                                              │
│         │                                                                    │
│         │  Approved PR / manual trigger                                      │
│         ▼                                                                    │
│                                                                              │
│  PRODUCTION (Cloud)                                                          │
│  ─────────────────                                                           │
│  Purpose: Serve real users                                                   │
│  Database: Production Neon (connection pooling)                              │
│  Services: Production Upstash/CloudAMQP                                      │
│  Monitoring: Full observability                                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Why Separate Environments?

| Problem | Local-Only Development | Multi-Environment |
|---------|----------------------|-------------------|
| "Works on my machine" | Frequent | Rare (Docker) |
| Production data corruption | Testing in prod | Isolated data |
| Deployment fear | High anxiety | Confidence |
| Configuration bugs | Discovered in prod | Caught in staging |

---

## Docker: Containerization Concepts

### The Problem Docker Solves

```
WITHOUT DOCKER:
───────────────

Developer A: "I have Node 18, PostgreSQL 14, Redis 6"
Developer B: "I have Node 16, PostgreSQL 15, Redis 7"
Developer C: "I'm on Windows, things work differently"
Server:      "I have Node 20, PostgreSQL 16, Redis 5"

Result: "It works on my machine!" 🤷


WITH DOCKER:
────────────

Everyone runs the SAME container:
  • Node 20.x (specified in Dockerfile)
  • PostgreSQL 15 (specified in docker-compose.yml)
  • Redis 7 (specified in docker-compose.yml)

Result: Same environment everywhere ✓
```

### Container Mental Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CONTAINER ANATOMY                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  IMAGE (Blueprint)                   CONTAINER (Running Instance)            │
│  ─────────────────                   ──────────────────────────             │
│                                                                              │
│  Like a class definition             Like an object instance                 │
│  Immutable                           Can be started/stopped                  │
│  Shared/versioned                    Isolated from other containers          │
│                                                                              │
│  Dockerfile → Build → Image → Run → Container                               │
│                                                                              │
│  Example:                                                                    │
│  ┌─────────────────────┐             ┌─────────────────────┐                │
│  │   node:20-alpine    │             │   api_container_1   │                │
│  │   (base image)      │ ──build───► │   (instance 1)      │                │
│  │   + your code       │             │                     │                │
│  │   + dependencies    │             │   Same image can    │                │
│  │                     │ ──build───► │   create multiple   │                │
│  │                     │             │   containers        │                │
│  └─────────────────────┘             └─────────────────────┘                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### CareCircle's Docker Setup

```yaml
# docker-compose.yml (conceptual)

services:
  # PostgreSQL - Primary database
  postgres:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data  # Data survives restart
    environment:
      POSTGRES_DB: carecircle
      
  # Redis - Cache and job queues
  redis:
    image: redis:7-alpine
    
  # RabbitMQ - Message broker
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "15672:15672"  # Management UI
      
  # API - Our NestJS backend
  api:
    build: ./apps/api
    depends_on:
      - postgres
      - redis
    environment:
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/carecircle
      
  # Workers - Background job processors
  workers:
    build: ./apps/workers
    depends_on:
      - redis
```

---

## Environment Configuration Philosophy

### The Twelve-Factor App Approach

```
PRINCIPLE: Configuration in environment, not code

❌ WRONG: Hardcoded config
const dbUrl = "postgresql://localhost:5432/carecircle";

✅ RIGHT: Environment variable
const dbUrl = process.env.DATABASE_URL;


WHY THIS MATTERS:
─────────────────

1. Same code, different environments
   Local:  DATABASE_URL=localhost
   Prod:   DATABASE_URL=production-server

2. Secrets stay secret
   JWT_SECRET never in code repository

3. Easy configuration changes
   Change env var, restart app (no rebuild)
```

### CareCircle's Config Structure

```
env/
├── base.env       # Common to all environments
├── local.env      # Local development overrides
└── cloud.env      # Production/cloud overrides

Scripts:
├── use-local.ps1  # Merges base + local
└── use-cloud.ps1  # Merges base + cloud

The merge creates .env:
  base.env + local.env → .env (for local development)
  base.env + cloud.env → .env (for production)
```

### Configuration Validation

```typescript
// Don't trust environment variables exist!

// In @carecircle/config package
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  JWT_EXPIRES_IN: z.string(),
  // ... etc
});

// Validate at startup
export const config = envSchema.parse(process.env);
// App won't start if config is invalid!
```

---

## CI/CD: Continuous Integration & Deployment

### The Pipeline Philosophy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CI/CD PHILOSOPHY                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  CONTINUOUS INTEGRATION                                                      │
│  ──────────────────────                                                      │
│  "Integrate early, integrate often"                                          │
│                                                                              │
│  Every commit triggers:                                                      │
│  1. Build the application                                                    │
│  2. Run tests (unit, integration)                                           │
│  3. Check code quality (lint, types)                                        │
│  4. Security scanning                                                        │
│                                                                              │
│  Benefit: Catch problems in minutes, not days                               │
│                                                                              │
│  CONTINUOUS DEPLOYMENT                                                       │
│  ────────────────────                                                        │
│  "If it passes, ship it"                                                     │
│                                                                              │
│  Successful CI triggers:                                                     │
│  1. Build production artifacts                                              │
│  2. Deploy to staging (automatic)                                           │
│  3. Deploy to production (manual gate or auto)                              │
│                                                                              │
│  Benefit: Consistent, repeatable deployments                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### CareCircle's Pipeline

```
Developer pushes code
         │
         ▼
    ┌─────────┐
    │  LINT   │──Failed──► Stop, notify developer
    └────┬────┘
         │ Pass
         ▼
    ┌─────────┐
    │  BUILD  │──Failed──► Stop, notify developer
    └────┬────┘
         │ Pass
         ▼
    ┌─────────┐
    │  TEST   │──Failed──► Stop, notify developer
    └────┬────┘
         │ Pass
         ▼
    ┌─────────┐
    │ STAGING │
    │ DEPLOY  │
    └────┬────┘
         │
         ▼ (if main branch)
    ┌──────────┐
    │PRODUCTION│
    │  DEPLOY  │
    └──────────┘
```

---

## Deployment Strategies

### Blue-Green Deployment (Concept)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       BLUE-GREEN DEPLOYMENT                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  BEFORE DEPLOYMENT:                                                          │
│  ──────────────────                                                          │
│                                                                              │
│  Users ─────► Load Balancer ─────► BLUE (v1.0) ← Current production         │
│                     │                                                        │
│                     └─────► GREEN (v1.0) ← Idle                             │
│                                                                              │
│                                                                              │
│  DURING DEPLOYMENT:                                                          │
│  ─────────────────                                                           │
│                                                                              │
│  Users ─────► Load Balancer ─────► BLUE (v1.0) ← Still serving              │
│                     │                                                        │
│                     └─────► GREEN (v1.1) ← Being deployed, tested           │
│                                                                              │
│                                                                              │
│  AFTER SWITCH:                                                               │
│  ─────────────                                                               │
│                                                                              │
│  Users ─────► Load Balancer ─────► BLUE (v1.0) ← Idle (rollback ready)      │
│                     │                                                        │
│                     └─────► GREEN (v1.1) ← Now serving production           │
│                                                                              │
│                                                                              │
│  BENEFIT: Zero-downtime deployments, instant rollback                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Rolling Deployment (Vercel/Serverless)

```
For serverless platforms like Vercel:

1. New deployment is created
2. Traffic gradually shifts to new version
3. Old version stays available for in-flight requests
4. Old version spun down after traffic drains

Simpler than blue-green, handled by platform
```

---

## Monitoring & Observability

### The Three Pillars

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       OBSERVABILITY PILLARS                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  LOGS                                                                        │
│  ────                                                                        │
│  What happened?                                                              │
│  • Request/response details                                                  │
│  • Errors with stack traces                                                  │
│  • Audit trail of actions                                                    │
│  Tool: Pino → aggregator (DataDog, LogTail)                                 │
│                                                                              │
│  METRICS                                                                     │
│  ───────                                                                     │
│  How is it performing?                                                       │
│  • Request latency (p50, p95, p99)                                          │
│  • Error rate                                                                │
│  • Queue depth                                                               │
│  Tool: Prometheus metrics                                                    │
│                                                                              │
│  TRACES                                                                      │
│  ──────                                                                      │
│  How did the request flow?                                                   │
│  • Request → API → DB → Cache → Response                                    │
│  • Where was time spent?                                                     │
│  Tool: Sentry or OpenTelemetry                                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### What to Monitor

```
HEALTH CHECKS:
──────────────
• /health - Overall app health
• /ready  - Ready to accept traffic
• /live   - Process is alive

CRITICAL METRICS:
─────────────────
• API response times
• Error rates (4xx, 5xx)
• Database connection pool
• Redis memory usage
• Queue depth (jobs waiting)
• Worker processing rate

ALERTS:
───────
• Error rate > 5%
• Response time p95 > 2s
• Queue depth > 1000
• Memory usage > 80%
```

---

## Scaling Concepts

### Horizontal vs Vertical Scaling

```
VERTICAL SCALING (Scale Up):
───────────────────────────
• Bigger server (more CPU, RAM)
• Simple: Just upgrade
• Limited: Has a ceiling
• Downtime: Usually required

Example: Upgrade from 2GB RAM to 8GB RAM


HORIZONTAL SCALING (Scale Out):
──────────────────────────────
• More servers (same size each)
• Complex: Need load balancing
• Unlimited: Add more servers
• No downtime: Add behind load balancer

Example: Run 4 API instances instead of 1
```

### CareCircle's Scaling Points

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         SCALING BOTTLENECKS                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  COMPONENT           │ SCALING APPROACH        │ BOTTLENECK SIGNS           │
│  ────────────────────┼─────────────────────────┼────────────────────────    │
│  API (NestJS)        │ Horizontal (replicas)   │ High CPU, slow responses   │
│  Workers (BullMQ)    │ Horizontal (more workers)│ Queue depth growing       │
│  PostgreSQL          │ Vertical + Read replicas│ Query latency, connections │
│  Redis               │ Vertical + Cluster      │ Memory, operations/sec     │
│                                                                              │
│  SCALING ORDER (typical):                                                    │
│  1. Add caching (Redis) - Often fixes 80% of issues                         │
│  2. Scale API horizontally - If CPU-bound                                   │
│  3. Add database read replicas - If read-heavy                              │
│  4. Database sharding - Rarely needed at our scale                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Security in Infrastructure

### Defense Layers

```
NETWORK LEVEL:
──────────────
• HTTPS everywhere (TLS 1.3)
• Firewall rules (only necessary ports)
• DDoS protection (Cloudflare, Vercel)

APPLICATION LEVEL:
──────────────────
• Rate limiting per IP/user
• Input validation
• CORS configuration

DATA LEVEL:
───────────
• Encryption at rest (database)
• Encryption in transit (TLS)
• Secrets in env vars, not code
```

---

## Quick Reference

### Docker Commands

```bash
# Start all services
docker compose up -d

# View running containers
docker compose ps

# View logs
docker compose logs -f api

# Rebuild after changes
docker compose up --build

# Stop all services
docker compose down

# Stop and remove volumes (DANGER: data loss)
docker compose down -v
```

### Common Deployment Issues

| Problem | Likely Cause | Solution |
|---------|--------------|----------|
| "Connection refused" | Service not ready | Check health endpoints, add retry |
| "502 Bad Gateway" | App crashed | Check logs, memory limits |
| "Environment variable undefined" | Missing env var | Check .env, redeploy |
| "Database connection failed" | Wrong credentials or network | Check DATABASE_URL, firewall |

---

*Next: [Docker Deep Dive](docker.md) | [CI/CD Pipeline](ci-cd.md)*


