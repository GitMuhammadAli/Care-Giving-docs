# Cloud Networking - Complete Guide

> **MUST REMEMBER**: VPC is your private network in the cloud. Subnets divide the VPC (public for internet-facing, private for backend). Security groups are stateful firewalls per resource. NAT Gateway lets private resources reach internet. VPC Peering/Transit Gateway connects VPCs. Always use private subnets for databases; public subnets only for load balancers.

---

## How to Explain Like a Senior Developer

"Cloud networking is about isolation and connectivity. A VPC is your private network - nothing gets in or out unless you allow it. You divide it into subnets: public subnets have a route to the internet (via Internet Gateway), private subnets don't. Put your ALB in public subnets, your EC2/containers in private subnets, and databases in isolated private subnets. Security groups are your firewalls - they're stateful, so if you allow traffic in, the response goes out automatically. NAT Gateway lets private resources (like Lambda in VPC) reach the internet for updates. For multi-VPC or multi-region, use Transit Gateway as a hub. The key principle: least privilege - only open what you need, and use private connectivity (VPC endpoints) for AWS services."

---

## Core Implementation

### VPC Architecture with Terraform

```hcl
# networking/vpc.tf

# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "main-vpc"
  }
}

# Internet Gateway (for public subnets)
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
}

# Public Subnets (3 AZs)
resource "aws_subnet" "public" {
  count                   = 3
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index * 2}.0/24"
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true
  
  tags = {
    Name = "public-subnet-${count.index + 1}"
    Type = "public"
  }
}

# Private Subnets (for application servers)
resource "aws_subnet" "private" {
  count             = 3
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index * 2 + 1}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  tags = {
    Name = "private-subnet-${count.index + 1}"
    Type = "private"
  }
}

# Database Subnets (isolated)
resource "aws_subnet" "database" {
  count             = 3
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${10 + count.index}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  tags = {
    Name = "database-subnet-${count.index + 1}"
    Type = "database"
  }
}

# Elastic IP for NAT Gateway
resource "aws_eip" "nat" {
  count  = 3  # One per AZ for HA
  domain = "vpc"
}

# NAT Gateways (one per AZ for HA)
resource "aws_nat_gateway" "main" {
  count         = 3
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  
  depends_on = [aws_internet_gateway.main]
}

# Route Tables
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
}

resource "aws_route_table" "private" {
  count  = 3
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  count          = 3
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count          = 3
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}

data "aws_availability_zones" "available" {
  state = "available"
}
```

### Security Groups

```hcl
# networking/security-groups.tf

# ALB Security Group
resource "aws_security_group" "alb" {
  name        = "alb-sg"
  description = "Security group for Application Load Balancer"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    description = "HTTPS from anywhere"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    description = "HTTP for redirect"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Application Security Group
resource "aws_security_group" "app" {
  name        = "app-sg"
  description = "Security group for application servers"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    description     = "HTTP from ALB"
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Database Security Group
resource "aws_security_group" "database" {
  name        = "database-sg"
  description = "Security group for databases"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    description     = "PostgreSQL from app"
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }
  
  # No egress to internet!
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = [aws_vpc.main.cidr_block]  # VPC only
  }
}

# Redis/ElastiCache Security Group
resource "aws_security_group" "cache" {
  name        = "cache-sg"
  description = "Security group for ElastiCache"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    description     = "Redis from app"
    from_port       = 6379
    to_port         = 6379
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }
}
```

### VPC Endpoints (Private AWS Access)

```hcl
# networking/vpc-endpoints.tf

# S3 Gateway Endpoint (free)
resource "aws_vpc_endpoint" "s3" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.s3"
  vpc_endpoint_type = "Gateway"
  
  route_table_ids = concat(
    [aws_route_table.public.id],
    aws_route_table.private[*].id
  )
}

# DynamoDB Gateway Endpoint (free)
resource "aws_vpc_endpoint" "dynamodb" {
  vpc_id            = aws_vpc.main.id
  service_name      = "com.amazonaws.${var.region}.dynamodb"
  vpc_endpoint_type = "Gateway"
  
  route_table_ids = concat(
    [aws_route_table.public.id],
    aws_route_table.private[*].id
  )
}

# Interface Endpoints (cost per hour + data)
resource "aws_vpc_endpoint" "ssm" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.ssm"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoints.id]
  private_dns_enabled = true
}

resource "aws_vpc_endpoint" "secrets_manager" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.secretsmanager"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoints.id]
  private_dns_enabled = true
}

resource "aws_vpc_endpoint" "ecr_api" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.ecr.api"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoints.id]
  private_dns_enabled = true
}

resource "aws_vpc_endpoint" "ecr_dkr" {
  vpc_id              = aws_vpc.main.id
  service_name        = "com.amazonaws.${var.region}.ecr.dkr"
  vpc_endpoint_type   = "Interface"
  subnet_ids          = aws_subnet.private[*].id
  security_group_ids  = [aws_security_group.vpc_endpoints.id]
  private_dns_enabled = true
}

# Security group for VPC endpoints
resource "aws_security_group" "vpc_endpoints" {
  name        = "vpc-endpoints-sg"
  description = "Security group for VPC endpoints"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    description = "HTTPS from VPC"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = [aws_vpc.main.cidr_block]
  }
}

variable "region" {
  default = "us-east-1"
}
```

### Transit Gateway (Multi-VPC)

```hcl
# networking/transit-gateway.tf

# Transit Gateway
resource "aws_ec2_transit_gateway" "main" {
  description                     = "Main Transit Gateway"
  auto_accept_shared_attachments  = "enable"
  default_route_table_association = "enable"
  default_route_table_propagation = "enable"
  dns_support                     = "enable"
  vpn_ecmp_support               = "enable"
  
  tags = {
    Name = "main-tgw"
  }
}

# Attach VPCs to Transit Gateway
resource "aws_ec2_transit_gateway_vpc_attachment" "production" {
  subnet_ids         = aws_subnet.private[*].id
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = aws_vpc.main.id
  
  tags = {
    Name = "production-attachment"
  }
}

resource "aws_ec2_transit_gateway_vpc_attachment" "staging" {
  subnet_ids         = aws_subnet.staging_private[*].id
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = aws_vpc.staging.id
  
  tags = {
    Name = "staging-attachment"
  }
}

# Route from production to staging via TGW
resource "aws_route" "prod_to_staging" {
  count                  = 3
  route_table_id         = aws_route_table.private[count.index].id
  destination_cidr_block = aws_vpc.staging.cidr_block
  transit_gateway_id     = aws_ec2_transit_gateway.main.id
}
```

---

## Real-World Scenarios

### Scenario 1: Three-Tier Architecture

```typescript
// networking/three-tier.ts

/**
 * Three-Tier VPC Architecture:
 * 
 * Tier 1: Public Subnets
 *   - ALB (load balancer)
 *   - NAT Gateways
 *   - Bastion hosts (if needed)
 * 
 * Tier 2: Private Subnets
 *   - ECS/EKS/EC2 (application)
 *   - Lambda (if VPC-attached)
 * 
 * Tier 3: Isolated Subnets
 *   - RDS/Aurora (database)
 *   - ElastiCache (cache)
 *   - No internet access
 */

// CDK Implementation
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as ecs from 'aws-cdk-lib/aws-ecs';
import * as elbv2 from 'aws-cdk-lib/aws-elasticloadbalancingv2';
import * as rds from 'aws-cdk-lib/aws-rds';

export class ThreeTierStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string) {
    super(scope, id);
    
    // VPC with public, private, and isolated subnets
    const vpc = new ec2.Vpc(this, 'VPC', {
      maxAzs: 3,
      natGateways: 3, // HA: one per AZ
      subnetConfiguration: [
        {
          name: 'Public',
          subnetType: ec2.SubnetType.PUBLIC,
          cidrMask: 24,
        },
        {
          name: 'Private',
          subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS,
          cidrMask: 24,
        },
        {
          name: 'Database',
          subnetType: ec2.SubnetType.PRIVATE_ISOLATED,
          cidrMask: 24,
        },
      ],
    });
    
    // Application Load Balancer in public subnets
    const alb = new elbv2.ApplicationLoadBalancer(this, 'ALB', {
      vpc,
      internetFacing: true,
      vpcSubnets: { subnetType: ec2.SubnetType.PUBLIC },
    });
    
    // ECS Cluster in private subnets
    const cluster = new ecs.Cluster(this, 'Cluster', {
      vpc,
    });
    
    // Database in isolated subnets
    const database = new rds.DatabaseInstance(this, 'Database', {
      engine: rds.DatabaseInstanceEngine.postgres({
        version: rds.PostgresEngineVersion.VER_14,
      }),
      vpc,
      vpcSubnets: { subnetType: ec2.SubnetType.PRIVATE_ISOLATED },
      multiAz: true,
    });
    
    // Allow app to connect to database
    database.connections.allowFrom(
      cluster,
      ec2.Port.tcp(5432),
      'Allow from ECS'
    );
  }
}
```

### Scenario 2: Hybrid Cloud with VPN

```hcl
# networking/hybrid-vpn.tf

# Customer Gateway (on-prem router)
resource "aws_customer_gateway" "onprem" {
  bgp_asn    = 65000
  ip_address = var.onprem_router_ip
  type       = "ipsec.1"
  
  tags = {
    Name = "on-prem-cgw"
  }
}

# Virtual Private Gateway
resource "aws_vpn_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "main-vgw"
  }
}

# Site-to-Site VPN Connection
resource "aws_vpn_connection" "main" {
  vpn_gateway_id      = aws_vpn_gateway.main.id
  customer_gateway_id = aws_customer_gateway.onprem.id
  type                = "ipsec.1"
  static_routes_only  = false  # Use BGP
  
  tags = {
    Name = "on-prem-vpn"
  }
}

# Enable route propagation
resource "aws_vpn_gateway_route_propagation" "private" {
  count          = 3
  vpn_gateway_id = aws_vpn_gateway.main.id
  route_table_id = aws_route_table.private[count.index].id
}

variable "onprem_router_ip" {
  description = "Public IP of on-premises router"
}
```

---

## Common Pitfalls

### 1. Putting Databases in Public Subnets

```hcl
# ❌ BAD: Database accessible from internet
resource "aws_db_instance" "bad" {
  publicly_accessible = true  # NEVER do this!
  # ...
}

# ✅ GOOD: Database in private/isolated subnet
resource "aws_db_instance" "good" {
  publicly_accessible    = false
  db_subnet_group_name   = aws_db_subnet_group.private.name
  vpc_security_group_ids = [aws_security_group.database.id]
}
```

### 2. Overly Permissive Security Groups

```hcl
# ❌ BAD: Open to the world
resource "aws_security_group_rule" "bad" {
  type        = "ingress"
  from_port   = 22
  to_port     = 22
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]  # SSH from anywhere!
}

# ✅ GOOD: Specific source
resource "aws_security_group_rule" "good" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion.id  # Only from bastion
}
```

### 3. Single NAT Gateway

```hcl
# ❌ BAD: Single NAT = single point of failure
resource "aws_nat_gateway" "bad" {
  # Only one NAT gateway for all AZs
}

# ✅ GOOD: NAT per AZ for high availability
resource "aws_nat_gateway" "good" {
  count         = 3  # One per AZ
  subnet_id     = aws_subnet.public[count.index].id
  allocation_id = aws_eip.nat[count.index].id
}
```

---

## Interview Questions

### Q1: What's the difference between public and private subnets?

**A:** **Public subnets** have a route to an Internet Gateway, and resources can have public IPs for direct internet access. **Private subnets** route internet traffic through NAT Gateway - resources can reach the internet but aren't directly accessible. Use public for ALBs, private for applications, isolated (no internet) for databases.

### Q2: How do security groups differ from NACLs?

**A:** **Security groups** are stateful (return traffic automatically allowed), applied to instances, allow rules only. **NACLs** are stateless (must explicitly allow return traffic), applied to subnets, support allow and deny rules, evaluated in order. Use security groups for most cases; NACLs for subnet-level blocking (e.g., block specific IPs).

### Q3: When would you use VPC Peering vs Transit Gateway?

**A:** **VPC Peering** for: simple 1-to-1 connections, lower cost, lower latency. **Transit Gateway** for: many VPCs (hub-spoke model), VPN attachments, transitive routing, centralized management. Peering doesn't scale past ~10 VPCs due to full-mesh complexity.

### Q4: How do you connect Lambda to resources in a VPC?

**A:** Configure Lambda with VPC, subnets, and security groups. Lambda runs inside your VPC and can access private resources. Caveats: adds cold start latency (~1s), needs NAT Gateway for internet access, consumes ENIs (consider ENI limits). Use only when needed (RDS, ElastiCache access).

---

## Quick Reference Checklist

### VPC Design
- [ ] Use multiple AZs for HA
- [ ] Separate public/private/isolated subnets
- [ ] Plan CIDR ranges for growth
- [ ] Enable VPC Flow Logs

### Security
- [ ] Least privilege security groups
- [ ] Databases in isolated subnets
- [ ] Use VPC endpoints for AWS services
- [ ] No 0.0.0.0/0 ingress rules

### Connectivity
- [ ] NAT Gateway per AZ for HA
- [ ] VPC endpoints for S3/DynamoDB (free)
- [ ] Transit Gateway for multi-VPC
- [ ] VPN/Direct Connect for hybrid

### Monitoring
- [ ] VPC Flow Logs enabled
- [ ] CloudWatch metrics for NAT
- [ ] GuardDuty for threat detection
- [ ] Config rules for compliance

---

*Last updated: February 2026*

