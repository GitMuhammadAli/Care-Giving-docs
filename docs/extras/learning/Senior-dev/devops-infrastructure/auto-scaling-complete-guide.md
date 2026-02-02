# 📈 Auto Scaling - Complete Guide

> A comprehensive guide to auto scaling - horizontal vs vertical scaling, scaling policies, metrics, AWS Auto Scaling, and Kubernetes HPA.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Auto scaling automatically adjusts computing resources based on demand - scaling out (adding instances) when load increases and scaling in (removing instances) when load decreases, optimizing for both performance and cost."

### The 7 Key Concepts (Remember These!)
```
1. HORIZONTAL SCALING  → Add/remove instances (scale out/in)
2. VERTICAL SCALING    → Change instance size (scale up/down)
3. SCALING POLICY      → Rules that trigger scaling actions
4. COOLDOWN PERIOD     → Time to wait between scaling actions
5. TARGET TRACKING     → Maintain metric at target value
6. STEP SCALING        → Scale based on metric thresholds
7. PREDICTIVE SCALING  → Scale based on forecasted demand
```

### Horizontal vs Vertical Scaling
```
┌─────────────────────────────────────────────────────────────────┐
│           HORIZONTAL vs VERTICAL SCALING                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  VERTICAL SCALING (Scale Up/Down)                              │
│  ────────────────────────────────                               │
│                                                                 │
│  Before:           After:                                       │
│  ┌──────────┐      ┌──────────────────┐                        │
│  │ 2 CPU    │  →   │     8 CPU        │                        │
│  │ 4GB RAM  │      │    32GB RAM      │                        │
│  └──────────┘      └──────────────────┘                        │
│                                                                 │
│  • Simple to implement                                         │
│  • Has upper limit (largest instance)                          │
│  • Often requires downtime                                     │
│  • Single point of failure                                     │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  HORIZONTAL SCALING (Scale Out/In)                             │
│  ─────────────────────────────────                              │
│                                                                 │
│  Before:           After:                                       │
│  ┌──────────┐      ┌──────────┐ ┌──────────┐ ┌──────────┐     │
│  │ Server 1 │  →   │ Server 1 │ │ Server 2 │ │ Server 3 │     │
│  └──────────┘      └──────────┘ └──────────┘ └──────────┘     │
│                           │            │            │          │
│                           └────────────┼────────────┘          │
│                                        │                        │
│                                   [Load Balancer]               │
│                                                                 │
│  • Highly scalable (no upper limit)                            │
│  • No downtime                                                 │
│  • Requires stateless design                                   │
│  • More complex (LB, state management)                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Auto Scaling Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                  AUTO SCALING FLOW                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌────────────────────────────────────────────────────────┐   │
│  │                     MONITORING                          │   │
│  │   CloudWatch / Prometheus / Custom Metrics              │   │
│  │   CPU: 85% | Memory: 70% | Requests: 5000/min          │   │
│  └────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌────────────────────────────────────────────────────────┐   │
│  │                   SCALING POLICIES                      │   │
│  │   Target Tracking: CPU < 70%                           │   │
│  │   Step Scaling: if CPU > 80% add 2 instances           │   │
│  │   Scheduled: Scale to 10 at 9 AM                       │   │
│  └────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌────────────────────────────────────────────────────────┐   │
│  │               AUTO SCALING GROUP                        │   │
│  │   Min: 2 | Desired: 4 | Max: 20                        │   │
│  │                                                        │   │
│  │   ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐                 │   │
│  │   │ EC2 │  │ EC2 │  │ EC2 │  │ EC2 │                 │   │
│  │   │  1  │  │  2  │  │  3  │  │  4  │                 │   │
│  │   └─────┘  └─────┘  └─────┘  └─────┘                 │   │
│  └────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌────────────────────────────────────────────────────────┐   │
│  │                   LOAD BALANCER                         │   │
│  │   Distributes traffic to healthy instances             │   │
│  └────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Target tracking"** | "We use target tracking to maintain CPU at 70%" |
| **"Cooldown period"** | "5-minute cooldown prevents scaling thrashing" |
| **"Warm pool"** | "Warm pool keeps pre-initialized instances ready" |
| **"Predictive scaling"** | "Predictive scaling handles our daily traffic patterns" |
| **"Scale-in protection"** | "Scale-in protection on processing nodes" |
| **"Launch template"** | "Launch templates define our instance configuration" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| CPU target | **60-70%** | Leave headroom for spikes |
| Cooldown period | **300 seconds** | Prevents thrashing |
| Health check grace | **300 seconds** | Time for instance startup |
| Min instances | **2+** | High availability |
| Scale-out speed | **Faster** | Prioritize availability |
| Scale-in speed | **Slower** | Gradual cost savings |

### The "Wow" Statement (Memorize This!)
> "We run 50+ microservices with auto scaling. Each service has an Auto Scaling Group with target tracking policy maintaining 65% average CPU utilization. We use ALB request count per target as a secondary metric to handle traffic spikes. Scale-out is aggressive (add 50% capacity when threshold breached), scale-in is conservative (10% reduction with 10-minute cooldown). Predictive scaling pre-warms capacity for our 9 AM traffic spike based on 2-week patterns. We use mixed instances policy - 70% spot, 30% on-demand for cost optimization. Warm pools keep 5 pre-initialized instances ready for instant scaling. During Black Friday, scheduled actions scale to minimum 50 instances, with max capacity at 200."

---

## 📚 Table of Contents

1. [AWS Auto Scaling](#1-aws-auto-scaling)
2. [Scaling Policies](#2-scaling-policies)
3. [Kubernetes HPA/VPA](#3-kubernetes-hpavpa)
4. [Custom Metrics Scaling](#4-custom-metrics-scaling)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. AWS Auto Scaling

```hcl
# ════════════════════════════════════════════════════════════════
# AWS AUTO SCALING GROUP - TERRAFORM
# ════════════════════════════════════════════════════════════════

# Launch Template
resource "aws_launch_template" "app" {
  name_prefix   = "app-"
  image_id      = data.aws_ami.amazon_linux.id
  instance_type = "t3.medium"

  # IAM instance profile
  iam_instance_profile {
    name = aws_iam_instance_profile.app.name
  }

  # Network
  network_interfaces {
    associate_public_ip_address = false
    security_groups             = [aws_security_group.app.id]
  }

  # User data for initialization
  user_data = base64encode(<<-EOF
    #!/bin/bash
    yum update -y
    yum install -y docker
    systemctl start docker
    docker pull myapp:latest
    docker run -d -p 3000:3000 myapp:latest
  EOF
  )

  # Block device
  block_device_mappings {
    device_name = "/dev/xvda"
    ebs {
      volume_size           = 20
      volume_type           = "gp3"
      delete_on_termination = true
      encrypted             = true
    }
  }

  # Instance metadata options
  metadata_options {
    http_endpoint               = "enabled"
    http_tokens                 = "required"  # IMDSv2
    http_put_response_hop_limit = 1
  }

  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = "app-instance"
    }
  }

  lifecycle {
    create_before_destroy = true
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "app" {
  name                = "app-asg"
  desired_capacity    = 4
  min_size            = 2
  max_size            = 20
  vpc_zone_identifier = var.private_subnet_ids

  # Launch template
  launch_template {
    id      = aws_launch_template.app.id
    version = "$Latest"
  }

  # Mixed instances policy (On-Demand + Spot)
  mixed_instances_policy {
    instances_distribution {
      on_demand_base_capacity                  = 2
      on_demand_percentage_above_base_capacity = 30
      spot_allocation_strategy                 = "capacity-optimized"
    }

    launch_template {
      launch_template_specification {
        launch_template_id = aws_launch_template.app.id
      }

      override {
        instance_type = "t3.medium"
      }
      override {
        instance_type = "t3a.medium"
      }
      override {
        instance_type = "t3.large"
      }
    }
  }

  # Health check
  health_check_type         = "ELB"
  health_check_grace_period = 300

  # Target groups
  target_group_arns = [aws_lb_target_group.app.arn]

  # Instance refresh for deployments
  instance_refresh {
    strategy = "Rolling"
    preferences {
      min_healthy_percentage = 75
      instance_warmup        = 120
    }
  }

  # Lifecycle hooks
  initial_lifecycle_hook {
    name                 = "launch-hook"
    default_result       = "CONTINUE"
    heartbeat_timeout    = 300
    lifecycle_transition = "autoscaling:EC2_INSTANCE_LAUNCHING"
  }

  # Warm pool
  warm_pool {
    pool_state                  = "Stopped"
    min_size                    = 2
    max_group_prepared_capacity = 5
  }

  # Tags
  tag {
    key                 = "Name"
    value               = "app-instance"
    propagate_at_launch = true
  }

  lifecycle {
    ignore_changes = [desired_capacity]  # Allow auto scaling to manage
  }
}

# ════════════════════════════════════════════════════════════════
# SCALING POLICIES
# ════════════════════════════════════════════════════════════════

# Target Tracking - CPU
resource "aws_autoscaling_policy" "cpu_target" {
  name                   = "cpu-target-tracking"
  autoscaling_group_name = aws_autoscaling_group.app.name
  policy_type            = "TargetTrackingScaling"

  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value     = 65.0
    disable_scale_in = false
  }
}

# Target Tracking - ALB Request Count
resource "aws_autoscaling_policy" "alb_requests" {
  name                   = "alb-request-tracking"
  autoscaling_group_name = aws_autoscaling_group.app.name
  policy_type            = "TargetTrackingScaling"

  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ALBRequestCountPerTarget"
      resource_label         = "${aws_lb.main.arn_suffix}/${aws_lb_target_group.app.arn_suffix}"
    }
    target_value = 1000  # 1000 requests per target per minute
  }
}

# Step Scaling - Scale Out
resource "aws_autoscaling_policy" "scale_out" {
  name                   = "scale-out"
  autoscaling_group_name = aws_autoscaling_group.app.name
  policy_type            = "StepScaling"
  adjustment_type        = "PercentChangeInCapacity"

  step_adjustment {
    scaling_adjustment          = 25  # Add 25% capacity
    metric_interval_lower_bound = 0
    metric_interval_upper_bound = 20
  }

  step_adjustment {
    scaling_adjustment          = 50  # Add 50% capacity
    metric_interval_lower_bound = 20
    metric_interval_upper_bound = 40
  }

  step_adjustment {
    scaling_adjustment          = 100  # Double capacity
    metric_interval_lower_bound = 40
  }
}

# CloudWatch Alarm for Step Scaling
resource "aws_cloudwatch_metric_alarm" "high_cpu" {
  alarm_name          = "high-cpu"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 60
  statistic           = "Average"
  threshold           = 80

  dimensions = {
    AutoScalingGroupName = aws_autoscaling_group.app.name
  }

  alarm_actions = [aws_autoscaling_policy.scale_out.arn]
}

# Scheduled Scaling
resource "aws_autoscaling_schedule" "morning_scale_up" {
  scheduled_action_name  = "morning-scale-up"
  autoscaling_group_name = aws_autoscaling_group.app.name
  min_size               = 10
  max_size               = 50
  desired_capacity       = 20
  recurrence             = "0 8 * * MON-FRI"  # 8 AM weekdays
  time_zone              = "America/New_York"
}

resource "aws_autoscaling_schedule" "evening_scale_down" {
  scheduled_action_name  = "evening-scale-down"
  autoscaling_group_name = aws_autoscaling_group.app.name
  min_size               = 2
  max_size               = 20
  desired_capacity       = 4
  recurrence             = "0 20 * * MON-FRI"  # 8 PM weekdays
}

# Predictive Scaling
resource "aws_autoscaling_policy" "predictive" {
  name                   = "predictive-scaling"
  autoscaling_group_name = aws_autoscaling_group.app.name
  policy_type            = "PredictiveScaling"

  predictive_scaling_configuration {
    metric_specification {
      target_value = 65

      predefined_load_metric_specification {
        predefined_metric_type = "ASGTotalCPUUtilization"
      }

      predefined_scaling_metric_specification {
        predefined_metric_type = "ASGAverageCPUUtilization"
      }
    }

    mode                         = "ForecastAndScale"
    scheduling_buffer_time       = 300
    max_capacity_breach_behavior = "IncreaseMaxCapacity"
    max_capacity_buffer          = 10
  }
}
```

---

## 2. Scaling Policies

```yaml
# ════════════════════════════════════════════════════════════════
# SCALING POLICY TYPES
# ════════════════════════════════════════════════════════════════

# TARGET TRACKING SCALING
# ─────────────────────────
# Automatically adjusts to maintain metric at target value
# Like a thermostat - set temperature, system maintains it

Target: CPU 65%
Current: 80%  →  Scale OUT (add instances to reduce load)
Current: 50%  →  Scale IN (remove instances, over-provisioned)

# Pros:
# - Simple to configure
# - Self-adjusting
# - Handles both scale-out and scale-in

# Cons:
# - Less control over exact scaling behavior
# - May scale more aggressively than needed

# ═══════════════════════════════════════════════════════════════

# STEP SCALING
# ─────────────
# Different scaling actions based on metric severity

Step 1: CPU 70-80%  →  Add 2 instances
Step 2: CPU 80-90%  →  Add 4 instances
Step 3: CPU > 90%   →  Add 6 instances

# Pros:
# - Fine-grained control
# - Proportional response to severity

# Cons:
# - More complex to configure
# - Requires CloudWatch alarms

# ═══════════════════════════════════════════════════════════════

# SIMPLE SCALING
# ───────────────
# Single scaling action with cooldown

Alarm: CPU > 80%  →  Add 2 instances (then wait 300s cooldown)

# Pros:
# - Simplest to understand

# Cons:
# - Can't respond to rapid changes during cooldown
# - Use target tracking or step scaling instead

# ═══════════════════════════════════════════════════════════════

# SCHEDULED SCALING
# ─────────────────
# Pre-planned capacity changes

08:00 AM: min=10, desired=20 (morning traffic)
08:00 PM: min=2, desired=4 (low traffic)
Black Friday: min=100, max=500

# Pros:
# - Predictable workloads
# - No scaling delay

# Cons:
# - Requires known patterns
# - May over/under provision if pattern changes

# ═══════════════════════════════════════════════════════════════

# PREDICTIVE SCALING
# ──────────────────
# ML-based forecasting of future demand

Analyzes: Last 14 days of traffic patterns
Predicts: Tomorrow's traffic pattern
Action: Pre-scales before predicted spike

# Pros:
# - Handles recurring patterns automatically
# - Proactive, not reactive

# Cons:
# - Needs 24+ hours of data
# - May not handle unpredictable spikes
```

---

## 3. Kubernetes HPA/VPA

```yaml
# ════════════════════════════════════════════════════════════════
# KUBERNETES HORIZONTAL POD AUTOSCALER (HPA)
# ════════════════════════════════════════════════════════════════

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  
  minReplicas: 3
  maxReplicas: 50
  
  metrics:
    # CPU utilization
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    
    # Memory utilization
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
    
    # Custom metric: requests per second
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: 1000
    
    # External metric: SQS queue depth
    - type: External
      external:
        metric:
          name: sqs_queue_depth
          selector:
            matchLabels:
              queue: orders
        target:
          type: Value
          value: 100
  
  behavior:
    # Scale up behavior
    scaleUp:
      stabilizationWindowSeconds: 0  # Scale immediately
      policies:
        - type: Percent
          value: 100  # Double capacity
          periodSeconds: 15
        - type: Pods
          value: 4  # Or add 4 pods
          periodSeconds: 15
      selectPolicy: Max  # Use the larger scaling action
    
    # Scale down behavior
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5 min before scaling down
      policies:
        - type: Percent
          value: 10  # Remove 10% of pods
          periodSeconds: 60
      selectPolicy: Min  # Use smaller scaling action (conservative)

---
# ════════════════════════════════════════════════════════════════
# KUBERNETES VERTICAL POD AUTOSCALER (VPA)
# ════════════════════════════════════════════════════════════════

apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  
  updatePolicy:
    updateMode: "Auto"  # "Off" for recommendations only
  
  resourcePolicy:
    containerPolicies:
      - containerName: app
        minAllowed:
          cpu: 100m
          memory: 128Mi
        maxAllowed:
          cpu: 4
          memory: 8Gi
        controlledResources: ["cpu", "memory"]
        controlledValues: RequestsAndLimits

---
# ════════════════════════════════════════════════════════════════
# KEDA - KUBERNETES EVENT-DRIVEN AUTOSCALER
# ════════════════════════════════════════════════════════════════

apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: app-scaler
spec:
  scaleTargetRef:
    name: app
  
  minReplicaCount: 1
  maxReplicaCount: 100
  
  pollingInterval: 15
  cooldownPeriod: 300
  
  triggers:
    # AWS SQS
    - type: aws-sqs-queue
      metadata:
        queueURL: https://sqs.us-east-1.amazonaws.com/123456789/orders
        queueLength: "10"
        awsRegion: "us-east-1"
    
    # Prometheus metric
    - type: prometheus
      metadata:
        serverAddress: http://prometheus.monitoring:9090
        metricName: http_requests_total
        threshold: "100"
        query: sum(rate(http_requests_total{app="myapp"}[2m]))
    
    # Kafka
    - type: kafka
      metadata:
        bootstrapServers: kafka:9092
        consumerGroup: my-group
        topic: orders
        lagThreshold: "50"
```

---

## 4. Custom Metrics Scaling

```typescript
// ════════════════════════════════════════════════════════════════
// CUSTOM METRICS FOR AUTO SCALING
// ════════════════════════════════════════════════════════════════

import { CloudWatch } from '@aws-sdk/client-cloudwatch';

const cloudwatch = new CloudWatch({ region: 'us-east-1' });

// Publish custom metric
async function publishMetric(
  name: string,
  value: number,
  unit: string,
  dimensions: Record<string, string>
) {
  await cloudwatch.putMetricData({
    Namespace: 'MyApp/Performance',
    MetricData: [
      {
        MetricName: name,
        Value: value,
        Unit: unit,
        Timestamp: new Date(),
        Dimensions: Object.entries(dimensions).map(([name, value]) => ({
          Name: name,
          Value: value
        }))
      }
    ]
  });
}

// Example: Queue depth metric
async function publishQueueDepth() {
  const queueDepth = await getQueueMessageCount();
  await publishMetric(
    'QueueDepth',
    queueDepth,
    'Count',
    { QueueName: 'orders', Environment: 'production' }
  );
}

// Example: Active connections metric
async function publishConnectionCount() {
  const connections = getActiveWebSocketConnections();
  await publishMetric(
    'ActiveConnections',
    connections,
    'Count',
    { Service: 'websocket', Environment: 'production' }
  );
}

// Example: Business metric - orders per minute
async function publishOrderRate() {
  const ordersPerMinute = await getOrderRate();
  await publishMetric(
    'OrdersPerMinute',
    ordersPerMinute,
    'Count/Second',
    { Service: 'api', Environment: 'production' }
  );
}

// Run metrics publisher
setInterval(async () => {
  await Promise.all([
    publishQueueDepth(),
    publishConnectionCount(),
    publishOrderRate()
  ]);
}, 60000); // Every minute
```

```hcl
# Custom metric scaling policy
resource "aws_autoscaling_policy" "queue_depth" {
  name                   = "queue-depth-scaling"
  autoscaling_group_name = aws_autoscaling_group.workers.name
  policy_type            = "TargetTrackingScaling"

  target_tracking_configuration {
    customized_metric_specification {
      metric_dimension {
        name  = "QueueName"
        value = "orders"
      }
      metric_name = "QueueDepth"
      namespace   = "MyApp/Performance"
      statistic   = "Average"
    }
    target_value = 10  # Keep ~10 messages per worker
  }
}
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# AUTO SCALING BEST PRACTICES
# ════════════════════════════════════════════════════════════════

# 1. Use multiple metrics
metrics:
  - CPU utilization (baseline)
  - Request count per target (traffic)
  - Queue depth (backlog)
  - Custom business metrics

# 2. Set appropriate thresholds
cpu_target: 65%  # Leave 35% headroom for spikes
# Not 90%! By the time scaling kicks in, you're already degraded

# 3. Scale out fast, scale in slow
scale_out:
  - React quickly to demand increases
  - Double capacity if needed
  - Short cooldown (60-120s)

scale_in:
  - Conservative removal
  - Remove 10-20% at a time
  - Long cooldown (300-600s)
  - Stabilization window (5-10 min)

# 4. Use warm pools / pre-scaling
warm_pool:
  min_size: 5  # Pre-initialized instances ready
  state: Stopped  # Or Hibernated for faster start

predictive_scaling:
  enabled: true  # Pre-scale based on patterns

# 5. Set min/max appropriately
min_size: 2      # Always have HA (multi-AZ)
max_size: 100    # Cost protection
# Don't set min=1 in production!

# 6. Health checks
health_check_type: ELB  # Not EC2 only
grace_period: 300       # Time for app startup
# Match your app's actual startup time

# 7. Instance diversity
mixed_instances:
  instance_types:
    - t3.medium
    - t3a.medium
    - t3.large
  spot_percentage: 70   # Cost savings
  on_demand_base: 2     # Guaranteed capacity

# 8. Multi-AZ deployment
availability_zones:
  - us-east-1a
  - us-east-1b
  - us-east-1c
# Spread instances across AZs
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# AUTO SCALING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Scaling threshold too high
# Bad
cpu_target: 90%
# Problem: By 90% CPU, latency is already degraded

# Good
cpu_target: 65%
# Leaves headroom for traffic spikes

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: No cooldown between actions
# Bad
cooldown: 0
# Problem: Rapid scale out/in (thrashing)

# Good
scale_out_cooldown: 120   # 2 minutes
scale_in_cooldown: 300    # 5 minutes

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Health check grace period too short
# Bad
health_check_grace_period: 60
# App takes 3 minutes to start, marked unhealthy immediately

# Good
health_check_grace_period: 300  # Match actual startup time

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Single metric scaling
# Bad - CPU only
policy:
  - cpu_utilization: 70%
# Misses memory pressure, queue buildup

# Good - Multiple metrics
policies:
  - cpu_utilization: 70%
  - memory_utilization: 80%
  - alb_request_count: 1000

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: min_size = 1
# Bad
min_size: 1
# Single point of failure, no HA

# Good
min_size: 2
# Spread across AZs for high availability

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Ignoring startup time in scaling
# Bad
# Instance launches, immediately receives traffic, fails

# Good
# Use ALB with slow start: 30 seconds
slow_start:
  duration: 30  # Gradually increase traffic to new instances

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 7: Not using scaling for deployments
# Bad
# Manual instance replacement during deployment

# Good
instance_refresh:
  strategy: Rolling
  min_healthy: 75%
  instance_warmup: 120
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is auto scaling?"**
> "Automatically adjusting compute capacity based on demand. Scale out (add instances) when load increases, scale in (remove) when it decreases. Ensures availability during peaks and cost optimization during low usage."

**Q: "Horizontal vs vertical scaling?"**
> "Horizontal: Add more instances (scale out/in). More complex but highly scalable, no downtime, requires stateless design. Vertical: Larger instance (scale up/down). Simpler but has limits, may require downtime. Horizontal preferred for cloud-native apps."

**Q: "What metrics trigger auto scaling?"**
> "CPU utilization (most common), memory, network I/O, ALB request count per target, queue depth (SQS), custom metrics (latency, business KPIs). Best practice: use multiple metrics for complete picture."

### Intermediate Questions

**Q: "What scaling policies does AWS offer?"**
> "Target tracking: Maintain metric at value (like thermostat). Step scaling: Different actions based on severity. Scheduled: Pre-planned capacity. Predictive: ML-based forecasting. Simple scaling: Single action with cooldown (deprecated in favor of step)."

**Q: "How do you handle scaling with stateful applications?"**
> "Externalize state (Redis, database). Use sticky sessions at LB if needed. Scale-in protection for processing instances. Graceful shutdown (finish work before termination). For stateful sets in K8s: careful scaling, ordered termination."

**Q: "What is a warm pool?"**
> "Pre-initialized instances kept ready for rapid scaling. Instances are launched and configured but stopped (or hibernated). When scaling needed, they start faster than fresh instances. Reduces time-to-serve from minutes to seconds."

### Advanced Questions

**Q: "How do you prevent scaling thrashing?"**
> "Set appropriate cooldown periods (5+ minutes for scale-in). Use stabilization windows in K8s HPA. Scale in slowly (10% at a time). Set buffer between scale-out and scale-in thresholds. Use predictive scaling to smooth capacity changes."

**Q: "How do you design auto scaling for cost optimization?"**
> "Use spot instances for non-critical workloads (70% spot). Reserved capacity for baseline, auto scale on-demand. Schedule scale-down during known low periods. Set max capacity limits. Use predictive scaling to avoid over-provisioning. Review and rightsize instance types."

**Q: "How do you scale based on queue depth?"**
> "Monitor queue length as custom metric. Target: messages per worker (e.g., 10 messages per instance). Scale out when queue grows. Consider message processing time. For SQS: Use ApproximateNumberOfMessages. For K8s: KEDA ScaledObject with queue trigger. Account for processing lag in thresholds."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  AUTO SCALING CHEAT SHEET                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SCALING TYPES:                                                 │
│  • Target Tracking - Maintain metric at target                 │
│  • Step Scaling    - Actions based on severity                 │
│  • Scheduled       - Pre-planned capacity changes              │
│  • Predictive      - ML-based forecasting                      │
│                                                                 │
│  KEY METRICS:                                                   │
│  • CPU Utilization        - Target 60-70%                      │
│  • ALB Request Count      - Requests per target                │
│  • Queue Depth            - Messages per worker                │
│  • Custom Metrics         - Business-specific                  │
│                                                                 │
│  RECOMMENDED SETTINGS:                                          │
│  • Min instances: 2+ (HA)                                      │
│  • CPU target: 65%                                             │
│  • Scale-out cooldown: 120s                                    │
│  • Scale-in cooldown: 300s                                     │
│  • Health check grace: 300s                                    │
│                                                                 │
│  SCALE BEHAVIOR:                                                │
│  • Scale OUT: Fast, aggressive (double capacity)               │
│  • Scale IN: Slow, conservative (10% reduction)                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

AWS ASG CLI:
┌────────────────────────────────────────────────────────────────┐
│ aws autoscaling describe-auto-scaling-groups                   │
│ aws autoscaling set-desired-capacity --desired-capacity 10     │
│ aws autoscaling update-auto-scaling-group --min-size 5         │
│ aws autoscaling start-instance-refresh                         │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

