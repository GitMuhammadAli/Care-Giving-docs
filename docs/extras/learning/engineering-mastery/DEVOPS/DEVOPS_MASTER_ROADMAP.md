# 🚀 DevOps Master Roadmap
## From Zero to Production-Ready in 16 Weeks

> **This is your SINGLE source of truth for DevOps learning.**  
> Stop switching between guides. Follow this roadmap phase by phase.

---

## 📍 Overview

```
YOUR DEVOPS JOURNEY (16 Weeks)
══════════════════════════════════════════════════════════════════════════════

 Week 1-3          Week 4-6           Week 7-9          Week 10-13        Week 14-16
┌─────────┐      ┌─────────┐       ┌─────────┐       ┌─────────┐       ┌─────────┐
│ PHASE 1 │ ──▶  │ PHASE 2 │  ──▶  │ PHASE 3 │  ──▶  │ PHASE 4 │  ──▶  │ PHASE 5 │
│         │      │         │       │         │       │         │       │         │
│ Linux   │      │ Cloud & │       │ Infra   │       │Container│       │ CI/CD & │
│ Bash    │      │ Infra   │       │ as Code │       │ & K8s   │       │  Cloud  │
│ Git     │      │ VPS     │       │Terraform│       │ Docker  │       │  AWS    │
└─────────┘      └─────────┘       └─────────┘       └─────────┘       └─────────┘
     │                │                 │                 │                 │
     ▼                ▼                 ▼                 ▼                 ▼
  Terminal        Deploy to         Automate          Containerize      Automate
  Mastery         Real Server       Everything        Everything        Pipelines

══════════════════════════════════════════════════════════════════════════════
```

---

## 🎯 Quick Links by Phase

| Phase | Topic | Duration | Start Here |
|-------|-------|----------|------------|
| **1** | Linux, Bash, Git | 3 weeks | [Phase 1 →](./phases/phase1-linux/README.md) |
| **2** | Cloud & Infrastructure | 3 weeks | [Phase 2 →](./phases/phase2-infrastructure/README.md) |
| **3** | Infrastructure as Code | 3 weeks | [Phase 3 →](./phases/phase3-iac/README.md) |
| **4** | Containers & Kubernetes | 4 weeks | [Phase 4 →](./phases/phase4-containers/README.md) |
| **5** | CI/CD & AWS | 3 weeks | [Phase 5 →](./phases/phase5-cicd-cloud/README.md) |

---

## 📅 Week-by-Week Schedule

### Phase 1: Linux Foundations (Weeks 1-3)

| Week | Focus | What You'll Learn | Hands-On |
|------|-------|-------------------|----------|
| **1** | Linux Basics | File system, navigation, permissions | Set up WSL2, navigate terminal |
| **2** | Bash Scripting | Variables, loops, conditionals, functions | Write automation scripts |
| **3** | Git Mastery | Commits, branches, merging, GitHub | Contribute to a project |

**Resources:**
- [devop-complete-guide.md](./devop-complete-guide.md) → Chapter 2: Linux Fundamentals
- [WSL2-DevOps-Beginner-Guide.md](./WSL2-DevOps-Beginner-Guide.md) → Phases 1-5

**Checkpoint:** Can you write a bash script that automates a task?

---

### Phase 2: Cloud & Infrastructure (Weeks 4-6)

| Week | Focus | What You'll Learn | Hands-On |
|------|-------|-------------------|----------|
| **4** | Networking & Web | DNS, HTTP, ports, firewalls | Configure firewall rules |
| **5** | Web Servers | Nginx, reverse proxy, SSL/HTTPS | Set up Nginx with SSL |
| **6** | VPS Deployment | Full server setup, security hardening | Deploy app to real VPS |

**Resources:**
- [Deployment-Fundamentals-Guide.md](./Deployment-Fundamentals-Guide.md) → Chapters 1-10
- [Complete-vps-setup-guide.md](./Complete-vps-setup-guide.md) → All phases
- [devop-complete-guide.md](./devop-complete-guide.md) → Chapters 4-6, 11, 14

**Checkpoint:** Can you deploy an app to a VPS with HTTPS?

---

### Phase 3: Infrastructure as Code (Weeks 7-9)

| Week | Focus | What You'll Learn | Hands-On |
|------|-------|-------------------|----------|
| **7** | Terraform Basics | HCL syntax, providers, resources | Provision cloud VM |
| **8** | Terraform Advanced | Modules, state, workspaces | Multi-environment setup |
| **9** | Pulumi & Comparison | Pulumi basics, when to use what | Compare approaches |

**Resources:**
- [devop-complete-guide.md](./devop-complete-guide.md) → Chapter 17: Infrastructure as Code
- [devop-complete-guide.md](./devop-complete-guide.md) → Chapter 32: Terraform Deep Dive

**Checkpoint:** Can you provision infrastructure with code?

---

### Phase 4: Containers & Orchestration (Weeks 10-13)

| Week | Focus | What You'll Learn | Hands-On |
|------|-------|-------------------|----------|
| **10** | Docker Fundamentals | Images, containers, Dockerfile | Containerize your app |
| **11** | Docker Compose | Multi-container apps, networking | Full stack in containers |
| **12** | Kubernetes Basics | Pods, deployments, services | Deploy to local K8s |
| **13** | Kubernetes Production | Scaling, health checks, secrets | Production-ready cluster |

**Resources:**
- [devop-complete-guide.md](./devop-complete-guide.md) → Chapter 19: Containers & Docker
- [devop-complete-guide.md](./devop-complete-guide.md) → Chapters 27-29: Kubernetes

**Checkpoint:** Can you deploy a scalable app on Kubernetes?

---

### Phase 5: CI/CD & Cloud (Weeks 14-16)

| Week | Focus | What You'll Learn | Hands-On |
|------|-------|-------------------|----------|
| **14** | GitHub Actions | Workflows, jobs, secrets | Automate tests & deployment |
| **15** | Jenkins | Pipelines, agents, plugins | Build Jenkins pipeline |
| **16** | AWS Core | EC2, S3, RDS, IAM, VPC | Deploy full app on AWS |

**Resources:**
- [devop-complete-guide.md](./devop-complete-guide.md) → Chapter 18: CI/CD Pipelines
- [../10-devops-sre.md](../10-devops-sre.md) → CI/CD section
- [../12-cloud-architecture.md](../12-cloud-architecture.md) → AWS Core Services

**Checkpoint:** Can you set up a complete CI/CD pipeline deploying to AWS?

---

## 📊 Progress Tracker

Use this to track your progress:

### Phase 1: Linux Foundations
- [ ] Week 1: Linux basics completed
- [ ] Week 2: Bash scripting completed
- [ ] Week 3: Git mastery completed
- [ ] **Phase 1 Project:** Automation script created

### Phase 2: Cloud & Infrastructure
- [ ] Week 4: Networking fundamentals completed
- [ ] Week 5: Nginx & SSL completed
- [ ] Week 6: VPS deployment completed
- [ ] **Phase 2 Project:** App deployed to VPS with HTTPS

### Phase 3: Infrastructure as Code
- [ ] Week 7: Terraform basics completed
- [ ] Week 8: Terraform advanced completed
- [ ] Week 9: Pulumi introduction completed
- [ ] **Phase 3 Project:** Infrastructure provisioned with Terraform

### Phase 4: Containers & Orchestration
- [ ] Week 10: Docker fundamentals completed
- [ ] Week 11: Docker Compose completed
- [ ] Week 12: Kubernetes basics completed
- [ ] Week 13: Kubernetes production completed
- [ ] **Phase 4 Project:** App running on Kubernetes

### Phase 5: CI/CD & Cloud
- [ ] Week 14: GitHub Actions completed
- [ ] Week 15: Jenkins completed
- [ ] Week 16: AWS core completed
- [ ] **Phase 5 Project:** Complete CI/CD pipeline on AWS

---

## 🗺️ Content Map

This roadmap references the following existing guides:

```
docs/extras/learning/engineering-mastery/
├── DEVOPS/
│   ├── DEVOPS_MASTER_ROADMAP.md     ← YOU ARE HERE
│   ├── phases/                       ← Phase-specific guides
│   │   ├── phase1-linux/
│   │   ├── phase2-infrastructure/
│   │   ├── phase3-iac/
│   │   ├── phase4-containers/
│   │   └── phase5-cicd-cloud/
│   ├── devop-complete-guide.md      ← 7,800 lines reference manual
│   ├── Complete-vps-setup-guide.md  ← VPS hands-on walkthrough
│   ├── Deployment-Fundamentals-Guide.md ← Concepts explained
│   ├── WSL2-DevOps-Beginner-Guide.md    ← Windows setup
│   └── Practical.md                     ← Hands-on exercises
├── 01-fundamentals.md
├── 10-devops-sre.md
└── 12-cloud-architecture.md
```

---

## ⏰ Time Commitment

| Schedule | Weekly Hours | Total Duration |
|----------|--------------|----------------|
| **Part-time** | 10-15 hours | 16 weeks |
| **Full-time** | 30-40 hours | 6-8 weeks |
| **Intensive** | 50+ hours | 4 weeks |

---

## 🎓 Certifications (Optional)

After completing this roadmap, you'll be ready for:

| Certification | After Phase | Prep Time |
|---------------|-------------|-----------|
| AWS Cloud Practitioner | Phase 5 | 2 weeks |
| Docker Certified Associate | Phase 4 | 2 weeks |
| Kubernetes CKA | Phase 4 | 4 weeks |
| HashiCorp Terraform Associate | Phase 3 | 2 weeks |

---

## 🚀 Start Now

**Your first step:** [Phase 1: Linux Foundations →](./phases/phase1-linux/README.md)

---

*Last updated: February 2026*

