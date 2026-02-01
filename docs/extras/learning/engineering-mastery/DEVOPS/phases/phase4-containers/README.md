# Phase 4: Containers & Orchestration
## Weeks 10-13 | Containerize Everything

> **Prerequisites:** Completed Phase 3 (Terraform)  
> **Time:** 10-15 hours per week  
> **Outcome:** Deploy scalable applications on Kubernetes

---

## 🎯 Learning Objectives

By the end of Phase 4, you will:
- [ ] Understand container concepts and benefits
- [ ] Write Dockerfiles and build images
- [ ] Use Docker Compose for multi-container apps
- [ ] Deploy applications to Kubernetes
- [ ] Manage scaling, updates, and secrets in K8s

---

## Why Containers?

```
TRADITIONAL DEPLOYMENT:              CONTAINER DEPLOYMENT:
┌─────────────────────────┐         ┌─────────────────────────┐
│        Server           │         │        Server           │
│  ┌───────┐ ┌───────┐   │         │  ┌───────────────────┐  │
│  │ App A │ │ App B │   │         │  │ Container Runtime │  │
│  │Node 14│ │Node 16│   │         │  │  ┌─────┐ ┌─────┐  │  │
│  │       │ │       │   │         │  │  │App A│ │App B│  │  │
│  │CONFLICT│         │   │         │  │  │ N14 │ │ N16 │  │  │
│  └───────┘ └───────┘   │         │  │  └─────┘ └─────┘  │  │
│  Shared OS, libraries   │         │  │  Isolated!        │  │
└─────────────────────────┘         │  └───────────────────┘  │
                                    └─────────────────────────┘

"Works on my machine" → "Works everywhere"
```

---

## 📅 Weekly Breakdown

### Week 10: Docker Fundamentals

**What you'll learn:**
- Container concepts (images, containers, layers)
- Dockerfile instructions
- Building and running containers
- Docker networking basics

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 19: Containers & Docker | 3 hours |
| [WSL2-DevOps-Beginner-Guide.md](../../WSL2-DevOps-Beginner-Guide.md) | Phase 6: Docker | 1 hour |

**Key concepts:**

```
DOCKER ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Dockerfile ──build──▶ Image ──run──▶ Container                │
│                                                                  │
│   "Recipe"              "Template"       "Running instance"     │
│                                                                  │
│   FROM node:20          Immutable        Has state              │
│   COPY . .              Shareable        Ephemeral              │
│   RUN npm install       Layered          Isolated               │
│   CMD ["npm", "start"]                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Hands-on exercises:**

```dockerfile
# Exercise 1: Basic Dockerfile
# Dockerfile

FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

```bash
# Exercise 2: Build and run
docker build -t my-app:1.0 .
docker run -d -p 3000:3000 --name my-app my-app:1.0

# Check logs
docker logs my-app

# Enter container
docker exec -it my-app sh

# Stop and remove
docker stop my-app
docker rm my-app
```

```dockerfile
# Exercise 3: Multi-stage build (production optimized)
# Dockerfile

# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

USER node
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

```bash
# Exercise 4: Docker commands
docker images                    # List images
docker ps                        # List running containers
docker ps -a                     # List all containers
docker logs <container>          # View logs
docker exec -it <container> sh   # Enter container
docker stop <container>          # Stop container
docker rm <container>            # Remove container
docker rmi <image>               # Remove image
docker system prune              # Clean up unused resources
```

**Checkpoint quiz:**
1. What's the difference between an image and a container?
2. Why use multi-stage builds?
3. What does `EXPOSE` do in a Dockerfile?

---

### Week 11: Docker Compose

**What you'll learn:**
- Multi-container applications
- Docker Compose file syntax
- Networking between containers
- Volumes for persistence
- Environment management

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [devop-complete-guide.md](../../devop-complete-guide.md) | Docker Compose sections | 2 hours |

**Key concepts:**

```
DOCKER COMPOSE ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────┐
│                     docker-compose.yml                          │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    carecircle-network                     │  │
│  │                                                           │  │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐              │  │
│  │  │   web   │───▶│   api   │───▶│postgres │              │  │
│  │  │  :3000  │    │  :3001  │    │  :5432  │              │  │
│  │  └─────────┘    └─────────┘    └─────────┘              │  │
│  │                       │              │                    │  │
│  │                       ▼              │                    │  │
│  │                 ┌─────────┐          │                    │  │
│  │                 │  redis  │◀─────────┘                    │  │
│  │                 │  :6379  │                               │  │
│  │                 └─────────┘                               │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

**Hands-on exercises:**

```yaml
# Exercise 1: Basic docker-compose.yml
version: '3.8'

services:
  web:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - API_URL=http://api:3001
    depends_on:
      - api

  api:
    build: ./backend
    ports:
      - "3001:3001"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

```yaml
# Exercise 2: Development overrides
# docker-compose.override.yml

version: '3.8'

services:
  api:
    build:
      target: development
    volumes:
      - ./backend:/app
      - /app/node_modules
    ports:
      - "9229:9229"  # Debug port
    environment:
      - NODE_ENV=development

  web:
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
```

```bash
# Exercise 3: Docker Compose commands
docker compose up                 # Start all services
docker compose up -d              # Start in background
docker compose down               # Stop and remove
docker compose logs               # View logs
docker compose logs api           # View specific service logs
docker compose ps                 # List services
docker compose exec api sh        # Enter service container
docker compose build              # Rebuild images
docker compose pull               # Pull latest images
```

```yaml
# Exercise 4: Production compose file
# docker-compose.prod.yml

version: '3.8'

services:
  api:
    image: ghcr.io/myorg/api:${TAG:-latest}
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:3001/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

**Checkpoint quiz:**
1. What's the purpose of `depends_on`?
2. How do containers communicate in Docker Compose?
3. What's the difference between `volumes` and `bind mounts`?

---

### Week 12: Kubernetes Basics

**What you'll learn:**
- Kubernetes architecture (control plane, nodes)
- Core objects (Pods, Deployments, Services)
- kubectl commands
- Local Kubernetes (minikube, kind, Docker Desktop)

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 27: Kubernetes Fundamentals | 4 hours |

**Key concepts:**

```
KUBERNETES ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────┐
│                        CONTROL PLANE                             │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐   │
│  │API Server │  │Scheduler  │  │Controller │  │   etcd    │   │
│  │           │  │           │  │  Manager  │  │           │   │
│  └───────────┘  └───────────┘  └───────────┘  └───────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│     NODE 1      │ │     NODE 2      │ │     NODE 3      │
│  ┌───────────┐  │ │  ┌───────────┐  │ │  ┌───────────┐  │
│  │  kubelet  │  │ │  │  kubelet  │  │ │  │  kubelet  │  │
│  └───────────┘  │ │  └───────────┘  │ │  └───────────┘  │
│  ┌─────┐┌─────┐ │ │  ┌─────┐┌─────┐ │ │  ┌─────┐       │
│  │ Pod ││ Pod │ │ │  │ Pod ││ Pod │ │ │  │ Pod │       │
│  └─────┘└─────┘ │ │  └─────┘└─────┘ │ │  └─────┘       │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

**Hands-on exercises:**

```yaml
# Exercise 1: Deployment
# deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
  labels:
    app: api
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
          image: my-api:1.0
          ports:
            - containerPort: 3001
          env:
            - name: NODE_ENV
              value: "production"
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"
```

```yaml
# Exercise 2: Service (expose internally)
# service.yaml

apiVersion: v1
kind: Service
metadata:
  name: api
spec:
  selector:
    app: api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3001
  type: ClusterIP
```

```yaml
# Exercise 3: Ingress (expose externally)
# ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: api.myapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api
                port:
                  number: 80
```

```bash
# Exercise 4: kubectl commands
kubectl apply -f deployment.yaml     # Create/update resource
kubectl get pods                     # List pods
kubectl get deployments              # List deployments
kubectl get services                 # List services
kubectl describe pod <pod-name>      # Pod details
kubectl logs <pod-name>              # View logs
kubectl exec -it <pod-name> -- sh    # Enter pod
kubectl delete -f deployment.yaml    # Delete resource
kubectl scale deployment api --replicas=5  # Scale
kubectl rollout status deployment/api      # Check rollout
kubectl rollout undo deployment/api        # Rollback
```

**Checkpoint quiz:**
1. What's the difference between a Pod and a Deployment?
2. How do Services enable communication between Pods?
3. What's the purpose of labels and selectors?

---

### Week 13: Kubernetes Production

**What you'll learn:**
- ConfigMaps and Secrets
- Health checks (liveness, readiness probes)
- Horizontal Pod Autoscaler (HPA)
- Persistent volumes
- Namespaces and RBAC

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapters 28-29: K8s Production | 4 hours |

**Hands-on exercises:**

```yaml
# Exercise 1: ConfigMap and Secret
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-config
data:
  NODE_ENV: "production"
  LOG_LEVEL: "info"

---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: api-secrets
type: Opaque
stringData:
  DATABASE_URL: "postgresql://user:pass@db:5432/myapp"
  JWT_SECRET: "super-secret-key"
```

```yaml
# Exercise 2: Using ConfigMap and Secret
# deployment-with-config.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  template:
    spec:
      containers:
        - name: api
          image: my-api:1.0
          envFrom:
            - configMapRef:
                name: api-config
            - secretRef:
                name: api-secrets
```

```yaml
# Exercise 3: Health checks
# deployment-with-probes.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  template:
    spec:
      containers:
        - name: api
          image: my-api:1.0
          livenessProbe:
            httpGet:
              path: /health/live
              port: 3001
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 3001
            initialDelaySeconds: 5
            periodSeconds: 5
```

```yaml
# Exercise 4: Horizontal Pod Autoscaler
# hpa.yaml
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

## 📋 Phase 4 Project: Full Stack on Kubernetes

**Deploy a complete application stack to Kubernetes:**
1. Frontend deployment + service
2. Backend deployment + service with health checks
3. PostgreSQL StatefulSet with persistent volume
4. Redis deployment
5. Ingress for external access
6. ConfigMaps and Secrets
7. Horizontal Pod Autoscaler

---

## ✅ Phase 4 Completion Checklist

Before moving to Phase 5, ensure you can:

- [ ] Write optimized Dockerfiles
- [ ] Use Docker Compose for local development
- [ ] Explain Kubernetes architecture
- [ ] Create Deployments, Services, and Ingresses
- [ ] Use ConfigMaps and Secrets
- [ ] Configure health probes
- [ ] Set up auto-scaling
- [ ] Complete the Phase 4 project

---

## 🔗 Quick Reference

**Docker commands:**
```bash
docker build -t name:tag .
docker run -d -p 3000:3000 name:tag
docker compose up -d
docker compose down
```

**kubectl commands:**
```bash
kubectl apply -f file.yaml
kubectl get pods/deployments/services
kubectl describe pod <name>
kubectl logs <pod>
kubectl exec -it <pod> -- sh
kubectl delete -f file.yaml
```

---

## ➡️ Next Step

Ready for Phase 5? [CI/CD & Cloud →](../phase5-cicd-cloud/README.md)

