# Cloud Cost Management - Complete Guide

> **MUST REMEMBER**: Cloud costs = compute + storage + network + requests. Optimize: right-size instances, use reserved/savings plans for steady workloads, spot for fault-tolerant jobs. Tag everything for cost allocation. Set budgets with alerts. Key metrics: cost per transaction, cost per user. 80% of costs often come from 20% of resources - find and optimize those first.

---

## How to Explain Like a Senior Developer

"Cloud bills grow quickly if you don't pay attention. The key is visibility first - tag everything, set up cost allocation, understand where money goes. Then optimize: right-size instances (most are oversized), use Reserved Instances or Savings Plans for baseline load (up to 72% savings), Spot for batch jobs (up to 90% savings). Storage costs add up - lifecycle policies move old data to cheaper tiers. Data transfer is often a surprise cost - keep traffic within AZ when possible, use VPC endpoints for AWS services. The goal isn't minimum cost, it's optimal cost - spending $1000 to save $100 isn't optimization. Track cost per business metric (cost per user, per transaction) to tie cloud spend to business value."

---

## Core Implementation

### Cost Allocation Tags

```hcl
# cost/tagging.tf

# Enforce tagging with AWS Organizations SCP
resource "aws_organizations_policy" "require_tags" {
  name    = "RequireCostAllocationTags"
  content = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "RequireTags"
        Effect = "Deny"
        Action = [
          "ec2:RunInstances",
          "ec2:CreateVolume",
          "rds:CreateDBInstance",
          "lambda:CreateFunction"
        ]
        Resource = "*"
        Condition = {
          Null = {
            "aws:RequestTag/Environment" = "true"
            "aws:RequestTag/Project"     = "true"
            "aws:RequestTag/Owner"       = "true"
            "aws:RequestTag/CostCenter"  = "true"
          }
        }
      }
    ]
  })
  type = "SERVICE_CONTROL_POLICY"
}

# Default tags for all resources (Terraform provider level)
provider "aws" {
  region = "us-east-1"
  
  default_tags {
    tags = {
      Environment = var.environment
      Project     = var.project_name
      Owner       = var.owner_email
      CostCenter  = var.cost_center
      ManagedBy   = "terraform"
    }
  }
}

# Tag propagation for Auto Scaling
resource "aws_autoscaling_group" "app" {
  # ... other config ...
  
  tag {
    key                 = "Environment"
    value               = var.environment
    propagate_at_launch = true
  }
  
  tag {
    key                 = "Project"
    value               = var.project_name
    propagate_at_launch = true
  }
}

variable "environment" {
  default = "production"
}

variable "project_name" {
  default = "my-app"
}

variable "owner_email" {
  default = "team@example.com"
}

variable "cost_center" {
  default = "engineering"
}
```

### AWS Budgets and Alerts

```typescript
// cost/budgets.ts
import {
  BudgetsClient,
  CreateBudgetCommand,
  Budget,
  NotificationType,
  ThresholdType,
  ComparisonOperator,
} from '@aws-sdk/client-budgets';

const budgets = new BudgetsClient({ region: 'us-east-1' });

interface BudgetConfig {
  name: string;
  amount: number;
  unit: 'USD';
  timeUnit: 'MONTHLY' | 'QUARTERLY' | 'ANNUALLY';
  alertThresholds: number[]; // Percentages
  alertEmails: string[];
  costFilters?: {
    Service?: string[];
    TagKeyValue?: string[];
  };
}

async function createBudget(config: BudgetConfig): Promise<void> {
  const budget: Budget = {
    BudgetName: config.name,
    BudgetType: 'COST',
    BudgetLimit: {
      Amount: config.amount.toString(),
      Unit: config.unit,
    },
    TimeUnit: config.timeUnit,
    CostFilters: config.costFilters,
    CostTypes: {
      IncludeTax: true,
      IncludeSubscription: true,
      UseBlended: false,
      IncludeRefund: false,
      IncludeCredit: false,
      IncludeUpfront: true,
      IncludeRecurring: true,
      IncludeOtherSubscription: true,
      IncludeSupport: true,
      IncludeDiscount: true,
      UseAmortized: false,
    },
  };
  
  const notifications = config.alertThresholds.map((threshold) => ({
    Notification: {
      NotificationType: 'ACTUAL' as NotificationType,
      ComparisonOperator: 'GREATER_THAN' as ComparisonOperator,
      Threshold: threshold,
      ThresholdType: 'PERCENTAGE' as ThresholdType,
    },
    Subscribers: config.alertEmails.map((email) => ({
      SubscriptionType: 'EMAIL' as const,
      Address: email,
    })),
  }));
  
  await budgets.send(
    new CreateBudgetCommand({
      AccountId: process.env.AWS_ACCOUNT_ID,
      Budget: budget,
      NotificationsWithSubscribers: notifications,
    })
  );
}

// Create budgets for different teams/projects
async function setupBudgets(): Promise<void> {
  // Overall account budget
  await createBudget({
    name: 'Monthly-Total',
    amount: 10000,
    unit: 'USD',
    timeUnit: 'MONTHLY',
    alertThresholds: [50, 80, 100, 120],
    alertEmails: ['finance@example.com', 'eng-leads@example.com'],
  });
  
  // Per-project budget
  await createBudget({
    name: 'ProjectA-Monthly',
    amount: 3000,
    unit: 'USD',
    timeUnit: 'MONTHLY',
    alertThresholds: [80, 100],
    alertEmails: ['projecta-team@example.com'],
    costFilters: {
      TagKeyValue: ['Project$ProjectA'],
    },
  });
  
  // Per-service budget (catch runaway services)
  await createBudget({
    name: 'EC2-Monthly',
    amount: 5000,
    unit: 'USD',
    timeUnit: 'MONTHLY',
    alertThresholds: [80, 100],
    alertEmails: ['infrastructure@example.com'],
    costFilters: {
      Service: ['Amazon Elastic Compute Cloud - Compute'],
    },
  });
}
```

### Reserved Instances and Savings Plans

```typescript
// cost/reservations.ts

/**
 * Savings Options:
 * 
 * 1. Reserved Instances (RI)
 *    - Specific to instance type and region
 *    - Up to 72% savings
 *    - 1 or 3 year terms
 *    - Standard (most savings) or Convertible (flexibility)
 * 
 * 2. Savings Plans
 *    - Compute Savings Plan: Any EC2, Lambda, Fargate
 *    - EC2 Instance Savings Plan: Specific instance family
 *    - Up to 72% savings
 *    - More flexible than RIs
 * 
 * 3. Spot Instances
 *    - Up to 90% savings
 *    - Can be interrupted
 *    - Good for: batch processing, stateless apps, CI/CD
 */

import {
  CostExplorerClient,
  GetReservationCoverageCommand,
  GetReservationUtilizationCommand,
  GetSavingsPlansCoverageCommand,
  GetSavingsPlansUtilizationCommand,
} from '@aws-sdk/client-cost-explorer';

const ce = new CostExplorerClient({ region: 'us-east-1' });

async function getReservationMetrics(): Promise<{
  coverage: number;
  utilization: number;
}> {
  const endDate = new Date();
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - 30);
  
  const timePeriod = {
    Start: startDate.toISOString().split('T')[0],
    End: endDate.toISOString().split('T')[0],
  };
  
  // Coverage: What % of usage is covered by reservations
  const coverageResponse = await ce.send(
    new GetReservationCoverageCommand({
      TimePeriod: timePeriod,
    })
  );
  
  // Utilization: What % of reservations are being used
  const utilizationResponse = await ce.send(
    new GetReservationUtilizationCommand({
      TimePeriod: timePeriod,
    })
  );
  
  return {
    coverage: parseFloat(
      coverageResponse.Total?.CoverageHours?.CoverageHoursPercentage || '0'
    ),
    utilization: parseFloat(
      utilizationResponse.Total?.UtilizationPercentage || '0'
    ),
  };
}

// Recommendations for purchasing RIs/Savings Plans
interface ReservationRecommendation {
  service: string;
  instanceType: string;
  region: string;
  onDemandCost: number;
  reservedCost: number;
  estimatedSavings: number;
  savingsPercentage: number;
  recommendedQuantity: number;
}

async function getReservationRecommendations(): Promise<ReservationRecommendation[]> {
  // In practice, use AWS Cost Explorer API or Trusted Advisor
  // This is a simplified example of the analysis
  
  const recommendations: ReservationRecommendation[] = [];
  
  // Analyze on-demand usage patterns
  // Look for consistent usage over 30+ days
  // Calculate break-even point vs reservation cost
  
  return recommendations;
}
```

### Spot Instance Strategy

```typescript
// cost/spot-instances.ts
import {
  EC2Client,
  RequestSpotInstancesCommand,
  DescribeSpotPriceHistoryCommand,
} from '@aws-sdk/client-ec2';

const ec2 = new EC2Client({ region: 'us-east-1' });

interface SpotConfig {
  instanceType: string;
  maxPrice?: string; // Optional: defaults to on-demand price
  availabilityZones: string[];
  interruptionBehavior: 'terminate' | 'stop' | 'hibernate';
}

// Get current spot prices
async function getSpotPrices(instanceTypes: string[]): Promise<
  Map<string, { az: string; price: number }[]>
> {
  const response = await ec2.send(
    new DescribeSpotPriceHistoryCommand({
      InstanceTypes: instanceTypes,
      ProductDescriptions: ['Linux/UNIX'],
      StartTime: new Date(),
    })
  );
  
  const prices = new Map<string, { az: string; price: number }[]>();
  
  for (const price of response.SpotPriceHistory || []) {
    const type = price.InstanceType!;
    if (!prices.has(type)) {
      prices.set(type, []);
    }
    prices.get(type)!.push({
      az: price.AvailabilityZone!,
      price: parseFloat(price.SpotPrice!),
    });
  }
  
  return prices;
}

// Spot with diversification for reliability
async function launchDiversifiedSpotFleet(config: {
  targetCapacity: number;
  instanceTypes: string[];
  maxPricePercentAboveOnDemand: number;
}): Promise<void> {
  // Use multiple instance types to reduce interruption risk
  // Spread across availability zones
  // Set max price slightly above current spot price
  
  const prices = await getSpotPrices(config.instanceTypes);
  
  // Sort instance types by current price
  const sortedTypes = config.instanceTypes.sort((a, b) => {
    const aPrice = Math.min(...(prices.get(a)?.map((p) => p.price) || [999]));
    const bPrice = Math.min(...(prices.get(b)?.map((p) => p.price) || [999]));
    return aPrice - bPrice;
  });
  
  // Launch configuration would use EC2 Fleet or Auto Scaling
  // with mixed instances policy
  console.log('Recommended instance order:', sortedTypes);
}

// Handle spot interruption
async function handleSpotInterruption(): Promise<void> {
  // Check for interruption notice (2-minute warning)
  try {
    const response = await fetch(
      'http://169.254.169.254/latest/meta-data/spot/instance-action',
      { signal: AbortSignal.timeout(1000) }
    );
    
    if (response.ok) {
      const action = await response.json();
      console.log('Spot interruption notice:', action);
      
      // Graceful shutdown:
      // 1. Stop accepting new requests
      // 2. Finish in-flight requests
      // 3. Save state if needed
      // 4. Deregister from load balancer
      
      await gracefulShutdown();
    }
  } catch {
    // No interruption notice
  }
}

async function gracefulShutdown(): Promise<void> {
  // Implement graceful shutdown
}
```

### Cost Analysis Dashboard

```typescript
// cost/analysis.ts
import {
  CostExplorerClient,
  GetCostAndUsageCommand,
  Granularity,
} from '@aws-sdk/client-cost-explorer';

const ce = new CostExplorerClient({ region: 'us-east-1' });

interface CostBreakdown {
  date: string;
  total: number;
  byService: Record<string, number>;
  byTag: Record<string, number>;
}

async function getCostBreakdown(
  startDate: string,
  endDate: string,
  granularity: 'DAILY' | 'MONTHLY' = 'DAILY'
): Promise<CostBreakdown[]> {
  const response = await ce.send(
    new GetCostAndUsageCommand({
      TimePeriod: {
        Start: startDate,
        End: endDate,
      },
      Granularity: granularity as Granularity,
      Metrics: ['UnblendedCost'],
      GroupBy: [
        { Type: 'DIMENSION', Key: 'SERVICE' },
      ],
    })
  );
  
  return (response.ResultsByTime || []).map((period) => {
    const byService: Record<string, number> = {};
    let total = 0;
    
    for (const group of period.Groups || []) {
      const service = group.Keys?.[0] || 'Unknown';
      const cost = parseFloat(group.Metrics?.UnblendedCost?.Amount || '0');
      byService[service] = cost;
      total += cost;
    }
    
    return {
      date: period.TimePeriod?.Start || '',
      total,
      byService,
      byTag: {},
    };
  });
}

// Calculate cost per business metric
async function getCostPerTransaction(
  startDate: string,
  endDate: string,
  transactionCount: number
): Promise<number> {
  const costs = await getCostBreakdown(startDate, endDate, 'MONTHLY');
  const totalCost = costs.reduce((sum, c) => sum + c.total, 0);
  
  return totalCost / transactionCount;
}

// Find top cost drivers
async function getTopCostDrivers(
  startDate: string,
  endDate: string,
  top: number = 10
): Promise<Array<{ service: string; cost: number; percentage: number }>> {
  const costs = await getCostBreakdown(startDate, endDate, 'MONTHLY');
  
  const serviceTotal: Record<string, number> = {};
  let grandTotal = 0;
  
  for (const period of costs) {
    for (const [service, cost] of Object.entries(period.byService)) {
      serviceTotal[service] = (serviceTotal[service] || 0) + cost;
      grandTotal += cost;
    }
  }
  
  return Object.entries(serviceTotal)
    .map(([service, cost]) => ({
      service,
      cost,
      percentage: (cost / grandTotal) * 100,
    }))
    .sort((a, b) => b.cost - a.cost)
    .slice(0, top);
}
```

---

## Real-World Scenarios

### Scenario 1: Right-Sizing Recommendations

```typescript
// cost/rightsizing.ts
import {
  ComputeOptimizerClient,
  GetEC2InstanceRecommendationsCommand,
} from '@aws-sdk/client-compute-optimizer';

const optimizer = new ComputeOptimizerClient({ region: 'us-east-1' });

interface RightsizingRecommendation {
  instanceId: string;
  currentType: string;
  recommendedType: string;
  currentMonthlyCost: number;
  recommendedMonthlyCost: number;
  savings: number;
  savingsPercentage: number;
  cpuUtilization: number;
  memoryUtilization: number;
}

async function getRightsizingRecommendations(): Promise<RightsizingRecommendation[]> {
  const response = await optimizer.send(
    new GetEC2InstanceRecommendationsCommand({})
  );
  
  const recommendations: RightsizingRecommendation[] = [];
  
  for (const rec of response.instanceRecommendations || []) {
    if (rec.finding === 'OVER_PROVISIONED') {
      const current = rec.currentInstanceType;
      const recommended = rec.recommendationOptions?.[0];
      
      if (recommended) {
        const currentCost = parseFloat(
          rec.utilizationMetrics?.find((m) => m.name === 'CPU')?.value || '0'
        );
        
        recommendations.push({
          instanceId: rec.instanceArn?.split('/').pop() || '',
          currentType: current || '',
          recommendedType: recommended.instanceType || '',
          currentMonthlyCost: 0, // Calculate from pricing API
          recommendedMonthlyCost: 0,
          savings: 0,
          savingsPercentage: recommended.savingsOpportunityPercentage || 0,
          cpuUtilization: currentCost,
          memoryUtilization: 0,
        });
      }
    }
  }
  
  return recommendations;
}

// Automated rightsizing (with approval workflow)
async function applyRightsizing(
  instanceId: string,
  newInstanceType: string
): Promise<void> {
  // 1. Create change request / ticket
  // 2. Wait for approval
  // 3. Schedule maintenance window
  // 4. Stop instance
  // 5. Change instance type
  // 6. Start instance
  // 7. Validate health
  // 8. Close ticket
}
```

### Scenario 2: FinOps Dashboard

```typescript
// cost/finops-dashboard.ts

interface FinOpsMetrics {
  currentMonthSpend: number;
  forecastedMonthSpend: number;
  budgetUtilization: number;
  reservationCoverage: number;
  reservationUtilization: number;
  spotSavings: number;
  wastage: number;
  costPerUser: number;
  costTrend: 'increasing' | 'decreasing' | 'stable';
}

async function getFinOpsMetrics(): Promise<FinOpsMetrics> {
  const [
    currentSpend,
    forecast,
    reservationMetrics,
    spotMetrics,
    wastage,
    userCount,
  ] = await Promise.all([
    getCurrentMonthSpend(),
    getForecast(),
    getReservationMetrics(),
    getSpotSavings(),
    calculateWastage(),
    getActiveUserCount(),
  ]);
  
  const budget = 10000; // From budget config
  
  return {
    currentMonthSpend: currentSpend,
    forecastedMonthSpend: forecast,
    budgetUtilization: (currentSpend / budget) * 100,
    reservationCoverage: reservationMetrics.coverage,
    reservationUtilization: reservationMetrics.utilization,
    spotSavings: spotMetrics,
    wastage,
    costPerUser: currentSpend / userCount,
    costTrend: determineTrend(currentSpend, getLastMonthSpend()),
  };
}

function determineTrend(
  current: number,
  previous: number
): 'increasing' | 'decreasing' | 'stable' {
  const change = ((current - previous) / previous) * 100;
  if (change > 5) return 'increasing';
  if (change < -5) return 'decreasing';
  return 'stable';
}

async function calculateWastage(): Promise<number> {
  // Sum of:
  // - Unused EBS volumes
  // - Unattached Elastic IPs
  // - Idle load balancers
  // - Over-provisioned instances
  // - Old snapshots
  return 0;
}

async function getCurrentMonthSpend(): Promise<number> { return 0; }
async function getForecast(): Promise<number> { return 0; }
async function getSpotSavings(): Promise<number> { return 0; }
async function getActiveUserCount(): Promise<number> { return 0; }
async function getLastMonthSpend(): Promise<number> { return 0; }
async function getReservationMetrics(): Promise<{ coverage: number; utilization: number }> {
  return { coverage: 0, utilization: 0 };
}
```

---

## Common Pitfalls

### 1. Unused Resources

```typescript
// Common wastage sources:
// - Unattached EBS volumes
// - Unused Elastic IPs ($0.005/hour = $43/month)
// - Idle load balancers ($16+/month)
// - Old snapshots
// - Stopped but not terminated instances
// - Dev environments running 24/7

// Automated cleanup script
async function cleanupUnusedResources(): Promise<void> {
  // Find and delete unattached EBS volumes older than 7 days
  // Release unused Elastic IPs
  // Delete old snapshots (keep last N)
  // Alert on idle load balancers
}
```

### 2. Data Transfer Costs

```typescript
// ❌ BAD: Cross-AZ traffic for every request
// Service in us-east-1a calling database in us-east-1b
// $0.01/GB each way = adds up fast

// ✅ GOOD: Keep traffic within same AZ when possible
// Use VPC endpoints for AWS services (no data transfer charge)
// Use CloudFront for outbound traffic to internet
// Compress data before transfer
```

### 3. Over-Provisioning "Just in Case"

```typescript
// ❌ BAD: m5.4xlarge "to be safe"
// Running at 10% CPU = 90% waste

// ✅ GOOD: Start small, monitor, scale up if needed
// Use Auto Scaling to handle peaks
// Review Compute Optimizer recommendations monthly
```

---

## Interview Questions

### Q1: How would you reduce cloud costs by 30%?

**A:** 1) Analyze current spend by service/tag. 2) Right-size over-provisioned instances. 3) Purchase Reserved Instances/Savings Plans for steady workloads (up to 72% savings). 4) Use Spot for fault-tolerant workloads (up to 90% savings). 5) Clean up unused resources. 6) Optimize data transfer. 7) Implement auto-scaling. 8) Use lifecycle policies for storage.

### Q2: Explain Reserved Instances vs Savings Plans.

**A:** **Reserved Instances**: Specific instance type/region, 1-3 year commitment, Standard (most savings) or Convertible (can change type). **Savings Plans**: Commit to $/hour spend, Compute SP applies to any EC2/Lambda/Fargate, EC2 Instance SP for specific family. Savings Plans are more flexible; RIs give slightly more savings for predictable workloads.

### Q3: How do you allocate cloud costs to teams?

**A:** 1) Implement mandatory tagging (Project, Team, Environment, CostCenter). 2) Use AWS Organizations with separate accounts per team/project. 3) Enable Cost Allocation Tags in billing. 4) Create budgets per tag/account. 5) Generate chargeback reports. 6) Review in monthly FinOps meetings.

### Q4: What's the 80/20 rule in cloud cost optimization?

**A:** 80% of costs typically come from 20% of resources. Focus optimization efforts on top cost drivers first. Common candidates: EC2 (compute), RDS (database), data transfer, S3 (if lots of requests). Optimizing the top 3-5 services usually yields most savings.

---

## Quick Reference Checklist

### Visibility
- [ ] Enable Cost Explorer
- [ ] Implement tagging strategy
- [ ] Set up cost allocation reports
- [ ] Create budgets with alerts

### Optimization
- [ ] Right-size instances
- [ ] Purchase RIs/Savings Plans
- [ ] Use Spot for appropriate workloads
- [ ] Implement auto-scaling

### Governance
- [ ] Monthly cost reviews
- [ ] Anomaly detection alerts
- [ ] Unused resource cleanup
- [ ] FinOps practices

### Metrics
- [ ] Track cost per user/transaction
- [ ] Monitor reservation utilization
- [ ] Measure optimization savings
- [ ] Review forecasts vs actuals

---

*Last updated: February 2026*

