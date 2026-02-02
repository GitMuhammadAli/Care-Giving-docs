# 💰 Cost Optimization - Complete Guide

> A comprehensive guide to cloud cost optimization - reserved instances, spot instances, rightsizing, FinOps, and cost management strategies.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Cloud cost optimization is the continuous practice of reducing cloud spend while maintaining performance and availability, through rightsizing resources, leveraging pricing models (reserved, spot), eliminating waste, and implementing FinOps practices for cost accountability."

### The 7 Key Concepts (Remember These!)
```
1. RIGHTSIZING       → Match resource size to actual usage
2. RESERVED          → Commit for 1-3 years for 30-72% savings
3. SPOT/PREEMPTIBLE  → Use spare capacity for 60-90% savings
4. SAVINGS PLANS     → Flexible commitment-based discounts
5. TAGGING           → Track costs by team/project/environment
6. WASTE ELIMINATION → Remove unused/idle resources
7. FINOPS            → Financial operations for cloud
```

### Cost Optimization Hierarchy
```
┌─────────────────────────────────────────────────────────────────┐
│                COST OPTIMIZATION HIERARCHY                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. ELIMINATE WASTE (First priority)                           │
│     ───────────────────────────────                             │
│     • Delete unused resources                                  │
│     • Remove idle instances                                    │
│     • Clean up old snapshots/backups                           │
│     • Savings: 10-30%                                          │
│                                                                 │
│  2. RIGHTSIZE RESOURCES                                        │
│     ────────────────────                                        │
│     • Match instance type to workload                          │
│     • Use appropriate storage tiers                            │
│     • Optimize database instances                              │
│     • Savings: 20-40%                                          │
│                                                                 │
│  3. USE PRICING MODELS                                         │
│     ───────────────────                                         │
│     • Reserved Instances for steady state                      │
│     • Spot Instances for fault-tolerant                        │
│     • Savings Plans for flexibility                            │
│     • Savings: 30-70%                                          │
│                                                                 │
│  4. ARCHITECTURAL OPTIMIZATION                                 │
│     ───────────────────────────                                 │
│     • Serverless where appropriate                             │
│     • Auto-scaling                                             │
│     • Multi-tier storage                                       │
│     • Savings: 20-50%                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### AWS Pricing Models
```
┌─────────────────────────────────────────────────────────────────┐
│                    AWS PRICING MODELS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ON-DEMAND                                                     │
│  ─────────                                                      │
│  • Pay per hour/second                                         │
│  • No commitment                                               │
│  • Full price                                                  │
│  • Good for: variable, unpredictable workloads                 │
│                                                                 │
│  RESERVED INSTANCES                                            │
│  ──────────────────                                             │
│  • 1 or 3 year commitment                                      │
│  • 30-72% savings vs on-demand                                 │
│  • Specific instance type/region                               │
│  • Good for: steady-state, predictable workloads               │
│                                                                 │
│  SAVINGS PLANS                                                  │
│  ─────────────                                                  │
│  • $/hour commitment for 1-3 years                             │
│  • 30-72% savings                                              │
│  • More flexible than RIs                                      │
│  • Good for: organizations with diverse workloads              │
│                                                                 │
│  SPOT INSTANCES                                                 │
│  ──────────────                                                 │
│  • Spare AWS capacity                                          │
│  • 60-90% savings                                              │
│  • Can be interrupted with 2-min warning                       │
│  • Good for: batch, stateless, fault-tolerant                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"FinOps"** | "We practice FinOps with cost allocation by team" |
| **"Rightsizing"** | "Rightsizing recommendations saved us 30% on compute" |
| **"Spot fleet"** | "We use spot fleet with multiple instance types for resilience" |
| **"Unit economics"** | "We track unit economics - cost per transaction" |
| **"Showback/chargeback"** | "Chargeback makes teams accountable for their spend" |
| **"Committed use discount"** | "3-year committed use gives us 57% discount" |

### Key Numbers to Remember
| Pricing Model | Savings | Commitment |
|---------------|---------|------------|
| Reserved (1yr, no upfront) | **~30%** | Flexibility |
| Reserved (3yr, all upfront) | **~72%** | Maximum savings |
| Savings Plans | **~30-72%** | Flexible |
| Spot Instances | **~60-90%** | Can be interrupted |
| S3 Intelligent-Tiering | **~40%** | Automatic |

### The "Wow" Statement (Memorize This!)
> "We reduced cloud spend by 45% through a comprehensive FinOps program. First, we implemented mandatory tagging for cost allocation by team, project, and environment. We identified and eliminated 20% waste - unused EBS volumes, idle load balancers, oversized dev instances. Rightsizing recommendations from AWS Cost Explorer moved us to better-fit instance types. For steady-state production, we purchased 3-year Compute Savings Plans covering 70% of baseline. Stateless workloads (batch processing, CI runners) run on spot instances with 80% savings. Auto-scaling ensures we don't over-provision for peak. Weekly cost reviews with engineering leads created accountability. Monthly showback reports to each team. Cost per transaction decreased 40% while traffic grew 60%."

---

## 📚 Table of Contents

1. [Cost Visibility](#1-cost-visibility)
2. [Reserved & Savings Plans](#2-reserved--savings-plans)
3. [Spot Instances](#3-spot-instances)
4. [Rightsizing](#4-rightsizing)
5. [Storage Optimization](#5-storage-optimization)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Cost Visibility

```hcl
# ════════════════════════════════════════════════════════════════
# AWS COST ALLOCATION TAGS - TERRAFORM
# ════════════════════════════════════════════════════════════════

# Provider-level default tags
provider "aws" {
  region = "us-east-1"
  
  default_tags {
    tags = {
      Environment = var.environment
      Team        = var.team
      Project     = var.project
      CostCenter  = var.cost_center
      ManagedBy   = "terraform"
    }
  }
}

# Enable cost allocation tags
resource "aws_ce_cost_allocation_tag" "tags" {
  for_each = toset(["Environment", "Team", "Project", "CostCenter"])
  
  tag_key = each.value
  status  = "Active"
}

# Budget with alerts
resource "aws_budgets_budget" "monthly" {
  name              = "monthly-budget"
  budget_type       = "COST"
  limit_amount      = "10000"
  limit_unit        = "USD"
  time_unit         = "MONTHLY"
  time_period_start = "2024-01-01_00:00"

  cost_filter {
    name = "TagKeyValue"
    values = [
      "user:Environment$production"
    ]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 80
    threshold_type            = "PERCENTAGE"
    notification_type         = "FORECASTED"
    subscriber_email_addresses = ["finance@example.com"]
    subscriber_sns_topic_arns = [aws_sns_topic.budget_alerts.arn]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 100
    threshold_type            = "PERCENTAGE"
    notification_type         = "ACTUAL"
    subscriber_email_addresses = ["finance@example.com", "engineering@example.com"]
  }
}

# Cost anomaly detection
resource "aws_ce_anomaly_monitor" "service" {
  name              = "service-anomaly-monitor"
  monitor_type      = "DIMENSIONAL"
  monitor_dimension = "SERVICE"
}

resource "aws_ce_anomaly_subscription" "alerts" {
  name      = "anomaly-alerts"
  frequency = "IMMEDIATE"

  monitor_arn_list = [
    aws_ce_anomaly_monitor.service.arn
  ]

  subscriber {
    type    = "EMAIL"
    address = "finance@example.com"
  }

  threshold_expression {
    dimension {
      key           = "ANOMALY_TOTAL_IMPACT_ABSOLUTE"
      values        = ["100"]  # Alert if anomaly > $100
      match_options = ["GREATER_THAN_OR_EQUAL"]
    }
  }
}
```

```python
# ════════════════════════════════════════════════════════════════
# COST REPORT SCRIPT
# ════════════════════════════════════════════════════════════════

import boto3
from datetime import datetime, timedelta

def generate_cost_report():
    ce = boto3.client('ce')
    
    end_date = datetime.today().strftime('%Y-%m-%d')
    start_date = (datetime.today() - timedelta(days=30)).strftime('%Y-%m-%d')
    
    # Cost by service
    response = ce.get_cost_and_usage(
        TimePeriod={
            'Start': start_date,
            'End': end_date
        },
        Granularity='MONTHLY',
        Metrics=['UnblendedCost'],
        GroupBy=[
            {'Type': 'DIMENSION', 'Key': 'SERVICE'}
        ]
    )
    
    print("Cost by Service (Last 30 Days)")
    print("="*50)
    for group in sorted(
        response['ResultsByTime'][0]['Groups'],
        key=lambda x: float(x['Metrics']['UnblendedCost']['Amount']),
        reverse=True
    )[:10]:
        service = group['Keys'][0]
        cost = float(group['Metrics']['UnblendedCost']['Amount'])
        print(f"{service}: ${cost:,.2f}")
    
    # Cost by team (using tags)
    response = ce.get_cost_and_usage(
        TimePeriod={
            'Start': start_date,
            'End': end_date
        },
        Granularity='MONTHLY',
        Metrics=['UnblendedCost'],
        GroupBy=[
            {'Type': 'TAG', 'Key': 'Team'}
        ]
    )
    
    print("\nCost by Team (Last 30 Days)")
    print("="*50)
    for group in response['ResultsByTime'][0]['Groups']:
        team = group['Keys'][0] or 'Untagged'
        cost = float(group['Metrics']['UnblendedCost']['Amount'])
        print(f"{team}: ${cost:,.2f}")

    # Rightsizing recommendations
    ce_rightsizing = boto3.client('ce')
    response = ce_rightsizing.get_rightsizing_recommendation(
        Service='AmazonEC2',
        Configuration={
            'RecommendationTarget': 'SAME_INSTANCE_FAMILY',
            'BenefitsConsidered': True
        }
    )
    
    print("\nRightsizing Recommendations")
    print("="*50)
    total_savings = 0
    for rec in response.get('RightsizingRecommendations', [])[:10]:
        current = rec['CurrentInstance']
        if rec['RightsizingType'] == 'Terminate':
            savings = float(rec['TerminateRecommendationDetail']['EstimatedMonthlySavings'])
            print(f"Terminate {current['InstanceId']}: Save ${savings:.2f}/month")
        else:
            modify = rec['ModifyRecommendationDetail']
            savings = float(modify['EstimatedMonthlySavings'])
            print(f"Resize {current['InstanceId']}: {current['InstanceType']} -> {modify['TargetInstances'][0]['InstanceType']}: Save ${savings:.2f}/month")
        total_savings += savings
    
    print(f"\nTotal potential monthly savings: ${total_savings:,.2f}")
```

---

## 2. Reserved & Savings Plans

```hcl
# ════════════════════════════════════════════════════════════════
# SAVINGS PLANS STRATEGY - TERRAFORM
# ════════════════════════════════════════════════════════════════

# Note: Savings Plans are purchased via AWS Console or CLI
# This documents the strategy

/*
SAVINGS PLANS STRATEGY:

1. Compute Savings Plans (Recommended)
   - Applies to EC2, Fargate, Lambda
   - Flexible across instance families, sizes, OS, tenancy
   - 1-year: ~30% savings
   - 3-year: ~50% savings (no upfront) to 66% (all upfront)

2. EC2 Instance Savings Plans
   - Specific to instance family in a region
   - Slightly higher discount than Compute SP
   - Less flexible

COVERAGE STRATEGY:
- Cover 70% of steady-state with Savings Plans
- Remaining 30% = buffer for:
  - Growth
  - Spot instances
  - Variable workloads
*/

# Calculate recommended commitment
locals {
  # Average monthly on-demand spend (last 3 months)
  average_monthly_spend = 50000  # $50,000
  
  # Target coverage (70% of baseline)
  target_coverage = 0.70
  
  # Recommended hourly commitment
  recommended_commitment = (local.average_monthly_spend * local.target_coverage) / 730
  # = $47.95/hour
}

# Reserved Instance for RDS (still uses RIs, not Savings Plans)
resource "aws_db_instance" "production" {
  # ... instance config ...
  
  # Tag for RI matching
  tags = {
    ReservedInstance = "true"
  }
}
```

```bash
# ════════════════════════════════════════════════════════════════
# SAVINGS PLANS ANALYSIS
# ════════════════════════════════════════════════════════════════

# Get Savings Plans recommendations
aws ce get-savings-plans-purchase-recommendation \
  --savings-plans-type COMPUTE_SP \
  --term-in-years ONE_YEAR \
  --payment-option NO_UPFRONT \
  --lookback-period-in-days THIRTY_DAYS

# View current Savings Plans utilization
aws ce get-savings-plans-utilization \
  --time-period Start=2024-01-01,End=2024-01-31

# View coverage
aws ce get-savings-plans-coverage \
  --time-period Start=2024-01-01,End=2024-01-31
```

---

## 3. Spot Instances

```hcl
# ════════════════════════════════════════════════════════════════
# SPOT INSTANCES - AUTO SCALING GROUP
# ════════════════════════════════════════════════════════════════

resource "aws_autoscaling_group" "spot_workers" {
  name                = "spot-workers"
  desired_capacity    = 10
  min_size            = 5
  max_size            = 50
  vpc_zone_identifier = var.private_subnet_ids

  # Mixed instances policy
  mixed_instances_policy {
    instances_distribution {
      # Base on-demand capacity for stability
      on_demand_base_capacity = 2
      
      # 80% spot above base
      on_demand_percentage_above_base_capacity = 20
      
      # Capacity-optimized = fewer interruptions
      spot_allocation_strategy = "capacity-optimized"
      
      # Max price (optional, defaults to on-demand price)
      # spot_max_price = "0.05"
    }

    launch_template {
      launch_template_specification {
        launch_template_id = aws_launch_template.worker.id
        version            = "$Latest"
      }

      # Multiple instance types for better availability
      override {
        instance_type     = "m5.large"
        weighted_capacity = "1"
      }
      override {
        instance_type     = "m5a.large"
        weighted_capacity = "1"
      }
      override {
        instance_type     = "m5n.large"
        weighted_capacity = "1"
      }
      override {
        instance_type     = "m4.large"
        weighted_capacity = "1"
      }
      override {
        instance_type     = "c5.large"  # Similar performance
        weighted_capacity = "1"
      }
    }
  }

  # Capacity rebalancing (proactive replacement)
  capacity_rebalance = true

  # Instance refresh for updates
  instance_refresh {
    strategy = "Rolling"
    preferences {
      min_healthy_percentage = 80
    }
  }

  tag {
    key                 = "Name"
    value               = "spot-worker"
    propagate_at_launch = true
  }
}

# Launch template with spot interruption handling
resource "aws_launch_template" "worker" {
  name_prefix   = "spot-worker-"
  image_id      = data.aws_ami.amazon_linux.id
  instance_type = "m5.large"

  user_data = base64encode(<<-EOF
    #!/bin/bash
    
    # Handle spot interruption
    cat > /etc/systemd/system/spot-interruption-handler.service << 'HANDLER'
    [Unit]
    Description=Spot Interruption Handler
    After=network.target

    [Service]
    Type=simple
    ExecStart=/usr/local/bin/spot-interruption-handler.sh
    Restart=always

    [Install]
    WantedBy=multi-user.target
    HANDLER

    cat > /usr/local/bin/spot-interruption-handler.sh << 'SCRIPT'
    #!/bin/bash
    while true; do
      # Check for interruption notice
      RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" http://169.254.169.254/latest/meta-data/spot/termination-time)
      if [ "$RESPONSE" == "200" ]; then
        echo "Spot interruption notice received!"
        # Graceful shutdown
        systemctl stop my-application
        # Deregister from load balancer
        aws elbv2 deregister-targets ...
        # Drain connections
        sleep 30
        exit 0
      fi
      sleep 5
    done
    SCRIPT
    
    chmod +x /usr/local/bin/spot-interruption-handler.sh
    systemctl enable spot-interruption-handler
    systemctl start spot-interruption-handler
  EOF
  )

  tag_specifications {
    resource_type = "instance"
    tags = {
      SpotInstance = "true"
    }
  }
}
```

```yaml
# ════════════════════════════════════════════════════════════════
# KUBERNETES SPOT INSTANCES (Karpenter)
# ════════════════════════════════════════════════════════════════

apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: spot-provisioner
spec:
  requirements:
    - key: karpenter.sh/capacity-type
      operator: In
      values: ["spot", "on-demand"]
    - key: node.kubernetes.io/instance-type
      operator: In
      values:
        - m5.large
        - m5.xlarge
        - m5a.large
        - m5a.xlarge
        - m5n.large
        - c5.large
        - c5.xlarge
    - key: topology.kubernetes.io/zone
      operator: In
      values:
        - us-east-1a
        - us-east-1b
        - us-east-1c
  
  # Prefer spot, fall back to on-demand
  providerRef:
    name: default
  
  # Consolidation - remove underutilized nodes
  consolidation:
    enabled: true
  
  # Limits
  limits:
    resources:
      cpu: 1000
      memory: 1000Gi
  
  # TTL
  ttlSecondsAfterEmpty: 30
  ttlSecondsUntilExpired: 2592000  # 30 days
```

---

## 4. Rightsizing

```python
# ════════════════════════════════════════════════════════════════
# RIGHTSIZING ANALYSIS SCRIPT
# ════════════════════════════════════════════════════════════════

import boto3
from datetime import datetime, timedelta

def analyze_ec2_utilization():
    ec2 = boto3.client('ec2')
    cloudwatch = boto3.client('cloudwatch')
    
    # Get all running instances
    instances = ec2.describe_instances(
        Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
    )
    
    recommendations = []
    
    for reservation in instances['Reservations']:
        for instance in reservation['Instances']:
            instance_id = instance['InstanceId']
            instance_type = instance['InstanceType']
            
            # Get CPU utilization (last 14 days)
            cpu_stats = cloudwatch.get_metric_statistics(
                Namespace='AWS/EC2',
                MetricName='CPUUtilization',
                Dimensions=[{'Name': 'InstanceId', 'Value': instance_id}],
                StartTime=datetime.utcnow() - timedelta(days=14),
                EndTime=datetime.utcnow(),
                Period=3600,
                Statistics=['Average', 'Maximum']
            )
            
            if cpu_stats['Datapoints']:
                avg_cpu = sum(d['Average'] for d in cpu_stats['Datapoints']) / len(cpu_stats['Datapoints'])
                max_cpu = max(d['Maximum'] for d in cpu_stats['Datapoints'])
                
                # Rightsizing recommendations
                if max_cpu < 20:
                    recommendations.append({
                        'instance_id': instance_id,
                        'instance_type': instance_type,
                        'avg_cpu': avg_cpu,
                        'max_cpu': max_cpu,
                        'recommendation': 'DOWNSIZE (max CPU < 20%)',
                        'suggested_action': 'Reduce instance size by 50%'
                    })
                elif max_cpu < 40:
                    recommendations.append({
                        'instance_id': instance_id,
                        'instance_type': instance_type,
                        'avg_cpu': avg_cpu,
                        'max_cpu': max_cpu,
                        'recommendation': 'CONSIDER DOWNSIZE (max CPU < 40%)',
                        'suggested_action': 'Review and potentially reduce size'
                    })
                elif avg_cpu < 10:
                    recommendations.append({
                        'instance_id': instance_id,
                        'instance_type': instance_type,
                        'avg_cpu': avg_cpu,
                        'max_cpu': max_cpu,
                        'recommendation': 'LOW UTILIZATION (avg CPU < 10%)',
                        'suggested_action': 'Review if instance is needed'
                    })
    
    return recommendations

def find_idle_resources():
    """Find unused resources"""
    ec2 = boto3.client('ec2')
    elb = boto3.client('elbv2')
    
    idle_resources = []
    
    # Unattached EBS volumes
    volumes = ec2.describe_volumes(
        Filters=[{'Name': 'status', 'Values': ['available']}]
    )
    for vol in volumes['Volumes']:
        idle_resources.append({
            'type': 'EBS Volume',
            'id': vol['VolumeId'],
            'size': f"{vol['Size']} GB",
            'monthly_cost': vol['Size'] * 0.10,  # Approximate gp2 cost
            'recommendation': 'Delete or snapshot if needed'
        })
    
    # Unattached Elastic IPs
    addresses = ec2.describe_addresses()
    for addr in addresses['Addresses']:
        if 'InstanceId' not in addr and 'NetworkInterfaceId' not in addr:
            idle_resources.append({
                'type': 'Elastic IP',
                'id': addr['AllocationId'],
                'monthly_cost': 3.60,  # ~$0.005/hour
                'recommendation': 'Release if not needed'
            })
    
    # Load balancers with no targets
    lbs = elb.describe_load_balancers()
    for lb in lbs['LoadBalancers']:
        target_groups = elb.describe_target_groups(
            LoadBalancerArn=lb['LoadBalancerArn']
        )
        
        has_targets = False
        for tg in target_groups['TargetGroups']:
            health = elb.describe_target_health(TargetGroupArn=tg['TargetGroupArn'])
            if health['TargetHealthDescriptions']:
                has_targets = True
                break
        
        if not has_targets:
            idle_resources.append({
                'type': 'Load Balancer',
                'id': lb['LoadBalancerName'],
                'monthly_cost': 20,  # Approximate ALB cost
                'recommendation': 'Delete if not in use'
            })
    
    return idle_resources

# Run analysis
if __name__ == "__main__":
    print("Rightsizing Recommendations")
    print("="*60)
    for rec in analyze_ec2_utilization():
        print(f"{rec['instance_id']} ({rec['instance_type']})")
        print(f"  Avg CPU: {rec['avg_cpu']:.1f}%, Max CPU: {rec['max_cpu']:.1f}%")
        print(f"  {rec['recommendation']}")
        print()
    
    print("\nIdle Resources")
    print("="*60)
    total_waste = 0
    for resource in find_idle_resources():
        print(f"{resource['type']}: {resource['id']}")
        print(f"  Monthly cost: ${resource['monthly_cost']:.2f}")
        print(f"  {resource['recommendation']}")
        total_waste += resource['monthly_cost']
        print()
    
    print(f"Total monthly waste: ${total_waste:.2f}")
```

---

## 5. Storage Optimization

```hcl
# ════════════════════════════════════════════════════════════════
# S3 LIFECYCLE POLICIES
# ════════════════════════════════════════════════════════════════

resource "aws_s3_bucket_lifecycle_configuration" "optimized" {
  bucket = aws_s3_bucket.data.id

  # Transition infrequently accessed data
  rule {
    id     = "transition-to-ia"
    status = "Enabled"

    filter {
      prefix = "logs/"
    }

    transition {
      days          = 30
      storage_class = "STANDARD_IA"  # 45% cheaper
    }

    transition {
      days          = 90
      storage_class = "GLACIER_IR"   # 68% cheaper
    }

    transition {
      days          = 180
      storage_class = "GLACIER_DEEP_ARCHIVE"  # 95% cheaper
    }

    expiration {
      days = 365
    }
  }

  # Intelligent tiering for unpredictable access
  rule {
    id     = "intelligent-tiering"
    status = "Enabled"

    filter {
      prefix = "data/"
    }

    transition {
      days          = 0
      storage_class = "INTELLIGENT_TIERING"
    }
  }

  # Delete incomplete multipart uploads
  rule {
    id     = "abort-incomplete-uploads"
    status = "Enabled"

    filter {}

    abort_incomplete_multipart_upload {
      days_after_initiation = 7
    }
  }

  # Delete old versions
  rule {
    id     = "delete-old-versions"
    status = "Enabled"

    filter {}

    noncurrent_version_expiration {
      noncurrent_days = 30
    }
  }
}

# S3 Intelligent-Tiering configuration
resource "aws_s3_bucket_intelligent_tiering_configuration" "config" {
  bucket = aws_s3_bucket.data.id
  name   = "EntireBucket"

  tiering {
    access_tier = "DEEP_ARCHIVE_ACCESS"
    days        = 180
  }

  tiering {
    access_tier = "ARCHIVE_ACCESS"
    days        = 90
  }
}

# ════════════════════════════════════════════════════════════════
# EBS OPTIMIZATION
# ════════════════════════════════════════════════════════════════

# Use gp3 instead of gp2 (20% cheaper, better performance)
resource "aws_ebs_volume" "optimized" {
  availability_zone = "us-east-1a"
  size              = 100
  type              = "gp3"  # Not gp2
  iops              = 3000   # gp3 baseline
  throughput        = 125    # gp3 baseline

  tags = {
    Name = "optimized-volume"
  }
}
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# COST OPTIMIZATION PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Over-committing to Reserved Instances
# Bad
reserved_coverage: 100%
# No room for growth, stuck if requirements change

# Good
reserved_coverage: 70%
# Buffer for growth, variable workloads, spot

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Not using Savings Plans flexibility
# Bad
# Buy Reserved Instances for EC2
# Then migrate to Fargate - RIs wasted!

# Good
# Buy Compute Savings Plans
# Applies to EC2, Fargate, Lambda
# Flexibility for architectural changes

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Ignoring data transfer costs
# Bad
architecture:
  - Backend in us-east-1
  - Database in us-west-2
  - S3 in eu-west-1
# $$$$ data transfer!

# Good
# Co-locate resources in same region/AZ
# Use VPC endpoints
# Cache at edge (CloudFront)

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Dev/test environments same as production
# Bad
dev_environment:
  instance_type: m5.4xlarge  # Same as prod!
  count: 10

# Good
dev_environment:
  instance_type: t3.medium  # Smaller
  count: 2
  schedule: "Stop outside business hours"

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: No resource tagging
# Bad
# "Who owns this? Which project? Can we delete it?"
# Unknown, so nothing gets cleaned up

# Good
required_tags:
  - Environment
  - Team
  - Project
  - CostCenter
# Weekly untagged resource report
# Auto-terminate untagged resources after 7 days

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Optimizing too early
# Bad
# Spend 2 weeks optimizing $100/month service
# 40 hours * $100/hour = $4000 engineering cost
# Payback: 40 months!

# Good
# Focus on top 5 cost drivers first
# $10K/month service: 10% savings = $1K/month
# Payback: immediate
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What are the AWS pricing models?"**
> "On-Demand: Pay as you go, full price. Reserved: 1-3 year commitment, 30-72% savings. Savings Plans: Hourly commitment, flexible across services, similar discounts. Spot: Spare capacity, 60-90% savings, can be interrupted."

**Q: "What is FinOps?"**
> "Financial operations for cloud. Combines finance, engineering, and business to optimize cloud spending. Key practices: cost visibility via tagging, accountability via showback/chargeback, rightsizing, commitment optimization, waste elimination. Cultural shift to make cost a first-class metric."

**Q: "How do you reduce waste?"**
> "Identify idle resources: unattached volumes, unused load balancers, stopped instances with attached storage. Schedule dev/test environments to stop nights/weekends. Delete old snapshots, unused AMIs. Tag everything, report on untagged resources."

### Intermediate Questions

**Q: "Savings Plans vs Reserved Instances?"**
> "Savings Plans: Flexible commitment ($/hour), applies across instance families and services (EC2, Fargate, Lambda). Reserved Instances: Specific instance type and region, slightly higher discount for EC2, still needed for RDS/ElastiCache. Recommendation: Savings Plans for compute, RIs for databases."

**Q: "How do you use Spot Instances effectively?"**
> "Use for fault-tolerant, stateless workloads. Diversify across multiple instance types and AZs for availability. Handle interruption gracefully (2-minute warning). Use capacity-optimized allocation. Mix with on-demand for stability (e.g., 20% on-demand base). Good for: batch processing, CI/CD runners, dev/test, stateless web tier."

**Q: "How do you implement cost allocation by team?"**
> "Mandatory tagging policy with Team, Project, CostCenter tags. Tag enforcement via AWS Config rules. Cost Explorer filtered by tags. Weekly/monthly cost reports per team. Showback (visibility) or chargeback (actual charge) to budget. Make teams accountable for their spend."

### Advanced Questions

**Q: "Design a cost optimization strategy for a $1M/month cloud spend."**
> "1) Visibility: Implement tagging, set up Cost Explorer, create dashboards. 2) Quick wins: Eliminate obvious waste (30-day unattached volumes, idle instances). 3) Rightsizing: Review utilization, downsize oversized instances. 4) Commitment: Analyze steady-state, purchase Savings Plans for 70% coverage. 5) Spot: Move fault-tolerant workloads to spot. 6) Architecture: Review data transfer costs, storage tiers. 7) Governance: Weekly cost reviews, budget alerts, anomaly detection. Target: 30-40% reduction."

**Q: "How do you balance cost optimization with reliability?"**
> "Don't optimize to the point of fragility. Keep buffer capacity for resilience. Use committed pricing for baseline, on-demand for buffer, spot for elasticity. Test thoroughly before rightsizing production. Automate scale-down but have fast scale-up. Monitor performance alongside cost. Cost per transaction is better metric than total cost."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                 COST OPTIMIZATION CHECKLIST                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  VISIBILITY:                                                    │
│  □ Tagging policy implemented                                  │
│  □ Cost Explorer dashboards created                            │
│  □ Budget alerts configured                                    │
│  □ Anomaly detection enabled                                   │
│                                                                 │
│  WASTE ELIMINATION:                                             │
│  □ Unused resources identified                                 │
│  □ Dev/test schedules implemented                              │
│  □ Old snapshots cleaned up                                    │
│  □ Unattached volumes deleted                                  │
│                                                                 │
│  RIGHTSIZING:                                                   │
│  □ Utilization reviewed                                        │
│  □ Oversized instances downsized                               │
│  □ gp2 → gp3 migration                                         │
│                                                                 │
│  PRICING OPTIMIZATION:                                          │
│  □ Savings Plans purchased (70% coverage)                      │
│  □ Spot instances for fault-tolerant                           │
│  □ Storage lifecycle policies                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

SAVINGS BY MODEL:
┌────────────────────────────────────────────────────────────────┐
│ Reserved (3yr, all upfront)  - Up to 72% savings               │
│ Savings Plans (3yr)          - Up to 66% savings               │
│ Spot Instances               - Up to 90% savings               │
│ S3 Intelligent-Tiering       - Up to 40% savings               │
│ gp3 vs gp2                   - 20% cheaper                     │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

