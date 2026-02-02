# 🐤 Canary Releases - Complete Guide

> A comprehensive guide to canary releases - gradual rollouts, traffic shifting, metrics-based promotion, and risk mitigation strategies.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Canary releases gradually shift production traffic from the current version to a new version (e.g., 1% → 10% → 50% → 100%), allowing real-world validation with a small user subset before full rollout, enabling early detection of issues with minimal blast radius."

### The 7 Key Concepts (Remember These!)
```
1. CANARY            → Small subset running new version
2. BASELINE          → Current production version
3. TRAFFIC WEIGHT    → Percentage routed to canary
4. PROMOTION         → Increase canary traffic (all good)
5. ROLLBACK          → Route 100% back to baseline (issues)
6. ANALYSIS          → Compare canary vs baseline metrics
7. BLAST RADIUS      → Impact scope if canary fails
```

### Canary Release Flow
```
┌─────────────────────────────────────────────────────────────────┐
│                   CANARY RELEASE FLOW                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PHASE 1: Initial Canary (1-5% traffic)                        │
│  ──────────────────────────────────────                         │
│                                                                 │
│      Users → [Load Balancer]                                   │
│                    │                                           │
│             ┌──────┴──────┐                                    │
│             │             │                                    │
│             ▼ 95%         ▼ 5%                                 │
│      ┌──────────┐   ┌──────────┐                              │
│      │ Baseline │   │  Canary  │                              │
│      │   v1.0   │   │   v2.0   │                              │
│      └──────────┘   └──────────┘                              │
│                                                                 │
│  PHASE 2: Gradual Increase                                     │
│  ─────────────────────────                                      │
│                                                                 │
│  1% → 5% → 10% → 25% → 50% → 100%                             │
│       ↑         ↑          ↑                                   │
│       └─────────┴──────────┴── Analysis at each stage         │
│                                                                 │
│  PHASE 3: Full Promotion (if successful)                       │
│  ───────────────────────────────────────                        │
│                                                                 │
│      Users → [Load Balancer] → [v2.0] ← 100% traffic          │
│                                                                 │
│              (v1.0 decommissioned)                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Canary vs Blue-Green vs Rolling
```
┌─────────────────────────────────────────────────────────────────┐
│                 DEPLOYMENT STRATEGY COMPARISON                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                    CANARY                                       │
│  Traffic:    1% ──▶ 10% ──▶ 50% ──▶ 100%                      │
│  Duration:   Hours to days                                     │
│  Risk:       Very low (small blast radius)                     │
│  Resources:  1.x capacity during rollout                       │
│  Rollback:   Fast (shift traffic back)                         │
│  Best for:   High-risk changes, large user bases               │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│                  BLUE-GREEN                                     │
│  Traffic:    0% ──▶ 100% (instant switch)                      │
│  Duration:   Minutes                                           │
│  Risk:       Medium (all-or-nothing)                           │
│  Resources:  2x capacity during switch                         │
│  Rollback:   Instant (switch back)                             │
│  Best for:   Quick deployments, confidence in testing          │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│                   ROLLING                                       │
│  Traffic:    Mixed during rollout                              │
│  Duration:   Minutes to hours                                  │
│  Risk:       Medium (gradual but not controlled)               │
│  Resources:  ~1x capacity                                      │
│  Rollback:   Slow (must roll back each instance)               │
│  Best for:   Routine updates, resource-constrained             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Blast radius"** | "Canary limits blast radius to 5% of users initially" |
| **"Progressive delivery"** | "We use progressive delivery with automated canary analysis" |
| **"Metrics-driven"** | "Promotion is metrics-driven, not time-based" |
| **"Statistical significance"** | "We wait for statistical significance before promotion" |
| **"Feature flags"** | "Feature flags complement canary for fine-grained control" |
| **"Traffic mirroring"** | "Shadow traffic for canary without user impact" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Initial canary | **1-5%** | Minimal blast radius |
| Analysis window | **15-30 min** | Per stage |
| Error rate threshold | **< 1%** | Promotion criteria |
| Latency threshold | **< 10% increase** | Acceptable degradation |
| Full rollout | **Hours to days** | Depends on confidence |
| Rollback time | **< 1 minute** | Traffic shift |

### The "Wow" Statement (Memorize This!)
> "We use Argo Rollouts for canary deployments with automated progressive delivery. Initial canary gets 5% traffic for 30 minutes while we compare error rates, latency P95, and business metrics against baseline using Prometheus. If canary metrics are within acceptable bounds (error rate <1%, latency <10% increase), we auto-promote to 25%, then 50%, then 100% - each stage with its own analysis window. If any metric breaches thresholds, automatic rollback triggers within seconds, notifies the team via Slack, and creates an incident. We also use header-based routing for internal testing before any user traffic. This gives us confidence to deploy to production multiple times daily with minimal risk."

---

## 📚 Table of Contents

1. [Implementation Patterns](#1-implementation-patterns)
2. [AWS Implementation](#2-aws-implementation)
3. [Kubernetes with Argo Rollouts](#3-kubernetes-with-argo-rollouts)
4. [Istio Traffic Management](#4-istio-traffic-management)
5. [Canary Analysis](#5-canary-analysis)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Implementation Patterns

```yaml
# ════════════════════════════════════════════════════════════════
# CANARY RELEASE PATTERNS
# ════════════════════════════════════════════════════════════════

# PATTERN 1: Percentage-Based
# ───────────────────────────
# Most common - route X% of traffic to canary

stages:
  - weight: 5%
    duration: 30m
    analysis: true
  - weight: 25%
    duration: 30m
    analysis: true
  - weight: 50%
    duration: 1h
    analysis: true
  - weight: 100%
    duration: -  # Final

# ════════════════════════════════════════════════════════════════

# PATTERN 2: User-Based Canary
# ────────────────────────────
# Specific users get canary (beta testers, internal, region)

canary_users:
  - internal_employees (header: X-Employee=true)
  - beta_testers (cookie: beta=true)
  - region: us-west-2
  - user_id % 100 < 5  # 5% of users by ID

# ════════════════════════════════════════════════════════════════

# PATTERN 3: Header-Based (Dark Launch)
# ─────────────────────────────────────
# Test canary with specific header before any real traffic

routing:
  - header: X-Canary=true → Canary
  - default → Baseline

# QA/developers test with header first
# Then enable percentage-based rollout

# ════════════════════════════════════════════════════════════════

# PATTERN 4: Shadow Traffic (Mirroring)
# ─────────────────────────────────────
# Copy traffic to canary without serving responses

traffic:
  primary: Baseline (serves responses)
  mirror: Canary (receives copy, responses discarded)

# Test canary under real load without user impact
# Useful for catching performance issues
```

---

## 2. AWS Implementation

```hcl
# ════════════════════════════════════════════════════════════════
# AWS CANARY WITH ALB WEIGHTED TARGET GROUPS
# ════════════════════════════════════════════════════════════════

resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.main.arn
  port              = 443
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS13-1-2-2021-06"
  certificate_arn   = var.certificate_arn

  default_action {
    type = "forward"
    forward {
      # Baseline (current production)
      target_group {
        arn    = aws_lb_target_group.baseline.arn
        weight = 95
      }
      # Canary (new version)
      target_group {
        arn    = aws_lb_target_group.canary.arn
        weight = 5
      }
    }
    # Stickiness ensures same user goes to same version
    stickiness {
      enabled  = true
      duration = 3600
    }
  }
}

# Baseline Target Group
resource "aws_lb_target_group" "baseline" {
  name        = "app-baseline-tg"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = var.vpc_id
  target_type = "instance"

  health_check {
    path                = "/health"
    healthy_threshold   = 2
    unhealthy_threshold = 3
    interval            = 10
  }
}

# Canary Target Group
resource "aws_lb_target_group" "canary" {
  name        = "app-canary-tg"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = var.vpc_id
  target_type = "instance"

  health_check {
    path                = "/health"
    healthy_threshold   = 2
    unhealthy_threshold = 3
    interval            = 10
  }
}
```

```bash
#!/bin/bash
# ════════════════════════════════════════════════════════════════
# AWS CANARY DEPLOYMENT SCRIPT
# ════════════════════════════════════════════════════════════════

set -e

LISTENER_ARN="arn:aws:elasticloadbalancing:..."
BASELINE_TG="arn:aws:elasticloadbalancing:...:targetgroup/app-baseline-tg/..."
CANARY_TG="arn:aws:elasticloadbalancing:...:targetgroup/app-canary-tg/..."

# Canary stages: weight, duration
STAGES=(
  "5:30"    # 5% for 30 minutes
  "25:30"   # 25% for 30 minutes
  "50:60"   # 50% for 60 minutes
  "100:0"   # 100% (final)
)

update_weights() {
  local canary_weight=$1
  local baseline_weight=$((100 - canary_weight))
  
  echo "Setting weights: baseline=$baseline_weight%, canary=$canary_weight%"
  
  aws elbv2 modify-listener \
    --listener-arn $LISTENER_ARN \
    --default-actions "[
      {
        \"Type\": \"forward\",
        \"ForwardConfig\": {
          \"TargetGroups\": [
            {\"TargetGroupArn\": \"$BASELINE_TG\", \"Weight\": $baseline_weight},
            {\"TargetGroupArn\": \"$CANARY_TG\", \"Weight\": $canary_weight}
          ],
          \"TargetGroupStickinessConfig\": {
            \"Enabled\": true,
            \"DurationSeconds\": 3600
          }
        }
      }
    ]"
}

analyze_canary() {
  local duration=$1
  echo "Analyzing canary for ${duration} minutes..."
  
  # Query CloudWatch for canary metrics
  local end_time=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
  local start_time=$(date -u -d "-${duration} minutes" +"%Y-%m-%dT%H:%M:%SZ")
  
  # Get error rate
  local error_rate=$(aws cloudwatch get-metric-statistics \
    --namespace "AWS/ApplicationELB" \
    --metric-name "HTTPCode_Target_5XX_Count" \
    --dimensions Name=TargetGroup,Value=$(basename $CANARY_TG) \
    --start-time $start_time \
    --end-time $end_time \
    --period 300 \
    --statistics Sum \
    --query 'Datapoints[0].Sum' \
    --output text)
  
  # Check threshold
  if [ "$error_rate" != "None" ] && [ $(echo "$error_rate > 100" | bc) -eq 1 ]; then
    echo "❌ Error rate too high: $error_rate"
    return 1
  fi
  
  echo "✅ Canary analysis passed"
  return 0
}

rollback() {
  echo "🚨 Rolling back to baseline!"
  update_weights 0
  exit 1
}

# Main deployment loop
for stage in "${STAGES[@]}"; do
  IFS=':' read -r weight duration <<< "$stage"
  
  echo "📈 Stage: ${weight}% canary weight"
  update_weights $weight
  
  if [ "$duration" -gt 0 ]; then
    sleep "${duration}m"
    
    if ! analyze_canary $duration; then
      rollback
    fi
  fi
done

echo "✅ Canary deployment complete - 100% traffic on new version"
```

---

## 3. Kubernetes with Argo Rollouts

```yaml
# ════════════════════════════════════════════════════════════════
# ARGO ROLLOUTS - CANARY DEPLOYMENT
# ════════════════════════════════════════════════════════════════

apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 10
  
  selector:
    matchLabels:
      app: myapp
  
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:2.0.0
          ports:
            - containerPort: 3000
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
  
  strategy:
    canary:
      # Canary service for direct access
      canaryService: myapp-canary
      stableService: myapp-stable
      
      # Traffic routing (Istio, Nginx, ALB, etc.)
      trafficRouting:
        istio:
          virtualService:
            name: myapp-vsvc
            routes:
              - primary
      
      # Canary steps
      steps:
        # Step 1: 5% traffic
        - setWeight: 5
        - pause: { duration: 30m }
        
        # Step 2: Analysis
        - analysis:
            templates:
              - templateName: canary-analysis
            args:
              - name: service-name
                value: myapp-canary
        
        # Step 3: 25% traffic
        - setWeight: 25
        - pause: { duration: 30m }
        
        # Step 4: 50% traffic
        - setWeight: 50
        - pause: { duration: 1h }
        
        # Step 5: Full rollout
        - setWeight: 100
      
      # Anti-affinity for canary pods
      antiAffinity:
        requiredDuringSchedulingIgnoredDuringExecution: {}
      
      # Traffic management
      maxSurge: "25%"
      maxUnavailable: 0

---
# ════════════════════════════════════════════════════════════════
# ANALYSIS TEMPLATE
# ════════════════════════════════════════════════════════════════

apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: canary-analysis
spec:
  args:
    - name: service-name
  
  metrics:
    # Success rate metric
    - name: success-rate
      interval: 5m
      count: 3
      successCondition: result[0] >= 0.99
      failureLimit: 1
      provider:
        prometheus:
          address: http://prometheus.monitoring:9090
          query: |
            sum(rate(http_requests_total{service="{{args.service-name}}", status=~"2.."}[5m]))
            /
            sum(rate(http_requests_total{service="{{args.service-name}}"}[5m]))
    
    # Latency P99 metric
    - name: latency-p99
      interval: 5m
      count: 3
      successCondition: result[0] <= 500
      failureLimit: 1
      provider:
        prometheus:
          address: http://prometheus.monitoring:9090
          query: |
            histogram_quantile(0.99, 
              sum(rate(http_request_duration_seconds_bucket{service="{{args.service-name}}"}[5m])) 
              by (le)
            ) * 1000
    
    # Error rate comparison with baseline
    - name: error-rate-comparison
      interval: 5m
      count: 3
      successCondition: result[0] <= 1.1  # Max 10% increase vs baseline
      failureLimit: 1
      provider:
        prometheus:
          address: http://prometheus.monitoring:9090
          query: |
            (
              sum(rate(http_requests_total{service="{{args.service-name}}", status=~"5.."}[5m]))
              /
              sum(rate(http_requests_total{service="{{args.service-name}}"}[5m]))
            )
            /
            (
              sum(rate(http_requests_total{service="myapp-stable", status=~"5.."}[5m]))
              /
              sum(rate(http_requests_total{service="myapp-stable"}[5m]))
            )

---
# Services
apiVersion: v1
kind: Service
metadata:
  name: myapp-stable
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: myapp-canary
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 3000
```

```bash
# Argo Rollouts commands
kubectl argo rollouts get rollout myapp -w        # Watch rollout
kubectl argo rollouts promote myapp               # Promote to next step
kubectl argo rollouts abort myapp                 # Abort and rollback
kubectl argo rollouts retry myapp                 # Retry failed rollout
kubectl argo rollouts set image myapp myapp=myapp:3.0.0  # Update image
```

---

## 4. Istio Traffic Management

```yaml
# ════════════════════════════════════════════════════════════════
# ISTIO VIRTUAL SERVICE FOR CANARY
# ════════════════════════════════════════════════════════════════

apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp-vsvc
spec:
  hosts:
    - myapp
  http:
    # Header-based routing for testing
    - match:
        - headers:
            x-canary:
              exact: "true"
      route:
        - destination:
            host: myapp-canary
            port:
              number: 80
    
    # Weighted routing for canary
    - route:
        - destination:
            host: myapp-stable
            port:
              number: 80
          weight: 95
        - destination:
            host: myapp-canary
            port:
              number: 80
          weight: 5

---
# ════════════════════════════════════════════════════════════════
# DESTINATION RULES
# ════════════════════════════════════════════════════════════════

apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp-dr
spec:
  host: myapp
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 100
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
  subsets:
    - name: stable
      labels:
        version: stable
    - name: canary
      labels:
        version: canary

---
# ════════════════════════════════════════════════════════════════
# TRAFFIC MIRRORING (Shadow Traffic)
# ════════════════════════════════════════════════════════════════

apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp-mirror
spec:
  hosts:
    - myapp
  http:
    - route:
        - destination:
            host: myapp-stable
      mirror:
        host: myapp-canary
      mirrorPercentage:
        value: 100.0  # Mirror 100% of traffic
```

---

## 5. Canary Analysis

```typescript
// ════════════════════════════════════════════════════════════════
// CANARY ANALYSIS SERVICE
// ════════════════════════════════════════════════════════════════

interface CanaryMetrics {
  errorRate: number;
  latencyP50: number;
  latencyP95: number;
  latencyP99: number;
  requestsPerSecond: number;
}

interface AnalysisResult {
  passed: boolean;
  metrics: {
    canary: CanaryMetrics;
    baseline: CanaryMetrics;
  };
  comparison: {
    errorRateRatio: number;
    latencyRatio: number;
  };
  reasons: string[];
}

class CanaryAnalyzer {
  private prometheusUrl: string;
  
  constructor(prometheusUrl: string) {
    this.prometheusUrl = prometheusUrl;
  }

  async analyze(
    canaryService: string,
    baselineService: string,
    durationMinutes: number
  ): Promise<AnalysisResult> {
    const [canaryMetrics, baselineMetrics] = await Promise.all([
      this.getMetrics(canaryService, durationMinutes),
      this.getMetrics(baselineService, durationMinutes),
    ]);

    const comparison = {
      errorRateRatio: canaryMetrics.errorRate / (baselineMetrics.errorRate || 0.001),
      latencyRatio: canaryMetrics.latencyP99 / baselineMetrics.latencyP99,
    };

    const reasons: string[] = [];
    let passed = true;

    // Check error rate
    if (canaryMetrics.errorRate > 0.01) {
      reasons.push(`Error rate ${(canaryMetrics.errorRate * 100).toFixed(2)}% exceeds 1%`);
      passed = false;
    }

    if (comparison.errorRateRatio > 1.5) {
      reasons.push(`Error rate ${comparison.errorRateRatio.toFixed(2)}x higher than baseline`);
      passed = false;
    }

    // Check latency
    if (comparison.latencyRatio > 1.2) {
      reasons.push(`P99 latency ${comparison.latencyRatio.toFixed(2)}x higher than baseline`);
      passed = false;
    }

    // Check request volume (ensure statistically significant)
    if (canaryMetrics.requestsPerSecond < 1) {
      reasons.push('Insufficient traffic for analysis');
      passed = false;
    }

    return {
      passed,
      metrics: { canary: canaryMetrics, baseline: baselineMetrics },
      comparison,
      reasons,
    };
  }

  private async getMetrics(service: string, minutes: number): Promise<CanaryMetrics> {
    const range = `[${minutes}m]`;
    
    const [errorRate, latencyP50, latencyP95, latencyP99, rps] = await Promise.all([
      this.query(`sum(rate(http_requests_total{service="${service}",status=~"5.."}${range})) / sum(rate(http_requests_total{service="${service}"}${range}))`),
      this.query(`histogram_quantile(0.50, sum(rate(http_request_duration_seconds_bucket{service="${service}"}${range})) by (le))`),
      this.query(`histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{service="${service}"}${range})) by (le))`),
      this.query(`histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket{service="${service}"}${range})) by (le))`),
      this.query(`sum(rate(http_requests_total{service="${service}"}${range}))`),
    ]);

    return {
      errorRate: errorRate || 0,
      latencyP50: (latencyP50 || 0) * 1000, // Convert to ms
      latencyP95: (latencyP95 || 0) * 1000,
      latencyP99: (latencyP99 || 0) * 1000,
      requestsPerSecond: rps || 0,
    };
  }

  private async query(promql: string): Promise<number> {
    const response = await fetch(
      `${this.prometheusUrl}/api/v1/query?query=${encodeURIComponent(promql)}`
    );
    const data = await response.json();
    return parseFloat(data.data?.result?.[0]?.value?.[1]) || 0;
  }
}

// Usage in CI/CD
async function runCanaryAnalysis() {
  const analyzer = new CanaryAnalyzer('http://prometheus:9090');
  
  const result = await analyzer.analyze(
    'myapp-canary',
    'myapp-stable',
    30 // 30 minutes
  );

  if (!result.passed) {
    console.error('Canary analysis failed:', result.reasons);
    // Trigger rollback
    process.exit(1);
  }

  console.log('Canary analysis passed, promoting...');
}
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# CANARY PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: No stickiness - users see inconsistent behavior
# Bad
canary_weight: 10%
stickiness: false
# User sees v1, refreshes, sees v2, confusing!

# Good
canary_weight: 10%
stickiness:
  enabled: true
  duration: 3600s  # 1 hour
# Same user consistently sees same version

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Insufficient traffic for analysis
# Bad
canary_weight: 1%
analysis_duration: 5m
# Only 100 requests total, not statistically significant

# Good
canary_weight: 5%
analysis_duration: 30m
min_requests: 1000
# Enough data for reliable analysis

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Only checking error rates
# Bad
success_criteria:
  - error_rate < 1%
# Misses latency degradation, memory leaks, etc.

# Good
success_criteria:
  - error_rate < 1%
  - latency_p99 < baseline + 10%
  - memory_usage < 80%
  - business_metric_conversion >= baseline - 5%

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Manual promotion decisions
# Bad
# Engineer checks metrics manually
# Delayed promotions, inconsistent decisions

# Good
# Automated analysis with clear thresholds
analysis:
  auto_promote: true
  auto_rollback: true
  thresholds:
    error_rate: 0.01
    latency_increase: 10%

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Database schema incompatibility
# Bad
# Canary uses new schema
# Baseline can't read canary's data

# Good
# Backward compatible schema changes
# Expand-contract pattern
# Feature flags for new functionality

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Ignoring business metrics
# Bad
# Only technical metrics
# Canary has good latency but breaks checkout flow

# Good
success_criteria:
  - technical:
      error_rate: < 1%
      latency: < 500ms
  - business:
      conversion_rate: >= baseline - 2%
      checkout_success: >= baseline - 1%
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is a canary release?"**
> "Gradually shifting traffic from current version to new version - typically 1% → 10% → 50% → 100%. Named after canary in coal mine - if canary (small traffic subset) has issues, roll back before affecting all users. Minimizes blast radius of bad deployments."

**Q: "Canary vs blue-green?"**
> "Blue-green: Instant 0% → 100% switch, full rollback. Canary: Gradual traffic shift with analysis at each stage. Canary is lower risk (small blast radius), better for high-traffic systems. Blue-green is faster, simpler, good when you have confidence."

**Q: "What metrics do you monitor during canary?"**
> "Error rate (5xx), latency (P50, P95, P99), throughput, resource usage (CPU, memory), and business metrics (conversion, checkout success). Compare canary vs baseline. Look for statistical significance before decisions."

### Intermediate Questions

**Q: "How do you ensure canary analysis is statistically significant?"**
> "Require minimum request count (e.g., 1000). Run analysis for sufficient duration (15-30 min). Compare ratios not absolutes (canary error rate / baseline error rate). Use confidence intervals. For low-traffic services, extend duration or increase canary percentage."

**Q: "How do you handle session affinity in canary?"**
> "Enable stickiness via cookie or IP hash. Once user hits canary, they stay on canary for session. Prevents confusing experience of seeing different versions. Configure duration based on typical session length."

**Q: "What's traffic mirroring and when do you use it?"**
> "Copy production traffic to canary without serving responses. Canary processes requests but responses are discarded. Test performance under real load without user impact. Use before percentage-based canary for high-risk changes."

### Advanced Questions

**Q: "How do you design automated canary promotion/rollback?"**
> "Define clear metric thresholds. Run analysis at each stage (5%, 25%, 50%, 100%). Compare canary metrics to baseline using statistical methods. Auto-promote if within bounds. Auto-rollback if any metric breaches threshold. Alert on-call on rollback. Create incident for investigation."

**Q: "How do you handle canary for database changes?"**
> "Schema changes must be backward compatible. Use expand-contract pattern. Canary and baseline must work with same database. Feature flags for new functionality. Never deploy breaking schema changes via canary - they require coordinated migration."

**Q: "How would you implement canary for a globally distributed system?"**
> "Start with single region. Monitor cross-region metrics. Consider regional canary (full canary in one region first). Account for latency between regions. Ensure data replication compatibility. Use region-specific traffic routing. Coordinate global rollout after regional success."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                   CANARY RELEASE CHECKLIST                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PRE-DEPLOYMENT:                                                │
│  □ Schema changes backward compatible                          │
│  □ Feature flags for new functionality                         │
│  □ Metrics instrumented for canary service                     │
│  □ Analysis thresholds defined                                 │
│                                                                 │
│  CANARY STAGES:                                                 │
│  □ Stage 1: 5% traffic, 30 min analysis                        │
│  □ Stage 2: 25% traffic, 30 min analysis                       │
│  □ Stage 3: 50% traffic, 1 hour analysis                       │
│  □ Stage 4: 100% traffic                                       │
│                                                                 │
│  ANALYSIS METRICS:                                              │
│  □ Error rate (< 1%, < 1.5x baseline)                          │
│  □ Latency P99 (< 10% increase)                                │
│  □ Resource usage                                              │
│  □ Business metrics                                            │
│                                                                 │
│  ROLLBACK TRIGGERS:                                             │
│  □ Error rate threshold breach                                 │
│  □ Latency threshold breach                                    │
│  □ Business metric degradation                                 │
│  □ Manual abort                                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

ARGO ROLLOUTS COMMANDS:
┌────────────────────────────────────────────────────────────────┐
│ kubectl argo rollouts get rollout <name> -w    # Watch        │
│ kubectl argo rollouts promote <name>           # Promote      │
│ kubectl argo rollouts abort <name>             # Abort        │
│ kubectl argo rollouts retry <name>             # Retry        │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

