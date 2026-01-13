# Free Third-Party Services Setup Guide

> **New**: CareCircle now includes RabbitMQ for event-driven architecture. See [EVENT_DRIVEN_ARCHITECTURE.md](./EVENT_DRIVEN_ARCHITECTURE.md) for details.

This guide explains how to set up FREE third-party services for CareCircle development. These services provide generous free tiers perfect for development and small-scale production.

## Quick Reference

| Service | Free Tier | Use Case |
|---------|-----------|----------|
| **Neon** | 0.5 GB storage, always free | PostgreSQL Database |
| **Upstash** | 10k commands/day | Redis Cache/Queue |
| **CloudAMQP** | 1M messages/month | RabbitMQ (Event Bus) |
| **Cloudinary** | 25 GB storage, 25k transforms | File Storage |
| **Mailtrap** | 1,000 emails/month | Email Testing |
| **Resend** | 3,000 emails/month | Production Email |
| **Firebase** | Generous free tier | Push Notifications |
| **Sentry** | 5,000 errors/month | Error Tracking |

---

## 1. Database - Neon (PostgreSQL)

**Free Tier**: 0.5 GB storage, auto-suspend after 5 min inactivity

### Setup Steps:

1. Go to [neon.tech](https://neon.tech)
2. Sign up with GitHub
3. Create a new project
4. Copy the connection string

### Configuration:

```env
DB_HOST=ep-xxx-xxx-123456.us-east-1.aws.neon.tech
DB_PORT=5432
DB_USERNAME=your-username
DB_PASSWORD=your-password
DB_DATABASE=carecircle
DB_SSL=true
```

### Alternative: Supabase

1. Go to [supabase.com](https://supabase.com)
2. Create a new project
3. Go to Settings → Database
4. Copy connection details

```env
DB_HOST=db.xxxxx.supabase.co
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=your-password
DB_DATABASE=postgres
DB_SSL=true
```

---

## 2. Redis - Upstash

**Free Tier**: 10,000 commands/day, 256 MB storage

### Setup Steps:

1. Go to [upstash.com](https://upstash.com)
2. Sign up with GitHub
3. Create a Redis database
4. Select a region close to your users

### Configuration:

```env
REDIS_HOST=xxx-xxx.upstash.io
REDIS_PORT=6379
REDIS_PASSWORD=your-upstash-password
REDIS_TLS=true
```

### Notes:
- Upstash uses TLS by default
- Data persists even on free tier
- Great for both cache and BullMQ queues

---

## 3. Message Queue - CloudAMQP (RabbitMQ)

**Free Tier**: 1M messages/month, 20 connections, 100 queues

### Setup Steps:

1. Go to [cloudamqp.com](https://www.cloudamqp.com)
2. Sign up with GitHub or email
3. Create a new instance (select "Little Lemur" - FREE)
4. Choose a region close to your users
5. Copy the AMQP URL from the dashboard

### Configuration:

```env
RABBITMQ_URL=amqps://username:password@jackrabbit.rmq.cloudamqp.com/vhost
```

### Management UI:

CloudAMQP provides a hosted RabbitMQ Management UI:
1. Go to your instance dashboard
2. Click "RabbitMQ Manager" button
3. View queues, exchanges, and message rates

### Features:
- Managed RabbitMQ cluster
- Automatic backups
- TLS by default
- Web-based management UI
- Message rate monitoring

### Why RabbitMQ for CareCircle?

CareCircle uses RabbitMQ for:
- **Reliable event delivery** - Emergency alerts MUST be delivered
- **Async processing** - Medication reminders, email sending
- **Decoupled services** - WebSocket gateway receives events from message bus
- **Audit logging** - All events are captured for compliance

See [EVENT_DRIVEN_ARCHITECTURE.md](./EVENT_DRIVEN_ARCHITECTURE.md) for full details.

---

## 4. File Storage - Cloudinary

**Free Tier**: 25 GB storage, 25,000 transformations/month

### Setup Steps:

1. Go to [cloudinary.com](https://cloudinary.com)
2. Sign up for free
3. Go to Dashboard → Settings
4. Copy your credentials

### Configuration:

```env
STORAGE_PROVIDER=cloudinary
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=123456789012345
CLOUDINARY_API_SECRET=your-api-secret
CLOUDINARY_FOLDER=carecircle
```

### Features:
- Automatic image optimization
- On-the-fly transformations
- CDN delivery
- Upload presets for security

### Example Upload Preset (Recommended):

1. Go to Settings → Upload
2. Add upload preset:
   - Name: `carecircle_documents`
   - Signing Mode: Signed
   - Folder: `carecircle`
   - Allowed formats: jpg, png, pdf, doc, docx

---

## 5. Email - Mailtrap (Development)

**Free Tier**: 1,000 emails/month, unlimited inboxes

### Setup Steps:

1. Go to [mailtrap.io](https://mailtrap.io)
2. Sign up for free
3. Go to Email Testing → Inboxes
4. Click on your inbox → API tab
5. Copy the API token

### Configuration:

```env
MAIL_PROVIDER=mailtrap
MAILTRAP_TOKEN=your-api-token
MAILTRAP_INBOX_ID=your-inbox-id
```

### Why Mailtrap:
- Emails never reach real addresses
- Visual email preview
- HTML/spam analysis
- Perfect for development

---

## 6. Email - Resend (Production)

**Free Tier**: 3,000 emails/month, 100 emails/day

### Setup Steps:

1. Go to [resend.com](https://resend.com)
2. Sign up with GitHub
3. Verify your domain (or use @resend.dev for testing)
4. Create an API key

### Configuration:

```env
MAIL_PROVIDER=resend
RESEND_API_KEY=re_xxxxxxxxxxxxx
MAIL_FROM=noreply@yourdomain.com
```

### Domain Setup:
1. Add DNS records provided by Resend
2. Wait for verification (usually < 24 hours)
3. Update MAIL_FROM to use your domain

---

## 7. Email - Brevo (Alternative)

**Free Tier**: 300 emails/day, unlimited contacts

### Setup Steps:

1. Go to [brevo.com](https://brevo.com)
2. Sign up for free
3. Go to SMTP & API → API Keys
4. Create a new API key

### Configuration:

```env
MAIL_PROVIDER=brevo
BREVO_API_KEY=xkeysib-xxxxxxxxxxxxx
```

---

## 8. Push Notifications - Firebase

**Free Tier**: Generous (practically unlimited for most apps)

### Setup Steps:

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Create a new project
3. Go to Project Settings → Service Accounts
4. Generate a new private key (JSON)

### Configuration:

```env
FIREBASE_SERVICE_ACCOUNT={"type":"service_account","project_id":"your-project",...}
```

### Frontend Setup:
1. Add Firebase SDK to your web app
2. Request notification permission
3. Send FCM token to your backend

---

## 9. Error Tracking - Sentry

**Free Tier**: 5,000 errors/month, 1 user

### Setup Steps:

1. Go to [sentry.io](https://sentry.io)
2. Sign up for free
3. Create a new project (NestJS)
4. Copy the DSN

### Configuration:

```env
SENTRY_DSN=https://xxxxx@xxxxx.ingest.sentry.io/xxxxx
```

### Integration:

```typescript
// apps/api/src/main.ts
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
});
```

---

## 10. Analytics - PostHog (Optional)

**Free Tier**: 1M events/month

### Setup Steps:

1. Go to [posthog.com](https://posthog.com)
2. Sign up for free
3. Create a project
4. Copy your API key

### Configuration:

```env
POSTHOG_API_KEY=phc_xxxxxxxxxxxxx
POSTHOG_HOST=https://app.posthog.com
```

---

## Local Development Stack

For fully offline development, use Docker:

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: carecircle
      POSTGRES_PASSWORD: carecircle
      POSTGRES_DB: carecircle

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  mailpit:
    image: axllent/mailpit
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Web UI

  minio:
    image: minio/minio
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    command: server /data --console-address ":9001"
```

---

## Production Recommendations

When moving to production, consider:

| Development | Production |
|------------|------------|
| Neon Free | Neon Pro / AWS RDS |
| Upstash Free | Upstash Pro / AWS ElastiCache |
| Cloudinary Free | Cloudinary Plus / AWS S3 |
| Mailtrap | Resend / SendGrid |
| Firebase FCM | Firebase FCM (same) |
| Sentry Free | Sentry Team |

---

## Cost Estimates (Growth)

For a growing app with ~1,000 users:

| Service | Monthly Cost |
|---------|-------------|
| Database (Neon Pro) | $19 |
| Redis (Upstash Pro) | $10 |
| Storage (Cloudinary Plus) | $89 |
| Email (Resend Pro) | $20 |
| **Total** | **~$140/mo** |

---

## Getting Help

- **Neon**: [neon.tech/docs](https://neon.tech/docs)
- **Upstash**: [upstash.com/docs](https://upstash.com/docs)
- **Cloudinary**: [cloudinary.com/documentation](https://cloudinary.com/documentation)
- **Mailtrap**: [mailtrap.io/api](https://mailtrap.io/api)
- **Resend**: [resend.com/docs](https://resend.com/docs)
- **Firebase**: [firebase.google.com/docs](https://firebase.google.com/docs)

