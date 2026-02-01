# 💾 Database Backup & Recovery - Complete Guide

> A comprehensive guide to database backup and recovery - point-in-time recovery, disaster recovery, backup strategies, and ensuring data durability.

---

## 🧠 MUST REMEMBER TO IMPRESS

### 1-Liner Definition
> "Database backup and recovery involves creating consistent copies of data that can restore the database to a specific point in time, with strategies defined by RPO (how much data can you lose) and RTO (how fast must you recover)."

### Key Terms
| Term | Meaning |
|------|---------|
| **RPO** | Recovery Point Objective (acceptable data loss: 1 hour, 5 minutes, 0) |
| **RTO** | Recovery Time Objective (max downtime: 4 hours, 15 minutes) |
| **PITR** | Point-In-Time Recovery (restore to any moment) |
| **WAL** | Write-Ahead Log (enables PITR) |
| **Full backup** | Complete copy of database |
| **Incremental** | Only changes since last backup |

---

## Core Concepts

```
BACKUP STRATEGIES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  FULL BACKUP (pg_dump / mysqldump)                             │
│  • Complete copy of all data                                   │
│  • Large size, long time                                       │
│  • Weekly typically                                            │
│                                                                  │
│  INCREMENTAL (WAL archiving)                                   │
│  • Only changes since last backup                              │
│  • Small, fast                                                 │
│  • Continuous or hourly                                        │
│                                                                  │
│  POINT-IN-TIME RECOVERY:                                        │
│  Full backup + WAL files = restore to any moment              │
│                                                                  │
│  ┌──────┐   ┌─────┐   ┌─────┐   ┌─────┐   ┌─────────────┐    │
│  │ Full │ + │ WAL │ + │ WAL │ + │ WAL │ = │ Any point   │    │
│  │ Sun  │   │ Mon │   │ Tue │   │ Wed │   │ recovery    │    │
│  └──────┘   └─────┘   └─────┘   └─────┘   └─────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### PostgreSQL PITR Setup

```bash
# ════════════════════════════════════════════════════════════════
# POSTGRESQL CONTINUOUS ARCHIVING
# ════════════════════════════════════════════════════════════════

# postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'aws s3 cp %p s3://backups/wal/%f'

# Take base backup
pg_basebackup -D /backup/base -Ft -z -P

# Recovery (restore to specific time)
# 1. Restore base backup
# 2. Create recovery.conf:
restore_command = 'aws s3 cp s3://backups/wal/%f %p'
recovery_target_time = '2024-01-15 14:30:00'
recovery_target_action = 'promote'
```

### Backup Testing (Critical!)

```bash
# ════════════════════════════════════════════════════════════════
# ALWAYS TEST RESTORES!
# ════════════════════════════════════════════════════════════════

# Monthly restore test:
# 1. Spin up test database server
# 2. Restore from backup
# 3. Verify data integrity
# 4. Check row counts match production
# 5. Document restore time (meets RTO?)

# Automated verification
pg_restore -d test_db backup.dump
psql -d test_db -c "SELECT COUNT(*) FROM orders;"
# Compare with production count
```

---

## RPO/RTO Planning

```
DISASTER RECOVERY TIERS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  TIER 1: Critical (payments, orders)                           │
│  RPO: 0 (no data loss)                                         │
│  RTO: < 15 minutes                                             │
│  Solution: Synchronous replication + auto-failover             │
│                                                                  │
│  TIER 2: Important (user data)                                 │
│  RPO: < 5 minutes                                              │
│  RTO: < 1 hour                                                 │
│  Solution: Async replication + WAL archiving                   │
│                                                                  │
│  TIER 3: Standard (logs, analytics)                            │
│  RPO: < 24 hours                                               │
│  RTO: < 4 hours                                                │
│  Solution: Daily backups                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "What are RPO and RTO?"**
> "RPO is how much data loss is acceptable - if RPO is 1 hour, you need backups at least hourly. RTO is how fast you must recover - if RTO is 15 minutes, you need hot standby, not restore from backup. These drive your backup strategy."

**Q: "How do you ensure backups are actually working?"**
> "Regular restore testing! Monthly, restore to a test server and verify data integrity. Check that row counts match, critical data exists, and restore time meets RTO. An untested backup is not a backup."

---

## Quick Reference

```
BACKUP & RECOVERY CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  STRATEGIES:                                                    │
│  • Full: Complete copy, weekly                                 │
│  • Incremental: Changes only, daily/hourly                    │
│  • PITR: Continuous WAL archiving                              │
│                                                                  │
│  KEY METRICS:                                                   │
│  • RPO: Acceptable data loss (0 to 24h)                        │
│  • RTO: Max recovery time (15min to 4h)                        │
│                                                                  │
│  BEST PRACTICES:                                                │
│  • Test restores monthly                                       │
│  • Backup to different region/account                          │
│  • Encrypt backups at rest                                     │
│  • Monitor backup jobs                                         │
│  • Document runbooks                                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```
