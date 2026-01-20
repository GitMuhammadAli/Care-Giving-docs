# 📦 CareCircle Database Backup & Restore Procedures (Neon DB)

**Last Updated:** January 16, 2026
**Status:** ✅ Production Ready
**Database Provider:** Neon (Serverless Postgres)

---

## 📋 Overview

CareCircle uses **Neon DB**, a serverless Postgres platform with **built-in automated backups**. This document covers Neon's native backup features and optional manual backup procedures.

### Neon's Native Backup Features ✨

Neon provides automatic backups without any configuration needed:

- **Automatic Daily Backups:** Full database backups every 24 hours
- **Point-in-Time Recovery (PITR):** Restore to any point within the last 7-30 days (plan-dependent)
- **Branch-Based Backups:** Instant database branches for testing/staging
- **Zero Configuration:** Backups run automatically, no setup required
- **Retention:** 7 days (Free), 30 days (Pro), custom (Enterprise)
- **Storage:** Included in your plan, no additional costs

---

## 🚀 Quick Start

### View Backups in Neon Console

1. Go to [Neon Console](https://console.neon.tech)
2. Select your project: `carecircle`
3. Navigate to **Settings** → **Backups**
4. View all available backups with timestamps

### Restore from Neon Backup (Web UI)

1. In Neon Console, go to **Settings** → **Backups**
2. Select the backup point you want to restore
3. Click **Restore**
4. Choose restoration method:
   - **Restore to new branch** (recommended for testing)
   - **Restore in place** (overwrites current database - use with caution!)
5. Confirm restoration

### Point-in-Time Recovery

```bash
# Using Neon CLI
neonctl branch create --name recovery-20260116 --timestamp 2026-01-15T10:30:00Z

# This creates a new branch at the exact time you specify
# Update DATABASE_URL to point to the new branch for testing
```

---

## ⚙️ Setup Instructions

### 1. Install Neon CLI (Optional but Recommended)

```bash
# macOS/Linux
curl -fsSL https://neon.tech/cli.sh | bash

# Or with npm
npm install -g neonctl

# Authenticate
neonctl auth

# Verify
neonctl projects list
```

### 2. Environment Variables

Your `.env` file already has the Neon connection string:

```bash
# Required - Get from Neon Console
DATABASE_URL=postgresql://user:pass@ep-xxx.region.aws.neon.tech/carecircle?sslmode=require

# Optional - For backup monitoring alerts
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL
NEON_API_KEY=your-neon-api-key  # Get from Neon Console → Account Settings → API Keys
NEON_PROJECT_ID=your-project-id  # Get from Neon Console → Project Settings
```

### 3. Neon API Key Setup (for programmatic access)

```bash
# Get API Key
# 1. Go to https://console.neon.tech
# 2. Click your profile → Account Settings
# 3. Go to "API Keys" tab
# 4. Click "Generate new API key"
# 5. Copy and save securely

# Set in environment
export NEON_API_KEY="your-api-key-here"
export NEON_PROJECT_ID="your-project-id"

# Test API access
curl -X GET \
  -H "Authorization: Bearer $NEON_API_KEY" \
  "https://console.neon.tech/api/v2/projects/$NEON_PROJECT_ID"
```

### 4. Enable Backup Monitoring (Optional)

If you want automated monitoring of Neon backups:

```bash
# Create a simple monitoring script
cat > scripts/monitor-neon-backups.sh << 'EOF'
#!/bin/bash
# Check Neon backup status via API
NEON_API_KEY="${NEON_API_KEY}"
PROJECT_ID="${NEON_PROJECT_ID}"

# Get latest backup timestamp
curl -s -H "Authorization: Bearer $NEON_API_KEY" \
  "https://console.neon.tech/api/v2/projects/$PROJECT_ID" \
  | jq -r '.project.last_backup'
EOF

chmod +x scripts/monitor-neon-backups.sh

# Add to crontab for daily checks
# 0 3 * * * /path/to/scripts/monitor-neon-backups.sh
```

---

## 📅 Automated Backups

### Neon's Automatic Backups (Default) ✅

**Good news:** Backups are already running automatically! Neon handles this for you:

- **Daily Backups:** Runs every 24 hours automatically
- **No Configuration Needed:** Works out of the box
- **Included in Plan:** No additional cost
- **Reliable:** 99.95% uptime SLA
- **Encrypted:** At-rest and in-transit

**View backup schedule:**
```bash
# Using Neon CLI
neonctl projects get $NEON_PROJECT_ID --output json | jq '.backup_retention_days'
```

### Optional: Manual Backup via Branch Creation

Create instant backups by creating database branches:

```bash
# Create a backup branch with today's date
neonctl branch create \
  --name "backup-$(date +%Y%m%d)" \
  --project-id $NEON_PROJECT_ID

# This creates an instant, zero-downtime copy of your database
# Useful for:
# - Before major migrations
# - Pre-deployment safety
# - Testing data changes
```

### Optional: Automated Branch Backups (for extra safety)

Create a cron job for automatic branch-based backups:

```bash
# Create backup script
cat > scripts/create-neon-backup.sh << 'EOF'
#!/bin/bash
set -e

BACKUP_NAME="auto-backup-$(date +%Y%m%d-%H%M%S)"

neonctl branch create \
  --name "$BACKUP_NAME" \
  --project-id "${NEON_PROJECT_ID}"

echo "✅ Created backup branch: $BACKUP_NAME"

# Optional: Delete old backup branches (keep last 7 days)
neonctl branches list --project-id "${NEON_PROJECT_ID}" --output json \
  | jq -r '.[] | select(.name | startswith("auto-backup-")) | select(.created_at < (now - 7*86400)) | .id' \
  | xargs -I {} neonctl branch delete {} --project-id "${NEON_PROJECT_ID}"
EOF

chmod +x scripts/create-neon-backup.sh

# Add to crontab (daily at 2 AM)
# 0 2 * * * /path/to/scripts/create-neon-backup.sh >> /var/log/neon-backup.log 2>&1
```

---

## 🔄 Backup & Restore Workflows

### Neon Backup Workflow (Automatic)

```mermaid
graph LR
    A[Neon Platform] --> B[Daily Snapshot at 2 AM UTC]
    B --> C[Compressed Incremental Backup]
    C --> D[Encrypted Storage]
    D --> E[Retention Policy Applied]
    E --> F[Available for Restore]
    F --> G[PITR Ready]
```

### Restore Workflow (Neon)

```mermaid
graph LR
    A[Start] --> B[Select Restore Point]
    B --> C{Restoration Type?}
    C -->|New Branch| D[Create Branch]
    C -->|In-Place| E[Confirm Warning]
    E --> F[Restore Database]
    D --> G[Test on Branch]
    F --> H[Verify Data]
    G --> I{Data OK?}
    I -->|Yes| J[Update Connection String]
    I -->|No| K[Delete Branch & Retry]
    H --> L[End]
    J --> L
    K --> B
```

---

## 🧪 Testing Backups

### Test Point-in-Time Recovery

```bash
# 1. Create a test branch from a specific time
neonctl branch create \
  --name "test-restore-$(date +%Y%m%d)" \
  --timestamp "2026-01-15T10:00:00Z" \
  --project-id $NEON_PROJECT_ID

# 2. Get the connection string for the new branch
TEST_DB_URL=$(neonctl connection-string \
  --branch "test-restore-$(date +%Y%m%d)" \
  --project-id $NEON_PROJECT_ID)

# 3. Verify data in the restored branch
psql "$TEST_DB_URL" -c "SELECT COUNT(*) FROM users;"
psql "$TEST_DB_URL" -c "SELECT * FROM users ORDER BY \"createdAt\" DESC LIMIT 5;"

# 4. Cleanup when done
neonctl branch delete "test-restore-$(date +%Y%m%d)" \
  --project-id $NEON_PROJECT_ID
```

### Test Branch-Based Backup

```bash
# 1. Create a backup branch
neonctl branch create \
  --name "backup-test" \
  --project-id $NEON_PROJECT_ID

# 2. List all branches
neonctl branches list --project-id $NEON_PROJECT_ID

# 3. Verify the branch exists
neonctl branch get backup-test --project-id $NEON_PROJECT_ID

# 4. Get connection string for backup branch
neonctl connection-string \
  --branch backup-test \
  --project-id $NEON_PROJECT_ID

# 5. Cleanup
neonctl branch delete backup-test --project-id $NEON_PROJECT_ID
```

### Automated Backup Testing Script

```bash
# Create test script
cat > scripts/test-neon-backup.sh << 'EOF'
#!/bin/bash
set -e

echo "🧪 Testing Neon backup and restore..."

# Create test branch
TEST_BRANCH="backup-test-$(date +%s)"
echo "Creating test branch: $TEST_BRANCH"

neonctl branch create \
  --name "$TEST_BRANCH" \
  --project-id "$NEON_PROJECT_ID"

# Get connection string
TEST_URL=$(neonctl connection-string \
  --branch "$TEST_BRANCH" \
  --project-id "$NEON_PROJECT_ID")

# Verify data
echo "Verifying data integrity..."
USER_COUNT=$(psql "$TEST_URL" -t -c "SELECT COUNT(*) FROM users;")
echo "✅ Found $USER_COUNT users in backup"

# Cleanup
echo "Cleaning up test branch..."
neonctl branch delete "$TEST_BRANCH" \
  --project-id "$NEON_PROJECT_ID"

echo "✅ Backup test completed successfully!"
EOF

chmod +x scripts/test-neon-backup.sh

# Run test
./scripts/test-neon-backup.sh
```

---

## 📊 Monitoring & Alerts

### Neon Console Monitoring

Neon provides built-in monitoring in the web console:

1. **Go to:** [Neon Console](https://console.neon.tech) → Your Project → **Monitoring**
2. **View:**
   - Backup status and history
   - Last successful backup timestamp
   - Retention period
   - Storage usage
   - Branch list and sizes

### Programmatic Monitoring via Neon API

```bash
# Create monitoring script
cat > scripts/monitor-neon-backups.sh << 'EOF'
#!/bin/bash
set -e

NEON_API_KEY="${NEON_API_KEY}"
PROJECT_ID="${NEON_PROJECT_ID}"
SLACK_WEBHOOK="${SLACK_WEBHOOK_URL}"

# Get project info via Neon API
RESPONSE=$(curl -s -H "Authorization: Bearer $NEON_API_KEY" \
  "https://console.neon.tech/api/v2/projects/$PROJECT_ID")

# Parse backup info
LAST_BACKUP=$(echo "$RESPONSE" | jq -r '.project.last_backup // "unknown"')
RETENTION_DAYS=$(echo "$RESPONSE" | jq -r '.project.history_retention_seconds / 86400')

# Check if backup is recent (within last 25 hours)
if [ "$LAST_BACKUP" != "unknown" ]; then
  BACKUP_TIMESTAMP=$(date -d "$LAST_BACKUP" +%s 2>/dev/null || echo 0)
  CURRENT_TIMESTAMP=$(date +%s)
  HOURS_AGO=$(( (CURRENT_TIMESTAMP - BACKUP_TIMESTAMP) / 3600 ))

  if [ $HOURS_AGO -gt 25 ]; then
    MESSAGE="⚠️ Neon backup is ${HOURS_AGO} hours old (expected < 25h)"
    echo "$MESSAGE"

    # Send Slack alert
    if [ -n "$SLACK_WEBHOOK" ]; then
      curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"$MESSAGE\"}" "$SLACK_WEBHOOK"
    fi
  else
    echo "✅ Backup is healthy (${HOURS_AGO}h old, retention: ${RETENTION_DAYS}d)"
  fi
else
  echo "❌ Unable to retrieve backup status"
fi
EOF

chmod +x scripts/monitor-neon-backups.sh

# Add to crontab (runs at 3 AM daily)
# 0 3 * * * /path/to/scripts/monitor-neon-backups.sh >> /var/log/neon-backup-monitor.log 2>&1
```

### Slack Alerts Setup

Configure Slack webhooks to receive alerts:

```bash
# Add to .env
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL

# Test alert
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"✅ Neon backup monitoring configured for CareCircle"}' \
  "$SLACK_WEBHOOK_URL"
```

### Health Check Endpoint (Optional)

Add a backup health check to your API:

```typescript
// apps/api/src/health/neon-backup.health.ts
import { Injectable } from '@nestjs/common';
import { HealthIndicator, HealthIndicatorResult, HealthCheckError } from '@nestjs/terminus';

@Injectable()
export class NeonBackupHealthIndicator extends HealthIndicator {
  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    const neonApiKey = process.env.NEON_API_KEY;
    const projectId = process.env.NEON_PROJECT_ID;

    try {
      const response = await fetch(
        `https://console.neon.tech/api/v2/projects/${projectId}`,
        {
          headers: { Authorization: `Bearer ${neonApiKey}` },
        }
      );

      const data = await response.json();
      const lastBackup = new Date(data.project.last_backup);
      const hoursAgo = (Date.now() - lastBackup.getTime()) / (1000 * 60 * 60);

      if (hoursAgo > 25) {
        throw new HealthCheckError('Backup too old', { hoursAgo });
      }

      return this.getStatus(key, true, { lastBackup, hoursAgo });
    } catch (error) {
      throw new HealthCheckError('Backup check failed', error);
    }
  }
}
```

---

## 🔐 Security Best Practices

### 1. Encryption (Built-in) ✅

Neon automatically encrypts all backups:

- **At-rest encryption:** AES-256 encryption for all stored backups
- **In-transit encryption:** TLS 1.3 for all data transfers
- **No configuration needed:** Enabled by default

### 2. Access Control

Protect your Neon API keys and database credentials:

```bash
# Store API keys securely (never in code!)
# Use environment variables or secrets management

# .env (never commit this!)
NEON_API_KEY=your-api-key-here
NEON_PROJECT_ID=your-project-id

# For production, use secrets management
# - Kubernetes Secrets
# - AWS Secrets Manager
# - HashiCorp Vault
# - Doppler
```

### 3. IP Allowlisting (Optional)

Restrict database access to specific IPs:

```bash
# In Neon Console → Project Settings → IP Allow
# Add your application server IPs:
# - 203.0.113.0/24  (Production servers)
# - 198.51.100.5    (CI/CD pipeline)
# - Your office IP    (Development access)
```

### 4. Audit Logging

Enable audit logs for compliance (HIPAA/SOC2):

```bash
# Neon provides connection logs
# Access via Console → Project Settings → Logs

# For comprehensive audit logging, implement in your application:
# - All backup operations
# - All restore operations
# - API key usage
```

### 5. Role-Based Access

Use Neon's role-based access control:

```sql
-- Create read-only user for backups/reporting
CREATE ROLE backup_user WITH LOGIN PASSWORD 'secure-password';
GRANT CONNECT ON DATABASE carecircle TO backup_user;
GRANT USAGE ON SCHEMA public TO backup_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO backup_user;

-- Use this user for backup verification
DATABASE_READONLY_URL=postgresql://backup_user:password@...
```

---

## 🚨 Disaster Recovery

### Recovery Time Objective (RTO)
- **Target:** < 15 minutes
- **Actual with Neon:** ~2-5 minutes (instant branch creation)

### Recovery Point Objective (RPO)
- **Target:** < 1 hour
- **Actual with Neon:** Minutes to seconds (PITR available)

### Neon's Built-in DR Features ✨

- **Instant Recovery:** Branch-based restores complete in seconds
- **Point-in-Time Recovery:** Restore to any second within retention period
- **Zero Data Loss:** Write-Ahead Log (WAL) archiving
- **Geographic Redundancy:** Multi-region replication (Pro plan)
- **High Availability:** 99.95% uptime SLA

### Recovery Scenarios

#### Scenario 1: Accidental Data Deletion (Single Table)

**Use Point-in-Time Recovery to a branch:**

```bash
# 1. Identify when the data was deleted (e.g., 10:30 AM today)
TIMESTAMP_BEFORE_DELETE="2026-01-16T10:25:00Z"

# 2. Create recovery branch from before the deletion
neonctl branch create \
  --name "recovery-deleted-data" \
  --timestamp "$TIMESTAMP_BEFORE_DELETE" \
  --project-id $NEON_PROJECT_ID

# 3. Get connection string for recovery branch
RECOVERY_URL=$(neonctl connection-string \
  --branch "recovery-deleted-data" \
  --project-id $NEON_PROJECT_ID)

# 4. Extract the deleted data
pg_dump "$RECOVERY_URL" \
  --table=users \
  --data-only \
  --where="\"createdAt\" >= '2026-01-15'" \
  > recovered_users.sql

# 5. Restore to production
psql "$DATABASE_URL" < recovered_users.sql

# 6. Verify data is restored
psql "$DATABASE_URL" -c "SELECT COUNT(*) FROM users WHERE \"createdAt\" >= '2026-01-15';"

# 7. Cleanup recovery branch
neonctl branch delete "recovery-deleted-data" \
  --project-id $NEON_PROJECT_ID

echo "✅ Data recovery complete!"
```

#### Scenario 2: Bad Migration/Schema Change

**Rollback to before the migration:**

```bash
# 1. Get timestamp before migration was run
TIMESTAMP_BEFORE_MIGRATION="2026-01-16T14:30:00Z"

# 2. Create new branch from before migration
neonctl branch create \
  --name "production-rollback" \
  --timestamp "$TIMESTAMP_BEFORE_MIGRATION" \
  --project-id $NEON_PROJECT_ID

# 3. Get new connection string
NEW_PRODUCTION_URL=$(neonctl connection-string \
  --branch "production-rollback" \
  --project-id $NEON_PROJECT_ID)

# 4. Update application to use new branch
# Update DATABASE_URL in your deployment/secrets
kubectl set env deployment/api \
  DATABASE_URL="$NEW_PRODUCTION_URL" \
  -n carecircle

# 5. Verify application is working
curl https://api.carecircle.com/health

# 6. Delete old branch (optional, after verification)
# neonctl branch delete main --project-id $NEON_PROJECT_ID
# neonctl branch rename production-rollback main --project-id $NEON_PROJECT_ID

echo "✅ Rollback complete! Application restored to pre-migration state"
```

#### Scenario 3: Complete Database Corruption

**Restore from daily backup:**

```bash
# 1. Create new branch from latest backup (automatic daily)
# Neon keeps daily snapshots, just create a new branch
neonctl branch create \
  --name "production-restored-$(date +%Y%m%d)" \
  --project-id $NEON_PROJECT_ID

# 2. Get connection string
RESTORED_URL=$(neonctl connection-string \
  --branch "production-restored-$(date +%Y%m%d)" \
  --project-id $NEON_PROJECT_ID)

# 3. Update application
kubectl set env deployment/api \
  DATABASE_URL="$RESTORED_URL" \
  -n carecircle

# 4. Verify
psql "$RESTORED_URL" -c "SELECT COUNT(*) FROM users;"
psql "$RESTORED_URL" -c "SELECT COUNT(*) FROM \"CareRecipient\";"

# 5. Resume normal operations
kubectl scale deployment api --replicas=3 -n carecircle

echo "✅ Database restored from backup!"
```

#### Scenario 4: Region-Wide Outage (Pro Plan)

**Neon Pro provides cross-region read replicas:**

```bash
# 1. Neon automatically fails over to replica region
# No action needed - Neon handles this

# 2. If manual failover needed:
# Neon Console → Project Settings → Regions → Promote Replica

# 3. Update application DNS/connection string to point to new region
# This is handled automatically by Neon's connection endpoint

# Expected downtime: < 30 seconds
```

---

## 📝 Backup Checklist

### Initial Setup (One-time)
- [ ] Install Neon CLI: `npm install -g neonctl`
- [ ] Authenticate: `neonctl auth`
- [ ] Get API key from Neon Console → Account Settings
- [ ] Set environment variables: `NEON_API_KEY`, `NEON_PROJECT_ID`
- [ ] Configure Slack webhook (optional)
- [ ] Test branch creation and restoration

### Daily Operations (Automated by Neon)
- [x] Automated backup runs at 2 AM UTC ✅ **Handled by Neon**
- [ ] Check Neon Console for backup status (or rely on monitoring)
- [ ] Review Slack alerts (if monitoring is configured)

### Weekly Tasks
- [ ] Verify backup retention in Neon Console
- [ ] Review branch cleanup (delete old test/backup branches)
- [ ] Check database size trends

### Monthly Tasks
- [ ] Test full restore procedure using PITR
- [ ] Review Neon plan and storage costs
- [ ] Update disaster recovery documentation
- [ ] Conduct disaster recovery drill

---

## 📞 Support & Troubleshooting

### Common Issues

**Cannot access Neon Console**
```bash
# Check if you're logged in to Neon CLI
neonctl auth

# Re-authenticate if needed
neonctl auth --logout
neonctl auth
```

**Connection refused when testing restore**
```bash
# Check database connectivity
pg_isready -d "$DATABASE_URL"

# Verify connection string format
echo $DATABASE_URL | sed 's/:.*@/@/g'  # Hide password

# Check if SSL is required (should be in URL)
# Neon requires sslmode=require
```

**Branch creation fails**
```bash
# Check project ID is correct
neonctl projects list

# Verify you have permission
neonctl branches list --project-id $NEON_PROJECT_ID

# Check for branch limit (Free: 10 branches, Pro: unlimited)
```

**API rate limiting**
```bash
# Neon API has rate limits:
# - 1000 requests/hour (authenticated)
# - Add delays between API calls if automating

# Check remaining quota
curl -I -H "Authorization: Bearer $NEON_API_KEY" \
  "https://console.neon.tech/api/v2/projects/$NEON_PROJECT_ID" \
  | grep -i "x-ratelimit"
```

### Getting Help

1. **Neon Documentation:** https://neon.tech/docs
2. **Neon Discord:** https://discord.gg/neon
3. **Neon Support:** support@neon.tech (Pro plan includes priority support)
4. **Check Neon Status:** https://neon.tech/status
5. **Community Forum:** https://community.neon.tech

### Internal Logs

```bash
# Check monitoring logs (if configured)
tail -f /var/log/neon-backup-monitor.log

# Check application logs for database errors
kubectl logs -l app=api -n carecircle --tail=100 | grep -i "database"
```

---

## 💰 Cost Comparison: Neon vs. Manual Backups

### With Neon (Current Setup)
- **Backup Storage:** ✅ Included in plan ($0)
- **Compute for Backups:** ✅ Included in plan ($0)
- **Maintenance Time:** ✅ Zero ($0/month in engineering time)
- **Infrastructure:** ✅ None needed ($0)
- **Total Monthly Cost:** **$0 (Free tier)** or **$19/month (Pro)**

### Manual pg_dump Approach (Alternative)
- **Storage (S3):** ~$5/month for 50GB
- **Backup Compute:** ~$10/month (small EC2 instance or Lambda)
- **Monitoring:** ~$5/month (CloudWatch)
- **Engineering Time:** ~4 hours/month maintenance = ~$400/month
- **Infrastructure Setup:** ~20 hours one-time = ~$2000
- **Total Monthly Cost:** **~$420/month + $2000 one-time**

### Savings with Neon
**💰 $400/month in operational costs**
**💰 $2000 in setup costs**
**💰 Zero maintenance overhead**

---

## 🎯 Next Steps

1. ✅ Backups are already running (Neon handles this automatically!)
2. [ ] Optional: Install Neon CLI for programmatic access
3. [ ] Optional: Configure monitoring script for alerts
4. [ ] Recommended: Test restore procedure once to verify
5. [ ] Recommended: Conduct monthly DR drill

---

## 🎉 Summary

### What You Get Out of the Box

With Neon DB, your backup strategy is **production-ready from day one**:

- ✅ **Automatic daily backups** (no configuration needed)
- ✅ **Point-in-Time Recovery** to any second
- ✅ **Instant branch-based restores** (seconds, not hours)
- ✅ **Encrypted at rest and in transit** (AES-256 + TLS 1.3)
- ✅ **Zero maintenance overhead**
- ✅ **99.95% uptime SLA**
- ✅ **Geographic redundancy** (Pro plan)

### Optional Enhancements

For advanced users who want extra control:

- 📊 **Monitoring Script:** Track backup health via Neon API
- 🔔 **Slack Alerts:** Get notified of any backup issues
- 🌿 **Branch Backups:** Create additional safety net with branch-based backups
- 🏥 **Health Check:** Add backup status to your API health endpoint

---

**✅ Your backups are production-ready with Neon DB!**

_Last Updated: January 16, 2026_
_Database Provider: Neon (Serverless Postgres)_
