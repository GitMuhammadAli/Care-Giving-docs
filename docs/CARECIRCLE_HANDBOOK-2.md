
## Auth & Security (Implementation-Level)

### Authentication Flow

**Registration → Verification → Login → Refresh**

```
┌───────────────────────────────────────────────────────────────────────────┐
│                    AUTHENTICATION FLOW DIAGRAM                             │
└───────────────────────────────────────────────────────────────────────────┘

1. REGISTRATION
   Frontend: POST /api/v1/auth/register
   Body: { email, password, fullName }
   │
   ├─→ API validates email format, password strength (min 8 chars, uppercase, number)
   ├─→ Hash password with bcrypt (cost factor 10)
   ├─→ Create User record (emailVerified=false, emailVerificationToken=UUID)
   ├─→ Send verification email via Mailtrap
   └─→ Return 201 Created (no tokens yet)

2. EMAIL VERIFICATION
   User clicks link: GET /api/v1/auth/verify-email?token=<UUID>
   │
   ├─→ API finds user by emailVerificationToken
   ├─→ Set emailVerified=true, emailVerificationToken=null
   ├─→ Redirect to /login with success message
   └─→ Return 200 OK

3. LOGIN
   Frontend: POST /api/v1/auth/login
   Body: { email, password }
   │
   ├─→ API finds user by email
   ├─→ Check emailVerified === true (else return 403 "Email not verified")
   ├─→ Compare password with bcrypt
   ├─→ Generate accessToken (JWT, 15min expiry, payload: { userId, email })
   ├─→ Generate refreshToken (UUID, 7 days expiry)
   ├─→ Hash refreshToken with sha256, store in Session table
   ├─→ Set HTTP-only cookies:
   │    ├─ accessToken (httpOnly, secure, sameSite=strict, maxAge=15min)
   │    └─ refreshToken (httpOnly, secure, sameSite=strict, maxAge=7days)
   └─→ Return 200 OK with user object

4. AUTHENTICATED REQUEST
   Frontend: GET /api/v1/users/me
   Headers: Cookie: accessToken=<JWT>
   │
   ├─→ JwtAuthGuard extracts JWT from cookie
   ├─→ Verify JWT signature with JWT_SECRET
   ├─→ Check expiration (if expired, return 401)
   ├─→ Attach user to request object
   └─→ Controller returns user data

5. TOKEN REFRESH
   Frontend: POST /api/v1/auth/refresh
   Headers: Cookie: refreshToken=<UUID>
   │
   ├─→ Extract refreshToken from cookie
   ├─→ Hash with sha256, lookup in Session table
   ├─→ Check expiresAt > now (else return 401 "Refresh token expired")
   ├─→ Delete old session (token rotation)
   ├─→ Generate NEW accessToken (15min)
   ├─→ Generate NEW refreshToken (7 days)
   ├─→ Create new Session record
   ├─→ Set new HTTP-only cookies
   └─→ Return 200 OK

6. LOGOUT
   Frontend: POST /api/v1/auth/logout
   Headers: Cookie: refreshToken=<UUID>
   │
   ├─→ Hash refreshToken, delete from Session table
   ├─→ Clear cookies (set maxAge=0)
   └─→ Return 200 OK
```

### Token Strategy

**Access Token (JWT):**
- **Format:** JSON Web Token (HS256 algorithm)
- **Secret:** `JWT_SECRET` from environment (256-bit random string)
- **Expiry:** 15 minutes
- **Payload:**
  ```json
  {
    "sub": "user-uuid",
    "email": "user@example.com",
    "iat": 1673456789,
    "exp": 1673457689
  }
  ```
- **Storage:** HTTP-only cookie (XSS protection)
- **Purpose:** Authorize API requests

**Refresh Token (Opaque):**
- **Format:** UUID v4 (random, non-predictable)
- **Storage (Client):** HTTP-only cookie
- **Storage (Server):** Hashed (SHA-256) in Session table
- **Expiry:** 7 days
- **Purpose:** Issue new access tokens without re-login

**Why This Strategy:**

| Requirement | Solution | Alternative (Why Not) |
|-------------|----------|----------------------|
| XSS Protection | HTTP-only cookies | localStorage (vulnerable to XSS) |
| CSRF Protection | SameSite=Strict | CSRF tokens (extra complexity) |
| Token Theft Mitigation | Short-lived access token (15min) | Long-lived tokens (higher risk) |
| Refresh Token Theft | Rotation (one-time use) | Reusable refresh tokens (replay attacks) |
| Logout Enforcement | Session table (revocable) | Stateless JWT (cannot revoke) |

**Security Controls:**

1. **Password Requirements:**
   - Minimum 8 characters
   - At least one uppercase letter
   - At least one number
   - At least one special character (enforced in frontend validation only, relaxed in backend)

2. **Hashing:**
   - **Passwords:** bcrypt with cost factor 10 (takes ~100ms to hash)
   - **Refresh tokens:** SHA-256 (fast, one-way)

3. **Rate Limiting:**
   - Login endpoint: 5 attempts per 15 minutes per IP
   - Register endpoint: 3 attempts per hour per IP
   - Password reset: 3 attempts per hour per email

4. **Session Management:**
   - Track IP address and user agent per session
   - Allow user to view and revoke active sessions
   - Auto-cleanup expired sessions (cron job, daily)

5. **Environment Secrets:**
   - **`JWT_SECRET`**: Generated with `openssl rand -base64 64`
   - **`JWT_REFRESH_SECRET`**: Separate secret for refresh tokens (if using JWT refresh)
   - **`ENCRYPTION_KEY`**: For encrypting sensitive data at rest (future use)

### How to Test Auth Quickly

**Smoke Test: Register → Login → Refresh → Logout**

```bash
BASE_URL="http://localhost:3001/api/v1"

# 1. Register
curl -X POST "$BASE_URL/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@carecircle.local",
    "password": "Test1234!",
    "fullName": "Test User"
  }'
# Expected: 201 Created

# 2. Login (skip email verification in dev mode if configured)
curl -X POST "$BASE_URL/auth/login" \
  -H "Content-Type: application/json" \
  -c cookies.txt \
  -d '{
    "email": "test@carecircle.local",
    "password": "Test1234!"
  }'
# Expected: 200 OK, cookies.txt contains accessToken and refreshToken

# 3. Access Protected Route
curl "$BASE_URL/users/me" \
  -b cookies.txt
# Expected: 200 OK, user object

# 4. Logout
curl -X POST "$BASE_URL/auth/logout" \
  -b cookies.txt
# Expected: 200 OK

# 5. Verify Logout
curl "$BASE_URL/users/me" \
  -b cookies.txt
# Expected: 401 Unauthorized
```

**Security Audit Checklist:**

- [ ] Passwords hashed with bcrypt (cost 10+)
- [ ] JWT signed with HS256 or RS256
- [ ] Access tokens expire within 1 hour (we use 15min)
- [ ] Refresh tokens stored hashed in database
- [ ] Refresh tokens rotate on use
- [ ] HTTP-only cookies prevent XSS
- [ ] SameSite=Strict prevents CSRF
- [ ] Rate limiting on auth endpoints
- [ ] Email verification required before login
- [ ] Password reset uses time-limited tokens
- [ ] Sessions track IP/user agent for anomaly detection
- [ ] HTTPS enforced in production (nginx config)

---

## Background Jobs (Workers + RabbitMQ)

### Synchronous vs Asynchronous

**Synchronous (Handled in API Request):**
- User registration (must return immediately)
- Login (must return immediately)
- Data queries (user expects instant response)
- File uploads (user waits for upload)

**Asynchronous (Handled by Workers):**
- Email sending (slow, can fail, user doesn't need to wait)
- Push notifications (broadcast to multiple users, not request-blocking)
- Scheduled reminders (medication, appointment)
- WebSocket broadcasts (decouple from HTTP request)
- Audit logging (compliance, not user-facing)

**Decision Rule:** If it takes >500ms, can fail independently, or must happen later, make it async.

### Event-Driven Architecture (RabbitMQ)

**Why RabbitMQ:**

| Requirement | Why RabbitMQ | Alternative (Why Not) |
|-------------|--------------|----------------------|
| At-least-once delivery | AMQP guarantees + outbox pattern | Redis Pub/Sub (fire-and-forget, no persistence) |
| Multiple consumers | Topic exchange routing | Kafka (overkill for <1M events/day) |
| Message persistence | Durable queues | BullMQ (Redis-based, no built-in HA) |
| Dead letter queues | Built-in DLX | SQS (vendor lock-in, higher cost) |
| Local dev simplicity | Docker Compose, mgmt UI | Kafka (requires Zookeeper, complex setup) |

**Tradeoffs:**
- **Pro:** Mature, reliable, familiar to most backend engineers
- **Pro:** Topic exchange enables flexible routing (e.g., send to WebSocket AND Notification consumers)
- **Con:** No native horizontal scaling (use CloudAMQP for clustering in cloud)
- **Con:** Requires learning AMQP concepts (exchanges, queues, bindings)

**RabbitMQ Topology:**

```
Exchange: domain.events (type: topic)
├─ Routing Key Pattern: <entity>.<action>.<status>.<familyId>
│  Examples:
│  ├─ medication.logged.given.family-uuid
│  ├─ emergency.alert.created.family-uuid
│  └─ appointment.reminder.scheduled.family-uuid
│
└─ Bindings:
   │
   ├─→ Queue: domain.websocket
   │   ├─ Binding: medication.* (all medication events)
   │   ├─ Binding: emergency.* (all emergency events)
   │   ├─ Binding: appointment.* (all appointment events)
   │   └─ Binding: shift.* (all shift events)
   │   Consumer: WebSocketConsumer
   │   └─→ Send event to Socket.io gateway → broadcast to family room
   │
   ├─→ Queue: domain.notifications
   │   ├─ Binding: medication.logged.* (medication logs)
   │   ├─ Binding: emergency.alert.created.* (emergency alerts)
   │   ├─ Binding: appointment.reminder.* (appointment reminders)
   │   └─ Binding: shift.*.* (all shift events)
   │   Consumer: NotificationConsumer
   │   └─→ Send push notifications to family members
   │
   └─→ Queue: domain.audit
       ├─ Binding: *.* (all events)
       Consumer: AuditConsumer
       └─→ Log to audit_log table for compliance
```

**Local Testing Workflow:**

```bash
# 1. Start RabbitMQ
docker compose up -d rabbitmq

# 2. Verify RabbitMQ is running
docker ps | grep rabbitmq

# 3. Access management UI
open http://localhost:15672
# Login: guest / guest

# 4. Start Workers
pnpm --filter @carecircle/workers dev

# 5. Trigger an event (via API)
curl -X POST http://localhost:3001/api/v1/medications/123/log \
  -H "Content-Type: application/json" \
  -H "Cookie: accessToken=<YOUR_JWT>" \
  -d '{"status": "GIVEN", "notes": "Test"}'

# 6. Check worker logs
# Should see event consumption and side effects
```

**Definition of Done (Background Jobs):**

- [ ] RabbitMQ container starts healthy
- [ ] Management UI accessible at `localhost:15672`
- [ ] Exchanges and queues auto-created on first message
- [ ] Workers connect and consume messages
- [ ] Outbox processor runs every 10 seconds
- [ ] Duplicate events are ignored (idempotency)
- [ ] WebSocket events propagate to frontend
- [ ] Push notifications deliver to subscribed devices
- [ ] Audit logs persist to database

---

## Storage + Email (Always Third-Party)

### Cloudinary (Document Storage)

**Why Cloudinary:**

| Requirement | Why Cloudinary | Alternative (Why Not) |
|-------------|----------------|----------------------|
| Image optimization | Auto-resize, format conversion (WebP) | S3 (requires manual optimization) |
| CDN delivery | Built-in global CDN | S3 + CloudFront (more config) |
| Zero maintenance | Managed service | MinIO (self-hosted, requires ops) |
| Free tier | 25GB storage, 25GB bandwidth | S3 (pay from $0.023/GB) |

**Usage Pattern:**

```typescript
// apps/api/src/system/module/storage/cloudinary.service.ts

async uploadDocument(file: Express.Multer.File, familyId: string): Promise<UploadResult> {
  return cloudinary.uploader.upload(file.path, {
    folder: `carecircle/${familyId}`,
    resource_type: 'auto',
    tags: ['document']
  });
}
```

**Environment Variables:**

```bash
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=123456789012345
CLOUDINARY_API_SECRET=<SECRET>
```

### Mailtrap (Email Delivery)

**Why Mailtrap:**

| Requirement | Why Mailtrap | Alternative (Why Not) |
|-------------|--------------|----------------------|
| Dev/staging testing | Inbox catches all emails, no real sends | Gmail SMTP (emails real users by mistake) |
| Production-ready | Mailtrap Send API (transactional emails) | Sendgrid (more expensive, complex) |
| Free tier | 500 emails/month (dev), 1000 emails/month (production) | SES (requires AWS account setup) |

**Environment Variables:**

```bash
# Development (Mailtrap Sandbox)
SMTP_HOST=sandbox.smtp.mailtrap.io
SMTP_PORT=2525
SMTP_USER=<MAILTRAP_INBOX_ID>
SMTP_PASS=<MAILTRAP_INBOX_PASSWORD>

# Production (Mailtrap Send API or SES)
SMTP_HOST=live.smtp.mailtrap.io
SMTP_PORT=587
SMTP_USER=api
SMTP_PASS=<MAILTRAP_API_TOKEN>
```

**Local Verification:**

```bash
# 1. Trigger email send
curl -X POST http://localhost:3001/api/v1/auth/register \
  -d '{"email": "test@example.com", "password": "Test1234!", "fullName": "Test User"}'

# 2. Check Mailtrap inbox
open https://mailtrap.io/inboxes

# 3. Verify email content contains verification link
```

---

## Profile A Runbook (Local Dev)

### Overview

**Goal:** Run infrastructure locally via Docker, run applications on host in development mode.

**Environment:** Windows/Mac/Linux developer machine

**Infrastructure Stack:**
- PostgreSQL: Docker container (port 5432)
- Redis: Docker container (port 6379)
- RabbitMQ: Docker container (ports 5672, 15672)
- Cloudinary: Third-party service (no local replacement)
- Mailtrap: Third-party service (no local replacement)

**Application Stack:**
- API: pnpm dev (hot reload, TypeScript watch mode)
- Web: pnpm dev (Next.js fast refresh)
- Workers: pnpm dev (nodemon auto-restart)

### Step-by-Step Setup

**Prerequisites:**

```bash
# Verify installations
node --version   # v20.x or later
pnpm --version   # v9.x or later
docker --version # v24.x or later
git --version    # v2.x or later
```

**Step 1: Clone Repository**

```bash
git clone https://github.com/your-org/carecircle.git
cd carecircle
```

**Step 2: Install Dependencies**

```bash
pnpm install
# Installs all workspace dependencies (~2-3 minutes)
```

**Step 3: Configure Environment (Local Profile)**

```bash
# Windows PowerShell
.\scripts\use-local.ps1

# Linux/Mac Bash
chmod +x scripts/use-local.sh
./scripts/use-local.sh
```

**What this script does:**
1. Merges `env/base.env` + `env/local.env` → `.env`
2. Copies `.env` to `apps/api/.env`, `apps/web/.env`, `apps/workers/.env`
3. Validates required environment variables exist

**Manual Environment Setup (if script fails):**

```bash
# Copy example files
cp env/base.env .env
cat env/local.env >> .env

# Edit .env and fill in third-party credentials:
# - CLOUDINARY_CLOUD_NAME, CLOUDINARY_API_KEY, CLOUDINARY_API_SECRET
# - SMTP_USER, SMTP_PASS (Mailtrap credentials)
# - JWT_SECRET, JWT_REFRESH_SECRET (generate with openssl)

# Generate secrets
openssl rand -base64 64  # Use for JWT_SECRET
openssl rand -base64 64  # Use for JWT_REFRESH_SECRET
openssl rand -hex 32     # Use for ENCRYPTION_KEY

# Copy to app directories
cp .env apps/api/.env
cp .env apps/web/.env
cp .env apps/workers/.env
```

**Step 4: Start Docker Infrastructure**

```bash
docker compose up -d

# Expected output:
# [+] Running 3/3
#  ✔ Container carecircle-postgres-1   Started
#  ✔ Container carecircle-redis-1      Started
#  ✔ Container carecircle-rabbitmq-1   Started
```

**Step 5: Verify Docker Services**

```bash
docker ps

# Should show 3 containers:
# - carecircle-postgres-1   (port 5432)
# - carecircle-redis-1      (port 6379)
# - carecircle-rabbitmq-1   (ports 5672, 15672)

# Check health
docker compose ps

# All containers should show "healthy" status
```

**Step 6: Initialize Database**

```bash
# Run TypeORM migrations
pnpm --filter @carecircle/api migration:run

# Expected output:
# query: SELECT * FROM "migrations"
# 0 migrations are already loaded in the database.
# 12 migrations were found in the source code.
# 12 migrations are new migrations that needs to be executed.
# Migration CreateUsers1234567890123 has been executed successfully.
# ...
# Migration CreateEventOutbox1234567899999 has been executed successfully.

# Verify tables created
docker exec -it carecircle-postgres-1 psql -U carecircle -d carecircle -c "\dt"

# Should list:user, family, family_member, family_invitation, care_recipient,
# medication, medication_log, appointment, document, emergency_alert,
# caregiver_shift, notification, push_subscription, session, event_outbox, etc.
```

**Step 7: Start API Server**

```bash
# Terminal 1
pnpm --filter @carecircle/api dev

# Expected output:
# [Nest] 12345  - 01/14/2026, 10:00:00 AM     LOG [NestFactory] Starting Nest application...
# [Nest] 12345  - 01/14/2026, 10:00:01 AM     LOG [InstanceLoader] DatabaseModule dependencies initialized
# [Nest] 12345  - 01/14/2026, 10:00:01 AM     LOG [InstanceLoader] ConfigModule dependencies initialized
# [Nest] 12345  - 01/14/2026, 10:00:02 AM     LOG [RoutesResolver] AuthController {/api/v1/auth}:
# [Nest] 12345  - 01/14/2026, 10:00:02 AM     LOG [RouterExplorer] Mapped {/api/v1/auth/register, POST} route
# ...
# [Nest] 12345  - 01/14/2026, 10:00:03 AM     LOG [NestApplication] Nest application successfully started
# [Nest] 12345  - 01/14/2026, 10:00:03 AM     LOG API Server running on http://localhost:3001
# [Nest] 12345  - 01/14/2026, 10:00:03 AM     LOG Swagger docs: http://localhost:3001/api
```

**Step 8: Verify API Health**

```bash
# Health check endpoint
curl http://localhost:3001/health

# Expected: {"status":"ok","database":"up","redis":"up","rabbitmq":"up"}

# Swagger docs
open http://localhost:3001/api

# Should show interactive API documentation
```

**Step 9: Start Web App**

```bash
# Terminal 2
pnpm --filter @carecircle/web dev

# Expected output:
#  ▲ Next.js 14.0.4
#  - Local:        http://localhost:3000
#  - Network:      http://192.168.1.10:3000
#
#  ✓ Ready in 2.3s
```

**Step 10: Start Workers**

```bash
# Terminal 3
pnpm --filter @carecircle/workers dev

# Expected output:
# [Workers] Starting scheduler...
# [Workers] Connecting to RabbitMQ: amqp://guest:guest@localhost:5672
# [Workers] RabbitMQ connected successfully
# [Workers] Scheduler started, cron jobs initialized
# [Workers] WebSocketConsumer listening on queue: domain.websocket
# [Workers] NotificationConsumer listening on queue: domain.notifications
# [Workers] AuditConsumer listening on queue: domain.audit
```

### Health Check Matrix

**Before running smoke tests, verify all services are healthy:**

| Service | Check Command | Expected Result |
|---------|--------------|-----------------|
| PostgreSQL | `docker exec carecircle-postgres-1 pg_isready -U carecircle` | `carecircle:5432 - accepting connections` |
| Redis | `docker exec carecircle-redis-1 redis-cli ping` | `PONG` |
| RabbitMQ | `curl -u guest:guest http://localhost:15672/api/overview` | JSON response with `{"node":"rabbit@..."}` |
| API | `curl http://localhost:3001/health` | `{"status":"ok","database":"up","redis":"up","rabbitmq":"up"}` |
| Web | `curl -I http://localhost:3000` | `HTTP/1.1 200 OK` |
| Workers | Check logs for "Scheduler started" | No errors in logs |

### Smoke Test Matrix

**Test each working feature according to the implementation checklist:**

#### 1. Authentication

```bash
# Register
curl -X POST http://localhost:3001/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@carecircle.local",
    "password": "Test1234!",
    "fullName": "Test User"
  }'
# ✅ Expected: 201 Created

# Login (skip email verification in dev if configured)
curl -X POST http://localhost:3001/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -c cookies.txt \
  -d '{
    "email": "test@carecircle.local",
    "password": "Test1234!"
  }'
# ✅ Expected: 200 OK, cookies.txt has tokens

# Access protected route
curl http://localhost:3001/api/v1/users/me -b cookies.txt
# ✅ Expected: 200 OK, user object

# Refresh token
curl -X POST http://localhost:3001/api/v1/auth/refresh -b cookies.txt -c cookies.txt
# ✅ Expected: 200 OK, new tokens

# Logout
curl -X POST http://localhost:3001/api/v1/auth/logout -b cookies.txt
# ✅ Expected: 200 OK
```

#### 2. Family Management

```bash
# Create family (requires auth)
curl -X POST http://localhost:3001/api/v1/families \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{"name": "Thompson Family"}'
# ✅ Expected: 201 Created, { id, name, createdById }

# Save family ID
FAMILY_ID="<family-id-from-response>"

# Send invitation
curl -X POST http://localhost:3001/api/v1/families/$FAMILY_ID/invitations \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "email": "member@example.com",
    "role": "CAREGIVER"
  }'
# ✅ Expected: 201 Created
# ✅ Check Mailtrap inbox for invitation email
```

#### 3. Care Recipient CRUD

```bash
# Create care recipient
curl -X POST http://localhost:3001/api/v1/families/$FAMILY_ID/care-recipients \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{
    "firstName": "Margaret",
    "lastName": "Thompson",
    "dateOfBirth": "1948-03-15",
    "gender": "FEMALE",
    "bloodType": "O+",
    "allergies": ["Penicillin", "Shellfish"],
    "conditions": ["Diabetes Type 2", "Hypertension"]
  }'
# ✅ Expected: 201 Created

# Save care recipient ID
CR_ID="<care-recipient-id>"

# List care recipients
curl http://localhost:3001/api/v1/families/$FAMILY_ID/care-recipients -b cookies.txt
# ✅ Expected: 200 OK, array with created care recipient

# Update care recipient
curl -X PUT http://localhost:3001/api/v1/families/$FAMILY_ID/care-recipients/$CR_ID \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{"notes": "Lives with daughter Sarah"}'
# ✅ Expected: 200 OK
```

#### 4. Authorization Enforcement

```bash
# Register second user (different family)
curl -X POST http://localhost:3001/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "other@example.com",
    "password": "Test1234!",
    "fullName": "Other User"
  }'

# Login as second user
curl -X POST http://localhost:3001/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -c cookies2.txt \
  -d '{"email": "other@example.com", "password": "Test1234!"}'

# Try to access first user's family
curl http://localhost:3001/api/v1/families/$FAMILY_ID/care-recipients -b cookies2.txt
# ✅ Expected: 403 Forbidden (not a family member)
```

#### 5. Real-Time Events

**Setup:**
1. Open browser: `http://localhost:3000`
2. Login and navigate to Medications page
3. Open browser console (F12)

**Test:**
```bash
# Trigger medication log event via API
curl -X POST http://localhost:3001/api/v1/medications/$MED_ID/log \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{"status": "GIVEN", "notes": "Took with breakfast"}'

# ✅ Check browser console: Should see WebSocket event received
# ✅ Check UI: Medication card should update in real-time
# ✅ Check worker logs: Event consumed by NotificationConsumer
```

### Definition of Done (Local Dev)

**Infrastructure:**
- [ ] Docker containers running: PostgreSQL, Redis, RabbitMQ
- [ ] All containers show "healthy" status
- [ ] RabbitMQ management UI accessible (localhost:15672)
- [ ] Database migrations applied successfully
- [ ] All tables exist in PostgreSQL

**Applications:**
- [ ] API starts on port 3001 without errors
- [ ] Swagger docs accessible (localhost:3001/api)
- [ ] Web starts on port 3000 without errors
- [ ] Workers connect to RabbitMQ and start consumers
- [ ] No TypeScript compilation errors

**Features:**
- [ ] Register → Login → Refresh → Logout passes
- [ ] Family creation passes
- [ ] Family invitation sends email (visible in Mailtrap)
- [ ] Care recipient CRUD passes
- [ ] Cross-family access returns 403 Forbidden
- [ ] Real-time WebSocket events propagate to frontend
- [ ] Background jobs consume events from RabbitMQ

**Common Issues & Fixes:**

| Issue | Symptom | Fix |
|-------|---------|-----|
| Port conflict | `Error: listen EADDRINUSE: address already in use :::3001` | Kill process: `npx kill-port 3001` |
| PostgreSQL not ready | `Error: Connection terminated unexpectedly` | Wait 10s for container startup, or check `docker logs carecircle-postgres-1` |
| Redis connection failed | `Error: ECONNREFUSED localhost:6379` | Check `docker ps`, restart: `docker compose restart redis` |
| RabbitMQ connection failed | `Error: Socket closed abruptly` | Check `docker logs carecircle-rabbitmq-1`, verify guest/guest credentials |
| Missing env variable | `Error: JWT_SECRET is required` | Run `.\scripts\use-local.ps1` again, verify `.env` file exists |
| Migration failed | `QueryFailedError: relation "user" already exists` | Database already migrated, safe to ignore. Or: `pnpm --filter @carecircle/api migration:revert` then re-run |

---

## Profile B Runbook (Cloud Deploy)

### Overview

**Goal:** Deploy all services to cloud/production with managed infrastructure.

**Environment:** Cloud provider (AWS, GCP, Azure, or multi-cloud)

**Infrastructure Stack (Managed Services):**
- PostgreSQL: Neon (or AWS RDS, Google Cloud SQL)
- Redis: Upstash (or AWS ElastiCache, Google Memorystore)
- RabbitMQ: CloudAMQP (or AWS AmazonMQ)
- Storage: Cloudinary (same as local)
- Email: Mailtrap Send API or AWS SES
- Container Registry: Docker Hub, AWS ECR, or Google Artifact Registry

**Application Stack (Containerized):**
- API: Docker container (4 replicas)
- Web: Docker container (3 replicas)
- Workers: Docker container (3 replicas)
- Nginx: Load balancer/reverse proxy

**Deployment Strategy:**
- **Docker Compose:** Single-server deployment (good for <10K users)
- **Kubernetes:** Multi-node cluster (required for >100K users)

### Docker Compose Production Deployment

**Prerequisites:**

1. **Cloud Infrastructure Setup:**

```bash
# Neon (PostgreSQL)
# Sign up: https://neon.tech
# Create database: "carecircle-prod"
# Get connection string: postgresql://user:pass@ep-xxx.neon.tech/carecircle-prod?sslmode=require
NEON_URL="<connection-string>"

# Upstash (Redis)
# Sign up: https://upstash.com
# Create database: "carecircle-prod"
# Get connection string: rediss://default:pass@global-xxx.upstash.io:6379
UPSTASH_URL="<connection-string>"

# CloudAMQP (RabbitMQ)
# Sign up: https://www.cloudamqp.com
# Create instance: "carecircle-prod" (Little Lemur plan, $0/month)
# Get URL: amqps://user:pass@xxx.cloudamqp.com/xxx
CLOUDAMQP_URL="<connection-string>"
```

2. **Server Setup:**

```bash
# Provision server (example: AWS EC2 t3.2xlarge, 8 vCPU, 32GB RAM)
# OS: Ubuntu 22.04 LTS
# Security groups: Allow 80 (HTTP), 443 (HTTPS), 22 (SSH)

# SSH into server
ssh ubuntu@<server-ip>

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
newgrp docker

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.23.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

**Step 1: Clone Repository**

```bash
git clone https://github.com/your-org/carecircle.git
cd carecircle
```

**Step 2: Configure Production Environment**

```bash
# Copy example file
cp .env.prod.example .env.prod

# Edit with production values
nano .env.prod
```

**.env.prod (Production Environment):**

```bash
# ============================================================
# Database (Neon)
# ============================================================
DATABASE_URL=<NEON_URL>
POSTGRES_USER=neon_user
POSTGRES_PASSWORD=<NEON_PASSWORD>
POSTGRES_DB=carecircle

# ============================================================
# Redis (Upstash)
# ============================================================
REDIS_URL=<UPSTASH_URL>
REDIS_PASSWORD=<UPSTASH_PASSWORD>

# ============================================================
# RabbitMQ (CloudAMQP)
# ============================================================
RABBITMQ_URL=<CLOUDAMQP_URL>
RABBITMQ_USER=cloudamqp_user
RABBITMQ_PASSWORD=<CLOUDAMQP_PASSWORD>
RABBITMQ_VHOST=cloudamqp_vhost

# ============================================================
# Storage (Cloudinary)
# ============================================================
CLOUDINARY_CLOUD_NAME=<YOUR_CLOUD_NAME>
CLOUDINARY_API_KEY=<YOUR_API_KEY>
CLOUDINARY_API_SECRET=<YOUR_API_SECRET>

# ============================================================
# Email (Mailtrap Send or SES)
# ============================================================
SMTP_HOST=live.smtp.mailtrap.io
SMTP_PORT=587
SMTP_USER=api
SMTP_PASS=<MAILTRAP_API_TOKEN>

# ============================================================
# JWT & Security (GENERATE NEW SECRETS!)
# ============================================================
JWT_SECRET=<openssl rand -base64 64>
JWT_REFRESH_SECRET=<openssl rand -base64 64>
ENCRYPTION_KEY=<openssl rand -hex 32>

# ============================================================
# Push Notifications (VAPID)
# ============================================================
# Generate: npx web-push generate-vapid-keys
VAPID_PUBLIC_KEY=<VAPID_PUBLIC_KEY>
VAPID_PRIVATE_KEY=<VAPID_PRIVATE_KEY>
VAPID_SUBJECT=mailto:admin@your-domain.com

# ============================================================
# MinIO (S3-compatible storage)
# ============================================================
MINIO_ROOT_USER=<MINIO_USER>
MINIO_ROOT_PASSWORD=<MINIO_PASSWORD>
S3_BUCKET=carecircle-documents

# ============================================================
# Application URLs
# ============================================================
NEXT_PUBLIC_API_URL=https://api.your-domain.com/api/v1
NEXT_PUBLIC_WS_URL=wss://api.your-domain.com
```

**Step 3: Build Docker Images**

```bash
# Windows PowerShell
.\scripts\deploy-prod.ps1 -Build

# Linux/Mac
./scripts/deploy-prod.sh build

# Or manually:
docker-compose -f docker-compose.prod.yml --env-file .env.prod build --parallel

# Expected output (15-20 minutes):
# [+] Building 1234.5s (67/67) FINISHED
#  => [api builder  1/11] FROM docker.io/library/node:20-alpine
#  => [api builder  2/11] WORKDIR /app
#  ...
#  => [web runner] COPY --from=builder /app/.next/standalone ./
#  => [workers runner] COPY --from=builder /app/apps/workers/dist ./dist
#  => => naming to docker.io/library/carecircle-api:latest
#  => => naming to docker.io/library/carecircle-web:latest
#  => => naming to docker.io/library/carecircle-workers:latest
```

**Step 4: Run Database Migrations**

```bash
# Temporary container to run migrations
docker run --rm \
  --env-file .env.prod \
  carecircle-api:latest \
  npm run migration:run

# Expected:
# Migration CreateUsers1234567890123 has been executed successfully.
# ...
# All migrations executed successfully.
```

**Step 5: Start All Services**

```bash
# Windows
.\scripts\deploy-prod.ps1 -Up

# Linux/Mac
./scripts/deploy-prod.sh up

# Or manually:
docker-compose -f docker-compose.prod.yml --env-file .env.prod up -d

# Expected output:
# [+] Running 9/9
#  ✔ Network carecircle-prod_default    Created
#  ✔ Volume carecircle-prod_rabbitmq_data  Created
#  ✔ Container carecircle-prod-nginx-1      Started
#  ✔ Container carecircle-prod-api-1        Started
#  ✔ Container carecircle-prod-api-2        Started
#  ✔ Container carecircle-prod-api-3        Started
#  ✔ Container carecircle-prod-api-4        Started
#  ✔ Container carecircle-prod-web-1        Started
#  ✔ Container carecircle-prod-web-2        Started
#  ✔ Container carecircle-prod-web-3        Started
#  ✔ Container carecircle-prod-workers-1    Started
#  ✔ Container carecircle-prod-workers-2    Started
#  ✔ Container carecircle-prod-workers-3    Started
```

**Step 6: Verify Deployment**

```bash
# Check all containers are healthy
docker-compose -f docker-compose.prod.yml ps

# Should show "healthy" for all services

# Check logs
docker-compose -f docker-compose.prod.yml logs -f

# Test health endpoint
curl http://localhost/health

# Expected: {"status":"ok","database":"up","redis":"up","rabbitmq":"up"}
```

**Step 7: SSL/TLS Setup (Production)**

```bash
# Install certbot
sudo apt-get update
sudo apt-get install -y certbot

# Obtain SSL certificate (Let's Encrypt)
sudo certbot certonly --standalone -d api.your-domain.com -d your-domain.com

# Certificates saved to:
# /etc/letsencrypt/live/your-domain.com/fullchain.pem
# /etc/letsencrypt/live/your-domain.com/privkey.pem

# Copy certificates to project
sudo cp /etc/letsencrypt/live/your-domain.com/fullchain.pem certs/
sudo cp /etc/letsencrypt/live/your-domain.com/privkey.pem certs/

# Uncomment SSL config in nginx.conf
# Restart nginx
docker-compose -f docker-compose.prod.yml restart nginx
```

**Step 8: DNS Configuration**

```bash
# Add A records in your DNS provider:
# api.your-domain.com   → <SERVER_IP>
# your-domain.com       → <SERVER_IP>

# Verify DNS propagation
dig api.your-domain.com
dig your-domain.com

# Test HTTPS
curl https://api.your-domain.com/health
curl https://your-domain.com
```

**Step 9: Smoke Tests (Production)**

```bash
# Use production URLs
BASE_URL="https://api.your-domain.com/api/v1"

# Register
curl -X POST $BASE_URL/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@your-domain.com",
    "password": "SecurePass123!",
    "fullName": "Admin User"
  }'

# Login
curl -X POST $BASE_URL/auth/login \
  -H "Content-Type: application/json" \
  -c cookies.txt \
  -d '{"email": "admin@your-domain.com", "password": "SecurePass123!"}'

# Create family
curl -X POST $BASE_URL/families \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d '{"name": "Production Test Family"}'

# ✅ All endpoints should return 2xx responses
```

### Definition of Done (Cloud Deploy)

**Infrastructure:**
- [ ] Neon database accessible, migrations applied
- [ ] Upstash Redis responds to PING
- [ ] CloudAMQP queues created and accepting messages
- [ ] SSL certificates installed and valid (verify: https://www.ssllabs.com/ssltest/)
- [ ] DNS records propagated (A records point to server IP)

**Application Deployment:**
- [ ] All Docker images built successfully
- [ ] All containers start and show "healthy" status
- [ ] Nginx serves Web app at root domain
- [ ] Nginx proxies API requests to /api/*
- [ ] WebSocket connections established (wss://)
- [ ] Static assets cached (check response headers for Cache-Control)

**Smoke Tests:**
- [ ] Register → Login → Logout passes
- [ ] Family creation and invitation passes
- [ ] Care recipient CRUD passes
- [ ] Real-time events propagate
- [ ] Push notifications deliver
- [ ] Email sending works (check Mailtrap or SES dashboard)

**Performance:**
- [ ] API response time <200ms (p95)
- [ ] Web page load time <2s (lighthouse score >90)
- [ ] WebSocket latency <100ms
- [ ] No memory leaks (check `docker stats` after 24h uptime)

**Security:**
- [ ] HTTPS enforced (HTTP redirects to HTTPS)
- [ ] HTTP-only cookies set
- [ ] Rate limiting active (test: 10 login attempts should block IP)
- [ ] CORS configured (only allow your-domain.com)
- [ ] No secrets in Docker images (verify: `docker history carecircle-api | grep SECRET` returns empty)

---

## Environment Contract & Governance

### Canonical Environment Variables

**All environment variables with their meanings, requirements, and defaults:**

| Variable | Required | Profile A (Local) | Profile B (Cloud) | Purpose |
|----------|----------|-------------------|-------------------|---------|
| **Database** ||||
| `DATABASE_URL` | ✅ | `postgresql://carecircle:1234@localhost:5432/carecircle` | `<NEON_URL>?sslmode=require` | PostgreSQL connection string |
| `POSTGRES_USER` | ✅ | `carecircle` | `neon_user` | DB username |
| `POSTGRES_PASSWORD` | ✅ | `1234` | `<NEON_PASSWORD>` | DB password |
| `POSTGRES_DB` | ✅ | `carecircle` | `carecircle` | DB name |
| **Redis** ||||
| `REDIS_URL` | ✅ | `redis://localhost:6379` | `<UPSTASH_URL>` | Redis connection string |
| `REDIS_PASSWORD` | ❌ | *(empty)* | `<UPSTASH_PASSWORD>` | Redis password |
| **RabbitMQ** ||||
| `RABBITMQ_URL` | ✅ | `amqp://guest:guest@localhost:5672` | `<CLOUDAMQP_URL>` | RabbitMQ connection string |
| `RABBITMQ_USER` | ❌ | `guest` | `cloudamqp_user` | RabbitMQ username |
| `RABBITMQ_PASSWORD` | ❌ | `guest` | `<CLOUDAMQP_PASSWORD>` | RabbitMQ password |
| `RABBITMQ_VHOST` | ❌ | `/` | `cloudamqp_vhost` | RabbitMQ virtual host |
| **Storage** ||||
| `CLOUDINARY_CLOUD_NAME` | ✅ | `<YOUR_CLOUD>` | `<YOUR_CLOUD>` | Cloudinary account |
| `CLOUDINARY_API_KEY` | ✅ | `<YOUR_KEY>` | `<YOUR_KEY>` | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | ✅ | `<YOUR_SECRET>` | `<YOUR_SECRET>` | Cloudinary API secret |
| `S3_BUCKET` | ❌ | `carecircle-dev` | `carecircle-prod` | Default bucket name |
| **Email** ||||
| `SMTP_HOST` | ✅ | `sandbox.smtp.mailtrap.io` | `live.smtp.mailtrap.io` | SMTP server |
| `SMTP_PORT` | ✅ | `2525` | `587` | SMTP port |
| `SMTP_USER` | ✅ | `<MAILTRAP_INBOX_ID>` | `api` | SMTP username |
| `SMTP_PASS` | ✅ | `<MAILTRAP_INBOX_PASSWORD>` | `<MAILTRAP_API_TOKEN>` | SMTP password |
| **Security** ||||
| `JWT_SECRET` | ✅ | `<openssl rand -base64 64>` | `<openssl rand -base64 64>` | JWT signing secret (256-bit) |
| `JWT_REFRESH_SECRET` | ✅ | `<openssl rand -base64 64>` | `<openssl rand -base64 64>` | Refresh token secret |
| `ENCRYPTION_KEY` | ✅ | `<openssl rand -hex 32>` | `<openssl rand -hex 32>` | Data encryption key |
| **Push Notifications** ||||
| `VAPID_PUBLIC_KEY` | ✅ | `<npx web-push generate-vapid-keys>` | `<same>` | VAPID public key |
| `VAPID_PRIVATE_KEY` | ✅ | `<npx web-push generate-vapid-keys>` | `<same>` | VAPID private key |
| `VAPID_SUBJECT` | ✅ | `mailto:dev@carecircle.local` | `mailto:admin@your-domain.com` | VAPID subject (email) |
| **Application** ||||
| `NODE_ENV` | ✅ | `development` | `production` | Environment mode |
| `API_PORT` | ❌ | `3001` | `3001` | API server port |
| `WEB_PORT` | ❌ | `3000` | `3000` | Web server port |
| `FRONTEND_URL` | ✅ | `http://localhost:3000` | `https://your-domain.com` | Frontend URL (for CORS, emails) |
| `NEXT_PUBLIC_API_URL` | ✅ | `http://localhost:3001/api/v1` | `https://api.your-domain.com/api/v1` | API URL (client-side) |
| `NEXT_PUBLIC_WS_URL` | ✅ | `ws://localhost:3001` | `wss://api.your-domain.com` | WebSocket URL (client-side) |

### Environment File Locations & Precedence

```
carecircle/
├── .env                          # Root env file (GENERATED, do not edit manually)
├── .env.example                  # Template for local development
├── .env.prod.example             # Template for production
│
├── env/                          # Profile sources (edit these)
│   ├── base.env                 # Shared config (JWT, Cloudinary, Mailtrap)
│   ├── local.env                # Local overrides (Docker localhost URLs)
│   └── cloud.env                # Cloud overrides (Neon, Upstash, CloudAMQP URLs)
│
├── apps/api/.env                # Copied from root (GENERATED)
├── apps/web/.env                # Copied from root (GENERATED)
└── apps/workers/.env            # Copied from root (GENERATED)
```

**Precedence Rules:**

1. **Profile scripts merge files:**
   - `use-local.ps1`: Merges `base.env` + `local.env` → `.env`
   - `use-cloud.ps1`: Merges `base.env` + `cloud.env` → `.env`

2. **Apps read from their local .env:**
   - API: `apps/api/.env` (fallback to root `.env`)
   - Web: `apps/web/.env` (fallback to root `.env`)
   - Workers: `apps/workers/.env` (fallback to root `.env`)

3. **Docker Compose uses:**
   - Development: `.env`
   - Production: `.env.prod` (explicitly specified with `--env-file .env.prod`)

**Preventing Drift:**

| Problem | Solution |
|---------|----------|
| Manually editing `.env` instead of profiles | Add `.env` to `.gitignore`, regenerate with `use-local.ps1` |
| Apps get different env values | Profile scripts copy `.env` to all apps, ensuring consistency |
| Forgot to copy after editing `env/base.env` | Profile scripts automatically copy on every run |
| Production uses dev values | Separate `.env.prod` file, never commit to Git |
| Missing required variable | Runtime validation (see below) |

### Runtime Environment Validation

**Strategy:** Fail fast on startup if required environment variables are missing or invalid.

**Implementation (Zod Schema Validation):**

```typescript
// apps/api/src/config/env.validation.ts

import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url(),
  RABBITMQ_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  JWT_REFRESH_SECRET: z.string().min(32),
  ENCRYPTION_KEY: z.string().length(64), // 32 bytes hex = 64 chars
  CLOUDINARY_CLOUD_NAME: z.string().min(1),
  CLOUDINARY_API_KEY: z.string().min(1),
  CLOUDINARY_API_SECRET: z.string().min(1),
  SMTP_HOST: z.string().min(1),
  SMTP_PORT: z.coerce.number().int().positive(),
  SMTP_USER: z.string().min(1),
  SMTP_PASS: z.string().min(1),
  VAPID_PUBLIC_KEY: z.string().startsWith('B'),
  VAPID_PRIVATE_KEY: z.string().min(1),
  VAPID_SUBJECT: z.string().email(),
});

export function validateEnv() {
  try {
    envSchema.parse(process.env);
    console.log('✅ Environment validation passed');
  } catch (error) {
    console.error('❌ Environment validation failed:');
    console.error(error.errors);
    process.exit(1); // Fail fast, don't start the app
  }
}

// Call in main.ts BEFORE bootstrapping Nest
validateEnv();
```

**Benefits:**
- **Catches errors early:** Before any code runs, not after a user hits an endpoint
- **Clear error messages:** Shows exactly which variable is missing or invalid
- **Type safety:** Zod infers types, provides autocomplete in IDE
- **Documentation:** Schema serves as single source of truth for required env vars

---

## Decision Log

### Why We Chose This Stack

**For each major component, explain: what we chose, why, tradeoffs, and alternatives.**

#### NestJS (Backend Framework)

**What:** NestJS is an opinionated, TypeScript-first framework for building server-side applications.

**Why:**
- **Enterprise-grade:** Dependency injection, modular architecture, decorators
- **TypeScript native:** First-class TypeScript support, no wrestling with types
- **Batteries included:** Guards, pipes, interceptors, filters, WebSocket gateway
- **Familiar patterns:** Similar to Angular (if team knows Angular) or Spring (if team knows Java)
- **Ecosystem:** Swagger auto-gen, TypeORM integration, Bull queues, etc.

**Tradeoffs:**
- **Pro:** Scales well to large teams, enforces patterns, reduces bikeshedding
- **Pro:** Great docs, active community, corporate backing (Trilon)
- **Con:** Opinionated (some devs prefer Express simplicity)
- **Con:** Slight learning curve for decorators/DI if coming from Express
- **Con:** Larger bundle size than Fastify (~20% slower than raw Express)

**Alternatives & Why Not:**

| Alternative | Why Not |
|-------------|---------|
| Express | Too barebones, no structure, team would reinvent NestJS patterns poorly |
| Fastify | Faster, but less opinionated, smaller ecosystem, no WebSocket gateway |
| Koa | Minimalist, requires more boilerplate, smaller community |
| Hono | Very fast, but immature, lacks enterprise features |
| tRPC | Great for monorepos, but ties frontend to backend types (over-coupling) |

**When to Reconsider:** If API becomes performance-critical (>10K req/s), consider migrating hot paths to Fastify while keeping Nest for structure.

---

#### TypeORM (Database ORM)

**What:** TypeORM is a TypeScript ORM that supports Active Record and Data Mapper patterns.

**Why:**
- **TypeScript-first:** Decorators for entities, type-safe queries
- **Migrations:** Built-in migration system with CLI
- **Relationships:** Easy to define one-to-many, many-to-many with decorators
- **Raw SQL escape hatch:** `queryBuilder` and `query()` for complex queries
- **NestJS integration:** `@nestjs/typeorm` package, seamless DI

**Tradeoffs:**
- **Pro:** Mature, widely used in NestJS ecosystem
- **Pro:** Active Record pattern is productive for CRUD
- **Con:** Query builder API is verbose for complex joins
- **Con:** N+1 query problem if not careful with `relations: ['user', 'family']`
- **Con:** Less trendy than Prisma (team morale risk if everyone wants Prisma)

**Alternatives & Why Not:**

| Alternative | Why Not |
|-------------|---------|
| Prisma | Better DX, but schema-first (not code-first), migrations are less flexible, no Active Record pattern |
| Drizzle ORM | Lightweight, type-safe, but new (risky), smaller ecosystem |
| Sequelize | Mature, but JavaScript-first, weaker TypeScript support |
| Knex.js | Query builder only, no ORM features, more boilerplate |

**When to Reconsider:** If we hit query performance issues, Prisma's query engine is faster. If we need multi-database support, Drizzle is more flexible.

**Note:** We have a `packages/database` folder with Prisma files (legacy), but we use TypeORM. This folder can be removed to avoid confusion.

---

#### RabbitMQ (Message Broker)

**What:** RabbitMQ is an AMQP message broker with routing, queues, and persistence.

**Why:**
- **Reliability:** At-least-once delivery, persistent queues, dead-letter exchanges
- **Routing:** Topic exchanges enable flexible routing (e.g., `medication.*` → WebSocket queue)
- **Operational maturity:** Used by millions, battle-tested, great docs
- **Local dev:** Easy to run in Docker, management UI at port 15672
- **Cloud options:** CloudAMQP (managed), AWS AmazonMQ, Google Cloud Pub/Sub adapter

**Tradeoffs:**
- **Pro:** Proven, reliable, scales to millions of messages
- **Pro:** Dead-letter queues for failed messages
- **Con:** Single-node bottleneck (use CloudAMQP clusters for HA)
- **Con:** AMQP protocol learning curve (exchanges, bindings, routing keys)
- **Con:** No built-in at-most-once semantics (must implement idempotency)

**Alternatives & Why Not:**

| Alternative | Why Not |
|-------------|---------|
| Redis Pub/Sub | Fire-and-forget, no persistence, messages lost if consumer down |
| BullMQ (Redis) | Great for job queues, but not for event broadcasting, no topic routing |
| Kafka | Overkill for <1M events/day, requires Zookeeper, complex ops |
| AWS SQS | Vendor lock-in, no topic routing (would need SNS + SQS), higher cost |
| Google Pub/Sub | Vendor lock-in, less flexible routing than RabbitMQ |

**When to Reconsider:** If event volume exceeds 10K events/second, migrate to Kafka for partitioning and log-based storage.

---

#### Redis (Cache & Session Store)

**What:** Redis is an in-memory key-value store used for caching and session management.

**Why:**
- **Performance:** Sub-millisecond latency, 100K+ ops/sec per node
- **Data structures:** Strings, hashes, lists, sets, sorted sets, pub/sub channels
- **TTL:** Automatic expiration for cache invalidation
- **NestJS integration:** `@nestjs/cache-manager` with Redis adapter
- **Upstash:** Serverless Redis with generous free tier (10K commands/day)

**Tradeoffs:**
- **Pro:** Industry standard, everyone knows Redis
- **Pro:** Upstash eliminates ops burden (no manual backups, scaling, patching)
- **Con:** Single-threaded (use Redis Cluster for horizontal scaling)
- **Con:** In-memory only (expensive at scale, use LRU eviction or persistence)
- **Con:** Upstash free tier limited (upgrade to $0.20/100K commands)

**Alternatives & Why Not:**

| Alternative | Why Not |
|-------------|---------|
| Memcached | Simpler, but no persistence, no data structures, no pub/sub |
| Self-hosted Redis | Requires ops overhead (backups, monitoring, scaling) |
| Dragonfly | Faster than Redis, but new, less tooling support |
| KeyDB | Redis fork with multithreading, but smaller community |

**When to Reconsider:** If cache hit rate is low (<80%), consider using Postgres materialized views instead.

---

#### Cloudinary (Document Storage)

**What:** Cloudinary is a media management SaaS with storage, CDN, and transformations.

**Why:**
- **Zero ops:** No S3 buckets to configure, no CloudFront setup
- **Transformations:** Resize, crop, format conversion on-the-fly
- **Free tier:** 25GB storage, 25GB bandwidth/month (good for MVP)
- **Security:** Signed URLs, folder-based access control
- **DX:** Simple SDK, great docs

**Tradeoffs:**
- **Pro:** Fastest time-to-market, no infrastructure setup
- **Pro:** Built-in image optimization (WebP, lazy loading)
- **Con:** Vendor lock-in (migration requires re-uploading all files)
- **Con:** Cost scales linearly ($0.10/GB beyond free tier)
- **Con:** Not suitable for large files (use S3 for video, backups)

**Alternatives & Why Not:**

| Alternative | Why Not |
|-------------|---------|
| AWS S3 | Requires VPC, IAM roles, CloudFront CDN setup, more config |
| MinIO | Self-hosted, requires ops, no CDN, no transformations |
| Backblaze B2 | Cheaper than S3, but no transformations, smaller ecosystem |
| Supabase Storage | Good for Supabase users, but we're not using Supabase |

**When to Reconsider:** If storage costs exceed $100/month, migrate to S3 + CloudFront + imgix (cheaper at scale).

---

#### Mailtrap (Email Testing & Sending)

**What:** Mailtrap is an email sandbox for testing and a transactional email API for production.

**Why:**
- **Dev safety:** Sandbox catches all emails, prevents accidental sends to real users
- **Free tier:** 500 emails/month (dev), 1000 emails/month (production)
- **Simple API:** SMTP or HTTP, no complex SDKs
- **Template editor:** WYSIWYG editor for email templates
- **Analytics:** Open/click tracking, bounce monitoring

**Tradeoffs:**
- **Pro:** Perfect for dev/staging, zero risk of emailing real users
- **Pro:** Fast to integrate, works with nodemailer
- **Con:** Free tier limited (scale to SES/Resend for >1K emails/month)
- **Con:** Less feature-rich than SendGrid (no A/B testing, segmentation)
- **Con:** SMTP slower than HTTP APIs (50-100ms per email)

**Alternatives & Why Not:**

| Alternative | Why Not |
|-------------|---------|
| AWS SES | Requires AWS account, exit sandbox mode (manual), more config |
| SendGrid | More expensive ($15/month for 40K emails), overkill for transactional |
| Resend | Better pricing ($0.01/100 emails), but newer, less proven |
| Postmark | Great for transactional, but $10/month minimum, no free tier |

**When to Reconsider:** If email volume exceeds 10K/month, migrate to Resend ($0.01/100 emails) or SES ($0.10/1000 emails).

---

## Troubleshooting Playbook

### Common Failure Modes

**If X breaks, check Y:**

#### API Won't Start

**Symptoms:**
- `Error: Cannot connect to database`
- `Error: ECONNREFUSED localhost:5432`
- App crashes immediately after `npm start`

**Diagnosis:**

```bash
# 1. Check DATABASE_URL in .env
cat apps/api/.env | grep DATABASE_URL

# 2. Test database connection directly
docker exec -it carecircle-postgres-1 psql -U carecircle -c "SELECT 1;"
# Expected: "1" (one row)

# 3. Check if database exists
docker exec -it carecircle-postgres-1 psql -U carecircle -l
# Should list "carecircle" database

# 4. Check PostgreSQL logs
docker logs carecircle-postgres-1 --tail 50

# 5. Verify TypeORM entities loaded
# Check apps/api/src/database/entities/ folder exists and has *.entity.ts files
```

**Fixes:**

| Cause | Fix |
|-------|-----|
| PostgreSQL not running | `docker compose up -d postgres` |
| Wrong password in DATABASE_URL | Edit `apps/api/.env`, restart API |
| Database doesn't exist | Create: `docker exec -it carecircle-postgres-1 psql -U postgres -c "CREATE DATABASE carecircle;"` |
| Migrations not run | `pnpm --filter @carecircle/api migration:run` |
| Port conflict (5432 in use) | Kill process: `npx kill-port 5432` or change port in docker-compose.yml |

---

#### Redis Connection Failed

**Symptoms:**
- `Error: connect ECONNREFUSED localhost:6379`
- Cache misses (slow API responses)
- Sessions not persisting (users logged out randomly)

**Diagnosis:**

```bash
# 1. Check REDIS_URL in .env
cat apps/api/.env | grep REDIS_URL

# 2. Test Redis connection
docker exec -it carecircle-redis-1 redis-cli ping
# Expected: "PONG"

# 3. Check Redis password (if set)
docker exec -it carecircle-redis-1 redis-cli -a <PASSWORD> ping

# 4. Check Redis logs
docker logs carecircle-redis-1 --tail 50

# 5. Check Redis memory usage
docker exec -it carecircle-redis-1 redis-cli INFO memory
```

**Fixes:**

| Cause | Fix |
|-------|-----|
| Redis not running | `docker compose up -d redis` |
| Wrong password | Edit `REDIS_PASSWORD` in .env, restart API |
| Redis out of memory | Check `maxmemory` policy in docker-compose.yml, or flush: `redis-cli FLUSHDB` |
| Port conflict (6379 in use) | Kill process: `npx kill-port 6379` |

---

#### RabbitMQ Connection Failed

**Symptoms:**
- `Error: Socket closed abruptly during opening handshake`
- Events not publishing (medication logs don't trigger notifications)
- Workers crash on startup

**Diagnosis:**

```bash
# 1. Check RABBITMQ_URL in .env
cat apps/workers/.env | grep RABBITMQ_URL

# 2. Check if RabbitMQ is running
docker ps | grep rabbitmq

# 3. Test RabbitMQ connection
curl -u guest:guest http://localhost:15672/api/overview
# Should return JSON with {"node":"rabbit@..."}

# 4. Check RabbitMQ logs
docker logs carecircle-rabbitmq-1 --tail 50

# 5. Check queues in management UI
open http://localhost:15672
# Login: guest / guest
```

**Fixes:**

| Cause | Fix |
|-------|-----|
| RabbitMQ not running | `docker compose up -d rabbitmq` |
| Wrong credentials | Default is `guest/guest`, edit .env if changed |
| Queues not auto-created | Publish one event via API to trigger auto-creation |
| Connection refused | Check RabbitMQ logs, may need to restart: `docker compose restart rabbitmq` |

---

#### WebSocket Connection Failed

**Symptoms:**
- Browser console: `WebSocket connection to 'ws://localhost:3001' failed`
- Real-time updates not working (medication logs don't update UI)
- `ERR_CONNECTION_REFUSED`

**Diagnosis:**

```bash
# 1. Check if API is running (WebSocket gateway is in API)
curl http://localhost:3001/health

# 2. Check NEXT_PUBLIC_WS_URL in web .env
cat apps/web/.env | grep NEXT_PUBLIC_WS_URL

# 3. Test WebSocket connection manually
npx wscat -c ws://localhost:3001/carecircle
# Should connect and show "connected"

# 4. Check API logs for WebSocket gateway errors
docker logs carecircle-prod-api-1 | grep -i websocket
```

**Fixes:**

| Cause | Fix |
|-------|-----|
| API not running | Start API: `pnpm --filter @carecircle/api dev` |
| Wrong WS URL in frontend | Edit `apps/web/.env`, set `NEXT_PUBLIC_WS_URL=ws://localhost:3001` |
| CORS blocking WebSocket | Check `apps/api/src/gateway/carecircle.gateway.ts`, ensure `cors: { origin: 'http://localhost:3000' }` |
| Nginx not proxying WebSocket | In nginx.conf, verify `/socket.io/` location has `proxy_set_header Upgrade $http_upgrade;` |

---

#### Build Failures

**Symptoms:**
- `docker build` fails
- `pnpm build` fails
- TypeScript compilation errors

**Diagnosis:**

```bash
# 1. Check for TypeScript errors
pnpm --filter @carecircle/api build

# 2. Check for missing dependencies
pnpm install

# 3. Check Docker build logs
docker build -f apps/api/Dockerfile . 2>&1 | tee build.log

# 4. Verify node_modules exists
ls apps/api/node_modules | head
```

**Fixes:**

| Cause | Fix |
|-------|-----|
| TypeScript errors | Fix errors shown in output, or run `tsc --noEmit` to see all |
| Missing dependencies | `pnpm install` |
| Out of memory (Docker) | Increase Docker memory: Settings → Resources → Memory → 8GB |
| Stale cache | `pnpm clean` → `rm -rf node_modules` → `pnpm install` |

---

## FAQ Pack

**25-30 high-signal Q&A entries:**

### Architecture & Design

**Q1: Why monorepo?**
A: Single repository for API, Web, Workers enables atomic changes (one PR updates backend + frontend), shared types without publishing packages, and simplified CI/CD. Alternatives (polyrepo) require coordinating versions across repos and publishing internal packages.

**Q2: Why pnpm instead of npm?**
A: pnpm is faster (3-5x install speed), uses less disk space (symlinks to global store), and prevents phantom dependencies (stricter than npm). Worth the small learning curve.

**Q3: How do profiles work (local vs cloud)?**
A: Profile scripts (`use-local.ps1`, `use-cloud.ps1`) merge `env/base.env` (shared config) with profile-specific overrides (`env/local.env` or `env/cloud.env`) to generate `.env`. This keeps one codebase with two deployment modes.

**Q4: Where does auth live?**
A: Authentication logic is in `apps/api/src/auth/`. JWT signing/verification in `JwtAuthGuard`. Refresh token rotation in `AuthService.refresh()`. Frontend stores tokens in HTTP-only cookies (set by API).

**Q5: How do we enforce family-scoped access?**
A: `FamilyAccessGuard` checks if user is a member of the family before allowing access. All care recipient queries join through `Family → FamilyMember → User` to ensure data isolation.

**Q6: Why RabbitMQ instead of Redis Pub/Sub?**
A: RabbitMQ guarantees message delivery (persistent queues), supports routing (topic exchanges), and has dead-letter queues. Redis Pub/Sub is fire-and-forget (messages lost if consumer down).

**Q7: How do workers scale?**
A: Workers are stateless consumers. Scale horizontally by running multiple instances (3 in production). RabbitMQ round-robins messages across consumers. Idempotency ensures duplicate processing is safe.

**Q8: How do secrets work in cloud?**
A: Secrets stored in `.env.prod` (never committed to Git). In Kubernetes, stored in `Secret` resources and mounted as env vars. In Docker Compose, passed via `--env-file .env.prod`.

**Q9: Can we switch from Cloudinary to S3?**
A: Yes, but requires re-uploading all documents. Create migration script to download from Cloudinary and upload to S3, update `cloudinaryUrl` → `s3Url` in database.

**Q10: Why TypeORM instead of Prisma?**
A: TypeORM was chosen for code-first entities (decorators), NestJS integration, and Active Record pattern. Prisma is schema-first (migrations less flexible). Both are viable; switching would require rewriting all entities and queries.

### Development & Operations

**Q11: How do I run just the API?**
A: `pnpm --filter @carecircle/api dev` (assumes Docker services already running).

**Q12: How do I run all apps at once?**
A: `pnpm dev` (runs API, Web, Workers concurrently via `turbo` or `concurrently`).

**Q13: How do I test auth quickly?**
A: See "How to Test Auth Quickly" section. TL;DR: `curl` register → login → access protected route → logout.

**Q14: Where are logs stored?**
A: Development: stdout (terminal). Production: Docker logs (`docker logs <container>`). Future: Centralized logging (Loki, CloudWatch, Datadog).

**Q15: How do I check if Docker services are healthy?**
A: `docker compose ps` shows health status. Individual checks: `docker exec carecircle-postgres-1 pg_isready`, `docker exec carecircle-redis-1 redis-cli ping`.

**Q16: How do I reset the database?**
A: **Local:** `docker compose down -v` (deletes volumes), then `docker compose up -d`, then `pnpm --filter @carecircle/api migration:run`. **Cloud:** Connect to Neon, drop tables, re-run migrations.

**Q17: How do I add a new API endpoint?**
A: 1. Create DTO in `apps/api/src/<module>/dto/`. 2. Add method to controller (`@Get()`, `@Post()`). 3. Add method to service (business logic). 4. Test with curl or Swagger.

**Q18: How do I add a new frontend page?**
A: 1. Create file in `apps/web/src/app/(app)/<page-name>/page.tsx`. 2. Use Next.js App Router conventions. 3. Fetch data with React Query. 4. Navigate to `http://localhost:3000/<page-name>`.

**Q19: How do I debug TypeScript errors?**
A: Run `pnpm build` to see all errors. Or use `tsc --noEmit` for type checking without emitting files. Check `tsconfig.json` for strict mode settings.

**Q20: How do I add a new environment variable?**
A: 1. Add to `env/base.env` (if shared) or `env/local.env`/`env/cloud.env` (if profile-specific). 2. Run `.\scripts\use-local.ps1` to regenerate `.env`. 3. Add to Zod schema in `apps/api/src/config/env.validation.ts`.

### Real-Time & Events

**Q21: How do real-time updates work?**
A: API publishes domain events to RabbitMQ. WebSocketConsumer listens, emits to Socket.io. Frontend listens on WebSocket, invalidates React Query cache, UI updates.

**Q22: How do I trigger an event manually?**
A: Call any API endpoint that publishes an event (e.g., `POST /medications/:id/log`). Check RabbitMQ management UI to see message in queue.

**Q23: What happens if RabbitMQ is down?**
A: Events are saved to `event_outbox` table (transactional outbox pattern). Outbox processor retries every 10 seconds until RabbitMQ is back up.

**Q24: How do I test push notifications locally?**
A: 1. Open Web app in Chrome. 2. Click "Enable Notifications". 3. Trigger an event (e.g., log medication). 4. Check browser notification. (Note: requires HTTPS in production, localhost works in dev.)

**Q25: Can I replay events?**
A: Yes, query `event_outbox` table, set `processed=false`, outbox processor will retry. Or: manually publish to RabbitMQ using management UI.

### Security & Compliance

**Q26: How do we prevent XSS?**
A: 1. HTTP-only cookies (tokens not accessible via JavaScript). 2. React auto-escapes JSX (prevents script injection). 3. CSP headers (future enhancement).

**Q27: How do we prevent CSRF?**
A: `SameSite=Strict` cookies (browser blocks cross-site requests). Alternative: CSRF tokens (not needed with SameSite).

**Q28: Are passwords hashed?**
A: Yes, bcrypt with cost factor 10 (~100ms per hash). Never store plaintext passwords.

**Q29: How do we audit who did what?**
A: Every event is consumed by `AuditConsumer`, logged to `audit_log` table with user ID, timestamp, and payload. Query for compliance reports.

**Q30: How do we handle GDPR data deletion?**
A: (Future) Implement "Delete User" endpoint that anonymizes/deletes all related data (care recipients, medications, documents). Requires cascading deletes or soft deletes.

---

## Conclusion

**You now have a complete operational handbook for CareCircle.**

**What you can do with this:**
- Onboard new engineers (point them to this doc)
- Debug production issues (use Troubleshooting Playbook)
- Make architectural decisions (refer to Decision Log)
- Deploy to cloud (follow Profile B Runbook)
- Answer stakeholder questions (use FAQ Pack)

**Next Steps:**
1. **Run Profile A Runbook** to verify local dev setup
2. **Run smoke tests** to confirm all features work
3. **Deploy to staging** using Profile B Runbook
4. **Iterate:** Update this handbook as the system evolves

**Handbook Maintenance:**
- Update Decision Log when you change tech stack
- Update FAQ when you answer the same question 3+ times
- Update Runbooks when you add new deployment steps
- Keep Environment Contract in sync with actual env vars

**Where to Find Help:**
- This handbook (you're reading it)
- `docs/guides/PROJECT_OVERVIEW.md` (high-level overview)
- `docs/QA_TEST_REPORT.md` (QA smoke test results)
- `DOCKER_DEPLOYMENT.md` (Docker deployment details)
- Swagger docs: `http://localhost:3001/api`

**Final Reminder:** This is a living document. Treat it as code—update it alongside feature changes. A stale handbook is worse than no handbook.

---

**End of Handbook**

**Version:** 1.0
**Last Updated:** 2026-01-14
**Next Review:** 2026-04-14 (quarterly)
