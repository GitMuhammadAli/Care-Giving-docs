# Chapter 12: Cloud Architecture

> "The cloud is just someone else's computer, but with superpowers."

---

## ☁️ Cloud Computing Models

```
┌─────────────────────────────────────────────────────────────────┐
│                    YOU MANAGE                                   │
│                                                                 │
│  On-Premises    │  IaaS        │  PaaS       │  SaaS           │
│  ────────────   │  ─────       │  ─────      │  ─────          │
│  Applications   │  Applications│  Applications│                 │
│  Data           │  Data        │  Data       │                 │
│  Runtime        │  Runtime     │             │                 │
│  Middleware     │  Middleware  │             │                 │
│  OS             │  OS          │             │                 │
│  Virtualization │              │             │                 │
│  Servers        │              │             │                 │
│  Storage        │              │             │                 │
│  Networking     │              │             │                 │
│                                                                 │
│                    PROVIDER MANAGES                             │
└─────────────────────────────────────────────────────────────────┘

Examples:
IaaS: AWS EC2, GCP Compute Engine, Azure VMs
PaaS: AWS Elastic Beanstalk, Heroku, Cloud Run
SaaS: Salesforce, Gmail, Slack
```

---

## 🏗️ AWS Core Services

### Compute

```
┌─────────────────────────────────────────────────────────────────┐
│ EC2 (Virtual Machines)                                          │
│ - Full control over instance                                    │
│ - Choose OS, hardware specs                                     │
│ - Pay per hour/second                                           │
│                                                                 │
│ Lambda (Serverless Functions)                                   │
│ - No server management                                          │
│ - Pay per invocation                                            │
│ - Auto-scales to zero                                           │
│                                                                 │
│ ECS/EKS (Containers)                                            │
│ - ECS: AWS-native container orchestration                       │
│ - EKS: Managed Kubernetes                                       │
│                                                                 │
│ Fargate (Serverless Containers)                                 │
│ - No EC2 instances to manage                                    │
│ - Define CPU/memory, AWS handles rest                           │
└─────────────────────────────────────────────────────────────────┘
```

### Database

```
┌─────────────────────────────────────────────────────────────────┐
│ RDS (Relational Database Service)                               │
│ - PostgreSQL, MySQL, MariaDB, Oracle, SQL Server                │
│ - Automated backups, multi-AZ                                   │
│                                                                 │
│ Aurora (AWS-optimized MySQL/PostgreSQL)                         │
│ - 5x MySQL, 3x PostgreSQL performance                           │
│ - Auto-scaling storage (up to 128TB)                            │
│ - Global Database for multi-region                              │
│                                                                 │
│ DynamoDB (NoSQL)                                                │
│ - Key-value and document store                                  │
│ - Single-digit millisecond latency                              │
│ - Serverless, scales automatically                              │
│                                                                 │
│ ElastiCache (Caching)                                           │
│ - Managed Redis or Memcached                                    │
│ - Sub-millisecond latency                                       │
└─────────────────────────────────────────────────────────────────┘
```

### Storage

```
┌─────────────────────────────────────────────────────────────────┐
│ S3 (Simple Storage Service)                                     │
│ - Object storage (files, images, backups)                       │
│ - 99.999999999% durability                                      │
│ - Storage classes: Standard, IA, Glacier                        │
│                                                                 │
│ EBS (Elastic Block Store)                                       │
│ - Block storage for EC2                                         │
│ - Like a virtual hard drive                                     │
│                                                                 │
│ EFS (Elastic File System)                                       │
│ - Shared file storage (NFS)                                     │
│ - Multiple instances access same files                          │
└─────────────────────────────────────────────────────────────────┘
```

### Networking

```
┌─────────────────────────────────────────────────────────────────┐
│ VPC (Virtual Private Cloud)                                     │
│ - Your private network in AWS                                   │
│ - Subnets (public/private)                                      │
│ - Security groups, NACLs                                        │
│                                                                 │
│ Route 53 (DNS)                                                  │
│ - Domain registration                                           │
│ - DNS routing (latency, geolocation, weighted)                  │
│                                                                 │
│ CloudFront (CDN)                                                │
│ - Content delivery network                                      │
│ - Edge locations worldwide                                      │
│                                                                 │
│ API Gateway                                                     │
│ - Managed API endpoints                                         │
│ - Rate limiting, authentication                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🌐 Multi-Region Architecture

```
                    Global DNS (Route 53)
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
    ┌──────────┐   ┌──────────┐   ┌──────────┐
    │ US-East  │   │ EU-West  │   │ AP-Tokyo │
    │  Region  │   │  Region  │   │  Region  │
    └────┬─────┘   └────┬─────┘   └────┬─────┘
         │              │              │
    ┌────┴────┐   ┌────┴────┐   ┌────┴────┐
    │ Primary │   │ Replica │   │ Replica │
    │   DB    │◄──│   DB    │──►│   DB    │
    └─────────┘   └─────────┘   └─────────┘
    
Strategies:
- Active-Active: All regions serve traffic
- Active-Passive: Standby regions for failover
- Follow-the-Sun: Route based on time of day
```

### Regional vs Global Services

```
Regional (deploy per region):
- EC2, RDS, Lambda
- ELB, ECS, EKS
- S3 (replication optional)

Global (single deployment):
- IAM
- Route 53
- CloudFront
- WAF, Shield
```

---

## 🔒 AWS Security Best Practices

### IAM (Identity and Access Management)

```json
// Principle of Least Privilege
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/uploads/*"
    }
  ]
}

// Use roles, not access keys
// Enable MFA for root account
// Regularly rotate credentials
```

### VPC Security

```
┌───────────────────────────────────────────────────────────────┐
│ VPC (10.0.0.0/16)                                             │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Public Subnet (10.0.1.0/24)                             │ │
│  │ ┌─────────────┐  ┌─────────────┐                       │ │
│  │ │     ALB     │  │   NAT GW    │                       │ │
│  │ └──────┬──────┘  └──────┬──────┘                       │ │
│  └────────┼────────────────┼───────────────────────────────┘ │
│           │                │                                  │
│  ┌────────┼────────────────┼───────────────────────────────┐ │
│  │ Private Subnet (10.0.2.0/24)                            │ │
│  │ ┌──────┴──────┐  ┌──────┴──────┐                       │ │
│  │ │  App Server │  │  App Server │                       │ │
│  │ └─────────────┘  └─────────────┘                       │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │ Database Subnet (10.0.3.0/24)                           │ │
│  │ ┌─────────────┐                                        │ │
│  │ │     RDS     │  (No internet access)                  │ │
│  │ └─────────────┘                                        │ │
│  └─────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────┘
```

---

## 💰 Cost Optimization

### Pricing Models

```
On-Demand:
- Pay per hour/second
- No commitment
- Highest price

Reserved Instances:
- 1-3 year commitment
- Up to 72% discount
- Best for predictable workloads

Spot Instances:
- Bid on spare capacity
- Up to 90% discount
- Can be interrupted (2 min warning)

Savings Plans:
- Commitment to $/hour
- Flexible across services
```

### Cost Optimization Tips

```
1. Right-size instances
   - Monitor utilization
   - Use smaller instances if possible
   
2. Use spot instances for:
   - Batch processing
   - CI/CD workers
   - Dev/test environments
   
3. Auto-scaling
   - Scale down during off-hours
   - Scale to zero when possible
   
4. Storage optimization
   - Use appropriate S3 tiers
   - Delete unused snapshots
   
5. Reserved capacity
   - Commit for predictable workloads
   - Use Savings Plans for flexibility
```

---

## 🏗️ Well-Architected Framework

### Five Pillars

```
1. Operational Excellence
   □ Automate operations
   □ Make frequent, small changes
   □ Learn from failures
   
2. Security
   □ Strong identity foundation
   □ Enable traceability
   □ Automate security
   
3. Reliability
   □ Test recovery procedures
   □ Scale horizontally
   □ Automate change management
   
4. Performance Efficiency
   □ Use serverless where possible
   □ Go global in minutes
   □ Experiment more often
   
5. Cost Optimization
   □ Stop guessing capacity
   □ Measure efficiency
   □ Analyze and attribute spending
```

---

## 📖 Further Reading

- AWS Well-Architected Framework
- AWS Architecture Center
- Cloud Design Patterns (Azure, applicable to all)
- "AWS Certified Solutions Architect Study Guide"

---

**Next:** [Chapter 13: AI/ML in Production →](./13-ai-ml-production.md)


