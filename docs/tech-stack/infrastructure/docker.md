# Docker Concepts

> Understanding containerization for CareCircle.

---

## 1. What Is Docker?

### Plain English Explanation

Docker is a tool that **packages applications with everything they need to run**.

Think of it like **shipping containers**:
- Before containers: Loading cargo loose on ships (messy, inconsistent)
- With containers: Standardized boxes that fit anywhere (predictable, portable)

Your app runs the same way on your laptop, your colleague's machine, and in production.

### The Core Problem Docker Solves

```
WITHOUT DOCKER:
───────────────
Developer A: "Works on my machine!" (Node 18, Ubuntu)
Developer B: "Broken on mine" (Node 16, Windows)
Production: "Crashes randomly" (Node 20, Alpine)

WITH DOCKER:
────────────
Everyone runs the SAME container
Same Node version, same OS, same dependencies
"Works everywhere"
```

---

## 2. Core Concepts & Terminology

### The Container Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DOCKER CONCEPTS                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  IMAGE                              CONTAINER                                │
│  ─────                              ─────────                                │
│  Blueprint/Recipe                   Running instance                         │
│  Read-only template                 Writable layer on top                   │
│  Built from Dockerfile              Created from image                       │
│                                                                              │
│  Analogy:                                                                    │
│  Image = Class                      Container = Object                       │
│  Image = Recipe                     Container = Cooked meal                  │
│                                                                              │
│                                                                              │
│  DOCKERFILE                         DOCKER COMPOSE                           │
│  ──────────                         ──────────────                           │
│  Instructions to build image        Orchestrate multiple containers          │
│  One service                        Multiple services together               │
│                                                                              │
│  Example:                           Example:                                 │
│  Build API image                    Run API + DB + Redis together            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Terminology

| Term | Definition |
|------|------------|
| **Image** | Packaged application template |
| **Container** | Running instance of an image |
| **Dockerfile** | Recipe to build an image |
| **Volume** | Persistent storage for containers |
| **Network** | Communication between containers |
| **Registry** | Storage for images (Docker Hub) |

---

## 3. How Docker Works

### Layer System

```
┌─────────────────────────────────────┐
│     Container (writable layer)      │  ← Your app's runtime changes
├─────────────────────────────────────┤
│     Application code                │  ← Your code
├─────────────────────────────────────┤
│     npm install (dependencies)      │  ← node_modules
├─────────────────────────────────────┤
│     Node.js runtime                 │  ← Node 20
├─────────────────────────────────────┤
│     Alpine Linux                    │  ← Base OS
└─────────────────────────────────────┘

Each layer is CACHED
Change code? Only rebuild top layers
Much faster rebuilds!
```

### Multi-Stage Builds

```dockerfile
# Stage 1: Build (large, has dev tools)
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production (small, only runtime)
FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/main.js"]

# Result: Small production image (no TypeScript, no dev deps)
```

---

## 4. CareCircle's Docker Setup

### Services

```yaml
# docker-compose.yml (local development)
services:
  postgres:     # Database
  redis:        # Cache
  rabbitmq:     # Message broker

# docker-compose.prod.yml (production)
services:
  api:          # NestJS backend (4 replicas)
  web:          # Next.js frontend (3 replicas)
  workers:      # Background jobs (3 replicas)
  nginx:        # Load balancer
```

### Image Structure

```
carecircle-api:latest
├── Node.js 20 Alpine
├── NestJS application
├── Production dependencies only
└── ~200MB

carecircle-web:latest
├── Node.js 20 Alpine
├── Next.js standalone output
├── Static assets
└── ~150MB

carecircle-workers:latest
├── Node.js 20 Alpine
├── BullMQ workers
├── Minimal dependencies
└── ~100MB
```

---

## 5. When to Use Docker ✅

### Use Docker When:
- Need consistent environments across team
- Deploying to any cloud provider
- Running multiple services locally
- Isolating dependencies between projects

### Use Docker Compose When:
- Running multiple containers together
- Local development with services
- Defining service relationships
- Single-server deployments

### Use Volumes When:
- Data must persist (databases)
- Sharing files between containers
- Development hot-reload

---

## 6. When to AVOID Patterns ❌

### DON'T Run as Root

```dockerfile
# ❌ BAD: Running as root (security risk)
FROM node:20-alpine
CMD ["node", "app.js"]

# ✅ GOOD: Run as non-root user
FROM node:20-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
CMD ["node", "app.js"]
```

### DON'T Include Secrets in Images

```dockerfile
# ❌ BAD: Secret in image
ENV DATABASE_PASSWORD=secret123

# ✅ GOOD: Pass at runtime
# docker run -e DATABASE_PASSWORD=$SECRET myimage
```

### DON'T Use Latest Tag in Production

```dockerfile
# ❌ BAD: Unpredictable version
FROM node:latest

# ✅ GOOD: Specific version
FROM node:20.11-alpine
```

---

## 7. Best Practices

### Dockerfile Best Practices

```dockerfile
# 1. Use specific base image versions
FROM node:20.11-alpine

# 2. Set working directory
WORKDIR /app

# 3. Copy package files first (cache optimization)
COPY package*.json ./

# 4. Install dependencies
RUN npm ci --only=production

# 5. Copy application code last
COPY . .

# 6. Use non-root user
USER node

# 7. Expose port
EXPOSE 3001

# 8. Use exec form for CMD
CMD ["node", "dist/main.js"]
```

### Layer Caching Strategy

```dockerfile
# SLOW: Reinstall deps on any code change
COPY . .
RUN npm install

# FAST: Only reinstall when package.json changes
COPY package*.json ./
RUN npm ci
COPY . .
```

---

## 8. Common Mistakes & How to Avoid Them

### Mistake 1: Large Images

```
❌ node:20 (900MB)
✅ node:20-alpine (150MB)

Alpine Linux is minimal, much smaller
```

### Mistake 2: Not Using .dockerignore

```
# .dockerignore
node_modules
.git
*.md
.env
dist
```

### Mistake 3: No Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3001/health || exit 1
```

---

## 9. Quick Reference

### Common Commands

```bash
# Build image
docker build -t myapp:latest .

# Run container
docker run -d -p 3001:3001 myapp:latest

# List containers
docker ps

# View logs
docker logs <container-id>

# Stop container
docker stop <container-id>

# Remove all stopped containers
docker container prune
```

### Docker Compose Commands

```bash
# Start all services
docker compose up -d

# Stop all services
docker compose down

# View logs
docker compose logs -f

# Rebuild and start
docker compose up -d --build

# Scale service
docker compose up -d --scale api=4
```

### Useful Flags

| Flag | Purpose |
|------|---------|
| `-d` | Detached (background) |
| `-p` | Port mapping |
| `-v` | Volume mount |
| `-e` | Environment variable |
| `--rm` | Remove after exit |
| `--name` | Container name |

---

## 10. CareCircle Docker Files

| File | Purpose |
|------|---------|
| `apps/api/Dockerfile` | Build API image |
| `apps/web/Dockerfile` | Build Web image |
| `apps/workers/Dockerfile` | Build Workers image |
| `docker-compose.yml` | Local development |
| `docker-compose.prod.yml` | Production deployment |
| `nginx.conf` | Nginx configuration |

---

*Next: [CI/CD](ci-cd.md) | [Infrastructure Overview](_INFRASTRUCTURE_OVERVIEW.md)*

