# ⏱️ Time-Series Databases - Complete Guide

> A comprehensive guide to time-series databases - InfluxDB, TimescaleDB, retention policies, downsampling, and optimizing for time-based data.

---

## 🧠 MUST REMEMBER TO IMPRESS

### 1-Liner Definition
> "Time-series databases are optimized for write-heavy workloads of timestamped data, with features like automatic data retention, downsampling, and time-based queries - handling millions of data points per second."

### Key Terms
| Term | Meaning |
|------|---------|
| **Retention policy** | Auto-delete data older than X days |
| **Downsampling** | Aggregate old data (1min → 1hour averages) |
| **Continuous query** | Auto-running aggregation queries |
| **Cardinality** | Number of unique tag combinations |

---

## Core Concepts

```
TIME-SERIES DATA MODEL:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Measurement: cpu_usage                                         │
│  Tags: host=server1, region=us-east (indexed, low cardinality)  │
│  Fields: value=85.5, temperature=72 (not indexed, values)      │
│  Timestamp: 2024-01-15T10:30:00Z                                │
│                                                                  │
│  Example data points:                                          │
│  cpu_usage,host=server1,region=us-east value=85.5 1705312200   │
│  cpu_usage,host=server2,region=us-west value=72.3 1705312200   │
│                                                                  │
│  OPTIMIZATIONS:                                                 │
│  • Append-only writes (no updates)                             │
│  • Time-based partitioning                                     │
│  • Columnar storage for compression                            │
│  • Automatic indexing on time                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### InfluxDB vs TimescaleDB

```
COMPARISON:
┌───────────────┬─────────────────────┬─────────────────────────┐
│ Feature       │ InfluxDB            │ TimescaleDB             │
├───────────────┼─────────────────────┼─────────────────────────┤
│ Based on      │ Custom engine       │ PostgreSQL extension    │
│ Query lang    │ Flux / InfluxQL     │ SQL                     │
│ JOINs         │ Limited             │ Full SQL JOINs          │
│ Ecosystem     │ Telegraf, Grafana   │ PostgreSQL tools        │
│ Best for      │ Pure metrics        │ Mixed workloads         │
└───────────────┴─────────────────────┴─────────────────────────┘
```

### Implementation

```sql
-- ════════════════════════════════════════════════════════════════
-- TIMESCALEDB SETUP
-- ════════════════════════════════════════════════════════════════

-- Create hypertable (time-partitioned table)
CREATE TABLE metrics (
    time        TIMESTAMPTZ NOT NULL,
    sensor_id   INTEGER,
    temperature DOUBLE PRECISION,
    humidity    DOUBLE PRECISION
);

SELECT create_hypertable('metrics', 'time');

-- Add retention policy (delete after 30 days)
SELECT add_retention_policy('metrics', INTERVAL '30 days');

-- Create continuous aggregate (downsampling)
CREATE MATERIALIZED VIEW metrics_hourly
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 hour', time) AS bucket,
    sensor_id,
    AVG(temperature) as avg_temp,
    MAX(temperature) as max_temp
FROM metrics
GROUP BY bucket, sensor_id;

-- Refresh policy
SELECT add_continuous_aggregate_policy('metrics_hourly',
    start_offset => INTERVAL '3 hours',
    end_offset => INTERVAL '1 hour',
    schedule_interval => INTERVAL '1 hour');

-- Query with time functions
SELECT 
    time_bucket('5 minutes', time) AS five_min,
    AVG(temperature)
FROM metrics
WHERE time > NOW() - INTERVAL '1 hour'
GROUP BY five_min
ORDER BY five_min;
```

---

## Interview Questions

**Q: "When would you use a time-series database?"**
> "For high-volume timestamped data: metrics, IoT sensors, logs, financial ticks. They're optimized for append-only writes, time-range queries, and automatic data lifecycle management. Traditional RDBMS struggle with billions of time-series rows."

**Q: "What is downsampling?"**
> "Aggregating fine-grained data into coarser intervals. Keep 1-second data for 7 days, 1-minute averages for 30 days, 1-hour averages for 1 year. Reduces storage while preserving trends for historical analysis."

---

## Quick Reference

```
TIME-SERIES CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  USE WHEN:                                                      │
│  • Metrics, monitoring, IoT                                    │
│  • Append-only, time-ordered data                              │
│  • Billions of data points                                     │
│                                                                  │
│  KEY FEATURES:                                                  │
│  • Retention policies (auto-delete old)                        │
│  • Downsampling (aggregate over time)                          │
│  • Time-based partitioning                                     │
│  • Efficient compression                                        │
│                                                                  │
│  OPTIONS:                                                       │
│  • InfluxDB: Pure time-series, Flux query                      │
│  • TimescaleDB: PostgreSQL + time-series                       │
│  • Prometheus: Metrics + alerting                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```
