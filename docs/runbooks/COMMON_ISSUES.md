# Common Issues & Troubleshooting

Quick solutions for frequently encountered issues in CareCircle development and production.

## Table of Contents

- [Local Development](#local-development)
  - [Docker Issues](#docker-issues)
  - [Database Issues](#database-issues)
  - [Node/pnpm Issues](#nodepnpm-issues)
  - [Environment Issues](#environment-issues)
- [Production/Staging](#productionstaging)
  - [Deployment Failures](#deployment-failures)
  - [Worker Issues](#worker-issues)
  - [Performance Issues](#performance-issues)
  - [Authentication Issues](#authentication-issues)

---

## Local Development

### Docker Issues

#### Services won't start

**Symptoms:** `docker-compose up` fails or services crash immediately

**Solutions:**

1. **Check if ports are in use:**
   ```bash
   # Windows
   netstat -ano | findstr :5432
   netstat -ano | findstr :6379
   
   # macOS/Linux
   lsof -i :5432
   lsof -i :6379
   ```
   
   Stop conflicting services or change ports in `docker-compose.override.yml`.

2. **Docker Desktop not running:**
   Start Docker Desktop and wait for it to fully initialize.

3. **Out of disk space:**
   ```bash
   docker system prune -a --volumes
   ```

4. **Corrupted volumes:**
   ```bash
   docker-compose down -v
   docker-compose up -d
   ```

#### PostgreSQL won't start

**Symptoms:** `postgres exited with code 1`

**Solutions:**

1. **Check logs:**
   ```bash
   docker-compose logs postgres
   ```

2. **Data directory corruption:**
   ```bash
   docker-compose down
   docker volume rm carecircle_postgres_data
   docker-compose up -d
   ```
   ⚠️ This deletes all local data.

3. **Insufficient shared memory (Linux):**
   Add to `docker-compose.override.yml`:
   ```yaml
   postgres:
     shm_size: 256mb
   ```

#### Redis connection refused

**Symptoms:** `ECONNREFUSED 127.0.0.1:6379`

**Solutions:**

1. **Check if Redis is running:**
   ```bash
   docker-compose ps redis
   docker-compose logs redis
   ```

2. **Restart Redis:**
   ```bash
   docker-compose restart redis
   ```

3. **Check environment:**
   Ensure `REDIS_HOST=localhost` (not `redis`) for local development.

---

### Database Issues

#### "Connection refused" or "ECONNREFUSED"

**Solutions:**

1. **Wait for PostgreSQL to be ready:**
   ```bash
   docker-compose ps postgres  # Should show "healthy"
   ```

2. **Check DATABASE_URL:**
   ```bash
   # Should be for local:
   postgresql://postgres:1234@localhost:5432/carecircle
   
   # NOT:
   postgresql://postgres:1234@postgres:5432/carecircle  # Docker network name
   ```

3. **Restart services:**
   ```bash
   docker-compose restart postgres
   ```

#### "Relation does not exist"

**Symptoms:** `relation "User" does not exist`

**Solutions:**

1. **Run migrations/push:**
   ```bash
   pnpm --filter @carecircle/database db:push
   ```

2. **Generate Prisma client:**
   ```bash
   pnpm --filter @carecircle/database generate
   ```

#### "Prisma Client not generated"

**Symptoms:** `PrismaClient is not a constructor` or import errors

**Solutions:**

```bash
pnpm --filter @carecircle/database generate
```

If that fails:
```bash
rm -rf packages/database/node_modules/.prisma
pnpm install
pnpm --filter @carecircle/database generate
```

#### Migration conflicts

**Symptoms:** Migration failed due to schema changes

**Solutions:**

1. **Development (reset is OK):**
   ```bash
   pnpm --filter @carecircle/database db:push --force-reset
   ```

2. **If you have custom migrations:**
   ```bash
   # Create a migration for the diff
   pnpm --filter @carecircle/database prisma migrate dev --name fix_schema
   ```

---

### Node/pnpm Issues

#### "Module not found"

**Solutions:**

1. **Reinstall dependencies:**
   ```bash
   pnpm install
   ```

2. **Clean and reinstall:**
   ```bash
   make clean-all
   pnpm install
   ```

3. **Check workspace dependencies:**
   ```bash
   pnpm why <package-name>
   ```

#### "Cannot find module @carecircle/xxx"

**Symptoms:** Internal package imports failing

**Solutions:**

1. **Build packages first:**
   ```bash
   pnpm build
   ```

2. **Check tsconfig paths:**
   Verify `tsconfig.json` has correct path mappings.

#### Turbo cache issues

**Symptoms:** Changes not reflected, stale builds

**Solutions:**

```bash
rm -rf .turbo
pnpm build
```

#### pnpm store corruption

**Symptoms:** Random install failures

**Solutions:**

```bash
pnpm store prune
rm -rf node_modules
pnpm install
```

---

### Environment Issues

#### "JWT_SECRET must be at least 32 characters"

**Solutions:**

1. Generate a proper secret:
   ```bash
   node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
   ```

2. Add to `.env`:
   ```env
   JWT_SECRET=your-generated-64-character-hex-string-here
   JWT_REFRESH_SECRET=another-generated-64-character-hex-string
   ```

#### Environment variables not loading

**Solutions:**

1. **Check file exists:**
   ```bash
   ls -la .env*
   ```

2. **Run env setup:**
   ```bash
   pnpm env:local  # or env:cloud
   ```

3. **Verify dotenv is loading:**
   Add at top of main.ts:
   ```typescript
   console.log('DATABASE_URL:', process.env.DATABASE_URL ? 'set' : 'NOT SET');
   ```

#### Wrong environment profile

**Symptoms:** Connecting to cloud DB when expecting local (or vice versa)

**Solutions:**

```bash
# Check current profile
cat .env | head -5

# Switch to local
pnpm env:local

# Switch to cloud
pnpm env:cloud

# Auto-detect
pnpm env:auto
```

---

## Production/Staging

### Deployment Failures

#### Docker build fails

**Solutions:**

1. **Check Dockerfile:**
   ```bash
   docker build -f apps/api/Dockerfile -t test .
   ```

2. **Clear Docker cache:**
   ```bash
   docker builder prune -f
   ```

3. **Check .dockerignore:**
   Ensure required files aren't being ignored.

#### Kubernetes deployment stuck

**Symptoms:** Pods in `Pending` or `CrashLoopBackOff`

**Solutions:**

1. **Check pod status:**
   ```bash
   kubectl describe pod -n carecircle <pod-name>
   ```

2. **Check events:**
   ```bash
   kubectl get events -n carecircle --sort-by='.lastTimestamp'
   ```

3. **Common causes:**
   - **ImagePullBackOff:** Check registry credentials
   - **CrashLoopBackOff:** Check container logs
   - **Pending:** Check resource quotas

#### Migration fails in deployment

**Solutions:**

1. **Check migration status:**
   ```bash
   kubectl exec -n carecircle <api-pod> -- npx prisma migrate status
   ```

2. **Run manually:**
   ```bash
   kubectl exec -n carecircle <api-pod> -- npx prisma migrate deploy
   ```

3. **If stuck on lock:**
   Check Prisma `_prisma_migrations` table for failed migrations.

---

### Worker Issues

#### Jobs not processing

**Symptoms:** Queue growing, jobs stuck

**Solutions:**

1. **Check worker health:**
   ```bash
   curl http://workers-host:3002/health
   ```

2. **Check Redis connection:**
   ```bash
   kubectl exec -n carecircle <worker-pod> -- redis-cli -h $REDIS_HOST ping
   ```

3. **Check for stalled jobs:**
   ```bash
   # In Redis
   KEYS bull:*:stalled
   ```

4. **Restart workers:**
   ```bash
   kubectl rollout restart deployment/carecircle-workers -n carecircle
   ```

#### Dead Letter Queue filling up

**Symptoms:** Jobs in DLQ, repeated failures

**Solutions:**

1. **Check DLQ contents:**
   ```bash
   # Via API or direct Redis
   LRANGE bull:dlq:failed 0 10
   ```

2. **Review error patterns:**
   Check worker logs for common error messages.

3. **Retry jobs:**
   Use the dead-letter worker or manual retry via BullMQ admin.

#### Worker memory issues

**Symptoms:** OOM kills, memory leaks

**Solutions:**

1. **Check memory usage:**
   ```bash
   kubectl top pod -n carecircle -l app=carecircle-workers
   ```

2. **Increase limits temporarily:**
   ```yaml
   resources:
     limits:
       memory: "1Gi"
   ```

3. **Check for leaks:**
   Review recent code changes, especially in job handlers.

---

### Performance Issues

#### Slow API responses

**Solutions:**

1. **Check database queries:**
   Enable Prisma query logging:
   ```env
   PRISMA_LOG_LEVEL=query,warn,error
   ```

2. **Check Redis connection:**
   Cache might not be hitting.

3. **Check resource utilization:**
   ```bash
   kubectl top pods -n carecircle
   ```

4. **Scale if needed:**
   ```bash
   kubectl scale deployment/carecircle-api --replicas=4 -n carecircle
   ```

#### High database load

**Solutions:**

1. **Check active connections:**
   ```sql
   SELECT count(*) FROM pg_stat_activity WHERE datname = 'carecircle';
   ```

2. **Check slow queries:**
   ```sql
   SELECT * FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;
   ```

3. **Add indexes for slow queries.**

4. **Review N+1 queries in code.**

#### Redis memory full

**Solutions:**

1. **Check memory:**
   ```bash
   redis-cli INFO memory
   ```

2. **Clear old cache:**
   ```bash
   redis-cli SCAN 0 MATCH "cache:*" COUNT 1000 | xargs redis-cli DEL
   ```

3. **Review eviction policy in Upstash console.**

---

### Authentication Issues

#### "Invalid token" or "Token expired"

**Solutions:**

1. **Check token in request:**
   Verify Authorization header format: `Bearer <token>`

2. **Check JWT_SECRET matches:**
   Same secret must be used for signing and verification.

3. **Check token expiration:**
   Default is 15 minutes for access token.

4. **Try refresh token flow:**
   Use refresh token to get new access token.

#### Sessions not persisting

**Solutions:**

1. **Check Redis connection:**
   Sessions are stored in Redis.

2. **Check cookie settings:**
   - Domain must match
   - Secure flag for HTTPS
   - SameSite settings

3. **Check CORS configuration:**
   Credentials must be allowed.

#### OAuth/SSO failures

**Solutions:**

1. **Check callback URLs:**
   Must exactly match configured in provider.

2. **Check client ID/secret:**
   Verify in environment variables.

3. **Check provider status:**
   Some providers have outages.

---

## Quick Reference

### Restart All Services (Local)

```bash
docker-compose down
docker-compose up -d
pnpm dev:all
```

### Full Reset (Local)

```bash
docker-compose down -v
make clean-all
pnpm install
docker-compose up -d
pnpm --filter @carecircle/database db:push
pnpm dev:all
```

### Check Everything (Local)

```bash
# Docker services
docker-compose ps

# Environment
cat .env | grep -E "DATABASE_URL|REDIS"

# Database connection
pnpm --filter @carecircle/database prisma db pull --print

# Dependencies
pnpm install --frozen-lockfile
```

---

**Still stuck?** Check the [Incident Response Runbook](./INCIDENT_RESPONSE.md) or ask in the team channel.

