# CI/CD Guide

Complete guide to the CareCircle CI/CD pipeline, deployment processes, and rollback procedures.

## Overview

CareCircle uses GitHub Actions for continuous integration and deployment:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Commit    │────▶│   CI Build  │────▶│   Staging   │────▶│ Production  │
│   & Push    │     │   & Test    │     │  (develop)  │     │ (tag v*.*)  │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
```

## Workflow Files

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `ci.yml` | Push/PR to main, develop | Run tests, lint, type check, security audit |
| `pr-checks.yml` | PR opened/updated | Validate PR title, branch naming, size |
| `security.yml` | Weekly, push to main | Security scanning, dependency audit |
| `deploy-staging.yml` | Push to develop | Deploy to staging environment |
| `deploy-production.yml` | Tag v*.* | Deploy to production environment |

## CI Pipeline (`ci.yml`)

Runs on every push and pull request to `main` and `develop` branches.

### Jobs

1. **Lint & Type Check** (parallel)
   - Runs ESLint on all packages
   - Type checks shared packages

2. **Security Audit** (parallel)
   - Runs `pnpm audit`
   - Checks for critical vulnerabilities

3. **Test** (parallel)
   - Spins up PostgreSQL and Redis services
   - Runs all tests with coverage
   - Uploads coverage report

4. **Build** (after lint + test)
   - Builds all packages
   - Verifies build artifacts exist

5. **Docker Build Test** (PRs only)
   - Builds Docker images to verify Dockerfiles work
   - Uses build caching for speed

### Required Environment Variables (CI)

```yaml
# Automatically available in GitHub Actions:
GITHUB_TOKEN          # For GitHub API calls
DATABASE_URL          # Set from service container
REDIS_URL             # Set from service container

# Test secrets (set in ci.yml):
JWT_SECRET            # Test-only JWT secret
JWT_REFRESH_SECRET    # Test-only refresh secret
```

## Staging Deployment (`deploy-staging.yml`)

Automatically deploys when code is pushed to `develop` branch.

### Process

1. **Build Docker Images**
   - Builds API, Web, and Workers images
   - Tags with `staging-{SHA}` and `staging-latest`
   - Pushes to GitHub Container Registry (ghcr.io)

2. **Deploy to Server**
   - SSHs into staging server
   - Pulls latest images
   - Runs `docker-compose up -d`
   - Executes database migrations

3. **Health Check**
   - Waits 30 seconds for services to start
   - Checks API and Web health endpoints
   - Fails deployment if unhealthy

4. **Notification**
   - Sends Slack notification on success/failure

### Required Secrets (Staging)

```yaml
STAGING_HOST          # Staging server hostname/IP
STAGING_USER          # SSH username
STAGING_SSH_KEY       # SSH private key
STAGING_API_URL       # API URL for health check
STAGING_WEB_URL       # Web URL for health check
SLACK_WEBHOOK_URL     # Slack notifications
```

## Production Deployment (`deploy-production.yml`)

Triggered by version tags (e.g., `v1.0.0`) or manual dispatch.

### Process

1. **Validate Release**
   - Checks version format matches `v*.*.*`
   - Extracts version number

2. **Build Docker Images**
   - Builds multi-platform images (linux/amd64, linux/arm64)
   - Tags with version and `latest`
   - Pushes to GitHub Container Registry

3. **Deploy to Kubernetes**
   - Updates deployment image tags
   - Waits for rollout to complete
   - Times out after 5 minutes

4. **Database Migrations**
   - Executes `prisma migrate deploy`
   - Runs in API pod

5. **Health Check**
   - Verifies API and Web endpoints
   - Triggers rollback if failed

6. **GitHub Release**
   - Creates release with auto-generated notes

7. **Rollback (on failure)**
   - Automatically reverts deployments
   - Notifies team via Slack

### Required Secrets (Production)

```yaml
KUBE_CONFIG           # Kubernetes config file
PRODUCTION_API_URL    # API URL for health check
PRODUCTION_WEB_URL    # Web URL for health check
SLACK_WEBHOOK_URL     # Slack notifications
```

## Creating a Release

### For Staging

```bash
# All commits to develop automatically deploy
git checkout develop
git merge feature/your-feature
git push origin develop
```

### For Production

```bash
# 1. Ensure develop is stable
git checkout develop
git pull origin develop

# 2. Merge to main
git checkout main
git merge develop
git push origin main

# 3. Create version tag
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```

### Manual Production Deploy

If you need to deploy a specific version manually:

1. Go to Actions > "Deploy to Production"
2. Click "Run workflow"
3. Enter the version (e.g., `v1.0.0`)
4. Click "Run workflow"

## Rollback Procedures

### Automatic Rollback

Production deployments automatically rollback if:
- Health checks fail after deployment
- Kubernetes rollout times out
- Post-deployment smoke tests fail

### Manual Rollback (Kubernetes)

```bash
# List deployment history
kubectl rollout history deployment/carecircle-api -n carecircle

# Rollback to previous version
kubectl rollout undo deployment/carecircle-api -n carecircle

# Rollback to specific revision
kubectl rollout undo deployment/carecircle-api --to-revision=3 -n carecircle

# Rollback all services
kubectl rollout undo deployment/carecircle-api -n carecircle
kubectl rollout undo deployment/carecircle-web -n carecircle
kubectl rollout undo deployment/carecircle-workers -n carecircle
```

### Manual Rollback (Docker Compose)

```bash
# On staging server
cd /opt/carecircle

# Pull previous image version
docker pull ghcr.io/your-org/carecircle/api:staging-{previous-sha}

# Update docker-compose to use old image
# Edit docker-compose.staging.yml or use explicit image tags

docker-compose -f docker-compose.staging.yml up -d
```

### Database Rollback

If a migration causes issues:

1. **Do NOT rollback automatically** - migrations are forward-only in production
2. Create a new migration to fix the issue
3. If critical, use Neon's Point-in-Time Recovery (see BACKUP_PROCEDURES.md)

## Environment Variables by Stage

### Development (Local)

| Variable | Source | Example |
|----------|--------|---------|
| DATABASE_URL | `env/local.env` | `postgresql://postgres:1234@localhost:5432/carecircle` |
| REDIS_HOST | `env/local.env` | `localhost` |
| NODE_ENV | `env/base.env` | `development` |

### Testing (CI)

| Variable | Source | Value |
|----------|--------|-------|
| DATABASE_URL | GitHub Actions service | `postgresql://postgres:postgres@localhost:5432/carecircle_test` |
| REDIS_URL | GitHub Actions service | `redis://localhost:6379` |
| NODE_ENV | `env/test.env` | `test` |

### Staging

| Variable | Source | Notes |
|----------|--------|-------|
| DATABASE_URL | GitHub Secrets | Staging database |
| All secrets | GitHub Secrets | `STAGING_*` prefix |
| NODE_ENV | docker-compose | `production` |

### Production

| Variable | Source | Notes |
|----------|--------|-------|
| DATABASE_URL | Kubernetes Secret | Production database |
| All secrets | Kubernetes Secret | `carecircle-secrets` |
| NODE_ENV | Kubernetes ConfigMap | `production` |

## Troubleshooting

### CI Failures

**Test failures:**
```bash
# Run tests locally with same environment
pnpm test

# Check for flaky tests
pnpm test -- --detectOpenHandles
```

**Docker build failures:**
```bash
# Build locally to debug
docker build -f apps/api/Dockerfile -t test .

# Check build logs in GitHub Actions
```

### Deployment Failures

**Staging deployment stuck:**
```bash
# SSH into staging server
ssh user@staging-host

# Check container status
docker ps -a
docker logs carecircle-api

# Check for port conflicts
netstat -tlnp | grep 4000
```

**Production rollout failed:**
```bash
# Check pod status
kubectl get pods -n carecircle

# Check pod events
kubectl describe pod -n carecircle <pod-name>

# Check logs
kubectl logs -n carecircle -l app=carecircle-api --tail=100
```

### Common Issues

1. **"Image pull backoff"**
   - Check if image was pushed successfully
   - Verify registry credentials in Kubernetes secret

2. **"Health check failed"**
   - Check if database migrations ran
   - Verify environment variables are set

3. **"Connection refused"**
   - Services may not be ready yet
   - Check if dependencies (DB, Redis) are healthy

## Adding New Secrets

### GitHub Actions Secrets

1. Go to Repository Settings > Secrets and variables > Actions
2. Click "New repository secret"
3. Add name and value
4. Reference in workflow: `${{ secrets.SECRET_NAME }}`

### Kubernetes Secrets

```bash
# Create/update secret
kubectl create secret generic carecircle-secrets \
  --from-literal=DATABASE_URL='...' \
  --from-literal=JWT_SECRET='...' \
  -n carecircle \
  --dry-run=client -o yaml | kubectl apply -f -

# Verify secret
kubectl get secret carecircle-secrets -n carecircle -o yaml
```

## Monitoring Deployments

### GitHub Actions

- Check Actions tab for workflow runs
- Enable notifications for failed workflows
- Review workflow run summaries

### Kubernetes

```bash
# Watch deployment status
kubectl rollout status deployment/carecircle-api -n carecircle -w

# View recent events
kubectl get events -n carecircle --sort-by='.lastTimestamp' | tail -20
```

### Health Endpoints

| Service | URL | What it checks |
|---------|-----|----------------|
| API | `/health` | DB, Redis, Memory, Disk |
| API | `/health/ready` | DB, Redis only |
| API | `/health/live` | Process alive |
| Workers | `:4001/health` | Redis, DB, Worker status |
| Workers | `:4001/ready` | Ready to process jobs |

---

See also:
- [LOCAL_SETUP.md](./LOCAL_SETUP.md) - Local development setup
- [DEPLOYMENT_CHECKLIST.md](./DEPLOYMENT_CHECKLIST.md) - Pre-deployment checklist
- [INCIDENT_RESPONSE.md](../runbooks/INCIDENT_RESPONSE.md) - Incident handling

