# CareCircle Free-Tier Deployment Guide

**Last Updated**: February 3, 2026
**Cost**: $0/month (all free tiers)
**Stack**: Vercel + Render + Neon + Upstash + CloudAMQP + Brevo

---

## Table of Contents

1. [Overview](#overview)
2. [Services Used](#services-used)
3. [Step 1: Database (Neon PostgreSQL)](#step-1-database-neon-postgresql)
4. [Step 2: Redis (Upstash)](#step-2-redis-upstash)
5. [Step 3: Message Queue (CloudAMQP)](#step-3-message-queue-cloudamqp)
6. [Step 4: Email Service (Brevo)](#step-4-email-service-brevo)
7. [Step 5: File Storage (Cloudinary)](#step-5-file-storage-cloudinary)
8. [Step 6: Backend Deployment (Render)](#step-6-backend-deployment-render)
9. [Step 7: Frontend Deployment (Vercel)](#step-7-frontend-deployment-vercel)
10. [Environment Variables Reference](#environment-variables-reference)
11. [Free-Tier Limits](#free-tier-limits)
12. [Troubleshooting](#troubleshooting)

---

## Overview

This guide deploys CareCircle using **100% free-tier services**. Perfect for:
- Development and testing
- Demo/portfolio projects
- Early-stage startups
- Learning deployments

### Architecture

```
┌─────────────────┐     ┌─────────────────┐
│   Frontend      │     │    Backend      │
│   (Vercel)      │────▶│    (Render)     │
│   Next.js       │     │    NestJS       │
└─────────────────┘     └────────┬────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
        ▼                        ▼                        ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│  PostgreSQL   │    │    Redis      │    │   RabbitMQ    │
│   (Neon)      │    │  (Upstash)    │    │  (CloudAMQP)  │
└───────────────┘    └───────────────┘    └───────────────┘

        ┌────────────────────────┬────────────────────────┐
        │                        │                        │
        ▼                        ▼                        ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│    Email      │    │    Files      │    │  Real-time    │
│   (Brevo)     │    │ (Cloudinary)  │    │   (Stream)    │
└───────────────┘    └───────────────┘    └───────────────┘
```

---

## Services Used

| Service | Purpose | Free Tier Limit |
|---------|---------|-----------------|
| **Neon** | PostgreSQL Database | 0.5 GB storage, auto-suspend |
| **Upstash** | Redis Cache | 10K commands/day |
| **CloudAMQP** | RabbitMQ Queue | 1M messages/month |
| **Brevo** | Email Service | 300 emails/day |
| **Cloudinary** | File Storage | 25GB bandwidth/month |
| **Render** | Backend Hosting | 750 hours/month, auto-sleep |
| **Vercel** | Frontend Hosting | 100GB bandwidth/month |
| **Stream** | Chat (optional) | 5M API calls/month |

---

## Step 1: Database (Neon PostgreSQL)

### 1.1 Create Account
1. Go to [neon.tech](https://neon.tech)
2. Sign up with GitHub or email
3. Create a new project named `carecircle`

### 1.2 Get Connection String
1. In your project, click **Connection Details**
2. Toggle **Pooled connection** ON (important!)
3. Copy the connection string

### 1.3 Connection String Format
```
postgresql://USERNAME:PASSWORD@ep-xxx-pooler.REGION.aws.neon.tech/neondb?sslmode=require
```

### 1.4 Important Notes
- **Auto-suspend**: Neon suspends after 5 minutes of inactivity
- **Cold start**: Takes 5-10 seconds to wake up
- **Our app handles this**: Retry logic added in `prisma.service.ts`

### 1.5 Environment Variables
```env
DATABASE_URL=postgresql://neondb_owner:YOUR_PASSWORD@ep-xxx-pooler.region.aws.neon.tech/neondb?sslmode=require
DB_HOST=ep-xxx-pooler.region.aws.neon.tech
DB_PORT=5432
DB_USERNAME=neondb_owner
DB_PASSWORD=YOUR_PASSWORD
DB_DATABASE=neondb
DB_SSL=true
```

---

## Step 2: Redis (Upstash)

### 2.1 Create Account
1. Go to [upstash.com](https://upstash.com)
2. Sign up and create a new Redis database
3. Select region closest to your Render deployment

### 2.2 Get Credentials
1. In database details, copy:
   - **Endpoint** (host)
   - **Password**
   - **REST URL** and **REST Token**

### 2.3 Environment Variables
```env
REDIS_HOST=xxx.upstash.io
REDIS_PORT=6379
REDIS_PASSWORD=YOUR_UPSTASH_PASSWORD
REDIS_TLS=true
UPSTASH_REDIS_REST_URL=https://xxx.upstash.io
UPSTASH_REDIS_REST_TOKEN=YOUR_REST_TOKEN
```

---

## Step 3: Message Queue (CloudAMQP)

### 3.1 Create Account
1. Go to [cloudamqp.com](https://www.cloudamqp.com)
2. Sign up and create a new instance
3. Select **Little Lemur** (free tier)

### 3.2 Get Connection URL
1. In instance details, copy the **AMQP URL**

### 3.3 Environment Variables
```env
AMQP_URL=amqps://USER:PASS@HOST/VHOST
AMQP_HOST=xxx.cloudamqp.com
AMQP_USER=your_user
AMQP_PASSWORD=your_password
AMQP_VHOST=your_vhost
AMQP_PORT=5672
AMQP_TLS=true
```

---

## Step 4: Email Service (Brevo)

### 4.1 Create Account
1. Go to [brevo.com](https://www.brevo.com)
2. Sign up for free account

### 4.2 Get SMTP Credentials
1. Go to **Settings** → **SMTP & API**
2. Copy SMTP settings:
   - Host: `smtp-relay.brevo.com`
   - Port: `587`
   - Login (SMTP User)
   - Password (SMTP Key)

### 4.3 Add Sender Email
1. Go to **Settings** → **Senders, Domains & Dedicated IPs** → **Senders**
2. Click **Add a sender**
3. Enter your email (e.g., `yourname@gmail.com`)
4. Verify via email link

### 4.4 Environment Variables
```env
MAIL_PROVIDER=smtp
SMTP_HOST=smtp-relay.brevo.com
SMTP_PORT=587
SMTP_USER=your_brevo_smtp_login
SMTP_PASS=your_brevo_smtp_key
MAIL_FROM=your_verified_email@gmail.com
MAIL_FROM_NAME=CareCircle
```

### 4.5 Important Notes
- **Must verify sender**: Emails from unverified addresses are rejected
- **No custom domain needed**: Use your personal Gmail
- **300 emails/day limit**: Tracked in app via LimitsService

---

## Step 5: File Storage (Cloudinary)

### 5.1 Create Account
1. Go to [cloudinary.com](https://cloudinary.com)
2. Sign up for free account

### 5.2 Get Credentials
1. Go to **Dashboard**
2. Copy Cloud Name, API Key, API Secret

### 5.3 Environment Variables
```env
STORAGE_PROVIDER=cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
CLOUDINARY_FOLDER=carecircle
CLOUDINARY_URL=cloudinary://API_KEY:API_SECRET@CLOUD_NAME
```

---

## Step 6: Backend Deployment (Render)

### 6.1 Create Account
1. Go to [render.com](https://render.com)
2. Sign up with GitHub

### 6.2 Create Web Service
1. Click **New** → **Web Service**
2. Connect your GitHub repository
3. Configure:
   - **Name**: `carecircle-api`
   - **Region**: Choose closest to users
   - **Branch**: `main`
   - **Root Directory**: (leave empty)
   - **Build Command**: `pnpm install && pnpm build`
   - **Start Command**: `pnpm --filter @carecircle/api start:prod`

### 6.3 Add Environment Variables
In Render dashboard → **Environment**, add ALL variables:

```env
# App
NODE_ENV=production
PORT=3000
API_PREFIX=api/v1
FRONTEND_URL=https://your-app.vercel.app

# Database (Neon)
DATABASE_URL=postgresql://...

# Redis (Upstash)
REDIS_HOST=xxx.upstash.io
REDIS_PORT=6379
REDIS_PASSWORD=xxx
REDIS_TLS=true

# RabbitMQ (CloudAMQP)
AMQP_URL=amqps://...

# Email (Brevo)
MAIL_PROVIDER=smtp
SMTP_HOST=smtp-relay.brevo.com
SMTP_PORT=587
SMTP_USER=xxx
SMTP_PASS=xxx
MAIL_FROM=your_verified_email@gmail.com
MAIL_FROM_NAME=CareCircle

# JWT
JWT_SECRET=your-super-secret-min-32-chars
JWT_REFRESH_SECRET=another-secret-min-32-chars
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# Security
ENCRYPTION_KEY=0123456789abcdef0123456789abcdef

# Cloudinary
CLOUDINARY_CLOUD_NAME=xxx
CLOUDINARY_API_KEY=xxx
CLOUDINARY_API_SECRET=xxx
CLOUDINARY_FOLDER=carecircle
```

### 6.4 Deploy
1. Click **Create Web Service**
2. Wait for build to complete
3. Your API is live at `https://carecircle-api.onrender.com`

### 6.5 Important Notes
- **Auto-sleep**: Free tier sleeps after 15 min of inactivity
- **Cold start**: Takes 30-50 seconds to wake up
- **750 hours/month**: Enough for ~31 days if always on

---

## Step 7: Frontend Deployment (Vercel)

### 7.1 Create Account
1. Go to [vercel.com](https://vercel.com)
2. Sign up with GitHub

### 7.2 Import Project
1. Click **Add New** → **Project**
2. Import your GitHub repository
3. Configure:
   - **Framework Preset**: Next.js
   - **Root Directory**: `apps/web`

### 7.3 Add Environment Variables
```env
NEXT_PUBLIC_API_URL=https://carecircle-api.onrender.com/api/v1
NEXT_PUBLIC_WS_URL=https://carecircle-api.onrender.com
NEXT_PUBLIC_APP_NAME=CareCircle
NEXT_PUBLIC_VAPID_PUBLIC_KEY=your_vapid_key
NEXT_PUBLIC_STREAM_API_KEY=your_stream_key
NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME=your_cloud_name
```

### 7.4 Deploy
1. Click **Deploy**
2. Your app is live at `https://your-app.vercel.app`

---

## Environment Variables Reference

### Required for Backend (Render)

| Variable | Description | Example |
|----------|-------------|---------|
| `NODE_ENV` | Environment | `production` |
| `PORT` | Server port | `3000` |
| `DATABASE_URL` | Neon connection string | `postgresql://...` |
| `REDIS_HOST` | Upstash host | `xxx.upstash.io` |
| `REDIS_PASSWORD` | Upstash password | `xxx` |
| `REDIS_TLS` | Enable TLS | `true` |
| `AMQP_URL` | CloudAMQP URL | `amqps://...` |
| `JWT_SECRET` | JWT signing key | Min 32 chars |
| `JWT_REFRESH_SECRET` | Refresh token key | Min 32 chars |
| `SMTP_HOST` | Brevo SMTP host | `smtp-relay.brevo.com` |
| `SMTP_PORT` | Brevo SMTP port | `587` |
| `SMTP_USER` | Brevo login | `xxx@smtp-brevo.com` |
| `SMTP_PASS` | Brevo password | `xsmtpsib-xxx` |
| `MAIL_FROM` | Verified sender email | `you@gmail.com` |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary name | `xxx` |
| `CLOUDINARY_API_KEY` | Cloudinary key | `xxx` |
| `CLOUDINARY_API_SECRET` | Cloudinary secret | `xxx` |
| `FRONTEND_URL` | Your Vercel URL | `https://xxx.vercel.app` |

### Required for Frontend (Vercel)

| Variable | Description | Example |
|----------|-------------|---------|
| `NEXT_PUBLIC_API_URL` | Backend API URL | `https://xxx.onrender.com/api/v1` |
| `NEXT_PUBLIC_WS_URL` | WebSocket URL | `https://xxx.onrender.com` |
| `NEXT_PUBLIC_APP_NAME` | App name | `CareCircle` |
| `NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME` | Cloudinary name | `xxx` |

---

## Free-Tier Limits

Our app tracks usage and warns at 80% of limits:

| Resource | Provider Limit | App Limit (80%) | Tracked |
|----------|---------------|-----------------|---------|
| Emails/day | 300 | 240 | Yes |
| File uploads/month | 25,000 | 20,000 | Yes |
| Max file size | - | 10 MB | Yes |
| Redis commands/day | 10,000 | 8,000 | Yes |
| RabbitMQ messages/month | 1,000,000 | 800,000 | Yes |

Check usage: `GET /api/v1/admin/system/usage`

---

## Troubleshooting

### Database Connection Error (P1001)
**Symptom**: "Can't reach database server"
**Cause**: Neon database is suspended
**Fix**:
1. Go to [console.neon.tech](https://console.neon.tech)
2. Click on your project - it will auto-wake
3. Wait 5-10 seconds and retry

### Emails Not Received
**Symptom**: Registration/invite emails not arriving
**Cause**: Sender email not verified in Brevo
**Fix**:
1. Go to Brevo → Senders → Add your email
2. Verify via email link
3. Update `MAIL_FROM` in Render to match

### Frontend Can't Connect to Backend
**Symptom**: Network errors, CORS issues
**Cause**: Wrong API URL or backend sleeping
**Fix**:
1. Check `NEXT_PUBLIC_API_URL` in Vercel
2. Ensure it points to your Render URL
3. Redeploy Vercel after changing env vars

### Backend Takes 30+ Seconds to Respond
**Symptom**: First request very slow
**Cause**: Render free tier was sleeping
**Fix**: This is expected. Free tier sleeps after 15 min inactivity.
- First request wakes it up (30-50 sec)
- Subsequent requests are fast
- Consider upgrading for production use

### Redis Connection Error
**Symptom**: "Redis connection failed"
**Cause**: TLS not enabled or wrong credentials
**Fix**:
1. Ensure `REDIS_TLS=true` in Render
2. Verify host and password from Upstash dashboard
3. App continues in "degraded mode" without cache

### RabbitMQ Queue Error
**Symptom**: "PRECONDITION_FAILED" for queue
**Cause**: Queue arguments changed
**Fix**:
1. Go to CloudAMQP Manager UI
2. Delete the conflicting queue
3. Redeploy - queue will be recreated

---

## Deployment Checklist

### Before First Deploy
- [ ] Create Neon database
- [ ] Create Upstash Redis
- [ ] Create CloudAMQP instance
- [ ] Create Brevo account
- [ ] Verify sender email in Brevo
- [ ] Create Cloudinary account
- [ ] Create Render account
- [ ] Create Vercel account

### Backend (Render)
- [ ] Create web service connected to GitHub
- [ ] Add all environment variables
- [ ] Deploy and verify logs show "Nest application successfully started"
- [ ] Test health endpoint: `https://your-api.onrender.com/api/v1/health`

### Frontend (Vercel)
- [ ] Import project from GitHub
- [ ] Set root directory to `apps/web`
- [ ] Add environment variables (especially `NEXT_PUBLIC_API_URL`)
- [ ] Deploy and verify app loads

### Post-Deploy
- [ ] Test user registration (creates account, sends email)
- [ ] Test login flow
- [ ] Check admin dashboard for usage stats
- [ ] Verify database has data via Neon console

---

## Updating Production

### After Code Changes
1. Push to `main` branch
2. Render auto-deploys backend
3. Vercel auto-deploys frontend

### After Environment Variable Changes
**Render**: Changes apply on next deploy (trigger manual if needed)
**Vercel**: Must redeploy for `NEXT_PUBLIC_*` changes (bundled at build)

### Database Migrations
```bash
# Push schema changes to Neon
pnpm --filter @carecircle/database db:push
```

---

## URLs Reference

After deployment, your app is available at:

| Service | URL |
|---------|-----|
| Frontend | `https://your-app.vercel.app` |
| Backend API | `https://your-api.onrender.com/api/v1` |
| Health Check | `https://your-api.onrender.com/api/v1/health` |
| Admin Usage | `https://your-api.onrender.com/api/v1/admin/system/usage` |
| Neon Dashboard | `https://console.neon.tech` |
| Render Dashboard | `https://dashboard.render.com` |
| Vercel Dashboard | `https://vercel.com/dashboard` |
| Brevo Dashboard | `https://app.brevo.com` |

---

## Support

- **GitHub Issues**: [github.com/GitMuhammadAli/Care-Giving/issues](https://github.com/GitMuhammadAli/Care-Giving/issues)
- **Neon Docs**: [neon.tech/docs](https://neon.tech/docs)
- **Render Docs**: [render.com/docs](https://render.com/docs)
- **Vercel Docs**: [vercel.com/docs](https://vercel.com/docs)
