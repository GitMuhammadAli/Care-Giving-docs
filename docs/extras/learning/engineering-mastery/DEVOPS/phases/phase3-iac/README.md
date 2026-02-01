# Phase 3: Infrastructure as Code
## Weeks 7-9 | Automate Everything

> **Prerequisites:** Completed Phase 2 (VPS deployment)  
> **Time:** 10-15 hours per week  
> **Outcome:** Provision cloud infrastructure using code

---

## 🎯 Learning Objectives

By the end of Phase 3, you will:
- [ ] Understand Infrastructure as Code (IaC) principles
- [ ] Write Terraform configurations
- [ ] Manage Terraform state and workspaces
- [ ] Use modules for reusable infrastructure
- [ ] Compare Terraform vs Pulumi

---

## Why Infrastructure as Code?

```
MANUAL PROVISIONING:                    INFRASTRUCTURE AS CODE:
┌─────────────────────────┐            ┌─────────────────────────┐
│ Click, click, click...  │            │ main.tf                 │
│ 50 steps in AWS console │            │                         │
│ "Did I configure X?"    │            │ resource "aws_instance" │
│ "What settings did I    │            │   ami = "ami-xxx"       │
│  use last time?"        │            │   instance_type = "t2"  │
│                         │            │ }                       │
│ Slow, error-prone,      │            │                         │
│ not reproducible        │            │ Fast, consistent,       │
│                         │            │ version controlled      │
└─────────────────────────┘            └─────────────────────────┘
```

**Benefits of IaC:**
- **Reproducible:** Same code = same infrastructure
- **Version controlled:** Track changes in Git
- **Reviewable:** Code review for infrastructure changes
- **Automatable:** CI/CD for infrastructure
- **Documentable:** Code IS documentation

---

## 📅 Weekly Breakdown

### Week 7: Terraform Basics

**What you'll learn:**
- Terraform concepts (providers, resources, data sources)
- HCL syntax
- Terraform workflow (init, plan, apply, destroy)
- Variables and outputs

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 17: Infrastructure as Code | 2 hours |
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 32: Terraform Deep Dive | 3 hours |

**Key concepts:**

```
TERRAFORM WORKFLOW:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Write Code    terraform init    terraform plan    terraform apply
│   ─────────────▶──────────────▶──────────────▶──────────────▶   │
│                                                                  │
│   main.tf        Downloads        Shows what       Creates       │
│   variables.tf   providers        will change      resources     │
│   outputs.tf                                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Hands-on exercises:**

```hcl
# Exercise 1: Your first Terraform configuration
# main.tf

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

# Create a VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "my-vpc"
  }
}

# Create a subnet
resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"
  
  tags = {
    Name = "public-subnet"
  }
}
```

```hcl
# Exercise 2: Variables
# variables.tf

variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "environment" {
  description = "Environment name"
  type        = string
}

# main.tf - using variables
provider "aws" {
  region = var.region
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
  
  tags = {
    Name        = "web-server"
    Environment = var.environment
  }
}
```

```hcl
# Exercise 3: Outputs
# outputs.tf

output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "instance_public_ip" {
  description = "Public IP of EC2 instance"
  value       = aws_instance.web.public_ip
}
```

```bash
# Exercise 4: Terraform commands
terraform init       # Initialize, download providers
terraform fmt        # Format code
terraform validate   # Validate syntax
terraform plan       # Preview changes
terraform apply      # Apply changes
terraform destroy    # Destroy resources
terraform output     # Show outputs
terraform state list # List resources in state
```

**Checkpoint quiz:**
1. What's the difference between `terraform plan` and `terraform apply`?
2. Where is Terraform state stored by default?
3. How do you reference a variable in Terraform?

---

### Week 8: Terraform Advanced

**What you'll learn:**
- Remote state (S3 + DynamoDB)
- Workspaces for multi-environment
- Modules for reusability
- Data sources
- Provisioners

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 32: Advanced sections | 3 hours |

**Key concepts:**

```
TERRAFORM STATE MANAGEMENT:

Local State (default):           Remote State (production):
┌─────────────────┐              ┌─────────────────┐
│ terraform.tfstate│              │     S3 Bucket   │
│                 │              │                 │
│ On your laptop  │              │ + DynamoDB Lock │
│ Not shared      │              │ Shared, locked  │
│ Can be lost     │              │ Versioned       │
└─────────────────┘              └─────────────────┘
```

**Hands-on exercises:**

```hcl
# Exercise 1: Remote state configuration
# backend.tf

terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

```hcl
# Exercise 2: Workspaces
# Use same code for dev/staging/prod

# commands:
# terraform workspace new dev
# terraform workspace new staging
# terraform workspace new prod
# terraform workspace select dev

# main.tf - workspace-aware configuration
locals {
  environment = terraform.workspace
  
  instance_types = {
    dev     = "t2.micro"
    staging = "t2.small"
    prod    = "t2.medium"
  }
}

resource "aws_instance" "web" {
  instance_type = local.instance_types[local.environment]
  
  tags = {
    Environment = local.environment
  }
}
```

```hcl
# Exercise 3: Creating a module
# modules/vpc/main.tf

variable "cidr_block" {
  type = string
}

variable "name" {
  type = string
}

resource "aws_vpc" "this" {
  cidr_block = var.cidr_block
  
  tags = {
    Name = var.name
  }
}

output "vpc_id" {
  value = aws_vpc.this.id
}

# Using the module (main.tf in root)
module "vpc" {
  source = "./modules/vpc"
  
  cidr_block = "10.0.0.0/16"
  name       = "production-vpc"
}

# Reference module output
resource "aws_subnet" "public" {
  vpc_id = module.vpc.vpc_id
  # ...
}
```

```hcl
# Exercise 4: Data sources (read existing resources)

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
}
```

**Checkpoint quiz:**
1. Why use remote state instead of local?
2. What's the purpose of DynamoDB in Terraform state management?
3. When would you use a Terraform module?

---

### Week 9: Pulumi & IaC Comparison

**What you'll learn:**
- Pulumi basics
- Real programming languages for IaC
- Terraform vs Pulumi comparison
- When to use which

**Read:**
| Resource | Section | Time |
|----------|---------|------|
| Pulumi Docs | Getting Started | 2 hours |
| [devop-complete-guide.md](../../devop-complete-guide.md) | Chapter 17: IaC comparison | 1 hour |

**Key concepts:**

```
TERRAFORM vs PULUMI:

Terraform (HCL):                 Pulumi (TypeScript):
┌─────────────────────────┐     ┌─────────────────────────────────┐
│ resource "aws_instance" │     │ const server = new aws.ec2.     │
│   "web" {               │     │   Instance("web", {             │
│   ami = "ami-xxx"       │     │     ami: "ami-xxx",             │
│   instance_type = "t2"  │     │     instanceType: "t2.micro",   │
│ }                       │     │   });                           │
└─────────────────────────┘     └─────────────────────────────────┘
        │                                    │
        ▼                                    ▼
   Domain-specific               Real programming language
   language (DSL)               (TypeScript, Python, Go)
```

**Pulumi hands-on:**

```typescript
// Exercise 1: Pulumi with TypeScript
// index.ts

import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";

// Create a VPC
const vpc = new aws.ec2.Vpc("my-vpc", {
    cidrBlock: "10.0.0.0/16",
    tags: {
        Name: "my-vpc",
    },
});

// Create a subnet
const subnet = new aws.ec2.Subnet("public-subnet", {
    vpcId: vpc.id,
    cidrBlock: "10.0.1.0/24",
    tags: {
        Name: "public-subnet",
    },
});

// Create an EC2 instance
const server = new aws.ec2.Instance("web-server", {
    ami: "ami-0c55b159cbfafe1f0",
    instanceType: "t2.micro",
    subnetId: subnet.id,
    tags: {
        Name: "web-server",
    },
});

// Export the public IP
export const publicIp = server.publicIp;
```

```bash
# Exercise 2: Pulumi commands
pulumi new aws-typescript  # Create new project
pulumi up                  # Deploy
pulumi preview             # Preview changes
pulumi destroy             # Destroy resources
pulumi stack               # Show stack info
```

**Comparison table:**

| Aspect | Terraform | Pulumi |
|--------|-----------|--------|
| Language | HCL (domain-specific) | TypeScript, Python, Go, C# |
| Learning curve | Lower | Higher (need programming) |
| IDE support | Basic | Full (autocomplete, types) |
| Testing | Limited | Unit tests possible |
| Community | Larger | Growing |
| State | tfstate | Pulumi Service or self-hosted |
| Best for | Most infrastructure | Complex logic, existing developers |

**When to use which:**
- **Terraform:** Standard infrastructure, team with varied backgrounds
- **Pulumi:** Complex conditionals, developers who prefer TypeScript/Python

---

## 📋 Phase 3 Project: Multi-Environment Infrastructure

**Create Terraform configuration that provisions:**
1. VPC with public and private subnets
2. EC2 instance for web server
3. RDS database in private subnet
4. Security groups
5. Support for dev/staging/prod environments using workspaces

**Project structure:**
```
terraform/
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── backend.tf
└── modules/
    ├── vpc/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    ├── ec2/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── rds/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

---

## ✅ Phase 3 Completion Checklist

Before moving to Phase 4, ensure you can:

- [ ] Explain benefits of Infrastructure as Code
- [ ] Write Terraform configurations with providers and resources
- [ ] Use variables, outputs, and locals
- [ ] Configure remote state with S3
- [ ] Use workspaces for multiple environments
- [ ] Create and use Terraform modules
- [ ] Understand Terraform vs Pulumi tradeoffs
- [ ] Complete the Phase 3 project

---

## 🔗 Quick Reference

**Terraform commands:**
```bash
terraform init      # Initialize
terraform fmt       # Format code
terraform validate  # Validate syntax
terraform plan      # Preview changes
terraform apply     # Apply changes
terraform destroy   # Destroy all
terraform output    # Show outputs
terraform workspace list    # List workspaces
terraform workspace new dev # Create workspace
```

**HCL syntax basics:**
```hcl
# Variable
variable "name" {
  type    = string
  default = "value"
}

# Resource
resource "aws_instance" "name" {
  ami = "ami-xxx"
}

# Data source
data "aws_ami" "name" {
  # ...
}

# Output
output "name" {
  value = aws_instance.name.id
}

# Module
module "vpc" {
  source = "./modules/vpc"
  name   = "my-vpc"
}
```

---

## ➡️ Next Step

Ready for Phase 4? [Containers & Kubernetes →](../phase4-containers/README.md)

