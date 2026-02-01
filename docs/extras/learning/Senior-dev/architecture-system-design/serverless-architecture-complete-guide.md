# ⚡ Serverless Architecture Complete Guide

> A comprehensive guide to serverless computing - Lambda, edge functions, cold starts, patterns, limitations, and when to use (or avoid) serverless.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Serverless is a cloud execution model where the provider manages infrastructure, scales automatically, and you pay only for actual compute time - no idle servers."

### The 5 Core Concepts (Memorize!)
```
1. FUNCTIONS AS A SERVICE (FaaS)  → Code runs in stateless containers triggered by events
2. COLD START                      → Delay when new container spins up (100ms - 10s)
3. EVENT-DRIVEN                    → Functions triggered by HTTP, queues, schedules, etc.
4. PAY-PER-EXECUTION              → Billed by invocations + duration, not uptime
5. AUTOMATIC SCALING              → 0 to 1000s of instances without configuration
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Cold start"** | "We mitigate cold starts with provisioned concurrency for latency-critical endpoints" |
| **"Provisioned concurrency"** | "We keep 10 warm instances to eliminate cold starts for the API" |
| **"Event source mapping"** | "SQS triggers Lambda via event source mapping with batch size of 10" |
| **"Execution context"** | "We reuse database connections by storing them in the execution context" |
| **"Invocation payload"** | "Lambda has 6MB sync invocation payload limit, use S3 for larger data" |
| **"Edge function"** | "Auth runs at the edge with 0ms cold start using Cloudflare Workers" |
| **"Function composition"** | "We use Step Functions for orchestrating multi-step workflows" |

### Key Numbers to Remember
| Metric | AWS Lambda | Cloudflare Workers | Vercel Edge |
|--------|------------|-------------------|-------------|
| **Max execution time** | 15 min | 30 sec (free), 15 min (paid) | 30 sec |
| **Memory** | 128MB - 10GB | 128MB | 128MB |
| **Payload size** | 6MB sync, 256KB async | 100MB | 4MB |
| **Cold start** | 100ms - 10s | ~0ms (V8 isolates) | ~0ms |
| **Free tier** | 1M requests/month | 100K requests/day | 100K/month |

### Cold Start Times (Know These!)
| Runtime | Typical Cold Start |
|---------|-------------------|
| **Node.js** | 100-500ms |
| **Python** | 100-500ms |
| **Go** | 50-200ms |
| **Java** | 3-10 seconds |
| **C#/.NET** | 1-3 seconds |
| **Rust** | 50-200ms |
| **Edge (V8 isolates)** | ~0ms |

### The "Wow" Statement (Memorize This!)
> "Serverless flips the economics of computing. Traditional servers: you pay for 24/7 capacity even at 2% utilization. Serverless: you pay only for actual execution - 1 million requests at 100ms each costs about $0.20. But it's not magic - cold starts can add 500ms latency, 15-minute max execution limits functions, and no persistent connections means you need connection pooling. We use serverless for event-driven workloads and APIs with variable traffic, but keep long-running processes on containers."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                    SERVERLESS ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   EVENT SOURCES                     FUNCTIONS                   │
│   ─────────────                     ─────────                   │
│   ┌─────────────┐                  ┌─────────────────┐          │
│   │ API Gateway │─────────────────►│   Lambda A      │          │
│   │   (HTTP)    │                  │   (API Handler) │          │
│   └─────────────┘                  └────────┬────────┘          │
│                                             │                    │
│   ┌─────────────┐                  ┌────────▼────────┐          │
│   │     SQS     │─────────────────►│   Lambda B      │          │
│   │   (Queue)   │                  │  (Background)   │          │
│   └─────────────┘                  └────────┬────────┘          │
│                                             │                    │
│   ┌─────────────┐                  ┌────────▼────────┐          │
│   │     S3      │─────────────────►│   Lambda C      │          │
│   │  (Upload)   │                  │ (Image Process) │          │
│   └─────────────┘                  └────────┬────────┘          │
│                                             │                    │
│   ┌─────────────┐                  ┌────────▼────────┐          │
│   │ CloudWatch  │─────────────────►│   Lambda D      │          │
│   │   (Cron)    │                  │   (Scheduled)   │          │
│   └─────────────┘                  └─────────────────┘          │
│                                                                  │
│   ┌─────────────┐                  ┌─────────────────┐          │
│   │    Edge     │                  │  Cloudflare     │          │
│   │  (Request)  │─────────────────►│    Workers      │          │
│   └─────────────┘                  │  (Auth, A/B)    │          │
│                                    └─────────────────┘          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is serverless?"**
> "Execution model where cloud provider manages infrastructure. Code runs in stateless containers, scales automatically, pay only for compute time used."

**Q: "What is a cold start?"**
> "Delay when a new container initializes - downloading code, starting runtime, running init code. Can be 100ms to 10s depending on runtime and dependencies."

**Q: "How do you mitigate cold starts?"**
> "Provisioned concurrency (keep warm instances), smaller packages, faster runtimes (Node/Go over Java), lazy loading, or edge functions with V8 isolates."

**Q: "What are the limitations of Lambda?"**
> "15 min max execution, 6MB payload, no persistent connections, cold starts, 10GB max memory, no GPU, vendor lock-in."

**Q: "When NOT to use serverless?"**
> "Long-running processes, WebSocket connections, high-performance computing, predictable high traffic (containers cheaper), GPU workloads."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "What is Serverless?"

**Junior Answer:**
> "It's where you don't manage servers. You just write functions."

**Senior Answer:**
> "Serverless is an **execution model** where the cloud provider dynamically manages infrastructure. There are two main types:

**1. Functions as a Service (FaaS)**
- Code runs in stateless, ephemeral containers
- Triggered by events (HTTP, queues, schedules, file uploads)
- Scales from 0 to thousands automatically
- Pay per invocation + duration (not uptime)

**2. Backend as a Service (BaaS)**
- Managed services for auth, database, storage
- Firebase, Supabase, Auth0

**Key tradeoffs:**
- ✅ No server management, automatic scaling, pay-per-use
- ❌ Cold starts (latency), execution limits (15 min), no persistent connections, vendor lock-in

I use serverless for APIs with variable traffic, event processing, and scheduled jobs. For long-running processes or consistent high traffic, containers are often better economically."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "How does scaling work?" | "Provider spins up new container instances automatically based on demand. Lambda can go from 0 to 1000+ concurrent executions. Each request gets its own container (or reuses warm one)." |
| "What causes cold starts?" | "Container provisioning, downloading code, starting runtime, running initialization code. Factors: package size, runtime (Java slow, Go fast), VPC (adds 1s), dependencies." |
| "How do you handle state?" | "Functions are stateless. Use external stores: DynamoDB, Redis, S3. Connection reuse via execution context. For workflows, use Step Functions." |
| "Serverless vs containers?" | "Serverless: variable traffic, event-driven, simple functions. Containers: steady traffic (cheaper), long-running, complex apps, predictable costs." |

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Providers & Platforms](#2-providers--platforms)
3. [Cold Starts](#3-cold-starts)
4. [Event Sources & Triggers](#4-event-sources--triggers)
5. [Patterns & Best Practices](#5-patterns--best-practices)
6. [Limitations & Gotchas](#6-limitations--gotchas)
7. [Cost Analysis](#7-cost-analysis)
8. [When to Use / Not Use](#8-when-to-use--not-use)
9. [Interview Questions](#9-interview-questions)

---

## 1. Core Concepts

### How Serverless Works

```
REQUEST LIFECYCLE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. EVENT ARRIVES (HTTP request, SQS message, S3 upload)        │
│          │                                                       │
│          ▼                                                       │
│  2. PROVIDER CHECKS: Is there a warm container?                 │
│          │                                                       │
│          ├── YES ──► Use existing container (WARM START ~1ms)   │
│          │                                                       │
│          └── NO ───► Provision new container (COLD START)       │
│                            │                                     │
│                            ▼                                     │
│                      ┌──────────────┐                           │
│                      │ Download code │ ─────► ~50-200ms         │
│                      │ Start runtime │ ─────► ~100-500ms        │
│                      │ Run init code │ ─────► Variable          │
│                      └──────────────┘                           │
│                            │                                     │
│                            ▼                                     │
│  3. HANDLER EXECUTES                                            │
│          │                                                       │
│          ▼                                                       │
│  4. RESPONSE RETURNED                                           │
│          │                                                       │
│          ▼                                                       │
│  5. CONTAINER STAYS WARM (for ~5-15 minutes)                   │
│          │                                                       │
│          └── Next request reuses this container                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Execution Context & Container Reuse

```typescript
// ════════════════════════════════════════════════════════════════
// EXECUTION CONTEXT: Code outside handler runs ONCE per container
// ════════════════════════════════════════════════════════════════

// OUTSIDE HANDLER - Runs once when container starts
// Reused across invocations in same container
import { DynamoDB } from '@aws-sdk/client-dynamodb';

// This connection is REUSED across warm invocations
const dynamodb = new DynamoDB({});

// This variable persists between invocations
let requestCount = 0;

// INSIDE HANDLER - Runs on EVERY invocation
export const handler = async (event: APIGatewayEvent) => {
  requestCount++;  // Increments across warm invocations
  console.log(`Request #${requestCount} in this container`);

  // Reuses the connection established above
  await dynamodb.getItem({ /* ... */ });

  return { statusCode: 200, body: 'OK' };
};

// ════════════════════════════════════════════════════════════════
// BEST PRACTICE: Initialize expensive resources outside handler
// ════════════════════════════════════════════════════════════════

// ❌ BAD: Creates new connection on EVERY invocation
export const badHandler = async (event) => {
  const db = new DynamoDB({});  // New connection every time!
  await db.getItem({ /* ... */ });
};

// ✅ GOOD: Reuses connection across warm invocations
const db = new DynamoDB({});  // Created once per container

export const goodHandler = async (event) => {
  await db.getItem({ /* ... */ });  // Reuses connection
};
```

### Serverless vs Traditional vs Containers

```
┌─────────────────────────────────────────────────────────────────┐
│              COMPARISON: Serverless vs Containers vs VMs        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ASPECT           │ SERVERLESS   │ CONTAINERS   │ VMs          │
│  ─────────────────────────────────────────────────────────────  │
│  Unit of deploy   │ Function     │ Image        │ Machine      │
│  Scaling          │ Automatic    │ Orchestrated │ Manual/Auto  │
│  Scale to zero    │ Yes          │ Possible     │ No           │
│  Cold start       │ 100ms-10s    │ Seconds      │ Minutes      │
│  Max runtime      │ 15 min       │ Unlimited    │ Unlimited    │
│  Pricing model    │ Per execution│ Per uptime   │ Per uptime   │
│  State            │ Stateless    │ Can be any   │ Stateful     │
│  Ops overhead     │ None         │ Medium       │ High         │
│  Vendor lock-in   │ High         │ Low          │ Low          │
│  Use case         │ Events, APIs │ Services     │ Legacy       │
│                                                                  │
│  WHEN TO USE:                                                   │
│  ─────────────────────────────────────────────────────────────  │
│  Serverless: Variable traffic, event-driven, simple functions  │
│  Containers: Consistent traffic, microservices, complex apps   │
│  VMs: Legacy apps, specific OS needs, lift-and-shift          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Providers & Platforms

### AWS Lambda

The original and most feature-rich serverless platform.

```typescript
// ════════════════════════════════════════════════════════════════
// AWS LAMBDA: Basic Handler
// ════════════════════════════════════════════════════════════════

import { APIGatewayProxyEvent, APIGatewayProxyResult, Context } from 'aws-lambda';

export const handler = async (
  event: APIGatewayProxyEvent,
  context: Context
): Promise<APIGatewayProxyResult> => {
  // event: Contains request data (path, method, body, headers)
  // context: Contains runtime info (requestId, remainingTime, etc.)

  console.log('Request ID:', context.awsRequestId);
  console.log('Remaining time:', context.getRemainingTimeInMillis(), 'ms');

  const body = JSON.parse(event.body || '{}');

  return {
    statusCode: 200,
    headers: {
      'Content-Type': 'application/json',
      'Access-Control-Allow-Origin': '*'
    },
    body: JSON.stringify({
      message: 'Hello from Lambda!',
      input: body
    })
  };
};

// ════════════════════════════════════════════════════════════════
// AWS LAMBDA: Different Event Sources
// ════════════════════════════════════════════════════════════════

// SQS Trigger
import { SQSEvent, SQSRecord } from 'aws-lambda';

export const sqsHandler = async (event: SQSEvent): Promise<void> => {
  for (const record of event.Records) {
    const message = JSON.parse(record.body);
    console.log('Processing message:', message);
    // Process each message
  }
  // If no error thrown, messages are deleted from queue
};

// S3 Trigger
import { S3Event } from 'aws-lambda';

export const s3Handler = async (event: S3Event): Promise<void> => {
  for (const record of event.Records) {
    const bucket = record.s3.bucket.name;
    const key = record.s3.object.key;
    console.log(`File uploaded: s3://${bucket}/${key}`);
    // Process the uploaded file
  }
};

// Scheduled (CloudWatch Events)
import { ScheduledEvent } from 'aws-lambda';

export const cronHandler = async (event: ScheduledEvent): Promise<void> => {
  console.log('Scheduled execution at:', event.time);
  // Run scheduled task (cleanup, reports, etc.)
};

// DynamoDB Streams
import { DynamoDBStreamEvent } from 'aws-lambda';

export const dynamoHandler = async (event: DynamoDBStreamEvent): Promise<void> => {
  for (const record of event.Records) {
    if (record.eventName === 'INSERT') {
      const newItem = record.dynamodb?.NewImage;
      console.log('New item:', newItem);
    }
  }
};
```

### Lambda Configuration

```yaml
# serverless.yml (Serverless Framework)
service: my-api

provider:
  name: aws
  runtime: nodejs18.x
  region: us-east-1
  memorySize: 256        # MB (128-10240)
  timeout: 30            # seconds (max 900)
  
  # Environment variables
  environment:
    DATABASE_URL: ${env:DATABASE_URL}
    NODE_ENV: production

functions:
  api:
    handler: src/handlers/api.handler
    events:
      - http:
          path: /users
          method: get
      - http:
          path: /users
          method: post
    
    # Provisioned concurrency (eliminates cold starts)
    provisionedConcurrency: 5
    
    # Reserved concurrency (limit max instances)
    reservedConcurrency: 100
    
    # VPC configuration (adds ~1s cold start)
    vpc:
      securityGroupIds:
        - sg-xxxx
      subnetIds:
        - subnet-xxxx

  processQueue:
    handler: src/handlers/queue.handler
    events:
      - sqs:
          arn: !GetAtt MyQueue.Arn
          batchSize: 10
          maximumBatchingWindow: 5
    
  scheduledTask:
    handler: src/handlers/cron.handler
    events:
      - schedule: rate(1 hour)    # or cron(0 * * * ? *)
```

### Cloudflare Workers (Edge Functions)

Near-zero cold starts using V8 isolates instead of containers.

```typescript
// ════════════════════════════════════════════════════════════════
// CLOUDFLARE WORKERS: Edge computing with ~0ms cold start
// ════════════════════════════════════════════════════════════════

// wrangler.toml
// name = "my-worker"
// main = "src/index.ts"
// compatibility_date = "2024-01-01"

export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
    const url = new URL(request.url);

    // Simple routing
    if (url.pathname === '/api/hello') {
      return new Response(JSON.stringify({ message: 'Hello from the edge!' }), {
        headers: { 'Content-Type': 'application/json' }
      });
    }

    if (url.pathname === '/api/user') {
      // Access KV storage (Cloudflare's key-value store)
      const user = await env.USERS_KV.get('user:123', 'json');
      return Response.json(user);
    }

    // Use Durable Objects for stateful edge computing
    if (url.pathname.startsWith('/api/counter')) {
      const id = env.COUNTER.idFromName('global');
      const counter = env.COUNTER.get(id);
      return counter.fetch(request);
    }

    return new Response('Not Found', { status: 404 });
  }
};

// Environment bindings
interface Env {
  USERS_KV: KVNamespace;
  COUNTER: DurableObjectNamespace;
  DATABASE: D1Database;  // Cloudflare's SQL database
}

// ════════════════════════════════════════════════════════════════
// DURABLE OBJECTS: Stateful edge computing
// ════════════════════════════════════════════════════════════════

export class Counter {
  private state: DurableObjectState;
  private count: number = 0;

  constructor(state: DurableObjectState) {
    this.state = state;
    // Load persisted state
    this.state.blockConcurrencyWhile(async () => {
      this.count = await this.state.storage.get('count') || 0;
    });
  }

  async fetch(request: Request): Promise<Response> {
    const url = new URL(request.url);

    if (url.pathname.endsWith('/increment')) {
      this.count++;
      await this.state.storage.put('count', this.count);
    }

    return Response.json({ count: this.count });
  }
}
```

### Vercel Functions / Edge Functions

```typescript
// ════════════════════════════════════════════════════════════════
// VERCEL SERVERLESS FUNCTION (Node.js runtime)
// ════════════════════════════════════════════════════════════════

// api/users.ts
import type { VercelRequest, VercelResponse } from '@vercel/node';

export default async function handler(
  req: VercelRequest,
  res: VercelResponse
) {
  if (req.method === 'GET') {
    const users = await fetchUsers();
    return res.status(200).json(users);
  }

  if (req.method === 'POST') {
    const user = await createUser(req.body);
    return res.status(201).json(user);
  }

  return res.status(405).json({ error: 'Method not allowed' });
}

// ════════════════════════════════════════════════════════════════
// VERCEL EDGE FUNCTION (~0ms cold start)
// ════════════════════════════════════════════════════════════════

// api/edge-hello.ts
export const config = {
  runtime: 'edge',  // Use edge runtime
};

export default async function handler(request: Request) {
  const { searchParams } = new URL(request.url);
  const name = searchParams.get('name') || 'World';

  return new Response(
    JSON.stringify({ message: `Hello, ${name}!` }),
    {
      status: 200,
      headers: { 'Content-Type': 'application/json' }
    }
  );
}

// ════════════════════════════════════════════════════════════════
// NEXT.JS MIDDLEWARE (Edge Function)
// ════════════════════════════════════════════════════════════════

// middleware.ts (runs on every request at the edge)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Authentication check
  const token = request.cookies.get('auth-token');
  
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  // A/B testing
  const bucket = Math.random() < 0.5 ? 'a' : 'b';
  const response = NextResponse.next();
  response.cookies.set('ab-bucket', bucket);

  // Geolocation-based routing
  const country = request.geo?.country || 'US';
  if (country === 'DE') {
    return NextResponse.rewrite(new URL('/de' + request.nextUrl.pathname, request.url));
  }

  return response;
}

export const config = {
  matcher: ['/((?!api|_next/static|favicon.ico).*)'],
};
```

### Provider Comparison

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    SERVERLESS PROVIDER COMPARISON                         │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  FEATURE          │ AWS Lambda  │ Cloudflare  │ Vercel      │ GCP       │
│  ─────────────────────────────────────────────────────────────────────── │
│  Cold start       │ 100ms-10s   │ ~0ms        │ ~0ms (edge) │ 100ms-10s │
│  Max duration     │ 15 min      │ 30s/15min   │ 30s edge    │ 60 min    │
│  Max memory       │ 10 GB       │ 128 MB      │ 1 GB        │ 32 GB     │
│  Languages        │ Many        │ JS/TS/Wasm  │ JS/TS       │ Many      │
│  Edge locations   │ Limited     │ 300+        │ Many        │ Limited   │
│  Database         │ DynamoDB    │ D1, KV, DO  │ Edge Config │ Firestore │
│  Pricing          │ Per ms      │ Per request │ Per request │ Per ms    │
│                                                                           │
│  BEST FOR:                                                               │
│  AWS Lambda    → Complex backends, AWS ecosystem, long-running tasks    │
│  Cloudflare    → Edge computing, global low latency, simple APIs        │
│  Vercel        → Next.js apps, frontend teams, simple APIs             │
│  GCP Functions → GCP ecosystem, Firebase integration                    │
│                                                                           │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Cold Starts

### What Causes Cold Starts?

```
COLD START BREAKDOWN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  PHASE                    │ TIME IMPACT    │ YOU CONTROL?       │
│  ─────────────────────────────────────────────────────────────  │
│  1. Container provision   │ ~50-100ms      │ No                │
│  2. Download code         │ ~50-200ms      │ Yes (package size)│
│  3. Start runtime         │ ~100ms-3s      │ Yes (runtime)     │
│  4. VPC ENI attach        │ ~1-2s          │ Yes (avoid VPC)   │
│  5. Init code execution   │ Variable       │ Yes (lazy load)   │
│  ─────────────────────────────────────────────────────────────  │
│  TOTAL                    │ 100ms - 10s+   │                   │
│                                                                  │
│  RUNTIME IMPACT:                                                │
│  ─────────────────────────────────────────────────────────────  │
│  Node.js / Python    │ ~200-500ms  │ Lightweight runtime       │
│  Go / Rust           │ ~50-200ms   │ Compiled, fast startup    │
│  Java                │ ~3-10s      │ JVM startup overhead      │
│  .NET                │ ~1-3s       │ CLR startup overhead      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Cold Start Mitigation Strategies

```typescript
// ════════════════════════════════════════════════════════════════
// STRATEGY 1: Provisioned Concurrency (AWS Lambda)
// Keep warm instances always ready
// ════════════════════════════════════════════════════════════════

// serverless.yml
// functions:
//   api:
//     handler: src/api.handler
//     provisionedConcurrency: 5  # Always 5 warm instances

// Cost: ~$15/month per provisioned instance (varies by memory)
// Best for: User-facing APIs where latency matters

// ════════════════════════════════════════════════════════════════
// STRATEGY 2: Keep Functions Warm (Ping)
// Scheduled invocation to prevent container shutdown
// ════════════════════════════════════════════════════════════════

// warmer.ts
export const handler = async (event: any) => {
  // Check if this is a warming request
  if (event.source === 'serverless-plugin-warmup') {
    console.log('Warming function...');
    return { statusCode: 200, body: 'Warmed!' };
  }

  // Normal request handling
  return handleRequest(event);
};

// serverless.yml with warmup plugin
// plugins:
//   - serverless-plugin-warmup
// custom:
//   warmup:
//     default:
//       enabled: true
//       events:
//         - schedule: rate(5 minutes)
//       concurrency: 3

// ════════════════════════════════════════════════════════════════
// STRATEGY 3: Smaller Packages
// Less code to download = faster cold starts
// ════════════════════════════════════════════════════════════════

// ❌ BAD: Huge package with everything
// package size: 50MB
import AWS from 'aws-sdk';  // Imports entire SDK

// ✅ GOOD: Import only what you need
// package size: 5MB
import { DynamoDB } from '@aws-sdk/client-dynamodb';

// ✅ BETTER: Use esbuild bundling with tree-shaking
// serverless.yml:
// package:
//   individually: true
// plugins:
//   - serverless-esbuild
// custom:
//   esbuild:
//     bundle: true
//     minify: true
//     exclude:
//       - '@aws-sdk/*'  # Exclude SDK (available in Lambda runtime)

// ════════════════════════════════════════════════════════════════
// STRATEGY 4: Lazy Loading
// Defer expensive initialization until needed
// ════════════════════════════════════════════════════════════════

// ❌ BAD: Load everything at startup
import { S3Client } from '@aws-sdk/client-s3';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { SESClient } from '@aws-sdk/client-ses';
import Stripe from 'stripe';
import { PrismaClient } from '@prisma/client';

// All initialized at cold start!
const s3 = new S3Client({});
const dynamodb = new DynamoDBClient({});
const ses = new SESClient({});
const stripe = new Stripe(process.env.STRIPE_KEY!);
const prisma = new PrismaClient();

// ✅ GOOD: Lazy load only when needed
let s3Client: S3Client | null = null;
let stripeClient: Stripe | null = null;

function getS3(): S3Client {
  if (!s3Client) {
    s3Client = new S3Client({});
  }
  return s3Client;
}

function getStripe(): Stripe {
  if (!stripeClient) {
    stripeClient = new Stripe(process.env.STRIPE_KEY!);
  }
  return stripeClient;
}

export const handler = async (event: any) => {
  // Only loads S3 client if this code path is taken
  if (event.path === '/upload') {
    const s3 = getS3();
    // ...
  }
  
  // Stripe only loaded for payment routes
  if (event.path === '/checkout') {
    const stripe = getStripe();
    // ...
  }
};

// ════════════════════════════════════════════════════════════════
// STRATEGY 5: Use Faster Runtimes
// ════════════════════════════════════════════════════════════════

// Cold start comparison:
// Java:    3-10 seconds
// .NET:    1-3 seconds
// Node.js: 200-500ms
// Python:  200-500ms
// Go:      50-200ms
// Rust:    50-150ms

// For latency-critical functions, consider Go or Rust
// For most web APIs, Node.js or Python are good tradeoffs

// ════════════════════════════════════════════════════════════════
// STRATEGY 6: Avoid VPC (or Use VPC with Hyperplane)
// ════════════════════════════════════════════════════════════════

// VPC adds ~1-2s to cold start for ENI (Elastic Network Interface) creation
// AWS now uses Hyperplane for VPC, reducing this significantly

// If you MUST use VPC:
// - Use VPC endpoints for AWS services (avoid NAT Gateway latency)
// - Enable Hyperplane (automatic in newer accounts)
// - Use provisioned concurrency

// If you DON'T need VPC:
// - Access AWS services directly (they have public endpoints)
// - Use IAM roles instead of VPC for security

// ════════════════════════════════════════════════════════════════
// STRATEGY 7: Use Edge Functions (V8 Isolates)
// ════════════════════════════════════════════════════════════════

// Edge functions use V8 isolates, not containers
// Result: Near-zero cold starts (~0ms)

// Cloudflare Workers, Vercel Edge, Deno Deploy
// Trade-off: More limited runtime (no Node.js APIs, limited libraries)

// Good for: Auth, redirects, A/B testing, geo-routing
// Not good for: Database queries, complex processing
```

### Cold Start Measurement

```typescript
// ════════════════════════════════════════════════════════════════
// MEASURE COLD STARTS: Track init time vs execution time
// ════════════════════════════════════════════════════════════════

const initStart = Date.now();

// Expensive initialization
import { DynamoDB } from '@aws-sdk/client-dynamodb';
const dynamodb = new DynamoDB({});

const initDuration = Date.now() - initStart;
console.log(`INIT_DURATION: ${initDuration}ms`);

let isWarm = false;

export const handler = async (event: any, context: any) => {
  const handlerStart = Date.now();
  
  // Log whether this is a cold or warm start
  console.log(`COLD_START: ${!isWarm}`);
  isWarm = true;

  // Your logic here
  await dynamodb.getItem({ /* ... */ });

  const handlerDuration = Date.now() - handlerStart;
  console.log(`HANDLER_DURATION: ${handlerDuration}ms`);

  // Total time for cold start = INIT_DURATION + HANDLER_DURATION
  // Warm start = only HANDLER_DURATION
  
  return { statusCode: 200, body: 'OK' };
};

// CloudWatch Insights query to analyze cold starts:
// fields @timestamp, @message
// | filter @message like /COLD_START: true/
// | stats count() as coldStarts by bin(1h)
```

### Cold Start vs Warm Start Visualization

```
TIMELINE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  COLD START (First request to new container):                  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │ Container │ Download │ Runtime │  Init  │    Handler    │  │
│  │  Provision│  Code    │  Start  │  Code  │   Execution   │  │
│  │   ~100ms  │  ~100ms  │ ~200ms  │ ~200ms │    ~50ms      │  │
│  ├──────────────────────────────────────────────────────────┤  │
│  │                    TOTAL: ~650ms                         │  │
│                                                                  │
│  WARM START (Subsequent request to same container):            │
│  ├────────────────────────┤                                    │
│  │    Handler Execution   │                                    │
│  │         ~50ms          │                                    │
│  ├────────────────────────┤                                    │
│  │     TOTAL: ~50ms       │                                    │
│                                                                  │
│  PROVISIONED CONCURRENCY (Pre-warmed):                         │
│  ├────────────────────────┤                                    │
│  │    Handler Execution   │  (Container already warm)         │
│  │         ~50ms          │                                    │
│  ├────────────────────────┤                                    │
│  │     TOTAL: ~50ms       │  (No cold start ever)             │
│                                                                  │
│  EDGE FUNCTION (V8 Isolate):                                   │
│  ├────────────────────────┤                                    │
│  │       Execution        │  (Isolate spins up in ~5ms)       │
│  │         ~55ms          │                                    │
│  ├────────────────────────┤                                    │
│  │     TOTAL: ~55ms       │  (Near-zero cold start)           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Event Sources & Triggers

### Common Event Sources

```
┌─────────────────────────────────────────────────────────────────┐
│                    LAMBDA EVENT SOURCES                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SYNCHRONOUS (Request-Response):                                │
│  ├── API Gateway         → REST/HTTP APIs                      │
│  ├── Application LB      → Load-balanced web apps              │
│  ├── Lambda Function URL → Direct HTTP invocation              │
│  └── SDK Invoke          → From other services/functions       │
│                                                                  │
│  ASYNCHRONOUS (Fire-and-Forget):                               │
│  ├── S3                  → File uploads/deletions              │
│  ├── SNS                 → Pub/sub notifications               │
│  ├── EventBridge         → Event bus patterns                  │
│  ├── IoT                 → Device events                       │
│  └── SDK InvokeAsync     → Background processing               │
│                                                                  │
│  POLLING (Lambda pulls from source):                           │
│  ├── SQS                 → Queue processing                    │
│  ├── Kinesis             → Stream processing                   │
│  ├── DynamoDB Streams    → Change data capture                 │
│  └── Kafka (MSK)         → Kafka consumers                     │
│                                                                  │
│  SCHEDULED:                                                     │
│  └── CloudWatch Events   → Cron jobs, rate-based              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Event Source Patterns

```typescript
// ════════════════════════════════════════════════════════════════
// PATTERN 1: API Gateway + Lambda (REST API)
// ════════════════════════════════════════════════════════════════

// GET /users/{id}
export const getUser = async (event: APIGatewayProxyEvent) => {
  const userId = event.pathParameters?.id;
  const user = await userService.getById(userId);
  
  if (!user) {
    return { statusCode: 404, body: JSON.stringify({ error: 'Not found' }) };
  }
  
  return { statusCode: 200, body: JSON.stringify(user) };
};

// POST /users
export const createUser = async (event: APIGatewayProxyEvent) => {
  const body = JSON.parse(event.body || '{}');
  
  // Validate input
  if (!body.email) {
    return { statusCode: 400, body: JSON.stringify({ error: 'Email required' }) };
  }
  
  const user = await userService.create(body);
  return { statusCode: 201, body: JSON.stringify(user) };
};

// ════════════════════════════════════════════════════════════════
// PATTERN 2: SQS + Lambda (Queue Processing with Backpressure)
// ════════════════════════════════════════════════════════════════

import { SQSEvent, SQSBatchResponse, SQSBatchItemFailure } from 'aws-lambda';

export const processQueue = async (event: SQSEvent): Promise<SQSBatchResponse> => {
  const batchItemFailures: SQSBatchItemFailure[] = [];

  // Process each message, track failures individually
  for (const record of event.Records) {
    try {
      const message = JSON.parse(record.body);
      await processMessage(message);
    } catch (error) {
      console.error(`Failed to process message ${record.messageId}:`, error);
      // Report this specific message as failed
      batchItemFailures.push({ itemIdentifier: record.messageId });
    }
  }

  // Return partial batch response
  // Only failed messages will return to queue
  return { batchItemFailures };
};

// serverless.yml configuration for backpressure:
// functions:
//   processQueue:
//     handler: src/queue.handler
//     reservedConcurrency: 10   # Limit concurrent executions (BACKPRESSURE!)
//     events:
//       - sqs:
//           arn: !GetAtt OrderQueue.Arn
//           batchSize: 10
//           maximumBatchingWindow: 5  # Wait up to 5s to batch
//           functionResponseType: ReportBatchItemFailures

// ════════════════════════════════════════════════════════════════
// PATTERN 3: S3 + Lambda (File Processing)
// ════════════════════════════════════════════════════════════════

import { S3Event } from 'aws-lambda';
import { S3Client, GetObjectCommand, PutObjectCommand } from '@aws-sdk/client-s3';

const s3 = new S3Client({});

export const processImage = async (event: S3Event): Promise<void> => {
  for (const record of event.Records) {
    const bucket = record.s3.bucket.name;
    const key = decodeURIComponent(record.s3.object.key);

    // Don't process our own output (prevent infinite loop!)
    if (key.startsWith('thumbnails/')) {
      return;
    }

    // Get the uploaded image
    const getCommand = new GetObjectCommand({ Bucket: bucket, Key: key });
    const response = await s3.send(getCommand);
    const imageBuffer = await response.Body?.transformToByteArray();

    // Process image (resize, compress, etc.)
    const thumbnail = await createThumbnail(imageBuffer);

    // Save thumbnail
    const putCommand = new PutObjectCommand({
      Bucket: bucket,
      Key: `thumbnails/${key}`,
      Body: thumbnail,
      ContentType: 'image/jpeg'
    });
    await s3.send(putCommand);
  }
};

// ════════════════════════════════════════════════════════════════
// PATTERN 4: EventBridge + Lambda (Event-Driven)
// ════════════════════════════════════════════════════════════════

interface OrderEvent {
  'detail-type': 'OrderPlaced' | 'OrderShipped' | 'OrderCancelled';
  source: 'orders-service';
  detail: {
    orderId: string;
    customerId: string;
    total: number;
  };
}

// Publish event
import { EventBridgeClient, PutEventsCommand } from '@aws-sdk/client-eventbridge';

const eventBridge = new EventBridgeClient({});

export const publishOrderPlaced = async (order: Order) => {
  await eventBridge.send(new PutEventsCommand({
    Entries: [{
      EventBusName: 'orders',
      Source: 'orders-service',
      DetailType: 'OrderPlaced',
      Detail: JSON.stringify({
        orderId: order.id,
        customerId: order.customerId,
        total: order.total
      })
    }]
  }));
};

// Subscribe to event
export const handleOrderPlaced = async (event: OrderEvent) => {
  const { orderId, customerId } = event.detail;
  
  // Send confirmation email
  await emailService.sendOrderConfirmation(customerId, orderId);
  
  // Start inventory reservation
  await inventoryService.reserve(orderId);
};

// ════════════════════════════════════════════════════════════════
// PATTERN 5: Step Functions (Workflow Orchestration)
// ════════════════════════════════════════════════════════════════

// State machine definition (serverless.yml)
// stepFunctions:
//   stateMachines:
//     orderWorkflow:
//       definition:
//         StartAt: ValidateOrder
//         States:
//           ValidateOrder:
//             Type: Task
//             Resource: !GetAtt ValidateOrderFunction.Arn
//             Next: ProcessPayment
//             Catch:
//               - ErrorEquals: [ValidationError]
//                 Next: OrderFailed
//           ProcessPayment:
//             Type: Task
//             Resource: !GetAtt ProcessPaymentFunction.Arn
//             Next: ReserveInventory
//             Catch:
//               - ErrorEquals: [PaymentError]
//                 Next: OrderFailed
//           ReserveInventory:
//             Type: Task
//             Resource: !GetAtt ReserveInventoryFunction.Arn
//             Next: OrderComplete
//           OrderComplete:
//             Type: Succeed
//           OrderFailed:
//             Type: Fail

// Individual step functions
export const validateOrder = async (event: { orderId: string }) => {
  const order = await orderRepo.findById(event.orderId);
  
  if (!order || order.items.length === 0) {
    throw new ValidationError('Invalid order');
  }
  
  return { orderId: event.orderId, validated: true };
};

export const processPayment = async (event: { orderId: string }) => {
  const order = await orderRepo.findById(event.orderId);
  const result = await paymentService.charge(order.customerId, order.total);
  
  if (!result.success) {
    throw new PaymentError('Payment failed');
  }
  
  return { orderId: event.orderId, paymentId: result.paymentId };
};
```

---

## 5. Patterns & Best Practices

### Database Connections

```typescript
// ════════════════════════════════════════════════════════════════
// PROBLEM: Too many database connections
// Each Lambda instance creates new connection
// ════════════════════════════════════════════════════════════════

// ❌ BAD: New connection per invocation
export const handler = async () => {
  const pool = new Pool({ connectionString: process.env.DATABASE_URL });
  const result = await pool.query('SELECT * FROM users');
  await pool.end();  // Connection closed, wasted!
  return result.rows;
};

// ════════════════════════════════════════════════════════════════
// SOLUTION 1: Reuse connections in execution context
// ════════════════════════════════════════════════════════════════

import { Pool } from 'pg';

// Connection created once per container
let pool: Pool | null = null;

function getPool(): Pool {
  if (!pool) {
    pool = new Pool({
      connectionString: process.env.DATABASE_URL,
      max: 1,  // Single connection per Lambda instance
      idleTimeoutMillis: 120000,  // Keep alive during warm period
    });
  }
  return pool;
}

export const handler = async () => {
  const db = getPool();
  const result = await db.query('SELECT * FROM users');
  return result.rows;
};

// ════════════════════════════════════════════════════════════════
// SOLUTION 2: Use RDS Proxy (Connection pooling)
// ════════════════════════════════════════════════════════════════

// RDS Proxy sits between Lambda and database
// Manages connection pool, handles surges
// Connect to proxy endpoint instead of database directly

const pool = new Pool({
  host: 'my-proxy.proxy-xxxxx.us-east-1.rds.amazonaws.com',
  // Lambda connects to proxy, proxy manages DB connections
});

// ════════════════════════════════════════════════════════════════
// SOLUTION 3: Use HTTP-based databases
// ════════════════════════════════════════════════════════════════

// PlanetScale, Neon, Turso - HTTP/WebSocket protocols
// No persistent connections needed, better for serverless

import { neon } from '@neondatabase/serverless';

const sql = neon(process.env.DATABASE_URL!);

export const handler = async () => {
  // HTTP-based query, no connection pool needed
  const users = await sql`SELECT * FROM users`;
  return users;
};
```

### Error Handling & Retries

```typescript
// ════════════════════════════════════════════════════════════════
// IDEMPOTENCY: Same request produces same result
// ════════════════════════════════════════════════════════════════

import { DynamoDB } from '@aws-sdk/client-dynamodb';

const dynamodb = new DynamoDB({});

export const processPayment = async (event: any) => {
  const { orderId, amount, idempotencyKey } = event;

  // Check if already processed
  const existing = await dynamodb.getItem({
    TableName: 'IdempotencyTable',
    Key: { pk: { S: idempotencyKey } }
  });

  if (existing.Item) {
    console.log('Already processed, returning cached result');
    return JSON.parse(existing.Item.result.S!);
  }

  // Process payment
  const result = await paymentService.charge(orderId, amount);

  // Store result for idempotency
  await dynamodb.putItem({
    TableName: 'IdempotencyTable',
    Key: { pk: { S: idempotencyKey } },
    Item: {
      pk: { S: idempotencyKey },
      result: { S: JSON.stringify(result) },
      ttl: { N: String(Math.floor(Date.now() / 1000) + 86400) }  // 24h TTL
    }
  });

  return result;
};

// ════════════════════════════════════════════════════════════════
// DEAD LETTER QUEUE: Handle failed messages
// ════════════════════════════════════════════════════════════════

// serverless.yml
// resources:
//   Resources:
//     MyQueue:
//       Type: AWS::SQS::Queue
//       Properties:
//         QueueName: my-queue
//         RedrivePolicy:
//           deadLetterTargetArn: !GetAtt DeadLetterQueue.Arn
//           maxReceiveCount: 3  # Move to DLQ after 3 failures
//     DeadLetterQueue:
//       Type: AWS::SQS::Queue
//       Properties:
//         QueueName: my-queue-dlq

// Handle DLQ separately
export const processDLQ = async (event: SQSEvent) => {
  for (const record of event.Records) {
    // Log failed message for investigation
    console.error('Failed message:', record.body);
    
    // Alert operations team
    await alertService.notify({
      type: 'DLQ_MESSAGE',
      queue: 'my-queue',
      message: record.body
    });
  }
};

// ════════════════════════════════════════════════════════════════
// CIRCUIT BREAKER: Prevent cascade failures
// ════════════════════════════════════════════════════════════════

class CircuitBreaker {
  private failures = 0;
  private lastFailure: number = 0;
  private readonly threshold = 5;
  private readonly resetTimeout = 30000;  // 30 seconds

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    // Check if circuit is open
    if (this.isOpen()) {
      throw new Error('Circuit breaker is open');
    }

    try {
      const result = await fn();
      this.failures = 0;  // Reset on success
      return result;
    } catch (error) {
      this.failures++;
      this.lastFailure = Date.now();
      throw error;
    }
  }

  private isOpen(): boolean {
    if (this.failures >= this.threshold) {
      // Check if reset timeout has passed
      if (Date.now() - this.lastFailure > this.resetTimeout) {
        this.failures = 0;
        return false;
      }
      return true;
    }
    return false;
  }
}

const paymentCircuit = new CircuitBreaker();

export const handler = async (event: any) => {
  try {
    return await paymentCircuit.execute(() => 
      paymentService.charge(event.amount)
    );
  } catch (error) {
    if (error.message === 'Circuit breaker is open') {
      // Return cached/fallback response
      return { status: 'pending', message: 'Payment service temporarily unavailable' };
    }
    throw error;
  }
};
```

### Observability

```typescript
// ════════════════════════════════════════════════════════════════
// STRUCTURED LOGGING
// ════════════════════════════════════════════════════════════════

import { Logger } from '@aws-lambda-powertools/logger';

const logger = new Logger({
  serviceName: 'orders-api',
  logLevel: 'INFO'
});

export const handler = async (event: any, context: any) => {
  // Add request context
  logger.addContext(context);
  
  logger.info('Processing order', {
    orderId: event.orderId,
    customerId: event.customerId
  });

  try {
    const result = await processOrder(event);
    logger.info('Order processed successfully', { result });
    return result;
  } catch (error) {
    logger.error('Order processing failed', { error, event });
    throw error;
  }
};

// ════════════════════════════════════════════════════════════════
// DISTRIBUTED TRACING (X-Ray)
// ════════════════════════════════════════════════════════════════

import { Tracer } from '@aws-lambda-powertools/tracer';

const tracer = new Tracer({ serviceName: 'orders-api' });

export const handler = async (event: any) => {
  const segment = tracer.getSegment();
  
  // Create subsegment for database call
  const dbSegment = segment?.addNewSubsegment('DynamoDB');
  try {
    const result = await dynamodb.getItem({ /* ... */ });
    dbSegment?.close();
    return result;
  } catch (error) {
    dbSegment?.addError(error);
    dbSegment?.close();
    throw error;
  }
};

// ════════════════════════════════════════════════════════════════
// CUSTOM METRICS
// ════════════════════════════════════════════════════════════════

import { Metrics, MetricUnits } from '@aws-lambda-powertools/metrics';

const metrics = new Metrics({ serviceName: 'orders-api' });

export const handler = async (event: any) => {
  const startTime = Date.now();
  
  try {
    const result = await processOrder(event);
    
    // Track success
    metrics.addMetric('OrderProcessed', MetricUnits.Count, 1);
    metrics.addMetric('OrderValue', MetricUnits.Count, result.total);
    
    return result;
  } catch (error) {
    // Track failure
    metrics.addMetric('OrderFailed', MetricUnits.Count, 1);
    throw error;
  } finally {
    // Track duration
    metrics.addMetric('ProcessingTime', MetricUnits.Milliseconds, Date.now() - startTime);
    metrics.publishStoredMetrics();
  }
};
```

---

## 6. Limitations & Gotchas

### Hard Limits (AWS Lambda)

```
┌─────────────────────────────────────────────────────────────────┐
│                    AWS LAMBDA LIMITS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  EXECUTION:                                                     │
│  ├── Max duration           │ 15 minutes (900 seconds)         │
│  ├── Memory                 │ 128 MB - 10,240 MB               │
│  ├── vCPUs                  │ Proportional to memory           │
│  ├── /tmp storage           │ 512 MB - 10,240 MB               │
│  └── Environment vars       │ 4 KB total                       │
│                                                                  │
│  PAYLOAD:                                                       │
│  ├── Sync invocation        │ 6 MB (request + response)        │
│  ├── Async invocation       │ 256 KB                           │
│  └── Streamed response      │ 20 MB                            │
│                                                                  │
│  CONCURRENCY:                                                   │
│  ├── Account default        │ 1,000 concurrent executions      │
│  ├── Per-function reserved  │ Up to account limit              │
│  └── Provisioned            │ Costs extra, keeps warm          │
│                                                                  │
│  DEPLOYMENT:                                                    │
│  ├── Package size (zipped)  │ 50 MB                            │
│  ├── Package size (unzipped)│ 250 MB                           │
│  └── Container image        │ 10 GB                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Common Gotchas

```typescript
// ════════════════════════════════════════════════════════════════
// GOTCHA 1: Function timeout during processing
// ════════════════════════════════════════════════════════════════

// ❌ BAD: Processing all items in one invocation
export const handler = async (event: any) => {
  const items = await db.query('SELECT * FROM huge_table');  // 1M rows
  
  for (const item of items) {
    await processItem(item);  // Times out after 15 minutes!
  }
};

// ✅ GOOD: Chunk processing with SQS
export const handler = async (event: any) => {
  const { offset, limit } = event;
  const items = await db.query(`SELECT * FROM huge_table LIMIT ${limit} OFFSET ${offset}`);
  
  for (const item of items) {
    await processItem(item);
  }
  
  // Queue next batch
  if (items.length === limit) {
    await sqs.sendMessage({
      QueueUrl: process.env.SELF_QUEUE_URL,
      MessageBody: JSON.stringify({ offset: offset + limit, limit })
    });
  }
};

// ════════════════════════════════════════════════════════════════
// GOTCHA 2: Payload size limits
// ════════════════════════════════════════════════════════════════

// ❌ BAD: Returning large data directly
export const handler = async (event: any) => {
  const report = await generateLargeReport();  // 10MB
  return {
    statusCode: 200,
    body: JSON.stringify(report)  // FAILS: > 6MB limit
  };
};

// ✅ GOOD: Use S3 for large payloads
export const handler = async (event: any) => {
  const report = await generateLargeReport();
  
  // Upload to S3
  const key = `reports/${Date.now()}.json`;
  await s3.putObject({
    Bucket: 'my-reports-bucket',
    Key: key,
    Body: JSON.stringify(report)
  });
  
  // Return presigned URL
  const url = await getSignedUrl(s3, new GetObjectCommand({
    Bucket: 'my-reports-bucket',
    Key: key
  }), { expiresIn: 3600 });
  
  return {
    statusCode: 200,
    body: JSON.stringify({ downloadUrl: url })
  };
};

// ════════════════════════════════════════════════════════════════
// GOTCHA 3: Cold start + VPC = Slow
// ════════════════════════════════════════════════════════════════

// VPC attachment adds ~1-2s to cold starts
// Use VPC only when necessary (RDS, ElastiCache, etc.)

// Alternative: Use VPC endpoints to avoid NAT Gateway
// serverless.yml:
// provider:
//   vpc:
//     securityGroupIds: [!Ref LambdaSG]
//     subnetIds: [!Ref PrivateSubnet1, !Ref PrivateSubnet2]
//   vpcEndpointIds:
//     - !Ref DynamoDBEndpoint  # Access DynamoDB without internet

// ════════════════════════════════════════════════════════════════
// GOTCHA 4: Statelessness - No persistent connections
// ════════════════════════════════════════════════════════════════

// ❌ BAD: WebSocket connections
// Lambda can't maintain persistent WebSocket connections

// ✅ GOOD: Use API Gateway WebSocket APIs
// API Gateway manages connections, Lambda handles messages

// ❌ BAD: In-memory caching between requests
let cache = {};  // Lost when container recycles!

// ✅ GOOD: External cache (Redis, DynamoDB)
const redis = new Redis(process.env.REDIS_URL);

export const handler = async (event: any) => {
  let data = await redis.get('my-key');
  if (!data) {
    data = await expensiveOperation();
    await redis.set('my-key', data, 'EX', 3600);
  }
  return data;
};

// ════════════════════════════════════════════════════════════════
// GOTCHA 5: Concurrent execution limits (Thundering Herd)
// ════════════════════════════════════════════════════════════════

// Problem: Traffic spike exhausts concurrent execution limit
// All additional requests fail with 429 (throttled)

// Solutions:
// 1. Request limit increase from AWS (up to 10,000+)
// 2. Use reserved concurrency to guarantee capacity
// 3. Use SQS to smooth traffic (queue absorbs spikes)
// 4. Implement backoff in clients

// ════════════════════════════════════════════════════════════════
// GOTCHA 6: No GPU support
// ════════════════════════════════════════════════════════════════

// Lambda doesn't support GPU workloads
// For ML inference, use:
// - SageMaker endpoints (managed ML)
// - EC2 with GPU instances
// - AWS Inferentia chips
// - Bedrock for LLMs

// ════════════════════════════════════════════════════════════════
// GOTCHA 7: Time-based triggers aren't precise
// ════════════════════════════════════════════════════════════════

// CloudWatch Events can have up to 1 minute delay
// For precise timing, use Step Functions wait states
// or EC2/ECS with cron
```

---

## 7. Cost Analysis

### Lambda Pricing Model

```
┌─────────────────────────────────────────────────────────────────┐
│                    LAMBDA PRICING (US-EAST-1)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WHAT YOU PAY FOR:                                              │
│  ├── Requests: $0.20 per 1 million requests                    │
│  ├── Duration: $0.0000166667 per GB-second                     │
│  └── Provisioned: $0.000004646 per GB-second (always on)       │
│                                                                  │
│  FREE TIER (Monthly):                                           │
│  ├── 1 million requests                                        │
│  └── 400,000 GB-seconds                                        │
│                                                                  │
│  EXAMPLE CALCULATION:                                           │
│  ─────────────────────────────────────────────────────────────  │
│  Requests: 10 million/month                                    │
│  Memory: 512 MB (0.5 GB)                                       │
│  Duration: 200ms average                                       │
│                                                                  │
│  GB-seconds = 10M × 0.5 GB × 0.2s = 1,000,000 GB-s            │
│                                                                  │
│  Cost breakdown:                                                │
│  Requests: 10M × $0.20/M = $2.00                              │
│  Duration: 1M GB-s × $0.0000166667 = $16.67                   │
│  ─────────────────────────────────────────────────────────────  │
│  TOTAL: ~$18.67/month for 10M requests                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Serverless vs Containers Cost Comparison

```
COST COMPARISON: When does serverless become expensive?

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SCENARIO 1: Low, Variable Traffic (Serverless wins!)          │
│  ─────────────────────────────────────────────────────────────  │
│  Traffic: 100K requests/month, 200ms avg                       │
│                                                                  │
│  Lambda (512MB):                                                │
│  └── Requests: 100K × $0.20/M = $0.02                         │
│  └── Duration: 100K × 0.5GB × 0.2s × $0.0000166667 = $0.17    │
│  └── TOTAL: ~$0.19/month                                       │
│                                                                  │
│  Fargate (0.5 vCPU, 1GB):                                      │
│  └── 24/7 uptime: ~$30/month minimum                           │
│                                                                  │
│  VERDICT: Serverless is 150x cheaper                           │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SCENARIO 2: High, Steady Traffic (Containers win!)            │
│  ─────────────────────────────────────────────────────────────  │
│  Traffic: 100M requests/month, 200ms avg, 24/7 steady          │
│                                                                  │
│  Lambda (512MB):                                                │
│  └── Requests: 100M × $0.20/M = $20                           │
│  └── Duration: 100M × 0.5GB × 0.2s × $0.0000166667 = $166.67  │
│  └── TOTAL: ~$186.67/month                                     │
│                                                                  │
│  Fargate (1 vCPU, 2GB, 3 instances):                           │
│  └── 3 × $35/month = $105/month                                │
│  └── Handles same load with headroom                           │
│                                                                  │
│  VERDICT: Containers are ~44% cheaper                          │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CROSSOVER POINT:                                               │
│  ─────────────────────────────────────────────────────────────  │
│  Generally, serverless is cheaper until:                       │
│  └── ~30-50% utilization of container equivalent               │
│  └── Or ~5-10M requests/month for typical APIs                 │
│                                                                  │
│  But consider:                                                  │
│  └── Serverless: No ops overhead ($$$ saved)                  │
│  └── Serverless: Auto-scaling included                        │
│  └── Containers: Predictable costs                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. When to Use / Not Use

### When TO Use Serverless

```
✅ USE SERVERLESS WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. VARIABLE/UNPREDICTABLE TRAFFIC                             │
│     └── Spiky traffic patterns                                 │
│     └── Scale to zero when idle                                │
│     └── Handle traffic bursts automatically                    │
│                                                                  │
│  2. EVENT-DRIVEN WORKLOADS                                      │
│     └── File processing (S3 uploads)                           │
│     └── Queue consumers (SQS, Kinesis)                         │
│     └── Webhooks                                               │
│     └── IoT data processing                                    │
│                                                                  │
│  3. SCHEDULED TASKS                                             │
│     └── Cron jobs that run periodically                        │
│     └── Report generation                                      │
│     └── Data cleanup                                           │
│                                                                  │
│  4. APIs WITH VARIABLE LOAD                                     │
│     └── MVPs and prototypes                                    │
│     └── Internal tools                                         │
│     └── Low-traffic production APIs                           │
│                                                                  │
│  5. MICROSERVICES / FUNCTIONS                                   │
│     └── Small, single-purpose functions                        │
│     └── Decoupled services                                     │
│                                                                  │
│  6. NO OPS CAPACITY                                             │
│     └── Small teams                                            │
│     └── Focus on product, not infrastructure                   │
│                                                                  │
│  GOOD EXAMPLES:                                                 │
│  └── Image/video processing triggers                          │
│  └── Notification systems                                      │
│  └── Chatbots and webhooks                                    │
│  └── Data transformation pipelines                            │
│  └── Scheduled reports and cleanups                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When NOT to Use Serverless

```
❌ DON'T USE SERVERLESS WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. LONG-RUNNING PROCESSES                                      │
│     └── Tasks > 15 minutes                                     │
│     └── Video transcoding (use MediaConvert)                   │
│     └── Large data processing (use EMR, Glue)                 │
│     └── Training ML models (use SageMaker)                    │
│                                                                  │
│  2. PERSISTENT CONNECTIONS                                      │
│     └── WebSocket servers (Lambda can't hold connections)     │
│     └── Long-polling                                           │
│     └── Game servers                                           │
│                                                                  │
│  3. HIGH, STEADY TRAFFIC                                        │
│     └── Predictable 24/7 load                                  │
│     └── Containers become cheaper                              │
│     └── Reserved instances even cheaper                       │
│                                                                  │
│  4. LATENCY-CRITICAL (<50ms)                                   │
│     └── Cold starts add 100ms-10s                              │
│     └── Use containers or provisioned concurrency             │
│                                                                  │
│  5. GPU/SPECIALIZED HARDWARE                                    │
│     └── ML inference (use SageMaker)                          │
│     └── Graphics processing                                    │
│     └── Scientific computing                                   │
│                                                                  │
│  6. COMPLEX STATEFUL APPLICATIONS                               │
│     └── Applications requiring in-memory state                │
│     └── Databases (obviously)                                  │
│                                                                  │
│  7. LARGE MONOLITHS                                             │
│     └── Cold start for 500MB package is painful               │
│     └── Exceeds 250MB unzipped limit                          │
│                                                                  │
│  BETTER ALTERNATIVES:                                          │
│  └── ECS/Fargate for long-running services                    │
│  └── EC2 for GPU workloads                                    │
│  └── AppRunner for simple containers                          │
│  └── EKS for complex orchestration                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Interview Questions & Answers

### Basic Questions

**Q1: What is serverless computing?**
> **A:** Serverless is a cloud execution model where the provider manages all infrastructure. Code runs in stateless, ephemeral containers triggered by events. Key characteristics:
> - **Auto-scaling**: From 0 to thousands of instances automatically
> - **Pay-per-execution**: Billed only for actual compute time
> - **No server management**: Provider handles provisioning, patching, scaling
> - **Event-driven**: Functions triggered by HTTP, queues, file uploads, schedules

**Q2: What is a cold start?**
> **A:** The latency when a new container must be initialized to handle a request. It includes:
> - Container provisioning (~50-100ms)
> - Code download (~50-200ms)
> - Runtime startup (~100ms-3s depending on language)
> - Init code execution (variable)
>
> Total: 100ms (Go) to 10+ seconds (Java with VPC). Warm starts reuse existing containers and take only a few milliseconds.

**Q3: How do you mitigate cold starts?**
> **A:** Several strategies:
> 1. **Provisioned concurrency** - Keep warm instances always ready ($$$)
> 2. **Smaller packages** - Less code to download (tree-shaking, esbuild)
> 3. **Faster runtimes** - Node.js/Go over Java
> 4. **Lazy loading** - Defer expensive initialization until needed
> 5. **Avoid VPC** - VPC adds ~1-2s for ENI creation
> 6. **Edge functions** - V8 isolates have near-zero cold starts
> 7. **Warming pings** - Scheduled invocations to keep containers hot

**Q4: What are the limitations of AWS Lambda?**
> **A:** Key limits:
> - **15 minute max duration** - Long processes need other solutions
> - **6MB sync payload** - Use S3 for larger data
> - **10GB max memory** - Can limit compute-heavy workloads
> - **No persistent connections** - Can't hold WebSocket connections
> - **Cold starts** - Latency for new containers
> - **No GPU** - ML workloads need SageMaker/EC2
> - **Vendor lock-in** - AWS-specific APIs

### Intermediate Questions

**Q5: How do you handle database connections in serverless?**
> **A:** Three approaches:
> 1. **Connection reuse** - Create connection outside handler, reuse in warm invocations
> 2. **Connection pooling (RDS Proxy)** - Proxy sits between Lambda and DB, manages pool
> 3. **HTTP-based databases** - PlanetScale, Neon use HTTP protocols, no persistent connections
>
> Key: Set `max: 1` connection per Lambda instance to avoid exhausting database connections during spikes.

**Q6: What is the difference between sync and async Lambda invocation?**
> **A:** 
> - **Synchronous**: Caller waits for response. API Gateway uses this. 6MB payload limit.
> - **Asynchronous**: Fire-and-forget. S3, SNS use this. 256KB payload limit. Built-in retries (2 attempts).
> - **Polling**: Lambda polls event source. SQS, Kinesis, DynamoDB Streams. Lambda controls batch size and concurrency.

**Q7: How do you handle errors and retries?**
> **A:** Depends on invocation type:
> - **Sync**: Return error to caller, they retry
> - **Async**: Automatic 2 retries, then to Dead Letter Queue (DLQ)
> - **Stream/Queue**: Message returns to queue after visibility timeout
>
> Best practices:
> - Make handlers **idempotent** (same input = same result)
> - Use **DLQ** for failed messages
> - Implement **circuit breaker** for external services
> - Use **SQS partial batch response** to report individual failures

**Q8: What is the execution context?**
> **A:** The runtime environment that persists between invocations in the same container:
> - Code outside the handler runs once per container
> - Variables, connections, cached data persist
> - Reused for 5-15 minutes of inactivity
>
> Use it to: Initialize SDK clients, database connections, load configuration. Don't use it for: Request-specific state.

### Advanced Questions

**Q9: Compare Lambda vs Containers. When would you choose each?**
> **A:**
> **Lambda**:
> - Variable/unpredictable traffic
> - Event-driven workloads
> - Short-running tasks (<15 min)
> - No ops capacity
> - Pay-per-execution economics
>
> **Containers**:
> - Steady, predictable traffic (cheaper at scale)
> - Long-running processes
> - Persistent connections (WebSocket)
> - Latency-critical (<50ms)
> - Large applications (>250MB)
>
> Crossover point: ~30-50% container utilization, or ~5-10M Lambda requests/month.

**Q10: How do you implement backpressure in serverless?**
> **A:** Several mechanisms:
> 1. **Reserved concurrency** - Limit max concurrent executions per function
> 2. **SQS with maxConcurrency** - Control polling rate
> 3. **API Gateway throttling** - Rate limit at API level
> 4. **Circuit breaker** - Stop calling failing services
> 5. **Partial batch failures** - Report failed messages individually, don't retry all
>
> Example: Set `reservedConcurrency: 10` to limit function to 10 parallel executions, protecting downstream databases.

**Q11: What are edge functions? When would you use them?**
> **A:** Functions that run at CDN edge locations (Cloudflare Workers, Vercel Edge):
> - **V8 isolates** instead of containers = ~0ms cold start
> - **Limited runtime** - No Node.js APIs, smaller libraries
> - **Low latency** - Run close to users globally
>
> Use for: Auth checks, redirects, A/B testing, geolocation, header manipulation
> Not for: Database queries, complex processing

**Q12: How do you design for observability in serverless?**
> **A:** Three pillars:
> 1. **Structured logging** - JSON logs with correlation IDs, request context
> 2. **Distributed tracing** - X-Ray to trace requests across functions
> 3. **Custom metrics** - Business metrics (orders processed, errors by type)
>
> Tools: CloudWatch Logs Insights, X-Ray, Lambda Powertools (structured logging, tracing, metrics in one package).

### Scenario Questions

**Q13: Design a serverless image processing pipeline**
> **A:** 
> ```
> User uploads to S3 → S3 triggers Lambda → Lambda processes image → 
> Lambda saves thumbnail to S3 → S3 triggers notification Lambda → 
> Lambda updates DB and notifies user
> ```
>
> Considerations:
> - Handle large images: Stream from S3, don't load entire file
> - Prevent infinite loops: Check prefix before processing
> - Timeout: 15 min max, use Step Functions for long processing
> - Error handling: DLQ for failed images, retry logic
> - Scaling: Lambda scales automatically, but limit concurrency if DB is bottleneck

**Q14: You're getting cold starts of 3-5 seconds. How do you debug and fix?**
> **A:** Debug:
> 1. Check runtime (Java = slow)
> 2. Check VPC (adds 1-2s)
> 3. Check package size (large = slow download)
> 4. Check init code (what runs outside handler?)
>
> Fix:
> 1. Switch to Node.js/Go if possible
> 2. Remove VPC if not needed, or use Hyperplane
> 3. Use esbuild for tree-shaking, smaller packages
> 4. Lazy-load dependencies
> 5. Use provisioned concurrency for critical paths
> 6. Consider edge functions for latency-critical endpoints

---

## 🎓 Key Takeaways

1. **Serverless = no server management** + auto-scaling + pay-per-execution
2. **Cold starts** are the main latency concern - 100ms to 10s
3. **Mitigate cold starts** with provisioned concurrency, smaller packages, faster runtimes
4. **Execution context** persists between invocations - reuse connections
5. **Limits**: 15 min duration, 6MB payload, no persistent connections
6. **Database connections**: Use pooling (RDS Proxy) or HTTP-based DBs
7. **Edge functions** have ~0ms cold start but limited runtime
8. **Cost crossover**: Serverless cheaper for variable traffic, containers for steady load
9. **Make handlers idempotent** - same input = same result
10. **Backpressure**: Reserved concurrency, SQS throttling, circuit breakers

---

## 📚 Resources

### Documentation
- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/)
- [Cloudflare Workers Docs](https://developers.cloudflare.com/workers/)
- [Vercel Edge Functions](https://vercel.com/docs/functions/edge-functions)

### Tools
- [Serverless Framework](https://www.serverless.com/)
- [AWS SAM](https://aws.amazon.com/serverless/sam/)
- [SST (Serverless Stack)](https://sst.dev/)
- [Lambda Powertools](https://docs.powertools.aws.dev/lambda/typescript/latest/)

### Books & Courses
- "Serverless Architectures on AWS" by Peter Sbarski
- AWS Certified Solutions Architect (covers Lambda patterns)
- freeCodeCamp Serverless tutorials


