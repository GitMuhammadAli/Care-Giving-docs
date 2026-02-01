# 🔔 Push Notifications - Complete Guide

> A comprehensive guide to push notifications - Web Push API, Firebase Cloud Messaging (FCM), Apple Push Notification service (APNs), VAPID, service workers, and notification strategies.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Push notifications deliver messages to users even when the app isn't active, using platform-specific services (FCM for Android/Web, APNs for iOS) with service workers handling delivery in browsers, requiring user permission and careful strategy to avoid notification fatigue."

### The 7 Key Concepts (Remember These!)
```
1. WEB PUSH API     → Browser standard for push (Service Worker + Push API)
2. VAPID            → Voluntary Application Server Identification (auth keys)
3. FCM              → Firebase Cloud Messaging (Google's push service)
4. APNS             → Apple Push Notification service
5. SERVICE WORKER   → Background script that receives and displays notifications
6. SUBSCRIPTION     → User's push endpoint (unique URL per device)
7. PAYLOAD          → Notification content (encrypted in transit)
```

### Push Notification Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                 PUSH NOTIFICATION FLOW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  YOUR SERVER                                                    │
│      │                                                         │
│      │ 1. Send notification request                            │
│      │    (with subscription endpoint)                         │
│      ▼                                                         │
│  ┌──────────────────────────────────────┐                     │
│  │         PUSH SERVICE                  │                     │
│  │  (FCM, APNs, Mozilla Push, etc.)     │                     │
│  └──────────────────────────────────────┘                     │
│      │                                                         │
│      │ 2. Routes to correct device                            │
│      │                                                         │
│      ▼                                                         │
│  ┌──────────────────────────────────────┐                     │
│  │         USER'S DEVICE                 │                     │
│  │  ┌────────────────────────────────┐  │                     │
│  │  │     Service Worker             │  │                     │
│  │  │     (receives 'push' event)    │  │                     │
│  │  └────────────────────────────────┘  │                     │
│  │              │                        │                     │
│  │              ▼                        │                     │
│  │  ┌────────────────────────────────┐  │                     │
│  │  │     Notification Display       │  │                     │
│  │  │     (OS notification center)   │  │                     │
│  │  └────────────────────────────────┘  │                     │
│  └──────────────────────────────────────┘                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Web Push vs Mobile Push
```
┌─────────────────────────────────────────────────────────────────┐
│                  WEB PUSH vs MOBILE PUSH                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  WEB PUSH (Browser)                                            │
│  ─────────────────                                             │
│  • Uses Web Push Protocol (RFC 8030)                           │
│  • Requires Service Worker                                     │
│  • VAPID for server authentication                             │
│  • Works via browser's push service                            │
│  • Limited payload (~4KB)                                       │
│  • Browser must be installed (not necessarily open)            │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  MOBILE PUSH                                                   │
│  ───────────                                                    │
│  FCM (Android/iOS):                                            │
│  • Firebase SDK integration                                    │
│  • Token-based authentication                                  │
│  • Data + Notification messages                                │
│  • Topics for group messaging                                  │
│                                                                 │
│  APNs (iOS only):                                              │
│  • Certificate or token-based auth                             │
│  • Required for iOS notifications                              │
│  • Rich notification support                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"VAPID keys"** | "We use VAPID keys for Web Push server authentication" |
| **"Subscription endpoint"** | "The subscription endpoint is the unique URL for this device's push service" |
| **"Service worker push event"** | "The service worker listens for push events even when the app is closed" |
| **"FCM data messages"** | "We use data messages instead of notification messages for custom handling" |
| **"Notification channels"** | "Android 8+ requires notification channels for user preference control" |
| **"Silent push"** | "We use silent push to trigger background data sync without alerting the user" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Web Push payload limit | **~4KB** | Encrypted content limit |
| FCM payload limit | **4KB** | Maximum message size |
| APNs payload limit | **4KB** | iOS notification limit |
| Permission prompt timing | **After user engagement** | Don't ask immediately |
| Notification retention (FCM) | **4 weeks** | Message time-to-live |
| iOS badge number max | **99+** | Display limitation |

### The "Wow" Statement (Memorize This!)
> "We implemented a multi-platform push notification system: for web, we use the Web Push API with VAPID authentication - the service worker subscribes to push, and we store the endpoint+keys in our database. For mobile, we use FCM which handles both Android and iOS (via APNs relay). We implemented notification channels on Android 8+ for user preference control. The key insight is using data messages instead of notification messages so our app handles display logic, enabling custom actions, grouping, and conditional suppression. We also implement exponential backoff for retries when push services return 429s, and we clean up stale tokens when we get 404s or 410s (subscription expired)."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   ┌────────────────────────────────────────────────────────┐   │
│   │                    YOUR BACKEND                        │   │
│   │  ┌─────────────────┐  ┌─────────────────────────────┐ │   │
│   │  │ Push Token DB   │  │ Notification Sender Service │ │   │
│   │  │ (endpoints,     │  │ • Batch sending             │ │   │
│   │  │  device tokens) │  │ • Retry logic               │ │   │
│   │  └─────────────────┘  │ • Token cleanup             │ │   │
│   │                       └─────────────────────────────────┘   │
│   └──────────────────┬─────────────────────────────────┬───┘   │
│                      │                                 │       │
│                      ▼                                 ▼       │
│   ┌──────────────────────────┐   ┌──────────────────────────┐ │
│   │       WEB PUSH           │   │      FCM / APNs          │ │
│   │  (Browser Push Service)  │   │   (Mobile Push)          │ │
│   └────────────┬─────────────┘   └──────────┬───────────────┘ │
│                │                             │                │
│                ▼                             ▼                │
│   ┌──────────────────────────┐   ┌──────────────────────────┐ │
│   │   Service Worker         │   │    Mobile App            │ │
│   │   (background receive)   │   │    (FCM SDK)             │ │
│   └──────────────────────────┘   └──────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is Web Push?"**
> "A browser API for sending notifications to users even when the site isn't open. Uses Service Workers to receive messages, VAPID for server auth, and browser-specific push services for delivery."

**Q: "What is VAPID?"**
> "Voluntary Application Server Identification. Public/private key pair that authenticates your server to push services. The public key is shared with the browser subscription, private key signs push requests."

**Q: "FCM data vs notification messages?"**
> "Notification messages are handled by FCM SDK - shown automatically. Data messages are passed to your app code, giving you control over display logic, grouping, and conditional showing. Always prefer data messages for flexibility."

**Q: "How do you handle failed push delivery?"**
> "Check the response: 404/410 means the token is invalid - remove from database. 429 means rate limited - exponential backoff. 500s from push service - retry with backoff. Track delivery receipts where possible."

**Q: "Why ask for notification permission after engagement?"**
> "Users deny permission 70-90% of the time on immediate prompts. Wait until they've engaged meaningfully, explain the value proposition, use a soft-ask UI first. This dramatically improves opt-in rates."

**Q: "What about iOS Safari push?"**
> "Safari on iOS 16.4+ finally supports Web Push, but requires the PWA to be added to home screen. Uses same Web Push API but with Apple-specific quirks in subscription handling."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How do push notifications work?"

**Junior Answer:**
> "You send a notification from the server and it appears on the user's device."

**Senior Answer:**
> "Push notifications involve multiple players:

1. **Subscription**: User grants permission, browser creates a subscription (endpoint URL + encryption keys) via the push service (Google, Mozilla, Apple's servers).

2. **Storage**: You store the subscription on your server, associated with the user.

3. **Sending**: To send, you make an HTTP request to the subscription endpoint, signed with your VAPID private key, with encrypted payload.

4. **Delivery**: Push service routes to the device. The service worker receives a `push` event and displays the notification.

For mobile, FCM/APNs handle the routing. The key differences are token management (device tokens vs subscription objects) and payload handling (data vs notification messages).

The critical considerations are permission UX (timing matters), token lifecycle management (cleanup invalid tokens), and notification strategy (batching, importance levels, silent updates)."

### When Asked: "How do you handle notification permissions?"

**Junior Answer:**
> "Just call `Notification.requestPermission()` when the page loads."

**Senior Answer:**
> "Never prompt immediately - users deny reflexively. Instead:

1. **Engagement trigger**: Wait until user shows intent (e.g., enables alerts on a product, saves an article).

2. **Soft ask**: Show custom UI explaining value ('Get notified when your order ships'). If they say yes, then request real permission.

3. **Explain value**: Make it clear what they'll get and how often.

4. **Handle denial gracefully**: Store in localStorage that they denied, don't keep asking. Offer alternative (email).

5. **Consider timing**: Don't interrupt critical flows. Ask after positive moments (successful purchase, not during checkout).

This approach can improve opt-in rates from ~5% to ~30%+."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "How do you batch notifications?" | "Group by topic/thread, use collapse keys so only latest in group shows, implement notification grouping on Android/iOS." |
| "What about notification fatigue?" | "Implement frequency caps, importance levels, user preference center, time-of-day awareness. Let users control granularly." |
| "How do you debug push issues?" | "Check subscription validity, verify VAPID keys, use push service debugging tools, implement delivery tracking, log push service responses." |
| "Cross-platform challenges?" | "Different payload formats, iOS needs APNs relay (via FCM or direct), Safari has quirks, permission UX varies. Use abstraction layer." |

---

## 📚 Table of Contents

1. [Web Push API](#1-web-push-api)
2. [Service Worker Setup](#2-service-worker-setup)
3. [VAPID and Server-Side](#3-vapid-and-server-side)
4. [Firebase Cloud Messaging (FCM)](#4-firebase-cloud-messaging-fcm)
5. [Apple Push Notification Service](#5-apple-push-notification-service)
6. [Permission UX](#6-permission-ux)
7. [Notification Strategy](#7-notification-strategy)
8. [Common Pitfalls](#8-common-pitfalls)
9. [Interview Questions](#9-interview-questions)

---

## 1. Web Push API

```typescript
// ════════════════════════════════════════════════════════════════
// WEB PUSH SUBSCRIPTION (CLIENT-SIDE)
// ════════════════════════════════════════════════════════════════

// Check support and permission
async function checkPushSupport(): Promise<{
  supported: boolean;
  permission: NotificationPermission;
}> {
  const supported = 'serviceWorker' in navigator && 
                   'PushManager' in window &&
                   'Notification' in window;
  
  return {
    supported,
    permission: supported ? Notification.permission : 'denied'
  };
}

// Subscribe to push notifications
async function subscribeToPush(
  vapidPublicKey: string
): Promise<PushSubscription | null> {
  try {
    // Register service worker
    const registration = await navigator.serviceWorker.register('/sw.js');
    await navigator.serviceWorker.ready;

    // Request permission
    const permission = await Notification.requestPermission();
    if (permission !== 'granted') {
      console.log('Notification permission denied');
      return null;
    }

    // Subscribe to push
    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,  // Required: must show notification
      applicationServerKey: urlBase64ToUint8Array(vapidPublicKey)
    });

    // Send subscription to your server
    await sendSubscriptionToServer(subscription);

    return subscription;
  } catch (error) {
    console.error('Push subscription failed:', error);
    return null;
  }
}

// Convert VAPID key
function urlBase64ToUint8Array(base64String: string): Uint8Array {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/-/g, '+')
    .replace(/_/g, '/');

  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);

  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }
  return outputArray;
}

// Send subscription to server
async function sendSubscriptionToServer(
  subscription: PushSubscription
): Promise<void> {
  await fetch('/api/push/subscribe', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      subscription: subscription.toJSON(),
      // Include any additional user context
      userId: getCurrentUserId(),
      userAgent: navigator.userAgent
    })
  });
}

// Unsubscribe
async function unsubscribeFromPush(): Promise<boolean> {
  const registration = await navigator.serviceWorker.ready;
  const subscription = await registration.pushManager.getSubscription();
  
  if (subscription) {
    // Notify server
    await fetch('/api/push/unsubscribe', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ endpoint: subscription.endpoint })
    });

    // Unsubscribe locally
    await subscription.unsubscribe();
    return true;
  }
  
  return false;
}

// Get existing subscription
async function getExistingSubscription(): Promise<PushSubscription | null> {
  const registration = await navigator.serviceWorker.ready;
  return registration.pushManager.getSubscription();
}
```

---

## 2. Service Worker Setup

```typescript
// ════════════════════════════════════════════════════════════════
// SERVICE WORKER (sw.js)
// ════════════════════════════════════════════════════════════════

// Push event - fired when notification received
self.addEventListener('push', (event: PushEvent) => {
  console.log('Push received:', event);

  let data: {
    title: string;
    body: string;
    icon?: string;
    badge?: string;
    image?: string;
    tag?: string;
    data?: any;
    actions?: NotificationAction[];
    requireInteraction?: boolean;
    silent?: boolean;
  };

  try {
    data = event.data?.json();
  } catch {
    data = {
      title: 'New Notification',
      body: event.data?.text() || 'You have a new notification'
    };
  }

  const options: NotificationOptions = {
    body: data.body,
    icon: data.icon || '/icons/notification-icon.png',
    badge: data.badge || '/icons/badge-icon.png',
    image: data.image,
    tag: data.tag,  // Replaces existing notification with same tag
    data: data.data,
    actions: data.actions,
    requireInteraction: data.requireInteraction || false,
    silent: data.silent || false,
    vibrate: [200, 100, 200]
  };

  // Must call showNotification to satisfy browser requirements
  event.waitUntil(
    self.registration.showNotification(data.title, options)
  );
});

// Notification click handler
self.addEventListener('notificationclick', (event: NotificationEvent) => {
  console.log('Notification clicked:', event);
  
  event.notification.close();

  const data = event.notification.data || {};
  const action = event.action; // Which action button was clicked

  let targetUrl = '/';

  if (action === 'view') {
    targetUrl = data.url || '/';
  } else if (action === 'dismiss') {
    return; // Just close
  } else {
    // Default click (not on action button)
    targetUrl = data.url || '/';
  }

  // Focus existing window or open new
  event.waitUntil(
    clients.matchAll({ type: 'window', includeUncontrolled: true })
      .then((windowClients) => {
        // Check if there's already a window open
        for (const client of windowClients) {
          if (client.url === targetUrl && 'focus' in client) {
            return client.focus();
          }
        }
        // Open new window
        if (clients.openWindow) {
          return clients.openWindow(targetUrl);
        }
      })
  );
});

// Notification close handler (user dismissed)
self.addEventListener('notificationclose', (event: NotificationEvent) => {
  console.log('Notification dismissed:', event.notification.tag);
  
  // Track dismissal for analytics
  const data = event.notification.data;
  if (data?.trackingId) {
    fetch('/api/analytics/notification-dismissed', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ trackingId: data.trackingId })
    });
  }
});

// Push subscription change (token refresh)
self.addEventListener('pushsubscriptionchange', (event: any) => {
  console.log('Push subscription changed');
  
  event.waitUntil(
    // Resubscribe and update server
    self.registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: event.oldSubscription?.options?.applicationServerKey
    })
    .then((subscription) => {
      return fetch('/api/push/subscription-changed', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          oldEndpoint: event.oldSubscription?.endpoint,
          newSubscription: subscription.toJSON()
        })
      });
    })
  );
});

// ════════════════════════════════════════════════════════════════
// ADVANCED SERVICE WORKER FEATURES
// ════════════════════════════════════════════════════════════════

// Group/stack notifications
async function showGroupedNotification(data: any) {
  // Get existing notifications
  const notifications = await self.registration.getNotifications({
    tag: data.groupTag
  });

  if (notifications.length > 0) {
    // Create summary notification
    const titles = notifications.map(n => n.title);
    titles.push(data.title);
    
    // Close existing
    notifications.forEach(n => n.close());
    
    // Show grouped
    await self.registration.showNotification(
      `${titles.length} new messages`,
      {
        body: titles.join(', '),
        tag: data.groupTag,
        data: { isGroup: true, items: titles }
      }
    );
  } else {
    await self.registration.showNotification(data.title, {
      body: data.body,
      tag: data.groupTag
    });
  }
}

// Background sync trigger from push
self.addEventListener('push', (event: PushEvent) => {
  const data = event.data?.json();
  
  if (data?.type === 'sync-trigger') {
    // Silent push to trigger background sync
    event.waitUntil(
      self.registration.sync.register('background-sync')
    );
    return;
  }
  
  // Normal notification
  event.waitUntil(
    self.registration.showNotification(data.title, { body: data.body })
  );
});
```

---

## 3. VAPID and Server-Side

```typescript
// ════════════════════════════════════════════════════════════════
// SERVER-SIDE WEB PUSH (Node.js)
// ════════════════════════════════════════════════════════════════

import webpush from 'web-push';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// Generate VAPID keys (do this once, store securely)
// const vapidKeys = webpush.generateVAPIDKeys();
// console.log(vapidKeys);

// Configure web-push
webpush.setVapidDetails(
  'mailto:admin@example.com',
  process.env.VAPID_PUBLIC_KEY!,
  process.env.VAPID_PRIVATE_KEY!
);

// Store subscription
interface PushSubscriptionData {
  endpoint: string;
  keys: {
    p256dh: string;
    auth: string;
  };
}

async function saveSubscription(
  userId: string,
  subscription: PushSubscriptionData
): Promise<void> {
  await prisma.pushSubscription.upsert({
    where: { endpoint: subscription.endpoint },
    update: {
      keys: subscription.keys,
      userId,
      updatedAt: new Date()
    },
    create: {
      endpoint: subscription.endpoint,
      keys: subscription.keys,
      userId
    }
  });
}

// Send notification
interface NotificationPayload {
  title: string;
  body: string;
  icon?: string;
  url?: string;
  tag?: string;
  data?: any;
  actions?: Array<{
    action: string;
    title: string;
    icon?: string;
  }>;
}

async function sendPushNotification(
  userId: string,
  payload: NotificationPayload
): Promise<{ success: number; failed: number }> {
  // Get all subscriptions for user
  const subscriptions = await prisma.pushSubscription.findMany({
    where: { userId }
  });

  let success = 0;
  let failed = 0;

  for (const sub of subscriptions) {
    try {
      await webpush.sendNotification(
        {
          endpoint: sub.endpoint,
          keys: sub.keys as any
        },
        JSON.stringify(payload),
        {
          TTL: 86400,  // Time to live: 24 hours
          urgency: 'normal',  // 'very-low' | 'low' | 'normal' | 'high'
          topic: payload.tag  // For collapsible notifications
        }
      );
      success++;
    } catch (error: any) {
      console.error('Push failed:', error.statusCode, error.body);
      
      // Handle specific errors
      if (error.statusCode === 404 || error.statusCode === 410) {
        // Subscription expired/invalid - remove from database
        await prisma.pushSubscription.delete({
          where: { endpoint: sub.endpoint }
        });
      }
      
      failed++;
    }
  }

  return { success, failed };
}

// Bulk send with batching
async function sendBulkNotification(
  userIds: string[],
  payload: NotificationPayload,
  batchSize = 100
): Promise<{ success: number; failed: number }> {
  let totalSuccess = 0;
  let totalFailed = 0;

  // Process in batches
  for (let i = 0; i < userIds.length; i += batchSize) {
    const batch = userIds.slice(i, i + batchSize);
    
    const results = await Promise.allSettled(
      batch.map(userId => sendPushNotification(userId, payload))
    );

    for (const result of results) {
      if (result.status === 'fulfilled') {
        totalSuccess += result.value.success;
        totalFailed += result.value.failed;
      } else {
        totalFailed++;
      }
    }

    // Rate limiting delay between batches
    if (i + batchSize < userIds.length) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }
  }

  return { success: totalSuccess, failed: totalFailed };
}

// ════════════════════════════════════════════════════════════════
// API ROUTES
// ════════════════════════════════════════════════════════════════

// Express.js example
import express from 'express';
const router = express.Router();

// Get VAPID public key
router.get('/vapid-public-key', (req, res) => {
  res.json({ publicKey: process.env.VAPID_PUBLIC_KEY });
});

// Subscribe
router.post('/subscribe', async (req, res) => {
  const { subscription, userId } = req.body;
  
  try {
    await saveSubscription(userId, subscription);
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: 'Failed to save subscription' });
  }
});

// Unsubscribe
router.post('/unsubscribe', async (req, res) => {
  const { endpoint } = req.body;
  
  try {
    await prisma.pushSubscription.delete({
      where: { endpoint }
    });
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: 'Failed to remove subscription' });
  }
});

// Send notification (admin endpoint)
router.post('/send', async (req, res) => {
  const { userId, userIds, notification } = req.body;
  
  try {
    let result;
    if (userIds) {
      result = await sendBulkNotification(userIds, notification);
    } else {
      result = await sendPushNotification(userId, notification);
    }
    res.json(result);
  } catch (error) {
    res.status(500).json({ error: 'Failed to send notification' });
  }
});

export default router;
```

---

## 4. Firebase Cloud Messaging (FCM)

```typescript
// ════════════════════════════════════════════════════════════════
// FCM SERVER-SIDE (Node.js)
// ════════════════════════════════════════════════════════════════

import admin from 'firebase-admin';

// Initialize Firebase Admin
admin.initializeApp({
  credential: admin.credential.cert({
    projectId: process.env.FIREBASE_PROJECT_ID,
    privateKey: process.env.FIREBASE_PRIVATE_KEY?.replace(/\\n/g, '\n'),
    clientEmail: process.env.FIREBASE_CLIENT_EMAIL
  })
});

const messaging = admin.messaging();

// Send to single device
async function sendToDevice(
  token: string,
  notification: {
    title: string;
    body: string;
    imageUrl?: string;
  },
  data?: Record<string, string>
): Promise<string> {
  const message: admin.messaging.Message = {
    token,
    notification: {
      title: notification.title,
      body: notification.body,
      imageUrl: notification.imageUrl
    },
    data,
    android: {
      priority: 'high',
      notification: {
        channelId: 'default',
        priority: 'high',
        defaultSound: true,
        defaultVibrateTimings: true
      }
    },
    apns: {
      payload: {
        aps: {
          alert: {
            title: notification.title,
            body: notification.body
          },
          sound: 'default',
          badge: 1
        }
      }
    },
    webpush: {
      notification: {
        icon: '/icons/notification-icon.png',
        badge: '/icons/badge-icon.png'
      },
      fcmOptions: {
        link: data?.url || '/'
      }
    }
  };

  try {
    const response = await messaging.send(message);
    return response;
  } catch (error: any) {
    console.error('FCM send failed:', error.code, error.message);
    
    if (error.code === 'messaging/registration-token-not-registered') {
      // Token invalid - remove from database
      await removeInvalidToken(token);
    }
    
    throw error;
  }
}

// Send to multiple devices
async function sendToMultipleDevices(
  tokens: string[],
  notification: { title: string; body: string },
  data?: Record<string, string>
): Promise<admin.messaging.BatchResponse> {
  const message: admin.messaging.MulticastMessage = {
    tokens,
    notification,
    data,
    android: {
      priority: 'high'
    },
    apns: {
      payload: {
        aps: {
          sound: 'default'
        }
      }
    }
  };

  const response = await messaging.sendEachForMulticast(message);
  
  // Handle failures
  if (response.failureCount > 0) {
    response.responses.forEach((resp, idx) => {
      if (!resp.success) {
        console.error(`Failed to send to token ${tokens[idx]}:`, resp.error);
        
        if (resp.error?.code === 'messaging/registration-token-not-registered') {
          removeInvalidToken(tokens[idx]);
        }
      }
    });
  }

  return response;
}

// Send to topic
async function sendToTopic(
  topic: string,
  notification: { title: string; body: string },
  data?: Record<string, string>
): Promise<string> {
  const message: admin.messaging.Message = {
    topic,
    notification,
    data
  };

  return messaging.send(message);
}

// Subscribe/unsubscribe tokens to topic
async function subscribeToTopic(
  tokens: string[],
  topic: string
): Promise<admin.messaging.MessagingTopicManagementResponse> {
  return messaging.subscribeToTopic(tokens, topic);
}

async function unsubscribeFromTopic(
  tokens: string[],
  topic: string
): Promise<admin.messaging.MessagingTopicManagementResponse> {
  return messaging.unsubscribeFromTopic(tokens, topic);
}

// ════════════════════════════════════════════════════════════════
// DATA MESSAGES vs NOTIFICATION MESSAGES
// ════════════════════════════════════════════════════════════════

// Notification message - handled by FCM SDK (shows automatically)
async function sendNotificationMessage(token: string) {
  return messaging.send({
    token,
    notification: {  // This makes it a notification message
      title: 'Hello',
      body: 'World'
    }
  });
}

// Data message - handled by your app (full control)
async function sendDataMessage(token: string) {
  return messaging.send({
    token,
    data: {  // Only data, no notification
      type: 'NEW_MESSAGE',
      messageId: '123',
      senderId: 'user456',
      senderName: 'John',
      preview: 'Hey, how are you?'
    },
    android: {
      priority: 'high'
    },
    apns: {
      payload: {
        aps: {
          'content-available': 1  // Silent notification for iOS
        }
      }
    }
  });
}

// ════════════════════════════════════════════════════════════════
// FCM CLIENT-SIDE (React/React Native)
// ════════════════════════════════════════════════════════════════

// Web (Firebase SDK)
import { initializeApp } from 'firebase/app';
import { getMessaging, getToken, onMessage } from 'firebase/messaging';

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID
};

const app = initializeApp(firebaseConfig);
const messaging = getMessaging(app);

// Get FCM token
async function getFCMToken(): Promise<string | null> {
  try {
    const permission = await Notification.requestPermission();
    if (permission !== 'granted') return null;

    const token = await getToken(messaging, {
      vapidKey: process.env.NEXT_PUBLIC_FIREBASE_VAPID_KEY
    });

    // Save token to your server
    await saveTokenToServer(token);

    return token;
  } catch (error) {
    console.error('Failed to get FCM token:', error);
    return null;
  }
}

// Handle foreground messages
onMessage(messaging, (payload) => {
  console.log('Foreground message:', payload);
  
  // Show custom UI notification
  if (payload.notification) {
    showInAppNotification({
      title: payload.notification.title!,
      body: payload.notification.body!,
      onClick: () => {
        // Handle click
      }
    });
  }
});

// ════════════════════════════════════════════════════════════════
// FCM SERVICE WORKER (firebase-messaging-sw.js)
// ════════════════════════════════════════════════════════════════

// firebase-messaging-sw.js
importScripts('https://www.gstatic.com/firebasejs/9.0.0/firebase-app-compat.js');
importScripts('https://www.gstatic.com/firebasejs/9.0.0/firebase-messaging-compat.js');

firebase.initializeApp({
  apiKey: '...',
  projectId: '...',
  messagingSenderId: '...',
  appId: '...'
});

const messaging = firebase.messaging();

// Background message handler
messaging.onBackgroundMessage((payload) => {
  console.log('Background message:', payload);

  // For data-only messages, show notification manually
  if (payload.data && !payload.notification) {
    const notificationTitle = payload.data.title || 'New notification';
    const notificationOptions = {
      body: payload.data.body,
      icon: '/icons/notification-icon.png',
      data: payload.data
    };

    self.registration.showNotification(notificationTitle, notificationOptions);
  }
});
```

---

## 5. Apple Push Notification Service

```typescript
// ════════════════════════════════════════════════════════════════
// APNS SERVER-SIDE (Node.js) - Token-based Auth
// ════════════════════════════════════════════════════════════════

import apn from '@parse/node-apn';
import jwt from 'jsonwebtoken';
import http2 from 'http2';

// Option 1: Using node-apn library
const apnProvider = new apn.Provider({
  token: {
    key: process.env.APNS_KEY_PATH!,  // .p8 file path
    keyId: process.env.APNS_KEY_ID!,
    teamId: process.env.APNS_TEAM_ID!
  },
  production: process.env.NODE_ENV === 'production'
});

async function sendAPNSNotification(
  deviceToken: string,
  notification: {
    title: string;
    body: string;
    badge?: number;
    sound?: string;
    data?: any;
  }
): Promise<void> {
  const note = new apn.Notification();
  
  note.alert = {
    title: notification.title,
    body: notification.body
  };
  note.badge = notification.badge;
  note.sound = notification.sound || 'default';
  note.topic = process.env.APNS_BUNDLE_ID!;  // Your app bundle ID
  note.payload = notification.data || {};
  note.expiry = Math.floor(Date.now() / 1000) + 86400;  // 24 hours
  note.priority = 10;  // 10 = immediate, 5 = power-efficient

  try {
    const result = await apnProvider.send(note, deviceToken);
    
    if (result.failed.length > 0) {
      const failure = result.failed[0];
      console.error('APNS failed:', failure.response);
      
      if (failure.response?.reason === 'BadDeviceToken' ||
          failure.response?.reason === 'Unregistered') {
        // Remove invalid token
        await removeInvalidToken(deviceToken);
      }
    }
  } catch (error) {
    console.error('APNS send error:', error);
    throw error;
  }
}

// Option 2: Direct HTTP/2 implementation
class APNSClient {
  private jwtToken: string | null = null;
  private jwtExpiry: number = 0;

  private generateJWT(): string {
    const now = Math.floor(Date.now() / 1000);
    
    // Reuse token if not expired (tokens last 1 hour)
    if (this.jwtToken && this.jwtExpiry > now + 300) {
      return this.jwtToken;
    }

    const privateKey = fs.readFileSync(process.env.APNS_KEY_PATH!);
    
    this.jwtToken = jwt.sign({}, privateKey, {
      algorithm: 'ES256',
      issuer: process.env.APNS_TEAM_ID!,
      header: {
        alg: 'ES256',
        kid: process.env.APNS_KEY_ID!
      },
      expiresIn: '1h'
    });
    
    this.jwtExpiry = now + 3600;
    return this.jwtToken;
  }

  async send(deviceToken: string, payload: any): Promise<void> {
    const host = process.env.NODE_ENV === 'production'
      ? 'api.push.apple.com'
      : 'api.sandbox.push.apple.com';

    const client = http2.connect(`https://${host}`);

    const headers = {
      ':method': 'POST',
      ':path': `/3/device/${deviceToken}`,
      ':scheme': 'https',
      'authorization': `bearer ${this.generateJWT()}`,
      'apns-topic': process.env.APNS_BUNDLE_ID!,
      'apns-push-type': 'alert',
      'apns-priority': '10',
      'apns-expiration': String(Math.floor(Date.now() / 1000) + 86400)
    };

    return new Promise((resolve, reject) => {
      const req = client.request(headers);
      
      req.setEncoding('utf8');
      
      let data = '';
      req.on('data', chunk => data += chunk);
      
      req.on('end', () => {
        client.close();
        if (data) {
          const response = JSON.parse(data);
          if (response.reason) {
            reject(new Error(response.reason));
          } else {
            resolve();
          }
        } else {
          resolve();
        }
      });

      req.on('error', reject);
      req.write(JSON.stringify(payload));
      req.end();
    });
  }
}

// ════════════════════════════════════════════════════════════════
// RICH NOTIFICATIONS (iOS)
// ════════════════════════════════════════════════════════════════

async function sendRichNotification(
  deviceToken: string,
  notification: {
    title: string;
    body: string;
    imageUrl?: string;
    category?: string;  // For custom actions
  }
): Promise<void> {
  const note = new apn.Notification();
  
  note.alert = {
    title: notification.title,
    body: notification.body
  };
  
  note.topic = process.env.APNS_BUNDLE_ID!;
  note.mutableContent = true;  // Required for notification service extension
  note.category = notification.category;
  
  note.payload = {
    'media-url': notification.imageUrl
  };

  await apnProvider.send(note, deviceToken);
}

// Silent notification (background update)
async function sendSilentNotification(
  deviceToken: string,
  data: any
): Promise<void> {
  const note = new apn.Notification();
  
  note.topic = process.env.APNS_BUNDLE_ID!;
  note.contentAvailable = true;  // Silent notification
  note.priority = 5;  // Must be 5 for silent
  note.payload = data;

  await apnProvider.send(note, deviceToken);
}
```

---

## 6. Permission UX

```typescript
// ════════════════════════════════════════════════════════════════
// PERMISSION UX BEST PRACTICES
// ════════════════════════════════════════════════════════════════

// React component for notification permission
import React, { useState, useEffect } from 'react';

interface PermissionPromptProps {
  onGranted: () => void;
  onDenied: () => void;
}

function NotificationPermissionPrompt({ onGranted, onDenied }: PermissionPromptProps) {
  const [showSoftAsk, setShowSoftAsk] = useState(false);
  const [hasDecided, setHasDecided] = useState(false);

  useEffect(() => {
    // Check if already decided
    const decided = localStorage.getItem('notification-permission-decided');
    if (decided) {
      setHasDecided(true);
      return;
    }

    // Check current permission
    if ('Notification' in window) {
      if (Notification.permission === 'granted') {
        onGranted();
        setHasDecided(true);
      } else if (Notification.permission === 'denied') {
        setHasDecided(true);
      }
    }
  }, []);

  const handleSoftAskYes = async () => {
    setShowSoftAsk(false);
    
    // Now request real permission
    const permission = await Notification.requestPermission();
    localStorage.setItem('notification-permission-decided', 'true');
    setHasDecided(true);

    if (permission === 'granted') {
      onGranted();
    } else {
      onDenied();
    }
  };

  const handleSoftAskNo = () => {
    setShowSoftAsk(false);
    localStorage.setItem('notification-permission-decided', 'true');
    setHasDecided(true);
    onDenied();
  };

  // Don't show if already decided
  if (hasDecided) return null;

  // Soft ask UI
  if (showSoftAsk) {
    return (
      <div className="notification-prompt-overlay">
        <div className="notification-prompt-card">
          <div className="notification-icon">🔔</div>
          <h3>Stay in the loop!</h3>
          <p>
            Get notified when your order ships, when there are price drops 
            on items you're watching, and important account updates.
          </p>
          <div className="notification-prompt-actions">
            <button 
              className="btn-secondary"
              onClick={handleSoftAskNo}
            >
              Not now
            </button>
            <button 
              className="btn-primary"
              onClick={handleSoftAskYes}
            >
              Enable notifications
            </button>
          </div>
          <p className="notification-prompt-note">
            You can change this anytime in settings
          </p>
        </div>
      </div>
    );
  }

  return null;
}

// Trigger soft ask at appropriate times
function useNotificationTrigger() {
  const triggerNotificationPrompt = () => {
    // Check if already asked
    if (localStorage.getItem('notification-permission-decided')) {
      return false;
    }

    // Check if supported
    if (!('Notification' in window)) {
      return false;
    }

    // Check if already granted/denied
    if (Notification.permission !== 'default') {
      return false;
    }

    return true;
  };

  return { triggerNotificationPrompt };
}

// Good trigger points:
// 1. After user completes purchase → "Get notified when it ships"
// 2. When user saves an item → "Notify when price drops"
// 3. After user creates account → "Welcome! Enable notifications"
// 4. When user engages with alerts feature → "Enable for real alerts"

// ════════════════════════════════════════════════════════════════
// NOTIFICATION PREFERENCES CENTER
// ════════════════════════════════════════════════════════════════

interface NotificationPreferences {
  orderUpdates: boolean;
  priceAlerts: boolean;
  marketing: boolean;
  security: boolean;
}

function NotificationPreferencesCenter() {
  const [prefs, setPrefs] = useState<NotificationPreferences>({
    orderUpdates: true,
    priceAlerts: true,
    marketing: false,
    security: true
  });

  useEffect(() => {
    // Load from server
    loadPreferences().then(setPrefs);
  }, []);

  const updatePreference = async (key: keyof NotificationPreferences, value: boolean) => {
    const newPrefs = { ...prefs, [key]: value };
    setPrefs(newPrefs);
    
    // Save to server
    await savePreferences(newPrefs);
    
    // Update FCM topics
    if (value) {
      await subscribeToNotificationTopic(key);
    } else {
      await unsubscribeFromNotificationTopic(key);
    }
  };

  return (
    <div className="preferences-center">
      <h2>Notification Settings</h2>
      
      <div className="preference-item">
        <div>
          <h4>Order Updates</h4>
          <p>Shipping, delivery, and order status</p>
        </div>
        <Toggle 
          checked={prefs.orderUpdates}
          onChange={(v) => updatePreference('orderUpdates', v)}
        />
      </div>

      <div className="preference-item">
        <div>
          <h4>Price Alerts</h4>
          <p>Price drops on watched items</p>
        </div>
        <Toggle 
          checked={prefs.priceAlerts}
          onChange={(v) => updatePreference('priceAlerts', v)}
        />
      </div>

      <div className="preference-item">
        <div>
          <h4>Marketing</h4>
          <p>Sales, promotions, and recommendations</p>
        </div>
        <Toggle 
          checked={prefs.marketing}
          onChange={(v) => updatePreference('marketing', v)}
        />
      </div>

      <div className="preference-item">
        <div>
          <h4>Security</h4>
          <p>Login alerts and security notifications</p>
        </div>
        <Toggle 
          checked={prefs.security}
          onChange={(v) => updatePreference('security', v)}
          disabled  // Security notifications always on
        />
      </div>
    </div>
  );
}
```

---

## 7. Notification Strategy

```typescript
// ════════════════════════════════════════════════════════════════
// NOTIFICATION STRATEGY AND BEST PRACTICES
// ════════════════════════════════════════════════════════════════

interface NotificationContext {
  userId: string;
  timezone: string;
  preferences: NotificationPreferences;
  recentNotifications: Date[];
}

class NotificationService {
  // Frequency capping
  private async shouldSend(
    context: NotificationContext,
    type: string,
    importance: 'high' | 'normal' | 'low'
  ): Promise<boolean> {
    // High importance always sends (security, etc.)
    if (importance === 'high') return true;

    // Check user preferences
    if (!this.isTypeEnabled(context.preferences, type)) {
      return false;
    }

    // Frequency cap: max 5 notifications per hour for normal
    const oneHourAgo = new Date(Date.now() - 3600000);
    const recentCount = context.recentNotifications.filter(
      d => d > oneHourAgo
    ).length;

    if (importance === 'normal' && recentCount >= 5) {
      return false;
    }

    // Low importance: max 3 per day
    if (importance === 'low') {
      const oneDayAgo = new Date(Date.now() - 86400000);
      const dailyCount = context.recentNotifications.filter(
        d => d > oneDayAgo
      ).length;
      if (dailyCount >= 3) return false;
    }

    return true;
  }

  // Time-aware sending
  private isGoodTimeToSend(timezone: string): boolean {
    const userTime = new Date().toLocaleString('en-US', { 
      timeZone: timezone,
      hour: 'numeric',
      hour12: false
    });
    const hour = parseInt(userTime);

    // Don't send between 10 PM and 8 AM user's local time
    return hour >= 8 && hour < 22;
  }

  // Schedule for optimal time
  async scheduleNotification(
    userId: string,
    notification: NotificationPayload,
    options: {
      importance: 'high' | 'normal' | 'low';
      scheduledFor?: Date;
      respectQuietHours?: boolean;
    }
  ): Promise<void> {
    const context = await this.getUserContext(userId);

    // High importance sends immediately regardless
    if (options.importance === 'high') {
      await this.sendImmediately(userId, notification);
      return;
    }

    // Check frequency cap
    if (!await this.shouldSend(context, notification.tag || 'general', options.importance)) {
      console.log('Notification suppressed due to frequency cap');
      return;
    }

    // Check quiet hours
    if (options.respectQuietHours !== false && !this.isGoodTimeToSend(context.timezone)) {
      // Schedule for 9 AM user's local time
      const scheduledTime = this.getNextGoodTime(context.timezone);
      await this.scheduleForLater(userId, notification, scheduledTime);
      return;
    }

    await this.sendImmediately(userId, notification);
  }

  // Notification grouping/batching
  async sendBatchedNotifications(
    userId: string,
    notifications: NotificationPayload[]
  ): Promise<void> {
    if (notifications.length === 0) return;

    if (notifications.length === 1) {
      await this.sendImmediately(userId, notifications[0]);
      return;
    }

    // Group by type/tag
    const groups = new Map<string, NotificationPayload[]>();
    for (const n of notifications) {
      const key = n.tag || 'default';
      if (!groups.has(key)) groups.set(key, []);
      groups.get(key)!.push(n);
    }

    // Send summary for each group
    for (const [tag, group] of groups) {
      if (group.length === 1) {
        await this.sendImmediately(userId, group[0]);
      } else {
        // Create summary notification
        await this.sendImmediately(userId, {
          title: `${group.length} new ${tag}`,
          body: group.map(n => n.title).join(', '),
          tag: `${tag}-summary`,
          data: {
            type: 'summary',
            items: group
          }
        });
      }
    }
  }

  // Collapse key for replacing notifications
  async sendCollapsibleNotification(
    userId: string,
    notification: NotificationPayload,
    collapseKey: string
  ): Promise<void> {
    // When sending to FCM/APNS, use collapse key
    // This replaces any existing notification with same key
    await this.send(userId, {
      ...notification,
      tag: collapseKey,  // Web Push
      // FCM uses collapseKey or apns-collapse-id
    });
  }

  private async getUserContext(userId: string): Promise<NotificationContext> {
    // Fetch from database
    return {
      userId,
      timezone: 'America/New_York',
      preferences: await this.getPreferences(userId),
      recentNotifications: await this.getRecentNotifications(userId)
    };
  }

  private isTypeEnabled(prefs: NotificationPreferences, type: string): boolean {
    // Map notification types to preference keys
    const mapping: Record<string, keyof NotificationPreferences> = {
      'order': 'orderUpdates',
      'price': 'priceAlerts',
      'promo': 'marketing',
      'security': 'security'
    };
    const key = mapping[type] || 'marketing';
    return prefs[key];
  }

  private getNextGoodTime(timezone: string): Date {
    // Calculate next 9 AM in user's timezone
    const now = new Date();
    const userNow = new Date(now.toLocaleString('en-US', { timeZone: timezone }));
    
    const next9AM = new Date(userNow);
    next9AM.setHours(9, 0, 0, 0);
    
    if (userNow.getHours() >= 9) {
      next9AM.setDate(next9AM.getDate() + 1);
    }
    
    return next9AM;
  }
}

// ════════════════════════════════════════════════════════════════
// A/B TESTING NOTIFICATIONS
// ════════════════════════════════════════════════════════════════

interface NotificationVariant {
  id: string;
  title: string;
  body: string;
  weight: number;  // Percentage (0-100)
}

async function sendABTestNotification(
  userId: string,
  testId: string,
  variants: NotificationVariant[]
): Promise<void> {
  // Deterministic variant selection based on userId
  const hash = simpleHash(userId + testId);
  const bucket = hash % 100;
  
  let cumulative = 0;
  let selectedVariant = variants[0];
  
  for (const variant of variants) {
    cumulative += variant.weight;
    if (bucket < cumulative) {
      selectedVariant = variant;
      break;
    }
  }

  // Track which variant was sent
  await trackABTestAssignment(userId, testId, selectedVariant.id);

  // Send notification
  await sendNotification(userId, {
    title: selectedVariant.title,
    body: selectedVariant.body,
    data: {
      abTestId: testId,
      variantId: selectedVariant.id
    }
  });
}

function simpleHash(str: string): number {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    hash = ((hash << 5) - hash) + str.charCodeAt(i);
    hash = hash & hash;
  }
  return Math.abs(hash);
}
```

---

## 8. Common Pitfalls

```typescript
// ════════════════════════════════════════════════════════════════
// PUSH NOTIFICATION PITFALLS
// ════════════════════════════════════════════════════════════════

// ❌ PITFALL 1: Asking for permission immediately
// Problem: 70-90% denial rate

// Bad
window.onload = () => {
  Notification.requestPermission();  // Immediate popup
};

// Good
// Wait for meaningful user action
document.getElementById('enable-alerts').onclick = () => {
  showSoftAskModal();  // Custom UI first
};

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 2: Not handling invalid tokens
// Problem: Wasted API calls, quota issues

// Bad
async function sendNotification(token: string, payload: any) {
  await fcm.send({ token, ...payload });  // Just send, ignore errors
}

// Good
async function sendNotification(token: string, payload: any) {
  try {
    await fcm.send({ token, ...payload });
  } catch (error: any) {
    if (error.code === 'messaging/registration-token-not-registered' ||
        error.code === 'messaging/invalid-registration-token') {
      // Remove invalid token from database
      await removeToken(token);
    } else if (error.code === 'messaging/message-rate-exceeded') {
      // Rate limited - queue for later
      await queueForRetry(token, payload);
    }
    throw error;
  }
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 3: Service worker not showing notification
// Problem: Browser may terminate service worker

// Bad
self.addEventListener('push', (event) => {
  const data = event.data?.json();
  self.registration.showNotification(data.title, { body: data.body });
  // No waitUntil - service worker may stop before notification shows
});

// Good
self.addEventListener('push', (event) => {
  const data = event.data?.json();
  event.waitUntil(  // Keep service worker alive
    self.registration.showNotification(data.title, { body: data.body })
  );
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 4: Not handling subscription change
// Problem: Push stops working after browser update

// Bad
// No handler for subscription change

// Good
self.addEventListener('pushsubscriptionchange', (event) => {
  event.waitUntil(
    self.registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: VAPID_PUBLIC_KEY
    })
    .then(subscription => {
      // Send new subscription to server
      return fetch('/api/push/refresh-subscription', {
        method: 'POST',
        body: JSON.stringify({
          old: event.oldSubscription?.endpoint,
          new: subscription.toJSON()
        })
      });
    })
  );
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 5: Sending too many notifications
// Problem: User turns off all notifications

// Bad
async function onNewMessage(message: Message) {
  await sendPushNotification(message.recipientId, {
    title: message.senderName,
    body: message.text
  });  // Every single message
}

// Good
const notificationQueue = new Map<string, Message[]>();
const BATCH_DELAY = 5000;

async function onNewMessage(message: Message) {
  const userId = message.recipientId;
  
  if (!notificationQueue.has(userId)) {
    notificationQueue.set(userId, []);
    
    // Batch messages for 5 seconds
    setTimeout(async () => {
      const messages = notificationQueue.get(userId)!;
      notificationQueue.delete(userId);
      
      if (messages.length === 1) {
        await sendPushNotification(userId, {
          title: messages[0].senderName,
          body: messages[0].text
        });
      } else {
        await sendPushNotification(userId, {
          title: `${messages.length} new messages`,
          body: `From ${[...new Set(messages.map(m => m.senderName))].join(', ')}`
        });
      }
    }, BATCH_DELAY);
  }
  
  notificationQueue.get(userId)!.push(message);
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 6: Not encrypting payload for Web Push
// Problem: Using web-push library incorrectly

// Note: web-push library handles encryption automatically
// But if implementing manually, you MUST encrypt the payload

// The Web Push protocol requires the payload to be encrypted
// using the p256dh and auth keys from the subscription

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 7: iOS Safari Web Push quirks
// Problem: Different behavior from other browsers

// iOS Safari (16.4+) Web Push requirements:
// 1. Must be a PWA added to home screen
// 2. Must have a manifest.json with display: standalone
// 3. VAPID key must be properly formatted
// 4. Notifications must be triggered by user gesture

async function requestPermissionIOS() {
  // On iOS, must be in response to user gesture
  if ('standalone' in navigator && (navigator as any).standalone) {
    // Running as PWA
    const permission = await Notification.requestPermission();
    return permission;
  } else {
    // Prompt user to add to home screen first
    showAddToHomeScreenPrompt();
    return 'denied';
  }
}
```

---

## 9. Interview Questions

### Basic Questions

**Q: "How do push notifications work?"**
> "User subscribes (grants permission, gets unique push endpoint). Your server stores the endpoint. To send, you POST to that endpoint with encrypted payload. Push service (FCM, Mozilla, Apple) routes to the device. Service worker receives `push` event and displays notification."

**Q: "What is VAPID?"**
> "Voluntary Application Server Identification. A public/private key pair that authenticates your server to push services. Public key is shared during subscription, private key signs requests. Prevents unauthorized servers from sending to your subscribers."

**Q: "Web Push vs FCM?"**
> "Web Push is the browser standard (works in all modern browsers). FCM is Google's service that implements Web Push for Chrome and also handles Android/iOS mobile push. FCM adds features like topics, analytics, and a unified API for all platforms."

### Intermediate Questions

**Q: "How do you handle notification permissions?"**
> "Never ask immediately - high denial rate. Wait for user engagement, show soft-ask UI first explaining value, then request real permission. Store decision in localStorage to avoid re-prompting. Offer granular preferences (order updates vs marketing). Handle denial gracefully with alternatives."

**Q: "FCM data vs notification messages?"**
> "Notification messages: FCM SDK displays automatically. Simple but limited control. Data messages: passed to your app code, full control over display logic. Always prefer data messages for: custom UI, conditional display, notification grouping, click handling. On iOS, data-only needs `content-available` for background delivery."

**Q: "How do you manage push tokens?"**
> "Store with user association. Handle token refresh (FCM onTokenRefresh, pushsubscriptionchange). Remove invalid tokens when you get 404/410 responses. For multi-device support, store array of tokens per user. Implement TTL or last-active tracking to clean up stale tokens."

### Advanced Questions

**Q: "How would you design a notification system at scale?"**
> "1. Token storage: Sharded database, index by user ID. 2. Sending: Queue-based with workers (Redis + Bull). Batch sends (FCM supports 500/request). Rate limiting per user and globally. 3. Reliability: Retry with exponential backoff, dead letter queue for persistent failures. 4. Monitoring: Track delivery rates, open rates, error rates by platform. 5. Preferences: User-level preferences, topic subscriptions, quiet hours. 6. Multi-platform: Abstraction layer over FCM/APNs/Web Push."

**Q: "How do you prevent notification fatigue?"**
> "Frequency capping (max N per hour/day). Importance levels (high always sends, low has strict limits). Grouping/batching related notifications. User preference center with granular controls. Time-of-day awareness (respect quiet hours). Collapse keys for updates (only latest shows). Track engagement - reduce frequency for users who don't interact."

**Q: "iOS Safari Web Push limitations?"**
> "Only works when PWA is added to home screen. Requires display: standalone in manifest. No silent push (content-available). Limited background time. Must request permission in response to user gesture. Different subscription format quirks. Generally more restrictive than other browsers."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  PUSH NOTIFICATION CHECKLIST                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PERMISSION UX:                                                 │
│  □ Never ask immediately                                       │
│  □ Use soft-ask UI first                                       │
│  □ Explain value proposition                                   │
│  □ Store decision to avoid re-prompting                        │
│                                                                 │
│  WEB PUSH:                                                      │
│  □ Generate and store VAPID keys                               │
│  □ Register service worker                                     │
│  □ Handle push event with waitUntil                            │
│  □ Handle pushsubscriptionchange                               │
│  □ Handle notificationclick for navigation                     │
│                                                                 │
│  FCM:                                                          │
│  □ Initialize Firebase Admin SDK                               │
│  □ Use data messages for control                               │
│  □ Handle token refresh                                        │
│  □ Configure notification channels (Android 8+)                │
│                                                                 │
│  TOKEN MANAGEMENT:                                              │
│  □ Remove invalid tokens (404/410)                             │
│  □ Retry on rate limits (429)                                  │
│  □ Handle token refresh                                        │
│  □ Clean up stale tokens periodically                          │
│                                                                 │
│  STRATEGY:                                                      │
│  □ Implement frequency capping                                 │
│  □ Group/batch related notifications                           │
│  □ Respect quiet hours                                         │
│  □ Provide preference center                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

PUSH SERVICES:
┌────────────────────────────────────────────────────────────────┐
│ Web Push   - Browser standard (Chrome, Firefox, Safari*)       │
│ FCM        - Firebase Cloud Messaging (Android, iOS, Web)      │
│ APNs       - Apple Push Notification service (iOS, macOS)      │
│ * Safari requires PWA on home screen for iOS                   │
└────────────────────────────────────────────────────────────────┘

ERROR HANDLING:
┌────────────────────────────────────────────────────────────────┐
│ 404/410 - Token invalid, remove from database                  │
│ 429     - Rate limited, exponential backoff                    │
│ 500+    - Server error, retry with backoff                     │
│ 400     - Bad request, check payload format                    │
└────────────────────────────────────────────────────────────────┘

PAYLOAD LIMITS:
┌────────────────────────────────────────────────────────────────┐
│ Web Push   ~4KB (encrypted)                                    │
│ FCM        4KB                                                 │
│ APNs       4KB                                                 │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

