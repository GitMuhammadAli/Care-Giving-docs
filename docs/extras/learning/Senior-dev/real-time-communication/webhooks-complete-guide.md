# 🪝 Webhooks - Complete Guide

> A comprehensive guide to webhooks - event delivery, retry strategies, signature verification (HMAC), idempotency, security best practices, and building reliable webhook systems.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Webhooks are HTTP callbacks that notify external systems of events in real-time by POSTing JSON payloads to subscriber-configured URLs, requiring signature verification (HMAC) for security, retry logic for reliability, and idempotency handling for exactly-once processing."

### The 7 Key Concepts (Remember These!)
```
1. HTTP CALLBACK    → POST request to subscriber's URL when event occurs
2. HMAC SIGNATURE   → Cryptographic signature to verify payload authenticity
3. RETRY LOGIC      → Exponential backoff for failed deliveries
4. IDEMPOTENCY      → Handling duplicate deliveries (same event, multiple times)
5. PAYLOAD          → JSON data describing the event
6. DELIVERY STATUS  → Track success/failure of webhook deliveries
7. DEAD LETTER      → Store permanently failed webhooks for investigation
```

### Webhook Flow Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                    WEBHOOK DELIVERY FLOW                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  YOUR APPLICATION                                               │
│  ┌──────────────┐                                              │
│  │ Event Occurs │                                              │
│  │ (payment,    │                                              │
│  │  order, etc) │                                              │
│  └──────┬───────┘                                              │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────────────────────────┐                      │
│  │        WEBHOOK QUEUE                  │                      │
│  │  • Queue delivery jobs               │                      │
│  │  • Handles retries                   │                      │
│  │  • Tracks delivery status            │                      │
│  └──────────────┬───────────────────────┘                      │
│                 │                                               │
│                 │ 1. Sign payload (HMAC)                       │
│                 │ 2. POST to endpoint                          │
│                 │ 3. Verify response (2xx)                     │
│                 ▼                                               │
│  ════════════════════════════════════════════════════════      │
│                 │                                               │
│                 ▼                                               │
│  SUBSCRIBER'S ENDPOINT                                         │
│  ┌──────────────────────────────────────┐                      │
│  │ 1. Verify signature                  │                      │
│  │ 2. Check idempotency key             │                      │
│  │ 3. Process event                     │                      │
│  │ 4. Return 200 OK                     │                      │
│  └──────────────────────────────────────┘                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### HMAC Signature Verification
```
┌─────────────────────────────────────────────────────────────────┐
│                   HMAC SIGNATURE FLOW                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SENDER SIDE:                                                   │
│  ─────────────                                                  │
│  1. payload = JSON.stringify(event)                            │
│  2. signature = HMAC-SHA256(payload, secret_key)               │
│  3. Send: POST /webhook                                        │
│           Header: X-Signature: sha256=<signature>              │
│           Body: <payload>                                      │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  RECEIVER SIDE:                                                 │
│  ──────────────                                                 │
│  1. Receive payload and signature header                       │
│  2. expected = HMAC-SHA256(payload, my_secret_key)             │
│  3. Compare: timingSafeEqual(expected, received_signature)     │
│  4. If match → process event                                   │
│     If no match → reject (401 Unauthorized)                    │
│                                                                 │
│  ⚠️  ALWAYS use timing-safe comparison to prevent timing       │
│      attacks!                                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Retry Strategy (Exponential Backoff)
```
┌─────────────────────────────────────────────────────────────────┐
│                 EXPONENTIAL BACKOFF STRATEGY                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Attempt 1: Immediate                                          │
│      ↓ (fail)                                                  │
│  Attempt 2: Wait 1 minute                                      │
│      ↓ (fail)                                                  │
│  Attempt 3: Wait 5 minutes                                     │
│      ↓ (fail)                                                  │
│  Attempt 4: Wait 30 minutes                                    │
│      ↓ (fail)                                                  │
│  Attempt 5: Wait 2 hours                                       │
│      ↓ (fail)                                                  │
│  Attempt 6: Wait 8 hours                                       │
│      ↓ (fail)                                                  │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━      │
│  Move to Dead Letter Queue                                     │
│  Notify admin                                                   │
│  Allow manual retry                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"HMAC-SHA256"** | "We sign payloads with HMAC-SHA256 for authenticity verification" |
| **"Timing-safe comparison"** | "We use timing-safe comparison to prevent timing attacks on signature verification" |
| **"Idempotency key"** | "Each event has an idempotency key so receivers can deduplicate" |
| **"Dead letter queue"** | "Failed webhooks go to dead letter queue for investigation" |
| **"Exponential backoff"** | "We retry with exponential backoff: 1m, 5m, 30m, 2h, 8h" |
| **"Delivery guarantee"** | "We provide at-least-once delivery, receivers must be idempotent" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Response timeout | **5-30 seconds** | Don't wait forever |
| Max retry attempts | **5-6** | Enough time for temporary issues |
| Total retry window | **24-72 hours** | Cover most outages |
| Success status codes | **2xx** | Only 200-299 means success |
| Signature header | **X-Signature** or **X-Hub-Signature-256** | Common conventions |
| Payload size limit | **64KB-1MB** | Reasonable for JSON |

### The "Wow" Statement (Memorize This!)
> "We built a webhook system that delivers 10M+ events daily. Events are queued in Redis (BullMQ) and workers POST to subscriber endpoints with HMAC-SHA256 signatures in the X-Signature header. We implement exponential backoff (1m, 5m, 30m, 2h, 8h) with max 6 retries over 72 hours. The receiver side verifies signatures using timing-safe comparison to prevent timing attacks, checks the idempotency key against their database to handle duplicates, then returns 200 immediately while processing asynchronously. Failed deliveries after all retries go to a dead letter queue with an admin dashboard for investigation and manual retry. We track delivery metrics (success rate, latency percentiles) and alert when a subscriber's endpoint has >10% failure rate."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌────────────┐                                                │
│  │   Event    │                                                │
│  │  Producer  │                                                │
│  └─────┬──────┘                                                │
│        │                                                        │
│        ▼                                                        │
│  ┌────────────────────────────────────────────────────────┐   │
│  │                   WEBHOOK SERVICE                       │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │   │
│  │  │ Subscriptions │  │   Delivery   │  │   Delivery   │ │   │
│  │  │   Database    │  │    Queue     │  │   Workers    │ │   │
│  │  │ (endpoints,   │  │  (Redis/     │  │ (sign, send, │ │   │
│  │  │  secrets)     │  │   BullMQ)    │  │  retry)      │ │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘ │   │
│  │                                                         │   │
│  │  ┌──────────────┐  ┌──────────────┐                    │   │
│  │  │ Dead Letter  │  │   Metrics    │                    │   │
│  │  │   Queue      │  │  Dashboard   │                    │   │
│  │  └──────────────┘  └──────────────┘                    │   │
│  └────────────────────────────────────────────────────────┘   │
│        │                                                        │
│        │ POST + Signature                                       │
│        ▼                                                        │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐   │
│  │  Subscriber A  │  │  Subscriber B  │  │  Subscriber C  │   │
│  └────────────────┘  └────────────────┘  └────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What are webhooks?"**
> "HTTP callbacks that notify external systems of events. When something happens (payment, order), you POST a JSON payload to subscriber-configured URLs. Enables real-time integration without polling."

**Q: "How do you secure webhooks?"**
> "Sign payloads with HMAC-SHA256 using a shared secret. Receiver verifies signature with timing-safe comparison. Include timestamp to prevent replay attacks. Use HTTPS only. IP whitelist if possible."

**Q: "How do you handle failed deliveries?"**
> "Retry with exponential backoff (1m, 5m, 30m, etc.). Track attempt count and status. After max retries (5-6), move to dead letter queue. Provide webhook logs and manual retry in dashboard."

**Q: "What is idempotency in webhooks?"**
> "Receivers may get the same event multiple times (network issues, retries). Include idempotency key (event ID). Receiver stores processed IDs and skips duplicates. Essential for at-least-once delivery."

**Q: "What response indicates success?"**
> "Any 2xx status code (200-299). Anything else (4xx, 5xx, timeout) triggers retry. Recommend receivers respond quickly (200) and process asynchronously."

**Q: "How do big platforms do webhooks (Stripe, GitHub)?"**
> "Queue-based delivery, HMAC signatures (stripe-signature, X-Hub-Signature-256), timestamp in signature to prevent replay, exponential backoff, webhook logs in dashboard, event IDs for idempotency, test endpoints."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How do webhooks work?"

**Junior Answer:**
> "You send an HTTP POST to a URL when something happens."

**Senior Answer:**
> "Webhooks are an event notification pattern:

1. **Subscription**: User registers their endpoint URL and receives a secret key for verification.

2. **Event triggers**: When an event occurs, we create a webhook delivery job:
   - Generate idempotency key (event ID)
   - Serialize payload to JSON
   - Sign payload with HMAC-SHA256 using subscriber's secret
   - Queue for delivery

3. **Delivery**: Worker POSTs to endpoint with signature header. If 2xx response, mark delivered. Otherwise, schedule retry with exponential backoff.

4. **Receiver responsibilities**: Verify signature (timing-safe), check idempotency key, respond 200 quickly, process asynchronously. Never trust unverified webhooks.

The key insight is webhooks provide 'at-least-once' delivery, not exactly-once. Network issues, retries, and our reliability measures mean receivers MUST be idempotent."

### When Asked: "How do you make webhooks reliable?"

**Junior Answer:**
> "Retry if it fails."

**Senior Answer:**
> "Reliability has multiple layers:

1. **Queue-based delivery**: Don't send inline - queue ensures event survives app crashes.

2. **Retry strategy**: Exponential backoff (1m, 5m, 30m, 2h, 8h) gives subscribers time to recover from outages while not hammering failing endpoints.

3. **Circuit breaker**: If an endpoint fails consistently, pause deliveries temporarily rather than burning through retries.

4. **Dead letter queue**: After max retries, store for investigation. Don't lose events.

5. **Monitoring**: Track success rates per endpoint. Alert on degradation.

6. **Receiver guidance**: Tell subscribers to respond 200 immediately, process async. Long-running handlers cause timeouts and unnecessary retries.

7. **Replay capability**: Allow subscribers to request redelivery of past events."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "Why HMAC not JWT?" | "HMAC is simpler, symmetric. Both parties share the secret. JWT adds complexity (asymmetric, parsing) that's unnecessary for webhook verification." |
| "Why timing-safe comparison?" | "Regular string comparison can leak signature length via timing. Attackers measure response times to guess valid signatures character by character." |
| "How do you handle ordering?" | "Webhooks may arrive out of order. Include timestamp or sequence number. Receivers should handle out-of-order delivery or check 'updated_at' before applying changes." |
| "What about webhook testing?" | "Provide test endpoints to validate receiver implementation. Allow sending test events. Tools like ngrok for local development." |

---

## 📚 Table of Contents

1. [Sending Webhooks](#1-sending-webhooks)
2. [Receiving Webhooks](#2-receiving-webhooks)
3. [HMAC Signature Verification](#3-hmac-signature-verification)
4. [Retry Strategies](#4-retry-strategies)
5. [Idempotency](#5-idempotency)
6. [Queue-Based Delivery](#6-queue-based-delivery)
7. [Webhook Management API](#7-webhook-management-api)
8. [Common Pitfalls](#8-common-pitfalls)
9. [Interview Questions](#9-interview-questions)

---

## 1. Sending Webhooks

```typescript
// ════════════════════════════════════════════════════════════════
// WEBHOOK SENDER SERVICE
// ════════════════════════════════════════════════════════════════

import crypto from 'crypto';
import axios from 'axios';

interface WebhookPayload {
  id: string;           // Unique event ID (idempotency key)
  type: string;         // Event type: 'order.created', 'payment.succeeded'
  timestamp: string;    // ISO timestamp
  data: any;            // Event-specific data
}

interface WebhookSubscription {
  id: string;
  url: string;
  secret: string;
  events: string[];     // Subscribed event types
  active: boolean;
}

// Generate HMAC signature
function generateSignature(payload: string, secret: string): string {
  return crypto
    .createHmac('sha256', secret)
    .update(payload, 'utf8')
    .digest('hex');
}

// Send webhook with signature
async function sendWebhook(
  subscription: WebhookSubscription,
  event: WebhookPayload
): Promise<{
  success: boolean;
  statusCode?: number;
  error?: string;
  duration: number;
}> {
  const payloadString = JSON.stringify(event);
  const timestamp = Math.floor(Date.now() / 1000);
  
  // Include timestamp in signature to prevent replay attacks
  const signaturePayload = `${timestamp}.${payloadString}`;
  const signature = generateSignature(signaturePayload, subscription.secret);

  const startTime = Date.now();

  try {
    const response = await axios.post(subscription.url, event, {
      headers: {
        'Content-Type': 'application/json',
        'X-Webhook-ID': event.id,
        'X-Webhook-Timestamp': timestamp.toString(),
        'X-Webhook-Signature': `v1=${signature}`,
        'User-Agent': 'MyApp-Webhooks/1.0'
      },
      timeout: 30000,  // 30 second timeout
      validateStatus: () => true  // Don't throw on non-2xx
    });

    const duration = Date.now() - startTime;

    if (response.status >= 200 && response.status < 300) {
      return { success: true, statusCode: response.status, duration };
    } else {
      return {
        success: false,
        statusCode: response.status,
        error: `HTTP ${response.status}`,
        duration
      };
    }
  } catch (error: any) {
    const duration = Date.now() - startTime;
    
    if (error.code === 'ECONNABORTED') {
      return { success: false, error: 'Timeout', duration };
    }
    if (error.code === 'ENOTFOUND') {
      return { success: false, error: 'DNS resolution failed', duration };
    }
    if (error.code === 'ECONNREFUSED') {
      return { success: false, error: 'Connection refused', duration };
    }
    
    return { success: false, error: error.message, duration };
  }
}

// ════════════════════════════════════════════════════════════════
// WEBHOOK EVENT TYPES
// ════════════════════════════════════════════════════════════════

// Define your event types
type WebhookEventType =
  | 'order.created'
  | 'order.updated'
  | 'order.cancelled'
  | 'payment.succeeded'
  | 'payment.failed'
  | 'customer.created'
  | 'customer.updated';

interface OrderCreatedEvent {
  type: 'order.created';
  data: {
    orderId: string;
    customerId: string;
    items: Array<{ productId: string; quantity: number; price: number }>;
    total: number;
    currency: string;
    createdAt: string;
  };
}

interface PaymentSucceededEvent {
  type: 'payment.succeeded';
  data: {
    paymentId: string;
    orderId: string;
    amount: number;
    currency: string;
    method: string;
    paidAt: string;
  };
}

// Event emitter for webhook dispatch
import { EventEmitter } from 'events';

class WebhookEventEmitter extends EventEmitter {
  async emit(eventType: WebhookEventType, data: any): Promise<boolean> {
    const event: WebhookPayload = {
      id: `evt_${Date.now()}_${crypto.randomBytes(8).toString('hex')}`,
      type: eventType,
      timestamp: new Date().toISOString(),
      data
    };

    // Queue webhook delivery for all relevant subscriptions
    await this.queueWebhookDelivery(event);
    
    return super.emit(eventType, event);
  }

  private async queueWebhookDelivery(event: WebhookPayload) {
    // Get all subscriptions for this event type
    const subscriptions = await getSubscriptionsForEvent(event.type);
    
    for (const subscription of subscriptions) {
      await webhookQueue.add('deliver', {
        subscriptionId: subscription.id,
        event
      }, {
        attempts: 6,
        backoff: {
          type: 'custom'
        }
      });
    }
  }
}

const webhookEmitter = new WebhookEventEmitter();

// Usage
webhookEmitter.emit('order.created', {
  orderId: 'ord_123',
  customerId: 'cus_456',
  items: [{ productId: 'prod_789', quantity: 2, price: 29.99 }],
  total: 59.98,
  currency: 'USD',
  createdAt: new Date().toISOString()
});
```

---

## 2. Receiving Webhooks

```typescript
// ════════════════════════════════════════════════════════════════
// WEBHOOK RECEIVER (Express.js)
// ════════════════════════════════════════════════════════════════

import express from 'express';
import crypto from 'crypto';

const app = express();

// IMPORTANT: Use raw body for signature verification
app.use('/webhooks', express.raw({ type: 'application/json' }));

// Webhook secret from the sender
const WEBHOOK_SECRET = process.env.WEBHOOK_SECRET!;

// Verify webhook signature
function verifyWebhookSignature(
  payload: Buffer,
  signature: string,
  timestamp: string,
  secret: string
): boolean {
  // Check timestamp to prevent replay attacks (5 minute window)
  const currentTime = Math.floor(Date.now() / 1000);
  const webhookTime = parseInt(timestamp, 10);
  
  if (Math.abs(currentTime - webhookTime) > 300) {
    console.warn('Webhook timestamp too old');
    return false;
  }

  // Extract signature value
  const signatureParts = signature.split('=');
  if (signatureParts.length !== 2 || signatureParts[0] !== 'v1') {
    return false;
  }
  const receivedSignature = signatureParts[1];

  // Calculate expected signature
  const signaturePayload = `${timestamp}.${payload.toString()}`;
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(signaturePayload, 'utf8')
    .digest('hex');

  // IMPORTANT: Timing-safe comparison
  try {
    return crypto.timingSafeEqual(
      Buffer.from(receivedSignature, 'hex'),
      Buffer.from(expectedSignature, 'hex')
    );
  } catch {
    return false;
  }
}

// Webhook endpoint
app.post('/webhooks/myapp', async (req, res) => {
  const signature = req.headers['x-webhook-signature'] as string;
  const timestamp = req.headers['x-webhook-timestamp'] as string;
  const webhookId = req.headers['x-webhook-id'] as string;

  // Verify required headers
  if (!signature || !timestamp) {
    return res.status(401).json({ error: 'Missing signature headers' });
  }

  // Verify signature
  if (!verifyWebhookSignature(req.body, signature, timestamp, WEBHOOK_SECRET)) {
    console.warn('Invalid webhook signature');
    return res.status(401).json({ error: 'Invalid signature' });
  }

  // Parse payload
  let event: WebhookPayload;
  try {
    event = JSON.parse(req.body.toString());
  } catch {
    return res.status(400).json({ error: 'Invalid JSON' });
  }

  // Check idempotency - have we processed this event?
  const alreadyProcessed = await checkIdempotencyKey(event.id);
  if (alreadyProcessed) {
    console.log(`Duplicate webhook: ${event.id}`);
    return res.status(200).json({ message: 'Already processed' });
  }

  // Respond immediately, process asynchronously
  res.status(200).json({ received: true });

  // Process asynchronously
  processWebhookAsync(event).catch(error => {
    console.error('Webhook processing error:', error);
  });
});

// Process webhook asynchronously
async function processWebhookAsync(event: WebhookPayload) {
  try {
    // Record that we're processing this event
    await storeIdempotencyKey(event.id);

    // Route to appropriate handler
    switch (event.type) {
      case 'order.created':
        await handleOrderCreated(event.data);
        break;
      case 'payment.succeeded':
        await handlePaymentSucceeded(event.data);
        break;
      case 'payment.failed':
        await handlePaymentFailed(event.data);
        break;
      default:
        console.log(`Unhandled event type: ${event.type}`);
    }

    // Mark as successfully processed
    await markIdempotencyKeyProcessed(event.id);
  } catch (error) {
    // Mark as failed for potential retry/investigation
    await markIdempotencyKeyFailed(event.id, error);
    throw error;
  }
}

// Event handlers
async function handleOrderCreated(data: any) {
  console.log('Processing order.created:', data.orderId);
  // Sync order to your system
  await syncOrder(data);
}

async function handlePaymentSucceeded(data: any) {
  console.log('Processing payment.succeeded:', data.paymentId);
  // Update order status, send confirmation email, etc.
  await fulfillOrder(data.orderId);
}

async function handlePaymentFailed(data: any) {
  console.log('Processing payment.failed:', data.paymentId);
  // Notify customer, update order status
  await handleFailedPayment(data.orderId);
}

// ════════════════════════════════════════════════════════════════
// FRAMEWORK-SPECIFIC EXAMPLES
// ════════════════════════════════════════════════════════════════

// Next.js API Route
// app/api/webhooks/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const body = await request.text();
  const signature = request.headers.get('x-webhook-signature');
  const timestamp = request.headers.get('x-webhook-timestamp');

  if (!signature || !timestamp) {
    return NextResponse.json({ error: 'Missing headers' }, { status: 401 });
  }

  // Verify signature
  const isValid = verifyWebhookSignature(
    Buffer.from(body),
    signature,
    timestamp,
    process.env.WEBHOOK_SECRET!
  );

  if (!isValid) {
    return NextResponse.json({ error: 'Invalid signature' }, { status: 401 });
  }

  const event = JSON.parse(body);
  
  // Process event...
  
  return NextResponse.json({ received: true });
}

// Stripe webhook example
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);
const endpointSecret = process.env.STRIPE_WEBHOOK_SECRET!;

app.post('/webhooks/stripe', express.raw({ type: 'application/json' }), async (req, res) => {
  const sig = req.headers['stripe-signature'] as string;

  let event: Stripe.Event;

  try {
    // Stripe SDK handles signature verification
    event = stripe.webhooks.constructEvent(req.body, sig, endpointSecret);
  } catch (err: any) {
    console.error('Stripe webhook error:', err.message);
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }

  // Handle specific event types
  switch (event.type) {
    case 'payment_intent.succeeded':
      const paymentIntent = event.data.object as Stripe.PaymentIntent;
      console.log('Payment succeeded:', paymentIntent.id);
      break;
    case 'customer.subscription.deleted':
      const subscription = event.data.object as Stripe.Subscription;
      console.log('Subscription cancelled:', subscription.id);
      break;
    default:
      console.log(`Unhandled event type: ${event.type}`);
  }

  res.json({ received: true });
});
```

---

## 3. HMAC Signature Verification

```typescript
// ════════════════════════════════════════════════════════════════
// HMAC SIGNATURE - COMPLETE IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

import crypto from 'crypto';

// ── SIGNING (Sender Side) ────────────────────────────────────────

interface SignatureOptions {
  algorithm?: 'sha256' | 'sha512';
  version?: string;
  includeTimestamp?: boolean;
}

function createWebhookSignature(
  payload: string | object,
  secret: string,
  options: SignatureOptions = {}
): {
  signature: string;
  timestamp: number;
  headers: Record<string, string>;
} {
  const {
    algorithm = 'sha256',
    version = 'v1',
    includeTimestamp = true
  } = options;

  const payloadString = typeof payload === 'string' 
    ? payload 
    : JSON.stringify(payload);
  
  const timestamp = Math.floor(Date.now() / 1000);
  
  // Include timestamp in signature to prevent replay attacks
  const signatureInput = includeTimestamp
    ? `${timestamp}.${payloadString}`
    : payloadString;

  const signature = crypto
    .createHmac(algorithm, secret)
    .update(signatureInput, 'utf8')
    .digest('hex');

  return {
    signature: `${version}=${signature}`,
    timestamp,
    headers: {
      'X-Webhook-Signature': `${version}=${signature}`,
      'X-Webhook-Timestamp': timestamp.toString(),
      'X-Webhook-Algorithm': algorithm
    }
  };
}

// ── VERIFICATION (Receiver Side) ────────────────────────────────

interface VerificationResult {
  valid: boolean;
  error?: string;
}

function verifyWebhookSignature(
  payload: string | Buffer,
  receivedSignature: string,
  timestamp: string | number,
  secret: string,
  options: {
    algorithm?: 'sha256' | 'sha512';
    maxAge?: number;  // Max age in seconds
  } = {}
): VerificationResult {
  const { algorithm = 'sha256', maxAge = 300 } = options;

  // 1. Validate timestamp (prevent replay attacks)
  const webhookTimestamp = typeof timestamp === 'string' 
    ? parseInt(timestamp, 10) 
    : timestamp;
    
  if (isNaN(webhookTimestamp)) {
    return { valid: false, error: 'Invalid timestamp' };
  }

  const currentTimestamp = Math.floor(Date.now() / 1000);
  const age = currentTimestamp - webhookTimestamp;

  if (age > maxAge) {
    return { valid: false, error: `Webhook too old: ${age}s > ${maxAge}s` };
  }

  if (age < -60) {  // Allow 1 minute clock skew into the future
    return { valid: false, error: 'Webhook timestamp in the future' };
  }

  // 2. Parse signature
  const signatureParts = receivedSignature.split('=');
  if (signatureParts.length !== 2) {
    return { valid: false, error: 'Invalid signature format' };
  }

  const [version, signature] = signatureParts;
  if (version !== 'v1') {
    return { valid: false, error: `Unsupported signature version: ${version}` };
  }

  // 3. Calculate expected signature
  const payloadString = Buffer.isBuffer(payload) 
    ? payload.toString('utf8') 
    : payload;
    
  const signatureInput = `${webhookTimestamp}.${payloadString}`;
  
  const expectedSignature = crypto
    .createHmac(algorithm, secret)
    .update(signatureInput, 'utf8')
    .digest('hex');

  // 4. Timing-safe comparison (CRITICAL!)
  try {
    const signatureBuffer = Buffer.from(signature, 'hex');
    const expectedBuffer = Buffer.from(expectedSignature, 'hex');

    if (signatureBuffer.length !== expectedBuffer.length) {
      return { valid: false, error: 'Signature length mismatch' };
    }

    const isValid = crypto.timingSafeEqual(signatureBuffer, expectedBuffer);
    
    if (!isValid) {
      return { valid: false, error: 'Signature mismatch' };
    }

    return { valid: true };
  } catch {
    return { valid: false, error: 'Signature comparison failed' };
  }
}

// ── WHY TIMING-SAFE COMPARISON? ─────────────────────────────────

/*
Regular string comparison (===) is vulnerable to timing attacks:

function vulnerableCompare(a: string, b: string): boolean {
  if (a.length !== b.length) return false;
  for (let i = 0; i < a.length; i++) {
    if (a[i] !== b[i]) return false;  // Returns early on mismatch!
  }
  return true;
}

An attacker can:
1. Send signature "a000000..." and measure response time
2. Send signature "b000000..." and measure response time
3. The correct first character will take slightly longer
4. Repeat for each character to discover the entire signature

crypto.timingSafeEqual always compares ALL bytes in constant time.
*/

// ── EXPRESS MIDDLEWARE ──────────────────────────────────────────

import { Request, Response, NextFunction } from 'express';

interface WebhookVerificationOptions {
  secret: string;
  signatureHeader?: string;
  timestampHeader?: string;
  maxAge?: number;
}

function webhookVerificationMiddleware(options: WebhookVerificationOptions) {
  const {
    secret,
    signatureHeader = 'x-webhook-signature',
    timestampHeader = 'x-webhook-timestamp',
    maxAge = 300
  } = options;

  return (req: Request, res: Response, next: NextFunction) => {
    const signature = req.headers[signatureHeader] as string;
    const timestamp = req.headers[timestampHeader] as string;

    if (!signature || !timestamp) {
      return res.status(401).json({
        error: 'Missing webhook signature headers'
      });
    }

    // Body must be raw buffer for accurate signature verification
    if (!Buffer.isBuffer(req.body)) {
      return res.status(500).json({
        error: 'Webhook endpoint must use raw body parser'
      });
    }

    const result = verifyWebhookSignature(
      req.body,
      signature,
      timestamp,
      secret,
      { maxAge }
    );

    if (!result.valid) {
      console.warn('Webhook verification failed:', result.error);
      return res.status(401).json({
        error: 'Invalid webhook signature',
        details: result.error
      });
    }

    // Signature valid - parse body and continue
    try {
      req.body = JSON.parse(req.body.toString());
      next();
    } catch {
      return res.status(400).json({ error: 'Invalid JSON payload' });
    }
  };
}

// Usage
app.post(
  '/webhooks/incoming',
  express.raw({ type: 'application/json' }),
  webhookVerificationMiddleware({ secret: process.env.WEBHOOK_SECRET! }),
  async (req, res) => {
    // req.body is now parsed JSON, signature is verified
    const event = req.body;
    // Process event...
    res.json({ received: true });
  }
);
```

---

## 4. Retry Strategies

```typescript
// ════════════════════════════════════════════════════════════════
// WEBHOOK RETRY STRATEGIES
// ════════════════════════════════════════════════════════════════

interface RetryConfig {
  maxAttempts: number;
  backoffSchedule: number[];  // Delays in milliseconds
  retryableStatusCodes: number[];
}

const DEFAULT_RETRY_CONFIG: RetryConfig = {
  maxAttempts: 6,
  backoffSchedule: [
    60_000,       // 1 minute
    300_000,      // 5 minutes
    1_800_000,    // 30 minutes
    7_200_000,    // 2 hours
    28_800_000,   // 8 hours
  ],
  retryableStatusCodes: [408, 429, 500, 502, 503, 504]
};

interface DeliveryAttempt {
  attemptNumber: number;
  timestamp: Date;
  statusCode?: number;
  error?: string;
  duration: number;
}

interface WebhookDelivery {
  id: string;
  subscriptionId: string;
  event: WebhookPayload;
  status: 'pending' | 'delivered' | 'failed' | 'dead';
  attempts: DeliveryAttempt[];
  nextRetryAt?: Date;
  createdAt: Date;
  deliveredAt?: Date;
}

class WebhookDeliveryService {
  private config: RetryConfig;

  constructor(config: Partial<RetryConfig> = {}) {
    this.config = { ...DEFAULT_RETRY_CONFIG, ...config };
  }

  // Calculate next retry time
  getNextRetryDelay(attemptNumber: number): number | null {
    if (attemptNumber >= this.config.maxAttempts) {
      return null;  // No more retries
    }

    const index = Math.min(attemptNumber - 1, this.config.backoffSchedule.length - 1);
    return this.config.backoffSchedule[index];
  }

  // Should we retry this failure?
  shouldRetry(attemptNumber: number, statusCode?: number, error?: string): boolean {
    // Max attempts reached
    if (attemptNumber >= this.config.maxAttempts) {
      return false;
    }

    // Network errors - always retry
    if (!statusCode && error) {
      return true;
    }

    // Check if status code is retryable
    if (statusCode && this.config.retryableStatusCodes.includes(statusCode)) {
      return true;
    }

    // 4xx errors (except 408, 429) are not retryable - client error
    if (statusCode && statusCode >= 400 && statusCode < 500) {
      return false;
    }

    return true;
  }

  async deliver(delivery: WebhookDelivery): Promise<void> {
    const subscription = await getSubscription(delivery.subscriptionId);
    const attemptNumber = delivery.attempts.length + 1;

    const result = await sendWebhook(subscription, delivery.event);

    const attempt: DeliveryAttempt = {
      attemptNumber,
      timestamp: new Date(),
      statusCode: result.statusCode,
      error: result.error,
      duration: result.duration
    };

    delivery.attempts.push(attempt);

    if (result.success) {
      delivery.status = 'delivered';
      delivery.deliveredAt = new Date();
      await this.updateDelivery(delivery);
      return;
    }

    // Check if we should retry
    if (this.shouldRetry(attemptNumber, result.statusCode, result.error)) {
      const retryDelay = this.getNextRetryDelay(attemptNumber);
      
      if (retryDelay !== null) {
        delivery.nextRetryAt = new Date(Date.now() + retryDelay);
        await this.updateDelivery(delivery);
        await this.scheduleRetry(delivery, retryDelay);
        return;
      }
    }

    // No more retries - move to dead letter
    delivery.status = 'dead';
    await this.updateDelivery(delivery);
    await this.moveToDeadLetter(delivery);
  }

  private async scheduleRetry(delivery: WebhookDelivery, delay: number): Promise<void> {
    await webhookQueue.add('deliver', 
      { deliveryId: delivery.id },
      { delay }
    );
  }

  private async moveToDeadLetter(delivery: WebhookDelivery): Promise<void> {
    await deadLetterQueue.add('failed-webhook', delivery);
    
    // Notify admin
    await notifyWebhookFailure(delivery);
  }

  private async updateDelivery(delivery: WebhookDelivery): Promise<void> {
    await prisma.webhookDelivery.update({
      where: { id: delivery.id },
      data: {
        status: delivery.status,
        attempts: delivery.attempts,
        nextRetryAt: delivery.nextRetryAt,
        deliveredAt: delivery.deliveredAt
      }
    });
  }
}

// ════════════════════════════════════════════════════════════════
// CIRCUIT BREAKER FOR FAILING ENDPOINTS
// ════════════════════════════════════════════════════════════════

interface CircuitBreakerState {
  failures: number;
  lastFailure: Date | null;
  state: 'closed' | 'open' | 'half-open';
  nextAttempt: Date | null;
}

class EndpointCircuitBreaker {
  private states: Map<string, CircuitBreakerState> = new Map();
  
  private readonly failureThreshold = 5;
  private readonly recoveryTime = 300_000;  // 5 minutes

  getState(endpointId: string): CircuitBreakerState {
    if (!this.states.has(endpointId)) {
      this.states.set(endpointId, {
        failures: 0,
        lastFailure: null,
        state: 'closed',
        nextAttempt: null
      });
    }
    return this.states.get(endpointId)!;
  }

  canAttempt(endpointId: string): boolean {
    const state = this.getState(endpointId);

    if (state.state === 'closed') {
      return true;
    }

    if (state.state === 'open') {
      // Check if recovery time has passed
      if (state.nextAttempt && new Date() >= state.nextAttempt) {
        state.state = 'half-open';
        return true;
      }
      return false;
    }

    // half-open - allow one attempt
    return true;
  }

  recordSuccess(endpointId: string): void {
    const state = this.getState(endpointId);
    state.failures = 0;
    state.state = 'closed';
    state.nextAttempt = null;
  }

  recordFailure(endpointId: string): void {
    const state = this.getState(endpointId);
    state.failures++;
    state.lastFailure = new Date();

    if (state.state === 'half-open' || state.failures >= this.failureThreshold) {
      state.state = 'open';
      state.nextAttempt = new Date(Date.now() + this.recoveryTime);
      
      console.log(`Circuit breaker OPEN for endpoint ${endpointId}`);
    }
  }
}

const circuitBreaker = new EndpointCircuitBreaker();

// Integration with delivery
async function deliverWithCircuitBreaker(
  delivery: WebhookDelivery
): Promise<void> {
  const endpointId = delivery.subscriptionId;

  if (!circuitBreaker.canAttempt(endpointId)) {
    // Circuit is open - queue for later
    const state = circuitBreaker.getState(endpointId);
    const delay = state.nextAttempt 
      ? state.nextAttempt.getTime() - Date.now() 
      : 60_000;
    
    await webhookQueue.add('deliver', 
      { deliveryId: delivery.id },
      { delay }
    );
    return;
  }

  const result = await sendWebhook(/* ... */);

  if (result.success) {
    circuitBreaker.recordSuccess(endpointId);
  } else {
    circuitBreaker.recordFailure(endpointId);
  }
}
```

---

## 5. Idempotency

```typescript
// ════════════════════════════════════════════════════════════════
// IDEMPOTENCY HANDLING (Receiver Side)
// ════════════════════════════════════════════════════════════════

import { Redis } from 'ioredis';
import { PrismaClient } from '@prisma/client';

const redis = new Redis();
const prisma = new PrismaClient();

interface ProcessedEvent {
  eventId: string;
  processedAt: Date;
  status: 'processing' | 'completed' | 'failed';
  result?: any;
  error?: string;
}

class IdempotencyManager {
  private readonly keyPrefix = 'webhook:processed:';
  private readonly lockPrefix = 'webhook:lock:';
  private readonly ttl = 86400 * 7;  // 7 days

  // Check if event was already processed
  async isProcessed(eventId: string): Promise<boolean> {
    const key = this.keyPrefix + eventId;
    const result = await redis.get(key);
    
    if (result) {
      const data: ProcessedEvent = JSON.parse(result);
      // If still processing, consider it processed (avoid duplicate processing)
      return data.status === 'completed' || data.status === 'processing';
    }
    
    return false;
  }

  // Acquire processing lock
  async acquireLock(eventId: string): Promise<boolean> {
    const lockKey = this.lockPrefix + eventId;
    const acquired = await redis.set(lockKey, '1', 'NX', 'EX', 60);
    return acquired === 'OK';
  }

  // Release processing lock
  async releaseLock(eventId: string): Promise<void> {
    const lockKey = this.lockPrefix + eventId;
    await redis.del(lockKey);
  }

  // Mark event as being processed
  async markProcessing(eventId: string): Promise<void> {
    const key = this.keyPrefix + eventId;
    const data: ProcessedEvent = {
      eventId,
      processedAt: new Date(),
      status: 'processing'
    };
    await redis.setex(key, this.ttl, JSON.stringify(data));
  }

  // Mark event as completed
  async markCompleted(eventId: string, result?: any): Promise<void> {
    const key = this.keyPrefix + eventId;
    const data: ProcessedEvent = {
      eventId,
      processedAt: new Date(),
      status: 'completed',
      result
    };
    await redis.setex(key, this.ttl, JSON.stringify(data));
    
    // Also store in database for long-term record
    await prisma.processedWebhook.upsert({
      where: { eventId },
      update: { status: 'completed', result, updatedAt: new Date() },
      create: { eventId, status: 'completed', result }
    });
  }

  // Mark event as failed
  async markFailed(eventId: string, error: string): Promise<void> {
    const key = this.keyPrefix + eventId;
    const data: ProcessedEvent = {
      eventId,
      processedAt: new Date(),
      status: 'failed',
      error
    };
    await redis.setex(key, this.ttl, JSON.stringify(data));
    
    await prisma.processedWebhook.upsert({
      where: { eventId },
      update: { status: 'failed', error, updatedAt: new Date() },
      create: { eventId, status: 'failed', error }
    });
  }

  // Get processing status
  async getStatus(eventId: string): Promise<ProcessedEvent | null> {
    const key = this.keyPrefix + eventId;
    const result = await redis.get(key);
    return result ? JSON.parse(result) : null;
  }
}

const idempotency = new IdempotencyManager();

// ════════════════════════════════════════════════════════════════
// WEBHOOK PROCESSING WITH IDEMPOTENCY
// ════════════════════════════════════════════════════════════════

async function processWebhook(event: WebhookPayload): Promise<{
  processed: boolean;
  duplicate: boolean;
  error?: string;
}> {
  const eventId = event.id;

  // 1. Check if already processed
  const alreadyProcessed = await idempotency.isProcessed(eventId);
  if (alreadyProcessed) {
    return { processed: true, duplicate: true };
  }

  // 2. Try to acquire lock (prevent concurrent processing)
  const lockAcquired = await idempotency.acquireLock(eventId);
  if (!lockAcquired) {
    // Another worker is processing this event
    return { processed: true, duplicate: true };
  }

  try {
    // 3. Double-check after acquiring lock
    const stillNew = !(await idempotency.isProcessed(eventId));
    if (!stillNew) {
      return { processed: true, duplicate: true };
    }

    // 4. Mark as processing
    await idempotency.markProcessing(eventId);

    // 5. Actually process the event
    const result = await handleWebhookEvent(event);

    // 6. Mark as completed
    await idempotency.markCompleted(eventId, result);

    return { processed: true, duplicate: false };

  } catch (error: any) {
    // Mark as failed
    await idempotency.markFailed(eventId, error.message);
    
    return { processed: false, duplicate: false, error: error.message };

  } finally {
    // Release lock
    await idempotency.releaseLock(eventId);
  }
}

// ════════════════════════════════════════════════════════════════
// IDEMPOTENT OPERATIONS
// ════════════════════════════════════════════════════════════════

// Make your handlers naturally idempotent

// BAD: Not idempotent
async function handlePaymentBad(event: any) {
  // This will credit twice if webhook is delivered twice!
  await creditAccount(event.data.userId, event.data.amount);
}

// GOOD: Idempotent
async function handlePaymentGood(event: any) {
  const { paymentId, userId, amount } = event.data;
  
  // Use payment ID as idempotency key for the operation
  const existingCredit = await prisma.accountCredit.findUnique({
    where: { paymentId }
  });
  
  if (existingCredit) {
    console.log(`Payment ${paymentId} already credited`);
    return existingCredit;
  }
  
  // Credit with payment ID reference
  return prisma.accountCredit.create({
    data: {
      paymentId,
      userId,
      amount,
      creditedAt: new Date()
    }
  });
}

// GOOD: Using database constraints
async function handleOrderCreated(event: any) {
  const { orderId, ...orderData } = event.data;
  
  // Upsert ensures idempotency
  return prisma.order.upsert({
    where: { externalId: orderId },
    update: orderData,
    create: {
      externalId: orderId,
      ...orderData
    }
  });
}

// GOOD: Check-then-act with proper locking
async function handleSubscriptionCancelled(event: any) {
  const { subscriptionId } = event.data;
  
  // Only cancel if currently active
  const result = await prisma.subscription.updateMany({
    where: {
      externalId: subscriptionId,
      status: 'active'  // Only update if active
    },
    data: {
      status: 'cancelled',
      cancelledAt: new Date()
    }
  });
  
  if (result.count === 0) {
    console.log(`Subscription ${subscriptionId} already cancelled or not found`);
  }
}
```

---

## 6. Queue-Based Delivery

```typescript
// ════════════════════════════════════════════════════════════════
// BULLMQ WEBHOOK DELIVERY QUEUE
// ════════════════════════════════════════════════════════════════

import { Queue, Worker, Job } from 'bullmq';
import { Redis } from 'ioredis';

const connection = new Redis({
  host: process.env.REDIS_HOST,
  port: parseInt(process.env.REDIS_PORT || '6379'),
  maxRetriesPerRequest: null
});

// Webhook delivery queue
const webhookQueue = new Queue('webhook-delivery', {
  connection,
  defaultJobOptions: {
    attempts: 6,
    backoff: {
      type: 'custom'
    },
    removeOnComplete: {
      age: 86400,   // Keep completed jobs for 24 hours
      count: 10000  // Keep last 10000 completed
    },
    removeOnFail: false  // Keep failed jobs for investigation
  }
});

// Custom backoff function
function getBackoffDelay(attemptsMade: number): number {
  const delays = [
    60_000,       // 1 minute
    300_000,      // 5 minutes
    1_800_000,    // 30 minutes
    7_200_000,    // 2 hours
    28_800_000,   // 8 hours
  ];
  
  const index = Math.min(attemptsMade - 1, delays.length - 1);
  return delays[index];
}

// Queue event
async function queueWebhookEvent(
  event: WebhookPayload,
  subscriptions: WebhookSubscription[]
): Promise<void> {
  const jobs = subscriptions.map(subscription => ({
    name: 'deliver',
    data: {
      subscriptionId: subscription.id,
      event,
      endpoint: subscription.url
    },
    opts: {
      jobId: `${event.id}-${subscription.id}`,  // Prevents duplicate jobs
      priority: getEventPriority(event.type)
    }
  }));

  await webhookQueue.addBulk(jobs);
}

function getEventPriority(eventType: string): number {
  // Lower number = higher priority
  const priorities: Record<string, number> = {
    'payment.succeeded': 1,
    'payment.failed': 1,
    'order.created': 2,
    'customer.created': 3
  };
  return priorities[eventType] ?? 5;
}

// ════════════════════════════════════════════════════════════════
// WEBHOOK DELIVERY WORKER
// ════════════════════════════════════════════════════════════════

const deliveryWorker = new Worker(
  'webhook-delivery',
  async (job: Job) => {
    const { subscriptionId, event, endpoint } = job.data;
    
    console.log(`Delivering webhook ${event.id} to ${endpoint} (attempt ${job.attemptsMade + 1})`);

    // Get subscription with secret
    const subscription = await getSubscription(subscriptionId);
    if (!subscription || !subscription.active) {
      console.log(`Subscription ${subscriptionId} not found or inactive`);
      return { skipped: true };
    }

    // Check circuit breaker
    if (!circuitBreaker.canAttempt(subscriptionId)) {
      throw new Error('Circuit breaker open');
    }

    // Send webhook
    const result = await sendWebhook(subscription, event);

    // Record metrics
    await recordDeliveryMetrics(subscriptionId, result);

    if (result.success) {
      circuitBreaker.recordSuccess(subscriptionId);
      
      // Store successful delivery
      await recordDelivery(event.id, subscriptionId, 'delivered', job.attemptsMade + 1);
      
      return { 
        delivered: true, 
        statusCode: result.statusCode,
        duration: result.duration 
      };
    }

    // Failed
    circuitBreaker.recordFailure(subscriptionId);
    
    // Record attempt
    await recordDeliveryAttempt(event.id, subscriptionId, result);

    // Throw to trigger retry
    throw new Error(result.error || `HTTP ${result.statusCode}`);
  },
  {
    connection,
    concurrency: 50,  // Process 50 webhooks concurrently
    limiter: {
      max: 100,       // Max 100 jobs
      duration: 1000  // Per second
    }
  }
);

// Custom backoff strategy
deliveryWorker.on('failed', async (job, err) => {
  if (job) {
    const delay = getBackoffDelay(job.attemptsMade);
    console.log(`Webhook delivery failed, will retry in ${delay}ms`);
  }
});

// Handle job completion
deliveryWorker.on('completed', (job, result) => {
  console.log(`Webhook delivered: ${job.id}`, result);
});

// Handle permanent failure
deliveryWorker.on('failed', async (job, err) => {
  if (job && job.attemptsMade >= job.opts.attempts!) {
    console.error(`Webhook permanently failed: ${job.id}`);
    
    // Move to dead letter queue
    await deadLetterQueue.add('failed-webhook', {
      jobId: job.id,
      data: job.data,
      error: err.message,
      attempts: job.attemptsMade,
      failedAt: new Date()
    });

    // Record permanent failure
    await recordDelivery(
      job.data.event.id, 
      job.data.subscriptionId, 
      'failed', 
      job.attemptsMade
    );
  }
});

// ════════════════════════════════════════════════════════════════
// DEAD LETTER QUEUE
// ════════════════════════════════════════════════════════════════

const deadLetterQueue = new Queue('webhook-dead-letter', { connection });

// Process dead letter queue (manual review/retry)
const deadLetterWorker = new Worker(
  'webhook-dead-letter',
  async (job: Job) => {
    // This is typically processed manually or by admin action
    // Just store for investigation
    await prisma.webhookDeadLetter.create({
      data: {
        originalJobId: job.data.jobId,
        subscriptionId: job.data.data.subscriptionId,
        eventId: job.data.data.event.id,
        eventType: job.data.data.event.type,
        payload: job.data.data.event,
        error: job.data.error,
        attempts: job.data.attempts,
        failedAt: job.data.failedAt
      }
    });

    // Notify admin
    await sendAdminAlert({
      type: 'webhook_permanent_failure',
      subscriptionId: job.data.data.subscriptionId,
      eventId: job.data.data.event.id,
      error: job.data.error
    });
  },
  { connection }
);

// Manual retry from dead letter
async function retryFromDeadLetter(deadLetterId: string): Promise<void> {
  const deadLetter = await prisma.webhookDeadLetter.findUnique({
    where: { id: deadLetterId }
  });

  if (!deadLetter) {
    throw new Error('Dead letter not found');
  }

  // Re-queue for delivery
  await webhookQueue.add('deliver', {
    subscriptionId: deadLetter.subscriptionId,
    event: deadLetter.payload,
    endpoint: (await getSubscription(deadLetter.subscriptionId))?.url
  }, {
    jobId: `retry-${deadLetter.originalJobId}-${Date.now()}`
  });

  // Mark dead letter as retried
  await prisma.webhookDeadLetter.update({
    where: { id: deadLetterId },
    data: { retriedAt: new Date() }
  });
}
```

---

## 7. Webhook Management API

```typescript
// ════════════════════════════════════════════════════════════════
// WEBHOOK SUBSCRIPTION MANAGEMENT API
// ════════════════════════════════════════════════════════════════

import express from 'express';
import crypto from 'crypto';
import { z } from 'zod';

const router = express.Router();

// Validation schemas
const createSubscriptionSchema = z.object({
  url: z.string().url(),
  events: z.array(z.string()).min(1),
  description: z.string().optional()
});

// Generate signing secret
function generateSecret(): string {
  return `whsec_${crypto.randomBytes(24).toString('base64url')}`;
}

// Create subscription
router.post('/webhooks/endpoints', async (req, res) => {
  const validation = createSubscriptionSchema.safeParse(req.body);
  if (!validation.success) {
    return res.status(400).json({ error: validation.error });
  }

  const { url, events, description } = validation.data;

  // Validate URL is reachable
  const isReachable = await validateEndpoint(url);
  if (!isReachable) {
    return res.status(400).json({ 
      error: 'Endpoint URL is not reachable' 
    });
  }

  // Create subscription
  const secret = generateSecret();
  
  const subscription = await prisma.webhookSubscription.create({
    data: {
      userId: req.user.id,
      url,
      events,
      secret,
      description,
      active: true
    }
  });

  res.status(201).json({
    id: subscription.id,
    url: subscription.url,
    events: subscription.events,
    secret,  // Only shown once!
    active: subscription.active,
    createdAt: subscription.createdAt
  });
});

// List subscriptions
router.get('/webhooks/endpoints', async (req, res) => {
  const subscriptions = await prisma.webhookSubscription.findMany({
    where: { userId: req.user.id },
    select: {
      id: true,
      url: true,
      events: true,
      active: true,
      description: true,
      createdAt: true,
      // Don't expose secret
    }
  });

  res.json({ data: subscriptions });
});

// Get subscription details
router.get('/webhooks/endpoints/:id', async (req, res) => {
  const subscription = await prisma.webhookSubscription.findUnique({
    where: { id: req.params.id, userId: req.user.id }
  });

  if (!subscription) {
    return res.status(404).json({ error: 'Subscription not found' });
  }

  res.json({
    id: subscription.id,
    url: subscription.url,
    events: subscription.events,
    active: subscription.active,
    description: subscription.description,
    createdAt: subscription.createdAt
  });
});

// Update subscription
router.patch('/webhooks/endpoints/:id', async (req, res) => {
  const updateSchema = z.object({
    url: z.string().url().optional(),
    events: z.array(z.string()).min(1).optional(),
    active: z.boolean().optional(),
    description: z.string().optional()
  });

  const validation = updateSchema.safeParse(req.body);
  if (!validation.success) {
    return res.status(400).json({ error: validation.error });
  }

  const updated = await prisma.webhookSubscription.update({
    where: { id: req.params.id, userId: req.user.id },
    data: validation.data
  });

  res.json({
    id: updated.id,
    url: updated.url,
    events: updated.events,
    active: updated.active
  });
});

// Delete subscription
router.delete('/webhooks/endpoints/:id', async (req, res) => {
  await prisma.webhookSubscription.delete({
    where: { id: req.params.id, userId: req.user.id }
  });

  res.status(204).send();
});

// Rotate secret
router.post('/webhooks/endpoints/:id/rotate-secret', async (req, res) => {
  const newSecret = generateSecret();

  await prisma.webhookSubscription.update({
    where: { id: req.params.id, userId: req.user.id },
    data: { secret: newSecret }
  });

  res.json({ secret: newSecret });
});

// ════════════════════════════════════════════════════════════════
// WEBHOOK DELIVERY LOGS
// ════════════════════════════════════════════════════════════════

// Get delivery attempts for an event
router.get('/webhooks/events/:eventId/deliveries', async (req, res) => {
  const deliveries = await prisma.webhookDelivery.findMany({
    where: { 
      eventId: req.params.eventId,
      subscription: { userId: req.user.id }
    },
    include: {
      attempts: true
    }
  });

  res.json({ data: deliveries });
});

// Get recent deliveries for a subscription
router.get('/webhooks/endpoints/:id/deliveries', async (req, res) => {
  const { limit = 50, status } = req.query;

  const deliveries = await prisma.webhookDelivery.findMany({
    where: {
      subscriptionId: req.params.id,
      subscription: { userId: req.user.id },
      ...(status && { status: status as string })
    },
    orderBy: { createdAt: 'desc' },
    take: parseInt(limit as string),
    include: {
      attempts: {
        orderBy: { timestamp: 'desc' },
        take: 1
      }
    }
  });

  res.json({ data: deliveries });
});

// Retry a failed delivery
router.post('/webhooks/deliveries/:id/retry', async (req, res) => {
  const delivery = await prisma.webhookDelivery.findUnique({
    where: { id: req.params.id },
    include: { subscription: true }
  });

  if (!delivery || delivery.subscription.userId !== req.user.id) {
    return res.status(404).json({ error: 'Delivery not found' });
  }

  if (delivery.status === 'delivered') {
    return res.status(400).json({ error: 'Already delivered' });
  }

  // Queue for immediate retry
  await webhookQueue.add('deliver', {
    subscriptionId: delivery.subscriptionId,
    event: delivery.event
  }, {
    jobId: `manual-retry-${delivery.id}-${Date.now()}`
  });

  res.json({ message: 'Retry queued' });
});

// ════════════════════════════════════════════════════════════════
// TEST ENDPOINT
// ════════════════════════════════════════════════════════════════

// Send test event
router.post('/webhooks/endpoints/:id/test', async (req, res) => {
  const subscription = await prisma.webhookSubscription.findUnique({
    where: { id: req.params.id, userId: req.user.id }
  });

  if (!subscription) {
    return res.status(404).json({ error: 'Subscription not found' });
  }

  const testEvent: WebhookPayload = {
    id: `test_${Date.now()}`,
    type: 'test.event',
    timestamp: new Date().toISOString(),
    data: {
      message: 'This is a test webhook event',
      timestamp: new Date().toISOString()
    }
  };

  // Send synchronously for immediate feedback
  const result = await sendWebhook(subscription, testEvent);

  res.json({
    success: result.success,
    statusCode: result.statusCode,
    error: result.error,
    duration: result.duration
  });
});

export default router;
```

---

## 8. Common Pitfalls

```typescript
// ════════════════════════════════════════════════════════════════
// WEBHOOK PITFALLS
// ════════════════════════════════════════════════════════════════

// ❌ PITFALL 1: Not using raw body for signature verification
// Problem: Parsed JSON may serialize differently

// Bad (Express)
app.use(express.json());  // This parses body before signature check!
app.post('/webhook', (req, res) => {
  const signature = req.headers['x-signature'];
  const isValid = verify(JSON.stringify(req.body), signature);  // May fail!
});

// Good
app.post('/webhook', 
  express.raw({ type: 'application/json' }),  // Keep as buffer
  (req, res) => {
    const signature = req.headers['x-signature'];
    const isValid = verify(req.body, signature);  // Original bytes
    req.body = JSON.parse(req.body.toString());  // Parse after verification
  }
);

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 2: Using regular string comparison for signatures
// Problem: Timing attack vulnerability

// Bad
function verifySignature(received: string, expected: string): boolean {
  return received === expected;  // Timing attack possible!
}

// Good
function verifySignature(received: string, expected: string): boolean {
  try {
    return crypto.timingSafeEqual(
      Buffer.from(received, 'hex'),
      Buffer.from(expected, 'hex')
    );
  } catch {
    return false;
  }
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 3: Processing webhook synchronously before responding
// Problem: Timeout, duplicate deliveries

// Bad
app.post('/webhook', async (req, res) => {
  // Long processing - sender may timeout and retry
  await processOrder(req.body);
  await sendConfirmationEmail(req.body);
  await updateInventory(req.body);
  res.json({ ok: true });  // 30 seconds later...
});

// Good
app.post('/webhook', async (req, res) => {
  // Respond immediately
  res.json({ received: true });
  
  // Process asynchronously
  processWebhookAsync(req.body).catch(console.error);
});

// Better - use a queue
app.post('/webhook', async (req, res) => {
  await webhookProcessingQueue.add('process', req.body);
  res.json({ received: true });
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 4: Not handling idempotency
// Problem: Duplicate processing

// Bad
app.post('/webhook', async (req, res) => {
  const { paymentId, amount } = req.body;
  await creditUserAccount(amount);  // Will credit multiple times!
  res.json({ ok: true });
});

// Good
app.post('/webhook', async (req, res) => {
  const { eventId, paymentId, amount } = req.body;
  
  // Check if already processed
  const existing = await db.processedEvents.findUnique({
    where: { eventId }
  });
  
  if (existing) {
    return res.json({ ok: true, duplicate: true });
  }
  
  await db.processedEvents.create({ data: { eventId, processedAt: new Date() }});
  await creditUserAccount(paymentId, amount);
  res.json({ ok: true });
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 5: Not including timestamp in signature
// Problem: Replay attacks

// Bad - signature without timestamp
const signature = hmac(payload, secret);

// Good - include timestamp
const timestamp = Math.floor(Date.now() / 1000);
const signature = hmac(`${timestamp}.${payload}`, secret);
// Header: X-Signature: v1=<sig>, X-Timestamp: <timestamp>

// Receiver verifies timestamp is recent (within 5 minutes)

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 6: Retrying on 4xx errors
// Problem: Wasted retries, never succeed

// Bad
if (response.status !== 200) {
  throw new Error('Retry');  // Will retry on 400, 401, 404...
}

// Good
if (response.status >= 200 && response.status < 300) {
  return { success: true };
}

// Only retry 5xx and specific 4xx
const retryable = [408, 429, 500, 502, 503, 504];
if (retryable.includes(response.status)) {
  throw new Error('Retry');  // Will be retried
}

// 400, 401, 403, 404 etc. - don't retry
return { success: false, permanent: true };

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 7: Not validating webhook URL on registration
// Problem: SSRF attacks

// Bad
app.post('/register-webhook', async (req, res) => {
  const { url } = req.body;
  await saveWebhookUrl(url);  // Could be internal URL!
});

// Good
app.post('/register-webhook', async (req, res) => {
  const { url } = req.body;
  
  // Validate URL
  const parsed = new URL(url);
  
  // Block internal/private IPs
  const blockedPatterns = [
    /^localhost$/i,
    /^127\./,
    /^10\./,
    /^172\.(1[6-9]|2[0-9]|3[0-1])\./,
    /^192\.168\./,
    /^169\.254\./,
    /^::1$/,
    /^fc00:/i,
    /^fe80:/i
  ];
  
  if (blockedPatterns.some(p => p.test(parsed.hostname))) {
    return res.status(400).json({ error: 'Invalid webhook URL' });
  }
  
  // Require HTTPS in production
  if (process.env.NODE_ENV === 'production' && parsed.protocol !== 'https:') {
    return res.status(400).json({ error: 'HTTPS required' });
  }
  
  await saveWebhookUrl(url);
});
```

---

## 9. Interview Questions

### Basic Questions

**Q: "What are webhooks?"**
> "HTTP callbacks that deliver event notifications in real-time. Instead of polling an API, you register an endpoint URL and receive POST requests when events occur. Common for payment processors (Stripe), source control (GitHub), and any system that needs to notify external services."

**Q: "How do you secure webhooks?"**
> "1) HMAC signature - sign payload with shared secret, receiver verifies. 2) Timestamp in signature - prevents replay attacks. 3) HTTPS only. 4) IP whitelisting if possible. 5) Timing-safe comparison for signature verification. 6) Validate URLs on registration (prevent SSRF)."

**Q: "What's the difference between webhooks and APIs?"**
> "APIs are pull-based - you request data when needed. Webhooks are push-based - you're notified when events happen. Webhooks are more efficient for real-time updates (no polling), but require you to expose an endpoint and handle retries/failures."

### Intermediate Questions

**Q: "How do you handle failed webhook deliveries?"**
> "Implement retry with exponential backoff (1m, 5m, 30m, 2h, 8h). After max attempts, move to dead letter queue for investigation. Track delivery status. Provide webhook logs in dashboard. Allow manual retry. Use circuit breaker for consistently failing endpoints."

**Q: "What is idempotency in webhooks?"**
> "Due to retries and network issues, receivers may get the same event multiple times. Include unique event ID (idempotency key). Receivers should store processed IDs and skip duplicates. Make handlers naturally idempotent (upserts, conditional updates)."

**Q: "How do you verify webhook signatures?"**
> "Sender: HMAC-SHA256(timestamp.payload, secret), sends as header. Receiver: 1) Check timestamp is recent (<5 min). 2) Compute expected signature. 3) Use timing-safe comparison. Must use raw body bytes, not parsed JSON."

### Advanced Questions

**Q: "How would you design a webhook system at scale?"**
> "Queue-based architecture: events go to message queue (Redis/Kafka), workers process and deliver. Per-endpoint rate limiting. Circuit breakers for failing endpoints. Separate queues by priority. Dead letter queue with admin dashboard. Metrics: delivery rate, latency, failure rate. Allow replay of past events. Horizontal scaling of workers."

**Q: "How do you handle event ordering?"**
> "Webhooks may arrive out of order. Options: 1) Include sequence number or timestamp, receiver reorders. 2) Include 'updated_at', receiver ignores older updates. 3) Accept out-of-order, fetch latest state via API after receiving webhook. 4) For strict ordering, use single partition in Kafka. Usually at-least-once + idempotency is sufficient."

**Q: "How do major platforms implement webhooks (Stripe, GitHub)?"**
> "Stripe: HMAC-SHA256 with timestamp in signature (`stripe-signature` header), 72-hour retry window, webhook endpoint verification, test mode events. GitHub: HMAC-SHA256 (`X-Hub-Signature-256`), delivery GUIDs, redeliver in UI, webhook ping on creation. Both: event logs, retry on 5xx only, immediate 4xx failure."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                    WEBHOOK SENDER CHECKLIST                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SIGNATURE:                                                     │
│  □ Sign with HMAC-SHA256                                       │
│  □ Include timestamp in signature                              │
│  □ Use unique event ID                                         │
│                                                                 │
│  DELIVERY:                                                      │
│  □ Queue-based (don't send inline)                             │
│  □ Exponential backoff retries                                 │
│  □ Circuit breaker for failing endpoints                       │
│  □ Dead letter queue                                           │
│                                                                 │
│  MONITORING:                                                    │
│  □ Track delivery success rate                                 │
│  □ Webhook logs/dashboard                                      │
│  □ Alert on endpoint degradation                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   WEBHOOK RECEIVER CHECKLIST                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  VERIFICATION:                                                  │
│  □ Use raw body (not parsed JSON)                              │
│  □ Verify signature with timing-safe comparison                │
│  □ Check timestamp is recent                                   │
│                                                                 │
│  PROCESSING:                                                    │
│  □ Respond 200 immediately                                     │
│  □ Process asynchronously                                      │
│  □ Handle idempotency (check event ID)                         │
│  □ Make handlers naturally idempotent                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

RETRY SCHEDULE (typical):
┌────────────────────────────────────────────────────────────────┐
│ Attempt 1: Immediate                                           │
│ Attempt 2: 1 minute                                            │
│ Attempt 3: 5 minutes                                           │
│ Attempt 4: 30 minutes                                          │
│ Attempt 5: 2 hours                                             │
│ Attempt 6: 8 hours                                             │
│ → Dead letter queue                                            │
└────────────────────────────────────────────────────────────────┘

SUCCESS/RETRY STATUS CODES:
┌────────────────────────────────────────────────────────────────┐
│ 2xx → Success (delivered)                                      │
│ 408, 429 → Retry (timeout, rate limited)                       │
│ 5xx → Retry (server error)                                     │
│ 400, 401, 403, 404 → Don't retry (client error)                │
└────────────────────────────────────────────────────────────────┘

SIGNATURE FORMAT:
┌────────────────────────────────────────────────────────────────┐
│ Header: X-Webhook-Signature: v1=<hex_signature>                │
│ Header: X-Webhook-Timestamp: <unix_timestamp>                  │
│ Signature input: {timestamp}.{payload}                         │
│ Algorithm: HMAC-SHA256(signature_input, secret)                │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

