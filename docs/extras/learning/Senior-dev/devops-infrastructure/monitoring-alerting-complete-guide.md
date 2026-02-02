# 📊 Monitoring & Alerting - Complete Guide

> A comprehensive guide to monitoring and alerting - Prometheus, Grafana, Datadog, metrics, dashboards, and alert design patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Monitoring collects and visualizes metrics from systems and applications (CPU, latency, error rates), while alerting triggers notifications when metrics breach thresholds, enabling proactive incident detection and response before users are impacted."

### The 7 Key Concepts (Remember These!)
```
1. METRICS          → Numerical measurements over time (counters, gauges, histograms)
2. TIME SERIES      → Metric values with timestamps
3. LABELS/TAGS      → Key-value metadata for filtering
4. SCRAPING         → Pull-based collection (Prometheus)
5. PUSH             → Push-based collection (StatsD, Datadog)
6. DASHBOARDS       → Visual representation of metrics
7. ALERTS           → Notifications when thresholds breached
```

### Monitoring Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                  MONITORING ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    METRIC SOURCES                         │  │
│  │                                                          │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────┐ │  │
│  │  │  Apps   │  │ Servers │  │   DBs   │  │ Load Balancer│ │  │
│  │  │/metrics │  │/metrics │  │ metrics │  │   metrics   │ │  │
│  │  └────┬────┘  └────┬────┘  └────┬────┘  └──────┬──────┘ │  │
│  └───────┼────────────┼────────────┼──────────────┼────────┘  │
│          │            │            │              │            │
│          └────────────┴────────────┴──────────────┘            │
│                              │                                  │
│                              ▼                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    PROMETHEUS                             │  │
│  │          (Scrapes, Stores, Queries metrics)              │  │
│  │                    PromQL queries                         │  │
│  └────────────────────────┬─────────────────────────────────┘  │
│                           │                                    │
│              ┌────────────┴────────────┐                      │
│              │                         │                      │
│              ▼                         ▼                      │
│  ┌──────────────────────┐  ┌──────────────────────┐          │
│  │       GRAFANA        │  │    ALERTMANAGER      │          │
│  │    (Dashboards)      │  │     (Routing,        │          │
│  │                      │  │    Notifications)    │          │
│  └──────────────────────┘  └──────────────────────┘          │
│                                       │                       │
│                                       ▼                       │
│                            ┌──────────────────────┐          │
│                            │   NOTIFICATIONS      │          │
│                            │  Slack, PagerDuty,   │          │
│                            │  Email, OpsGenie     │          │
│                            └──────────────────────┘          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Four Golden Signals
```
┌─────────────────────────────────────────────────────────────────┐
│                    FOUR GOLDEN SIGNALS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. LATENCY                                                    │
│     ─────────                                                   │
│     Time to serve a request                                    │
│     Metrics: p50, p95, p99 response time                       │
│     Alert: p99 > 500ms for 5 minutes                           │
│                                                                 │
│  2. TRAFFIC                                                     │
│     ───────                                                     │
│     Demand on the system                                       │
│     Metrics: requests/second, concurrent users                 │
│     Alert: Traffic drop > 50% (possible issue)                 │
│                                                                 │
│  3. ERRORS                                                      │
│     ──────                                                      │
│     Rate of failed requests                                    │
│     Metrics: 5xx rate, error percentage                        │
│     Alert: Error rate > 1% for 5 minutes                       │
│                                                                 │
│  4. SATURATION                                                  │
│     ──────────                                                  │
│     How "full" the system is                                   │
│     Metrics: CPU %, memory %, disk I/O, queue depth            │
│     Alert: CPU > 80% for 10 minutes                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Four Golden Signals"** | "We monitor the four golden signals: latency, traffic, errors, saturation" |
| **"SLI/SLO/SLA"** | "Our SLO is 99.9% availability, tracked via error rate SLI" |
| **"Cardinality"** | "We control metric cardinality to keep Prometheus performant" |
| **"Recording rules"** | "Recording rules pre-compute expensive queries" |
| **"Alert fatigue"** | "We tune alerts carefully to avoid alert fatigue" |
| **"On-call runbook"** | "Each alert has a runbook with investigation steps" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Scrape interval | **15-30 seconds** | Balance freshness vs load |
| Alert threshold | **5 minutes** | Avoid flapping |
| P99 latency target | **< 500ms** | Good user experience |
| Error rate threshold | **< 1%** | Typical SLO |
| CPU threshold | **< 80%** | Leave headroom |
| Metric retention | **15-30 days** | Balance storage vs history |

### The "Wow" Statement (Memorize This!)
> "We use Prometheus for metrics collection with Grafana for visualization. Every service exposes /metrics endpoint with RED metrics (Rate, Errors, Duration). We monitor the four golden signals: latency P50/P95/P99, request rate, error percentage, and CPU/memory saturation. Alerts follow SLO-based approach - we alert on 99.9% availability breach using multi-window burn rate. Alertmanager routes to Slack for warnings, PagerDuty for critical. Every alert has a runbook with investigation steps. Recording rules pre-compute expensive queries like percentiles. We use Prometheus federation for multi-cluster, Thanos for long-term storage. Dashboards follow RED method pattern with separate views for service health, dependencies, and business metrics."

---

## 📚 Table of Contents

1. [Prometheus](#1-prometheus)
2. [Grafana](#2-grafana)
3. [Application Metrics](#3-application-metrics)
4. [Alert Design](#4-alert-design)
5. [Datadog](#5-datadog)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Prometheus

```yaml
# ════════════════════════════════════════════════════════════════
# PROMETHEUS CONFIGURATION
# prometheus.yml
# ════════════════════════════════════════════════════════════════

global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: production
    region: us-east-1

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

# Recording and alerting rules
rule_files:
  - /etc/prometheus/rules/*.yml

# Scrape configurations
scrape_configs:
  # Prometheus self-monitoring
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node exporter (system metrics)
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']

  # Kubernetes service discovery
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      # Only scrape pods with prometheus.io/scrape annotation
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      # Use custom port from annotation
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: (\d+)
        target_label: __address__
        replacement: ${1}
      # Add pod labels
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      # Add namespace label
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: namespace
      # Add pod name label
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: pod

  # Application services
  - job_name: 'api-service'
    metrics_path: /metrics
    static_configs:
      - targets: ['api-service:3000']

---
# ════════════════════════════════════════════════════════════════
# RECORDING RULES (Pre-compute expensive queries)
# rules/recording.yml
# ════════════════════════════════════════════════════════════════

groups:
  - name: api_metrics
    interval: 30s
    rules:
      # Request rate by service
      - record: job:http_requests:rate5m
        expr: sum by (job) (rate(http_requests_total[5m]))

      # Error rate by service
      - record: job:http_errors:rate5m
        expr: sum by (job) (rate(http_requests_total{status=~"5.."}[5m]))

      # Error percentage
      - record: job:http_error_rate:ratio
        expr: |
          job:http_errors:rate5m / job:http_requests:rate5m

      # Latency percentiles
      - record: job:http_request_duration_seconds:p99
        expr: |
          histogram_quantile(0.99, 
            sum by (job, le) (rate(http_request_duration_seconds_bucket[5m]))
          )

      - record: job:http_request_duration_seconds:p95
        expr: |
          histogram_quantile(0.95, 
            sum by (job, le) (rate(http_request_duration_seconds_bucket[5m]))
          )

      - record: job:http_request_duration_seconds:p50
        expr: |
          histogram_quantile(0.50, 
            sum by (job, le) (rate(http_request_duration_seconds_bucket[5m]))
          )

---
# ════════════════════════════════════════════════════════════════
# PROMQL EXAMPLES
# ════════════════════════════════════════════════════════════════

# Instant vector (current values)
http_requests_total{job="api"}

# Range vector (values over time)
http_requests_total{job="api"}[5m]

# Rate (per-second increase)
rate(http_requests_total{job="api"}[5m])

# Increase (total increase over time)
increase(http_requests_total{job="api"}[1h])

# Sum by label
sum by (status) (rate(http_requests_total[5m]))

# Histogram percentiles
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))

# Error rate
sum(rate(http_requests_total{status=~"5.."}[5m])) 
/ 
sum(rate(http_requests_total[5m]))

# Top 5 by request rate
topk(5, sum by (endpoint) (rate(http_requests_total[5m])))

# Absent (for up/down detection)
absent(up{job="api"})
```

---

## 2. Grafana

```json
// ════════════════════════════════════════════════════════════════
// GRAFANA DASHBOARD JSON
// ════════════════════════════════════════════════════════════════

{
  "title": "API Service Dashboard",
  "tags": ["api", "production"],
  "timezone": "browser",
  "panels": [
    {
      "title": "Request Rate",
      "type": "timeseries",
      "gridPos": { "x": 0, "y": 0, "w": 8, "h": 8 },
      "targets": [
        {
          "expr": "sum(rate(http_requests_total{job=\"api\"}[5m]))",
          "legendFormat": "Requests/sec"
        }
      ]
    },
    {
      "title": "Error Rate",
      "type": "gauge",
      "gridPos": { "x": 8, "y": 0, "w": 4, "h": 8 },
      "targets": [
        {
          "expr": "sum(rate(http_requests_total{job=\"api\",status=~\"5..\"}[5m])) / sum(rate(http_requests_total{job=\"api\"}[5m])) * 100"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "thresholds": {
            "steps": [
              { "color": "green", "value": null },
              { "color": "yellow", "value": 1 },
              { "color": "red", "value": 5 }
            ]
          },
          "unit": "percent",
          "max": 10
        }
      }
    },
    {
      "title": "Latency Percentiles",
      "type": "timeseries",
      "gridPos": { "x": 12, "y": 0, "w": 12, "h": 8 },
      "targets": [
        {
          "expr": "histogram_quantile(0.99, sum by (le) (rate(http_request_duration_seconds_bucket{job=\"api\"}[5m])))",
          "legendFormat": "p99"
        },
        {
          "expr": "histogram_quantile(0.95, sum by (le) (rate(http_request_duration_seconds_bucket{job=\"api\"}[5m])))",
          "legendFormat": "p95"
        },
        {
          "expr": "histogram_quantile(0.50, sum by (le) (rate(http_request_duration_seconds_bucket{job=\"api\"}[5m])))",
          "legendFormat": "p50"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "s"
        }
      }
    },
    {
      "title": "Status Codes",
      "type": "piechart",
      "gridPos": { "x": 0, "y": 8, "w": 8, "h": 8 },
      "targets": [
        {
          "expr": "sum by (status) (increase(http_requests_total{job=\"api\"}[1h]))",
          "legendFormat": "{{status}}"
        }
      ]
    },
    {
      "title": "CPU Usage",
      "type": "timeseries",
      "gridPos": { "x": 8, "y": 8, "w": 8, "h": 8 },
      "targets": [
        {
          "expr": "rate(process_cpu_seconds_total{job=\"api\"}[5m]) * 100",
          "legendFormat": "CPU %"
        }
      ]
    },
    {
      "title": "Memory Usage",
      "type": "timeseries",
      "gridPos": { "x": 16, "y": 8, "w": 8, "h": 8 },
      "targets": [
        {
          "expr": "process_resident_memory_bytes{job=\"api\"}",
          "legendFormat": "Memory"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "unit": "bytes"
        }
      }
    }
  ],
  "templating": {
    "list": [
      {
        "name": "job",
        "type": "query",
        "query": "label_values(http_requests_total, job)",
        "current": { "text": "api", "value": "api" }
      },
      {
        "name": "instance",
        "type": "query",
        "query": "label_values(http_requests_total{job=\"$job\"}, instance)"
      }
    ]
  }
}
```

---

## 3. Application Metrics

```typescript
// ════════════════════════════════════════════════════════════════
// PROMETHEUS METRICS - NODE.JS APPLICATION
// ════════════════════════════════════════════════════════════════

import express from 'express';
import promClient from 'prom-client';

// Create a Registry
const register = new promClient.Registry();

// Add default metrics (CPU, memory, event loop)
promClient.collectDefaultMetrics({ register });

// Custom metrics

// Counter - always goes up
const httpRequestsTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'path', 'status'],
  registers: [register]
});

// Histogram - distribution of values (for latency)
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'path', 'status'],
  buckets: [0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1, 5],
  registers: [register]
});

// Gauge - can go up and down
const activeConnections = new promClient.Gauge({
  name: 'http_active_connections',
  help: 'Number of active HTTP connections',
  registers: [register]
});

// Summary - similar to histogram but calculated on client
const httpRequestSummary = new promClient.Summary({
  name: 'http_request_duration_summary',
  help: 'HTTP request duration summary',
  labelNames: ['method', 'path'],
  percentiles: [0.5, 0.9, 0.99],
  registers: [register]
});

// Middleware to track metrics
function metricsMiddleware(req: Request, res: Response, next: NextFunction) {
  const startTime = Date.now();
  activeConnections.inc();

  // Normalize path to avoid high cardinality
  const path = normalizePath(req.path);

  res.on('finish', () => {
    const duration = (Date.now() - startTime) / 1000;
    const labels = {
      method: req.method,
      path,
      status: res.statusCode.toString()
    };

    httpRequestsTotal.inc(labels);
    httpRequestDuration.observe(labels, duration);
    httpRequestSummary.observe({ method: req.method, path }, duration);
    activeConnections.dec();
  });

  next();
}

// Normalize paths to avoid cardinality explosion
function normalizePath(path: string): string {
  return path
    .replace(/\/users\/[^\/]+/, '/users/:id')
    .replace(/\/orders\/[^\/]+/, '/orders/:id')
    .replace(/\/\d+/g, '/:id');
}

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

// Business metrics
const ordersCreated = new promClient.Counter({
  name: 'orders_created_total',
  help: 'Total orders created',
  labelNames: ['payment_method', 'status'],
  registers: [register]
});

const orderValue = new promClient.Histogram({
  name: 'order_value_dollars',
  help: 'Order value in dollars',
  buckets: [10, 50, 100, 500, 1000, 5000],
  registers: [register]
});

// Track business events
function onOrderCreated(order: Order) {
  ordersCreated.inc({
    payment_method: order.paymentMethod,
    status: 'created'
  });
  orderValue.observe(order.total);
}
```

---

## 4. Alert Design

```yaml
# ════════════════════════════════════════════════════════════════
# ALERTMANAGER CONFIGURATION
# alertmanager.yml
# ════════════════════════════════════════════════════════════════

global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/xxx'
  pagerduty_url: 'https://events.pagerduty.com/v2/enqueue'

route:
  receiver: 'slack-notifications'
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  
  routes:
    # Critical alerts to PagerDuty
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
      continue: true
    
    # All alerts to Slack
    - match_re:
        severity: warning|critical
      receiver: 'slack-notifications'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - channel: '#alerts'
        send_resolved: true
        title: '{{ .Status | toUpper }}: {{ .CommonLabels.alertname }}'
        text: |
          *Alert:* {{ .CommonLabels.alertname }}
          *Severity:* {{ .CommonLabels.severity }}
          *Service:* {{ .CommonLabels.service }}
          *Description:* {{ .CommonAnnotations.description }}
          *Runbook:* {{ .CommonAnnotations.runbook_url }}
        actions:
          - type: button
            text: 'Runbook'
            url: '{{ .CommonAnnotations.runbook_url }}'
          - type: button
            text: 'Dashboard'
            url: '{{ .CommonAnnotations.dashboard_url }}'

  - name: 'pagerduty-critical'
    pagerduty_configs:
      - service_key: '<pagerduty-service-key>'
        severity: critical
        description: '{{ .CommonLabels.alertname }}: {{ .CommonAnnotations.summary }}'

inhibit_rules:
  # Don't alert on warnings if critical is firing
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'cluster', 'service']

---
# ════════════════════════════════════════════════════════════════
# ALERT RULES
# rules/alerts.yml
# ════════════════════════════════════════════════════════════════

groups:
  - name: api_alerts
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
          /
          sum(rate(http_requests_total[5m])) by (service)
          > 0.01
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate on {{ $labels.service }}"
          description: "Error rate is {{ $value | humanizePercentage }} (threshold: 1%)"
          runbook_url: "https://wiki.example.com/runbooks/high-error-rate"
          dashboard_url: "https://grafana.example.com/d/api-dashboard"

      # High latency
      - alert: HighLatency
        expr: |
          histogram_quantile(0.99, 
            sum by (service, le) (rate(http_request_duration_seconds_bucket[5m]))
          ) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency on {{ $labels.service }}"
          description: "P99 latency is {{ $value | humanizeDuration }}"
          runbook_url: "https://wiki.example.com/runbooks/high-latency"

      # Service down
      - alert: ServiceDown
        expr: up{job="api"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.job }} is down"
          description: "{{ $labels.instance }} has been down for more than 1 minute"
          runbook_url: "https://wiki.example.com/runbooks/service-down"

      # High CPU
      - alert: HighCPU
        expr: |
          rate(process_cpu_seconds_total[5m]) * 100 > 80
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value | humanize }}%"

      # Memory pressure
      - alert: HighMemory
        expr: |
          process_resident_memory_bytes / 1024 / 1024 / 1024 > 3
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is {{ $value | humanize }} GB"

  - name: slo_alerts
    rules:
      # SLO burn rate alert (multi-window)
      - alert: SLOBudgetBurn
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[1h])) by (service)
            /
            sum(rate(http_requests_total[1h])) by (service)
          ) > (14.4 * 0.001)  # 14.4x budget burn rate
          and
          (
            sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
            /
            sum(rate(http_requests_total[5m])) by (service)
          ) > (14.4 * 0.001)
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "SLO budget burning too fast for {{ $labels.service }}"
          description: "At current error rate, will exhaust monthly error budget in < 2 hours"
```

---

## 5. Datadog

```typescript
// ════════════════════════════════════════════════════════════════
// DATADOG INTEGRATION - NODE.JS
// ════════════════════════════════════════════════════════════════

import tracer from 'dd-trace';
import { StatsD } from 'hot-shots';

// Initialize tracer
tracer.init({
  service: 'api-service',
  env: process.env.NODE_ENV,
  version: process.env.APP_VERSION,
  logInjection: true,
});

// Initialize StatsD client
const dogstatsd = new StatsD({
  host: process.env.DD_AGENT_HOST || 'localhost',
  port: 8125,
  prefix: 'api.',
  globalTags: {
    env: process.env.NODE_ENV,
    service: 'api-service',
  },
});

// Custom metrics
function trackRequest(method: string, path: string, status: number, duration: number) {
  const tags = [`method:${method}`, `path:${path}`, `status:${status}`];
  
  // Count
  dogstatsd.increment('requests.total', 1, tags);
  
  // Timing (histogram)
  dogstatsd.histogram('requests.duration', duration, tags);
  
  // Gauge
  dogstatsd.gauge('requests.active', activeRequestCount);
}

// Business metrics
function trackOrder(order: Order) {
  dogstatsd.increment('orders.created', 1, [
    `payment_method:${order.paymentMethod}`,
  ]);
  dogstatsd.histogram('orders.value', order.total);
}

// Express middleware
app.use((req, res, next) => {
  const startTime = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - startTime;
    trackRequest(req.method, normalizePath(req.path), res.statusCode, duration);
  });
  
  next();
});
```

```yaml
# Datadog Agent configuration
# datadog.yaml

api_key: <your-api-key>

logs_enabled: true
apm_config:
  enabled: true
  apm_non_local_traffic: true

process_config:
  enabled: true

dogstatsd_port: 8125

# Kubernetes integration
kubernetes:
  kubelet_tls_verify: false
  
# Autodiscovery
config_providers:
  - name: kubernetes
    polling: true
    
listeners:
  - name: kubernetes
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# MONITORING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: High cardinality labels
# Bad
http_requests_total{user_id="...", request_id="..."}
# Millions of unique combinations = performance issues

# Good
http_requests_total{method="GET", path="/api/users", status="200"}
# Bounded cardinality, meaningful aggregation

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Alerting on symptoms, not causes
# Bad
alert: HighCPU
expr: cpu > 80%
# CPU is symptom, not cause

# Good - Alert on user impact
alert: HighLatency
expr: http_request_duration_p99 > 500ms
# User-facing impact

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Alert fatigue
# Bad
alert: APIError
expr: http_errors_total > 0
for: 1m
# Any error triggers alert - too noisy!

# Good
alert: HighErrorRate
expr: error_rate > 0.01
for: 5m
# Only significant, sustained issues

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: No alert runbooks
# Bad
alert: DatabaseConnectionError
# What do I do?!

# Good
alert: DatabaseConnectionError
annotations:
  runbook_url: "https://wiki.example.com/runbooks/db-connection"
# Step-by-step investigation guide

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Missing critical alerts
# Bad
# Only monitoring CPU and memory
# Miss: error rates, latency, queue depth, dependencies

# Good - Four Golden Signals
- Latency (p50, p95, p99)
- Traffic (requests/sec)
- Errors (error rate)
- Saturation (CPU, memory, disk, connections)

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Averages instead of percentiles
# Bad
alert: HighLatency
expr: avg(http_request_duration) > 1s
# 99% of users might have 5s latency!

# Good
alert: HighLatency
expr: histogram_quantile(0.99, ...) > 500ms
# P99 captures tail latency
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What are the four golden signals?"**
> "Latency: response time. Traffic: request rate. Errors: error rate. Saturation: resource utilization. Monitor these four for any service, and you'll catch most issues."

**Q: "Prometheus vs Datadog?"**
> "Prometheus: Open source, pull-based, self-hosted, TSDB optimized for metrics, great for Kubernetes. Datadog: SaaS, push-based, unified platform (metrics, logs, traces, APM), easier to operate, costs based on hosts/metrics."

**Q: "What metrics should every service expose?"**
> "RED: Rate (requests/sec), Errors (error rate), Duration (latency percentiles). Plus resource metrics: CPU, memory, connections. Business metrics specific to service function."

### Intermediate Questions

**Q: "How do you avoid alert fatigue?"**
> "Alert on user impact, not symptoms. Use appropriate thresholds (not every error). Require duration (for: 5m, not instant). Group related alerts. Have on-call runbooks. Review and tune alerts regularly. Only page for actionable, urgent issues."

**Q: "What are recording rules?"**
> "Pre-computed queries stored as new time series. For expensive queries like percentiles. Run periodically (30s). Query recorded metric instead of raw computation. Essential for dashboards and alerts that would otherwise be slow."

**Q: "How do you design SLO-based alerts?"**
> "Define SLO (99.9% availability). Calculate error budget (0.1% errors/month). Alert on burn rate - if burning budget faster than sustainable (e.g., would exhaust in 2 hours). Multi-window: check both long (1h) and short (5m) windows to avoid flapping."

### Advanced Questions

**Q: "How do you handle metric cardinality?"**
> "Avoid unbounded labels (user_id, request_id). Normalize paths (/users/123 → /users/:id). Use recording rules to pre-aggregate. Set limits in Prometheus. Monitor cardinality itself. Drop unnecessary labels at scrape time."

**Q: "How do you monitor microservices?"**
> "Each service exposes /metrics. Prometheus scrapes with service discovery. Labels identify service, instance, environment. Distributed tracing (Jaeger) for request flows. Service mesh (Istio) for uniform telemetry. Centralized dashboards per service and system-wide."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  MONITORING CHECKLIST                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  FOUR GOLDEN SIGNALS:                                          │
│  □ Latency (p50, p95, p99)                                     │
│  □ Traffic (requests/second)                                   │
│  □ Errors (error rate percentage)                              │
│  □ Saturation (CPU, memory, disk)                              │
│                                                                 │
│  ALERTS:                                                        │
│  □ Alert on user impact                                        │
│  □ Appropriate thresholds                                      │
│  □ Duration requirement (5+ minutes)                           │
│  □ Runbook for each alert                                      │
│  □ Routing (Slack warnings, PagerDuty critical)                │
│                                                                 │
│  DASHBOARDS:                                                    │
│  □ Service health overview                                     │
│  □ Dependencies status                                         │
│  □ Business metrics                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

PROMQL CHEAT SHEET:
┌────────────────────────────────────────────────────────────────┐
│ rate(metric[5m])          - Per-second rate                    │
│ sum by (label) (metric)   - Aggregate by label                 │
│ histogram_quantile(0.99)  - Percentile from histogram          │
│ increase(metric[1h])      - Total increase over period         │
│ absent(metric)            - Alert if metric missing            │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

