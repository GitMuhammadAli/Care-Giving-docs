# 🆘 Disaster Recovery - Complete Guide

> A comprehensive guide to disaster recovery - backup strategies, RTO, RPO, failover, business continuity, and DR testing.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Disaster Recovery (DR) is the strategy and processes to restore critical systems and data after a catastrophic event, defined by RTO (Recovery Time Objective - how fast you recover) and RPO (Recovery Point Objective - how much data loss is acceptable)."

### The 7 Key Concepts (Remember These!)
```
1. RTO (Recovery Time Objective)  → Max acceptable downtime
2. RPO (Recovery Point Objective) → Max acceptable data loss
3. BACKUP                        → Copy of data for restoration
4. REPLICATION                   → Real-time data copying
5. FAILOVER                      → Switching to backup systems
6. FAILBACK                      → Returning to primary systems
7. DR SITE                       → Secondary location for recovery
```

### RTO vs RPO
```
┌─────────────────────────────────────────────────────────────────┐
│                       RTO vs RPO                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Timeline: Data Loss ◀────────────────────▶ Downtime           │
│                                                                 │
│  ─────────┼─────────────────┼─────────────────┼─────────────   │
│           │                 │                 │                 │
│      Last Backup      DISASTER          System Restored        │
│                                                                 │
│           │◀────RPO────▶│                                      │
│           │              │◀─────────RTO─────────▶│             │
│                                                                 │
│  RPO = Recovery Point Objective                                │
│  ─────────────────────────────                                  │
│  "How much data can we afford to lose?"                        │
│  RPO = 1 hour → Can lose up to 1 hour of data                  │
│  RPO = 0 → Zero data loss (synchronous replication)            │
│                                                                 │
│  RTO = Recovery Time Objective                                 │
│  ─────────────────────────────                                  │
│  "How long can we be down?"                                    │
│  RTO = 4 hours → Must be back online within 4 hours            │
│  RTO = 0 → No downtime (active-active)                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### DR Strategy Tiers
```
┌─────────────────────────────────────────────────────────────────┐
│                    DR STRATEGY TIERS                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TIER 1: Backup & Restore (Cold)                               │
│  ────────────────────────────────                               │
│  • Periodic backups to offsite/cloud                           │
│  • Manual restoration                                          │
│  • RTO: Days | RPO: Hours to days                              │
│  • Cost: $                                                     │
│                                                                 │
│  TIER 2: Pilot Light                                           │
│  ───────────────────                                            │
│  • Minimal infrastructure always running                        │
│  • Data replicated, compute scaled up on failover              │
│  • RTO: Hours | RPO: Minutes to hours                          │
│  • Cost: $$                                                    │
│                                                                 │
│  TIER 3: Warm Standby                                          │
│  ────────────────────                                           │
│  • Scaled-down version of production running                   │
│  • Continuous data replication                                 │
│  • Scale up on failover                                        │
│  • RTO: Minutes to hours | RPO: Seconds to minutes             │
│  • Cost: $$$                                                   │
│                                                                 │
│  TIER 4: Hot Standby / Active-Active                           │
│  ───────────────────────────────────                            │
│  • Full production capacity in multiple regions                │
│  • Synchronous replication                                     │
│  • Automatic failover                                          │
│  • RTO: Seconds | RPO: Zero                                    │
│  • Cost: $$$$                                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"RTO/RPO"** | "Our SLA requires RTO of 1 hour and RPO of 15 minutes" |
| **"PITR"** | "Point-in-time recovery lets us restore to any second" |
| **"Failover"** | "Automated failover triggers when primary health check fails" |
| **"Multi-region"** | "We run multi-region active-active for zero-downtime DR" |
| **"Chaos engineering"** | "We use chaos engineering to validate DR procedures" |
| **"Runbook"** | "DR runbook documents step-by-step recovery procedures" |

### Key Numbers to Remember
| Metric | Typical Value | Notes |
|--------|---------------|-------|
| Tier 1 RTO | **Days** | Backup/restore |
| Tier 2 RTO | **Hours** | Pilot light |
| Tier 3 RTO | **Minutes** | Warm standby |
| Tier 4 RTO | **Seconds** | Hot/active-active |
| Backup frequency | **Daily + hourly** | Full + incremental |
| Backup retention | **30-90 days** | Compliance dependent |
| DR test frequency | **Quarterly** | At minimum |

### The "Wow" Statement (Memorize This!)
> "We designed our DR strategy for RTO of 15 minutes and RPO of 5 minutes. Primary region is us-east-1 with DR in us-west-2. Databases use Aurora Global Database with async replication (typically <1 second lag). Application tier is deployed to both regions via CI/CD. Route53 health checks detect primary failure and automatically route traffic to DR region. We use S3 cross-region replication for static assets and backups. Database point-in-time recovery is enabled for the last 35 days. We run quarterly DR drills - actually failing over to DR region and running production traffic. Runbooks document every step. Last drill achieved 12-minute RTO with zero data loss."

---

## 📚 Table of Contents

1. [Backup Strategies](#1-backup-strategies)
2. [Database DR](#2-database-dr)
3. [Multi-Region Architecture](#3-multi-region-architecture)
4. [Failover Automation](#4-failover-automation)
5. [DR Testing](#5-dr-testing)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Backup Strategies

```hcl
# ════════════════════════════════════════════════════════════════
# AWS BACKUP CONFIGURATION - TERRAFORM
# ════════════════════════════════════════════════════════════════

# Backup vault
resource "aws_backup_vault" "main" {
  name        = "production-backup-vault"
  kms_key_arn = aws_kms_key.backup.arn

  tags = {
    Environment = "production"
  }
}

# Cross-region backup vault
resource "aws_backup_vault" "dr" {
  provider = aws.dr_region
  name     = "dr-backup-vault"
}

# Backup plan
resource "aws_backup_plan" "production" {
  name = "production-backup-plan"

  # Daily backups
  rule {
    rule_name         = "daily-backup"
    target_vault_name = aws_backup_vault.main.name
    schedule          = "cron(0 5 * * ? *)"  # 5 AM UTC daily

    lifecycle {
      delete_after = 35  # 35 days retention
    }

    # Copy to DR region
    copy_action {
      destination_vault_arn = aws_backup_vault.dr.arn
      lifecycle {
        delete_after = 35
      }
    }

    # Recovery point tags
    recovery_point_tags = {
      BackupType = "daily"
    }
  }

  # Hourly backups (incremental)
  rule {
    rule_name         = "hourly-backup"
    target_vault_name = aws_backup_vault.main.name
    schedule          = "cron(0 * * * ? *)"  # Every hour

    lifecycle {
      delete_after = 7  # 7 days retention
    }

    recovery_point_tags = {
      BackupType = "hourly"
    }
  }

  # Monthly backups (compliance)
  rule {
    rule_name         = "monthly-backup"
    target_vault_name = aws_backup_vault.main.name
    schedule          = "cron(0 5 1 * ? *)"  # 1st of month

    lifecycle {
      cold_storage_after = 30
      delete_after       = 365  # 1 year retention
    }

    copy_action {
      destination_vault_arn = aws_backup_vault.dr.arn
      lifecycle {
        delete_after = 365
      }
    }
  }
}

# Backup selection - what to backup
resource "aws_backup_selection" "production" {
  name         = "production-resources"
  iam_role_arn = aws_iam_role.backup.arn
  plan_id      = aws_backup_plan.production.id

  # Select by tag
  selection_tag {
    type  = "STRINGEQUALS"
    key   = "Backup"
    value = "true"
  }

  # Or specific resources
  resources = [
    aws_db_instance.main.arn,
    aws_dynamodb_table.main.arn,
    aws_efs_file_system.main.arn,
  ]
}

# S3 Cross-Region Replication
resource "aws_s3_bucket_replication_configuration" "backup" {
  bucket = aws_s3_bucket.data.id
  role   = aws_iam_role.replication.arn

  rule {
    id     = "replicate-all"
    status = "Enabled"

    filter {}

    destination {
      bucket        = aws_s3_bucket.dr_data.arn
      storage_class = "STANDARD_IA"
    }

    delete_marker_replication {
      status = "Enabled"
    }
  }
}
```

---

## 2. Database DR

```hcl
# ════════════════════════════════════════════════════════════════
# DATABASE DR STRATEGIES
# ════════════════════════════════════════════════════════════════

# Aurora Global Database (Multi-Region)
resource "aws_rds_global_cluster" "main" {
  global_cluster_identifier = "production-global"
  engine                    = "aurora-postgresql"
  engine_version            = "15.4"
  database_name             = "myapp"
}

# Primary cluster
resource "aws_rds_cluster" "primary" {
  cluster_identifier        = "production-primary"
  engine                    = "aurora-postgresql"
  engine_version            = "15.4"
  global_cluster_identifier = aws_rds_global_cluster.main.id
  master_username           = "admin"
  master_password           = var.db_password

  # Enable PITR
  backup_retention_period = 35
  preferred_backup_window = "03:00-04:00"

  # Enable deletion protection
  deletion_protection = true
}

# Secondary cluster (DR region)
resource "aws_rds_cluster" "secondary" {
  provider = aws.dr_region

  cluster_identifier        = "production-secondary"
  engine                    = "aurora-postgresql"
  engine_version            = "15.4"
  global_cluster_identifier = aws_rds_global_cluster.main.id

  # Secondary is read-only until failover
  backup_retention_period = 35

  depends_on = [aws_rds_cluster.primary]
}

# RDS Read Replica (Cross-Region)
resource "aws_db_instance" "replica" {
  provider = aws.dr_region

  identifier          = "production-replica"
  replicate_source_db = aws_db_instance.primary.arn

  instance_class         = "db.r6g.large"
  storage_encrypted      = true
  auto_minor_version_upgrade = true

  # Can be promoted to standalone in DR
}
```

```sql
-- ════════════════════════════════════════════════════════════════
-- POSTGRESQL POINT-IN-TIME RECOVERY
-- ════════════════════════════════════════════════════════════════

-- Check current WAL position
SELECT pg_current_wal_lsn();

-- Enable WAL archiving (postgresql.conf)
-- archive_mode = on
-- archive_command = 'aws s3 cp %p s3://my-wal-archive/%f'

-- Restore to specific point in time
-- In recovery.conf:
-- restore_command = 'aws s3 cp s3://my-wal-archive/%f %p'
-- recovery_target_time = '2024-01-15 10:30:00'
-- recovery_target_action = 'promote'
```

---

## 3. Multi-Region Architecture

```hcl
# ════════════════════════════════════════════════════════════════
# MULTI-REGION DR ARCHITECTURE
# ════════════════════════════════════════════════════════════════

# Route53 Health Check
resource "aws_route53_health_check" "primary" {
  fqdn              = "api.us-east-1.example.com"
  port              = 443
  type              = "HTTPS"
  resource_path     = "/health"
  failure_threshold = "3"
  request_interval  = "10"

  tags = {
    Name = "primary-health-check"
  }
}

# Route53 Failover Routing
resource "aws_route53_record" "api_primary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "api.example.com"
  type    = "A"

  failover_routing_policy {
    type = "PRIMARY"
  }

  set_identifier  = "primary"
  health_check_id = aws_route53_health_check.primary.id

  alias {
    name                   = aws_lb.primary.dns_name
    zone_id                = aws_lb.primary.zone_id
    evaluate_target_health = true
  }
}

resource "aws_route53_record" "api_secondary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "api.example.com"
  type    = "A"

  failover_routing_policy {
    type = "SECONDARY"
  }

  set_identifier = "secondary"

  alias {
    name                   = aws_lb.secondary.dns_name
    zone_id                = aws_lb.secondary.zone_id
    evaluate_target_health = true
  }
}

# Active-Active with Latency Routing
resource "aws_route53_record" "api_us_east" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "api.example.com"
  type    = "A"

  latency_routing_policy {
    region = "us-east-1"
  }

  set_identifier  = "us-east-1"
  health_check_id = aws_route53_health_check.us_east.id

  alias {
    name                   = aws_lb.us_east.dns_name
    zone_id                = aws_lb.us_east.zone_id
    evaluate_target_health = true
  }
}

resource "aws_route53_record" "api_us_west" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "api.example.com"
  type    = "A"

  latency_routing_policy {
    region = "us-west-2"
  }

  set_identifier  = "us-west-2"
  health_check_id = aws_route53_health_check.us_west.id

  alias {
    name                   = aws_lb.us_west.dns_name
    zone_id                = aws_lb.us_west.zone_id
    evaluate_target_health = true
  }
}
```

---

## 4. Failover Automation

```python
#!/usr/bin/env python3
# ════════════════════════════════════════════════════════════════
# AUTOMATED FAILOVER SCRIPT
# ════════════════════════════════════════════════════════════════

import boto3
import time
from datetime import datetime

class DisasterRecoveryManager:
    def __init__(self, primary_region, dr_region):
        self.primary_region = primary_region
        self.dr_region = dr_region
        self.rds_primary = boto3.client('rds', region_name=primary_region)
        self.rds_dr = boto3.client('rds', region_name=dr_region)
        self.route53 = boto3.client('route53')
        
    def check_primary_health(self, health_check_id):
        """Check if primary region is healthy"""
        cloudwatch = boto3.client('cloudwatch', region_name='us-east-1')
        
        response = cloudwatch.get_metric_statistics(
            Namespace='AWS/Route53',
            MetricName='HealthCheckStatus',
            Dimensions=[{'Name': 'HealthCheckId', 'Value': health_check_id}],
            StartTime=datetime.utcnow() - timedelta(minutes=5),
            EndTime=datetime.utcnow(),
            Period=60,
            Statistics=['Average']
        )
        
        if response['Datapoints']:
            return response['Datapoints'][-1]['Average'] == 1.0
        return False
    
    def failover_aurora_global_cluster(self, global_cluster_id, target_cluster_arn):
        """Failover Aurora Global Database to DR region"""
        print(f"Initiating Aurora failover to {self.dr_region}")
        
        # Remove secondary from global cluster (promotes to standalone)
        self.rds_dr.remove_from_global_cluster(
            GlobalClusterIdentifier=global_cluster_id,
            DbClusterIdentifier=target_cluster_arn
        )
        
        # Wait for cluster to become available
        waiter = self.rds_dr.get_waiter('db_cluster_available')
        waiter.wait(DBClusterIdentifier=target_cluster_arn.split(':')[-1])
        
        print("Aurora failover complete")
        
    def update_dns(self, hosted_zone_id, record_name, new_target):
        """Update Route53 to point to DR"""
        print(f"Updating DNS {record_name} to {new_target}")
        
        self.route53.change_resource_record_sets(
            HostedZoneId=hosted_zone_id,
            ChangeBatch={
                'Changes': [{
                    'Action': 'UPSERT',
                    'ResourceRecordSet': {
                        'Name': record_name,
                        'Type': 'CNAME',
                        'TTL': 60,
                        'ResourceRecords': [{'Value': new_target}]
                    }
                }]
            }
        )
        
    def execute_failover(self, config):
        """Execute complete DR failover"""
        start_time = time.time()
        
        print("="*50)
        print("INITIATING DISASTER RECOVERY FAILOVER")
        print(f"Time: {datetime.utcnow().isoformat()}")
        print("="*50)
        
        # Step 1: Failover database
        self.failover_aurora_global_cluster(
            config['global_cluster_id'],
            config['dr_cluster_arn']
        )
        
        # Step 2: Update application config (if needed)
        # self.update_parameter_store(...)
        
        # Step 3: Update DNS
        self.update_dns(
            config['hosted_zone_id'],
            config['record_name'],
            config['dr_endpoint']
        )
        
        # Step 4: Notify team
        self.send_notification(
            f"DR Failover Complete. RTO: {time.time() - start_time:.0f} seconds"
        )
        
        elapsed = time.time() - start_time
        print(f"\nFailover completed in {elapsed:.0f} seconds")
        return elapsed

# Usage
if __name__ == "__main__":
    dr_manager = DisasterRecoveryManager('us-east-1', 'us-west-2')
    
    config = {
        'global_cluster_id': 'production-global',
        'dr_cluster_arn': 'arn:aws:rds:us-west-2:...',
        'hosted_zone_id': 'Z1234567890',
        'record_name': 'api.example.com',
        'dr_endpoint': 'api-dr.us-west-2.elb.amazonaws.com'
    }
    
    dr_manager.execute_failover(config)
```

---

## 5. DR Testing

```yaml
# ════════════════════════════════════════════════════════════════
# DR TEST PLAN
# ════════════════════════════════════════════════════════════════

dr_test:
  name: "Q1 2024 DR Drill"
  date: "2024-01-15"
  duration: "4 hours"
  participants:
    - SRE Team
    - Database Team
    - Application Team
    - Security Team

  objectives:
    - Validate RTO of 15 minutes
    - Validate RPO of 5 minutes
    - Test runbook accuracy
    - Train team on DR procedures

  pre_test:
    - [ ] Notify stakeholders
    - [ ] Verify DR environment is ready
    - [ ] Confirm backup integrity
    - [ ] Review runbooks
    - [ ] Set up monitoring dashboards

  test_scenario:
    description: "Simulated us-east-1 region outage"
    steps:
      - time: "T+0"
        action: "Simulate primary region failure"
        method: "Disable health check / block traffic"
        
      - time: "T+1"
        action: "Verify failover triggered"
        expected: "Route53 detects failure within 30 seconds"
        
      - time: "T+2"
        action: "Confirm database failover"
        expected: "Aurora secondary promoted within 5 minutes"
        
      - time: "T+5"
        action: "Verify application in DR region"
        expected: "All services healthy"
        
      - time: "T+10"
        action: "Run smoke tests"
        expected: "Critical paths functional"
        
      - time: "T+15"
        action: "Confirm production traffic flowing"
        expected: "Users can complete transactions"

  metrics_to_capture:
    - Actual RTO (time to full recovery)
    - Actual RPO (data loss, if any)
    - Number of manual interventions
    - Issues encountered
    - Runbook accuracy

  post_test:
    - [ ] Failback to primary region
    - [ ] Document actual RTO/RPO
    - [ ] Capture lessons learned
    - [ ] Update runbooks
    - [ ] Schedule follow-up fixes
```

```bash
#!/bin/bash
# ════════════════════════════════════════════════════════════════
# DR SMOKE TEST SCRIPT
# ════════════════════════════════════════════════════════════════

set -e

DR_ENDPOINT="https://api-dr.example.com"
TESTS_PASSED=0
TESTS_FAILED=0

echo "Running DR Smoke Tests against $DR_ENDPOINT"
echo "============================================"

# Health check
echo -n "Health check... "
if curl -sf "$DR_ENDPOINT/health" > /dev/null; then
    echo "PASS"
    ((TESTS_PASSED++))
else
    echo "FAIL"
    ((TESTS_FAILED++))
fi

# Database connectivity
echo -n "Database connectivity... "
if curl -sf "$DR_ENDPOINT/health/db" > /dev/null; then
    echo "PASS"
    ((TESTS_PASSED++))
else
    echo "FAIL"
    ((TESTS_FAILED++))
fi

# Authentication
echo -n "Authentication... "
TOKEN=$(curl -sf -X POST "$DR_ENDPOINT/auth/login" \
    -H "Content-Type: application/json" \
    -d '{"email":"test@example.com","password":"test"}' | jq -r '.token')
if [ -n "$TOKEN" ] && [ "$TOKEN" != "null" ]; then
    echo "PASS"
    ((TESTS_PASSED++))
else
    echo "FAIL"
    ((TESTS_FAILED++))
fi

# Read operation
echo -n "Read operation... "
if curl -sf -H "Authorization: Bearer $TOKEN" "$DR_ENDPOINT/api/users/me" > /dev/null; then
    echo "PASS"
    ((TESTS_PASSED++))
else
    echo "FAIL"
    ((TESTS_FAILED++))
fi

# Write operation
echo -n "Write operation... "
if curl -sf -X POST -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    -d '{"test": true}' \
    "$DR_ENDPOINT/api/test/write" > /dev/null; then
    echo "PASS"
    ((TESTS_PASSED++))
else
    echo "FAIL"
    ((TESTS_FAILED++))
fi

echo ""
echo "Results: $TESTS_PASSED passed, $TESTS_FAILED failed"

if [ $TESTS_FAILED -gt 0 ]; then
    exit 1
fi
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# DR PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Never testing DR
# Bad
dr_tests: "We'll test when we need it"
# When disaster strikes, procedures don't work!

# Good
dr_tests:
  frequency: quarterly
  type: actual_failover
  documentation: updated_after_each_test

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: DR region not kept up to date
# Bad
# Infrastructure deployed to primary, forget DR
# DR region has old configs, missing services

# Good
# Use IaC to deploy to both regions
# CI/CD deploys to both regions
# Regular verification that DR matches primary

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Only backing up data, not configs
# Bad
backups:
  - databases
# Missing: secrets, configs, certificates, IAM

# Good
backups:
  - databases
  - secrets_manager
  - parameter_store
  - terraform_state
  - certificates
  - dns_configs

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: No runbook or outdated runbook
# Bad
# "I think the command is something like..."
# Steps changed, runbook not updated

# Good
runbook:
  location: wiki.example.com/dr-runbook
  last_updated: 2024-01-15
  last_tested: 2024-01-15
  owner: sre-team@example.com

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: RTO/RPO not aligned with business
# Bad
rto: 24_hours  # Business loses $1M/hour!
cost: minimal

# Good
# Calculate cost of downtime
# Set RTO/RPO based on business impact
# Budget accordingly
business_impact: $1M/hour
rto: 15_minutes
infrastructure_cost: $50K/month  # Worth it!

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Single point of failure in DR process
# Bad
# DR requires manual database promotion by one DBA
# DBA is on vacation during disaster

# Good
# Automated failover
# Multiple trained personnel
# Documented procedures
# Cross-training
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is RTO and RPO?"**
> "RTO (Recovery Time Objective): Maximum acceptable downtime - how long until systems are restored. RPO (Recovery Point Objective): Maximum acceptable data loss - how much data can we lose. Example: RTO=1 hour, RPO=15 minutes means we must be back online within 1 hour and can lose at most 15 minutes of data."

**Q: "What are the DR strategy tiers?"**
> "1) Backup/Restore (Cold): Periodic backups, manual restore. Days RTO. 2) Pilot Light: Minimal infra running, scale up on failover. Hours RTO. 3) Warm Standby: Scaled-down production running, quick scale up. Minutes RTO. 4) Hot/Active-Active: Full capacity in multiple regions. Seconds RTO."

**Q: "What should be included in backups?"**
> "Data: databases, files, object storage. Configuration: secrets, parameters, environment configs. Infrastructure: Terraform state, CloudFormation templates. Application: container images, deployment configs. Documentation: runbooks, architecture diagrams."

### Intermediate Questions

**Q: "How do you test DR?"**
> "Quarterly DR drills with actual failover to DR region. Run production traffic through DR. Measure actual RTO/RPO. Document issues found. Update runbooks. Involve all relevant teams. Some companies do 'game days' with simulated disasters."

**Q: "How does Aurora Global Database work for DR?"**
> "Primary cluster in main region handles writes. Secondary clusters in other regions receive async replication (usually <1 second lag). On failover, secondary promoted to standalone primary. Takes ~1 minute. Cross-region read replicas for read scaling and DR."

**Q: "How do you handle state/sessions in multi-region?"**
> "Externalize sessions to shared store (ElastiCache Global Datastore). Use database for persistent state with replication. Consider eventual consistency implications. Sticky sessions at DNS level if needed. Design applications to be stateless where possible."

### Advanced Questions

**Q: "How do you calculate acceptable RTO/RPO?"**
> "Calculate cost of downtime (revenue loss, SLA penalties, reputation). Calculate cost of data loss. Compare against cost of DR infrastructure. Business decision on acceptable risk. Example: $100K/hour downtime cost → invest in 15-minute RTO infrastructure."

**Q: "How do you handle DR for a database with strict consistency requirements?"**
> "Synchronous replication between regions (impacts latency). Or accept RPO > 0 with async replication. Use distributed databases designed for multi-region (CockroachDB, Spanner). Careful transaction design. Consider whether consistency can be relaxed during DR event."

**Q: "Design a DR strategy for a global e-commerce platform."**
> "Active-active in 3+ regions. Global load balancing (Route53/CloudFront). Each region can handle full load. Global database (Aurora Global, Spanner). Async replication with conflict resolution. Feature flags to disable writes in affected region. Regional caches. Regular DR drills. RTO: seconds, RPO: minimal with conflict resolution."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                      DR CHECKLIST                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BACKUPS:                                                       │
│  □ Databases backed up (daily + hourly)                        │
│  □ Configs/secrets backed up                                   │
│  □ Cross-region replication enabled                            │
│  □ Backup restoration tested                                   │
│                                                                 │
│  DR ENVIRONMENT:                                                │
│  □ DR region infrastructure provisioned                        │
│  □ Same version as production                                  │
│  □ Data replicated continuously                                │
│  □ DNS failover configured                                     │
│                                                                 │
│  DOCUMENTATION:                                                 │
│  □ Runbook exists and is current                               │
│  □ RTO/RPO defined and documented                              │
│  □ Contact list for DR events                                  │
│  □ Post-incident process defined                               │
│                                                                 │
│  TESTING:                                                       │
│  □ DR tested quarterly (minimum)                               │
│  □ Actual failover performed                                   │
│  □ Results documented                                          │
│  □ Issues addressed                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

DR STRATEGY TIERS:
┌────────────────────────────────────────────────────────────────┐
│ Backup/Restore  - RTO: Days    | RPO: Hours   | Cost: $       │
│ Pilot Light     - RTO: Hours   | RPO: Minutes | Cost: $$      │
│ Warm Standby    - RTO: Minutes | RPO: Seconds | Cost: $$$     │
│ Active-Active   - RTO: Seconds | RPO: Zero    | Cost: $$$$    │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

