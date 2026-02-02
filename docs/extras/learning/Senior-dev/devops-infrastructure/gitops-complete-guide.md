# 🔄 GitOps - Complete Guide

> A comprehensive guide to GitOps - ArgoCD, Flux, declarative infrastructure, Git as single source of truth, and continuous deployment patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "GitOps is an operational framework where Git is the single source of truth for declarative infrastructure and applications - changes are made via Git commits/PRs, and automated operators continuously reconcile the actual state of systems with the desired state defined in Git."

### The 7 Key Concepts (Remember These!)
```
1. DECLARATIVE       → Desired state described, not imperative steps
2. VERSIONED         → All changes tracked in Git history
3. AUTOMATED         → Operators continuously sync state
4. RECONCILIATION    → Compare desired vs actual, fix drift
5. PULL-BASED        → Operator pulls from Git (not push)
6. IMMUTABLE         → Deploy new versions, don't modify in place
7. AUDIT TRAIL       → Git provides complete change history
```

### GitOps vs Traditional CI/CD
```
┌─────────────────────────────────────────────────────────────────┐
│                 GITOPS vs TRADITIONAL CI/CD                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TRADITIONAL CI/CD (Push-based)                                │
│  ──────────────────────────────                                 │
│                                                                 │
│  Developer → Git → CI → Build → Push → Deploy to Cluster       │
│                                    ↓                           │
│                              [CI has cluster                   │
│                               credentials]                     │
│                                                                 │
│  • CI/CD pipeline pushes changes to cluster                   │
│  • Pipeline needs cluster credentials                          │
│  • State defined in pipeline, not in repo                      │
│  • Drift can occur between runs                                │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  GITOPS (Pull-based)                                           │
│  ───────────────────                                            │
│                                                                 │
│  Developer → Git Repo (Config) ←─── GitOps Operator            │
│                ↓                           ↓                   │
│          [Source of Truth]          [Reconciles state]         │
│                                            ↓                   │
│                                      Kubernetes Cluster        │
│                                                                 │
│  • Operator in cluster pulls from Git                          │
│  • No external credentials needed                              │
│  • Git is single source of truth                               │
│  • Continuous reconciliation prevents drift                    │
│  • Complete audit trail in Git history                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### GitOps Workflow
```
┌─────────────────────────────────────────────────────────────────┐
│                     GITOPS WORKFLOW                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. CHANGE REQUEST                                             │
│     Developer creates PR with config changes                   │
│     ┌─────────────────────────────────────────────────────┐   │
│     │  deployment.yaml:                                    │   │
│     │  - image: myapp:v1.0.0                              │   │
│     │  + image: myapp:v1.1.0                              │   │
│     └─────────────────────────────────────────────────────┘   │
│                              ↓                                  │
│  2. REVIEW & APPROVE                                           │
│     Code review, tests pass, PR merged                        │
│                              ↓                                  │
│  3. GIT UPDATED                                                │
│     Main branch now has new desired state                     │
│                              ↓                                  │
│  4. OPERATOR DETECTS                                           │
│     ArgoCD/Flux detects Git change                           │
│                              ↓                                  │
│  5. RECONCILIATION                                             │
│     Operator compares Git (desired) vs Cluster (actual)       │
│     Difference detected: image version                        │
│                              ↓                                  │
│  6. SYNC                                                        │
│     Operator applies changes to cluster                       │
│     Deployment updated, new pods rolling out                  │
│                              ↓                                  │
│  7. VERIFICATION                                               │
│     Health checks confirm successful deployment               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Single source of truth"** | "Git is our single source of truth for all cluster state" |
| **"Reconciliation loop"** | "ArgoCD's reconciliation loop detects and fixes drift" |
| **"Declarative"** | "We use declarative manifests - describe what, not how" |
| **"Drift detection"** | "Continuous drift detection ensures cluster matches Git" |
| **"App of Apps"** | "We use App of Apps pattern to manage multiple applications" |
| **"Hydrate"** | "Kustomize hydrates our base manifests with environment overlays" |

### Key Numbers to Remember
| Metric | Value | Notes |
|--------|-------|-------|
| Sync interval | **3 minutes** | Default ArgoCD poll |
| Reconciliation | **Continuous** | Compare desired vs actual |
| Rollback | **Seconds** | Git revert + sync |
| Webhook sync | **~30 seconds** | Faster than polling |
| History | **Infinite** | Git stores everything |

### The "Wow" Statement (Memorize This!)
> "We practice GitOps with ArgoCD managing 100+ microservices across 5 Kubernetes clusters. Our monorepo structure separates base manifests from environment-specific overlays using Kustomize. Developers submit PRs to change deployment configs - image tags, replicas, resource limits. PRs trigger validation (kubeval, policy checks with OPA). On merge, ArgoCD detects changes within 3 minutes (or instantly via webhook) and syncs to clusters. The App of Apps pattern bootstraps entire environments. Automated sync for dev/staging, manual sync with approval for production. ArgoCD provides drift detection and auto-healing - any manual kubectl changes get reverted. Rollback is a Git revert away. Complete audit trail of who changed what, when, and why."

---

## 📚 Table of Contents

1. [ArgoCD](#1-argocd)
2. [Flux](#2-flux)
3. [Repository Structure](#3-repository-structure)
4. [Kustomize](#4-kustomize)
5. [Helm with GitOps](#5-helm-with-gitops)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. ArgoCD

```yaml
# ════════════════════════════════════════════════════════════════
# ARGOCD APPLICATION
# ════════════════════════════════════════════════════════════════

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  
  source:
    repoURL: https://github.com/myorg/gitops-repo.git
    targetRevision: main
    path: apps/myapp/overlays/production
    
    # For Kustomize
    kustomize:
      images:
        - myregistry/myapp:v1.2.3
    
    # Or for Helm
    # helm:
    #   valueFiles:
    #     - values-production.yaml
    #   parameters:
    #     - name: image.tag
    #       value: v1.2.3
  
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp
  
  syncPolicy:
    automated:
      prune: true        # Delete resources not in Git
      selfHeal: true     # Revert manual changes
      allowEmpty: false  # Don't sync if repo is empty
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  
  # Health checks
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas  # Ignore if HPA manages replicas

---
# ════════════════════════════════════════════════════════════════
# ARGOCD PROJECT
# ════════════════════════════════════════════════════════════════

apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production applications
  
  # Allowed source repos
  sourceRepos:
    - 'https://github.com/myorg/*'
  
  # Allowed destination clusters/namespaces
  destinations:
    - namespace: '*'
      server: https://kubernetes.default.svc
  
  # Allowed resources
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
    - group: 'rbac.authorization.k8s.io'
      kind: ClusterRole
    - group: 'rbac.authorization.k8s.io'
      kind: ClusterRoleBinding
  
  # Denied resources
  namespaceResourceBlacklist:
    - group: ''
      kind: Secret  # Secrets managed separately
  
  # Roles
  roles:
    - name: developer
      description: Developer access
      policies:
        - p, proj:production:developer, applications, get, production/*, allow
        - p, proj:production:developer, applications, sync, production/*, allow
      groups:
        - developers

---
# ════════════════════════════════════════════════════════════════
# APP OF APPS PATTERN
# ════════════════════════════════════════════════════════════════

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: apps
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/gitops-repo.git
    targetRevision: main
    path: apps  # Contains Application manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

# apps/ directory contains:
# - myapp.yaml (Application)
# - another-app.yaml (Application)
# - monitoring.yaml (Application)
# Each Application points to its own path in repo
```

```bash
# ════════════════════════════════════════════════════════════════
# ARGOCD CLI COMMANDS
# ════════════════════════════════════════════════════════════════

# Login
argocd login argocd.example.com --username admin

# List applications
argocd app list

# Get application details
argocd app get myapp

# Sync application
argocd app sync myapp

# Sync with prune
argocd app sync myapp --prune

# Rollback to previous version
argocd app rollback myapp

# View history
argocd app history myapp

# Diff (preview changes)
argocd app diff myapp

# Set image (updates Git)
argocd app set myapp --kustomize-image myregistry/myapp:v1.2.4

# Hard refresh (clear cache)
argocd app get myapp --hard-refresh

# Delete application
argocd app delete myapp --cascade  # Also delete resources
```

---

## 2. Flux

```yaml
# ════════════════════════════════════════════════════════════════
# FLUX - GITREPOSITORY
# ════════════════════════════════════════════════════════════════

apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: gitops-repo
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/myorg/gitops-repo.git
  ref:
    branch: main
  secretRef:
    name: github-token  # For private repos

---
# ════════════════════════════════════════════════════════════════
# FLUX - KUSTOMIZATION
# ════════════════════════════════════════════════════════════════

apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: myapp
  namespace: flux-system
spec:
  interval: 10m
  targetNamespace: myapp
  sourceRef:
    kind: GitRepository
    name: gitops-repo
  path: ./apps/myapp/overlays/production
  prune: true
  timeout: 2m
  
  # Health checks
  healthChecks:
    - apiVersion: apps/v1
      kind: Deployment
      name: myapp
      namespace: myapp
  
  # Post-build variable substitution
  postBuild:
    substitute:
      CLUSTER_NAME: production
      DOMAIN: example.com
    substituteFrom:
      - kind: ConfigMap
        name: cluster-config
      - kind: Secret
        name: cluster-secrets

---
# ════════════════════════════════════════════════════════════════
# FLUX - HELMRELEASE
# ════════════════════════════════════════════════════════════════

apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: nginx-ingress
  namespace: ingress-system
spec:
  interval: 10m
  chart:
    spec:
      chart: ingress-nginx
      version: 4.x
      sourceRef:
        kind: HelmRepository
        name: ingress-nginx
        namespace: flux-system
  values:
    controller:
      replicaCount: 2
      resources:
        limits:
          cpu: 200m
          memory: 256Mi

---
# ════════════════════════════════════════════════════════════════
# FLUX - IMAGE AUTOMATION
# ════════════════════════════════════════════════════════════════

apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageRepository
metadata:
  name: myapp
  namespace: flux-system
spec:
  image: myregistry/myapp
  interval: 1m

---
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImagePolicy
metadata:
  name: myapp
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: myapp
  policy:
    semver:
      range: 1.x.x

---
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: myapp
  namespace: flux-system
spec:
  interval: 1m
  sourceRef:
    kind: GitRepository
    name: gitops-repo
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        name: flux
        email: flux@example.com
      messageTemplate: 'Update image to {{.NewTag}}'
    push:
      branch: main
  update:
    path: ./apps/myapp
    strategy: Setters
```

```bash
# ════════════════════════════════════════════════════════════════
# FLUX CLI COMMANDS
# ════════════════════════════════════════════════════════════════

# Bootstrap Flux
flux bootstrap github \
  --owner=myorg \
  --repository=gitops-repo \
  --branch=main \
  --path=clusters/production \
  --personal

# Check Flux status
flux check

# Get all Flux resources
flux get all

# Reconcile immediately
flux reconcile kustomization myapp

# View logs
flux logs --follow

# Suspend/resume
flux suspend kustomization myapp
flux resume kustomization myapp

# Export resources
flux export source git gitops-repo > git-source.yaml

# Create source
flux create source git myrepo \
  --url=https://github.com/myorg/myrepo \
  --branch=main

# Create kustomization
flux create kustomization myapp \
  --source=myrepo \
  --path=./apps/myapp \
  --prune=true \
  --interval=10m
```

---

## 3. Repository Structure

```
# ════════════════════════════════════════════════════════════════
# GITOPS REPOSITORY STRUCTURE
# ════════════════════════════════════════════════════════════════

gitops-repo/
├── README.md
├── apps/                          # Application manifests
│   ├── myapp/
│   │   ├── base/                  # Base Kustomize manifests
│   │   │   ├── kustomization.yaml
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   ├── configmap.yaml
│   │   │   └── hpa.yaml
│   │   └── overlays/              # Environment-specific
│   │       ├── development/
│   │       │   ├── kustomization.yaml
│   │       │   └── patch-replicas.yaml
│   │       ├── staging/
│   │       │   ├── kustomization.yaml
│   │       │   └── patch-resources.yaml
│   │       └── production/
│   │           ├── kustomization.yaml
│   │           ├── patch-replicas.yaml
│   │           └── patch-resources.yaml
│   │
│   ├── another-app/
│   │   ├── base/
│   │   └── overlays/
│   │
│   └── monitoring/
│       ├── prometheus/
│       └── grafana/
│
├── infrastructure/                # Cluster infrastructure
│   ├── base/
│   │   ├── ingress-nginx/
│   │   ├── cert-manager/
│   │   └── external-secrets/
│   └── overlays/
│       ├── development/
│       ├── staging/
│       └── production/
│
├── clusters/                      # Cluster-specific configs
│   ├── development/
│   │   ├── flux-system/          # Flux bootstrap
│   │   ├── apps.yaml             # App of Apps
│   │   └── infrastructure.yaml
│   ├── staging/
│   │   ├── flux-system/
│   │   ├── apps.yaml
│   │   └── infrastructure.yaml
│   └── production/
│       ├── flux-system/
│       ├── apps.yaml
│       └── infrastructure.yaml
│
└── scripts/                       # Helper scripts
    ├── validate.sh
    └── generate-secrets.sh
```

---

## 4. Kustomize

```yaml
# ════════════════════════════════════════════════════════════════
# KUSTOMIZE - BASE
# apps/myapp/base/kustomization.yaml
# ════════════════════════════════════════════════════════════════

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
  - configmap.yaml
  - hpa.yaml

commonLabels:
  app: myapp
  managed-by: kustomize

---
# apps/myapp/base/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myregistry/myapp:latest  # Will be overridden
          ports:
            - containerPort: 3000
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 200m
              memory: 256Mi

---
# ════════════════════════════════════════════════════════════════
# KUSTOMIZE - PRODUCTION OVERLAY
# apps/myapp/overlays/production/kustomization.yaml
# ════════════════════════════════════════════════════════════════

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: myapp-production

resources:
  - ../../base

# Image override
images:
  - name: myregistry/myapp
    newTag: v1.2.3

# Replicas override
replicas:
  - name: myapp
    count: 5

# Config/Secret generation
configMapGenerator:
  - name: myapp-config
    behavior: merge
    literals:
      - LOG_LEVEL=info
      - ENVIRONMENT=production

secretGenerator:
  - name: myapp-secrets
    files:
      - secrets/api-key.txt

# Patches
patches:
  # Strategic merge patch
  - path: patch-resources.yaml
  
  # JSON patch
  - target:
      kind: Deployment
      name: myapp
    patch: |-
      - op: add
        path: /spec/template/spec/containers/0/env/-
        value:
          name: NEW_ENV_VAR
          value: "production-value"

# Common annotations
commonAnnotations:
  environment: production

---
# apps/myapp/overlays/production/patch-resources.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
        - name: myapp
          resources:
            requests:
              cpu: 500m
              memory: 512Mi
            limits:
              cpu: 1000m
              memory: 1Gi
```

---

## 5. Helm with GitOps

```yaml
# ════════════════════════════════════════════════════════════════
# HELM WITH ARGOCD
# ════════════════════════════════════════════════════════════════

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-helm
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/gitops-repo.git
    targetRevision: main
    path: charts/myapp
    
    helm:
      # Values files
      valueFiles:
        - values.yaml
        - values-production.yaml
      
      # Parameter overrides
      parameters:
        - name: image.tag
          value: v1.2.3
        - name: replicaCount
          value: "5"
      
      # Skip CRDs
      skipCrds: false
      
      # Release name
      releaseName: myapp
  
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

---
# ════════════════════════════════════════════════════════════════
# HELM CHART VALUES IN GIT
# charts/myapp/values-production.yaml
# ════════════════════════════════════════════════════════════════

replicaCount: 5

image:
  repository: myregistry/myapp
  tag: v1.2.3
  pullPolicy: IfNotPresent

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 5
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70

ingress:
  enabled: true
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix

env:
  - name: NODE_ENV
    value: production
  - name: LOG_LEVEL
    value: info
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# GITOPS PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Secrets in Git
# Bad
apiVersion: v1
kind: Secret
data:
  password: c3VwZXJzZWNyZXQ=  # In Git history forever!

# Good
# Use External Secrets Operator, Sealed Secrets, or SOPS
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
spec:
  secretStoreRef:
    name: aws-secrets-manager
  target:
    name: myapp-secrets
  data:
    - secretKey: password
      remoteRef:
        key: myapp/production
        property: password

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: No environment separation
# Bad
# Single manifest for all environments
apps/myapp/deployment.yaml  # Same config everywhere!

# Good
# Use Kustomize overlays
apps/myapp/base/
apps/myapp/overlays/development/
apps/myapp/overlays/staging/
apps/myapp/overlays/production/

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Automated sync to production
# Bad
syncPolicy:
  automated:
    prune: true  # Auto-deletes resources in production!

# Good
# Manual sync for production with approval
syncPolicy:
  automated: false  # Require manual sync
# Or automated with safeguards
syncPolicy:
  automated:
    prune: false  # Don't auto-delete
    selfHeal: true  # But do revert manual changes

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: No validation in PR
# Bad
# Merge broken manifests, ArgoCD shows OutOfSync forever

# Good
# CI validates manifests before merge
# .github/workflows/validate.yaml
jobs:
  validate:
    steps:
      - run: kustomize build apps/myapp/overlays/production
      - run: kubeval --strict manifests/
      - run: conftest test manifests/ -p policies/

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Ignoring drift
# Bad
# Manual kubectl changes allowed
# No self-healing enabled
# Actual state diverges from Git

# Good
syncPolicy:
  automated:
    selfHeal: true  # Revert any manual changes
# Alert on drift
# Document that manual changes will be reverted

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Monolithic applications
# Bad
# One giant Application with everything
# Slow sync, blast radius is entire cluster

# Good
# One Application per microservice
# App of Apps pattern for organization
# Independent sync and rollback
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is GitOps?"**
> "Operational framework where Git is single source of truth for declarative infrastructure. Changes made via Git commits/PRs. Operators (ArgoCD, Flux) continuously reconcile actual state with desired state in Git. Benefits: version control, audit trail, rollback via git revert, consistent environments."

**Q: "ArgoCD vs Flux?"**
> "Both implement GitOps. ArgoCD: Great UI, Application CRD model, more batteries-included. Flux: More modular (separate controllers), GitOps Toolkit approach, better for building custom workflows. ArgoCD often preferred for ease of use, Flux for flexibility."

**Q: "What is reconciliation?"**
> "Continuously comparing desired state (Git) with actual state (cluster). When drift detected, operator applies changes to make actual match desired. Runs on interval (e.g., 3 minutes) or triggered by webhook. Ensures cluster never deviates from Git."

### Intermediate Questions

**Q: "How do you handle secrets in GitOps?"**
> "Never store plain secrets in Git. Options: 1) Sealed Secrets - encrypt with cluster key, safe to store encrypted in Git. 2) External Secrets Operator - reference secrets from Vault, AWS Secrets Manager. 3) SOPS - encrypt specific fields in YAML. 4) Vault Agent Injector. I prefer External Secrets for production."

**Q: "How do you structure a GitOps repository?"**
> "Monorepo or polyrepo. Common structure: /apps for application manifests (base + overlays per environment), /infrastructure for cluster components, /clusters for cluster-specific configs. Kustomize for environment-specific overlays. Separate directories enable granular permissions and clear ownership."

**Q: "What is App of Apps pattern?"**
> "Bootstrap pattern where one ArgoCD Application manages other Applications. Root Application points to directory containing Application manifests. Enables: managing entire environments from one place, consistent configuration, easy cluster bootstrapping. Like Terraform modules but for ArgoCD."

### Advanced Questions

**Q: "How do you implement progressive delivery with GitOps?"**
> "Combine GitOps with Argo Rollouts. Define Rollout resources (instead of Deployment) in Git. Rollout strategy (canary, blue-green) defined declaratively. ArgoCD syncs Rollout from Git, Argo Rollouts executes the progressive delivery. Analysis templates for automated promotion/rollback."

**Q: "How do you handle multi-cluster GitOps?"**
> "Options: 1) One repo, multiple cluster directories, ArgoCD per cluster. 2) Central ArgoCD managing remote clusters. 3) Cluster API + GitOps for cluster lifecycle. Use Kustomize components for shared base, cluster-specific overlays. ApplicationSets for templating across clusters."

**Q: "Design GitOps for a microservices platform."**
> "Monorepo with /apps/{service}/base and /overlays/{env}. Each service has ArgoCD Application. App of Apps bootstraps environments. Infrastructure managed separately. PR workflow: create branch, modify config, CI validates, review, merge. Auto-sync for dev/staging, manual with approval for prod. External Secrets for sensitive data. Monitoring for sync status and drift. Documented runbooks for common operations."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                     GITOPS CHECKLIST                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  REPOSITORY:                                                    │
│  □ Declarative manifests in Git                                │
│  □ Environment separation (overlays)                           │
│  □ No plain secrets committed                                  │
│  □ CI validation for manifests                                 │
│                                                                 │
│  ARGOCD/FLUX:                                                   │
│  □ Applications defined per service                            │
│  □ Health checks configured                                    │
│  □ Sync policies appropriate per env                           │
│  □ Self-heal enabled (revert manual changes)                   │
│                                                                 │
│  WORKFLOW:                                                      │
│  □ Changes via PR only                                         │
│  □ PR validation (lint, validate, policy)                      │
│  □ Review and approval process                                 │
│  □ Automated sync for non-prod                                 │
│  □ Manual/approved sync for production                         │
│                                                                 │
│  SECURITY:                                                      │
│  □ Secrets via External Secrets/Sealed Secrets                 │
│  □ RBAC for ArgoCD access                                      │
│  □ Audit logging enabled                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

ARGOCD CLI QUICK REFERENCE:
┌────────────────────────────────────────────────────────────────┐
│ argocd app list                    - List applications         │
│ argocd app sync <app>              - Sync application          │
│ argocd app diff <app>              - Preview changes           │
│ argocd app rollback <app>          - Rollback                  │
│ argocd app history <app>           - View history              │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

