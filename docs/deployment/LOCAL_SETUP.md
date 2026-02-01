# Local Development Setup

Quick-start guide for setting up the CareCircle development environment.

## Prerequisites

Before you begin, ensure you have the following installed:

| Tool | Version | Installation |
|------|---------|--------------|
| Node.js | 20.x LTS | [nodejs.org](https://nodejs.org/) |
| pnpm | 9.x | `npm install -g pnpm` |
| Docker | 24.x+ | [docker.com](https://www.docker.com/products/docker-desktop) |
| Docker Compose | 2.x+ | Included with Docker Desktop |
| Git | 2.x+ | [git-scm.com](https://git-scm.com/) |

### Verify Installation

```bash
node --version    # Should show v20.x.x
pnpm --version    # Should show 9.x.x
docker --version  # Should show Docker version 24.x.x
```

## Quick Setup

### Using Make (Recommended)

```bash
# Clone the repository
git clone https://github.com/your-org/carecircle.git
cd carecircle

# Run setup (installs deps, starts Docker, sets up DB)
make setup

# Start development
make dev
```

### Manual Setup

```bash
# 1. Clone the repository
git clone https://github.com/your-org/carecircle.git
cd carecircle

# 2. Install dependencies
pnpm install

# 3. Start Docker services (PostgreSQL, Redis, RabbitMQ)
docker-compose up -d

# 4. Wait for services to be ready
docker-compose ps  # All should be "healthy"

# 5. Setup environment
cp .env.local .env  # Or use: pnpm env:local

# 6. Generate Prisma client
pnpm --filter @carecircle/database generate

# 7. Push schema to database
pnpm --filter @carecircle/database db:push

# 8. Start development servers
pnpm dev:all
```

## Services & Ports

After setup, the following services will be available:

| Service | URL | Description |
|---------|-----|-------------|
| Web App | http://localhost:3000 | Next.js frontend |
| API | http://localhost:4000 | NestJS backend |
| API Docs | http://localhost:4000/api | Swagger documentation |
| Workers Health | http://localhost:4001/health | Worker health check |
| PostgreSQL | localhost:5432 | Database |
| Redis | localhost:6379 | Cache & queues |
| RabbitMQ | localhost:5672 | Message broker |
| RabbitMQ UI | http://localhost:15672 | Management (guest/guest) |

### Development Tools (via docker-compose.override.yml)

| Tool | URL | Description |
|------|-----|-------------|
| Adminer | http://localhost:8080 | Database GUI |
| Redis Commander | http://localhost:8081 | Redis GUI |
| Mailhog | http://localhost:8025 | Email testing |

## Environment Configuration

### Available Profiles

| File | Purpose |
|------|---------|
| `.env.local` | Local Docker services (default) |
| `.env.cloud` | Cloud services (Neon, Upstash) |
| `.env.example` | Template with all variables |

### Switching Profiles

```bash
# Auto-detect environment (checks if Docker services are running)
pnpm env:auto

# Force local profile
pnpm env:local

# Force cloud profile
pnpm env:cloud
```

### Key Environment Variables

```env
# Database
DATABASE_URL=postgresql://postgres:1234@localhost:5432/carecircle

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# API
JWT_SECRET=your-secret-min-32-chars
JWT_REFRESH_SECRET=your-refresh-secret-min-32-chars

# Workers
HEALTH_PORT=4001
```

## Development Workflow

### Starting Services

```bash
# Start everything
make dev

# Or start individually:
make dev-api      # API only
make dev-web      # Web only
make dev-workers  # Workers only
```

### Common Commands

```bash
# Run tests
make test

# Run linting
make lint

# Build all packages
make build

# View logs
make logs

# Stop services
make stop
```

### Database Commands

```bash
# Push schema changes (development)
make db-push

# Run migrations (production-like)
make db-migrate

# Open Prisma Studio (GUI)
make db-studio

# Seed database
make db-seed
```

## Project Structure

```
carecircle/
├── apps/
│   ├── api/          # NestJS backend
│   ├── web/          # Next.js frontend
│   └── workers/      # Background job processors
├── packages/
│   ├── config/       # Shared configuration
│   ├── database/     # Prisma schema & client
│   └── logger/       # Shared logging
├── docker-compose.yml          # Core services
├── docker-compose.override.yml # Dev tools
├── Makefile                    # Development commands
└── turbo.json                  # Turborepo config
```

## Troubleshooting

### Docker Issues

**Services not starting:**
```bash
# Check service status
docker-compose ps

# View logs for specific service
docker-compose logs postgres

# Restart services
docker-compose down && docker-compose up -d
```

**Port already in use:**
```bash
# Find process using port
lsof -i :5432  # macOS/Linux
netstat -ano | findstr :5432  # Windows

# Kill process or change port in docker-compose.override.yml
```

### Database Issues

**Connection refused:**
1. Check if PostgreSQL is running: `docker-compose ps postgres`
2. Verify DATABASE_URL in your .env file
3. Wait for healthcheck: services may need 10-15 seconds

**Schema out of sync:**
```bash
# Reset and re-push schema
make db-push
```

### Node/pnpm Issues

**Module not found:**
```bash
# Clean and reinstall
make clean-all
pnpm install
```

**Prisma client errors:**
```bash
# Regenerate Prisma client
pnpm --filter @carecircle/database generate
```

### Windows-Specific

**Line ending issues:**
```bash
git config core.autocrlf false
```

**Docker memory:**
- Allocate at least 4GB RAM to Docker Desktop
- Settings → Resources → Memory

## Next Steps

1. **Explore the API**: Visit http://localhost:4000/api for Swagger docs
2. **Create test user**: Use the API or seed script
3. **Read the architecture**: See [docs/tech-stack/architecture/](../tech-stack/architecture/)
4. **Check feature docs**: See [docs/features/](../features/)

## Getting Help

- **Common Issues**: See [COMMON_ISSUES.md](../runbooks/COMMON_ISSUES.md)
- **CI/CD Guide**: See [CI_CD_GUIDE.md](./CI_CD_GUIDE.md)
- **Full Documentation**: See [DOCUMENTATION_INDEX.md](../DOCUMENTATION_INDEX.md)

