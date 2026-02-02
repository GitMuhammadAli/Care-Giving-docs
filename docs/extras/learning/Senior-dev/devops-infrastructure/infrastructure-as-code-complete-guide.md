# 🏗️ Infrastructure as Code - Complete Guide

> A comprehensive guide to Infrastructure as Code (IaC) - Terraform, Pulumi, CloudFormation, state management, best practices, and production patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure through machine-readable configuration files rather than manual processes, enabling version control, consistency, repeatability, and automation of infrastructure deployments."

### The 7 Key Concepts (Remember These!)
```
1. DECLARATIVE     → Define desired state, tool determines how to achieve it
2. IMPERATIVE      → Define step-by-step commands to execute
3. STATE           → Record of current infrastructure (Terraform state file)
4. DRIFT           → Difference between actual and declared infrastructure
5. MODULES         → Reusable, composable infrastructure components
6. PROVIDERS       → Plugins that interact with cloud APIs
7. IDEMPOTENT      → Same config applied multiple times = same result
```

### IaC Tool Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│                    IAC TOOL COMPARISON                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TERRAFORM (HashiCorp)                                         │
│  • HCL (HashiCorp Configuration Language)                      │
│  • Multi-cloud (AWS, GCP, Azure, etc.)                         │
│  • Large provider ecosystem                                    │
│  • State management required                                   │
│                                                                 │
│  PULUMI                                                         │
│  • Real programming languages (TS, Python, Go)                 │
│  • Multi-cloud                                                 │
│  • Testing with standard frameworks                            │
│  • State managed by Pulumi service or self-hosted              │
│                                                                 │
│  AWS CLOUDFORMATION                                             │
│  • AWS-native, JSON/YAML                                       │
│  • Deep AWS integration                                        │
│  • No external state management                                │
│  • AWS-only                                                    │
│                                                                 │
│  AWS CDK                                                        │
│  • Programming languages → CloudFormation                      │
│  • AWS-only                                                    │
│  • Higher-level constructs                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Terraform Workflow
```
┌─────────────────────────────────────────────────────────────────┐
│                    TERRAFORM WORKFLOW                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. WRITE                                                       │
│     ┌────────────────────────────────────────────────────────┐ │
│     │  main.tf, variables.tf, outputs.tf                     │ │
│     └────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  2. INIT                                                        │
│     ┌────────────────────────────────────────────────────────┐ │
│     │  terraform init                                        │ │
│     │  • Download providers                                  │ │
│     │  • Initialize backend (state storage)                  │ │
│     └────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  3. PLAN                                                        │
│     ┌────────────────────────────────────────────────────────┐ │
│     │  terraform plan                                        │ │
│     │  • Compare config to state                             │ │
│     │  • Show what will change (+/-/~)                       │ │
│     └────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  4. APPLY                                                       │
│     ┌────────────────────────────────────────────────────────┐ │
│     │  terraform apply                                       │ │
│     │  • Execute changes                                     │ │
│     │  • Update state file                                   │ │
│     └────────────────────────────────────────────────────────┘ │
│                              │                                  │
│                              ▼                                  │
│  5. STATE                                                       │
│     ┌────────────────────────────────────────────────────────┐ │
│     │  terraform.tfstate (or remote backend)                 │ │
│     │  • Tracks real resources                               │ │
│     │  • Maps config to actual infrastructure                │ │
│     └────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Declarative"** | "Terraform is declarative - we define desired state, not imperative steps" |
| **"Drift detection"** | "We run drift detection to find manual changes to infrastructure" |
| **"Remote state"** | "We use S3 with DynamoDB locking for remote state" |
| **"State locking"** | "State locking prevents concurrent modifications" |
| **"Modules"** | "We use modules to create reusable infrastructure components" |
| **"Blast radius"** | "We split state files to limit blast radius of changes" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Max Terraform state size | **~100MB** practical | Performance degrades |
| Plan timeout | **10-30 min** typical | Depends on resource count |
| State refresh interval | **On every plan/apply** | Can be skipped with -refresh=false |
| Recommended resources/state | **< 500** | Split if larger |
| Parallelism default | **10** | Concurrent operations |

### The "Wow" Statement (Memorize This!)
> "We manage 200+ microservices infrastructure with Terraform. We use a monorepo with environment-specific workspaces (dev/staging/prod). State is stored in S3 with DynamoDB locking to prevent concurrent modifications. We modularized common patterns - VPC, EKS cluster, RDS, etc. - and compose them per service. CI/CD runs terraform plan on PRs with Atlantis for review, terraform apply on merge. We use Terragrunt for DRY configuration across environments. Drift detection runs daily, alerting when manual changes are detected. Sentinel policies enforce compliance (no public S3 buckets, required tags). For sensitive resources, we use terraform plan -target for surgical changes with minimal blast radius."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    GIT REPOSITORY                        │   │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────────────────┐ │   │
│  │  │ modules/  │ │ envs/     │ │ .github/workflows/    │ │   │
│  │  │  vpc/     │ │  dev/     │ │  terraform.yml        │ │   │
│  │  │  eks/     │ │  staging/ │ │                       │ │   │
│  │  │  rds/     │ │  prod/    │ │                       │ │   │
│  │  └───────────┘ └───────────┘ └───────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                              ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    CI/CD PIPELINE                        │   │
│  │  PR: terraform plan → Review → Merge: terraform apply   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│              ┌───────────────┼───────────────┐                 │
│              ▼               ▼               ▼                 │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐      │
│  │   STATE (S3)  │  │   STATE (S3)  │  │   STATE (S3)  │      │
│  │   dev.tfstate │  │ staging.tfstate│ │  prod.tfstate │      │
│  └───────────────┘  └───────────────┘  └───────────────┘      │
│                              │                                  │
│              ┌───────────────┼───────────────┐                 │
│              ▼               ▼               ▼                 │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐      │
│  │  AWS Dev      │  │  AWS Staging  │  │  AWS Prod     │      │
│  │  Account      │  │  Account      │  │  Account      │      │
│  └───────────────┘  └───────────────┘  └───────────────┘      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📚 Table of Contents

1. [Terraform Basics](#1-terraform-basics)
2. [State Management](#2-state-management)
3. [Modules](#3-modules)
4. [AWS CloudFormation](#4-aws-cloudformation)
5. [Pulumi](#5-pulumi)
6. [Best Practices](#6-best-practices)
7. [Common Pitfalls](#7-common-pitfalls)
8. [Interview Questions](#8-interview-questions)

---

## 1. Terraform Basics

```hcl
# ════════════════════════════════════════════════════════════════
# TERRAFORM - BASIC STRUCTURE
# ════════════════════════════════════════════════════════════════

# providers.tf - Provider configuration
terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  # Remote backend
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "terraform"
      Project     = var.project_name
    }
  }
}

# ════════════════════════════════════════════════════════════════
# variables.tf - Input variables
# ════════════════════════════════════════════════════════════════

variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "allowed_cidrs" {
  description = "CIDR blocks allowed to access"
  type        = list(string)
  default     = []
}

variable "tags" {
  description = "Additional tags"
  type        = map(string)
  default     = {}
}

# ════════════════════════════════════════════════════════════════
# main.tf - Resources
# ════════════════════════════════════════════════════════════════

# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "${var.project_name}-vpc"
  }
}

# Subnets
resource "aws_subnet" "public" {
  count             = 3
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.project_name}-public-${count.index + 1}"
    Type = "public"
  }
}

# Security Group
resource "aws_security_group" "web" {
  name        = "${var.project_name}-web-sg"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  lifecycle {
    create_before_destroy = true
  }
}

# EC2 Instance
resource "aws_instance" "web" {
  count         = var.environment == "prod" ? 3 : 1
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public[count.index % length(aws_subnet.public)].id
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
  EOF
  
  tags = merge(var.tags, {
    Name = "${var.project_name}-web-${count.index + 1}"
  })
}

# ════════════════════════════════════════════════════════════════
# data.tf - Data sources
# ════════════════════════════════════════════════════════════════

data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

data "aws_caller_identity" "current" {}

# ════════════════════════════════════════════════════════════════
# outputs.tf - Output values
# ════════════════════════════════════════════════════════════════

output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "instance_public_ips" {
  description = "Public IPs of EC2 instances"
  value       = aws_instance.web[*].public_ip
}

output "security_group_id" {
  description = "Security group ID"
  value       = aws_security_group.web.id
  sensitive   = false
}
```

```bash
# ════════════════════════════════════════════════════════════════
# TERRAFORM COMMANDS
# ════════════════════════════════════════════════════════════════

# Initialize (download providers, setup backend)
terraform init

# Validate configuration syntax
terraform validate

# Format code
terraform fmt -recursive

# Plan (preview changes)
terraform plan -out=tfplan

# Apply changes
terraform apply tfplan
# or
terraform apply -auto-approve  # Skip confirmation (CI/CD)

# Destroy resources
terraform destroy

# Show current state
terraform show

# List resources in state
terraform state list

# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0

# Targeted operations
terraform plan -target=aws_instance.web
terraform apply -target=module.vpc

# Workspace management
terraform workspace list
terraform workspace new staging
terraform workspace select prod
```

---

## 2. State Management

```hcl
# ════════════════════════════════════════════════════════════════
# REMOTE STATE - S3 + DYNAMODB
# ════════════════════════════════════════════════════════════════

# backend.tf
terraform {
  backend "s3" {
    bucket         = "mycompany-terraform-state"
    key            = "services/api/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
    
    # Optional: assume role for cross-account
    # role_arn = "arn:aws:iam::ACCOUNT_ID:role/TerraformRole"
  }
}

# Bootstrap state bucket and DynamoDB table
# state-bootstrap/main.tf

resource "aws_s3_bucket" "terraform_state" {
  bucket = "mycompany-terraform-state"
  
  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
}

# ════════════════════════════════════════════════════════════════
# STATE DATA SOURCE (Reference Other State)
# ════════════════════════════════════════════════════════════════

# Reference another terraform state
data "terraform_remote_state" "vpc" {
  backend = "s3"
  config = {
    bucket = "mycompany-terraform-state"
    key    = "networking/vpc/terraform.tfstate"
    region = "us-east-1"
  }
}

# Use outputs from that state
resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.vpc.outputs.private_subnet_ids[0]
  vpc_security_group_ids = [
    data.terraform_remote_state.vpc.outputs.default_security_group_id
  ]
}
```

```bash
# ════════════════════════════════════════════════════════════════
# STATE OPERATIONS
# ════════════════════════════════════════════════════════════════

# List all resources in state
terraform state list

# Show details of a resource
terraform state show aws_instance.web

# Move resource in state (rename)
terraform state mv aws_instance.web aws_instance.app

# Remove resource from state (doesn't destroy)
terraform state rm aws_instance.web

# Pull state to local file
terraform state pull > terraform.tfstate.backup

# Push local state (dangerous!)
terraform state push terraform.tfstate

# Import existing resource
terraform import aws_s3_bucket.existing bucket-name

# Force unlock state
terraform force-unlock LOCK_ID
```

---

## 3. Modules

```hcl
# ════════════════════════════════════════════════════════════════
# MODULE - VPC EXAMPLE
# modules/vpc/main.tf
# ════════════════════════════════════════════════════════════════

variable "name" {
  description = "VPC name"
  type        = string
}

variable "cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "azs" {
  description = "Availability zones"
  type        = list(string)
}

variable "private_subnets" {
  description = "Private subnet CIDRs"
  type        = list(string)
}

variable "public_subnets" {
  description = "Public subnet CIDRs"
  type        = list(string)
}

# VPC
resource "aws_vpc" "this" {
  cidr_block           = var.cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = var.name
  }
}

# Internet Gateway
resource "aws_internet_gateway" "this" {
  vpc_id = aws_vpc.this.id
  
  tags = {
    Name = "${var.name}-igw"
  }
}

# Public Subnets
resource "aws_subnet" "public" {
  count             = length(var.public_subnets)
  vpc_id            = aws_vpc.this.id
  cidr_block        = var.public_subnets[count.index]
  availability_zone = var.azs[count.index]
  
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.name}-public-${var.azs[count.index]}"
  }
}

# Private Subnets
resource "aws_subnet" "private" {
  count             = length(var.private_subnets)
  vpc_id            = aws_vpc.this.id
  cidr_block        = var.private_subnets[count.index]
  availability_zone = var.azs[count.index]
  
  tags = {
    Name = "${var.name}-private-${var.azs[count.index]}"
  }
}

# NAT Gateway
resource "aws_eip" "nat" {
  count  = length(var.public_subnets)
  domain = "vpc"
}

resource "aws_nat_gateway" "this" {
  count         = length(var.public_subnets)
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  
  tags = {
    Name = "${var.name}-nat-${var.azs[count.index]}"
  }
}

# Outputs
output "vpc_id" {
  value = aws_vpc.this.id
}

output "public_subnet_ids" {
  value = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  value = aws_subnet.private[*].id
}

# ════════════════════════════════════════════════════════════════
# USING THE MODULE
# envs/prod/main.tf
# ════════════════════════════════════════════════════════════════

module "vpc" {
  source = "../../modules/vpc"
  
  name = "prod-vpc"
  cidr = "10.0.0.0/16"
  azs  = ["us-east-1a", "us-east-1b", "us-east-1c"]
  
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
}

# Reference module outputs
resource "aws_instance" "app" {
  subnet_id = module.vpc.private_subnet_ids[0]
}

# ════════════════════════════════════════════════════════════════
# USING PUBLIC MODULES
# ════════════════════════════════════════════════════════════════

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "prod-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  single_nat_gateway = false
  
  tags = {
    Environment = "prod"
  }
}
```

---

## 4. AWS CloudFormation

```yaml
# ════════════════════════════════════════════════════════════════
# CLOUDFORMATION TEMPLATE
# ════════════════════════════════════════════════════════════════

AWSTemplateFormatVersion: '2010-09-09'
Description: 'Production Web Application Stack'

# Parameters
Parameters:
  Environment:
    Type: String
    Default: prod
    AllowedValues: [dev, staging, prod]
    
  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues: [t3.micro, t3.small, t3.medium]
    
  VpcCIDR:
    Type: String
    Default: 10.0.0.0/16

# Conditions
Conditions:
  IsProd: !Equals [!Ref Environment, prod]

# Mappings
Mappings:
  RegionAMI:
    us-east-1:
      AMI: ami-0123456789abcdef0
    us-west-2:
      AMI: ami-0fedcba9876543210

# Resources
Resources:
  # VPC
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCIDR
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Sub ${Environment}-vpc

  # Internet Gateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Sub ${Environment}-igw

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  # Public Subnet
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: !Select [0, !Cidr [!Ref VpcCIDR, 4, 8]]
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub ${Environment}-public-subnet

  # Security Group
  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Web server security group
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  # EC2 Instance
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !If [IsProd, t3.medium, !Ref InstanceType]
      ImageId: !FindInMap [RegionAMI, !Ref 'AWS::Region', AMI]
      SubnetId: !Ref PublicSubnet
      SecurityGroupIds:
        - !Ref WebSecurityGroup
      Tags:
        - Key: Name
          Value: !Sub ${Environment}-web-server
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd

# Outputs
Outputs:
  VpcId:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub ${Environment}-VpcId

  WebServerPublicIP:
    Description: Web server public IP
    Value: !GetAtt WebServer.PublicIp
```

```bash
# CloudFormation CLI commands
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=Environment,ParameterValue=prod

aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml

aws cloudformation delete-stack --stack-name my-stack

aws cloudformation describe-stacks --stack-name my-stack
```

---

## 5. Pulumi

```typescript
// ════════════════════════════════════════════════════════════════
// PULUMI - TYPESCRIPT EXAMPLE
// index.ts
// ════════════════════════════════════════════════════════════════

import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

// Configuration
const config = new pulumi.Config();
const environment = config.require("environment");
const instanceType = config.get("instanceType") || "t3.micro";

// VPC
const vpc = new aws.ec2.Vpc("main-vpc", {
    cidrBlock: "10.0.0.0/16",
    enableDnsHostnames: true,
    enableDnsSupport: true,
    tags: {
        Name: `${environment}-vpc`,
        Environment: environment,
    },
});

// Public Subnet
const publicSubnet = new aws.ec2.Subnet("public-subnet", {
    vpcId: vpc.id,
    cidrBlock: "10.0.1.0/24",
    availabilityZone: "us-east-1a",
    mapPublicIpOnLaunch: true,
    tags: {
        Name: `${environment}-public-subnet`,
    },
});

// Security Group
const webSg = new aws.ec2.SecurityGroup("web-sg", {
    vpcId: vpc.id,
    description: "Web server security group",
    ingress: [
        {
            protocol: "tcp",
            fromPort: 443,
            toPort: 443,
            cidrBlocks: ["0.0.0.0/0"],
        },
        {
            protocol: "tcp",
            fromPort: 80,
            toPort: 80,
            cidrBlocks: ["0.0.0.0/0"],
        },
    ],
    egress: [
        {
            protocol: "-1",
            fromPort: 0,
            toPort: 0,
            cidrBlocks: ["0.0.0.0/0"],
        },
    ],
});

// Get latest Amazon Linux AMI
const ami = aws.ec2.getAmi({
    mostRecent: true,
    owners: ["amazon"],
    filters: [
        {
            name: "name",
            values: ["amzn2-ami-hvm-*-x86_64-gp2"],
        },
    ],
});

// EC2 Instance
const webServer = new aws.ec2.Instance("web-server", {
    ami: ami.then(ami => ami.id),
    instanceType: instanceType,
    subnetId: publicSubnet.id,
    vpcSecurityGroupIds: [webSg.id],
    tags: {
        Name: `${environment}-web-server`,
    },
    userData: `#!/bin/bash
        yum update -y
        yum install -y httpd
        systemctl start httpd
        systemctl enable httpd
    `,
});

// Exports
export const vpcId = vpc.id;
export const publicSubnetId = publicSubnet.id;
export const webServerPublicIp = webServer.publicIp;
```

```bash
# Pulumi CLI commands
pulumi new aws-typescript       # Create new project
pulumi config set environment prod
pulumi preview                  # Preview changes
pulumi up                       # Deploy
pulumi destroy                  # Destroy
pulumi stack output vpcId       # Get output
```

---

## 6. Best Practices

```hcl
# ════════════════════════════════════════════════════════════════
# IAC BEST PRACTICES
# ════════════════════════════════════════════════════════════════

# 1. Use consistent naming conventions
resource "aws_instance" "web" {
  tags = {
    Name        = "${var.project}-${var.environment}-web"
    Environment = var.environment
    Project     = var.project
    ManagedBy   = "terraform"
  }
}

# 2. Use variables for all configurable values
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
  
  validation {
    condition     = can(regex("^t3\\.", var.instance_type))
    error_message = "Instance type must be from t3 family."
  }
}

# 3. Use locals for computed values
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project
    ManagedBy   = "terraform"
  }
  
  name_prefix = "${var.project}-${var.environment}"
}

# 4. Use lifecycle rules appropriately
resource "aws_instance" "web" {
  lifecycle {
    create_before_destroy = true  # Zero-downtime replacement
    prevent_destroy       = true  # Protect critical resources
    ignore_changes        = [     # Ignore external changes
      tags["LastModified"],
    ]
  }
}

# 5. Use data sources for existing resources
data "aws_vpc" "existing" {
  filter {
    name   = "tag:Name"
    values = ["production-vpc"]
  }
}

# 6. Sensitive values
variable "database_password" {
  description = "Database password"
  type        = string
  sensitive   = true  # Won't show in logs
}

output "connection_string" {
  value     = "postgresql://${var.db_user}:${var.db_password}@${aws_db_instance.main.endpoint}"
  sensitive = true
}

# 7. Use moved blocks for refactoring
moved {
  from = aws_instance.web
  to   = aws_instance.application
}

# 8. Use depends_on only when necessary
resource "aws_instance" "app" {
  # Explicit dependency (usually not needed)
  depends_on = [aws_internet_gateway.main]
}
```

---

## 7. Common Pitfalls

```hcl
# ════════════════════════════════════════════════════════════════
# IAC PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Hardcoded values
# Bad
resource "aws_instance" "web" {
  instance_type = "t3.micro"
  ami           = "ami-0123456789"
}

# Good
resource "aws_instance" "web" {
  instance_type = var.instance_type
  ami           = data.aws_ami.amazon_linux.id
}

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 2: No state locking
# Bad
terraform {
  backend "s3" {
    bucket = "my-state"
    key    = "terraform.tfstate"
  }
}

# Good - with DynamoDB locking
terraform {
  backend "s3" {
    bucket         = "my-state"
    key            = "terraform.tfstate"
    dynamodb_table = "terraform-locks"  # Prevents concurrent access
    encrypt        = true
  }
}

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 3: Storing secrets in state
# Bad - secret in plain text in state
resource "aws_db_instance" "main" {
  password = "supersecret123"
}

# Good - use secrets manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 4: Monolithic state file
# Bad - everything in one state
# envs/prod/main.tf (hundreds of resources)

# Good - split by component/service
# networking/vpc/main.tf
# services/api/main.tf
# databases/rds/main.tf

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 5: No version constraints
# Bad
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
}

# Good - pin versions
terraform {
  required_version = ">= 1.5.0, < 2.0.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"  # >= 5.0, < 6.0
    }
  }
}

# ════════════════════════════════════════════════════════════════
# ❌ PITFALL 6: Not using terraform plan in CI
# Bad - apply directly
# ci: terraform apply -auto-approve

# Good - plan first, require approval
# ci: 
#   terraform plan -out=tfplan
#   # Review plan output
#   terraform apply tfplan
```

---

## 8. Interview Questions

### Basic Questions

**Q: "What is Infrastructure as Code?"**
> "Managing infrastructure through code rather than manual processes. Benefits: version control, consistency, repeatability, automation, documentation. Tools: Terraform (multi-cloud, HCL), CloudFormation (AWS, YAML/JSON), Pulumi (programming languages)."

**Q: "Terraform vs CloudFormation?"**
> "Terraform: Multi-cloud, HCL language, requires state management, large provider ecosystem. CloudFormation: AWS-only, YAML/JSON, no external state (AWS manages), deeper AWS integration. Use Terraform for multi-cloud or complex setups, CloudFormation for AWS-native."

**Q: "What is Terraform state?"**
> "Records mapping between config and real resources. Contains resource IDs, metadata. Essential for plan/apply to know what exists. Store remotely (S3) with locking (DynamoDB) for team collaboration."

### Intermediate Questions

**Q: "How do you manage Terraform state in a team?"**
> "Remote backend (S3, GCS, Terraform Cloud). State locking with DynamoDB or equivalent. Workspaces or separate state files per environment. CI/CD for plan/apply. Never edit state manually. Regular backups via versioning."

**Q: "How do you handle secrets in Terraform?"**
> "Never hardcode. Mark variables as sensitive. Use data sources for secrets managers (AWS Secrets Manager, Vault). Encrypt state at rest. Consider terraform-sops for encrypted files. State still contains secrets - secure backend access."

**Q: "What are Terraform modules?"**
> "Reusable infrastructure components. Encapsulate resources, variables, outputs. Can be local or from registry. Enable DRY, consistency, best practices. Use versioning for public modules."

### Advanced Questions

**Q: "How do you handle drift detection?"**
> "Run terraform plan regularly (scheduled). Compare plan output for unexpected changes. Use tools like driftctl for comprehensive detection. Remediate by either updating code to match or apply to revert. Consider why drift occurred - process issue?"

**Q: "How do you structure Terraform for large organizations?"**
> "Separate state per service/component to limit blast radius. Modules for reusable patterns. Environment-specific configurations (workspaces or directories). CI/CD with plan reviews. Remote state data sources for cross-stack references. Terragrunt for DRY. Policy enforcement with Sentinel/OPA."

**Q: "How do you test Terraform code?"**
> "terraform validate for syntax. terraform plan for preview. Terratest for integration tests (create, verify, destroy). Checkov/tfsec for security scanning. Kitchen-Terraform for compliance. Terraform Cloud/Enterprise for policy checks."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                    TERRAFORM CHEAT SHEET                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  WORKFLOW:                                                      │
│  terraform init      - Initialize (providers, backend)         │
│  terraform validate  - Validate syntax                         │
│  terraform plan      - Preview changes                         │
│  terraform apply     - Execute changes                         │
│  terraform destroy   - Remove all resources                    │
│                                                                 │
│  STATE:                                                         │
│  terraform state list   - List resources                       │
│  terraform state show   - Show resource details                │
│  terraform state mv     - Move/rename resource                 │
│  terraform state rm     - Remove from state                    │
│  terraform import       - Import existing resource             │
│                                                                 │
│  WORKSPACE:                                                     │
│  terraform workspace list/new/select/delete                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

FILE STRUCTURE:
┌────────────────────────────────────────────────────────────────┐
│ main.tf         - Primary resources                            │
│ variables.tf    - Input variables                              │
│ outputs.tf      - Output values                                │
│ providers.tf    - Provider configuration                       │
│ data.tf         - Data sources                                 │
│ locals.tf       - Local values                                 │
│ versions.tf     - Version constraints                          │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

