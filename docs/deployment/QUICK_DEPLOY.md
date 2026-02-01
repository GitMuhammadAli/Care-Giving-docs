# Quick Deploy Guide - 90 Minutes to Production

Deploy CareCircle to production for **$0/month** in about 90 minutes.

## Prerequisites

- GitHub account (for repository and CI/CD)
- Node.js 20+ and pnpm installed locally
- 5 browser tabs ready for service signups

---

## Phase 1: Sign Up for Free Services (20 min)

Open these links and create accounts:

| # | Service | Link | What You Get |
|---|---------|------|--------------|
| 1 | **Vercel** | [vercel.com/signup](https://vercel.com/signup) | Frontend hosting (100GB/month) |
| 2 | **Render** | [render.com](https://render.com) | Backend hosting (750 hrs/month) |
| 3 | **Neon** | [neon.tech](https://neon.tech) | PostgreSQL (3GB free) |
| 4 | **Upstash** | [upstash.com](https://upstash.com) | Redis (10K cmds/day) |
| 5 | **Cloudinary** | [cloudinary.com](https://cloudinary.com) | File storage (25GB) |

**Sign up with GitHub** for faster setup.

---

## Phase 2: Database Setup - Neon (10 min)

1. **Create Project**
   - Go to [console.neon.tech](https://console.neon.tech)
   - Click "New Project"
   - Name: `carecircle-prod`
   - Region: `US East` (or closest to your users)
   - Click "Create Project"

2. **Copy Connection String**
   ```
   postgresql://username:password@ep-xxx.us-east-2.aws.neon.tech/neondb?sslmode=require
   ```
   Save this - you'll need it in Phase 4.

3. **Run Migrations**
   ```bash
   # In your local project
   export DATABASE_URL="your-neon-connection-string"
   pnpm --filter @carecircle/database db:push
   ```

---

## Phase 3: Redis Setup - Upstash (5 min)

1. **Create Database**
   - Go to [console.upstash.com](https://console.upstash.com)
   - Click "Create Database"
   - Name: `carecircle-cache`
   - Region: `US East` (same as Neon)
   - Type: `Regional`

2. **Copy Credentials**
   ```
   UPSTASH_REDIS_REST_URL=https://xxx.upstash.io
   UPSTASH_REDIS_REST_TOKEN=AXlkXXX...
   ```

---

## Phase 4: Backend Deploy - Render (15 min)

1. **Create Web Service**
   - Go to [dashboard.render.com](https://dashboard.render.com)
   - Click "New" → "Web Service"
   - Connect your GitHub repo
   - Configure:
     ```
     Name: carecircle-api
     Region: Oregon (US West)
     Branch: main (or develop)
     Root Directory: apps/api
     Runtime: Node
     Build Command: npm install && npm run build
     Start Command: npm run start:prod
     Instance Type: Free
     ```

2. **Add Environment Variables** (click "Advanced")
   ```bash
   NODE_ENV=production
   PORT=3000
   
   # Database (from Neon)
   DATABASE_URL=postgresql://username:password@ep-xxx.neon.tech/neondb?sslmode=require
   
   # Redis (from Upstash)
   REDIS_HOST=xxx.upstash.io
   REDIS_PORT=6379
   REDIS_PASSWORD=your-upstash-password
   REDIS_TLS=true
   
   # JWT (generate with: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))")
   JWT_SECRET=your-64-char-secret-here
   JWT_REFRESH_SECRET=another-64-char-secret-here
   
   # Cloudinary
   CLOUDINARY_CLOUD_NAME=your-cloud-name
   CLOUDINARY_API_KEY=your-api-key
   CLOUDINARY_API_SECRET=your-api-secret
   
   # Frontend URL (update after Vercel deploy)
   FRONTEND_URL=https://your-app.vercel.app
   ```

3. **Deploy**
   - Click "Create Web Service"
   - Wait 5-10 minutes for build
   - Note your URL: `https://carecircle-api.onrender.com`

4. **Verify**
   ```bash
   curl https://carecircle-api.onrender.com/health
   # Should return: {"status":"ok",...}
   ```

---

## Phase 5: Frontend Deploy - Vercel (15 min)

1. **Import Project**
   - Go to [vercel.com/new](https://vercel.com/new)
   - Import your GitHub repository
   - Configure:
     ```
     Framework: Next.js
     Root Directory: apps/web
     Build Command: npm run build
     Output Directory: .next
     ```

2. **Add Environment Variables**
   ```bash
   NEXT_PUBLIC_API_URL=https://carecircle-api.onrender.com/api/v1
   NEXT_PUBLIC_WS_URL=https://carecircle-api.onrender.com
   NEXT_PUBLIC_APP_URL=https://your-app.vercel.app
   NEXT_PUBLIC_APP_NAME=CareCircle
   
   # Cloudinary (for image display)
   NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME=your-cloud-name
   ```

3. **Deploy**
   - Click "Deploy"
   - Wait 2-3 minutes
   - Note your URL: `https://carecircle-xxx.vercel.app`

4. **Update Backend CORS**
   - Go back to Render dashboard
   - Update `FRONTEND_URL` to your Vercel URL
   - Render will auto-redeploy

---

## Phase 6: Keep-Alive Setup (10 min)

Render free tier sleeps after 15 min. Prevent this:

1. **UptimeRobot** (Free)
   - Go to [uptimerobot.com](https://uptimerobot.com)
   - Sign up and verify email
   - Add Monitor:
     ```
     Type: HTTP(s)
     Name: CareCircle API
     URL: https://carecircle-api.onrender.com/health
     Interval: 5 minutes
     ```

---

## Phase 7: Verify Everything (15 min)

### Checklist

- [ ] **API Health**: `curl https://your-api.onrender.com/health`
- [ ] **Web Loads**: Open `https://your-app.vercel.app`
- [ ] **Can Register**: Create a test account
- [ ] **Can Login**: Login works
- [ ] **Database**: Data persists after logout/login

### Quick Fixes

**API returns 502/503:**
- Wait 30 seconds (cold start)
- Check Render logs for errors

**Database connection failed:**
- Verify DATABASE_URL has `?sslmode=require`
- Check Neon console for connection status

**CORS errors:**
- Verify FRONTEND_URL matches your Vercel URL exactly
- Include `https://` prefix

**Images not loading:**
- Check Cloudinary credentials
- Verify CLOUDINARY_CLOUD_NAME is set in both Render and Vercel

---

## Cost Summary

| Service | Free Tier | When You Pay |
|---------|-----------|--------------|
| Vercel | 100GB/month | After 100GB bandwidth |
| Render | 750 hrs/month | Never (if 1 service) |
| Neon | 3GB storage | After 3GB database |
| Upstash | 10K cmds/day | After 10K daily commands |
| Cloudinary | 25GB storage | After 25GB files |
| UptimeRobot | 50 monitors | Never (free forever) |

**Total: $0/month** for ~1,000-2,000 active users

---

## Quick Commands Reference

```bash
# Local development
make setup          # First-time setup
make dev            # Start development
make test           # Run tests

# Database
make db-push        # Push schema changes
make db-studio      # Open Prisma Studio

# Docker
docker compose up -d        # Start services
docker compose logs -f      # View logs
docker compose down         # Stop services

# Environment
pnpm env:local      # Switch to local
pnpm env:cloud      # Switch to cloud
```

---

## Next Steps

1. **Custom Domain**: Add in Vercel Settings → Domains
2. **SSL**: Automatic with Vercel
3. **Monitoring**: Add Sentry for error tracking
4. **Email**: Configure Mailtrap or SendGrid

---

## Need Help?

- [Full Deployment Guide](./FREE_DEPLOYMENT_GUIDE.md)
- [CI/CD Guide](./CI_CD_GUIDE.md)
- [Troubleshooting](../runbooks/COMMON_ISSUES.md)
- [Incident Response](../runbooks/INCIDENT_RESPONSE.md)

