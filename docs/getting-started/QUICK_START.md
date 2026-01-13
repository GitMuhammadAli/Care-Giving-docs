# ⚡ Quick Start Guide

> Get CareCircle running in 5 minutes.

---

## Prerequisites

```bash
# Check you have these installed:
node --version    # Should be 18+
pnpm --version    # Should be 8+
docker --version  # Should be 20+
```

---

## Step 1: Clone & Install

```bash
git clone https://github.com/yourorg/carecircle.git
cd carecircle
pnpm install
```

---

## Step 2: Environment Setup

```bash
cp env.example .env
```

The default `.env` works for local development. No changes needed!

---

## Step 3: Start Infrastructure

```bash
docker-compose up -d
```

This starts:
- PostgreSQL (database)
- Redis (cache)
- RabbitMQ (events)
- Mailpit (email testing)

Wait ~10 seconds for services to initialize.

---

## Step 4: Database Setup

```bash
pnpm db:migrate
```

---

## Step 5: Start Development

```bash
pnpm dev
```

---

## 🎉 You're Running!

| What | Where |
|------|-------|
| **Web App** | http://localhost:3000 |
| **API** | http://localhost:3001 |
| **API Docs** | http://localhost:3001/api/docs |
| **Emails** | http://localhost:8025 |

---

## First Steps

1. Open http://localhost:3000
2. Click "Get Started" or "Sign Up"
3. Register a new account
4. Check Mailpit for verification email
5. Explore the dashboard!

---

## Common Commands

```bash
# Start everything
pnpm dev

# Start only API
pnpm dev:api

# Start only frontend
pnpm dev:web

# Stop Docker services
docker-compose down

# View logs
docker-compose logs -f
```

---

## Troubleshooting

### Port already in use?

```bash
# Find what's using port 3000
netstat -ano | findstr :3000

# Or change the port in .env
PORT=3002
```

### Database connection failed?

```bash
# Restart Docker services
docker-compose down
docker-compose up -d

# Wait 10 seconds, then try again
```

### Need to reset everything?

```bash
# Stop services
docker-compose down -v  # -v removes volumes (data)

# Start fresh
docker-compose up -d
pnpm db:migrate
pnpm db:seed  # Optional: add sample data
```

---

## Next Steps

- [Project Overview](../guides/PROJECT_OVERVIEW.md) - Understand the full system
- [Authentication Guide](../guides/AUTHENTICATION.md) - How auth works
- [Architecture](../architecture/) - Deep dive into code

---

_Back to [Getting Started](./README.md) | [Documentation Index](../README.md)_

