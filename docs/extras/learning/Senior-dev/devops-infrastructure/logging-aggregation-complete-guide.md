# 📝 Logging & Aggregation - Complete Guide

> A comprehensive guide to logging and aggregation - ELK stack, structured logging, log rotation, centralized logging, and observability patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Logging aggregation collects, processes, and centralizes log data from distributed systems into a searchable, analyzable platform (like ELK or Loki), enabling debugging, monitoring, auditing, and incident response at scale through structured logging and correlation."

### The 7 Key Concepts (Remember These!)
```
1. STRUCTURED LOGGING → JSON format with consistent fields
2. LOG LEVELS         → DEBUG, INFO, WARN, ERROR, FATAL
3. CORRELATION ID     → Track requests across services
4. LOG AGGREGATION    → Centralize logs from all sources
5. LOG SHIPPING       → Forward logs to central system
6. LOG ROTATION       → Manage log file size and retention
7. LOG ANALYSIS       → Search, filter, visualize patterns
```

### Logging Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                   CENTRALIZED LOGGING ARCHITECTURE              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    LOG SOURCES                            │  │
│  │                                                          │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────┐ │  │
│  │  │Service A│  │Service B│  │Service C│  │ Load Balancer│ │  │
│  │  │ (stdout)│  │ (file)  │  │ (syslog)│  │    (logs)   │ │  │
│  │  └────┬────┘  └────┬────┘  └────┬────┘  └──────┬──────┘ │  │
│  └───────┼────────────┼────────────┼──────────────┼────────┘  │
│          │            │            │              │            │
│          ▼            ▼            ▼              ▼            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    LOG SHIPPERS                           │  │
│  │        Fluentd / Fluent Bit / Filebeat / Vector          │  │
│  └────────────────────────┬─────────────────────────────────┘  │
│                           │                                    │
│                           ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                  LOG PROCESSING                           │  │
│  │             Logstash / Fluentd / Vector                   │  │
│  │     (Parse, Transform, Enrich, Filter, Route)            │  │
│  └────────────────────────┬─────────────────────────────────┘  │
│                           │                                    │
│                           ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    LOG STORAGE                            │  │
│  │           Elasticsearch / Loki / CloudWatch              │  │
│  └────────────────────────┬─────────────────────────────────┘  │
│                           │                                    │
│                           ▼                                    │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                 VISUALIZATION & ANALYSIS                  │  │
│  │               Kibana / Grafana / OpenSearch              │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Log Level Guidelines
```
┌─────────────────────────────────────────────────────────────────┐
│                     LOG LEVEL GUIDELINES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  FATAL / CRITICAL                                              │
│  ─────────────────                                              │
│  Application cannot continue, immediate attention needed       │
│  Example: Database connection lost, out of memory              │
│  Alert: Page on-call immediately                               │
│                                                                 │
│  ERROR                                                          │
│  ─────                                                          │
│  Operation failed, but application continues                   │
│  Example: API call failed, invalid user input                  │
│  Alert: Create ticket, investigate soon                        │
│                                                                 │
│  WARN                                                           │
│  ────                                                           │
│  Unexpected but handled, potential issue                       │
│  Example: Retry succeeded, deprecated API used                 │
│  Alert: Review periodically                                    │
│                                                                 │
│  INFO                                                           │
│  ────                                                           │
│  Normal operations, significant events                         │
│  Example: Server started, user logged in, order placed         │
│  Alert: None (operational awareness)                           │
│                                                                 │
│  DEBUG                                                          │
│  ─────                                                          │
│  Detailed information for debugging                            │
│  Example: Function entry/exit, variable values                 │
│  Alert: None (development only)                                │
│                                                                 │
│  TRACE                                                          │
│  ─────                                                          │
│  Very detailed, step-by-step execution                         │
│  Example: Every database query, HTTP request details           │
│  Alert: None (extreme debugging only)                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Structured logging"** | "We use structured JSON logging for machine-parseable logs" |
| **"Correlation ID"** | "Correlation IDs let us trace requests across microservices" |
| **"Log shipping"** | "Fluent Bit ships logs to our centralized ELK stack" |
| **"Cardinality"** | "We avoid high cardinality fields to control index size" |
| **"Hot-warm-cold"** | "Our indices use hot-warm-cold tiering for cost efficiency" |
| **"Log sampling"** | "We sample debug logs at 10% to reduce storage costs" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Retention hot tier | **7 days** | Recent searchable logs |
| Retention warm | **30 days** | Reduced performance tier |
| Retention cold | **90 days** | Archive, compliance |
| Max log line | **10KB** | Prevent massive entries |
| Log volume budget | **1-5 GB/day/service** | Cost control |
| Index shard size | **20-50 GB** | Elasticsearch optimal |

### The "Wow" Statement (Memorize This!)
> "We run centralized logging for 100+ microservices using the EFK stack - Fluent Bit as lightweight shipper, Fluentd for processing, Elasticsearch for storage with Kibana dashboards. All services emit structured JSON logs with consistent fields: timestamp, level, service, correlationId, userId. Correlation IDs propagate through HTTP headers, letting us trace requests across service boundaries. We use hot-warm-cold tiering - 7 days hot on SSD for active debugging, 30 days warm, 90 days cold for compliance. Index lifecycle management handles rollover at 50GB or 1 day. We set up alerts for error rate spikes and pattern detection for anomalies. Debug logs are sampled at 10% to control costs, but we can enable full debugging via feature flag when investigating issues."

---

## 📚 Table of Contents

1. [Structured Logging](#1-structured-logging)
2. [ELK Stack](#2-elk-stack)
3. [Fluent Bit & Fluentd](#3-fluent-bit--fluentd)
4. [Loki](#4-loki)
5. [Log Rotation](#5-log-rotation)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Structured Logging

```typescript
// ════════════════════════════════════════════════════════════════
// STRUCTURED LOGGING - NODE.JS IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

import pino from 'pino';
import { randomUUID } from 'crypto';

// Logger configuration
const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label }),
    bindings: () => ({}), // Remove pid, hostname from each log
  },
  base: {
    service: process.env.SERVICE_NAME || 'api',
    version: process.env.APP_VERSION || '1.0.0',
    environment: process.env.NODE_ENV || 'development',
  },
  timestamp: () => `,"timestamp":"${new Date().toISOString()}"`,
  // Don't pretty print in production
  ...(process.env.NODE_ENV === 'development' && {
    transport: {
      target: 'pino-pretty',
      options: { colorize: true }
    }
  })
});

// Request context middleware (Express)
import { AsyncLocalStorage } from 'async_hooks';

interface RequestContext {
  correlationId: string;
  userId?: string;
  requestId: string;
}

const asyncLocalStorage = new AsyncLocalStorage<RequestContext>();

function requestContextMiddleware(req: Request, res: Response, next: NextFunction) {
  const context: RequestContext = {
    correlationId: req.headers['x-correlation-id'] as string || randomUUID(),
    userId: req.user?.id,
    requestId: randomUUID(),
  };
  
  // Propagate correlation ID to downstream services
  res.setHeader('x-correlation-id', context.correlationId);
  
  asyncLocalStorage.run(context, () => next());
}

// Context-aware logger
function getLogger() {
  const context = asyncLocalStorage.getStore();
  return logger.child({
    correlationId: context?.correlationId,
    userId: context?.userId,
    requestId: context?.requestId,
  });
}

// Usage examples
class OrderService {
  async createOrder(orderData: OrderInput) {
    const log = getLogger();
    
    log.info({ orderData: { ...orderData, creditCard: '[REDACTED]' } }, 
      'Creating order');
    
    try {
      const order = await this.repository.create(orderData);
      
      log.info({ 
        orderId: order.id, 
        total: order.total,
        itemCount: order.items.length 
      }, 'Order created successfully');
      
      return order;
    } catch (error) {
      log.error({ 
        error: {
          message: error.message,
          stack: error.stack,
          code: error.code
        },
        orderData: { userId: orderData.userId }
      }, 'Failed to create order');
      
      throw error;
    }
  }
}

// Log output (JSON)
/*
{
  "level": "info",
  "timestamp": "2024-01-15T10:30:00.000Z",
  "service": "order-service",
  "version": "1.2.3",
  "environment": "production",
  "correlationId": "abc-123-def-456",
  "userId": "user_789",
  "requestId": "req_xyz",
  "orderId": "order_123",
  "total": 99.99,
  "itemCount": 3,
  "msg": "Order created successfully"
}
*/
```

```typescript
// ════════════════════════════════════════════════════════════════
// LOGGING BEST PRACTICES
// ════════════════════════════════════════════════════════════════

// ✅ DO: Use structured logging with context
log.info({ userId, action: 'login', ip: req.ip }, 'User logged in');

// ❌ DON'T: String concatenation
log.info(`User ${userId} logged in from ${req.ip}`);

// ✅ DO: Separate data from message
log.error({ error: err, userId }, 'Payment failed');

// ❌ DON'T: Include data in message
log.error(`Payment failed for user ${userId}: ${err.message}`);

// ✅ DO: Redact sensitive data
log.info({ 
  user: { id: user.id, email: user.email },
  payment: { last4: card.last4 }  // Not full card number
}, 'Payment processed');

// ❌ DON'T: Log sensitive data
log.info({ user, creditCard }, 'Payment processed');

// ✅ DO: Use appropriate log levels
log.debug({ query }, 'Database query');     // Development only
log.info({ orderId }, 'Order created');      // Normal operations
log.warn({ retryCount }, 'Retry attempt');   // Potential issues
log.error({ error }, 'Operation failed');    // Errors

// ✅ DO: Include enough context for debugging
log.error({
  error: {
    message: err.message,
    stack: err.stack,
    code: err.code
  },
  request: {
    method: req.method,
    path: req.path,
    params: req.params
  },
  user: { id: req.user?.id }
}, 'API request failed');
```

---

## 2. ELK Stack

```yaml
# ════════════════════════════════════════════════════════════════
# ELASTICSEARCH CONFIGURATION
# elasticsearch.yml
# ════════════════════════════════════════════════════════════════

cluster.name: production-logs
node.name: es-node-1

# Network
network.host: 0.0.0.0
http.port: 9200

# Discovery
discovery.seed_hosts:
  - es-node-1
  - es-node-2
  - es-node-3
cluster.initial_master_nodes:
  - es-node-1
  - es-node-2
  - es-node-3

# Paths
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch

# Memory
bootstrap.memory_lock: true

# Security
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true

---
# ════════════════════════════════════════════════════════════════
# INDEX LIFECYCLE MANAGEMENT (ILM)
# ════════════════════════════════════════════════════════════════

PUT _ilm/policy/logs-policy
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_age": "1d",
            "max_size": "50gb",
            "max_docs": 100000000
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "set_priority": {
            "priority": 50
          },
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "set_priority": {
            "priority": 0
          },
          "freeze": {}
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}

---
# ════════════════════════════════════════════════════════════════
# INDEX TEMPLATE
# ════════════════════════════════════════════════════════════════

PUT _index_template/logs-template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1,
      "index.lifecycle.name": "logs-policy",
      "index.lifecycle.rollover_alias": "logs"
    },
    "mappings": {
      "properties": {
        "timestamp": { "type": "date" },
        "level": { "type": "keyword" },
        "service": { "type": "keyword" },
        "version": { "type": "keyword" },
        "environment": { "type": "keyword" },
        "correlationId": { "type": "keyword" },
        "userId": { "type": "keyword" },
        "message": { "type": "text" },
        "error": {
          "properties": {
            "message": { "type": "text" },
            "stack": { "type": "text" },
            "code": { "type": "keyword" }
          }
        },
        "http": {
          "properties": {
            "method": { "type": "keyword" },
            "path": { "type": "keyword" },
            "status": { "type": "integer" },
            "duration": { "type": "integer" }
          }
        }
      }
    }
  }
}

---
# ════════════════════════════════════════════════════════════════
# LOGSTASH CONFIGURATION
# logstash.conf
# ════════════════════════════════════════════════════════════════

input {
  beats {
    port => 5044
  }
  
  kafka {
    bootstrap_servers => "kafka:9092"
    topics => ["logs"]
    codec => json
  }
}

filter {
  # Parse JSON logs
  json {
    source => "message"
    target => "parsed"
  }
  
  # Extract timestamp
  date {
    match => ["[parsed][timestamp]", "ISO8601"]
    target => "@timestamp"
  }
  
  # Add fields from parsed JSON
  mutate {
    rename => {
      "[parsed][level]" => "level"
      "[parsed][service]" => "service"
      "[parsed][correlationId]" => "correlationId"
      "[parsed][msg]" => "message"
    }
    remove_field => ["parsed", "host", "agent"]
  }
  
  # GeoIP for client IPs
  if [clientIp] {
    geoip {
      source => "clientIp"
      target => "geo"
    }
  }
  
  # Fingerprint for deduplication
  fingerprint {
    source => ["correlationId", "timestamp", "message"]
    target => "[@metadata][fingerprint]"
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{[service]}-%{+YYYY.MM.dd}"
    document_id => "%{[@metadata][fingerprint]}"
    user => "elastic"
    password => "${ELASTIC_PASSWORD}"
  }
}
```

---

## 3. Fluent Bit & Fluentd

```yaml
# ════════════════════════════════════════════════════════════════
# FLUENT BIT CONFIGURATION (Lightweight log shipper)
# fluent-bit.conf
# ════════════════════════════════════════════════════════════════

[SERVICE]
    Flush         5
    Daemon        Off
    Log_Level     info
    Parsers_File  parsers.conf
    HTTP_Server   On
    HTTP_Listen   0.0.0.0
    HTTP_Port     2020

# Kubernetes container logs
[INPUT]
    Name              tail
    Tag               kube.*
    Path              /var/log/containers/*.log
    Parser            docker
    DB                /var/log/flb_kube.db
    Mem_Buf_Limit     5MB
    Skip_Long_Lines   On
    Refresh_Interval  10

# Kubernetes metadata enrichment
[FILTER]
    Name                kubernetes
    Match               kube.*
    Kube_URL            https://kubernetes.default.svc:443
    Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token
    Kube_Tag_Prefix     kube.var.log.containers.
    Merge_Log           On
    Keep_Log            Off
    K8S-Logging.Parser  On
    K8S-Logging.Exclude On

# Parse JSON logs
[FILTER]
    Name          parser
    Match         kube.*
    Key_Name      log
    Parser        json
    Reserve_Data  On

# Add custom fields
[FILTER]
    Name          modify
    Match         *
    Add           cluster production
    Add           region us-east-1

# Output to Elasticsearch
[OUTPUT]
    Name            es
    Match           kube.*
    Host            elasticsearch.logging.svc.cluster.local
    Port            9200
    Logstash_Format On
    Logstash_Prefix logs
    Retry_Limit     False
    tls             On
    tls.verify      Off
    HTTP_User       ${ELASTIC_USER}
    HTTP_Passwd     ${ELASTIC_PASSWORD}

# Output to Loki (alternative)
[OUTPUT]
    Name          loki
    Match         kube.*
    Host          loki.logging.svc.cluster.local
    Port          3100
    Labels        job=fluentbit, cluster=production
    Auto_Kubernetes_Labels on

---
# ════════════════════════════════════════════════════════════════
# FLUENT BIT PARSERS
# parsers.conf
# ════════════════════════════════════════════════════════════════

[PARSER]
    Name        docker
    Format      json
    Time_Key    time
    Time_Format %Y-%m-%dT%H:%M:%S.%L
    Time_Keep   On

[PARSER]
    Name        json
    Format      json
    Time_Key    timestamp
    Time_Format %Y-%m-%dT%H:%M:%S.%LZ

[PARSER]
    Name        nginx
    Format      regex
    Regex       ^(?<remote>[^ ]*) - (?<user>[^ ]*) \[(?<time>[^\]]*)\] "(?<method>\S+)(?: +(?<path>[^\"]*?)(?: +\S*)?)?" (?<code>[^ ]*) (?<size>[^ ]*)(?: "(?<referer>[^\"]*)" "(?<agent>[^\"]*)")?$
    Time_Key    time
    Time_Format %d/%b/%Y:%H:%M:%S %z
```

```yaml
# ════════════════════════════════════════════════════════════════
# KUBERNETES DAEMONSET FOR FLUENT BIT
# ════════════════════════════════════════════════════════════════

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: logging
spec:
  selector:
    matchLabels:
      app: fluent-bit
  template:
    metadata:
      labels:
        app: fluent-bit
    spec:
      serviceAccountName: fluent-bit
      containers:
        - name: fluent-bit
          image: fluent/fluent-bit:2.1
          resources:
            limits:
              memory: 200Mi
              cpu: 200m
            requests:
              memory: 100Mi
              cpu: 100m
          volumeMounts:
            - name: varlog
              mountPath: /var/log
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
              readOnly: true
            - name: config
              mountPath: /fluent-bit/etc/
          env:
            - name: ELASTIC_USER
              valueFrom:
                secretKeyRef:
                  name: elastic-credentials
                  key: username
            - name: ELASTIC_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: elastic-credentials
                  key: password
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
        - name: varlibdockercontainers
          hostPath:
            path: /var/lib/docker/containers
        - name: config
          configMap:
            name: fluent-bit-config
      tolerations:
        - key: node-role.kubernetes.io/master
          operator: Exists
          effect: NoSchedule
```

---

## 4. Loki

```yaml
# ════════════════════════════════════════════════════════════════
# GRAFANA LOKI CONFIGURATION
# loki-config.yaml
# ════════════════════════════════════════════════════════════════

auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096

common:
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1
  ring:
    instance_addr: 127.0.0.1
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb_shipper:
    active_index_directory: /loki/boltdb-shipper-active
    cache_location: /loki/boltdb-shipper-cache
    cache_ttl: 24h
    shared_store: filesystem
  filesystem:
    directory: /loki/chunks

# For S3 storage (production)
# storage_config:
#   aws:
#     s3: s3://us-east-1/my-loki-bucket
#     bucketnames: my-loki-bucket
#     region: us-east-1

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h  # 7 days
  max_entries_limit_per_query: 5000

chunk_store_config:
  max_look_back_period: 0s

table_manager:
  retention_deletes_enabled: true
  retention_period: 720h  # 30 days

ruler:
  alertmanager_url: http://alertmanager:9093

---
# ════════════════════════════════════════════════════════════════
# LOGQL QUERIES (Loki Query Language)
# ════════════════════════════════════════════════════════════════

# Basic label queries
{app="myapp"}
{app="myapp", environment="production"}
{app=~"myapp|otherapp"}  # Regex match

# Line filters
{app="myapp"} |= "error"           # Contains
{app="myapp"} != "debug"           # Not contains
{app="myapp"} |~ "error|warning"   # Regex match

# JSON parsing
{app="myapp"} | json | level="error"
{app="myapp"} | json | status >= 500
{app="myapp"} | json | line_format "{{.level}} - {{.message}}"

# Metrics from logs
count_over_time({app="myapp"} |= "error" [5m])
rate({app="myapp"} |= "error" [5m])
sum by (status) (count_over_time({app="myapp"} | json [5m]))

# Aggregations
sum(rate({app="myapp"}[5m])) by (level)
topk(10, sum(rate({app="myapp"} |= "error" [5m])) by (path))
```

---

## 5. Log Rotation

```bash
# ════════════════════════════════════════════════════════════════
# LOGROTATE CONFIGURATION
# /etc/logrotate.d/myapp
# ════════════════════════════════════════════════════════════════

/var/log/myapp/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 myapp myapp
    sharedscripts
    postrotate
        # Signal app to reopen log files
        systemctl reload myapp || true
    endscript
}

# ════════════════════════════════════════════════════════════════
# DOCKER LOGGING CONFIGURATION
# daemon.json
# ════════════════════════════════════════════════════════════════
```

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "5",
    "compress": "true"
  }
}
```

```yaml
# Docker Compose log configuration
services:
  app:
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "5"
        compress: "true"
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# LOGGING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Logging sensitive data
# Bad
log.info({ user }, 'User created');  # May include password

# Good
log.info({ 
  userId: user.id, 
  email: user.email 
}, 'User created');

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Unstructured logging
# Bad
log.info(`User ${userId} logged in from ${ip}`);
# Can't search by userId or ip

# Good
log.info({ userId, ip }, 'User logged in');
# Searchable: level:info AND userId:123

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: No correlation ID
# Bad
# Service A: "Processing order"
# Service B: "Charging payment"
# Can't link these together!

# Good
# Service A: {"correlationId": "abc", "msg": "Processing order"}
# Service B: {"correlationId": "abc", "msg": "Charging payment"}
# Search: correlationId:abc shows full flow

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: High cardinality labels
# Bad (Loki/Prometheus)
{userId="unique-for-every-user"}  # Millions of label values!

# Good
{app="myapp", environment="prod"}  # Low cardinality labels
# userId as log field, not label

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: No log rotation
# Bad
# Log file grows unbounded, fills disk
log.info('...')  # Writes to ever-growing file

# Good
# Configure logrotate or use logging driver
logging:
  driver: json-file
  options:
    max-size: 100m
    max-file: 5

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Debug logging in production
# Bad
LOG_LEVEL=debug  # 100x log volume, performance impact

# Good
LOG_LEVEL=info  # Normal operations
# Enable debug via feature flag when needed
```

---

## 7. Interview Questions

### Basic Questions

**Q: "Why is structured logging important?"**
> "Structured logging (JSON) is machine-parseable. Enables searching by specific fields (userId, correlationId), aggregation, alerting on patterns. Unstructured logs require regex parsing which is fragile and slow. Structured logs are the foundation of observability."

**Q: "What log levels do you use and when?"**
> "DEBUG: Development details. INFO: Normal operations (user login, order created). WARN: Handled but unexpected (retry succeeded). ERROR: Operation failed but app continues. FATAL: App cannot continue. Production typically runs at INFO, DEBUG enabled for investigation."

**Q: "What is a correlation ID?"**
> "Unique identifier that follows a request across services. Generated at entry point, propagated via HTTP headers. Enables tracing a single user request through all microservices. Essential for debugging distributed systems."

### Intermediate Questions

**Q: "ELK vs Loki?"**
> "ELK: Full-text indexing, powerful querying, higher resource usage. Good for complex log analysis. Loki: Label-based indexing, stores raw logs, much lower resource usage. Good for Kubernetes-native, Grafana integration. Loki for cost-effective, ELK for complex search requirements."

**Q: "How do you handle log volume and costs?"**
> "Log sampling for verbose levels (10% debug). Index lifecycle management (hot/warm/cold tiers). Retention policies (7/30/90 days). Avoid high cardinality fields. Set max log line size. Alert on volume spikes. Consider cost per GB when choosing platform."

**Q: "How do you ship logs in Kubernetes?"**
> "DaemonSet running Fluent Bit on each node. Reads from /var/log/containers. Enriches with Kubernetes metadata (pod, namespace, labels). Ships to Elasticsearch/Loki. Alternatively, sidecar pattern for specific requirements."

### Advanced Questions

**Q: "How do you design logging for compliance (GDPR, PCI)?"**
> "Never log PII (passwords, full credit cards, SSN). Mask or tokenize sensitive data. Retention policies aligned with compliance (delete after X days). Access controls on log systems. Audit logs for who accessed what. Consider encryption at rest."

**Q: "How do you debug production issues using logs?"**
> "1) Find error logs near incident time. 2) Get correlation ID from error. 3) Search all services for that correlation ID. 4) Reconstruct request flow. 5) Identify where failure occurred. 6) Check related metrics and traces. 7) Correlate with deployment events."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                    LOGGING CHECKLIST                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LOG FORMAT:                                                    │
│  □ JSON structured logs                                        │
│  □ Consistent timestamp format (ISO8601)                       │
│  □ Include: level, service, correlationId, message             │
│  □ No sensitive data (passwords, tokens, PII)                  │
│                                                                 │
│  CORRELATION:                                                   │
│  □ Generate correlation ID at entry point                      │
│  □ Propagate via HTTP headers                                  │
│  □ Include in all log entries                                  │
│                                                                 │
│  INFRASTRUCTURE:                                                │
│  □ Centralized log aggregation                                 │
│  □ Log rotation configured                                     │
│  □ Retention policies set                                      │
│  □ Alerts for error spikes                                     │
│                                                                 │
│  OPERATIONS:                                                    │
│  □ Production: INFO level default                              │
│  □ Debug can be enabled when needed                            │
│  □ Log volume monitored                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

LOG LEVELS:
┌────────────────────────────────────────────────────────────────┐
│ FATAL - App cannot continue, page on-call                      │
│ ERROR - Operation failed, create ticket                        │
│ WARN  - Unexpected but handled                                 │
│ INFO  - Normal operations (default production)                 │
│ DEBUG - Detailed for debugging                                 │
│ TRACE - Very detailed, rarely used                             │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

