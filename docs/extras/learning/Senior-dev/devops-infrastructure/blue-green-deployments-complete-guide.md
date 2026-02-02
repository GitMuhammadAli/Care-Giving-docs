# 🔵🟢 Blue-Green Deployments - Complete Guide

> A comprehensive guide to blue-green deployments - zero-downtime deployments, rollback strategies, traffic switching, and implementation patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Blue-green deployment is a release strategy that maintains two identical production environments (blue and green) - traffic is switched from the current version (blue) to the new version (green) instantaneously, enabling zero-downtime deployments and instant rollback by switching back."

### The 7 Key Concepts (Remember These!)
```
1. BLUE ENVIRONMENT   → Current production (receiving traffic)
2. GREEN ENVIRONMENT  → New version (idle, being prepared)
3. TRAFFIC SWITCH     → Instant cutover from blue to green
4. ROLLBACK          → Switch back to blue if issues detected
5. ENVIRONMENT PARITY → Blue and green must be identical
6. DATABASE STRATEGY  → Handle schema changes carefully
7. SMOKE TESTS       → Validate green before switching
```

### Blue-Green Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                BLUE-GREEN DEPLOYMENT                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PHASE 1: Deploy to Green (Blue is live)                       │
│  ─────────────────────────────────────────                      │
│                                                                 │
│      Users → [Router/LB] → [BLUE v1.0] ← 100% traffic          │
│                    │                                           │
│                    └──X──▶ [GREEN v2.0] ← Deploy here          │
│                           (no traffic)                          │
│                                                                 │
│  PHASE 2: Switch Traffic                                        │
│  ──────────────────────                                         │
│                                                                 │
│      Users → [Router/LB] → [GREEN v2.0] ← 100% traffic         │
│                    │                                           │
│                    └──X──▶ [BLUE v1.0]  ← Idle (rollback ready)│
│                                                                 │
│  PHASE 3: Rollback (if needed)                                 │
│  ────────────────────────────                                   │
│                                                                 │
│      Users → [Router/LB] → [BLUE v1.0] ← Back to v1.0          │
│                    │                                           │
│                    └──X──▶ [GREEN v2.0] ← Fix and redeploy     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Comparison with Other Strategies
```
┌─────────────────────────────────────────────────────────────────┐
│              DEPLOYMENT STRATEGY COMPARISON                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  RECREATE                                                       │
│  ─────────                                                      │
│  Stop v1 → Start v2                                            │
│  ✗ Downtime during transition                                  │
│  ✓ Simple, low resource cost                                   │
│                                                                 │
│  ROLLING UPDATE                                                 │
│  ──────────────                                                 │
│  Gradually replace v1 pods with v2                             │
│  ✓ Zero downtime                                               │
│  ✗ Mixed versions during rollout                               │
│  ✗ Slow rollback (must re-roll)                                │
│                                                                 │
│  BLUE-GREEN                                                     │
│  ──────────                                                     │
│  Full v2 environment, instant switch                           │
│  ✓ Zero downtime                                               │
│  ✓ Instant rollback                                            │
│  ✓ Full testing before switch                                  │
│  ✗ 2x resources during deployment                              │
│                                                                 │
│  CANARY                                                         │
│  ──────                                                         │
│  Gradual traffic shift: 1% → 10% → 50% → 100%                  │
│  ✓ Risk minimization                                           │
│  ✓ Real user testing                                           │
│  ✗ Slower rollout                                              │
│  ✗ More complex monitoring                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Instant cutover"** | "Blue-green gives us instant cutover with single switch" |
| **"Rollback in seconds"** | "Issues? Rollback in seconds by switching back to blue" |
| **"Environment parity"** | "We maintain strict environment parity between blue and green" |
| **"Pre-flight checks"** | "Green passes pre-flight checks before receiving traffic" |
| **"Dual-write"** | "We use dual-write for database compatibility" |
| **"Traffic drain"** | "Allow traffic to drain before decommissioning" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Deployment time | **Minutes** | Deploy + smoke test |
| Rollback time | **Seconds** | Just traffic switch |
| Resource overhead | **2x** | Two environments |
| Smoke test coverage | **Critical paths** | Verify before switch |
| DNS TTL | **60 seconds or less** | Fast propagation |
| Idle environment retention | **1-24 hours** | Rollback window |

### The "Wow" Statement (Memorize This!)
> "We use blue-green deployments for all production releases. CI/CD deploys to the idle environment (green), runs comprehensive smoke tests including API health checks, critical user journeys, and database connectivity. Once green passes all checks, we switch ALB target groups atomically - traffic moves from blue to green in under a second. We monitor error rates, latency P99, and business metrics for 15 minutes post-switch. If any metric breaches thresholds, automated rollback switches back to blue instantly. For database changes, we use expand-contract pattern - schema changes are backward compatible, applied before code deploy. Blue stays warm for 4 hours as rollback safety net. This gives us zero-downtime deployments with <5 second rollback capability."

---

## 📚 Table of Contents

1. [Implementation Patterns](#1-implementation-patterns)
2. [AWS Implementation](#2-aws-implementation)
3. [Kubernetes Implementation](#3-kubernetes-implementation)
4. [Database Strategies](#4-database-strategies)
5. [Rollback Strategies](#5-rollback-strategies)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Implementation Patterns

```yaml
# ════════════════════════════════════════════════════════════════
# BLUE-GREEN DEPLOYMENT PATTERNS
# ════════════════════════════════════════════════════════════════

# PATTERN 1: DNS-Based Switch
# ───────────────────────────
# Use Route53 weighted routing

Phase 1 (Blue Live):
  blue.example.com → Blue ALB (weight: 100)
  green.example.com → Green ALB (weight: 0)
  api.example.com → CNAME to blue.example.com

Phase 2 (Switch to Green):
  api.example.com → CNAME to green.example.com
  # Or use weighted routing: green weight = 100, blue weight = 0

Pros:
  - Simple to implement
  - Works with any infrastructure

Cons:
  - DNS TTL causes propagation delay
  - Some clients cache DNS longer
  - Not instant (minutes, not seconds)

# ════════════════════════════════════════════════════════════════

# PATTERN 2: Load Balancer Target Group Switch
# ────────────────────────────────────────────
# Recommended for AWS - instant switch

Phase 1 (Blue Live):
  ALB Listener → Blue Target Group (instances running v1)
  Green Target Group exists but no traffic

Phase 2 (Deploy to Green):
  Deploy v2 to Green Target Group instances
  Run smoke tests against Green directly

Phase 3 (Switch):
  ALB Listener → Green Target Group
  # Instant! No DNS propagation

Pros:
  - Instant traffic switch
  - No DNS delays
  - Works with health checks

Cons:
  - Cloud-provider specific
  - Requires target group management

# ════════════════════════════════════════════════════════════════

# PATTERN 3: Kubernetes Service Selector
# ──────────────────────────────────────

Phase 1 (Blue Live):
  Service selector: version=blue
  Deployment blue: version=blue, replicas=10
  Deployment green: version=green, replicas=0

Phase 2 (Deploy to Green):
  Scale green deployment: replicas=10
  Run smoke tests via green service

Phase 3 (Switch):
  Update service selector: version=green
  # Instant switch via kube-proxy

Pros:
  - Native Kubernetes
  - Fast switch (seconds)
  - Easy rollback

Cons:
  - Both deployments need resources
  - Requires version labels
```

---

## 2. AWS Implementation

```hcl
# ════════════════════════════════════════════════════════════════
# AWS BLUE-GREEN WITH ALB - TERRAFORM
# ════════════════════════════════════════════════════════════════

# ALB (single load balancer for both environments)
resource "aws_lb" "main" {
  name               = "app-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = var.public_subnet_ids
}

# Blue Target Group
resource "aws_lb_target_group" "blue" {
  name        = "app-blue-tg"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = var.vpc_id
  target_type = "instance"

  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 10
    path                = "/health"
    timeout             = 5
    unhealthy_threshold = 3
  }

  deregistration_delay = 30
}

# Green Target Group
resource "aws_lb_target_group" "green" {
  name        = "app-green-tg"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = var.vpc_id
  target_type = "instance"

  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 10
    path                = "/health"
    timeout             = 5
    unhealthy_threshold = 3
  }

  deregistration_delay = 30
}

# HTTPS Listener - Points to active environment
resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.main.arn
  port              = 443
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS13-1-2-2021-06"
  certificate_arn   = var.certificate_arn

  # Default to blue (active)
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.blue.arn
  }

  lifecycle {
    ignore_changes = [default_action]  # Managed by deployment
  }
}

# Blue Auto Scaling Group
resource "aws_autoscaling_group" "blue" {
  name                = "app-blue-asg"
  desired_capacity    = var.blue_desired_capacity
  min_size            = var.blue_min_size
  max_size            = var.blue_max_size
  vpc_zone_identifier = var.private_subnet_ids
  target_group_arns   = [aws_lb_target_group.blue.arn]

  launch_template {
    id      = aws_launch_template.blue.id
    version = "$Latest"
  }

  health_check_type         = "ELB"
  health_check_grace_period = 300

  tag {
    key                 = "Environment"
    value               = "blue"
    propagate_at_launch = true
  }
}

# Green Auto Scaling Group
resource "aws_autoscaling_group" "green" {
  name                = "app-green-asg"
  desired_capacity    = var.green_desired_capacity
  min_size            = var.green_min_size
  max_size            = var.green_max_size
  vpc_zone_identifier = var.private_subnet_ids
  target_group_arns   = [aws_lb_target_group.green.arn]

  launch_template {
    id      = aws_launch_template.green.id
    version = "$Latest"
  }

  health_check_type         = "ELB"
  health_check_grace_period = 300

  tag {
    key                 = "Environment"
    value               = "green"
    propagate_at_launch = true
  }
}
```

```bash
#!/bin/bash
# ════════════════════════════════════════════════════════════════
# BLUE-GREEN DEPLOYMENT SCRIPT
# ════════════════════════════════════════════════════════════════

set -e

# Configuration
ALB_LISTENER_ARN="arn:aws:elasticloadbalancing:..."
BLUE_TG_ARN="arn:aws:elasticloadbalancing:...:targetgroup/app-blue-tg/..."
GREEN_TG_ARN="arn:aws:elasticloadbalancing:...:targetgroup/app-green-tg/..."
GREEN_ASG_NAME="app-green-asg"
SMOKE_TEST_URL="https://green-internal.example.com"

echo "🔵 Starting Blue-Green Deployment"

# Step 1: Get current active environment
CURRENT_TG=$(aws elbv2 describe-listeners \
  --listener-arns $ALB_LISTENER_ARN \
  --query 'Listeners[0].DefaultActions[0].TargetGroupArn' \
  --output text)

if [ "$CURRENT_TG" == "$BLUE_TG_ARN" ]; then
  ACTIVE="blue"
  IDLE="green"
  IDLE_TG_ARN=$GREEN_TG_ARN
  IDLE_ASG=$GREEN_ASG_NAME
else
  ACTIVE="green"
  IDLE="blue"
  IDLE_TG_ARN=$BLUE_TG_ARN
  IDLE_ASG="app-blue-asg"
fi

echo "📍 Current active: $ACTIVE"
echo "📍 Deploying to: $IDLE"

# Step 2: Scale up idle environment
echo "📈 Scaling up $IDLE environment..."
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name $IDLE_ASG \
  --min-size 3 \
  --max-size 10 \
  --desired-capacity 3

# Step 3: Wait for instances to be healthy
echo "⏳ Waiting for instances to be healthy..."
aws elbv2 wait target-in-service \
  --target-group-arn $IDLE_TG_ARN

# Step 4: Run smoke tests
echo "🧪 Running smoke tests..."
SMOKE_TEST_RESULT=$(curl -s -o /dev/null -w "%{http_code}" $SMOKE_TEST_URL/health)
if [ "$SMOKE_TEST_RESULT" != "200" ]; then
  echo "❌ Smoke tests failed! Aborting deployment."
  exit 1
fi

# Additional smoke tests
./scripts/smoke-tests.sh $SMOKE_TEST_URL
if [ $? -ne 0 ]; then
  echo "❌ Integration smoke tests failed!"
  exit 1
fi

echo "✅ Smoke tests passed"

# Step 5: Switch traffic
echo "🔄 Switching traffic to $IDLE..."
aws elbv2 modify-listener \
  --listener-arn $ALB_LISTENER_ARN \
  --default-actions Type=forward,TargetGroupArn=$IDLE_TG_ARN

echo "✅ Traffic switched to $IDLE"

# Step 6: Monitor (optional - could be separate job)
echo "👀 Monitoring for errors..."
sleep 60

# Check error rate
# If errors detected, auto-rollback would trigger here

# Step 7: Scale down old environment (after monitoring period)
echo "📉 Scaling down $ACTIVE environment..."
# Keep at minimum for rollback capability
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name "app-${ACTIVE}-asg" \
  --min-size 1 \
  --max-size 1 \
  --desired-capacity 1

echo "✅ Blue-Green deployment complete!"
echo "🔵 Old ($ACTIVE) kept at min capacity for rollback"
```

---

## 3. Kubernetes Implementation

```yaml
# ════════════════════════════════════════════════════════════════
# KUBERNETES BLUE-GREEN DEPLOYMENT
# ════════════════════════════════════════════════════════════════

# Blue Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-blue
  labels:
    app: myapp
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
        - name: app
          image: myapp:1.0.0
          ports:
            - containerPort: 3000
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 5

---
# Green Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-green
  labels:
    app: myapp
    version: green
spec:
  replicas: 0  # Start with 0, scale up for deployment
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
        - name: app
          image: myapp:2.0.0
          ports:
            - containerPort: 3000
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 5

---
# Production Service (points to active environment)
apiVersion: v1
kind: Service
metadata:
  name: app
spec:
  selector:
    app: myapp
    version: blue  # Switch this to change environments
  ports:
    - port: 80
      targetPort: 3000

---
# Blue Test Service (for smoke tests)
apiVersion: v1
kind: Service
metadata:
  name: app-blue
spec:
  selector:
    app: myapp
    version: blue
  ports:
    - port: 80
      targetPort: 3000

---
# Green Test Service (for smoke tests)
apiVersion: v1
kind: Service
metadata:
  name: app-green
spec:
  selector:
    app: myapp
    version: green
  ports:
    - port: 80
      targetPort: 3000
```

```bash
#!/bin/bash
# ════════════════════════════════════════════════════════════════
# KUBERNETES BLUE-GREEN DEPLOYMENT SCRIPT
# ════════════════════════════════════════════════════════════════

set -e

NEW_IMAGE="myapp:2.0.0"
NAMESPACE="production"

# Get current active version
CURRENT_VERSION=$(kubectl get svc app -n $NAMESPACE -o jsonpath='{.spec.selector.version}')

if [ "$CURRENT_VERSION" == "blue" ]; then
  NEW_VERSION="green"
  OLD_VERSION="blue"
else
  NEW_VERSION="blue"
  OLD_VERSION="green"
fi

echo "Current: $CURRENT_VERSION, Deploying to: $NEW_VERSION"

# Step 1: Update image in idle deployment
kubectl set image deployment/app-$NEW_VERSION \
  app=$NEW_IMAGE \
  -n $NAMESPACE

# Step 2: Scale up idle deployment
kubectl scale deployment app-$NEW_VERSION \
  --replicas=3 \
  -n $NAMESPACE

# Step 3: Wait for rollout
kubectl rollout status deployment/app-$NEW_VERSION -n $NAMESPACE

# Step 4: Run smoke tests
echo "Running smoke tests..."
kubectl run smoke-test --rm -i --restart=Never \
  --image=curlimages/curl -- \
  curl -f http://app-$NEW_VERSION/health

# Step 5: Switch traffic
kubectl patch svc app -n $NAMESPACE \
  -p "{\"spec\":{\"selector\":{\"version\":\"$NEW_VERSION\"}}}"

echo "Traffic switched to $NEW_VERSION"

# Step 6: Monitor
sleep 60

# Step 7: Scale down old version (keep 1 for rollback)
kubectl scale deployment app-$OLD_VERSION \
  --replicas=1 \
  -n $NAMESPACE

echo "Deployment complete!"
```

---

## 4. Database Strategies

```yaml
# ════════════════════════════════════════════════════════════════
# DATABASE STRATEGIES FOR BLUE-GREEN
# ════════════════════════════════════════════════════════════════

# CHALLENGE:
# Blue and Green may need different database schemas
# Can't have downtime during migration

# STRATEGY 1: Expand-Contract Pattern
# ────────────────────────────────────
# Make schema changes backward compatible

Phase 1 - EXPAND (while Blue is live):
  - Add new columns (nullable)
  - Add new tables
  - Keep old columns/tables
  
Phase 2 - DEPLOY (switch to Green):
  - Green code uses new schema
  - Blue code still works (old schema compatible)
  
Phase 3 - CONTRACT (after Green is stable):
  - Remove old columns
  - Remove old tables
  - Clean up

Example:
  # Adding a new required column "email_verified"
  
  # Phase 1: Add nullable column
  ALTER TABLE users ADD COLUMN email_verified BOOLEAN DEFAULT false;
  
  # Phase 2: Deploy Green (code handles both cases)
  
  # Phase 3: Make column required (after Green stable)
  ALTER TABLE users ALTER COLUMN email_verified SET NOT NULL;

# ════════════════════════════════════════════════════════════════

# STRATEGY 2: Dual-Write
# ──────────────────────
# Write to both old and new schema/database

Blue v1 → writes to old schema
Green v2 → writes to both old AND new schema

# During transition, both schemas are in sync
# After switch, stop writing to old schema

# ════════════════════════════════════════════════════════════════

# STRATEGY 3: Separate Databases
# ──────────────────────────────
# Each environment has its own database

Blue  → Database-Blue
Green → Database-Green

# Requires data sync strategy:
# - Read replicas
# - CDC (Change Data Capture)
# - Event sourcing

# Complex but provides complete isolation
```

```typescript
// ════════════════════════════════════════════════════════════════
// BACKWARD COMPATIBLE CODE EXAMPLE
// ════════════════════════════════════════════════════════════════

// Migration: Adding "email_verified" field to users

// BAD: Breaking change
interface User {
  id: string;
  email: string;
  emailVerified: boolean; // Required - breaks Blue
}

// GOOD: Backward compatible
interface User {
  id: string;
  email: string;
  emailVerified?: boolean; // Optional - works with both schemas
}

// Code handles both cases
function isEmailVerified(user: User): boolean {
  // Default to false if field doesn't exist (Blue schema)
  return user.emailVerified ?? false;
}

// Write handles both cases
async function createUser(data: CreateUserInput) {
  const user = {
    id: generateId(),
    email: data.email,
    // Include new field if schema supports it
    ...(await schemaSupportsEmailVerified() && {
      emailVerified: false
    })
  };
  await db.users.insert(user);
}
```

---

## 5. Rollback Strategies

```yaml
# ════════════════════════════════════════════════════════════════
# ROLLBACK STRATEGIES
# ════════════════════════════════════════════════════════════════

# AUTOMATIC ROLLBACK
# ──────────────────
# Triggered by monitoring alerts

triggers:
  - error_rate > 1%
  - latency_p99 > 2000ms
  - health_check_failures > 3
  - business_metric_drop > 10%

action: |
  1. Alert on-call
  2. Switch traffic back to Blue
  3. Log rollback event
  4. Create incident ticket

# ════════════════════════════════════════════════════════════════

# MANUAL ROLLBACK
# ───────────────
# Human-initiated based on observation

# AWS ALB
aws elbv2 modify-listener \
  --listener-arn $LISTENER_ARN \
  --default-actions Type=forward,TargetGroupArn=$BLUE_TG_ARN

# Kubernetes
kubectl patch svc app -p '{"spec":{"selector":{"version":"blue"}}}'

# DNS (slower)
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123 \
  --change-batch file://rollback-dns.json

# ════════════════════════════════════════════════════════════════

# ROLLBACK TIMELINE
# ─────────────────

0-15 min:   Instant rollback available (switch traffic)
15-60 min:  Rollback available, may need warm-up time
1-4 hours:  Rollback available, Blue at min capacity
4+ hours:   Blue decommissioned, need redeploy to rollback
```

```bash
#!/bin/bash
# ════════════════════════════════════════════════════════════════
# AUTOMATED ROLLBACK SCRIPT
# ════════════════════════════════════════════════════════════════

# Called by monitoring system when thresholds breached

set -e

LISTENER_ARN=$1
ROLLBACK_TG_ARN=$2
SLACK_WEBHOOK=$3

echo "🚨 Initiating automatic rollback!"

# Switch traffic immediately
aws elbv2 modify-listener \
  --listener-arn $LISTENER_ARN \
  --default-actions Type=forward,TargetGroupArn=$ROLLBACK_TG_ARN

# Notify team
curl -X POST $SLACK_WEBHOOK \
  -H 'Content-type: application/json' \
  -d '{
    "text": "🚨 AUTOMATIC ROLLBACK TRIGGERED",
    "attachments": [{
      "color": "danger",
      "fields": [
        {"title": "Environment", "value": "Production", "short": true},
        {"title": "Action", "value": "Traffic switched to previous version", "short": true}
      ]
    }]
  }'

# Create incident ticket
./create-incident.sh "Automatic rollback triggered"

echo "✅ Rollback complete"
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# BLUE-GREEN PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Database schema incompatibility
# Bad
# Green requires new required column
# Blue fails when switched back

# Good
# Use expand-contract pattern
# All changes backward compatible

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Long-running requests cut off
# Bad
# Switch traffic, in-flight requests to Blue fail

# Good
# Connection draining on old environment
# Wait for requests to complete before full switch
deregistration_delay: 30  # ALB waits 30s for in-flight

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Session state lost
# Bad
# User sessions stored in Blue
# Switch to Green, users logged out

# Good
# Externalize session storage (Redis)
# Or: Session replication between environments

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: DNS TTL too high
# Bad
dns_ttl: 3600  # 1 hour
# Some clients still hitting old environment

# Good
dns_ttl: 60  # 1 minute
# Or: Use ALB target groups (instant, no DNS)

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Insufficient smoke tests
# Bad
# Only check /health
# Miss critical functionality issues

# Good
smoke_tests:
  - health_check
  - authentication_flow
  - key_api_endpoints
  - database_connectivity
  - external_service_connectivity

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Decommissioning too fast
# Bad
# Scale down Blue immediately after switch
# Issue detected, Blue not available

# Good
# Keep Blue at min capacity for rollback window
rollback_window: 4_hours
# Only decommission after confidence established
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is blue-green deployment?"**
> "Maintaining two identical production environments. Blue is current live, Green is new version. Deploy to Green, test it, then switch all traffic instantly. If issues, switch back to Blue. Zero downtime, instant rollback."

**Q: "Benefits of blue-green?"**
> "Zero downtime deployment. Instant rollback capability. Full testing before going live. Reduced deployment risk. Clear separation between versions. Consistent environment (no mixed versions during deployment)."

**Q: "Blue-green vs rolling deployment?"**
> "Rolling: Gradual replacement, mixed versions during rollout, slower rollback. Blue-green: Instant switch, no mixed versions, instant rollback, but requires 2x resources during deployment. Blue-green better for risk-averse, rolling better for resource-constrained."

### Intermediate Questions

**Q: "How do you handle database changes?"**
> "Expand-contract pattern: 1) Expand - add new columns/tables as nullable/optional (backward compatible). 2) Deploy - new code uses new schema. 3) Contract - remove old schema after stable. Never make breaking changes. Both Blue and Green must work with same database."

**Q: "How do you test Green before switching?"**
> "Smoke tests via internal endpoint or test service. Check: health endpoints, authentication, critical APIs, database connectivity, external services. Run integration tests. Optional: synthetic transactions. Only switch after all tests pass."

**Q: "How do you handle session state?"**
> "Externalize sessions to Redis or database shared by both environments. When traffic switches, sessions persist. Alternative: session replication between environments. Never store sessions only in application memory."

### Advanced Questions

**Q: "How would you implement automated rollback?"**
> "Monitor key metrics post-switch: error rate, latency P99, health check failures, business metrics. Set thresholds. If breached, automatically switch traffic back to Blue. Alert on-call. Create incident. Keep Blue warm for fast rollback. Typical monitoring window: 15-30 minutes."

**Q: "How do you handle long-running requests during switch?"**
> "Connection draining - configure deregistration delay (30-60s). ALB stops sending new requests but allows in-flight to complete. For very long requests: consider WebSocket upgrade, or gradual traffic shift before full cutover."

**Q: "Blue-green with microservices?"**
> "Each service does independent blue-green. Ensure API compatibility between versions. Use API versioning. Service mesh (Istio) can help manage traffic switching. Coordinate releases for breaking changes. Consider using canary for tightly coupled services."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  BLUE-GREEN CHECKLIST                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PRE-DEPLOYMENT:                                                │
│  □ Green environment provisioned and identical to Blue         │
│  □ Database changes are backward compatible                    │
│  □ Session storage externalized                                │
│  □ Smoke tests prepared                                        │
│                                                                 │
│  DEPLOYMENT:                                                    │
│  □ Deploy new version to Green                                 │
│  □ Run smoke tests against Green                               │
│  □ Verify health checks passing                                │
│  □ Verify database connectivity                                │
│                                                                 │
│  TRAFFIC SWITCH:                                                │
│  □ Switch traffic to Green                                     │
│  □ Monitor error rates and latency                             │
│  □ Monitor business metrics                                    │
│  □ Keep Blue warm for rollback                                 │
│                                                                 │
│  POST-DEPLOYMENT:                                               │
│  □ Monitor for 15-30 minutes                                   │
│  □ Scale down Blue after confidence                            │
│  □ Update documentation                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

ROLLBACK COMMAND:
┌────────────────────────────────────────────────────────────────┐
│ AWS: aws elbv2 modify-listener --default-actions ...           │
│ K8s: kubectl patch svc app -p '{"spec":{"selector":...}}'     │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

