# 🐋 Container Orchestration - Complete Guide

> A comprehensive guide to container orchestration - Docker Swarm, AWS ECS, container networking, service discovery, and orchestration patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Container orchestration automates the deployment, scaling, networking, and management of containerized applications across clusters of hosts, handling scheduling, load balancing, service discovery, and self-healing to ensure applications run reliably at scale."

### The 7 Key Concepts (Remember These!)
```
1. SCHEDULING       → Deciding which host runs which container
2. SERVICE DISCOVERY → Finding services across dynamic containers
3. LOAD BALANCING   → Distributing traffic to container instances
4. SCALING          → Adding/removing container instances
5. NETWORKING       → Container-to-container communication
6. HEALTH CHECKS    → Detecting and replacing failed containers
7. SECRETS          → Managing sensitive configuration
```

### Orchestration Platform Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│              CONTAINER ORCHESTRATION COMPARISON                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  KUBERNETES                                                    │
│  ──────────                                                     │
│  • Industry standard, most features                            │
│  • Steeper learning curve                                      │
│  • Most portable (any cloud, on-prem)                          │
│  • Largest ecosystem                                           │
│  • Best for: Complex, large-scale deployments                  │
│                                                                 │
│  AWS ECS (Elastic Container Service)                           │
│  ──────────────────────────────────                             │
│  • AWS-native, simpler than K8s                                │
│  • Deep AWS integration (IAM, ALB, CloudWatch)                 │
│  • Fargate = serverless containers                             │
│  • Best for: AWS-only, simpler needs                           │
│                                                                 │
│  DOCKER SWARM                                                   │
│  ────────────                                                   │
│  • Built into Docker                                           │
│  • Simplest to set up                                          │
│  • Good for small-medium deployments                           │
│  • Best for: Docker-native, simpler needs                      │
│                                                                 │
│  NOMAD (HashiCorp)                                             │
│  ────────────────                                               │
│  • Multi-workload (containers, VMs, batch)                     │
│  • Simple, single binary                                       │
│  • Good Consul/Vault integration                               │
│  • Best for: Mixed workloads, HashiCorp stack                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Container Networking Models
```
┌─────────────────────────────────────────────────────────────────┐
│                 CONTAINER NETWORKING                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BRIDGE (Default)                                              │
│  ────────────────                                               │
│  Container ─┬─ [Bridge Network] ─── Host ─── Internet          │
│  Container ─┘                                                  │
│  • Containers on same bridge can communicate                   │
│  • Port mapping to expose services                             │
│                                                                 │
│  HOST                                                           │
│  ────                                                           │
│  Container ──── [Host Network Stack] ──── Internet              │
│  • No network isolation                                        │
│  • Best performance (no NAT overhead)                          │
│                                                                 │
│  OVERLAY (Swarm/K8s)                                           │
│  ───────────────────                                            │
│  Host 1: Container A ─┐                                        │
│                       ├─ [Overlay Network] ─┤                  │
│  Host 2: Container B ─┘                     │                  │
│  • Spans multiple hosts                                        │
│  • Encrypted communication                                     │
│                                                                 │
│  AWSVPC (ECS)                                                   │
│  ────────────                                                   │
│  Container ──── [ENI - Own IP] ──── VPC ──── Internet          │
│  • Each task gets VPC IP address                               │
│  • Security groups per task                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Task definition"** | "ECS task definition specifies container image, resources, ports" |
| **"Service discovery"** | "Service discovery via Cloud Map lets services find each other" |
| **"Sidecar pattern"** | "We use sidecar containers for logging and proxying" |
| **"Rolling update"** | "ECS rolling updates replace containers with zero downtime" |
| **"Fargate"** | "Fargate eliminates infrastructure management - serverless containers" |
| **"awsvpc mode"** | "awsvpc mode gives each task its own ENI and security group" |

### Key Numbers to Remember
| Metric | Value | Notes |
|--------|-------|-------|
| ECS tasks per service | **1000** max | Soft limit |
| Fargate vCPU | **0.25 - 16** | Per task |
| Fargate memory | **0.5 - 120 GB** | Per task |
| Swarm managers | **3, 5, or 7** | Odd number for quorum |
| Health check grace | **60+ seconds** | For slow-starting apps |

### The "Wow" Statement (Memorize This!)
> "We run 50+ microservices on AWS ECS with Fargate for serverless container management - no EC2 instances to manage. Each service has a task definition specifying container image, CPU/memory, ports, and IAM roles. Services are deployed behind ALB with path-based routing. We use awsvpc networking mode so each task gets its own ENI and security group. Service discovery via AWS Cloud Map enables service-to-service communication. Blue-green deployments via CodeDeploy shift traffic gradually with automatic rollback on health check failures. Auto scaling based on CPU and custom CloudWatch metrics. Secrets from AWS Secrets Manager injected at runtime. Our deployment pipeline: GitHub → CodePipeline → CodeBuild → ECR → ECS with zero-downtime rolling updates."

---

## 📚 Table of Contents

1. [Docker Swarm](#1-docker-swarm)
2. [AWS ECS](#2-aws-ecs)
3. [ECS Fargate](#3-ecs-fargate)
4. [Container Networking](#4-container-networking)
5. [Service Discovery](#5-service-discovery)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Docker Swarm

```yaml
# ════════════════════════════════════════════════════════════════
# DOCKER SWARM - STACK DEPLOYMENT
# docker-compose.yml (Swarm mode)
# ════════════════════════════════════════════════════════════════

version: '3.8'

services:
  api:
    image: myregistry/api:${VERSION:-latest}
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
        order: start-first
      rollback_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
      placement:
        constraints:
          - node.role == worker
          - node.labels.type == api
        preferences:
          - spread: node.availability_zone
    ports:
      - target: 3000
        published: 80
        mode: ingress
    networks:
      - frontend
      - backend
    secrets:
      - db_password
      - api_key
    configs:
      - source: api_config
        target: /app/config.json
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

  worker:
    image: myregistry/worker:${VERSION:-latest}
    deploy:
      mode: global  # One per node
      resources:
        limits:
          cpus: '1'
          memory: 1G
    networks:
      - backend
    environment:
      - REDIS_URL=redis://redis:6379

  redis:
    image: redis:7-alpine
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager
    volumes:
      - redis-data:/data
    networks:
      - backend

  nginx:
    image: nginx:alpine
    deploy:
      replicas: 2
      placement:
        max_replicas_per_node: 1
    ports:
      - "443:443"
    configs:
      - source: nginx_config
        target: /etc/nginx/nginx.conf
    secrets:
      - source: ssl_cert
        target: /etc/nginx/ssl/cert.pem
      - source: ssl_key
        target: /etc/nginx/ssl/key.pem
    networks:
      - frontend

networks:
  frontend:
    driver: overlay
    attachable: true
  backend:
    driver: overlay
    internal: true  # No external access

volumes:
  redis-data:
    driver: local

secrets:
  db_password:
    external: true
  api_key:
    external: true
  ssl_cert:
    file: ./certs/cert.pem
  ssl_key:
    file: ./certs/key.pem

configs:
  api_config:
    file: ./config/api.json
  nginx_config:
    file: ./config/nginx.conf
```

```bash
# ════════════════════════════════════════════════════════════════
# DOCKER SWARM COMMANDS
# ════════════════════════════════════════════════════════════════

# Initialize swarm
docker swarm init --advertise-addr 192.168.1.1

# Join as worker
docker swarm join --token SWMTKN-1-xxx 192.168.1.1:2377

# Join as manager
docker swarm join-token manager

# Deploy stack
docker stack deploy -c docker-compose.yml myapp

# List services
docker service ls

# Scale service
docker service scale myapp_api=5

# Update service
docker service update --image myregistry/api:v2 myapp_api

# Rolling update with health check
docker service update \
  --update-parallelism 1 \
  --update-delay 10s \
  --update-failure-action rollback \
  --health-cmd "curl -f http://localhost:3000/health" \
  --health-interval 30s \
  myapp_api

# Rollback
docker service rollback myapp_api

# View logs
docker service logs -f myapp_api

# Drain node for maintenance
docker node update --availability drain node-1

# Create secret
echo "mysecret" | docker secret create db_password -

# Create config
docker config create api_config ./config.json
```

---

## 2. AWS ECS

```hcl
# ════════════════════════════════════════════════════════════════
# AWS ECS - TERRAFORM
# ════════════════════════════════════════════════════════════════

# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = "production-cluster"

  setting {
    name  = "containerInsights"
    value = "enabled"
  }

  configuration {
    execute_command_configuration {
      logging = "OVERRIDE"
      log_configuration {
        cloud_watch_log_group_name = aws_cloudwatch_log_group.ecs_exec.name
      }
    }
  }
}

# Capacity providers for mixed Fargate/EC2
resource "aws_ecs_cluster_capacity_providers" "main" {
  cluster_name = aws_ecs_cluster.main.name

  capacity_providers = [
    "FARGATE",
    "FARGATE_SPOT",
    aws_ecs_capacity_provider.ec2.name
  ]

  default_capacity_provider_strategy {
    capacity_provider = "FARGATE"
    weight            = 1
    base              = 1
  }
}

# Task Definition
resource "aws_ecs_task_definition" "api" {
  family                   = "api-task"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = 512
  memory                   = 1024
  execution_role_arn       = aws_iam_role.ecs_execution.arn
  task_role_arn           = aws_iam_role.ecs_task.arn

  container_definitions = jsonencode([
    {
      name      = "api"
      image     = "${aws_ecr_repository.api.repository_url}:${var.image_tag}"
      essential = true

      portMappings = [
        {
          containerPort = 3000
          protocol      = "tcp"
        }
      ]

      environment = [
        {
          name  = "NODE_ENV"
          value = "production"
        }
      ]

      secrets = [
        {
          name      = "DATABASE_URL"
          valueFrom = aws_secretsmanager_secret.db_url.arn
        },
        {
          name      = "API_KEY"
          valueFrom = "${aws_secretsmanager_secret.api_keys.arn}:api_key::"
        }
      ]

      logConfiguration = {
        logDriver = "awslogs"
        options = {
          "awslogs-group"         = aws_cloudwatch_log_group.api.name
          "awslogs-region"        = var.region
          "awslogs-stream-prefix" = "api"
        }
      }

      healthCheck = {
        command     = ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"]
        interval    = 30
        timeout     = 5
        retries     = 3
        startPeriod = 60
      }

      mountPoints = [
        {
          sourceVolume  = "efs-data"
          containerPath = "/app/data"
          readOnly      = false
        }
      ]
    },
    # Sidecar container for logging
    {
      name      = "log-router"
      image     = "amazon/aws-for-fluent-bit:latest"
      essential = false

      firelensConfiguration = {
        type = "fluentbit"
      }
    }
  ])

  volume {
    name = "efs-data"
    efs_volume_configuration {
      file_system_id     = aws_efs_file_system.data.id
      transit_encryption = "ENABLED"
    }
  }
}

# ECS Service
resource "aws_ecs_service" "api" {
  name                               = "api-service"
  cluster                           = aws_ecs_cluster.main.id
  task_definition                   = aws_ecs_task_definition.api.arn
  desired_count                     = 3
  launch_type                       = "FARGATE"
  platform_version                  = "LATEST"
  health_check_grace_period_seconds = 60

  network_configuration {
    subnets          = var.private_subnet_ids
    security_groups  = [aws_security_group.ecs_tasks.id]
    assign_public_ip = false
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.api.arn
    container_name   = "api"
    container_port   = 3000
  }

  service_registries {
    registry_arn = aws_service_discovery_service.api.arn
  }

  deployment_controller {
    type = "ECS"  # Or "CODE_DEPLOY" for blue-green
  }

  deployment_configuration {
    maximum_percent         = 200
    minimum_healthy_percent = 100
  }

  enable_execute_command = true  # For debugging

  lifecycle {
    ignore_changes = [desired_count]  # Managed by auto scaling
  }
}

# Auto Scaling
resource "aws_appautoscaling_target" "api" {
  max_capacity       = 20
  min_capacity       = 3
  resource_id        = "service/${aws_ecs_cluster.main.name}/${aws_ecs_service.api.name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

resource "aws_appautoscaling_policy" "api_cpu" {
  name               = "api-cpu-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.api.resource_id
  scalable_dimension = aws_appautoscaling_target.api.scalable_dimension
  service_namespace  = aws_appautoscaling_target.api.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
    target_value       = 70
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
  }
}
```

---

## 3. ECS Fargate

```hcl
# ════════════════════════════════════════════════════════════════
# ECS FARGATE - SERVERLESS CONTAINERS
# ════════════════════════════════════════════════════════════════

# Fargate task definition
resource "aws_ecs_task_definition" "fargate" {
  family                   = "api-fargate"
  network_mode             = "awsvpc"  # Required for Fargate
  requires_compatibilities = ["FARGATE"]
  
  # Fargate CPU/Memory combinations
  cpu    = 256   # 0.25 vCPU
  memory = 512   # 512 MB
  
  # Valid combinations:
  # CPU    | Memory
  # 256    | 512, 1024, 2048
  # 512    | 1024-4096
  # 1024   | 2048-8192
  # 2048   | 4096-16384
  # 4096   | 8192-30720
  # 8192   | 16384-61440
  # 16384  | 32768-122880

  execution_role_arn = aws_iam_role.ecs_execution.arn
  task_role_arn      = aws_iam_role.ecs_task.arn

  # Enable Fargate ephemeral storage (up to 200GB)
  ephemeral_storage {
    size_in_gib = 50
  }

  runtime_platform {
    operating_system_family = "LINUX"
    cpu_architecture        = "ARM64"  # Or X86_64, ARM64 is cheaper
  }

  container_definitions = jsonencode([
    {
      name      = "api"
      image     = "myregistry/api:latest"
      essential = true
      
      portMappings = [{
        containerPort = 3000
        protocol      = "tcp"
      }]

      # Resource allocation within task
      cpu    = 128  # Soft limit
      memory = 256  # Soft limit

      logConfiguration = {
        logDriver = "awslogs"
        options = {
          "awslogs-group"         = "/ecs/api"
          "awslogs-region"        = "us-east-1"
          "awslogs-stream-prefix" = "fargate"
        }
      }
    }
  ])
}

# Fargate service with Spot
resource "aws_ecs_service" "fargate_spot" {
  name            = "api-fargate-spot"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.fargate.arn
  desired_count   = 5

  # Use Fargate Spot for cost savings (up to 70%)
  capacity_provider_strategy {
    capacity_provider = "FARGATE_SPOT"
    weight            = 3  # 60% Spot
  }
  
  capacity_provider_strategy {
    capacity_provider = "FARGATE"
    weight            = 2  # 40% On-Demand
    base              = 2  # Always have 2 on-demand
  }

  network_configuration {
    subnets          = var.private_subnet_ids
    security_groups  = [aws_security_group.ecs.id]
    assign_public_ip = false
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.api.arn
    container_name   = "api"
    container_port   = 3000
  }
}
```

---

## 4. Container Networking

```hcl
# ════════════════════════════════════════════════════════════════
# ECS NETWORKING - AWSVPC MODE
# ════════════════════════════════════════════════════════════════

# Security group for ECS tasks
resource "aws_security_group" "ecs_tasks" {
  name        = "ecs-tasks-sg"
  description = "Security group for ECS tasks"
  vpc_id      = var.vpc_id

  # Allow inbound from ALB
  ingress {
    from_port       = 3000
    to_port         = 3000
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  # Allow container-to-container communication
  ingress {
    from_port = 0
    to_port   = 0
    protocol  = "-1"
    self      = true
  }

  # Allow outbound
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# VPC Endpoints for private ECS (no internet required)
resource "aws_vpc_endpoint" "ecr_api" {
  vpc_id              = var.vpc_id
  service_name        = "com.amazonaws.${var.region}.ecr.api"
  vpc_endpoint_type   = "Interface"
  private_dns_enabled = true
  subnet_ids          = var.private_subnet_ids
  security_group_ids  = [aws_security_group.vpc_endpoints.id]
}

resource "aws_vpc_endpoint" "ecr_dkr" {
  vpc_id              = var.vpc_id
  service_name        = "com.amazonaws.${var.region}.ecr.dkr"
  vpc_endpoint_type   = "Interface"
  private_dns_enabled = true
  subnet_ids          = var.private_subnet_ids
  security_group_ids  = [aws_security_group.vpc_endpoints.id]
}

resource "aws_vpc_endpoint" "s3" {
  vpc_id            = var.vpc_id
  service_name      = "com.amazonaws.${var.region}.s3"
  vpc_endpoint_type = "Gateway"
  route_table_ids   = var.private_route_table_ids
}

resource "aws_vpc_endpoint" "logs" {
  vpc_id              = var.vpc_id
  service_name        = "com.amazonaws.${var.region}.logs"
  vpc_endpoint_type   = "Interface"
  private_dns_enabled = true
  subnet_ids          = var.private_subnet_ids
  security_group_ids  = [aws_security_group.vpc_endpoints.id]
}
```

---

## 5. Service Discovery

```hcl
# ════════════════════════════════════════════════════════════════
# AWS CLOUD MAP SERVICE DISCOVERY
# ════════════════════════════════════════════════════════════════

# Private DNS namespace
resource "aws_service_discovery_private_dns_namespace" "main" {
  name        = "internal"
  description = "Internal service discovery namespace"
  vpc         = var.vpc_id
}

# Service discovery service
resource "aws_service_discovery_service" "api" {
  name = "api"

  dns_config {
    namespace_id = aws_service_discovery_private_dns_namespace.main.id

    dns_records {
      ttl  = 10
      type = "A"
    }

    routing_policy = "MULTIVALUE"
  }

  health_check_custom_config {
    failure_threshold = 1
  }
}

# ECS service with service discovery
resource "aws_ecs_service" "api" {
  name            = "api"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.api.arn
  desired_count   = 3

  service_registries {
    registry_arn   = aws_service_discovery_service.api.arn
    container_name = "api"
    container_port = 3000
  }

  # Other configuration...
}

# Now other services can reach api at:
# api.internal (resolves to task IPs)
```

```typescript
// Application code using service discovery
const http = require('http');

// Instead of hardcoded URLs:
// const API_URL = 'http://10.0.1.5:3000';

// Use service discovery DNS:
const API_URL = 'http://api.internal:3000';

async function callApiService() {
  const response = await fetch(`${API_URL}/health`);
  return response.json();
}
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# CONTAINER ORCHESTRATION PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: No health checks
# Bad
container_definitions:
  - name: api
    image: myapi:latest
    # No health check - unhealthy containers keep running!

# Good
container_definitions:
  - name: api
    image: myapi:latest
    healthCheck:
      command: ["CMD-SHELL", "curl -f http://localhost:3000/health"]
      interval: 30
      timeout: 5
      retries: 3
      startPeriod: 60

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Insufficient resources
# Bad
cpu: 128
memory: 256
# App crashes with OOM, slow performance

# Good
# Test actual usage, add buffer
cpu: 512
memory: 1024
# Monitor and adjust based on metrics

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Secrets in environment variables
# Bad
environment:
  - name: DATABASE_PASSWORD
    value: "supersecret123"  # Visible in console, logs!

# Good
secrets:
  - name: DATABASE_PASSWORD
    valueFrom: arn:aws:secretsmanager:...:db-password

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Public subnets for tasks
# Bad
network_configuration:
  subnets: public_subnets
  assign_public_ip: true  # Security risk!

# Good
network_configuration:
  subnets: private_subnets
  assign_public_ip: false
# Use ALB in public subnet, tasks in private

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: No graceful shutdown
# Bad
# Container killed immediately, drops connections

# Good
# Handle SIGTERM in application
stopTimeout: 30  # Give app time to drain connections

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Single replica in production
# Bad
desired_count: 1  # No redundancy!

# Good
desired_count: 3  # Minimum for production
deployment_configuration:
  minimum_healthy_percent: 66
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is container orchestration?"**
> "Automating deployment, scaling, networking, and management of containers across a cluster. Handles: scheduling (which host runs container), service discovery, load balancing, health checks, auto-scaling, rolling updates. Examples: Kubernetes, ECS, Docker Swarm."

**Q: "ECS vs Kubernetes?"**
> "ECS: AWS-native, simpler, deep AWS integration (IAM, ALB), Fargate for serverless. Kubernetes: Portable across clouds, more features, steeper learning curve, larger ecosystem. Use ECS for AWS-only, simpler needs. K8s for multi-cloud, complex requirements."

**Q: "What is Fargate?"**
> "Serverless compute for containers. No EC2 instances to manage - AWS handles infrastructure. Specify CPU/memory per task, pay per second of use. Good for variable workloads, simpler operations. Trade-off: higher per-unit cost than EC2."

### Intermediate Questions

**Q: "How does ECS service discovery work?"**
> "AWS Cloud Map integration. Create private DNS namespace in VPC. Register ECS service with Cloud Map. Each task gets DNS entry (task IP). Services find each other by DNS name (api.internal). Automatic registration/deregistration as tasks start/stop."

**Q: "How do you handle secrets in ECS?"**
> "AWS Secrets Manager or Parameter Store. Reference secrets in task definition. Injected as environment variables at runtime. Never in image or task definition values. IAM roles control access to secrets. Secrets Manager for rotation, Parameter Store for simpler needs."

**Q: "What is awsvpc network mode?"**
> "Each task gets its own ENI (Elastic Network Interface) with VPC IP. Task-level security groups. Direct VPC networking (no port mapping needed). Required for Fargate. Better security isolation than bridge mode."

### Advanced Questions

**Q: "How do you implement blue-green deployment in ECS?"**
> "Use CodeDeploy deployment controller. Two target groups (blue and green). Deploy new version to green. CodeDeploy shifts traffic (all-at-once, linear, or canary). Health checks validate. Automatic rollback on failures. Can also do manually with ALB listener rules."

**Q: "How do you optimize ECS costs?"**
> "Fargate Spot for fault-tolerant workloads (70% savings). Right-size CPU/memory based on actual usage. Use ARM64 (Graviton) - 20% cheaper, often faster. Savings Plans for steady-state. Auto-scaling to match demand. Spot instances for EC2 launch type."

**Q: "Design ECS architecture for a microservices application."**
> "ECS cluster per environment. Each microservice = task definition + service. ALB with path-based routing to services. awsvpc networking, private subnets. Service discovery via Cloud Map. Secrets from Secrets Manager. Centralized logging to CloudWatch. Auto-scaling per service. CI/CD with CodePipeline. Fargate for most services, EC2 for specific needs (GPU, etc.)."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                    ECS CHECKLIST                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TASK DEFINITION:                                               │
│  □ Health check configured                                     │
│  □ Resource limits set                                         │
│  □ Secrets from Secrets Manager                                │
│  □ Logging to CloudWatch                                       │
│                                                                 │
│  SERVICE:                                                       │
│  □ Multiple replicas (3+)                                      │
│  □ Load balancer configured                                    │
│  □ Auto-scaling enabled                                        │
│  □ Service discovery registered                                │
│                                                                 │
│  NETWORKING:                                                    │
│  □ awsvpc mode (Fargate requires)                              │
│  □ Private subnets                                             │
│  □ Security groups configured                                  │
│  □ VPC endpoints for ECR/S3/CloudWatch                         │
│                                                                 │
│  SECURITY:                                                      │
│  □ Execution role (pull images, write logs)                    │
│  □ Task role (app permissions)                                 │
│  □ No secrets in environment values                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

FARGATE PRICING (us-east-1):
┌────────────────────────────────────────────────────────────────┐
│ vCPU:    $0.04048 per hour                                     │
│ Memory:  $0.004445 per GB per hour                             │
│ Spot:    Up to 70% discount                                    │
│ ARM64:   20% cheaper than x86                                  │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

