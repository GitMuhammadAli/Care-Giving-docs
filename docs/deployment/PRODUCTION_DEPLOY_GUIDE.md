# CareCircle Production Deployment Guide

Complete guide for deploying CareCircle to production with CI/CD automation.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Prerequisites](#prerequisites)
3. [Part 1: Deploy Frontend (Vercel)](#part-1-deploy-frontend-vercel)
4. [Part 2: Deploy Backend (Render)](#part-2-deploy-backend-render)
5. [Part 3: Connect Frontend & Backend](#part-3-connect-frontend--backend)
6. [Part 4: Keep Backend Awake (UptimeRobot)](#part-4-keep-backend-awake-uptimerobot)
7. [Part 5: CI/CD Automation](#part-5-cicd-automation)
8. [Part 6: Verification & Testing](#part-6-verification--testing)
9. [Troubleshooting](#troubleshooting)

---

## Architecture Overview

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│     Vercel      │────▶│     Render      │────▶│   Neon (DB)     │
│   (Frontend)    │     │   (Backend)     │     │   PostgreSQL    │
│                 │     │                 │     │                 │
└─────────────────┘     └────────┬────────┘     └─────────────────┘
                                 │
                    ┌────────────┼────────────┐
                    │            │            │
                    ▼            ▼            ▼
             ┌──────────┐ ┌──────────┐ ┌──────────┐
             │ Upstash  │ │CloudAMQP │ │Cloudinary│
             │ (Redis)  │ │(RabbitMQ)│ │ (Files)  │
             └──────────┘ └──────────┘ └──────────┘
```

| Component | Platform | Cost | Purpose |
|-----------|----------|------|---------|
| Frontend | Vercel | Free | Next.js hosting |
| Backend | Render | Free | NestJS API |
| Database | Neon | Free | PostgreSQL |
| Cache | Upstash | Free | Redis |
| Queue | CloudAMQP | Free | RabbitMQ |
| Storage | Cloudinary | Free | File uploads |
| Monitoring | UptimeRobot | Free | Keep-alive pings |

---

## Prerequisites

Before deploying, ensure you have:

- [ ] GitHub account with repository access
- [ ] All third-party services configured (see `docs/getting-started/FREE_SERVICES_SETUP.md`)
- [ ] Environment variables ready (stored securely, NOT in this doc)

---

## Part 1: Deploy Frontend (Vercel)

### Step 1: Create Vercel Account
1. Go to [vercel.com](https://vercel.com)
2. Click **"Login"** → **"Continue with GitHub"**
3. Authorize Vercel

### Step 2: Import Project
1. Click **"Add New..."** → **"Project"**
2. Find repository: **Care-Giving**
3. Click **"Import"**

### Step 3: Configure Project

| Setting | Value |
|---------|-------|
| Project Name | `carecircle` (or preferred name) |
| Framework Preset | Next.js (auto-detected) |
| Root Directory | `apps/web` |
| Branch | `main` |

### Step 4: Add Environment Variables

Add these variables (get values from your secure storage):

| Variable Name | Description |
|---------------|-------------|
| `NEXT_PUBLIC_API_URL` | Backend API URL (update after Render deploy) |
| `NEXT_PUBLIC_WS_URL` | WebSocket URL (same as API base URL) |
| `NEXT_PUBLIC_APP_NAME` | Application name |
| `NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME` | Cloudinary cloud name |
| `NEXT_PUBLIC_STREAM_API_KEY` | Stream Chat API key |
| `NEXT_PUBLIC_VAPID_PUBLIC_KEY` | VAPID public key for push notifications |

### Step 5: Deploy
1. Click **"Deploy"**
2. Wait 2-3 minutes
3. Note your URL: `https://your-app.vercel.app`

### Vercel CI/CD (Automatic)
- ✅ Auto-deploys on every push to `main` branch
- ✅ Preview deployments for pull requests
- ✅ Automatic SSL certificates
- ✅ Global CDN distribution

---

## Part 2: Deploy Backend (Render)

### Step 1: Create Render Account
1. Go to [dashboard.render.com](https://dashboard.render.com)
2. Click **"Sign Up"** → **"GitHub"**
3. Authorize Render

### Step 2: Create Web Service
1. Click **"New +"** → **"Web Service"**
2. Select **"Build and deploy from a Git repository"**
3. Connect your GitHub repository: **Care-Giving**
4. Click **"Connect"**

### Step 3: Configure Service

| Setting | Value |
|---------|-------|
| Name | `carecircle-api` |
| Region | Oregon (US West) |
| Branch | `main` |
| Root Directory | `apps/api` |
| Runtime | Node |
| Instance Type | Free |

### Step 4: Set Build & Start Commands

| Setting | Value |
|---------|-------|
| Build Command | `pnpm install --prod=false && pnpm run build` |
| Start Command | `node dist/main.js` |
| Health Check Path | `/health` |

### Step 5: Add Environment Variables

Add all required environment variables (see `env/prod.env.example` for the full list):

| Category | Variables |
|----------|-----------|
| **App** | `NODE_ENV`, `PORT`, `API_PREFIX` |
| **Database** | `DATABASE_URL` |
| **Redis** | `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`, `REDIS_TLS` |
| **Queue** | `AMQP_URL` |
| **Auth** | `JWT_SECRET`, `JWT_REFRESH_SECRET`, `JWT_EXPIRES_IN`, `JWT_REFRESH_EXPIRES_IN` |
| **Security** | `ENCRYPTION_KEY` |
| **Storage** | `STORAGE_PROVIDER`, `CLOUDINARY_*` |
| **Email** | `MAIL_PROVIDER`, `SMTP_*` |
| **Push** | `VAPID_PRIVATE_KEY`, `VAPID_SUBJECT` |
| **CORS** | `FRONTEND_URL`, `CORS_ORIGIN` |

### Step 6: Deploy
1. Click **"Create Web Service"**
2. Wait 5-10 minutes for build
3. Note your URL: `https://carecircle-api.onrender.com`

### Render CI/CD (Automatic)
- ✅ Auto-deploys on every push to `main` branch
- ✅ Automatic SSL certificates
- ✅ Health check monitoring
- ⚠️ Free tier sleeps after 15 min inactivity (see Part 4)

---

## Part 3: Connect Frontend & Backend

After both are deployed, update the URLs:

### Update Vercel Environment Variables
1. Go to Vercel → Your Project → **Settings** → **Environment Variables**
2. Update these variables:

| Variable | Value |
|----------|-------|
| `NEXT_PUBLIC_API_URL` | `https://your-render-url.onrender.com/api/v1` |
| `NEXT_PUBLIC_WS_URL` | `https://your-render-url.onrender.com` |

3. Click **Save**

### Redeploy Vercel
1. Go to **Deployments** tab
2. Click **"..."** on latest deployment
3. Click **"Redeploy"**

### Verify Connection
1. Open your Vercel URL
2. Open browser DevTools → Network tab
3. Check that API calls go to your Render URL

---

## Part 4: Keep Backend Awake (UptimeRobot)

Render free tier sleeps after 15 minutes of inactivity. Set up monitoring to keep it awake.

### Option A: UptimeRobot (Recommended)

1. Go to [uptimerobot.com](https://uptimerobot.com)
2. Create free account
3. Click **"+ Add New Monitor"**
4. Configure:

| Setting | Value |
|---------|-------|
| Monitor Type | HTTP(s) |
| Friendly Name | CareCircle API |
| URL | `https://your-render-url.onrender.com/health` |
| Monitoring Interval | 5 minutes |

5. Click **"Create Monitor"**

### Option B: GitHub Actions (Alternative)

The repository includes a keep-alive workflow. To enable:

1. Go to GitHub → Repository → **Settings** → **Secrets and variables** → **Actions**
2. Add new secret:
   - Name: `API_HEALTH_URL`
   - Value: `https://your-render-url.onrender.com/health`
3. The workflow at `.github/workflows/keep-alive.yml` will ping every 14 minutes

### Option C: Cron-job.org (Alternative)

1. Go to [cron-job.org](https://cron-job.org)
2. Create free account
3. Create new cron job:
   - URL: `https://your-render-url.onrender.com/health`
   - Schedule: Every 5 minutes

---

## Part 5: CI/CD Automation

Both platforms automatically deploy on push to `main`. Here's how the pipeline works:

### Deployment Flow

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Developer  │     │    GitHub    │     │   Platforms  │
│  pushes code │────▶│   receives   │────▶│  auto-deploy │
│   to main    │     │    commit    │     │              │
└──────────────┘     └──────────────┘     └──────────────┘
                                                  │
                            ┌─────────────────────┼─────────────────────┐
                            │                     │                     │
                            ▼                     ▼                     ▼
                     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
                     │    Vercel    │     │    Render    │     │   GitHub     │
                     │   Frontend   │     │   Backend    │     │   Actions    │
                     │  ~2-3 mins   │     │  ~5-10 mins  │     │    Tests     │
                     └──────────────┘     └──────────────┘     └──────────────┘
```

### GitHub Actions Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `ci.yml` | Push/PR | Run tests and linting |
| `deploy-production.yml` | Push to main | Production deployment |
| `keep-alive.yml` | Schedule (14 min) | Ping API to prevent sleep |
| `security.yml` | Push/PR | Security scanning |

### Manual Deployment

If you need to trigger a manual deploy:

**Vercel:**
1. Go to Deployments → Click "..." → "Redeploy"

**Render:**
1. Go to Deployments → "Manual Deploy" → "Deploy latest commit"

### Rollback

**Vercel:**
1. Go to Deployments
2. Find previous working deployment
3. Click "..." → "Promote to Production"

**Render:**
1. Go to Deployments
2. Find previous working deployment
3. Click "Rollback"

---

## Part 6: Verification & Testing

### Health Checks

| Endpoint | Expected Response |
|----------|-------------------|
| `https://your-render-url.onrender.com/health` | `{"status":"ok",...}` |
| `https://your-render-url.onrender.com/health/ready` | `{"status":"ok",...}` |
| `https://your-render-url.onrender.com/health/live` | `{"status":"ok",...}` |

### Functional Tests

1. **Homepage loads**: Open Vercel URL
2. **Registration works**: Create test account
3. **Login works**: Login with test account
4. **API connected**: Check Network tab for successful API calls
5. **Data persists**: Logout and login, verify data is saved

### Performance Baseline

| Metric | Target | Tool |
|--------|--------|------|
| First load | < 3s | Chrome DevTools |
| API response | < 500ms | Network tab |
| Health check | < 200ms | UptimeRobot |

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| **Render: Build fails with "prisma not found"** | Use build command: `pnpm install --prod=false && pnpm run build` |
| **Render: "yarn start" error** | Set start command: `node dist/main.js` |
| **Render: 502/503 errors** | Wait 30-60 seconds (cold start), check logs |
| **Vercel: CORS errors** | Verify `FRONTEND_URL` and `CORS_ORIGIN` match Vercel URL |
| **Vercel: API not connecting** | Check `NEXT_PUBLIC_API_URL` includes `/api/v1` |
| **Database errors** | Verify `DATABASE_URL` has `?sslmode=require` |
| **Redis errors** | Verify `REDIS_TLS=true` for Upstash |

### Checking Logs

**Render:**
1. Go to service dashboard
2. Click on failed deployment
3. View "Logs" section

**Vercel:**
1. Go to project dashboard
2. Click on deployment
3. View "Build Logs" or "Runtime Logs"

### Support Resources

- Render Docs: [render.com/docs](https://render.com/docs)
- Vercel Docs: [vercel.com/docs](https://vercel.com/docs)
- Project Issues: GitHub Issues tab

---

## Quick Reference

### URLs to Bookmark

| Service | URL |
|---------|-----|
| Vercel Dashboard | `https://vercel.com/dashboard` |
| Render Dashboard | `https://dashboard.render.com` |
| UptimeRobot | `https://uptimerobot.com/dashboard` |
| Neon Console | `https://console.neon.tech` |
| Upstash Console | `https://console.upstash.com` |
| CloudAMQP | `https://customer.cloudamqp.com` |
| Cloudinary | `https://console.cloudinary.com` |

### Deployment Commands (Local)

```bash
# Switch to main branch
git checkout main

# Make changes and commit
git add .
git commit -m "feat: your changes"

# Push to trigger auto-deploy
git push origin main

# Both Vercel and Render will auto-deploy!
```

---

## Summary

| Step | Platform | Time |
|------|----------|------|
| 1. Deploy Frontend | Vercel | 5 min |
| 2. Deploy Backend | Render | 10 min |
| 3. Connect URLs | Both | 2 min |
| 4. Set up Monitoring | UptimeRobot | 2 min |
| 5. Verify | Browser | 5 min |

**Total setup time: ~25 minutes**

After initial setup, all future deployments are automatic on `git push`!

---

*Last updated: February 2026*

