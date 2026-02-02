# 🔄 CI/CD Pipelines - Complete Guide

> A comprehensive guide to CI/CD pipelines - GitHub Actions, GitLab CI, Jenkins, pipeline design patterns, testing strategies, and deployment automation.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "CI/CD (Continuous Integration/Continuous Delivery) automates the software delivery process - CI merges and tests code frequently to catch issues early, while CD automatically deploys validated code to production through a series of stages including build, test, and release."

### The 7 Key Concepts (Remember These!)
```
1. CONTINUOUS INTEGRATION  → Merge code frequently, run automated tests
2. CONTINUOUS DELIVERY     → Automate release process (manual deploy trigger)
3. CONTINUOUS DEPLOYMENT   → Fully automated deployment to production
4. PIPELINE               → Series of stages: build → test → deploy
5. ARTIFACTS              → Built outputs (binaries, containers, packages)
6. ENVIRONMENTS           → dev → staging → production
7. GATES                  → Quality checks before promotion
```

### CI/CD Pipeline Flow
```
┌─────────────────────────────────────────────────────────────────┐
│                     CI/CD PIPELINE FLOW                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CODE COMMIT                                                    │
│      │                                                         │
│      ▼                                                         │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                 CONTINUOUS INTEGRATION                    │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────┐ │  │
│  │  │  Lint   │─▶│  Build  │─▶│  Unit   │─▶│ Integration │ │  │
│  │  │         │  │         │  │  Tests  │  │   Tests     │ │  │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────────┘ │  │
│  └──────────────────────────────────────────────────────────┘  │
│      │                                                         │
│      ▼                                                         │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              CONTINUOUS DELIVERY/DEPLOYMENT               │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────┐ │  │
│  │  │ Package │─▶│  Dev    │─▶│ Staging │─▶│ Production  │ │  │
│  │  │Artifact │  │ Deploy  │  │ Deploy  │  │  Deploy     │ │  │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────────┘ │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### CI/CD Tool Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│                    CI/CD TOOL COMPARISON                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  GITHUB ACTIONS                                                 │
│  • YAML-based workflows in .github/workflows/                  │
│  • Deep GitHub integration                                     │
│  • Large marketplace of actions                                │
│  • Free for public repos                                       │
│                                                                 │
│  GITLAB CI                                                      │
│  • .gitlab-ci.yml in repo root                                 │
│  • Built into GitLab platform                                  │
│  • Strong container registry integration                       │
│  • Auto DevOps features                                        │
│                                                                 │
│  JENKINS                                                        │
│  • Jenkinsfile (declarative or scripted)                       │
│  • Self-hosted, highly customizable                            │
│  • Huge plugin ecosystem                                       │
│  • Steeper learning curve                                      │
│                                                                 │
│  CIRCLECI                                                       │
│  • .circleci/config.yml                                        │
│  • Fast execution, good caching                                │
│  • Orbs for reusable config                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Pipeline as Code"** | "We define our pipeline as code in version control for reproducibility" |
| **"Artifact promotion"** | "Artifacts are promoted through environments, not rebuilt" |
| **"Shift left"** | "We shift left by running security scans early in the pipeline" |
| **"Trunk-based development"** | "We use trunk-based development with short-lived feature branches" |
| **"Quality gates"** | "Quality gates prevent deployment if coverage drops below 80%" |
| **"GitOps"** | "We use GitOps - Git is the source of truth for deployments" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Build time target | **< 10 minutes** | Fast feedback loop |
| Test coverage | **> 80%** | Quality gate threshold |
| Deploy frequency | **Multiple/day** | DORA metric |
| Lead time | **< 1 hour** | Code to production |
| MTTR | **< 1 hour** | Mean time to recovery |
| Change failure rate | **< 15%** | DORA metric |

### The "Wow" Statement (Memorize This!)
> "We implemented a trunk-based CI/CD pipeline using GitHub Actions. On every push, we run parallel jobs for linting, unit tests, and security scans. Integration tests run against a containerized environment using Docker Compose. Artifacts are built once and promoted through environments - dev deploys automatically, staging requires passing E2E tests, and production needs manual approval. We use semantic versioning with conventional commits for automatic changelog generation. Quality gates enforce 80% coverage, zero critical vulnerabilities, and performance budgets. Our pipeline achieves sub-10-minute builds with aggressive caching, and we deploy to production 15+ times daily with a <5% change failure rate."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌─────────┐     ┌────────────────────────────────────────┐   │
│  │  Dev    │────▶│           CI/CD PLATFORM               │   │
│  │ Commit  │     │  (GitHub Actions / GitLab CI / Jenkins)│   │
│  └─────────┘     └────────────────────────────────────────┘   │
│                              │                                  │
│        ┌─────────────────────┼─────────────────────┐           │
│        ▼                     ▼                     ▼           │
│  ┌──────────┐         ┌──────────┐         ┌──────────┐       │
│  │   Lint   │         │  Build   │         │   Test   │       │
│  │  Stage   │         │  Stage   │         │  Stage   │       │
│  └──────────┘         └──────────┘         └──────────┘       │
│                              │                                  │
│                              ▼                                  │
│                    ┌──────────────────┐                        │
│                    │ Artifact Registry │                        │
│                    │ (Docker, npm, etc)│                        │
│                    └──────────────────┘                        │
│                              │                                  │
│        ┌─────────────────────┼─────────────────────┐           │
│        ▼                     ▼                     ▼           │
│  ┌──────────┐         ┌──────────┐         ┌──────────┐       │
│  │   Dev    │ ──────▶ │ Staging  │ ──────▶ │Production│       │
│  │   Env    │         │   Env    │         │   Env    │       │
│  └──────────┘         └──────────┘         └──────────┘       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is CI/CD?"**
> "CI is continuously merging and testing code to catch issues early. CD is automating the release process. Continuous Delivery = automated pipeline with manual deploy trigger. Continuous Deployment = fully automated to production."

**Q: "What makes a good CI/CD pipeline?"**
> "Fast feedback (<10 min builds), reliable tests, artifact promotion (build once, deploy many), quality gates, environment parity, rollback capability, and security scanning integrated early."

**Q: "GitHub Actions vs Jenkins?"**
> "Actions: YAML-based, hosted, deep GitHub integration, marketplace. Jenkins: Self-hosted, highly customizable, Groovy scripting, huge plugin ecosystem. Actions for simplicity and GitHub-centric workflows, Jenkins for complex enterprise needs."

**Q: "How do you handle secrets in pipelines?"**
> "Never in code. Use platform secrets (GitHub Secrets, GitLab CI variables). For more complex needs: HashiCorp Vault, AWS Secrets Manager. Mask in logs, rotate regularly, least-privilege access."

**Q: "What are DORA metrics?"**
> "DevOps Research metrics: Deployment Frequency, Lead Time for Changes, Change Failure Rate, Mean Time to Recovery. Elite teams deploy multiple times daily with <15% failure rate and <1 hour recovery."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How do you design a CI/CD pipeline?"

**Junior Answer:**
> "Run tests and deploy when they pass."

**Senior Answer:**
> "Pipeline design considers several factors:

1. **Speed vs Thoroughness**: Parallelize independent jobs. Run fast tests first (lint, unit), slow tests later (E2E, performance). Use caching aggressively.

2. **Build Once, Deploy Many**: Create immutable artifacts once, promote through environments. Never rebuild for different environments.

3. **Quality Gates**: Enforce coverage thresholds, security scans, performance budgets. Fail fast, fail early.

4. **Environment Progression**: dev → staging → production with increasing rigor. Staging should mirror production.

5. **Deployment Strategy**: Blue-green or canary for zero-downtime. Automatic rollback on health check failures.

6. **Observability**: Pipeline metrics, deployment tracking, correlation IDs from commit to production.

The goal is fast, reliable feedback with confidence that what's deployed is what was tested."

### When Asked: "How do you handle flaky tests in CI?"

**Junior Answer:**
> "Just rerun the pipeline."

**Senior Answer:**
> "Flaky tests are a serious problem - they erode trust in the pipeline:

1. **Detection**: Track test pass/fail rates. Flag tests that fail intermittently.

2. **Quarantine**: Move flaky tests to a separate job that doesn't block deployment but still reports.

3. **Root Cause**: Common causes are timing issues, shared state, external dependencies. Add proper waits, isolate tests, mock external services.

4. **Retry Strategy**: Implement test-level retry (not job-level) with limit of 2-3. Require consistent passes.

5. **Culture**: Treat flaky test creation as a bug. Don't merge code that introduces flakiness.

Never just ignore or blindly rerun - that's how you end up with an unreliable pipeline that people stop trusting."

---

## 📚 Table of Contents

1. [GitHub Actions](#1-github-actions)
2. [GitLab CI](#2-gitlab-ci)
3. [Jenkins](#3-jenkins)
4. [Pipeline Design Patterns](#4-pipeline-design-patterns)
5. [Testing in CI](#5-testing-in-ci)
6. [Security in CI/CD](#6-security-in-cicd)
7. [Deployment Strategies](#7-deployment-strategies)
8. [Common Pitfalls](#8-common-pitfalls)
9. [Interview Questions](#9-interview-questions)

---

## 1. GitHub Actions

```yaml
# ════════════════════════════════════════════════════════════════
# GITHUB ACTIONS - COMPLETE CI/CD WORKFLOW
# .github/workflows/ci-cd.yml
# ════════════════════════════════════════════════════════════════

name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  workflow_dispatch:  # Manual trigger

# Environment variables available to all jobs
env:
  NODE_VERSION: '20'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

# Concurrency: Cancel in-progress runs for same branch
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # ══════════════════════════════════════════════════════════════
  # LINT & TYPE CHECK
  # ══════════════════════════════════════════════════════════════
  lint:
    name: Lint & Type Check
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Run TypeScript check
        run: npm run type-check

  # ══════════════════════════════════════════════════════════════
  # UNIT TESTS
  # ══════════════════════════════════════════════════════════════
  test-unit:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm run test:unit -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true

  # ══════════════════════════════════════════════════════════════
  # INTEGRATION TESTS
  # ══════════════════════════════════════════════════════════════
  test-integration:
    name: Integration Tests
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run migrations
        run: npm run db:migrate
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/testdb

      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379

  # ══════════════════════════════════════════════════════════════
  # SECURITY SCAN
  # ══════════════════════════════════════════════════════════════
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'

  # ══════════════════════════════════════════════════════════════
  # BUILD
  # ══════════════════════════════════════════════════════════════
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint, test-unit]
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/
          retention-days: 7

      # Docker build
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha,prefix=
            type=ref,event=branch
            type=semver,pattern={{version}}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ══════════════════════════════════════════════════════════════
  # E2E TESTS
  # ══════════════════════════════════════════════════════════════
  test-e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: [build]
    
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: build
          path: dist/

      - name: Run E2E tests
        run: npm run test:e2e

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/

  # ══════════════════════════════════════════════════════════════
  # DEPLOY TO STAGING
  # ══════════════════════════════════════════════════════════════
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: [build, test-integration, security]
    if: github.ref == 'refs/heads/main'
    environment:
      name: staging
      url: https://staging.example.com
    
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to staging
        run: |
          echo "Deploying to staging..."
          # Example: kubectl, AWS CLI, or other deployment tool
        env:
          KUBE_CONFIG: ${{ secrets.KUBE_CONFIG_STAGING }}

      - name: Run smoke tests
        run: |
          curl -f https://staging.example.com/health || exit 1

      - name: Notify Slack
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Deployed to staging: ${{ github.sha }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

  # ══════════════════════════════════════════════════════════════
  # DEPLOY TO PRODUCTION
  # ══════════════════════════════════════════════════════════════
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: [deploy-staging, test-e2e]
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://example.com
    
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to production
        run: |
          echo "Deploying to production..."
        env:
          KUBE_CONFIG: ${{ secrets.KUBE_CONFIG_PRODUCTION }}

      - name: Verify deployment
        run: |
          curl -f https://example.com/health || exit 1

      - name: Create GitHub Release
        if: startsWith(github.ref, 'refs/tags/v')
        uses: softprops/action-gh-release@v1
        with:
          generate_release_notes: true

# ════════════════════════════════════════════════════════════════
# REUSABLE WORKFLOW
# .github/workflows/reusable-deploy.yml
# ════════════════════════════════════════════════════════════════
```

```yaml
# Reusable workflow for deployment
name: Reusable Deploy

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      image-tag:
        required: true
        type: string
    secrets:
      KUBE_CONFIG:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - name: Deploy
        run: |
          echo "Deploying ${{ inputs.image-tag }} to ${{ inputs.environment }}"
```

---

## 2. GitLab CI

```yaml
# ════════════════════════════════════════════════════════════════
# GITLAB CI - COMPLETE PIPELINE
# .gitlab-ci.yml
# ════════════════════════════════════════════════════════════════

# Default settings
default:
  image: node:20-alpine
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
      - .npm/

# Variables
variables:
  DOCKER_HOST: tcp://docker:2376
  DOCKER_TLS_CERTDIR: "/certs"
  DOCKER_TLS_VERIFY: 1
  DOCKER_CERT_PATH: "$DOCKER_TLS_CERTDIR/client"
  REGISTRY: registry.gitlab.com
  IMAGE_NAME: $CI_REGISTRY_IMAGE

# Stages
stages:
  - prepare
  - test
  - build
  - deploy-staging
  - deploy-production

# ══════════════════════════════════════════════════════════════
# PREPARE STAGE
# ══════════════════════════════════════════════════════════════
install:
  stage: prepare
  script:
    - npm ci --cache .npm --prefer-offline
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

# ══════════════════════════════════════════════════════════════
# TEST STAGE
# ══════════════════════════════════════════════════════════════
lint:
  stage: test
  needs: [install]
  script:
    - npm run lint
    - npm run type-check

unit-tests:
  stage: test
  needs: [install]
  script:
    - npm run test:unit -- --coverage
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
      junit: junit.xml

integration-tests:
  stage: test
  needs: [install]
  services:
    - name: postgres:15
      alias: db
    - name: redis:7
      alias: cache
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: test
    POSTGRES_PASSWORD: test
    DATABASE_URL: postgresql://test:test@db:5432/testdb
    REDIS_URL: redis://cache:6379
  script:
    - npm run db:migrate
    - npm run test:integration

security-scan:
  stage: test
  needs: [install]
  script:
    - npm audit --audit-level=high
  allow_failure: true

sast:
  stage: test
  include:
    - template: Security/SAST.gitlab-ci.yml

# ══════════════════════════════════════════════════════════════
# BUILD STAGE
# ══════════════════════════════════════════════════════════════
build-app:
  stage: build
  needs: [lint, unit-tests]
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

build-docker:
  stage: build
  needs: [build-app]
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHA -t $IMAGE_NAME:latest .
    - docker push $IMAGE_NAME:$CI_COMMIT_SHA
    - docker push $IMAGE_NAME:latest
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

# ══════════════════════════════════════════════════════════════
# DEPLOY STAGING
# ══════════════════════════════════════════════════════════════
deploy-staging:
  stage: deploy-staging
  needs: [build-docker, integration-tests]
  image: bitnami/kubectl:latest
  environment:
    name: staging
    url: https://staging.example.com
  script:
    - kubectl config use-context staging
    - kubectl set image deployment/app app=$IMAGE_NAME:$CI_COMMIT_SHA
    - kubectl rollout status deployment/app
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

smoke-test-staging:
  stage: deploy-staging
  needs: [deploy-staging]
  script:
    - apk add --no-cache curl
    - |
      for i in $(seq 1 30); do
        curl -sf https://staging.example.com/health && exit 0
        sleep 10
      done
      exit 1
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

# ══════════════════════════════════════════════════════════════
# DEPLOY PRODUCTION
# ══════════════════════════════════════════════════════════════
deploy-production:
  stage: deploy-production
  needs: [smoke-test-staging]
  image: bitnami/kubectl:latest
  environment:
    name: production
    url: https://example.com
  script:
    - kubectl config use-context production
    - kubectl set image deployment/app app=$IMAGE_NAME:$CI_COMMIT_SHA
    - kubectl rollout status deployment/app
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
  allow_failure: false

# ══════════════════════════════════════════════════════════════
# ROLLBACK (Manual)
# ══════════════════════════════════════════════════════════════
rollback-production:
  stage: deploy-production
  image: bitnami/kubectl:latest
  environment:
    name: production
  script:
    - kubectl config use-context production
    - kubectl rollout undo deployment/app
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
  allow_failure: true
```

---

## 3. Jenkins

```groovy
// ════════════════════════════════════════════════════════════════
// JENKINSFILE - DECLARATIVE PIPELINE
// Jenkinsfile
// ════════════════════════════════════════════════════════════════

pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: node
                image: node:20
                command: [cat]
                tty: true
              - name: docker
                image: docker:24-dind
                securityContext:
                  privileged: true
            '''
        }
    }

    environment {
        REGISTRY = 'registry.example.com'
        IMAGE_NAME = 'my-app'
        DOCKER_CREDENTIALS = credentials('docker-registry')
        SLACK_WEBHOOK = credentials('slack-webhook')
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    stages {
        // ══════════════════════════════════════════════════════════
        // CHECKOUT & INSTALL
        // ══════════════════════════════════════════════════════════
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                container('node') {
                    sh 'npm ci'
                }
            }
        }

        // ══════════════════════════════════════════════════════════
        // PARALLEL TESTING
        // ══════════════════════════════════════════════════════════
        stage('Test') {
            parallel {
                stage('Lint') {
                    steps {
                        container('node') {
                            sh 'npm run lint'
                        }
                    }
                }
                
                stage('Unit Tests') {
                    steps {
                        container('node') {
                            sh 'npm run test:unit -- --coverage'
                        }
                    }
                    post {
                        always {
                            publishHTML([
                                reportDir: 'coverage/lcov-report',
                                reportFiles: 'index.html',
                                reportName: 'Coverage Report'
                            ])
                        }
                    }
                }
                
                stage('Security Scan') {
                    steps {
                        container('node') {
                            sh 'npm audit --audit-level=high'
                        }
                    }
                }
            }
        }

        // ══════════════════════════════════════════════════════════
        // BUILD
        // ══════════════════════════════════════════════════════════
        stage('Build Application') {
            steps {
                container('node') {
                    sh 'npm run build'
                }
            }
        }

        stage('Build Docker Image') {
            when {
                branch 'main'
            }
            steps {
                container('docker') {
                    script {
                        def imageTag = "${REGISTRY}/${IMAGE_NAME}:${GIT_COMMIT[0..7]}"
                        sh """
                            docker build -t ${imageTag} .
                            docker login -u ${DOCKER_CREDENTIALS_USR} -p ${DOCKER_CREDENTIALS_PSW} ${REGISTRY}
                            docker push ${imageTag}
                        """
                        env.IMAGE_TAG = imageTag
                    }
                }
            }
        }

        // ══════════════════════════════════════════════════════════
        // DEPLOY STAGING
        // ══════════════════════════════════════════════════════════
        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                script {
                    withKubeConfig([credentialsId: 'kubeconfig-staging']) {
                        sh """
                            kubectl set image deployment/my-app app=${env.IMAGE_TAG}
                            kubectl rollout status deployment/my-app --timeout=300s
                        """
                    }
                }
            }
        }

        stage('Smoke Tests') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                    for i in $(seq 1 30); do
                        curl -sf https://staging.example.com/health && exit 0
                        sleep 10
                    done
                    exit 1
                '''
            }
        }

        // ══════════════════════════════════════════════════════════
        // DEPLOY PRODUCTION
        // ══════════════════════════════════════════════════════════
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            input {
                message 'Deploy to production?'
                ok 'Deploy'
                submitter 'admin,release-manager'
            }
            steps {
                script {
                    withKubeConfig([credentialsId: 'kubeconfig-production']) {
                        sh """
                            kubectl set image deployment/my-app app=${env.IMAGE_TAG}
                            kubectl rollout status deployment/my-app --timeout=300s
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            slackSend(
                channel: '#deployments',
                color: 'good',
                message: "✅ Build succeeded: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                channel: '#deployments',
                color: 'danger',
                message: "❌ Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
        always {
            cleanWs()
        }
    }
}

// ════════════════════════════════════════════════════════════════
// SHARED LIBRARY EXAMPLE
// vars/standardPipeline.groovy
// ════════════════════════════════════════════════════════════════
```

```groovy
// Shared library for reusable pipeline
def call(Map config = [:]) {
    pipeline {
        agent any
        
        stages {
            stage('Build') {
                steps {
                    script {
                        if (config.buildCommand) {
                            sh config.buildCommand
                        } else {
                            sh 'npm run build'
                        }
                    }
                }
            }
            
            stage('Test') {
                steps {
                    sh 'npm test'
                }
            }
            
            stage('Deploy') {
                when {
                    expression { config.deploy == true }
                }
                steps {
                    sh "deploy.sh ${config.environment}"
                }
            }
        }
    }
}

// Usage in Jenkinsfile:
// @Library('my-shared-library') _
// standardPipeline(
//     buildCommand: 'npm run build:prod',
//     deploy: true,
//     environment: 'staging'
// )
```

---

## 4. Pipeline Design Patterns

```yaml
# ════════════════════════════════════════════════════════════════
# PIPELINE DESIGN PATTERNS
# ════════════════════════════════════════════════════════════════

# PATTERN 1: Fan-out/Fan-in (Parallel execution)
# Run independent jobs in parallel, then converge

name: Fan-out Fan-in

jobs:
  # Fan-out: Parallel jobs
  lint:
    runs-on: ubuntu-latest
    steps:
      - run: npm run lint

  test-unit:
    runs-on: ubuntu-latest
    steps:
      - run: npm run test:unit

  test-integration:
    runs-on: ubuntu-latest
    steps:
      - run: npm run test:integration

  security:
    runs-on: ubuntu-latest
    steps:
      - run: npm audit

  # Fan-in: Wait for all parallel jobs
  build:
    needs: [lint, test-unit, test-integration, security]
    runs-on: ubuntu-latest
    steps:
      - run: npm run build

# ════════════════════════════════════════════════════════════════
# PATTERN 2: Matrix builds (Test across configurations)
# ════════════════════════════════════════════════════════════════

name: Matrix Build

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [18, 20, 22]
        exclude:
          - os: windows-latest
            node: 18
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm test

# ════════════════════════════════════════════════════════════════
# PATTERN 3: Conditional deployment
# ════════════════════════════════════════════════════════════════

name: Conditional Deploy

on:
  push:
    branches: [main, develop]
    tags: ['v*']

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to dev
        if: github.ref == 'refs/heads/develop'
        run: ./deploy.sh dev

      - name: Deploy to staging
        if: github.ref == 'refs/heads/main'
        run: ./deploy.sh staging

      - name: Deploy to production
        if: startsWith(github.ref, 'refs/tags/v')
        run: ./deploy.sh production

# ════════════════════════════════════════════════════════════════
# PATTERN 4: Artifact promotion
# Build once, promote through environments
# ════════════════════════════════════════════════════════════════

name: Artifact Promotion

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      artifact-id: ${{ steps.upload.outputs.artifact-id }}
    steps:
      - run: npm run build
      - id: upload
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ github.sha }}
          path: dist/

  deploy-staging:
    needs: build
    environment: staging
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-${{ github.sha }}
      - run: ./deploy.sh staging

  deploy-production:
    needs: deploy-staging
    environment: production
    runs-on: ubuntu-latest
    steps:
      # Same artifact, no rebuild!
      - uses: actions/download-artifact@v4
        with:
          name: build-${{ github.sha }}
      - run: ./deploy.sh production

# ════════════════════════════════════════════════════════════════
# PATTERN 5: Monorepo pipeline
# Only build/test affected packages
# ════════════════════════════════════════════════════════════════

name: Monorepo CI

on:
  push:
    paths:
      - 'packages/**'
      - 'package.json'

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      packages: ${{ steps.changes.outputs.packages }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - id: changes
        run: |
          # Detect which packages changed
          CHANGED=$(git diff --name-only ${{ github.event.before }} ${{ github.sha }} | grep '^packages/' | cut -d'/' -f2 | sort -u | jq -R -s -c 'split("\n")[:-1]')
          echo "packages=$CHANGED" >> $GITHUB_OUTPUT

  build-packages:
    needs: detect-changes
    if: needs.detect-changes.outputs.packages != '[]'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        package: ${{ fromJson(needs.detect-changes.outputs.packages) }}
    steps:
      - uses: actions/checkout@v4
      - run: npm run build --workspace=packages/${{ matrix.package }}
      - run: npm test --workspace=packages/${{ matrix.package }}
```

---

## 5. Testing in CI

```yaml
# ════════════════════════════════════════════════════════════════
# TESTING STRATEGIES IN CI
# ════════════════════════════════════════════════════════════════

name: Comprehensive Testing

jobs:
  # ══════════════════════════════════════════════════════════════
  # UNIT TESTS - Fast, isolated
  # ══════════════════════════════════════════════════════════════
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:unit -- --coverage --maxWorkers=4
      
      # Coverage enforcement
      - name: Check coverage threshold
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% is below 80% threshold"
            exit 1
          fi

  # ══════════════════════════════════════════════════════════════
  # INTEGRATION TESTS - With real dependencies
  # ══════════════════════════════════════════════════════════════
  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/postgres

  # ══════════════════════════════════════════════════════════════
  # E2E TESTS - Full application tests
  # ══════════════════════════════════════════════════════════════
  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Install Playwright
        run: npx playwright install --with-deps
        
      - name: Build application
        run: npm run build
        
      - name: Start server
        run: npm run start &
        env:
          PORT: 3000
          
      - name: Wait for server
        run: npx wait-on http://localhost:3000
        
      - name: Run E2E tests
        run: npm run test:e2e
        
      - name: Upload test artifacts
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-traces
          path: test-results/

  # ══════════════════════════════════════════════════════════════
  # VISUAL REGRESSION TESTS
  # ══════════════════════════════════════════════════════════════
  visual-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # For comparing with base branch
          
      - run: npm ci
      
      - name: Run visual tests
        run: npm run test:visual
        env:
          PERCY_TOKEN: ${{ secrets.PERCY_TOKEN }}
          
      - name: Upload visual diff
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: visual-diff
          path: .visual-regression/

  # ══════════════════════════════════════════════════════════════
  # PERFORMANCE TESTS
  # ══════════════════════════════════════════════════════════════
  performance-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v10
        with:
          configPath: './lighthouserc.json'
          uploadArtifacts: true
          temporaryPublicStorage: true
          
      - name: Performance budget check
        run: |
          # Fail if performance budget exceeded
          npm run test:performance -- --budget
```

---

## 6. Security in CI/CD

```yaml
# ════════════════════════════════════════════════════════════════
# SECURITY SCANNING IN CI/CD
# ════════════════════════════════════════════════════════════════

name: Security Pipeline

on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 6 * * *'  # Daily at 6 AM

jobs:
  # ══════════════════════════════════════════════════════════════
  # DEPENDENCY SCANNING
  # ══════════════════════════════════════════════════════════════
  dependency-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: NPM Audit
        run: npm audit --audit-level=moderate
        continue-on-error: true
        
      - name: Snyk Dependency Scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
          
      - name: OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          path: '.'
          format: 'HTML'
          
      - name: Upload report
        uses: actions/upload-artifact@v4
        with:
          name: dependency-check-report
          path: reports/

  # ══════════════════════════════════════════════════════════════
  # SAST - Static Application Security Testing
  # ══════════════════════════════════════════════════════════════
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/owasp-top-ten
            
      - name: CodeQL Analysis
        uses: github/codeql-action/init@v2
        with:
          languages: javascript, typescript
          
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2

  # ══════════════════════════════════════════════════════════════
  # SECRET SCANNING
  # ══════════════════════════════════════════════════════════════
  secrets-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          
      - name: Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          
      - name: TruffleHog
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD

  # ══════════════════════════════════════════════════════════════
  # CONTAINER SCANNING
  # ══════════════════════════════════════════════════════════════
  container-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .
        
      - name: Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  # ══════════════════════════════════════════════════════════════
  # DAST - Dynamic Application Security Testing
  # ══════════════════════════════════════════════════════════════
  dast:
    runs-on: ubuntu-latest
    needs: [build]
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy test environment
        run: docker-compose up -d
        
      - name: Wait for app
        run: npx wait-on http://localhost:3000
        
      - name: ZAP Scan
        uses: zaproxy/action-full-scan@v0.7.0
        with:
          target: 'http://localhost:3000'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'
```

---

## 7. Deployment Strategies

```yaml
# ════════════════════════════════════════════════════════════════
# DEPLOYMENT STRATEGIES
# ════════════════════════════════════════════════════════════════

# STRATEGY 1: Rolling Update
# Gradually replace old instances with new ones

name: Rolling Deployment

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Rolling update
        run: |
          kubectl set image deployment/myapp \
            app=${{ env.IMAGE }}:${{ github.sha }}
          
          # Wait for rollout to complete
          kubectl rollout status deployment/myapp --timeout=300s
          
      - name: Verify deployment
        run: |
          # Check new pods are healthy
          kubectl get pods -l app=myapp
          
          # Run health check
          curl -f https://example.com/health

# ════════════════════════════════════════════════════════════════
# STRATEGY 2: Blue-Green Deployment
# Switch traffic between two identical environments

name: Blue-Green Deployment

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Determine current color
        id: current
        run: |
          CURRENT=$(kubectl get svc myapp -o jsonpath='{.spec.selector.version}')
          if [ "$CURRENT" = "blue" ]; then
            echo "new=green" >> $GITHUB_OUTPUT
          else
            echo "new=blue" >> $GITHUB_OUTPUT
          fi
          
      - name: Deploy to inactive color
        run: |
          kubectl set image deployment/myapp-${{ steps.current.outputs.new }} \
            app=${{ env.IMAGE }}:${{ github.sha }}
          kubectl rollout status deployment/myapp-${{ steps.current.outputs.new }}
          
      - name: Test new deployment
        run: |
          # Test the new deployment directly
          NEW_IP=$(kubectl get svc myapp-${{ steps.current.outputs.new }} -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
          curl -f http://$NEW_IP/health
          
      - name: Switch traffic
        run: |
          # Update service selector to point to new color
          kubectl patch svc myapp -p \
            '{"spec":{"selector":{"version":"${{ steps.current.outputs.new }}"}}}'
          
      - name: Verify switch
        run: |
          sleep 10
          curl -f https://example.com/health

# ════════════════════════════════════════════════════════════════
# STRATEGY 3: Canary Deployment
# Gradually shift traffic to new version

name: Canary Deployment

jobs:
  deploy-canary:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy canary (10% traffic)
        run: |
          # Deploy new version as canary
          kubectl apply -f - <<EOF
          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: myapp-canary
          spec:
            replicas: 1
            selector:
              matchLabels:
                app: myapp
                version: canary
            template:
              metadata:
                labels:
                  app: myapp
                  version: canary
              spec:
                containers:
                - name: app
                  image: ${{ env.IMAGE }}:${{ github.sha }}
          EOF
          
      - name: Configure traffic split (10% canary)
        run: |
          # Using Istio VirtualService
          kubectl apply -f - <<EOF
          apiVersion: networking.istio.io/v1beta1
          kind: VirtualService
          metadata:
            name: myapp
          spec:
            hosts:
            - myapp
            http:
            - route:
              - destination:
                  host: myapp
                  subset: stable
                weight: 90
              - destination:
                  host: myapp
                  subset: canary
                weight: 10
          EOF
          
      - name: Monitor canary (10 minutes)
        run: |
          for i in $(seq 1 10); do
            # Check error rate
            ERROR_RATE=$(curl -s http://prometheus/api/v1/query?query=rate(http_errors_total{version="canary"}[1m]) | jq '.data.result[0].value[1]')
            if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
              echo "Error rate too high, rolling back"
              exit 1
            fi
            sleep 60
          done
          
  promote-canary:
    needs: deploy-canary
    runs-on: ubuntu-latest
    steps:
      - name: Promote canary to 50%
        run: |
          # Increase canary traffic
          # ... similar to above with weight: 50

      - name: Full promotion
        run: |
          # Update stable deployment
          kubectl set image deployment/myapp-stable \
            app=${{ env.IMAGE }}:${{ github.sha }}
          
          # Remove canary
          kubectl delete deployment myapp-canary
          
          # Reset traffic to 100% stable
          kubectl apply -f virtualservice-stable-only.yaml
```

---

## 8. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# CI/CD PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Rebuilding for each environment
# Problem: Different builds may have different behavior

# Bad
jobs:
  deploy-staging:
    steps:
      - run: npm run build  # Build for staging
      - run: deploy-staging.sh
      
  deploy-production:
    steps:
      - run: npm run build  # Rebuild for production - DIFFERENT ARTIFACT!
      - run: deploy-production.sh

# Good - Build once, deploy many
jobs:
  build:
    steps:
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-${{ github.sha }}
          path: dist/
          
  deploy-staging:
    needs: build
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-${{ github.sha }}
      - run: deploy-staging.sh
      
  deploy-production:
    needs: deploy-staging
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-${{ github.sha }}  # SAME ARTIFACT
      - run: deploy-production.sh

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 2: Secrets in code or logs
# ════════════════════════════════════════════════════════════════

# Bad
steps:
  - run: |
      echo "Deploying with key: ${{ secrets.API_KEY }}"  # Logged!
      curl -H "Authorization: ${{ secrets.API_KEY }}" https://api.example.com

# Good
steps:
  - run: curl -H "Authorization: $API_KEY" https://api.example.com
    env:
      API_KEY: ${{ secrets.API_KEY }}  # Masked in logs

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 3: No caching
# Problem: Slow builds
# ════════════════════════════════════════════════════════════════

# Bad
steps:
  - run: npm install  # Downloads every time

# Good
steps:
  - uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: 'npm'  # Caches node_modules
  - run: npm ci

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 4: Flaky tests blocking deployment
# ════════════════════════════════════════════════════════════════

# Bad
jobs:
  test:
    steps:
      - run: npm test  # Flaky test fails randomly, blocks everything

# Good - Quarantine flaky tests
jobs:
  test-stable:
    steps:
      - run: npm test -- --testPathIgnorePatterns=flaky
      
  test-flaky:
    steps:
      - run: npm test -- --testPathPattern=flaky
    continue-on-error: true  # Don't block deployment
    # But still report results for monitoring

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 5: No rollback plan
# ════════════════════════════════════════════════════════════════

# Bad - No easy way to rollback
steps:
  - run: kubectl apply -f deployment.yaml

# Good - Keep previous version, easy rollback
steps:
  - name: Deploy with rollback capability
    run: |
      # Record current version for rollback
      PREVIOUS=$(kubectl get deployment myapp -o jsonpath='{.spec.template.spec.containers[0].image}')
      echo "PREVIOUS_IMAGE=$PREVIOUS" >> $GITHUB_ENV
      
      # Deploy new version
      kubectl set image deployment/myapp app=$NEW_IMAGE
      
      # Wait and verify
      if ! kubectl rollout status deployment/myapp --timeout=300s; then
        echo "Deployment failed, rolling back"
        kubectl set image deployment/myapp app=$PREVIOUS
        exit 1
      fi

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 6: Deploying on every commit to main
# ════════════════════════════════════════════════════════════════

# Bad - Risky for production
on:
  push:
    branches: [main]
jobs:
  deploy-production:  # Auto-deploy to prod on every push!

# Good - Manual approval for production
jobs:
  deploy-staging:
    # Auto-deploy to staging
    
  deploy-production:
    needs: deploy-staging
    environment: production  # Requires approval
    # Or use tags for releases
    if: startsWith(github.ref, 'refs/tags/v')
```

---

## 9. Interview Questions

### Basic Questions

**Q: "What is CI/CD?"**
> "CI (Continuous Integration) is automatically building and testing code on every commit to catch issues early. CD (Continuous Delivery) automates the release pipeline - code that passes CI can be deployed at the push of a button. Continuous Deployment goes further: every passing commit is automatically deployed to production."

**Q: "What are the stages of a typical CI/CD pipeline?"**
> "1) Source - Code commit triggers pipeline. 2) Build - Compile, transpile, bundle. 3) Test - Unit, integration, E2E tests. 4) Security - Vulnerability scanning, SAST. 5) Package - Create deployable artifact. 6) Deploy - Staging then production. 7) Verify - Health checks, smoke tests."

**Q: "GitHub Actions vs Jenkins?"**
> "GitHub Actions: YAML-based, hosted by GitHub, deep repo integration, marketplace of actions, great for GitHub-centric workflows. Jenkins: Self-hosted, highly customizable with Groovy, massive plugin ecosystem, good for complex enterprise pipelines. Actions is simpler to start, Jenkins offers more control."

### Intermediate Questions

**Q: "How do you handle secrets in CI/CD?"**
> "Never commit secrets. Use platform secret stores (GitHub Secrets, GitLab CI variables). For complex needs, use Vault or cloud secret managers. Mask secrets in logs. Rotate regularly. Use environment-specific secrets. Service accounts with least privilege."

**Q: "What is artifact promotion?"**
> "Building the artifact once and promoting it through environments (dev → staging → prod) rather than rebuilding. Ensures what you tested is exactly what you deploy. Use immutable artifacts with version tags. Prevents 'works on my machine' issues."

**Q: "How do you handle flaky tests?"**
> "Track flakiness with test analytics. Quarantine flaky tests to a separate job that doesn't block. Fix root causes: timing issues, shared state, external dependencies. Implement test-level retries with limits. Create a culture where introducing flakiness is treated as a bug."

### Advanced Questions

**Q: "How would you design a CI/CD pipeline for microservices?"**
> "Monorepo: Detect changed services, build/test only affected. Multi-repo: Each service has its own pipeline. Shared concerns: Common base images, shared CI templates. Testing: Service-level tests, contract tests between services. Deployment: Independent deployability per service, feature flags for coordination."

**Q: "How do you achieve zero-downtime deployments?"**
> "Strategies: Rolling updates (gradual replacement), blue-green (switch traffic between environments), canary (gradual traffic shift). Requirements: Health checks, graceful shutdown, connection draining, database backward compatibility, feature flags for breaking changes."

**Q: "What are DORA metrics and why do they matter?"**
> "DevOps Research metrics: Deployment Frequency (how often), Lead Time (commit to production), Change Failure Rate (percentage causing issues), MTTR (recovery time). Elite performers: deploy multiple times daily, <1 hour lead time, <15% failure rate, <1 hour recovery. These metrics correlate with organizational performance."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                     CI/CD PIPELINE CHECKLIST                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BUILD:                                                         │
│  □ Lint and type checking                                      │
│  □ Build application                                           │
│  □ Build Docker image                                          │
│  □ Cache dependencies                                          │
│                                                                 │
│  TEST:                                                          │
│  □ Unit tests with coverage                                    │
│  □ Integration tests                                           │
│  □ E2E tests                                                   │
│  □ Performance tests                                           │
│                                                                 │
│  SECURITY:                                                      │
│  □ Dependency scanning                                         │
│  □ SAST (static analysis)                                      │
│  □ Secret scanning                                             │
│  □ Container scanning                                          │
│                                                                 │
│  DEPLOY:                                                        │
│  □ Artifact promotion (not rebuild)                            │
│  □ Environment progression                                     │
│  □ Health checks                                               │
│  □ Rollback capability                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

DORA METRICS (Elite Performance):
┌────────────────────────────────────────────────────────────────┐
│ Deployment Frequency   - Multiple times per day                │
│ Lead Time for Changes  - Less than one hour                    │
│ Change Failure Rate    - 0-15%                                 │
│ Time to Recovery       - Less than one hour                    │
└────────────────────────────────────────────────────────────────┘

DEPLOYMENT STRATEGIES:
┌────────────────────────────────────────────────────────────────┐
│ Rolling    - Gradual replacement, some downtime risk           │
│ Blue-Green - Two environments, instant switch, more resources  │
│ Canary     - Gradual traffic shift, best for large scale       │
│ Recreate   - Stop old, start new, has downtime                 │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

