# CareCircle Docker Deployment Guide

This guide covers deploying CareCircle using Docker for production environments supporting 1M+ concurrent users.

## Architecture Overview

### Services

- **API (4 replicas)**: NestJS backend with Socket.io WebSocket support
- **Web (3 replicas)**: Next.js frontend with PWA capabilities
- **Workers (3 replicas)**: Background job processors
- **PostgreSQL**: Primary database with optimized settings for high load
- **Redis**: Caching layer with LRU eviction policy
- **RabbitMQ**: Message broker for event-driven architecture
- **MinIO**: S3-compatible object storage
- **Nginx**: Reverse proxy with load balancing and rate limiting

### Resource Allocation (for 1M+ users)

```
Total CPU: ~24 cores
Total Memory: ~16GB

PostgreSQL: 4GB RAM (200 max connections)
Redis: 2GB RAM (10K max clients)
RabbitMQ: 2GB RAM
API: 4x 2GB = 8GB RAM
Web: 3x 1GB = 3GB RAM
Workers: 3x 512MB = 1.5GB RAM
Nginx: 256MB RAM
MinIO: 1GB RAM
```

## Prerequisites

1. **Docker** (20.10+)
2. **Docker Compose** (2.0+)
3. **Minimum System Requirements**:
   - CPU: 24 cores
   - RAM: 32GB
   - Disk: 500GB SSD
   - Network: 1Gbps

## Quick Start

### 1. Environment Setup

Copy the example environment file:

```bash
cp .env.prod.example .env.prod
```

Edit `.env.prod` and fill in all required values:

```bash
# Generate secrets
openssl rand -base64 64  # For JWT_SECRET and JWT_REFRESH_SECRET
openssl rand -hex 32     # For ENCRYPTION_KEY

# Generate VAPID keys for push notifications
npx web-push generate-vapid-keys
```

### 2. Build Images

**Windows (PowerShell):**
```powershell
.\scripts\deploy-prod.ps1 -Build
```

**Linux/Mac:**
```bash
chmod +x scripts/deploy-prod.sh
./scripts/deploy-prod.sh build
```

Build time: ~15-20 minutes for all services

### 3. Start Services

**Windows:**
```powershell
.\scripts\deploy-prod.ps1 -Up
```

**Linux/Mac:**
```bash
./scripts/deploy-prod.sh up
```

### 4. Verify Deployment

Check all services are healthy:

```bash
docker-compose -f docker-compose.prod.yml ps
```

All services should show "healthy" status.

### 5. Access the Application

- **Web App**: http://localhost (or your domain)
- **API**: http://localhost/api/v1
- **API Documentation**: http://localhost/api-docs
- **MinIO Console**: http://localhost:9001

## Manual Docker Commands

### Build specific service

```bash
docker-compose -f docker-compose.prod.yml --env-file .env.prod build api
docker-compose -f docker-compose.prod.yml --env-file .env.prod build web
docker-compose -f docker-compose.prod.yml --env-file .env.prod build workers
```

### Start services

```bash
# All services
docker-compose -f docker-compose.prod.yml --env-file .env.prod up -d

# Specific service
docker-compose -f docker-compose.prod.yml --env-file .env.prod up -d api
```

### View logs

```bash
# All services
docker-compose -f docker-compose.prod.yml logs -f

# Specific service
docker-compose -f docker-compose.prod.yml logs -f api
docker-compose -f docker-compose.prod.yml logs -f web
```

### Stop services

```bash
docker-compose -f docker-compose.prod.yml down
```

### Scale services manually

```bash
docker-compose -f docker-compose.prod.yml up -d --scale api=8 --scale web=5 --scale workers=5
```

## Performance Tuning

### PostgreSQL

The database is configured for high performance with:

- **200 max connections**: Supports high concurrent queries
- **2GB shared buffers**: Optimal for 4GB RAM allocation
- **6GB effective cache**: Leverages system memory for queries
- **WAL optimization**: Improved write performance

### Redis

Configured for caching with:

- **2GB max memory**: LRU eviction when full
- **10,000 max clients**: Supports high concurrent connections
- **Append-only file**: Data persistence with fsync every second

### Nginx

Load balancing with:

- **Rate limiting**: 100 req/s for API, 200 req/s for web
- **Connection limiting**: 10 concurrent for API, 20 for web
- **Static caching**: 1 year for immutable assets, 7 days for media
- **Gzip compression**: Reduces bandwidth by ~70%

## Monitoring

### Health Checks

All services include health checks:

```bash
# Check API health
curl http://localhost/health

# Check individual container health
docker inspect --format='{{.State.Health.Status}}' carecircle-prod-api-1
```

### Resource Usage

Monitor resource consumption:

```bash
# All containers
docker stats

# Specific service
docker stats carecircle-prod-api-1
```

### Logs

Centralized logging:

```bash
# Last 100 lines
docker-compose -f docker-compose.prod.yml logs --tail=100

# Follow logs in real-time
docker-compose -f docker-compose.prod.yml logs -f

# Filter by service
docker-compose -f docker-compose.prod.yml logs -f api web
```

## Database Management

### Run Migrations

```bash
docker-compose -f docker-compose.prod.yml exec api npm run migration:run
```

### Create Backup

```bash
docker-compose -f docker-compose.prod.yml exec postgres pg_dump -U carecircle carecircle > backup.sql
```

### Restore Backup

```bash
docker-compose -f docker-compose.prod.yml exec -T postgres psql -U carecircle carecircle < backup.sql
```

## SSL/TLS Configuration

For HTTPS in production:

1. Obtain SSL certificates (Let's Encrypt recommended):

```bash
certbot certonly --standalone -d your-domain.com
```

2. Copy certificates:

```bash
mkdir -p certs
cp /etc/letsencrypt/live/your-domain.com/fullchain.pem certs/
cp /etc/letsencrypt/live/your-domain.com/privkey.pem certs/
```

3. Uncomment SSL configuration in `nginx.conf`

4. Update `.env.prod`:

```bash
NEXT_PUBLIC_API_URL=https://your-domain.com/api/v1
NEXT_PUBLIC_WS_URL=wss://your-domain.com
```

5. Restart nginx:

```bash
docker-compose -f docker-compose.prod.yml restart nginx
```

## Scaling for 1M+ Users

### Horizontal Scaling

The architecture supports horizontal scaling:

```bash
# Scale API to 8 instances
docker-compose -f docker-compose.prod.yml up -d --scale api=8

# Scale Web to 6 instances
docker-compose -f docker-compose.prod.yml up -d --scale web=6

# Scale Workers to 6 instances
docker-compose -f docker-compose.prod.yml up -d --scale workers=6
```

### Database Scaling

For very high loads, consider:

1. **Read Replicas**: PostgreSQL streaming replication
2. **Connection Pooling**: PgBouncer for connection management
3. **Partitioning**: Table partitioning for large datasets

### Redis Clustering

For distributed caching:

1. Use Redis Cluster mode
2. Or implement consistent hashing
3. Or use Redis Sentinel for HA

### Load Balancer

For multi-server deployment:

1. Use external load balancer (AWS ELB, GCP Load Balancer)
2. Or HAProxy/Traefik for advanced routing
3. Configure health checks and session affinity

## Troubleshooting

### Service won't start

```bash
# Check logs
docker-compose -f docker-compose.prod.yml logs service-name

# Check configuration
docker-compose -f docker-compose.prod.yml config

# Restart service
docker-compose -f docker-compose.prod.yml restart service-name
```

### Out of memory

```bash
# Check memory usage
docker stats

# Increase memory limits in docker-compose.prod.yml
# Or scale down replicas
```

### Database connection errors

```bash
# Check PostgreSQL logs
docker-compose -f docker-compose.prod.yml logs postgres

# Verify connection string
docker-compose -f docker-compose.prod.yml exec api env | grep DATABASE_URL

# Test connection
docker-compose -f docker-compose.prod.yml exec postgres psql -U carecircle -c "SELECT 1"
```

### High CPU usage

```bash
# Identify problematic container
docker stats

# Check application logs
docker-compose -f docker-compose.prod.yml logs -f service-name

# Consider scaling horizontally
```

## Maintenance

### Update Application

```bash
# Pull latest code
git pull

# Rebuild images
docker-compose -f docker-compose.prod.yml build

# Rolling update (zero downtime)
docker-compose -f docker-compose.prod.yml up -d --no-deps --build api
docker-compose -f docker-compose.prod.yml up -d --no-deps --build web
docker-compose -f docker-compose.prod.yml up -d --no-deps --build workers
```

### Clean Up

```bash
# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Complete cleanup (WARNING: removes all data)
docker-compose -f docker-compose.prod.yml down -v --rmi all
```

## Security Best Practices

1. **Never commit** `.env.prod` to version control
2. **Use strong passwords** for all services (min 32 characters)
3. **Rotate secrets** regularly (quarterly recommended)
4. **Enable SSL/TLS** for production
5. **Use private networks** for service communication
6. **Regular updates** for base images and dependencies
7. **Implement monitoring** and alerting
8. **Regular backups** of database and volumes

## Support

For issues or questions:

- Check logs: `docker-compose -f docker-compose.prod.yml logs`
- Review configuration: `docker-compose -f docker-compose.prod.yml config`
- Consult [docs/guides/PROJECT_OVERVIEW.md](docs/guides/PROJECT_OVERVIEW.md)
