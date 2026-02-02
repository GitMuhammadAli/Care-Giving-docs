# Cloud Security - Complete Guide

> **MUST REMEMBER**: IAM is identity and access management - users, roles, policies. Principle of least privilege: grant minimum permissions needed. Use roles (not access keys) for services. Policies: identity-based (attached to user/role) vs resource-based (attached to S3/Lambda). Always use MFA, rotate credentials, encrypt at rest and in transit. CloudTrail for audit, GuardDuty for threat detection.

---

## How to Explain Like a Senior Developer

"Cloud security is about identity, permissions, and encryption. IAM answers 'who can do what to which resources'. Users are for humans, roles are for services - never embed access keys in code, use roles instead. Policies define permissions: Allow/Deny + Action + Resource + Conditions. Start with zero permissions and add what's needed - least privilege. Encrypt everything: at rest (KMS), in transit (TLS). For auditing: CloudTrail logs every API call, Config tracks resource changes, GuardDuty detects threats. The shared responsibility model: AWS secures the cloud (physical, hypervisor), you secure what's IN the cloud (data, access, configuration)."

---

## Core Implementation

### IAM Policies and Roles

```json
// security/iam-policies.json

// Example: Minimal policy for Lambda accessing DynamoDB
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DynamoDBTableAccess",
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem",
        "dynamodb:Query"
      ],
      "Resource": [
        "arn:aws:dynamodb:us-east-1:123456789:table/MyTable",
        "arn:aws:dynamodb:us-east-1:123456789:table/MyTable/index/*"
      ]
    },
    {
      "Sid": "CloudWatchLogs",
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:us-east-1:123456789:log-group:/aws/lambda/my-function:*"
    }
  ]
}

// Example: S3 bucket policy (resource-based)
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontAccess",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::123456789:distribution/ABCD1234"
        }
      }
    },
    {
      "Sid": "DenyUnencryptedUploads",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "aws:kms"
        }
      }
    }
  ]
}

// Example: Cross-account role assumption
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::OTHER_ACCOUNT_ID:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "unique-external-id"
        },
        "Bool": {
          "aws:MultiFactorAuthPresent": "true"
        }
      }
    }
  ]
}
```

### IAM with Terraform

```hcl
# security/iam.tf

# Lambda execution role
resource "aws_iam_role" "lambda_execution" {
  name = "lambda-execution-role"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "lambda.amazonaws.com"
        }
      }
    ]
  })
}

# Policy for Lambda to access specific resources
resource "aws_iam_role_policy" "lambda_policy" {
  name = "lambda-policy"
  role = aws_iam_role.lambda_execution.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      # DynamoDB access - specific table only
      {
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:PutItem",
          "dynamodb:Query"
        ]
        Resource = [
          aws_dynamodb_table.main.arn,
          "${aws_dynamodb_table.main.arn}/index/*"
        ]
      },
      # S3 access - specific bucket and prefix
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject"
        ]
        Resource = "${aws_s3_bucket.data.arn}/uploads/*"
      },
      # Secrets Manager - specific secret
      {
        Effect = "Allow"
        Action = "secretsmanager:GetSecretValue"
        Resource = aws_secretsmanager_secret.api_key.arn
      },
      # KMS for decryption
      {
        Effect = "Allow"
        Action = [
          "kms:Decrypt",
          "kms:GenerateDataKey"
        ]
        Resource = aws_kms_key.main.arn
      }
    ]
  })
}

# Attach managed policy for basic Lambda execution
resource "aws_iam_role_policy_attachment" "lambda_basic" {
  role       = aws_iam_role.lambda_execution.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

# VPC access for Lambda (if needed)
resource "aws_iam_role_policy_attachment" "lambda_vpc" {
  role       = aws_iam_role.lambda_execution.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole"
}
```

### Secrets Management

```typescript
// security/secrets.ts
import {
  SecretsManagerClient,
  GetSecretValueCommand,
  CreateSecretCommand,
  UpdateSecretCommand,
  RotateSecretCommand,
} from '@aws-sdk/client-secrets-manager';

const secretsManager = new SecretsManagerClient({
  region: process.env.AWS_REGION,
});

// Cache secrets to avoid repeated API calls
const secretsCache = new Map<string, { value: any; expiresAt: number }>();
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes

async function getSecret(secretName: string): Promise<any> {
  // Check cache
  const cached = secretsCache.get(secretName);
  if (cached && cached.expiresAt > Date.now()) {
    return cached.value;
  }
  
  // Fetch from Secrets Manager
  const response = await secretsManager.send(
    new GetSecretValueCommand({
      SecretId: secretName,
    })
  );
  
  const value = response.SecretString
    ? JSON.parse(response.SecretString)
    : response.SecretBinary;
  
  // Cache the result
  secretsCache.set(secretName, {
    value,
    expiresAt: Date.now() + CACHE_TTL,
  });
  
  return value;
}

// Usage in application
async function getDatabaseCredentials(): Promise<{
  username: string;
  password: string;
  host: string;
}> {
  return getSecret('prod/database/credentials');
}

// Create secret with automatic rotation
async function createRotatableSecret(
  name: string,
  value: object,
  rotationLambdaArn: string
): Promise<void> {
  // Create secret
  await secretsManager.send(
    new CreateSecretCommand({
      Name: name,
      SecretString: JSON.stringify(value),
      KmsKeyId: process.env.KMS_KEY_ID,
    })
  );
  
  // Enable rotation
  await secretsManager.send(
    new RotateSecretCommand({
      SecretId: name,
      RotationLambdaARN: rotationLambdaArn,
      RotationRules: {
        AutomaticallyAfterDays: 30,
      },
    })
  );
}

// Rotation Lambda handler
import { SecretsManagerRotationEvent } from 'aws-lambda';

export async function rotateSecret(
  event: SecretsManagerRotationEvent
): Promise<void> {
  const { SecretId, ClientRequestToken, Step } = event;
  
  switch (Step) {
    case 'createSecret':
      await createNewSecretVersion(SecretId, ClientRequestToken);
      break;
    case 'setSecret':
      await setSecretInService(SecretId, ClientRequestToken);
      break;
    case 'testSecret':
      await testSecret(SecretId, ClientRequestToken);
      break;
    case 'finishSecret':
      await finishRotation(SecretId, ClientRequestToken);
      break;
  }
}

async function createNewSecretVersion(
  secretId: string,
  token: string
): Promise<void> {
  // Generate new password
  const newPassword = generateSecurePassword();
  
  await secretsManager.send(
    new UpdateSecretCommand({
      SecretId: secretId,
      ClientRequestToken: token,
      SecretString: JSON.stringify({ password: newPassword }),
    })
  );
}

function generateSecurePassword(): string {
  const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*';
  const array = new Uint8Array(32);
  crypto.getRandomValues(array);
  return Array.from(array, x => chars[x % chars.length]).join('');
}

async function setSecretInService(secretId: string, token: string): Promise<void> {}
async function testSecret(secretId: string, token: string): Promise<void> {}
async function finishRotation(secretId: string, token: string): Promise<void> {}
```

### Encryption with KMS

```typescript
// security/encryption.ts
import {
  KMSClient,
  EncryptCommand,
  DecryptCommand,
  GenerateDataKeyCommand,
} from '@aws-sdk/client-kms';
import * as crypto from 'crypto';

const kms = new KMSClient({ region: process.env.AWS_REGION });
const KMS_KEY_ID = process.env.KMS_KEY_ID!;

// Encrypt small data directly with KMS
async function kmsEncrypt(plaintext: string): Promise<string> {
  const response = await kms.send(
    new EncryptCommand({
      KeyId: KMS_KEY_ID,
      Plaintext: Buffer.from(plaintext),
    })
  );
  
  return Buffer.from(response.CiphertextBlob!).toString('base64');
}

// Decrypt KMS-encrypted data
async function kmsDecrypt(ciphertext: string): Promise<string> {
  const response = await kms.send(
    new DecryptCommand({
      CiphertextBlob: Buffer.from(ciphertext, 'base64'),
    })
  );
  
  return Buffer.from(response.Plaintext!).toString();
}

// Envelope encryption for large data
// KMS encrypts a data key, data key encrypts the actual data
async function envelopeEncrypt(
  plaintext: Buffer
): Promise<{ encryptedData: string; encryptedKey: string }> {
  // Generate data key
  const response = await kms.send(
    new GenerateDataKeyCommand({
      KeyId: KMS_KEY_ID,
      KeySpec: 'AES_256',
    })
  );
  
  const dataKey = response.Plaintext!;
  const encryptedDataKey = response.CiphertextBlob!;
  
  // Encrypt data with data key
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-gcm', dataKey, iv);
  
  const encrypted = Buffer.concat([
    cipher.update(plaintext),
    cipher.final(),
  ]);
  const authTag = cipher.getAuthTag();
  
  // Combine IV + auth tag + encrypted data
  const encryptedData = Buffer.concat([iv, authTag, encrypted]);
  
  // Clear plaintext key from memory
  dataKey.fill(0);
  
  return {
    encryptedData: encryptedData.toString('base64'),
    encryptedKey: Buffer.from(encryptedDataKey).toString('base64'),
  };
}

async function envelopeDecrypt(
  encryptedData: string,
  encryptedKey: string
): Promise<Buffer> {
  // Decrypt data key with KMS
  const keyResponse = await kms.send(
    new DecryptCommand({
      CiphertextBlob: Buffer.from(encryptedKey, 'base64'),
    })
  );
  
  const dataKey = keyResponse.Plaintext!;
  
  // Parse encrypted data
  const data = Buffer.from(encryptedData, 'base64');
  const iv = data.subarray(0, 16);
  const authTag = data.subarray(16, 32);
  const ciphertext = data.subarray(32);
  
  // Decrypt with data key
  const decipher = crypto.createDecipheriv('aes-256-gcm', dataKey, iv);
  decipher.setAuthTag(authTag);
  
  const decrypted = Buffer.concat([
    decipher.update(ciphertext),
    decipher.final(),
  ]);
  
  // Clear key from memory
  Buffer.from(dataKey).fill(0);
  
  return decrypted;
}
```

---

## Real-World Scenarios

### Scenario 1: Secure API with Multiple Auth Methods

```typescript
// security/api-auth.ts
import { APIGatewayProxyHandler, APIGatewayProxyEvent } from 'aws-lambda';
import { verify, JwtPayload } from 'jsonwebtoken';

interface AuthContext {
  userId: string;
  roles: string[];
  authMethod: 'jwt' | 'api-key' | 'iam';
}

// Custom authorizer for API Gateway
export const authorizer: APIGatewayProxyHandler = async (event) => {
  try {
    const authContext = await authenticate(event);
    
    return {
      statusCode: 200,
      body: JSON.stringify({
        principalId: authContext.userId,
        policyDocument: generatePolicy(authContext, event.methodArn),
        context: authContext,
      }),
    };
  } catch (error) {
    return {
      statusCode: 401,
      body: JSON.stringify({ error: 'Unauthorized' }),
    };
  }
};

async function authenticate(event: APIGatewayProxyEvent): Promise<AuthContext> {
  const authHeader = event.headers['Authorization'] || event.headers['authorization'];
  const apiKey = event.headers['x-api-key'];
  
  // JWT Bearer token
  if (authHeader?.startsWith('Bearer ')) {
    const token = authHeader.slice(7);
    const decoded = verify(token, process.env.JWT_SECRET!) as JwtPayload;
    
    return {
      userId: decoded.sub!,
      roles: decoded.roles || [],
      authMethod: 'jwt',
    };
  }
  
  // API Key
  if (apiKey) {
    const keyData = await validateApiKey(apiKey);
    return {
      userId: keyData.clientId,
      roles: keyData.scopes,
      authMethod: 'api-key',
    };
  }
  
  throw new Error('No valid authentication provided');
}

function generatePolicy(
  context: AuthContext,
  methodArn: string
): object {
  const [, , , region, accountId, apiId, stage] = methodArn.split(/[:/]/);
  
  // Define resource patterns based on roles
  const resources: string[] = [];
  
  if (context.roles.includes('admin')) {
    resources.push(`arn:aws:execute-api:${region}:${accountId}:${apiId}/${stage}/*`);
  } else if (context.roles.includes('user')) {
    resources.push(`arn:aws:execute-api:${region}:${accountId}:${apiId}/${stage}/GET/*`);
    resources.push(`arn:aws:execute-api:${region}:${accountId}:${apiId}/${stage}/*/users/${context.userId}/*`);
  } else {
    resources.push(`arn:aws:execute-api:${region}:${accountId}:${apiId}/${stage}/GET/public/*`);
  }
  
  return {
    Version: '2012-10-17',
    Statement: [
      {
        Effect: 'Allow',
        Action: 'execute-api:Invoke',
        Resource: resources,
      },
    ],
  };
}

async function validateApiKey(apiKey: string): Promise<{ clientId: string; scopes: string[] }> {
  // Validate against stored API keys
  const hashedKey = hashApiKey(apiKey);
  const keyData = await getApiKeyFromDB(hashedKey);
  
  if (!keyData || keyData.expiresAt < Date.now()) {
    throw new Error('Invalid or expired API key');
  }
  
  return keyData;
}

function hashApiKey(key: string): string {
  return require('crypto').createHash('sha256').update(key).digest('hex');
}

async function getApiKeyFromDB(hash: string): Promise<any> {
  return null; // Implement
}
```

### Scenario 2: Security Monitoring and Alerting

```hcl
# security/monitoring.tf

# CloudTrail for API auditing
resource "aws_cloudtrail" "main" {
  name                          = "main-trail"
  s3_bucket_name               = aws_s3_bucket.cloudtrail.id
  include_global_service_events = true
  is_multi_region_trail        = true
  enable_log_file_validation   = true
  
  kms_key_id = aws_kms_key.cloudtrail.arn
  
  event_selector {
    read_write_type           = "All"
    include_management_events = true
    
    data_resource {
      type   = "AWS::S3::Object"
      values = ["arn:aws:s3"]
    }
  }
}

# GuardDuty for threat detection
resource "aws_guardduty_detector" "main" {
  enable                       = true
  finding_publishing_frequency = "FIFTEEN_MINUTES"
  
  datasources {
    s3_logs {
      enable = true
    }
    kubernetes {
      audit_logs {
        enable = true
      }
    }
  }
}

# Security Hub for centralized findings
resource "aws_securityhub_account" "main" {}

# Enable AWS Config for compliance
resource "aws_config_configuration_recorder" "main" {
  name     = "main"
  role_arn = aws_iam_role.config.arn
  
  recording_group {
    all_supported = true
  }
}

# Config rule: S3 buckets must have encryption
resource "aws_config_config_rule" "s3_encryption" {
  name = "s3-bucket-server-side-encryption-enabled"
  
  source {
    owner             = "AWS"
    source_identifier = "S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED"
  }
}

# Config rule: No public S3 buckets
resource "aws_config_config_rule" "s3_public" {
  name = "s3-bucket-public-read-prohibited"
  
  source {
    owner             = "AWS"
    source_identifier = "S3_BUCKET_PUBLIC_READ_PROHIBITED"
  }
}

# CloudWatch alarm for root account usage
resource "aws_cloudwatch_metric_alarm" "root_usage" {
  alarm_name          = "root-account-usage"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "RootAccountUsage"
  namespace           = "CloudTrailMetrics"
  period              = 300
  statistic           = "Sum"
  threshold           = 0
  alarm_description   = "Root account was used"
  alarm_actions       = [aws_sns_topic.alerts.arn]
}

# SNS topic for security alerts
resource "aws_sns_topic" "alerts" {
  name = "security-alerts"
}
```

---

## Common Pitfalls

### 1. Using Root Account

```typescript
// ❌ BAD: Root account for daily operations
// Root has unlimited access, can't be restricted

// ✅ GOOD: Use IAM users/roles with least privilege
// Enable MFA on root, use only for billing/account tasks
```

### 2. Hardcoding Credentials

```typescript
// ❌ BAD: Credentials in code
const aws = new AWS.S3({
  accessKeyId: 'AKIAIOSFODNN7EXAMPLE',
  secretAccessKey: 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY',
});

// ✅ GOOD: Use IAM roles (for AWS resources) or environment variables
const aws = new S3Client({}); // Uses default credential chain

// Credential chain order:
// 1. Environment variables
// 2. Shared credentials file
// 3. IAM role (EC2, Lambda, ECS)

import { S3Client } from '@aws-sdk/client-s3';
```

### 3. Overly Permissive Policies

```json
// ❌ BAD: Admin access for Lambda
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}

// ✅ GOOD: Minimum required permissions
{
  "Effect": "Allow",
  "Action": [
    "dynamodb:GetItem",
    "dynamodb:PutItem"
  ],
  "Resource": "arn:aws:dynamodb:us-east-1:123456789:table/MyTable"
}
```

---

## Interview Questions

### Q1: Explain the difference between IAM users, groups, and roles.

**A:** **Users** are identities for people, have long-term credentials (password, access keys). **Groups** are collections of users for easier permission management. **Roles** are identities assumed temporarily by services or users, have no long-term credentials. Use roles for EC2, Lambda, cross-account access. Use users only when roles aren't possible.

### Q2: What is the principle of least privilege?

**A:** Grant only the minimum permissions needed to perform a task. Start with zero permissions, add only what's required. Use specific resource ARNs instead of wildcards. Add conditions to further restrict (IP, MFA, time). Review and remove unused permissions regularly.

### Q3: How would you securely store and access secrets?

**A:** Use Secrets Manager or Parameter Store, not environment variables for sensitive data. Enable encryption with KMS. Use IAM roles to control access. Enable automatic rotation. Cache secrets with TTL to reduce API calls. Never log secrets. Use separate secrets per environment.

### Q4: What is the shared responsibility model?

**A:** AWS is responsible for security OF the cloud (physical security, hypervisor, managed services). Customer is responsible for security IN the cloud (data, IAM, encryption, network config, OS patches for EC2). For managed services (RDS, Lambda), AWS handles more; for IaaS (EC2), customer handles more.

---

## Quick Reference Checklist

### IAM
- [ ] Enable MFA for all users
- [ ] Use roles for services, not access keys
- [ ] Apply least privilege
- [ ] Review permissions regularly
- [ ] Use conditions in policies

### Encryption
- [ ] Encrypt data at rest (KMS)
- [ ] Encrypt data in transit (TLS)
- [ ] Use envelope encryption for large data
- [ ] Rotate encryption keys

### Secrets
- [ ] Use Secrets Manager/Parameter Store
- [ ] Enable automatic rotation
- [ ] Never hardcode credentials
- [ ] Audit secret access

### Monitoring
- [ ] Enable CloudTrail
- [ ] Enable GuardDuty
- [ ] Set up Config rules
- [ ] Alert on security events

---

*Last updated: February 2026*

