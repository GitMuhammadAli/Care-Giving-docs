# Multi-Cloud Strategies - Complete Guide

> **MUST REMEMBER**: Multi-cloud means using multiple cloud providers (AWS + GCP + Azure). Reasons: avoid vendor lock-in, leverage best-of-breed services, compliance requirements, disaster recovery. Challenges: complexity, skill requirements, data transfer costs, inconsistent APIs. Use abstraction layers (Terraform, Kubernetes) for portability. Hybrid cloud = cloud + on-premises.

---

## How to Explain Like a Senior Developer

"Multi-cloud is running workloads across multiple providers - AWS, Azure, GCP, etc. The appeal: avoid lock-in, use best services from each cloud, meet compliance requirements. The reality: it's complex and expensive. You need expertise in multiple platforms, data transfer between clouds costs money, and 'portable' code still has cloud-specific pieces. Most companies are actually 'cloud-first' with one provider plus edge cases in others. If you do multi-cloud, use abstraction layers: Terraform for infrastructure, Kubernetes for compute, cloud-agnostic databases. Hybrid cloud (cloud + on-prem) is more common - companies migrate gradually while keeping some workloads on-premises. The key is being intentional: know WHY you're multi-cloud, not just doing it because it sounds good."

---

## Core Implementation

### Cloud-Agnostic Infrastructure with Terraform

```hcl
# multicloud/main.tf

# Define providers for multiple clouds
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

provider "google" {
  project = var.gcp_project
  region  = var.gcp_region
}

provider "azurerm" {
  features {}
  subscription_id = var.azure_subscription
}

# Variables for cloud selection
variable "primary_cloud" {
  type    = string
  default = "aws"
  validation {
    condition     = contains(["aws", "gcp", "azure"], var.primary_cloud)
    error_message = "Primary cloud must be aws, gcp, or azure."
  }
}

variable "aws_region" {
  default = "us-east-1"
}

variable "gcp_project" {
  default = ""
}

variable "gcp_region" {
  default = "us-central1"
}

variable "azure_subscription" {
  default = ""
}

# Abstract compute resource
module "compute" {
  source = "./modules/compute"
  
  cloud         = var.primary_cloud
  instance_type = var.instance_size
  count         = var.instance_count
}

variable "instance_size" {
  default = "small"
}

variable "instance_count" {
  default = 2
}
```

```hcl
# multicloud/modules/compute/main.tf

# Cloud-agnostic compute abstraction
variable "cloud" {
  type = string
}

variable "instance_type" {
  type = string
}

variable "count" {
  type = number
}

# Map abstract sizes to cloud-specific types
locals {
  instance_types = {
    aws = {
      small  = "t3.small"
      medium = "t3.medium"
      large  = "t3.large"
    }
    gcp = {
      small  = "e2-small"
      medium = "e2-medium"
      large  = "e2-standard-2"
    }
    azure = {
      small  = "Standard_B1s"
      medium = "Standard_B2s"
      large  = "Standard_B2ms"
    }
  }
}

# AWS EC2
resource "aws_instance" "main" {
  count         = var.cloud == "aws" ? var.count : 0
  ami           = data.aws_ami.ubuntu.id
  instance_type = local.instance_types.aws[var.instance_type]
  
  tags = {
    Name = "multicloud-instance-${count.index}"
  }
}

# GCP Compute Engine
resource "google_compute_instance" "main" {
  count        = var.cloud == "gcp" ? var.count : 0
  name         = "multicloud-instance-${count.index}"
  machine_type = local.instance_types.gcp[var.instance_type]
  zone         = "${var.gcp_region}-a"
  
  boot_disk {
    initialize_params {
      image = "ubuntu-os-cloud/ubuntu-2204-lts"
    }
  }
  
  network_interface {
    network = "default"
    access_config {}
  }
}

# Azure Virtual Machine
resource "azurerm_linux_virtual_machine" "main" {
  count               = var.cloud == "azure" ? var.count : 0
  name                = "multicloud-instance-${count.index}"
  resource_group_name = azurerm_resource_group.main[0].name
  location            = azurerm_resource_group.main[0].location
  size                = local.instance_types.azure[var.instance_type]
  admin_username      = "adminuser"
  
  network_interface_ids = [
    azurerm_network_interface.main[count.index].id
  ]
  
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }
  
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

variable "gcp_region" {
  default = "us-central1"
}

resource "azurerm_resource_group" "main" {
  count    = var.cloud == "azure" ? 1 : 0
  name     = "multicloud-rg"
  location = "East US"
}

resource "azurerm_network_interface" "main" {
  count               = var.cloud == "azure" ? var.count : 0
  name                = "multicloud-nic-${count.index}"
  location            = azurerm_resource_group.main[0].location
  resource_group_name = azurerm_resource_group.main[0].name
  
  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.main[0].id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_virtual_network" "main" {
  count               = var.cloud == "azure" ? 1 : 0
  name                = "multicloud-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main[0].location
  resource_group_name = azurerm_resource_group.main[0].name
}

resource "azurerm_subnet" "main" {
  count                = var.cloud == "azure" ? 1 : 0
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main[0].name
  virtual_network_name = azurerm_virtual_network.main[0].name
  address_prefixes     = ["10.0.1.0/24"]
}
```

### Cloud-Agnostic Application Layer

```typescript
// multicloud/storage-abstraction.ts

/**
 * Abstract storage interface that works across clouds
 */

interface CloudStorage {
  upload(key: string, data: Buffer, options?: UploadOptions): Promise<string>;
  download(key: string): Promise<Buffer>;
  delete(key: string): Promise<void>;
  list(prefix: string): Promise<string[]>;
  getSignedUrl(key: string, expiresIn: number): Promise<string>;
}

interface UploadOptions {
  contentType?: string;
  metadata?: Record<string, string>;
  isPublic?: boolean;
}

// AWS S3 Implementation
import { S3Client, PutObjectCommand, GetObjectCommand, DeleteObjectCommand, ListObjectsV2Command } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import { Readable } from 'stream';

class S3Storage implements CloudStorage {
  private client: S3Client;
  private bucket: string;
  
  constructor(bucket: string, region: string) {
    this.client = new S3Client({ region });
    this.bucket = bucket;
  }
  
  async upload(key: string, data: Buffer, options?: UploadOptions): Promise<string> {
    await this.client.send(new PutObjectCommand({
      Bucket: this.bucket,
      Key: key,
      Body: data,
      ContentType: options?.contentType,
      Metadata: options?.metadata,
      ACL: options?.isPublic ? 'public-read' : 'private',
    }));
    
    return `s3://${this.bucket}/${key}`;
  }
  
  async download(key: string): Promise<Buffer> {
    const response = await this.client.send(new GetObjectCommand({
      Bucket: this.bucket,
      Key: key,
    }));
    
    const stream = response.Body as Readable;
    const chunks: Buffer[] = [];
    for await (const chunk of stream) {
      chunks.push(chunk);
    }
    return Buffer.concat(chunks);
  }
  
  async delete(key: string): Promise<void> {
    await this.client.send(new DeleteObjectCommand({
      Bucket: this.bucket,
      Key: key,
    }));
  }
  
  async list(prefix: string): Promise<string[]> {
    const response = await this.client.send(new ListObjectsV2Command({
      Bucket: this.bucket,
      Prefix: prefix,
    }));
    
    return (response.Contents || []).map(obj => obj.Key!);
  }
  
  async getSignedUrl(key: string, expiresIn: number): Promise<string> {
    const command = new GetObjectCommand({
      Bucket: this.bucket,
      Key: key,
    });
    return getSignedUrl(this.client, command, { expiresIn });
  }
}

// GCP Cloud Storage Implementation
import { Storage } from '@google-cloud/storage';

class GCSStorage implements CloudStorage {
  private storage: Storage;
  private bucket: string;
  
  constructor(bucket: string, projectId: string) {
    this.storage = new Storage({ projectId });
    this.bucket = bucket;
  }
  
  async upload(key: string, data: Buffer, options?: UploadOptions): Promise<string> {
    const file = this.storage.bucket(this.bucket).file(key);
    
    await file.save(data, {
      contentType: options?.contentType,
      metadata: { metadata: options?.metadata },
      public: options?.isPublic,
    });
    
    return `gs://${this.bucket}/${key}`;
  }
  
  async download(key: string): Promise<Buffer> {
    const [contents] = await this.storage.bucket(this.bucket).file(key).download();
    return contents;
  }
  
  async delete(key: string): Promise<void> {
    await this.storage.bucket(this.bucket).file(key).delete();
  }
  
  async list(prefix: string): Promise<string[]> {
    const [files] = await this.storage.bucket(this.bucket).getFiles({ prefix });
    return files.map(f => f.name);
  }
  
  async getSignedUrl(key: string, expiresIn: number): Promise<string> {
    const [url] = await this.storage.bucket(this.bucket).file(key).getSignedUrl({
      action: 'read',
      expires: Date.now() + expiresIn * 1000,
    });
    return url;
  }
}

// Factory for creating cloud storage
function createStorage(config: {
  provider: 'aws' | 'gcp' | 'azure';
  bucket: string;
  region?: string;
  projectId?: string;
}): CloudStorage {
  switch (config.provider) {
    case 'aws':
      return new S3Storage(config.bucket, config.region || 'us-east-1');
    case 'gcp':
      return new GCSStorage(config.bucket, config.projectId || '');
    case 'azure':
      throw new Error('Azure implementation not shown');
    default:
      throw new Error(`Unknown provider: ${config.provider}`);
  }
}

// Usage - same code works across clouds
const storage = createStorage({
  provider: process.env.CLOUD_PROVIDER as any || 'aws',
  bucket: process.env.STORAGE_BUCKET!,
  region: process.env.CLOUD_REGION,
});

async function example() {
  await storage.upload('data/file.json', Buffer.from('{}'), {
    contentType: 'application/json',
  });
  
  const data = await storage.download('data/file.json');
  console.log(data.toString());
}
```

### Kubernetes for Portable Compute

```yaml
# multicloud/k8s/deployment.yaml

# Kubernetes provides compute portability across clouds
# Same manifests work on EKS, GKE, AKS

apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
  labels:
    app: api-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-server
  template:
    metadata:
      labels:
        app: api-server
    spec:
      containers:
      - name: api
        image: myregistry/api-server:latest
        ports:
        - containerPort: 8080
        env:
        # Cloud-agnostic config via ConfigMap/Secrets
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: database-url
        - name: STORAGE_PROVIDER
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: storage-provider
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: api-server
spec:
  selector:
    app: api-server
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP

---
# Cloud-specific ingress handled separately
# or use ingress-nginx which works across clouds
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - api.example.com
    secretName: api-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-server
            port:
              number: 80
```

---

## Real-World Scenarios

### Scenario 1: Active-Active Multi-Cloud

```typescript
// multicloud/active-active.ts

/**
 * Active-Active Multi-Cloud Architecture:
 * - Traffic split between AWS and GCP
 * - Data replicated between clouds
 * - Automatic failover on cloud issues
 */

interface CloudEndpoint {
  cloud: string;
  region: string;
  endpoint: string;
  healthy: boolean;
  latency: number;
}

class MultiCloudLoadBalancer {
  private endpoints: CloudEndpoint[] = [];
  
  constructor(endpoints: CloudEndpoint[]) {
    this.endpoints = endpoints;
    this.startHealthChecks();
  }
  
  // Route to best available endpoint
  getEndpoint(userRegion?: string): CloudEndpoint {
    const healthy = this.endpoints.filter(e => e.healthy);
    
    if (healthy.length === 0) {
      throw new Error('No healthy endpoints available');
    }
    
    // If user region provided, prefer same cloud/region
    if (userRegion) {
      const regional = healthy.find(e => e.region === userRegion);
      if (regional) return regional;
    }
    
    // Otherwise, return lowest latency
    return healthy.sort((a, b) => a.latency - b.latency)[0];
  }
  
  private async startHealthChecks(): Promise<void> {
    setInterval(async () => {
      for (const endpoint of this.endpoints) {
        endpoint.healthy = await this.checkHealth(endpoint);
        endpoint.latency = await this.measureLatency(endpoint);
      }
    }, 10000);
  }
  
  private async checkHealth(endpoint: CloudEndpoint): Promise<boolean> {
    try {
      const response = await fetch(`${endpoint.endpoint}/health`, {
        signal: AbortSignal.timeout(5000),
      });
      return response.ok;
    } catch {
      return false;
    }
  }
  
  private async measureLatency(endpoint: CloudEndpoint): Promise<number> {
    const start = Date.now();
    try {
      await fetch(`${endpoint.endpoint}/ping`, {
        signal: AbortSignal.timeout(5000),
      });
      return Date.now() - start;
    } catch {
      return Infinity;
    }
  }
}

// Data replication between clouds
class CrossCloudReplicator {
  constructor(
    private primary: CloudStorage,
    private secondary: CloudStorage
  ) {}
  
  async replicate(key: string): Promise<void> {
    const data = await this.primary.download(key);
    await this.secondary.upload(key, data);
  }
  
  async replicateAll(prefix: string): Promise<void> {
    const keys = await this.primary.list(prefix);
    
    // Replicate in parallel with rate limiting
    const batchSize = 10;
    for (let i = 0; i < keys.length; i += batchSize) {
      const batch = keys.slice(i, i + batchSize);
      await Promise.all(batch.map(key => this.replicate(key)));
    }
  }
}

interface CloudStorage {
  download(key: string): Promise<Buffer>;
  upload(key: string, data: Buffer): Promise<string>;
  list(prefix: string): Promise<string[]>;
}
```

### Scenario 2: Hybrid Cloud with On-Premises

```typescript
// multicloud/hybrid.ts

/**
 * Hybrid Cloud Pattern:
 * - Sensitive data stays on-premises
 * - Bursting to cloud for peak load
 * - Cloud for DR/backup
 */

interface DataClassification {
  PUBLIC: 'public';
  INTERNAL: 'internal';
  CONFIDENTIAL: 'confidential';
  RESTRICTED: 'restricted';
}

class HybridDataRouter {
  private onPremEndpoint: string;
  private cloudEndpoint: string;
  
  constructor(onPrem: string, cloud: string) {
    this.onPremEndpoint = onPrem;
    this.cloudEndpoint = cloud;
  }
  
  // Route based on data classification
  getEndpoint(classification: string): string {
    switch (classification) {
      case 'restricted':
      case 'confidential':
        // Sensitive data stays on-prem
        return this.onPremEndpoint;
      case 'internal':
      case 'public':
      default:
        // Non-sensitive can go to cloud
        return this.cloudEndpoint;
    }
  }
  
  // Burst to cloud when on-prem is overloaded
  async getEndpointWithBursting(
    classification: string,
    onPremLoad: number
  ): Promise<string> {
    // Always on-prem for restricted data
    if (classification === 'restricted') {
      return this.onPremEndpoint;
    }
    
    // Burst to cloud if on-prem load > 80%
    if (onPremLoad > 80) {
      console.log('Bursting to cloud due to high on-prem load');
      return this.cloudEndpoint;
    }
    
    return this.onPremEndpoint;
  }
}

// VPN/Direct Connect connectivity
interface HybridConnectivity {
  type: 'vpn' | 'direct-connect' | 'expressroute';
  bandwidth: string;
  latency: number;
  encrypted: boolean;
}

const connectivityOptions: HybridConnectivity[] = [
  {
    type: 'vpn',
    bandwidth: '1 Gbps',
    latency: 50, // ms
    encrypted: true,
  },
  {
    type: 'direct-connect',
    bandwidth: '10 Gbps',
    latency: 5, // ms
    encrypted: false, // Encrypt at application layer
  },
];
```

---

## Common Pitfalls

### 1. Assuming Easy Portability

```typescript
// ❌ BAD: Using cloud-specific services everywhere
import { DynamoDB } from '@aws-sdk/client-dynamodb';
// This code is locked to AWS

// ✅ GOOD: Abstract cloud-specific code
interface Database {
  get(key: string): Promise<any>;
  put(key: string, value: any): Promise<void>;
}

// Implementations for each cloud
class DynamoDBAdapter implements Database {
  async get(key: string) { /* ... */ return {}; }
  async put(key: string, value: any) { /* ... */ }
}

class FirestoreAdapter implements Database {
  async get(key: string) { /* ... */ return {}; }
  async put(key: string, value: any) { /* ... */ }
}
```

### 2. Ignoring Data Transfer Costs

```typescript
// ❌ BAD: Frequent cross-cloud data transfer
// AWS us-east-1 ↔ GCP us-central1
// ~$0.12/GB each way = expensive!

// ✅ GOOD: Minimize cross-cloud traffic
// - Keep data close to compute
// - Replicate asynchronously
// - Compress before transfer
// - Cache frequently accessed data locally
```

### 3. Building Multi-Cloud Without Need

```typescript
// ❌ BAD: Multi-cloud for "flexibility"
// Result: 2x complexity, 2x skills needed, 2x cost

// ✅ GOOD: Be intentional about multi-cloud
// Valid reasons:
// - Regulatory requirement
// - Specific service only on one cloud
// - Genuine lock-in risk for your workload
// - M&A brought in different cloud stack
```

---

## Interview Questions

### Q1: Why would a company choose multi-cloud?

**A:** 1) Avoid vendor lock-in (negotiation leverage, risk mitigation). 2) Best-of-breed services (GCP for ML, AWS for breadth). 3) Compliance requirements (data sovereignty, government mandates). 4) Disaster recovery across providers. 5) Acquisitions bringing different stacks. However, it adds complexity and cost, so benefits must outweigh costs.

### Q2: How do you make applications portable across clouds?

**A:** 1) Use Kubernetes for compute (EKS/GKE/AKS). 2) Abstract cloud services behind interfaces. 3) Use Terraform for infrastructure. 4) Use cloud-agnostic databases (PostgreSQL, MongoDB) when possible. 5) Containerize everything. 6) Externalize configuration. Not everything should be portable - evaluate case by case.

### Q3: What is the difference between multi-cloud and hybrid cloud?

**A:** **Multi-cloud**: Using multiple public cloud providers (AWS + GCP). **Hybrid cloud**: Combining public cloud with private cloud or on-premises infrastructure. You can have both: multi-cloud hybrid (AWS + Azure + on-prem). Hybrid is more common as companies migrate gradually.

### Q4: What are the main challenges of multi-cloud?

**A:** 1) Skill requirements (teams need to know multiple platforms). 2) Management complexity (different consoles, APIs, billing). 3) Data transfer costs between clouds. 4) Security consistency across providers. 5) Networking complexity (cross-cloud connectivity). 6) Support and troubleshooting across vendors.

---

## Quick Reference Checklist

### Strategy
- [ ] Define clear reasons for multi-cloud
- [ ] Calculate total cost (not just compute)
- [ ] Assess team skills
- [ ] Identify which workloads benefit

### Portability
- [ ] Use containers (Docker, Kubernetes)
- [ ] Abstract cloud services
- [ ] Infrastructure as Code (Terraform)
- [ ] Cloud-agnostic databases where possible

### Connectivity
- [ ] Plan cross-cloud networking
- [ ] Consider data transfer costs
- [ ] Implement consistent security
- [ ] Set up unified monitoring

### Operations
- [ ] Unified logging/monitoring
- [ ] Consistent deployment pipelines
- [ ] Cross-cloud disaster recovery
- [ ] Cost tracking per cloud

---

*Last updated: February 2026*

