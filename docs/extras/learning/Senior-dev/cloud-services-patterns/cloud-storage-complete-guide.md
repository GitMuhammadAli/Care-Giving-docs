# Cloud Storage - Complete Guide

> **MUST REMEMBER**: Object storage (S3, GCS, Azure Blob) is infinitely scalable, cheap for storage, expensive for requests. Use presigned URLs for secure direct upload/download. Lifecycle policies automate moving to cheaper tiers or deletion. Storage classes: Standard (frequent access), IA (infrequent), Glacier (archive). Enable versioning for important data, encryption at rest by default.

---

## How to Explain Like a Senior Developer

"Object storage is the workhorse of cloud architecture - it's where you put files, backups, logs, static assets. Unlike file systems, it's flat (no directories, just keys with slashes) and infinitely scalable. The cost model is storage + requests + bandwidth - storage is cheap, but millions of small GET requests add up. Presigned URLs let clients upload/download directly to S3, bypassing your servers. Lifecycle policies are essential: move logs to Infrequent Access after 30 days, Glacier after 90, delete after a year. Always enable versioning for user data - it's saved me countless times. Use multipart upload for large files and consider S3 Transfer Acceleration for global users."

---

## Core Implementation

### Presigned URLs for Direct Upload

```typescript
// storage/presigned-urls.ts
import {
  S3Client,
  PutObjectCommand,
  GetObjectCommand,
} from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import { randomUUID } from 'crypto';

const s3 = new S3Client({ region: process.env.AWS_REGION });
const BUCKET = process.env.BUCKET_NAME!;

interface UploadConfig {
  contentType: string;
  maxSize?: number;  // bytes
  expiresIn?: number; // seconds
  metadata?: Record<string, string>;
}

interface PresignedUpload {
  uploadUrl: string;
  key: string;
  expiresAt: Date;
}

// Generate presigned URL for upload
async function createUploadUrl(
  userId: string,
  filename: string,
  config: UploadConfig
): Promise<PresignedUpload> {
  // Generate unique key with user prefix
  const fileId = randomUUID();
  const extension = filename.split('.').pop() || '';
  const key = `uploads/${userId}/${fileId}.${extension}`;
  
  const expiresIn = config.expiresIn || 3600; // 1 hour default
  
  const command = new PutObjectCommand({
    Bucket: BUCKET,
    Key: key,
    ContentType: config.contentType,
    Metadata: {
      'user-id': userId,
      'original-name': filename,
      ...config.metadata,
    },
  });
  
  const uploadUrl = await getSignedUrl(s3, command, { expiresIn });
  
  return {
    uploadUrl,
    key,
    expiresAt: new Date(Date.now() + expiresIn * 1000),
  };
}

// Generate presigned URL for download
async function createDownloadUrl(
  key: string,
  options?: {
    expiresIn?: number;
    filename?: string; // Force download with this filename
  }
): Promise<string> {
  const command = new GetObjectCommand({
    Bucket: BUCKET,
    Key: key,
    ResponseContentDisposition: options?.filename
      ? `attachment; filename="${options.filename}"`
      : undefined,
  });
  
  return getSignedUrl(s3, command, {
    expiresIn: options?.expiresIn || 3600,
  });
}

// API endpoint for upload URL
import { APIGatewayProxyHandler } from 'aws-lambda';

export const getUploadUrl: APIGatewayProxyHandler = async (event) => {
  const userId = event.requestContext.authorizer?.claims?.sub;
  const body = JSON.parse(event.body || '{}');
  
  // Validate content type
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'application/pdf'];
  if (!allowedTypes.includes(body.contentType)) {
    return {
      statusCode: 400,
      body: JSON.stringify({ error: 'Invalid content type' }),
    };
  }
  
  const upload = await createUploadUrl(userId, body.filename, {
    contentType: body.contentType,
    maxSize: 10 * 1024 * 1024, // 10MB
  });
  
  return {
    statusCode: 200,
    body: JSON.stringify(upload),
  };
};

// Client-side upload
async function uploadFile(file: File): Promise<string> {
  // 1. Get presigned URL from our API
  const response = await fetch('/api/upload-url', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      filename: file.name,
      contentType: file.type,
    }),
  });
  
  const { uploadUrl, key } = await response.json();
  
  // 2. Upload directly to S3
  await fetch(uploadUrl, {
    method: 'PUT',
    headers: { 'Content-Type': file.type },
    body: file,
  });
  
  return key;
}
```

### Multipart Upload for Large Files

```typescript
// storage/multipart-upload.ts
import {
  S3Client,
  CreateMultipartUploadCommand,
  UploadPartCommand,
  CompleteMultipartUploadCommand,
  AbortMultipartUploadCommand,
  CompletedPart,
} from '@aws-sdk/client-s3';
import { Readable } from 'stream';

const s3 = new S3Client({ region: process.env.AWS_REGION });
const BUCKET = process.env.BUCKET_NAME!;
const PART_SIZE = 10 * 1024 * 1024; // 10MB parts

interface MultipartUploadProgress {
  uploaded: number;
  total: number;
  percentage: number;
}

async function uploadLargeFile(
  key: string,
  data: Buffer,
  contentType: string,
  onProgress?: (progress: MultipartUploadProgress) => void
): Promise<void> {
  // Initiate multipart upload
  const createResponse = await s3.send(new CreateMultipartUploadCommand({
    Bucket: BUCKET,
    Key: key,
    ContentType: contentType,
  }));
  
  const uploadId = createResponse.UploadId!;
  
  try {
    const parts: CompletedPart[] = [];
    const totalParts = Math.ceil(data.length / PART_SIZE);
    
    for (let partNumber = 1; partNumber <= totalParts; partNumber++) {
      const start = (partNumber - 1) * PART_SIZE;
      const end = Math.min(partNumber * PART_SIZE, data.length);
      const partData = data.subarray(start, end);
      
      // Upload part
      const uploadResponse = await s3.send(new UploadPartCommand({
        Bucket: BUCKET,
        Key: key,
        UploadId: uploadId,
        PartNumber: partNumber,
        Body: partData,
      }));
      
      parts.push({
        PartNumber: partNumber,
        ETag: uploadResponse.ETag,
      });
      
      // Report progress
      onProgress?.({
        uploaded: end,
        total: data.length,
        percentage: Math.round((end / data.length) * 100),
      });
    }
    
    // Complete upload
    await s3.send(new CompleteMultipartUploadCommand({
      Bucket: BUCKET,
      Key: key,
      UploadId: uploadId,
      MultipartUpload: { Parts: parts },
    }));
    
  } catch (error) {
    // Abort on failure
    await s3.send(new AbortMultipartUploadCommand({
      Bucket: BUCKET,
      Key: key,
      UploadId: uploadId,
    }));
    throw error;
  }
}

// Resumable upload with presigned URLs
interface ResumableUpload {
  uploadId: string;
  key: string;
  partUrls: Array<{
    partNumber: number;
    url: string;
  }>;
}

async function createResumableUpload(
  key: string,
  fileSize: number,
  contentType: string
): Promise<ResumableUpload> {
  const { getSignedUrl } = await import('@aws-sdk/s3-request-presigner');
  
  // Create multipart upload
  const createResponse = await s3.send(new CreateMultipartUploadCommand({
    Bucket: BUCKET,
    Key: key,
    ContentType: contentType,
  }));
  
  const uploadId = createResponse.UploadId!;
  const totalParts = Math.ceil(fileSize / PART_SIZE);
  
  // Generate presigned URLs for each part
  const partUrls = await Promise.all(
    Array.from({ length: totalParts }, async (_, i) => {
      const partNumber = i + 1;
      const command = new UploadPartCommand({
        Bucket: BUCKET,
        Key: key,
        UploadId: uploadId,
        PartNumber: partNumber,
      });
      
      const url = await getSignedUrl(s3, command, { expiresIn: 86400 });
      
      return { partNumber, url };
    })
  );
  
  return {
    uploadId,
    key,
    partUrls,
  };
}
```

### Lifecycle Policies

```typescript
// storage/lifecycle-policies.ts

/**
 * S3 Lifecycle Policy Configuration
 * 
 * Automatically:
 * - Transition to cheaper storage classes
 * - Delete old/incomplete uploads
 * - Expire temporary files
 */

import {
  S3Client,
  PutBucketLifecycleConfigurationCommand,
} from '@aws-sdk/client-s3';

const s3 = new S3Client({ region: process.env.AWS_REGION });

async function configureLifecyclePolicy(bucket: string): Promise<void> {
  await s3.send(new PutBucketLifecycleConfigurationCommand({
    Bucket: bucket,
    LifecycleConfiguration: {
      Rules: [
        // User uploads: keep accessible
        {
          ID: 'UserUploads',
          Filter: { Prefix: 'uploads/' },
          Status: 'Enabled',
          Transitions: [
            {
              Days: 30,
              StorageClass: 'STANDARD_IA', // Infrequent Access
            },
            {
              Days: 90,
              StorageClass: 'GLACIER_IR', // Glacier Instant Retrieval
            },
          ],
        },
        
        // Logs: archive and delete
        {
          ID: 'Logs',
          Filter: { Prefix: 'logs/' },
          Status: 'Enabled',
          Transitions: [
            {
              Days: 7,
              StorageClass: 'STANDARD_IA',
            },
            {
              Days: 30,
              StorageClass: 'GLACIER',
            },
          ],
          Expiration: {
            Days: 365, // Delete after 1 year
          },
        },
        
        // Temp files: delete quickly
        {
          ID: 'TempFiles',
          Filter: { Prefix: 'temp/' },
          Status: 'Enabled',
          Expiration: {
            Days: 1,
          },
        },
        
        // Clean up incomplete multipart uploads
        {
          ID: 'AbortIncompleteMultipartUpload',
          Filter: { Prefix: '' },
          Status: 'Enabled',
          AbortIncompleteMultipartUpload: {
            DaysAfterInitiation: 7,
          },
        },
        
        // Delete old versions
        {
          ID: 'DeleteOldVersions',
          Filter: { Prefix: '' },
          Status: 'Enabled',
          NoncurrentVersionTransitions: [
            {
              NoncurrentDays: 30,
              StorageClass: 'STANDARD_IA',
            },
          ],
          NoncurrentVersionExpiration: {
            NoncurrentDays: 90,
          },
        },
        
        // Delete expired delete markers
        {
          ID: 'CleanupDeleteMarkers',
          Filter: { Prefix: '' },
          Status: 'Enabled',
          Expiration: {
            ExpiredObjectDeleteMarker: true,
          },
        },
      ],
    },
  }));
}

// Terraform equivalent
const terraformConfig = `
resource "aws_s3_bucket_lifecycle_configuration" "main" {
  bucket = aws_s3_bucket.main.id

  rule {
    id     = "user-uploads"
    status = "Enabled"

    filter {
      prefix = "uploads/"
    }

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    transition {
      days          = 90
      storage_class = "GLACIER_IR"
    }
  }

  rule {
    id     = "logs"
    status = "Enabled"

    filter {
      prefix = "logs/"
    }

    transition {
      days          = 7
      storage_class = "STANDARD_IA"
    }

    expiration {
      days = 365
    }
  }

  rule {
    id     = "abort-incomplete-uploads"
    status = "Enabled"

    abort_incomplete_multipart_upload {
      days_after_initiation = 7
    }
  }
}
`;
```

### Storage Classes and Cost Optimization

```typescript
// storage/cost-optimization.ts

/**
 * S3 Storage Classes (as of 2024):
 * 
 * Standard           - Frequent access, highest availability
 * Standard-IA        - Infrequent access, 30-day minimum
 * One Zone-IA        - Infrequent, single AZ (cheaper, less durable)
 * Intelligent-Tiering - Auto-moves between tiers
 * Glacier Instant    - Archive with millisecond access
 * Glacier Flexible   - Archive, minutes to hours retrieval
 * Glacier Deep       - Cheapest, 12+ hour retrieval
 */

import {
  S3Client,
  PutObjectCommand,
  CopyObjectCommand,
  RestoreObjectCommand,
} from '@aws-sdk/client-s3';

const s3 = new S3Client({ region: process.env.AWS_REGION });

// Upload with specific storage class
async function uploadWithStorageClass(
  bucket: string,
  key: string,
  body: Buffer,
  storageClass: 'STANDARD' | 'STANDARD_IA' | 'ONEZONE_IA' | 'INTELLIGENT_TIERING' | 'GLACIER'
): Promise<void> {
  await s3.send(new PutObjectCommand({
    Bucket: bucket,
    Key: key,
    Body: body,
    StorageClass: storageClass,
  }));
}

// Change storage class of existing object
async function changeStorageClass(
  bucket: string,
  key: string,
  newStorageClass: string
): Promise<void> {
  await s3.send(new CopyObjectCommand({
    Bucket: bucket,
    CopySource: `${bucket}/${key}`,
    Key: key,
    StorageClass: newStorageClass,
    MetadataDirective: 'COPY',
  }));
}

// Restore from Glacier
async function restoreFromGlacier(
  bucket: string,
  key: string,
  days: number,
  tier: 'Expedited' | 'Standard' | 'Bulk' = 'Standard'
): Promise<void> {
  await s3.send(new RestoreObjectCommand({
    Bucket: bucket,
    Key: key,
    RestoreRequest: {
      Days: days, // How long to keep restored copy
      GlacierJobParameters: {
        Tier: tier,
        // Expedited: 1-5 minutes, expensive
        // Standard: 3-5 hours
        // Bulk: 5-12 hours, cheapest
      },
    },
  }));
}

// Cost analysis
interface StorageCostEstimate {
  storageClass: string;
  storageCostPerGB: number;
  retrievalCostPerGB: number;
  requestCostPer1000: number;
  minimumStorageDays: number;
}

const storageCosts: StorageCostEstimate[] = [
  {
    storageClass: 'STANDARD',
    storageCostPerGB: 0.023,
    retrievalCostPerGB: 0,
    requestCostPer1000: 0.0004,
    minimumStorageDays: 0,
  },
  {
    storageClass: 'STANDARD_IA',
    storageCostPerGB: 0.0125,
    retrievalCostPerGB: 0.01,
    requestCostPer1000: 0.001,
    minimumStorageDays: 30,
  },
  {
    storageClass: 'GLACIER_INSTANT',
    storageCostPerGB: 0.004,
    retrievalCostPerGB: 0.03,
    requestCostPer1000: 0.01,
    minimumStorageDays: 90,
  },
  {
    storageClass: 'GLACIER_DEEP',
    storageCostPerGB: 0.00099,
    retrievalCostPerGB: 0.02,
    requestCostPer1000: 0.025,
    minimumStorageDays: 180,
  },
];

function estimateMonthlyCost(
  sizeGB: number,
  accessesPerMonth: number,
  storageClass: string
): number {
  const costs = storageCosts.find(c => c.storageClass === storageClass);
  if (!costs) throw new Error('Unknown storage class');
  
  const storageCost = sizeGB * costs.storageCostPerGB;
  const retrievalCost = (sizeGB * accessesPerMonth / 30) * costs.retrievalCostPerGB;
  const requestCost = (accessesPerMonth / 1000) * costs.requestCostPer1000;
  
  return storageCost + retrievalCost + requestCost;
}
```

---

## Real-World Scenarios

### Scenario 1: Image Upload and Processing Pipeline

```typescript
// storage/image-pipeline.ts

/**
 * Pipeline:
 * 1. Client gets presigned URL
 * 2. Client uploads directly to S3
 * 3. S3 triggers Lambda
 * 4. Lambda processes (resize, thumbnail)
 * 5. Lambda saves processed versions
 * 6. Client is notified (WebSocket/webhook)
 */

import { S3Event, S3Handler } from 'aws-lambda';
import {
  S3Client,
  GetObjectCommand,
  PutObjectCommand,
} from '@aws-sdk/client-s3';
import sharp from 'sharp';

const s3 = new S3Client({});

interface ImageVariant {
  suffix: string;
  width: number;
  height?: number;
  quality: number;
  format: 'jpeg' | 'webp' | 'png';
}

const variants: ImageVariant[] = [
  { suffix: 'thumb', width: 150, height: 150, quality: 80, format: 'webp' },
  { suffix: 'small', width: 400, quality: 85, format: 'webp' },
  { suffix: 'medium', width: 800, quality: 85, format: 'webp' },
  { suffix: 'large', width: 1200, quality: 85, format: 'webp' },
];

export const processImage: S3Handler = async (event: S3Event) => {
  for (const record of event.Records) {
    const bucket = record.s3.bucket.name;
    const key = decodeURIComponent(record.s3.object.key.replace(/\+/g, ' '));
    
    // Skip if already processed
    if (key.includes('/processed/')) continue;
    
    try {
      // Download original
      const response = await s3.send(new GetObjectCommand({
        Bucket: bucket,
        Key: key,
      }));
      
      const originalBuffer = await streamToBuffer(response.Body as any);
      
      // Generate variants
      const baseKey = key.replace(/\.[^.]+$/, '');
      
      await Promise.all(
        variants.map(async (variant) => {
          let pipeline = sharp(originalBuffer);
          
          if (variant.height) {
            pipeline = pipeline.resize(variant.width, variant.height, {
              fit: 'cover',
            });
          } else {
            pipeline = pipeline.resize(variant.width, undefined, {
              withoutEnlargement: true,
            });
          }
          
          pipeline = pipeline[variant.format]({ quality: variant.quality });
          
          const processedBuffer = await pipeline.toBuffer();
          
          const processedKey = `${baseKey}/processed/${variant.suffix}.${variant.format}`;
          
          await s3.send(new PutObjectCommand({
            Bucket: bucket,
            Key: processedKey,
            Body: processedBuffer,
            ContentType: `image/${variant.format}`,
            CacheControl: 'public, max-age=31536000',
          }));
        })
      );
      
      console.log(`Processed ${key} into ${variants.length} variants`);
      
    } catch (error) {
      console.error(`Failed to process ${key}:`, error);
      throw error;
    }
  }
};

async function streamToBuffer(stream: any): Promise<Buffer> {
  const chunks: Buffer[] = [];
  for await (const chunk of stream) {
    chunks.push(chunk);
  }
  return Buffer.concat(chunks);
}
```

### Scenario 2: Static Website Hosting with CDN

```typescript
// storage/static-hosting.ts

/**
 * S3 + CloudFront for static website:
 * - S3 bucket for storage
 * - CloudFront for CDN/HTTPS
 * - Route 53 for DNS
 * - ACM for SSL certificate
 */

// Terraform configuration
const terraformConfig = `
# S3 Bucket for website
resource "aws_s3_bucket" "website" {
  bucket = "my-website-bucket"
}

resource "aws_s3_bucket_website_configuration" "website" {
  bucket = aws_s3_bucket.website.id

  index_document {
    suffix = "index.html"
  }

  error_document {
    key = "404.html"
  }
}

# Block public access (CloudFront will access via OAI)
resource "aws_s3_bucket_public_access_block" "website" {
  bucket = aws_s3_bucket.website.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# CloudFront Origin Access Identity
resource "aws_cloudfront_origin_access_identity" "website" {
  comment = "OAI for website"
}

# Bucket policy for CloudFront access
resource "aws_s3_bucket_policy" "website" {
  bucket = aws_s3_bucket.website.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          AWS = aws_cloudfront_origin_access_identity.website.iam_arn
        }
        Action   = "s3:GetObject"
        Resource = "\${aws_s3_bucket.website.arn}/*"
      }
    ]
  })
}

# CloudFront Distribution
resource "aws_cloudfront_distribution" "website" {
  enabled             = true
  default_root_object = "index.html"
  aliases             = ["www.example.com"]

  origin {
    domain_name = aws_s3_bucket.website.bucket_regional_domain_name
    origin_id   = "S3-website"

    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.website.cloudfront_access_identity_path
    }
  }

  default_cache_behavior {
    allowed_methods        = ["GET", "HEAD", "OPTIONS"]
    cached_methods         = ["GET", "HEAD"]
    target_origin_id       = "S3-website"
    viewer_protocol_policy = "redirect-to-https"
    compress               = true

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }

    min_ttl     = 0
    default_ttl = 86400
    max_ttl     = 31536000
  }

  # SPA routing - return index.html for 404s
  custom_error_response {
    error_code         = 404
    response_code      = 200
    response_page_path = "/index.html"
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    acm_certificate_arn      = aws_acm_certificate.website.arn
    ssl_support_method       = "sni-only"
    minimum_protocol_version = "TLSv1.2_2021"
  }
}
`;
```

---

## Common Pitfalls

### 1. Not Using Presigned URLs

```typescript
// ❌ BAD: Proxying file through your server
app.post('/upload', async (req, res) => {
  const file = req.file;
  await s3.putObject({ Body: file.buffer }); // Double bandwidth!
});

// ✅ GOOD: Direct upload with presigned URL
app.get('/upload-url', async (req, res) => {
  const url = await getSignedUrl(s3, new PutObjectCommand({...}));
  res.json({ url }); // Client uploads directly to S3
});

import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import express from 'express';
const app = express();
const s3 = new S3Client({});
```

### 2. Ignoring Storage Class Costs

```typescript
// ❌ BAD: Everything in Standard
await s3.send(new PutObjectCommand({
  Key: 'logs/2024-01-01.json', // Rarely accessed, should be IA!
  Body: logData,
}));

// ✅ GOOD: Use appropriate storage class
await s3.send(new PutObjectCommand({
  Key: 'logs/2024-01-01.json',
  Body: logData,
  StorageClass: 'STANDARD_IA', // 50% cheaper storage
}));

const logData = '';
```

### 3. Not Handling Incomplete Uploads

```typescript
// ❌ BAD: No cleanup of failed multipart uploads
// Incomplete uploads accumulate and cost money!

// ✅ GOOD: Lifecycle rule to abort incomplete uploads
const lifecycleRule = {
  ID: 'AbortIncompleteMultipartUpload',
  Status: 'Enabled',
  AbortIncompleteMultipartUpload: {
    DaysAfterInitiation: 7,
  },
};
```

---

## Interview Questions

### Q1: When would you use presigned URLs vs proxying through your server?

**A:** **Presigned URLs** for: large files (avoid double bandwidth), direct uploads/downloads, reducing server load, better performance. **Proxy** for: small files, when you need to process/validate on upload, when you need to track/log all access, when clients can't make direct S3 calls.

### Q2: How do you choose the right storage class?

**A:** Based on access patterns and cost. Standard for frequently accessed data. Standard-IA for data accessed less than once a month but needs instant access. Glacier Instant for archival with rare but immediate access needs. Glacier Deep Archive for compliance/backup data rarely accessed. Use Intelligent-Tiering when access patterns are unpredictable.

### Q3: How do you secure S3 buckets?

**A:** 1) Block public access by default. 2) Use bucket policies and IAM for access control. 3) Enable encryption (SSE-S3 or SSE-KMS). 4) Enable versioning and MFA delete for important data. 5) Use VPC endpoints for private access. 6) Enable access logging. 7) Use presigned URLs for temporary access.

### Q4: How do you handle large file uploads?

**A:** Use multipart upload - split file into parts (5MB-5GB each), upload in parallel, complete when all parts uploaded. Benefits: resume on failure, parallel upload for speed, required for files >5GB. For client uploads, generate presigned URLs for each part.

---

## Quick Reference Checklist

### Upload/Download
- [ ] Use presigned URLs for direct access
- [ ] Implement multipart upload for large files
- [ ] Set appropriate content types
- [ ] Configure CORS if needed

### Cost Optimization
- [ ] Configure lifecycle policies
- [ ] Use appropriate storage classes
- [ ] Clean up incomplete uploads
- [ ] Monitor storage metrics

### Security
- [ ] Block public access
- [ ] Enable encryption at rest
- [ ] Use IAM policies
- [ ] Enable access logging

### Performance
- [ ] Use CloudFront for distribution
- [ ] Enable transfer acceleration
- [ ] Compress before upload
- [ ] Use parallel uploads

---

*Last updated: February 2026*

