# 🐳 Docker Deep Dive - Complete Guide

> A comprehensive guide to Docker - multi-stage builds, optimization techniques, security best practices, Docker Compose, and production deployment patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Docker is a containerization platform that packages applications with their dependencies into isolated, portable units called containers, using Linux namespaces for isolation and cgroups for resource control, enabling consistent environments from development to production."

### The 7 Key Concepts (Remember These!)
```
1. IMAGE          → Read-only template (layered filesystem)
2. CONTAINER      → Running instance of an image
3. DOCKERFILE     → Build instructions for image
4. LAYER          → Each instruction creates a cached layer
5. VOLUME         → Persistent data storage outside container
6. NETWORK        → Container communication
7. REGISTRY       → Image storage (Docker Hub, ECR, GCR)
```

### Docker Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                    DOCKER ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    DOCKER CLIENT                          │  │
│  │    docker build | docker run | docker pull | docker push  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                              │ REST API                         │
│                              ▼                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    DOCKER DAEMON                          │  │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐ │  │
│  │  │   Images    │ │ Containers  │ │     Networks        │ │  │
│  │  │   Manager   │ │   Manager   │ │     Manager         │ │  │
│  │  └─────────────┘ └─────────────┘ └─────────────────────┘ │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                     containerd                            │  │
│  │              (Container Runtime)                          │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                        runc                               │  │
│  │         (OCI Runtime - creates containers)                │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Image Layers
```
┌─────────────────────────────────────────────────────────────────┐
│                    DOCKER IMAGE LAYERS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Dockerfile:                   Image Layers:                   │
│  ────────────                  ─────────────                   │
│                                                                 │
│  FROM node:20-alpine    ──▶   [Base Layer - 50MB]             │
│                                      │                         │
│  WORKDIR /app           ──▶   [Metadata Layer - 0KB]          │
│                                      │                         │
│  COPY package*.json .   ──▶   [Package Files - 100KB]         │
│                                      │                         │
│  RUN npm ci             ──▶   [Dependencies - 150MB] ◀─ CACHED│
│                                      │                         │
│  COPY . .               ──▶   [App Code - 5MB]                │
│                                      │                         │
│  RUN npm run build      ──▶   [Build Output - 10MB]           │
│                                      │                         │
│  CMD ["node", "dist"]   ──▶   [Metadata Layer - 0KB]          │
│                                                                 │
│  ⚠️ Order matters! Put rarely-changing layers first           │
│     to maximize cache efficiency.                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Multi-Stage Build
```
┌─────────────────────────────────────────────────────────────────┐
│                   MULTI-STAGE BUILD                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  STAGE 1: Build                    STAGE 2: Production         │
│  ──────────────────                ──────────────────          │
│                                                                 │
│  FROM node:20 AS builder           FROM node:20-alpine         │
│  WORKDIR /app                      WORKDIR /app                │
│  COPY package*.json .              COPY --from=builder         │
│  RUN npm ci                             /app/dist ./dist       │
│  COPY . .                          COPY --from=builder         │
│  RUN npm run build                      /app/node_modules      │
│                                         ./node_modules         │
│  ┌─────────────────────┐          CMD ["node", "dist"]        │
│  │ Full Node.js: 1GB   │                                      │
│  │ Dev dependencies    │          ┌─────────────────────┐     │
│  │ Build tools         │          │ Alpine: 150MB       │     │
│  │ Source code         │          │ Only prod deps      │     │
│  └─────────────────────┘          │ Built artifacts     │     │
│           │                        └─────────────────────┘     │
│           │                                                    │
│           └──── Only built artifacts copied ────▶              │
│                 (90% smaller final image!)                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Multi-stage build"** | "We use multi-stage builds to keep production images small" |
| **"Layer caching"** | "We order Dockerfile instructions to maximize layer caching" |
| **"Non-root user"** | "Containers run as non-root user for security" |
| **"Distroless"** | "We use distroless images - minimal attack surface, no shell" |
| **"Health check"** | "Health checks enable orchestrator to detect unhealthy containers" |
| **"BuildKit"** | "BuildKit enables parallel builds and better caching" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Alpine base size | **~5MB** | Minimal base image |
| Distroless size | **~2MB** | No OS, just runtime |
| Layer limit | **127** | Max layers in image |
| Default memory | **Unlimited** | Always set limits! |
| Healthcheck interval | **30s** | Default check frequency |
| Image scan target | **0 critical** | No critical vulnerabilities |

### The "Wow" Statement (Memorize This!)
> "We optimized our Docker images from 1.2GB to 150MB using multi-stage builds. Stage one uses node:20 with all build tools, runs npm ci and builds the app. Stage two uses node:20-alpine, copies only the dist folder and production node_modules. We order Dockerfile instructions for cache efficiency - package.json copied before source code so npm install layer is cached when only code changes. Containers run as non-root user (node:node), we set resource limits (memory, CPU), and include health checks for orchestrator integration. We use BuildKit for parallel layer builds and scan images with Trivy in CI - zero critical vulnerabilities policy."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌─────────────────┐        ┌─────────────────────────────┐   │
│  │   Dockerfile    │ ─────▶ │     Docker Build            │   │
│  └─────────────────┘        │  (BuildKit + Layer Cache)   │   │
│                             └─────────────────────────────┘   │
│                                         │                      │
│                                         ▼                      │
│                             ┌─────────────────────────────┐   │
│                             │      Docker Image           │   │
│                             │   (Immutable, Layered)      │   │
│                             └─────────────────────────────┘   │
│                                         │                      │
│              ┌──────────────────────────┼──────────────────┐  │
│              │                          │                  │  │
│              ▼                          ▼                  ▼  │
│  ┌───────────────────┐   ┌───────────────────┐  ┌──────────┐ │
│  │   Container 1     │   │   Container 2     │  │ Registry │ │
│  │ (Running Instance)│   │ (Running Instance)│  │ (Storage)│ │
│  │                   │   │                   │  │          │ │
│  │ ┌───────────────┐ │   │ ┌───────────────┐ │  │ - Push   │ │
│  │ │    App        │ │   │ │    App        │ │  │ - Pull   │ │
│  │ └───────────────┘ │   │ └───────────────┘ │  │ - Tag    │ │
│  │ ┌───────────────┐ │   │ ┌───────────────┐ │  │          │ │
│  │ │   Volume      │ │   │ │   Volume      │ │  └──────────┘ │
│  │ └───────────────┘ │   │ └───────────────┘ │              │
│  └───────────────────┘   └───────────────────┘              │
│                                                              │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is the difference between image and container?"**
> "Image is a read-only template with layered filesystem - the blueprint. Container is a running instance of an image with writable layer on top. Many containers can run from one image."

**Q: "How do you optimize Docker images?"**
> "Multi-stage builds (separate build and runtime). Use smaller base images (alpine, distroless). Order Dockerfile for cache (dependencies before code). Combine RUN commands to reduce layers. Use .dockerignore. Remove unnecessary files in same layer they're created."

**Q: "COPY vs ADD?"**
> "COPY just copies files. ADD can also extract tar archives and fetch URLs. Best practice: use COPY for simplicity and clarity. Only use ADD when you specifically need tar extraction."

**Q: "CMD vs ENTRYPOINT?"**
> "ENTRYPOINT is the executable that runs. CMD provides default arguments to ENTRYPOINT. ENTRYPOINT for 'what to run', CMD for 'default arguments'. CMD can be overridden at runtime, ENTRYPOINT is harder to override."

**Q: "How do you handle secrets in Docker?"**
> "Never put secrets in images or Dockerfile. Use environment variables (not ideal but common). Better: Docker secrets (Swarm), mounted secret files, or external secret managers. BuildKit has --secret flag for build-time secrets."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How do you optimize Docker builds?"

**Junior Answer:**
> "Use a smaller base image."

**Senior Answer:**
> "Docker optimization is multi-faceted:

1. **Multi-stage builds**: Separate build environment (with compilers, dev tools) from runtime. Copy only artifacts to final stage. Can reduce image size by 90%.

2. **Layer caching**: Order Dockerfile instructions by change frequency. Copy package.json before source code so dependencies layer is cached when only code changes.

3. **Base image selection**: Alpine (~5MB) for most cases. Distroless for maximum security (no shell, minimal attack surface). Slim variants as middle ground.

4. **Minimize layers**: Combine related RUN commands. Clean up in the same layer (rm cache files). Use BuildKit for parallel builds.

5. **Security**: Run as non-root user. Scan images for vulnerabilities. Pin versions for reproducibility. Don't expose unnecessary ports.

These optimizations improve build time, reduce storage costs, speed up deployments, and minimize security attack surface."

### When Asked: "How do you handle Docker in production?"

**Junior Answer:**
> "Just run docker-compose up."

**Senior Answer:**
> "Production Docker requires several considerations:

1. **Image registry**: Private registry (ECR, GCR, Harbor). Image signing for integrity. Automated vulnerability scanning.

2. **Resource limits**: Always set memory and CPU limits. Prevents noisy neighbor problems. OOM killer behavior with --oom-kill-disable.

3. **Health checks**: Proper HEALTHCHECK instructions. Enables orchestrator to detect and replace unhealthy containers.

4. **Logging**: Use json-file or external logging driver. Consider log rotation. Centralize with ELK/Loki.

5. **Orchestration**: Use Kubernetes, ECS, or Swarm. Handle scaling, rolling updates, service discovery.

6. **Security**: Non-root user, read-only filesystem where possible, minimal capabilities, secret management.

7. **Networking**: Proper network segmentation. TLS for service communication. Load balancing considerations."

---

## 📚 Table of Contents

1. [Dockerfile Best Practices](#1-dockerfile-best-practices)
2. [Multi-Stage Builds](#2-multi-stage-builds)
3. [Image Optimization](#3-image-optimization)
4. [Docker Compose](#4-docker-compose)
5. [Networking](#5-networking)
6. [Volumes and Storage](#6-volumes-and-storage)
7. [Security](#7-security)
8. [Common Pitfalls](#8-common-pitfalls)
9. [Interview Questions](#9-interview-questions)

---

## 1. Dockerfile Best Practices

```dockerfile
# ════════════════════════════════════════════════════════════════
# DOCKERFILE BEST PRACTICES
# ════════════════════════════════════════════════════════════════

# 1. Use specific base image tags (not latest)
FROM node:20.10.0-alpine3.19

# 2. Set labels for metadata
LABEL maintainer="team@example.com"
LABEL version="1.0.0"
LABEL description="Production API server"

# 3. Set working directory
WORKDIR /app

# 4. Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 -G nodejs

# 5. Copy dependency files first (cache optimization)
COPY package.json package-lock.json ./

# 6. Install dependencies with clean cache
RUN npm ci --only=production && \
    npm cache clean --force

# 7. Copy application code
COPY --chown=nodejs:nodejs . .

# 8. Build application (if needed)
RUN npm run build

# 9. Remove unnecessary files
RUN rm -rf src tests *.md

# 10. Set environment variables
ENV NODE_ENV=production \
    PORT=3000

# 11. Expose port (documentation)
EXPOSE 3000

# 12. Add healthcheck
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# 13. Switch to non-root user
USER nodejs

# 14. Use exec form for CMD (proper signal handling)
CMD ["node", "dist/server.js"]


# ════════════════════════════════════════════════════════════════
# .dockerignore - ALWAYS INCLUDE
# ════════════════════════════════════════════════════════════════

# .dockerignore
node_modules
npm-debug.log
Dockerfile*
docker-compose*
.dockerignore
.git
.gitignore
.env*
*.md
tests
coverage
.nyc_output
.vscode
.idea
*.log
dist
build
```

---

## 2. Multi-Stage Builds

```dockerfile
# ════════════════════════════════════════════════════════════════
# MULTI-STAGE BUILD - NODE.JS APPLICATION
# ════════════════════════════════════════════════════════════════

# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app

# Copy package files
COPY package.json package-lock.json ./

# Install all dependencies (including dev)
RUN npm ci

# Stage 2: Builder
FROM node:20-alpine AS builder
WORKDIR /app

# Copy dependencies from deps stage
COPY --from=deps /app/node_modules ./node_modules

# Copy source code
COPY . .

# Build application
RUN npm run build

# Prune dev dependencies
RUN npm prune --production

# Stage 3: Production
FROM node:20-alpine AS production
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 -G nodejs

# Copy only necessary files from builder
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./

# Set environment
ENV NODE_ENV=production

# Health check
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

# Switch to non-root user
USER nodejs

EXPOSE 3000

CMD ["node", "dist/server.js"]


# ════════════════════════════════════════════════════════════════
# MULTI-STAGE BUILD - REACT APPLICATION
# ════════════════════════════════════════════════════════════════

# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build

# Stage 2: Production with nginx
FROM nginx:alpine AS production

# Copy custom nginx config
COPY nginx.conf /etc/nginx/nginx.conf

# Copy built assets from builder
COPY --from=builder /app/build /usr/share/nginx/html

# Add healthcheck
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD wget -q --spider http://localhost:80/health || exit 1

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]


# ════════════════════════════════════════════════════════════════
# MULTI-STAGE WITH TEST STAGE
# ════════════════════════════════════════════════════════════════

# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Stage 2: Test (can be run separately)
FROM deps AS test
COPY . .
RUN npm run lint && npm run test

# Stage 3: Build (only if tests pass)
FROM deps AS builder
COPY . .
RUN npm run build

# Stage 4: Production
FROM node:20-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=deps /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]

# Build commands:
# docker build --target test -t myapp:test .
# docker build --target production -t myapp:prod .
```

---

## 3. Image Optimization

```dockerfile
# ════════════════════════════════════════════════════════════════
# IMAGE OPTIMIZATION TECHNIQUES
# ════════════════════════════════════════════════════════════════

# ──────────────────────────────────────────────────────────────
# Technique 1: Choose the right base image
# ──────────────────────────────────────────────────────────────

# node:20         - Full Debian (~1GB)   - Development
# node:20-slim    - Minimal Debian (~200MB) - Balance
# node:20-alpine  - Alpine Linux (~150MB) - Small
# gcr.io/distroless/nodejs20 (~50MB) - Production, no shell

# ──────────────────────────────────────────────────────────────
# Technique 2: Combine RUN commands
# ──────────────────────────────────────────────────────────────

# Bad - Multiple layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*

# Good - Single layer, cleanup in same layer
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# ──────────────────────────────────────────────────────────────
# Technique 3: Use BuildKit features
# ──────────────────────────────────────────────────────────────

# syntax=docker/dockerfile:1.4

# Mount cache for package managers
FROM node:20-alpine AS builder
WORKDIR /app

# Cache npm packages between builds
RUN --mount=type=cache,target=/root/.npm \
    npm ci

# Mount secrets securely (not stored in image)
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm ci

# Parallel execution with heredocs
RUN <<EOF
    npm run build
    npm prune --production
EOF

# ──────────────────────────────────────────────────────────────
# Technique 4: Use distroless for production
# ──────────────────────────────────────────────────────────────

FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Distroless - no shell, minimal attack surface
FROM gcr.io/distroless/nodejs20-debian12
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["dist/server.js"]

# ──────────────────────────────────────────────────────────────
# Technique 5: Layer ordering for cache
# ──────────────────────────────────────────────────────────────

FROM node:20-alpine
WORKDIR /app

# 1. System dependencies (rarely change)
RUN apk add --no-cache dumb-init

# 2. Package files (change when deps change)
COPY package.json package-lock.json ./

# 3. Install dependencies (cached if package files unchanged)
RUN npm ci --only=production

# 4. Application code (changes frequently)
COPY . .

# 5. Build (runs when code changes)
RUN npm run build

CMD ["dumb-init", "node", "dist/server.js"]
```

```bash
# ════════════════════════════════════════════════════════════════
# BUILD COMMANDS AND OPTIMIZATION
# ════════════════════════════════════════════════════════════════

# Enable BuildKit (faster, better caching)
export DOCKER_BUILDKIT=1

# Build with cache
docker build -t myapp:latest .

# Build with no cache (for debugging)
docker build --no-cache -t myapp:latest .

# Build specific stage
docker build --target builder -t myapp:builder .

# Build with build arguments
docker build --build-arg NODE_ENV=production -t myapp:prod .

# Build with secret (BuildKit)
docker build --secret id=npmrc,src=.npmrc -t myapp:latest .

# Analyze image size
docker images myapp
docker history myapp:latest

# Inspect image layers
docker inspect myapp:latest | jq '.[0].RootFS.Layers'

# Use dive for detailed layer analysis
dive myapp:latest
```

---

## 4. Docker Compose

```yaml
# ════════════════════════════════════════════════════════════════
# DOCKER COMPOSE - COMPLETE EXAMPLE
# docker-compose.yml
# ════════════════════════════════════════════════════════════════

version: '3.9'

services:
  # ════════════════════════════════════════════════════════════
  # APPLICATION
  # ════════════════════════════════════════════════════════════
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
      args:
        - NODE_ENV=production
    image: myapp:${VERSION:-latest}
    container_name: myapp
    restart: unless-stopped
    
    # Environment
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:${DB_PASSWORD}@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    env_file:
      - .env
    
    # Ports
    ports:
      - "3000:3000"
    
    # Health check
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    
    # Resources
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 128M
    
    # Dependencies
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    
    # Volumes
    volumes:
      - app-logs:/app/logs
    
    # Networks
    networks:
      - frontend
      - backend

  # ════════════════════════════════════════════════════════════
  # DATABASE
  # ════════════════════════════════════════════════════════════
  db:
    image: postgres:15-alpine
    container_name: myapp-db
    restart: unless-stopped
    
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=myapp
    
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    
    networks:
      - backend

  # ════════════════════════════════════════════════════════════
  # REDIS CACHE
  # ════════════════════════════════════════════════════════════
  redis:
    image: redis:7-alpine
    container_name: myapp-redis
    restart: unless-stopped
    
    command: redis-server --appendonly yes --maxmemory 128mb --maxmemory-policy allkeys-lru
    
    volumes:
      - redis-data:/data
    
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    
    networks:
      - backend

  # ════════════════════════════════════════════════════════════
  # NGINX REVERSE PROXY
  # ════════════════════════════════════════════════════════════
  nginx:
    image: nginx:alpine
    container_name: myapp-nginx
    restart: unless-stopped
    
    ports:
      - "80:80"
      - "443:443"
    
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    
    depends_on:
      - app
    
    networks:
      - frontend

# ════════════════════════════════════════════════════════════════
# VOLUMES
# ════════════════════════════════════════════════════════════════
volumes:
  postgres-data:
    driver: local
  redis-data:
    driver: local
  app-logs:
    driver: local

# ════════════════════════════════════════════════════════════════
# NETWORKS
# ════════════════════════════════════════════════════════════════
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access


# ════════════════════════════════════════════════════════════════
# OVERRIDE FOR DEVELOPMENT
# docker-compose.override.yml (auto-loaded)
# ════════════════════════════════════════════════════════════════
```

```yaml
# docker-compose.override.yml
version: '3.9'

services:
  app:
    build:
      target: development
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=app:*
    command: npm run dev
    ports:
      - "3000:3000"
      - "9229:9229"  # Debug port
```

```bash
# Docker Compose commands
docker-compose up -d                    # Start all services
docker-compose up -d --build            # Rebuild and start
docker-compose down                     # Stop and remove
docker-compose down -v                  # Also remove volumes
docker-compose logs -f app              # Follow logs
docker-compose exec app sh              # Shell into container
docker-compose ps                       # List containers
docker-compose config                   # Validate and view config
```

---

## 5. Networking

```yaml
# ════════════════════════════════════════════════════════════════
# DOCKER NETWORKING
# ════════════════════════════════════════════════════════════════

# Network types:
# - bridge (default): Isolated network for container communication
# - host: Share host's network stack (no isolation)
# - none: No networking
# - overlay: Multi-host networking (Swarm/K8s)

# docker-compose.yml
version: '3.9'

services:
  # Frontend network (exposed to internet)
  web:
    networks:
      - frontend

  # Backend services (internal only)
  api:
    networks:
      - frontend  # Receives requests from web
      - backend   # Talks to database

  # Database (completely internal)
  db:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # Cannot access internet
```

```bash
# ════════════════════════════════════════════════════════════════
# NETWORK COMMANDS
# ════════════════════════════════════════════════════════════════

# Create network
docker network create mynetwork

# List networks
docker network ls

# Inspect network
docker network inspect mynetwork

# Connect container to network
docker network connect mynetwork mycontainer

# Disconnect container
docker network disconnect mynetwork mycontainer

# Run container on specific network
docker run --network mynetwork myimage

# DNS: Containers can reach each other by service name
# e.g., from 'app' container: curl http://db:5432
```

---

## 6. Volumes and Storage

```yaml
# ════════════════════════════════════════════════════════════════
# DOCKER VOLUMES AND STORAGE
# ════════════════════════════════════════════════════════════════

version: '3.9'

services:
  app:
    volumes:
      # Named volume (managed by Docker)
      - app-data:/app/data
      
      # Bind mount (host path)
      - ./config:/app/config:ro
      
      # Anonymous volume (prevents overwrite)
      - /app/node_modules
      
      # tmpfs (in-memory, not persisted)
      - type: tmpfs
        target: /app/tmp
        tmpfs:
          size: 100M

volumes:
  app-data:
    driver: local
    driver_opts:
      type: none
      device: /data/app
      o: bind
```

```bash
# ════════════════════════════════════════════════════════════════
# VOLUME COMMANDS
# ════════════════════════════════════════════════════════════════

# Create volume
docker volume create myvolume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect myvolume

# Remove volume
docker volume rm myvolume

# Remove unused volumes
docker volume prune

# Backup volume
docker run --rm -v myvolume:/data -v $(pwd):/backup \
    alpine tar cvf /backup/backup.tar /data

# Restore volume
docker run --rm -v myvolume:/data -v $(pwd):/backup \
    alpine tar xvf /backup/backup.tar -C /
```

---

## 7. Security

```dockerfile
# ════════════════════════════════════════════════════════════════
# DOCKER SECURITY BEST PRACTICES
# ════════════════════════════════════════════════════════════════

# 1. Use specific image versions (not :latest)
FROM node:20.10.0-alpine3.19

# 2. Create and use non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

# 3. Set filesystem permissions
RUN chown -R appuser:appgroup /app

# 4. Switch to non-root user
USER appuser

# 5. Make filesystem read-only where possible
# In docker run: --read-only --tmpfs /tmp

# 6. Drop capabilities
# In docker run: --cap-drop=ALL --cap-add=NET_BIND_SERVICE

# 7. Set security options
# In docker run: --security-opt=no-new-privileges:true

# 8. Don't store secrets in image
# Use runtime secrets or environment variables

# 9. Scan for vulnerabilities
# RUN npm audit --audit-level=critical
```

```yaml
# ════════════════════════════════════════════════════════════════
# SECURE DOCKER COMPOSE
# ════════════════════════════════════════════════════════════════

version: '3.9'

services:
  app:
    image: myapp:latest
    
    # Security options
    security_opt:
      - no-new-privileges:true
    
    # Drop all capabilities, add only needed
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    
    # Read-only filesystem
    read_only: true
    tmpfs:
      - /tmp
      - /app/logs
    
    # Resource limits (prevent DoS)
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
          pids: 100  # Limit processes
    
    # Non-root user
    user: "1001:1001"
    
    # Health check
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

```bash
# ════════════════════════════════════════════════════════════════
# SECURITY SCANNING
# ════════════════════════════════════════════════════════════════

# Scan with Trivy
trivy image myapp:latest

# Scan with Snyk
snyk container test myapp:latest

# Scan with Docker Scout
docker scout cves myapp:latest

# Scan Dockerfile
hadolint Dockerfile

# Check for secrets in image
docker history --no-trunc myapp:latest | grep -i secret
```

---

## 8. Common Pitfalls

```dockerfile
# ════════════════════════════════════════════════════════════════
# DOCKER PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Running as root
# Bad
FROM node:20
COPY . .
CMD ["node", "server.js"]  # Runs as root!

# Good
FROM node:20
RUN useradd -m appuser
USER appuser
COPY --chown=appuser . .
CMD ["node", "server.js"]

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 2: Using :latest tag
# Bad
FROM node:latest  # Changes unpredictably

# Good
FROM node:20.10.0-alpine3.19  # Specific version

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 3: Not leveraging cache
# Bad - Code change invalidates everything
COPY . .
RUN npm install

# Good - Dependencies cached separately
COPY package*.json ./
RUN npm install
COPY . .

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 4: Large images
# Bad - Full OS with dev tools
FROM ubuntu:latest
RUN apt-get update && apt-get install -y nodejs npm
COPY . .

# Good - Minimal base, multi-stage
FROM node:20-alpine AS builder
COPY . .
RUN npm ci && npm run build

FROM node:20-alpine
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 5: Secrets in image
# Bad - Secret stored in layer
COPY .env .
ENV API_KEY=secret123

# Good - Runtime secrets
# docker run -e API_KEY=$API_KEY myapp
# Or use Docker secrets / external secret manager

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 6: No health check
# Bad - Orchestrator can't detect unhealthy container
CMD ["node", "server.js"]

# Good
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1
CMD ["node", "server.js"]

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 7: Shell form vs exec form
# Bad - Shell form, signals not passed to app
CMD npm start

# Good - Exec form, proper signal handling
CMD ["node", "server.js"]
# Or use dumb-init/tini for signal forwarding
CMD ["dumb-init", "node", "server.js"]
```

---

## 9. Interview Questions

### Basic Questions

**Q: "What is Docker?"**
> "Docker is a containerization platform that packages applications with dependencies into portable containers. Uses Linux namespaces for isolation, cgroups for resource control. Containers share the host kernel but are isolated from each other. Enables consistent environments from dev to prod."

**Q: "Image vs container?"**
> "Image is a read-only template - layered filesystem with app and dependencies. Container is a running instance with a writable layer on top. Like class vs object - one image can spawn many containers."

**Q: "How do layers work?"**
> "Each Dockerfile instruction creates a layer. Layers are cached and reused. Order matters - put frequently changing things last. When a layer changes, all subsequent layers rebuild. Layers are read-only; container adds writable layer on top."

### Intermediate Questions

**Q: "How do you optimize Docker images?"**
> "1) Multi-stage builds - separate build and runtime. 2) Use alpine/distroless base images. 3) Order Dockerfile for cache efficiency. 4) Combine RUN commands, cleanup in same layer. 5) Use .dockerignore. 6) Remove dev dependencies in production."

**Q: "COPY vs ADD?"**
> "COPY simply copies files. ADD can also extract tar and fetch URLs. Best practice: use COPY for clarity, ADD only when you need tar extraction. Neither downloads URLs in modern Docker (security)."

**Q: "How do you handle secrets?"**
> "Never bake secrets into images. Options: Environment variables (visible in inspect), Docker secrets (Swarm), mounted files, external secret managers (Vault, AWS Secrets Manager). BuildKit --secret for build-time secrets."

### Advanced Questions

**Q: "How do you secure Docker containers?"**
> "1) Non-root user inside container. 2) Read-only filesystem where possible. 3) Drop all capabilities, add only needed. 4) Scan images for vulnerabilities. 5) Use specific image tags. 6) Resource limits. 7) Network segmentation. 8) no-new-privileges flag."

**Q: "Docker vs VMs?"**
> "VMs virtualize hardware, each has full OS - heavyweight, slow startup, strong isolation. Containers share host kernel, virtualize at OS level - lightweight, fast startup, process-level isolation. Containers for microservices/cloud native, VMs for strong isolation/different OS needs."

**Q: "Multi-stage builds?"**
> "Multiple FROM statements in Dockerfile. Each stage can use different base image. Final image only includes what you COPY from previous stages. Enables using full build tools in build stage, minimal runtime in production stage. Can reduce image size by 90%+."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                    DOCKERFILE CHECKLIST                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BASE IMAGE:                                                    │
│  □ Use specific version tag                                    │
│  □ Use smallest appropriate base (alpine/distroless)           │
│                                                                 │
│  BUILD:                                                         │
│  □ Multi-stage build                                           │
│  □ Order instructions for cache                                │
│  □ Combine RUN commands                                        │
│  □ Use .dockerignore                                           │
│                                                                 │
│  SECURITY:                                                      │
│  □ Run as non-root user                                        │
│  □ No secrets in image                                         │
│  □ Scan for vulnerabilities                                    │
│  □ Set resource limits                                         │
│                                                                 │
│  PRODUCTION:                                                    │
│  □ Health check                                                │
│  □ Proper signal handling (exec form)                          │
│  □ Logging configuration                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

COMMON COMMANDS:
┌────────────────────────────────────────────────────────────────┐
│ docker build -t name:tag .      Build image                    │
│ docker run -d -p 80:80 name     Run container                  │
│ docker exec -it container sh    Shell into container           │
│ docker logs -f container        Follow logs                    │
│ docker-compose up -d            Start services                 │
│ docker-compose down -v          Stop and remove volumes        │
└────────────────────────────────────────────────────────────────┘

IMAGE SIZE COMPARISON:
┌────────────────────────────────────────────────────────────────┐
│ node:20          ~1GB    (Full Debian)                         │
│ node:20-slim     ~200MB  (Minimal Debian)                      │
│ node:20-alpine   ~150MB  (Alpine Linux)                        │
│ distroless       ~50MB   (No OS, just runtime)                 │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

