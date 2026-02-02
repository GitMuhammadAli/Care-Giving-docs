# ☸️ Kubernetes Basics - Complete Guide

> A comprehensive guide to Kubernetes - pods, services, deployments, ingress, ConfigMaps, secrets, and production patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Kubernetes (K8s) is a container orchestration platform that automates deployment, scaling, and management of containerized applications across clusters of machines, using declarative configuration and a control loop that continuously reconciles desired state with actual state."

### The 7 Key Concepts (Remember These!)
```
1. POD             → Smallest deployable unit (1+ containers)
2. DEPLOYMENT      → Manages pod replicas and updates
3. SERVICE         → Stable networking for pods (load balancing)
4. INGRESS         → External HTTP/HTTPS routing
5. CONFIGMAP       → Configuration data (non-sensitive)
6. SECRET          → Sensitive data (base64 encoded)
7. NAMESPACE       → Virtual cluster isolation
```

### Kubernetes Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                   KUBERNETES ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌────────────────────────────────────────────────────────────┐│
│  │                    CONTROL PLANE                           ││
│  │  ┌─────────────┐ ┌─────────────┐ ┌───────────────────────┐││
│  │  │ API Server  │ │   etcd      │ │  Controller Manager   │││
│  │  │(kubectl→)   │ │ (state DB)  │ │ (reconciliation loop) │││
│  │  └─────────────┘ └─────────────┘ └───────────────────────┘││
│  │  ┌─────────────────────────────────────────────────────────┐│
│  │  │              Scheduler (assigns pods to nodes)          ││
│  │  └─────────────────────────────────────────────────────────┘│
│  └────────────────────────────────────────────────────────────┘│
│                              │                                  │
│                              ▼                                  │
│  ┌────────────────────────────────────────────────────────────┐│
│  │                      WORKER NODES                          ││
│  │                                                            ││
│  │  ┌─────────────────────┐    ┌─────────────────────┐       ││
│  │  │      Node 1         │    │      Node 2         │       ││
│  │  │  ┌───────────────┐  │    │  ┌───────────────┐  │       ││
│  │  │  │    kubelet    │  │    │  │    kubelet    │  │       ││
│  │  │  └───────────────┘  │    │  └───────────────┘  │       ││
│  │  │  ┌───────────────┐  │    │  ┌───────────────┐  │       ││
│  │  │  │  kube-proxy   │  │    │  │  kube-proxy   │  │       ││
│  │  │  └───────────────┘  │    │  └───────────────┘  │       ││
│  │  │  ┌─────┐ ┌─────┐   │    │  ┌─────┐ ┌─────┐   │       ││
│  │  │  │ Pod │ │ Pod │   │    │  │ Pod │ │ Pod │   │       ││
│  │  │  └─────┘ └─────┘   │    │  └─────┘ └─────┘   │       ││
│  │  └─────────────────────┘    └─────────────────────┘       ││
│  └────────────────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Pod, Deployment, Service Relationship
```
┌─────────────────────────────────────────────────────────────────┐
│              POD, DEPLOYMENT, SERVICE RELATIONSHIP              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DEPLOYMENT (manages replicas and updates)                     │
│  ┌────────────────────────────────────────────────────────────┐│
│  │  ReplicaSet (ensures desired # of pods running)            ││
│  │  ┌────────────────────────────────────────────────────────┐││
│  │  │                                                        │││
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │││
│  │  │  │    Pod 1    │  │    Pod 2    │  │    Pod 3    │   │││
│  │  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │   │││
│  │  │  │ │Container│ │  │ │Container│ │  │ │Container│ │   │││
│  │  │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │   │││
│  │  │  │ 10.0.1.5    │  │ 10.0.1.6    │  │ 10.0.2.3    │   │││
│  │  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘   │││
│  │  │         │                │                │          │││
│  │  └─────────┼────────────────┼────────────────┼──────────┘││
│  └────────────┼────────────────┼────────────────┼───────────┘│
│               │                │                │            │
│               └────────────────┼────────────────┘            │
│                                │                             │
│  SERVICE (stable endpoint, load balances to pods)            │
│  ┌─────────────────────────────┴────────────────────────────┐│
│  │  myapp-service:80  ───▶  selector: app=myapp             ││
│  │  (ClusterIP: 10.96.0.15)                                 ││
│  └──────────────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Declarative"** | "K8s is declarative - we define desired state, controller reconciles" |
| **"Control loop"** | "The controller runs a control loop comparing desired vs actual state" |
| **"Immutable infrastructure"** | "We treat pods as immutable - replace, don't modify" |
| **"Service discovery"** | "Services provide DNS-based service discovery within cluster" |
| **"Rolling update"** | "Deployments do rolling updates with zero downtime" |
| **"Readiness probe"** | "Readiness probes ensure traffic only goes to healthy pods" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Pods per node | **110** default | Kubernetes limit |
| Services per namespace | **5,000** | Soft limit |
| Nodes per cluster | **5,000** | Max supported |
| Default pod termination | **30 seconds** | Grace period |
| Default rolling update | **25%** max unavailable | Deployment strategy |
| Readiness probe default | **10 seconds** | Initial delay |

### The "Wow" Statement (Memorize This!)
> "We run a microservices platform on Kubernetes with 50+ services. Each service has a Deployment managing 3-10 replicas with resource requests and limits. We use horizontal pod autoscaling based on CPU and custom metrics from Prometheus. Services expose ClusterIP endpoints with readiness probes ensuring only healthy pods receive traffic. Ingress with cert-manager handles TLS termination and routing. ConfigMaps hold environment-specific configuration, Secrets store database credentials. We implement GitOps with ArgoCD - Git commits trigger deployments. Rolling updates with PodDisruptionBudgets ensure zero downtime. Namespaces separate environments (dev/staging/prod) with resource quotas and network policies."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│     EXTERNAL TRAFFIC                                           │
│           │                                                    │
│           ▼                                                    │
│  ┌────────────────┐                                           │
│  │    Ingress     │  (TLS termination, routing rules)        │
│  └────────┬───────┘                                           │
│           │                                                    │
│           ▼                                                    │
│  ┌────────────────┐       ┌────────────────┐                 │
│  │ Service: web   │──────▶│ Service: api   │                 │
│  │ (ClusterIP)    │       │ (ClusterIP)    │                 │
│  └────────┬───────┘       └────────┬───────┘                 │
│           │                        │                          │
│     ┌─────┴─────┐            ┌─────┴─────┐                   │
│     ▼           ▼            ▼           ▼                   │
│  ┌─────┐     ┌─────┐     ┌─────┐     ┌─────┐                │
│  │ Pod │     │ Pod │     │ Pod │     │ Pod │                │
│  └─────┘     └─────┘     └─────┘     └─────┘                │
│                                         │                    │
│                                         ▼                    │
│                              ┌────────────────┐             │
│                              │ Service: db    │             │
│                              │ (Headless)     │             │
│                              └────────┬───────┘             │
│                                       │                      │
│                                  ┌────┴────┐                │
│                                  ▼         ▼                │
│                              ┌─────┐   ┌─────┐              │
│                              │ Pod │   │ Pod │              │
│                              │(PVC)│   │(PVC)│              │
│                              └─────┘   └─────┘              │
│                                                              │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is a Pod?"**
> "Smallest deployable unit in Kubernetes. Contains one or more containers that share network namespace (same IP), storage volumes, and lifecycle. Usually one container per pod, but sidecar patterns use multiple."

**Q: "Deployment vs ReplicaSet vs Pod?"**
> "Pod is the running container(s). ReplicaSet ensures desired number of pod replicas. Deployment manages ReplicaSets, enabling rolling updates and rollbacks. Use Deployments, not ReplicaSets directly."

**Q: "Service types?"**
> "ClusterIP (default): Internal cluster access. NodePort: Expose on each node's IP. LoadBalancer: Cloud provider load balancer. ExternalName: DNS CNAME. Headless: No load balancing, direct pod access."

**Q: "How does Kubernetes networking work?"**
> "Every pod gets unique IP. Pods can communicate with any pod without NAT. Services provide stable IPs and DNS names. kube-proxy manages iptables/IPVS rules for service routing."

**Q: "What are probes?"**
> "Liveness: Is container alive? Restart if fails. Readiness: Is container ready for traffic? Remove from service if fails. Startup: Is container started? For slow-starting apps."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How does Kubernetes work?"

**Junior Answer:**
> "It runs containers in a cluster."

**Senior Answer:**
> "Kubernetes implements a declarative, desired-state model:

1. **You declare** desired state in YAML (3 replicas of app v2.0).

2. **API Server** receives and stores in etcd.

3. **Controller Manager** runs control loops comparing desired vs actual state.

4. **Scheduler** assigns unscheduled pods to nodes based on resources, constraints.

5. **kubelet** on each node watches for assigned pods, manages container runtime.

6. **Services and networking** provided by kube-proxy and CNI plugins.

The power is in the reconciliation loop - if a pod dies, controller notices the discrepancy and creates a new one. You don't manage individual servers; you describe what you want and Kubernetes makes it happen."

### When Asked: "How do you handle configuration in Kubernetes?"

**Junior Answer:**
> "Use environment variables."

**Senior Answer:**
> "Configuration management in K8s has several layers:

1. **ConfigMaps**: Non-sensitive config. Can be mounted as files or exposed as env vars. Separate config from image.

2. **Secrets**: Sensitive data (base64 encoded, not encrypted by default). Use external secret managers (Vault, AWS Secrets Manager) with operators for real security.

3. **Environment-specific**: Different ConfigMaps/Secrets per namespace (dev/staging/prod). Kustomize or Helm for templating.

4. **Dynamic config**: ConfigMap updates can be picked up without restart (when mounted as volume). Or use a sidecar that watches for changes.

5. **Immutable ConfigMaps**: In production, use immutable ConfigMaps with version suffixes. Forces explicit deployment for config changes."

---

## 📚 Table of Contents

1. [Pods](#1-pods)
2. [Deployments](#2-deployments)
3. [Services](#3-services)
4. [Ingress](#4-ingress)
5. [ConfigMaps and Secrets](#5-configmaps-and-secrets)
6. [Storage](#6-storage)
7. [Scaling](#7-scaling)
8. [Common Pitfalls](#8-common-pitfalls)
9. [Interview Questions](#9-interview-questions)

---

## 1. Pods

```yaml
# ════════════════════════════════════════════════════════════════
# POD - BASIC EXAMPLE
# ════════════════════════════════════════════════════════════════

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    version: v1
spec:
  containers:
    - name: myapp
      image: myregistry/myapp:1.0.0
      ports:
        - containerPort: 3000
      
      # Resource management
      resources:
        requests:
          memory: "128Mi"
          cpu: "100m"
        limits:
          memory: "256Mi"
          cpu: "500m"
      
      # Health checks
      livenessProbe:
        httpGet:
          path: /health
          port: 3000
        initialDelaySeconds: 10
        periodSeconds: 10
        failureThreshold: 3
      
      readinessProbe:
        httpGet:
          path: /ready
          port: 3000
        initialDelaySeconds: 5
        periodSeconds: 5
      
      # Environment variables
      env:
        - name: NODE_ENV
          value: "production"
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database_host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database_password
      
      # Volume mounts
      volumeMounts:
        - name: config-volume
          mountPath: /app/config
        - name: data-volume
          mountPath: /app/data
  
  # Volumes
  volumes:
    - name: config-volume
      configMap:
        name: app-config
    - name: data-volume
      persistentVolumeClaim:
        claimName: app-data-pvc
  
  # Pod scheduling
  nodeSelector:
    disktype: ssd
  
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "high-memory"
      effect: "NoSchedule"


# ════════════════════════════════════════════════════════════════
# POD WITH SIDECAR PATTERN
# ════════════════════════════════════════════════════════════════

apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  containers:
    # Main application container
    - name: app
      image: myapp:1.0.0
      ports:
        - containerPort: 3000
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app
    
    # Sidecar: Log shipper
    - name: log-shipper
      image: fluent-bit:latest
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app
          readOnly: true
    
    # Sidecar: Envoy proxy
    - name: envoy
      image: envoyproxy/envoy:v1.28.0
      ports:
        - containerPort: 9901
  
  volumes:
    - name: shared-logs
      emptyDir: {}


# ════════════════════════════════════════════════════════════════
# INIT CONTAINERS
# ════════════════════════════════════════════════════════════════

apiVersion: v1
kind: Pod
metadata:
  name: app-with-init
spec:
  initContainers:
    # Wait for database to be ready
    - name: wait-for-db
      image: busybox:1.28
      command: ['sh', '-c', 'until nc -z db-service 5432; do echo waiting for db; sleep 2; done']
    
    # Run database migrations
    - name: migrate
      image: myapp:1.0.0
      command: ['npm', 'run', 'db:migrate']
      env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database_url
  
  containers:
    - name: app
      image: myapp:1.0.0
      ports:
        - containerPort: 3000
```

---

## 2. Deployments

```yaml
# ════════════════════════════════════════════════════════════════
# DEPLOYMENT - COMPLETE EXAMPLE
# ════════════════════════════════════════════════════════════════

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: production
  labels:
    app: myapp
spec:
  replicas: 3
  
  # How to find pods to manage
  selector:
    matchLabels:
      app: myapp
  
  # Update strategy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max pods above desired during update
      maxUnavailable: 0  # Zero downtime
  
  # Minimum time pod should be ready before considered available
  minReadySeconds: 10
  
  # Revision history limit
  revisionHistoryLimit: 5
  
  # Pod template
  template:
    metadata:
      labels:
        app: myapp
        version: v1.2.0
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "3000"
    spec:
      # Service account for pod
      serviceAccountName: myapp-sa
      
      # Security context
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      
      containers:
        - name: myapp
          image: myregistry/myapp:1.2.0
          imagePullPolicy: Always
          
          ports:
            - name: http
              containerPort: 3000
          
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          
          # Probes
          livenessProbe:
            httpGet:
              path: /health/live
              port: http
            initialDelaySeconds: 15
            periodSeconds: 20
            timeoutSeconds: 5
            failureThreshold: 3
          
          readinessProbe:
            httpGet:
              path: /health/ready
              port: http
            initialDelaySeconds: 5
            periodSeconds: 10
            timeoutSeconds: 3
            failureThreshold: 3
          
          # Environment from ConfigMap and Secret
          envFrom:
            - configMapRef:
                name: myapp-config
            - secretRef:
                name: myapp-secrets
          
          # Volume mounts
          volumeMounts:
            - name: config
              mountPath: /app/config
              readOnly: true
      
      # Volumes
      volumes:
        - name: config
          configMap:
            name: myapp-config
      
      # Image pull secrets
      imagePullSecrets:
        - name: registry-credentials
      
      # Pod anti-affinity (spread across nodes)
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - myapp
                topologyKey: kubernetes.io/hostname
      
      # Graceful shutdown
      terminationGracePeriodSeconds: 60
      
      # Topology spread
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: ScheduleAnyway
          labelSelector:
            matchLabels:
              app: myapp

---
# ════════════════════════════════════════════════════════════════
# POD DISRUPTION BUDGET
# ════════════════════════════════════════════════════════════════

apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2  # Or use maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
```

```bash
# ════════════════════════════════════════════════════════════════
# DEPLOYMENT COMMANDS
# ════════════════════════════════════════════════════════════════

# Apply deployment
kubectl apply -f deployment.yaml

# Check rollout status
kubectl rollout status deployment/myapp

# View rollout history
kubectl rollout history deployment/myapp

# Rollback to previous version
kubectl rollout undo deployment/myapp

# Rollback to specific revision
kubectl rollout undo deployment/myapp --to-revision=2

# Scale deployment
kubectl scale deployment/myapp --replicas=5

# Update image
kubectl set image deployment/myapp myapp=myregistry/myapp:1.3.0

# Pause/resume rollout
kubectl rollout pause deployment/myapp
kubectl rollout resume deployment/myapp
```

---

## 3. Services

```yaml
# ════════════════════════════════════════════════════════════════
# SERVICE TYPES
# ════════════════════════════════════════════════════════════════

# ClusterIP (default) - Internal cluster access only
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
    - name: http
      port: 80           # Service port
      targetPort: 3000   # Container port
      protocol: TCP

---
# NodePort - Expose on each node's IP
apiVersion: v1
kind: Service
metadata:
  name: myapp-nodeport
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 3000
      nodePort: 30080  # Port on each node (30000-32767)

---
# LoadBalancer - Cloud provider load balancer
apiVersion: v1
kind: Service
metadata:
  name: myapp-lb
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 3000

---
# Headless Service - No load balancing, returns pod IPs
apiVersion: v1
kind: Service
metadata:
  name: myapp-headless
spec:
  clusterIP: None  # Makes it headless
  selector:
    app: myapp
  ports:
    - port: 3000

---
# ExternalName - DNS CNAME to external service
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: mydb.example.com
```

---

## 4. Ingress

```yaml
# ════════════════════════════════════════════════════════════════
# INGRESS - HTTP/HTTPS ROUTING
# ════════════════════════════════════════════════════════════════

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    # Nginx ingress controller annotations
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    
    # Cert-manager for automatic TLS
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  
  # TLS configuration
  tls:
    - hosts:
        - myapp.example.com
        - api.example.com
      secretName: myapp-tls-secret
  
  # Routing rules
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
          
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
    
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80

---
# ════════════════════════════════════════════════════════════════
# CERTIFICATE ISSUER (cert-manager)
# ════════════════════════════════════════════════════════════════

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod-key
    solvers:
      - http01:
          ingress:
            class: nginx
```

---

## 5. ConfigMaps and Secrets

```yaml
# ════════════════════════════════════════════════════════════════
# CONFIGMAP
# ════════════════════════════════════════════════════════════════

apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  # Simple key-value pairs
  DATABASE_HOST: "db-service"
  DATABASE_PORT: "5432"
  LOG_LEVEL: "info"
  
  # File content
  app.properties: |
    server.port=3000
    server.timeout=30000
    cache.enabled=true
  
  nginx.conf: |
    server {
      listen 80;
      location / {
        proxy_pass http://localhost:3000;
      }
    }

---
# ════════════════════════════════════════════════════════════════
# SECRET
# ════════════════════════════════════════════════════════════════

apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
type: Opaque
stringData:  # Use stringData for plain text (auto base64 encoded)
  DATABASE_PASSWORD: "super-secret-password"
  API_KEY: "sk_live_xxx"
  
# Or use data with base64 encoded values
data:
  JWT_SECRET: c3VwZXItc2VjcmV0LWp3dC1rZXk=  # base64 encoded

---
# ════════════════════════════════════════════════════════════════
# USING CONFIGMAP AND SECRET IN POD
# ════════════════════════════════════════════════════════════════

apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
    - name: myapp
      image: myapp:1.0.0
      
      # Method 1: Individual environment variables
      env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: myapp-config
              key: DATABASE_HOST
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: DATABASE_PASSWORD
      
      # Method 2: All keys as environment variables
      envFrom:
        - configMapRef:
            name: myapp-config
        - secretRef:
            name: myapp-secrets
      
      # Method 3: Mount as files
      volumeMounts:
        - name: config-volume
          mountPath: /app/config
        - name: secret-volume
          mountPath: /app/secrets
          readOnly: true
  
  volumes:
    - name: config-volume
      configMap:
        name: myapp-config
        items:
          - key: app.properties
            path: application.properties
    - name: secret-volume
      secret:
        secretName: myapp-secrets
```

---

## 6. Storage

```yaml
# ════════════════════════════════════════════════════════════════
# PERSISTENT VOLUME AND CLAIM
# ════════════════════════════════════════════════════════════════

# StorageClass (usually provided by cloud provider)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
volumeBindingMode: WaitForFirstConsumer

---
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myapp-data
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 10Gi

---
# Using PVC in Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1  # Note: ReadWriteOnce limits to single pod
  template:
    spec:
      containers:
        - name: myapp
          volumeMounts:
            - name: data
              mountPath: /app/data
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: myapp-data

---
# ════════════════════════════════════════════════════════════════
# STATEFULSET (for stateful applications)
# ════════════════════════════════════════════════════════════════

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:15
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  
  # Each replica gets its own PVC
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: fast-ssd
        resources:
          requests:
            storage: 20Gi
```

---

## 7. Scaling

```yaml
# ════════════════════════════════════════════════════════════════
# HORIZONTAL POD AUTOSCALER
# ════════════════════════════════════════════════════════════════

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  
  minReplicas: 3
  maxReplicas: 20
  
  metrics:
    # CPU-based scaling
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    
    # Memory-based scaling
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
    
    # Custom metrics (e.g., from Prometheus)
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: 1000
  
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5 min before scale down
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
        - type: Pods
          value: 4
          periodSeconds: 15
      selectPolicy: Max

---
# ════════════════════════════════════════════════════════════════
# VERTICAL POD AUTOSCALER
# ════════════════════════════════════════════════════════════════

apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: "Auto"  # Or "Off" for recommendations only
  resourcePolicy:
    containerPolicies:
      - containerName: myapp
        minAllowed:
          cpu: 100m
          memory: 128Mi
        maxAllowed:
          cpu: 2
          memory: 4Gi
```

```bash
# Scaling commands
kubectl scale deployment myapp --replicas=5
kubectl autoscale deployment myapp --min=3 --max=10 --cpu-percent=70
kubectl get hpa
kubectl describe hpa myapp-hpa
```

---

## 8. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# KUBERNETES PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: No resource limits
# Bad - Can consume all node resources
spec:
  containers:
    - name: myapp
      image: myapp:1.0.0
      # No resources specified!

# Good - Set requests AND limits
spec:
  containers:
    - name: myapp
      image: myapp:1.0.0
      resources:
        requests:
          memory: "256Mi"
          cpu: "250m"
        limits:
          memory: "512Mi"
          cpu: "500m"

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 2: No health checks
# Bad - K8s can't detect unhealthy pods
spec:
  containers:
    - name: myapp
      image: myapp:1.0.0
      ports:
        - containerPort: 3000

# Good - Liveness AND readiness probes
spec:
  containers:
    - name: myapp
      image: myapp:1.0.0
      livenessProbe:
        httpGet:
          path: /health
          port: 3000
        initialDelaySeconds: 15
      readinessProbe:
        httpGet:
          path: /ready
          port: 3000
        initialDelaySeconds: 5

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 3: Using latest tag
# Bad - Unpredictable, not reproducible
spec:
  containers:
    - name: myapp
      image: myapp:latest

# Good - Specific version
spec:
  containers:
    - name: myapp
      image: myapp:1.2.3
      imagePullPolicy: IfNotPresent

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 4: Not handling graceful shutdown
# Bad - Connections dropped during shutdown
# (no special handling)

# Good - Handle SIGTERM, set grace period
spec:
  terminationGracePeriodSeconds: 60
  containers:
    - name: myapp
      lifecycle:
        preStop:
          exec:
            command: ["/bin/sh", "-c", "sleep 10"]  # Allow in-flight requests

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 5: Single replica for production
# Bad - No redundancy
spec:
  replicas: 1

# Good - Multiple replicas with anti-affinity
spec:
  replicas: 3
  template:
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  app: myapp
              topologyKey: kubernetes.io/hostname

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 6: Storing secrets in ConfigMap
# Bad - ConfigMaps are not encrypted
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_PASSWORD: "secret123"  # DON'T DO THIS!

# Good - Use Secrets (or better: external secret manager)
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  DATABASE_PASSWORD: "secret123"
```

---

## 9. Interview Questions

### Basic Questions

**Q: "What is a Pod?"**
> "Smallest deployable unit in Kubernetes. Contains one or more containers sharing network namespace (same IP), storage, and lifecycle. Usually one container per pod. Multi-container pods for sidecar patterns (logging, proxy)."

**Q: "Difference between Deployment and StatefulSet?"**
> "Deployment: Stateless apps, pods are interchangeable, can be scaled up/down freely. StatefulSet: Stateful apps, pods have stable network identity and persistent storage, ordered deployment/scaling. Use StatefulSet for databases, Kafka."

**Q: "What are the Service types?"**
> "ClusterIP: Internal only (default). NodePort: Exposes on node IP:port. LoadBalancer: Cloud provider LB. ExternalName: DNS alias. Headless (clusterIP: None): Returns pod IPs directly."

### Intermediate Questions

**Q: "How does rolling update work?"**
> "Deployment creates new ReplicaSet with new pods, gradually scales up new while scaling down old. Controlled by maxSurge (extra pods allowed) and maxUnavailable. Readiness probes ensure traffic only goes to ready pods. Can rollback if issues detected."

**Q: "Explain liveness vs readiness probes."**
> "Liveness: Is the container alive? If fails, kubelet restarts container. Detects deadlocks, infinite loops. Readiness: Is container ready for traffic? If fails, removed from service endpoints but not restarted. For dependencies, initialization."

**Q: "How do you handle secrets in Kubernetes?"**
> "Kubernetes Secrets are base64 encoded (not encrypted by default). Better: Enable encryption at rest, use external secret managers (Vault, AWS Secrets Manager) with operators like External Secrets. RBAC to limit access. Avoid mounting as env vars (visible in logs)."

### Advanced Questions

**Q: "How does Kubernetes networking work?"**
> "Every pod gets unique IP (no NAT). All pods can communicate directly. CNI plugins (Calico, Cilium) implement networking. Services provide stable IPs via kube-proxy (iptables/IPVS). Ingress for external HTTP routing. Network policies for segmentation."

**Q: "How do you ensure high availability?"**
> "Multiple replicas spread across nodes (podAntiAffinity). PodDisruptionBudgets prevent too many pods being down. Topology spread across zones. Proper readiness probes. Resource requests for scheduling. Autoscaling for load. Control plane HA (3+ masters)."

**Q: "How would you debug a pod that keeps crashing?"**
> "1) kubectl describe pod - events, state. 2) kubectl logs - app logs. 3) kubectl logs --previous - crashed container logs. 4) Check resource limits (OOMKilled). 5) Check probes (restart loop). 6) kubectl exec for interactive debugging. 7) Check dependencies (init containers)."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                    KUBECTL CHEAT SHEET                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  VIEWING:                                                       │
│  kubectl get pods/deploy/svc/ing -n namespace                  │
│  kubectl describe pod <name>                                   │
│  kubectl logs <pod> [-f] [--previous]                          │
│  kubectl get events --sort-by='.lastTimestamp'                 │
│                                                                 │
│  INTERACTING:                                                   │
│  kubectl exec -it <pod> -- /bin/sh                             │
│  kubectl port-forward <pod> 8080:80                            │
│  kubectl cp <pod>:/path/file ./local                           │
│                                                                 │
│  MODIFYING:                                                     │
│  kubectl apply -f manifest.yaml                                │
│  kubectl delete -f manifest.yaml                               │
│  kubectl scale deploy/<name> --replicas=5                      │
│  kubectl rollout undo deploy/<name>                            │
│                                                                 │
│  DEBUGGING:                                                     │
│  kubectl run debug --rm -it --image=busybox -- sh              │
│  kubectl top pods/nodes                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

SERVICE TYPES:
┌────────────────────────────────────────────────────────────────┐
│ ClusterIP     - Internal cluster access only (default)         │
│ NodePort      - Expose on node IP:port (30000-32767)          │
│ LoadBalancer  - Cloud provider load balancer                   │
│ ExternalName  - DNS CNAME alias                                │
│ Headless      - Direct pod access (clusterIP: None)           │
└────────────────────────────────────────────────────────────────┘

PROBES:
┌────────────────────────────────────────────────────────────────┐
│ Liveness   - Is it alive? Restart if fails                     │
│ Readiness  - Is it ready? Remove from LB if fails             │
│ Startup    - Has it started? For slow-starting apps            │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

