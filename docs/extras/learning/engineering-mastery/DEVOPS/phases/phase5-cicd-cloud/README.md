# Phase 5: CI/CD & Cloud (AWS)
## Weeks 14-16 | Automate Deployment Pipelines

> **Prerequisites:** Completed Phase 4 (Kubernetes)  
> **Time:** 10-15 hours per week  
> **Outcome:** Fully automated deployment pipeline to AWS

---

## 🎯 Learning Objectives

By the end of Phase 5, you will:
- [ ] Build CI/CD pipelines with GitHub Actions
- [ ] Set up Jenkins for enterprise CI/CD
- [ ] Understand AWS core services (EC2, S3, RDS, IAM)
- [ ] Deploy applications to AWS
- [ ] Implement blue-green and canary deployments

---

## What is CI/CD?

```
CONTINUOUS INTEGRATION / CONTINUOUS DEPLOYMENT:

┌────────────────────────────────────────────────────────────────────┐
│                                                                     │
│   Developer                   CI Pipeline                   CD      │
│   ─────────                   ───────────                ────────   │
│                                                                     │
│   git push ──▶ Build ──▶ Test ──▶ Lint ──▶ Security ──▶ Deploy    │
│       │          │        │        │          │           │        │
│       ▼          ▼        ▼        ▼          ▼           ▼        │
│   Trigger     Compile   Unit    ESLint    Snyk/Trivy   Staging    │
│              TypeScript  Jest   Prettier               Production  │
│                         E2E                                        │
│                                                                     │
│   AUTOMATED! No manual steps. Fast feedback.                       │
│                                                                     │
└────────────────────────────────────────────────────────────────────┘
```

---

## 📅 Weekly Breakdown

### Week 14: GitHub Actions

**What you'll learn:**
- GitHub Actions concepts (workflows, jobs, steps)
- Building CI pipelines
- Deploying to various platforms
- Secrets management
- Matrix builds and caching

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 18: CI/CD Pipelines | 3 hours |
| [../../10-devops-sre.md](../../10-devops-sre.md) | CI/CD section | 2 hours |

**Key concepts:**

```
GITHUB ACTIONS STRUCTURE:

.github/
└── workflows/
    ├── ci.yml          # Runs on every push
    ├── deploy.yml      # Runs on merge to main
    └── pr-checks.yml   # Runs on pull requests

Workflow ──▶ Jobs ──▶ Steps
   │           │         │
   │           │         └── Individual commands
   │           └── Run in parallel or sequence
   └── Triggered by events (push, PR, schedule)
```

**Hands-on exercises:**

```yaml
# Exercise 1: Basic CI workflow
# .github/workflows/ci.yml

name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linting
        run: npm run lint
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
```

```yaml
# Exercise 2: Deploy to production
# .github/workflows/deploy.yml

name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install and build
        run: |
          npm ci
          npm run build
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

```yaml
# Exercise 3: Matrix builds (test multiple versions)
# .github/workflows/matrix.yml

name: Matrix Build

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
        os: [ubuntu-latest, windows-latest]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      
      - run: npm ci
      - run: npm test
```

```yaml
# Exercise 4: Docker build and push
# .github/workflows/docker.yml

name: Docker Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
```

**GitHub Actions cheatsheet:**
```yaml
# Common triggers
on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight
  workflow_dispatch:      # Manual trigger

# Useful actions
actions/checkout@v4       # Clone repo
actions/setup-node@v4     # Setup Node.js
actions/cache@v4          # Cache dependencies
docker/build-push-action@v5  # Build Docker images

# Secrets
${{ secrets.MY_SECRET }}

# Environment variables
env:
  NODE_ENV: production
```

**Checkpoint quiz:**
1. What triggers a GitHub Actions workflow?
2. What's the difference between `jobs` and `steps`?
3. How do you store sensitive values like API keys?

---

### Week 15: Jenkins

**What you'll learn:**
- Jenkins architecture (controller, agents)
- Jenkinsfile and Pipeline syntax
- Blue Ocean UI
- Jenkins plugins
- Distributed builds

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [devop-complete-guide.md](../../devop-complete-guide.md) | Jenkins sections | 2 hours |

**Key concepts:**

```
JENKINS ARCHITECTURE:

┌─────────────────────────────────────────────────────────────────┐
│                     JENKINS CONTROLLER                          │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  • Manages pipelines                                      │  │
│  │  • Stores configuration                                   │  │
│  │  • Schedules builds                                       │  │
│  │  • Web UI                                                 │  │
│  └──────────────────────────────────────────────────────────┘  │
│                            │                                    │
│            ┌───────────────┼───────────────┐                   │
│            ▼               ▼               ▼                   │
│     ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│     │ Agent 1  │    │ Agent 2  │    │ Agent 3  │              │
│     │ (Linux)  │    │ (Windows)│    │ (Docker) │              │
│     └──────────┘    └──────────┘    └──────────┘              │
│                                                                 │
│     Agents execute the actual builds                           │
└─────────────────────────────────────────────────────────────────┘
```

**Hands-on exercises:**

```groovy
// Exercise 1: Basic Jenkinsfile
// Jenkinsfile

pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Build succeeded!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
```

```groovy
// Exercise 2: Multi-branch pipeline with stages
// Jenkinsfile

pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "myapp"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }
        
        stage('Test') {
            steps {
                sh 'docker run ${DOCKER_IMAGE}:${DOCKER_TAG} npm test'
            }
        }
        
        stage('Push') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                sh './deploy.sh staging'
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            input {
                message "Deploy to production?"
                ok "Yes, deploy!"
            }
            steps {
                sh './deploy.sh production'
            }
        }
    }
}
```

```groovy
// Exercise 3: Parallel stages
// Jenkinsfile

pipeline {
    agent any
    
    stages {
        stage('Parallel Tests') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
                stage('E2E Tests') {
                    steps {
                        sh 'npm run test:e2e'
                    }
                }
            }
        }
    }
}
```

**Jenkins vs GitHub Actions:**

| Aspect | GitHub Actions | Jenkins |
|--------|---------------|---------|
| Hosting | GitHub-managed | Self-hosted |
| Setup | Zero config | Requires setup |
| UI | Good | Blue Ocean is great |
| Plugins | Actions marketplace | 1800+ plugins |
| Cost | Free for public repos | Free (self-host) |
| Best for | GitHub projects | Enterprise, complex pipelines |

---

### Week 16: AWS Core Services

**What you'll learn:**
- AWS fundamentals (regions, AZs)
- EC2 (virtual machines)
- S3 (object storage)
- RDS (managed databases)
- IAM (security and access)
- VPC (networking)

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [../12-cloud-architecture.md](../../12-cloud-architecture.md) | AWS Core Services | 3 hours |
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 26: Cloud Fundamentals | 2 hours |

**Key concepts:**

```
AWS CORE SERVICES:

┌─────────────────────────────────────────────────────────────────┐
│                        YOUR VPC                                  │
│                                                                  │
│   ┌─────────────────────────┐  ┌─────────────────────────────┐ │
│   │    Public Subnet        │  │    Private Subnet           │ │
│   │                         │  │                             │ │
│   │  ┌─────────────────┐    │  │  ┌─────────────────┐       │ │
│   │  │  EC2 (Web/API)  │    │  │  │  RDS (Database) │       │ │
│   │  │  + ALB          │◄───┼──┼─▶│  PostgreSQL     │       │ │
│   │  └─────────────────┘    │  │  └─────────────────┘       │ │
│   │          │              │  │                             │ │
│   │          ▼              │  │  ┌─────────────────┐       │ │
│   │  ┌─────────────────┐    │  │  │  ElastiCache    │       │ │
│   │  │ S3 (Static)     │    │  │  │  (Redis)        │       │ │
│   │  └─────────────────┘    │  │  └─────────────────┘       │ │
│   │                         │  │                             │ │
│   └─────────────────────────┘  └─────────────────────────────┘ │
│                                                                  │
│   Security Groups = Firewall rules per resource                 │
│   IAM = Who can access what                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**AWS services overview:**

| Service | Purpose | Use Case |
|---------|---------|----------|
| **EC2** | Virtual machines | Run applications |
| **S3** | Object storage | Static files, backups |
| **RDS** | Managed database | PostgreSQL, MySQL |
| **ElastiCache** | Managed Redis/Memcached | Caching |
| **ALB** | Load balancer | Distribute traffic |
| **Route 53** | DNS | Domain management |
| **CloudFront** | CDN | Global content delivery |
| **IAM** | Access management | Users, roles, policies |
| **VPC** | Virtual network | Isolated network |
| **ECS/EKS** | Container orchestration | Run Docker/K8s |
| **Lambda** | Serverless functions | Event-driven compute |

**Hands-on exercises:**

```bash
# Exercise 1: AWS CLI setup
aws configure
# Enter: Access Key ID, Secret Access Key, Region, Output format

# Verify
aws sts get-caller-identity
```

```bash
# Exercise 2: EC2 operations
# Launch instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t2.micro \
  --key-name my-key \
  --security-groups my-sg

# List instances
aws ec2 describe-instances

# Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
```

```bash
# Exercise 3: S3 operations
# Create bucket
aws s3 mb s3://my-unique-bucket-name

# Upload file
aws s3 cp myfile.txt s3://my-bucket/

# Sync directory
aws s3 sync ./dist s3://my-bucket/static/

# List objects
aws s3 ls s3://my-bucket/
```

```bash
# Exercise 4: RDS operations
# Create PostgreSQL instance
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username admin \
  --master-user-password mypassword \
  --allocated-storage 20
```

```yaml
# Exercise 5: Deploy to AWS with GitHub Actions
# .github/workflows/aws-deploy.yml

name: Deploy to AWS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Login to ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Build and push Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/myapp:$IMAGE_TAG .
          docker push $ECR_REGISTRY/myapp:$IMAGE_TAG
      
      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster my-cluster \
            --service my-service \
            --force-new-deployment
```

**Checkpoint quiz:**
1. What's the difference between EC2 and Lambda?
2. When would you use S3 vs EBS?
3. What is an IAM role and when would you use one?

---

## 📋 Phase 5 Project: Complete CI/CD Pipeline

**Build an end-to-end pipeline that:**
1. Triggers on push to main branch
2. Runs tests and linting
3. Builds Docker image
4. Pushes to container registry
5. Deploys to AWS (ECS or EC2)
6. Sends Slack notification on completion
7. Supports rollback

**Pipeline diagram:**
```
git push → GitHub Actions → Test → Build Image → Push to ECR → Deploy to ECS → Slack
    │                                                              │
    └── On failure: Rollback to previous version ──────────────────┘
```

---

## ✅ Phase 5 Completion Checklist

Before completing this roadmap, ensure you can:

- [ ] Create GitHub Actions workflows for CI/CD
- [ ] Write Jenkinsfiles for complex pipelines
- [ ] Explain AWS core services (EC2, S3, RDS, IAM)
- [ ] Deploy applications to AWS
- [ ] Set up proper IAM roles and security groups
- [ ] Implement automated deployments
- [ ] Complete the Phase 5 project (full CI/CD pipeline)

---

## 🔗 Quick Reference

**GitHub Actions:**
```yaml
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
```

**Jenkins:**
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps { sh 'npm ci' }
        }
    }
}
```

**AWS CLI:**
```bash
aws ec2 describe-instances
aws s3 sync ./dist s3://bucket/
aws ecs update-service --cluster X --service Y
```

---

## 🎉 Congratulations!

You have completed the DevOps Master Roadmap!

**You now know:**
- Linux, Bash, Git (Phase 1)
- Networking, Nginx, SSL, VPS (Phase 2)
- Terraform, Infrastructure as Code (Phase 3)
- Docker, Kubernetes (Phase 4)
- CI/CD, AWS (Phase 5)

**Next steps:**
1. Build real projects
2. Get certified (AWS, CKA, Terraform)
3. Explore advanced topics (Service Mesh, GitOps, Chaos Engineering)
4. Contribute to open source

---

## 🔙 Back to Roadmap

[← Back to Master Roadmap](../../DEVOPS_MASTER_ROADMAP.md)

