# 🚀 Getting Started

> Setup guides to get CareCircle running on your machine.

---

## Guides in This Section

| Guide | Description | Time |
|-------|-------------|------|
| [Quick Start](./QUICK_START.md) | Get running with minimal setup | 5 min |
| [Free Services Setup](./FREE_SERVICES_SETUP.md) | Configure free tier cloud services | 15 min |

---

## Prerequisites

Before you begin, ensure you have:

- **Node.js 18+** - [Download](https://nodejs.org/)
- **pnpm 8+** - `npm install -g pnpm`
- **Docker Desktop** - [Download](https://www.docker.com/products/docker-desktop/)
- **Git** - [Download](https://git-scm.com/)

---

## Quick Start (5 Minutes)

```bash
# 1. Clone the repository
git clone https://github.com/yourorg/carecircle.git
cd carecircle

# 2. Install dependencies
pnpm install

# 3. Copy environment file
cp env.example .env

# 4. Start Docker services
docker-compose up -d

# 5. Run database migrations
pnpm db:migrate

# 6. Start development servers
pnpm dev
```

---

## What Gets Started

| Service | URL | Description |
|---------|-----|-------------|
| 🌐 **Web App** | http://localhost:3000 | Next.js frontend |
| 🔌 **API Server** | http://localhost:3001 | NestJS backend |
| 📚 **Swagger** | http://localhost:3001/api/docs | API documentation |
| 🐘 **PostgreSQL** | localhost:5432 | Database |
| 🔴 **Redis** | localhost:6379 | Cache & sessions |
| 🐰 **RabbitMQ** | http://localhost:15672 | Message queue |
| 📧 **Mailpit** | http://localhost:8025 | Email testing |

---

## Environment Variables

Key variables in `.env`:

```env
# Database
DATABASE_URL=postgresql://carecircle:carecircle_dev@localhost:5432/carecircle

# Redis
REDIS_URL=redis://localhost:6379

# RabbitMQ
RABBITMQ_URL=amqp://carecircle:carecircle_dev@localhost:5672

# JWT Secrets (change in production!)
JWT_SECRET=your-super-secret-key-change-me
JWT_REFRESH_SECRET=another-super-secret-key-change-me

# Frontend URL
FRONTEND_URL=http://localhost:3000
```

See [env.example](../../env.example) for all available options.

---

## Next Steps

1. **Understand the project** → [Project Overview](../guides/PROJECT_OVERVIEW.md)
2. **Setup free cloud services** → [Free Services Setup](./FREE_SERVICES_SETUP.md)
3. **Learn the architecture** → [Architecture](../architecture/)

---

_Back to [Documentation Index](../README.md)_

