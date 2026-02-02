# 📱 Progressive Web Apps (PWA) - Complete Guide

> A comprehensive guide to Progressive Web Apps - manifest files, service workers, offline support, installability, and building app-like web experiences.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Progressive Web Apps use modern web capabilities (service workers, manifest, HTTPS) to deliver app-like experiences that are reliable (offline), fast (cached), and engaging (installable, push notifications)."

### The 7 Key Concepts (Remember These!)
```
1. MANIFEST          → Metadata for install (name, icons, display)
2. SERVICE WORKER    → Script for offline, caching, background tasks
3. HTTPS             → Required for security
4. INSTALLABILITY    → Add to home screen
5. OFFLINE SUPPORT   → Works without network
6. PUSH NOTIFICATIONS → Engage users when away
7. APP SHELL         → Minimal UI cached for instant load
```

### PWA Requirements
```
┌─────────────────────────────────────────────────────────────────┐
│              PWA REQUIREMENTS                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  MINIMUM REQUIREMENTS:                                         │
│  ─────────────────────                                          │
│  □ HTTPS (or localhost)                                        │
│  □ Web App Manifest with:                                      │
│    • name or short_name                                        │
│    • icons (192px and 512px)                                   │
│    • start_url                                                 │
│    • display (standalone/fullscreen/minimal-ui)                │
│  □ Service Worker (for installability)                         │
│                                                                 │
│  LIGHTHOUSE PWA AUDIT:                                         │
│  ─────────────────────                                          │
│  □ Fast and reliable (loads fast, works offline)               │
│  □ Installable (manifest + SW + HTTPS)                         │
│  □ PWA optimized (splash screen, theme color)                  │
│                                                                 │
│  ENHANCED CAPABILITIES:                                        │
│  ──────────────────────                                         │
│  □ Offline support                                             │
│  □ Push notifications                                          │
│  □ Background sync                                             │
│  □ Share target                                                │
│  □ File handling                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Service Worker Strategies
```
┌─────────────────────────────────────────────────────────────────┐
│              CACHING STRATEGIES                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CACHE FIRST (Cache, falling back to Network)                  │
│  ────────────────────────────────────────────                   │
│  • Check cache → if miss, fetch network                        │
│  • Best for: Static assets, fonts, images                      │
│  • Fast, may be stale                                          │
│                                                                 │
│  NETWORK FIRST (Network, falling back to Cache)                │
│  ────────────────────────────────────────────────               │
│  • Try network → if fail, use cache                            │
│  • Best for: API calls, frequently changing data               │
│  • Fresh data, offline fallback                                │
│                                                                 │
│  STALE-WHILE-REVALIDATE                                        │
│  ─────────────────────────                                      │
│  • Return cache immediately, fetch update for next time        │
│  • Best for: Assets that update occasionally                   │
│  • Instant load, eventually fresh                              │
│                                                                 │
│  CACHE ONLY                                                    │
│  ───────────                                                    │
│  • Only serve from cache                                       │
│  • Best for: Precached static assets                           │
│                                                                 │
│  NETWORK ONLY                                                  │
│  ────────────                                                   │
│  • Always go to network                                        │
│  • Best for: Non-cacheable requests (analytics)                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"App shell"** | "We precache the app shell for instant loading" |
| **"Workbox"** | "We use Workbox for service worker caching strategies" |
| **"beforeinstallprompt"** | "We capture beforeinstallprompt to show custom install UI" |
| **"Background sync"** | "Background sync queues requests when offline" |
| **"Cache-first"** | "Static assets use cache-first strategy" |
| **"Installable"** | "PWA is installable once it meets criteria" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Icon sizes | **192px, 512px** | Required for manifest |
| SW max age | **24 hours** | Browser update check |
| Cache storage | **~50MB** | Per origin limit |
| Install prompt | **User gesture** | Can't auto-show |

### The "Wow" Statement (Memorize This!)
> "Our PWA has sub-second repeat loads by precaching the app shell - HTML, CSS, core JS. Static assets use cache-first strategy, API calls use stale-while-revalidate. Service worker is built with Workbox for simpler strategy configuration. We capture beforeinstallprompt and show a custom install banner after user engagement. Offline mode shows cached content plus a banner. Background sync queues form submissions when offline. Push notifications are opt-in with clear value prop. Lighthouse PWA score is 100. The result: 40% of mobile users installed the PWA, engagement increased 2x vs mobile web."

---

## 📚 Table of Contents

1. [Web App Manifest](#1-web-app-manifest)
2. [Service Workers](#2-service-workers)
3. [Caching Strategies](#3-caching-strategies)
4. [Install Experience](#4-install-experience)
5. [Push Notifications](#5-push-notifications)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Web App Manifest

```json
// public/manifest.json
{
  "name": "My Awesome App",
  "short_name": "MyApp",
  "description": "An awesome progressive web app",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#3b82f6",
  "orientation": "portrait-primary",
  "scope": "/",
  "icons": [
    {
      "src": "/icons/icon-72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/icons/icon-maskable-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }
  ],
  "screenshots": [
    {
      "src": "/screenshots/screenshot1.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide"
    },
    {
      "src": "/screenshots/screenshot2.png",
      "sizes": "750x1334",
      "type": "image/png",
      "form_factor": "narrow"
    }
  ],
  "shortcuts": [
    {
      "name": "New Task",
      "short_name": "New",
      "description": "Create a new task",
      "url": "/tasks/new",
      "icons": [{ "src": "/icons/new-task.png", "sizes": "96x96" }]
    }
  ],
  "share_target": {
    "action": "/share",
    "method": "POST",
    "enctype": "multipart/form-data",
    "params": {
      "title": "title",
      "text": "text",
      "url": "url",
      "files": [
        {
          "name": "images",
          "accept": ["image/*"]
        }
      ]
    }
  }
}
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <!-- Manifest link -->
  <link rel="manifest" href="/manifest.json">
  
  <!-- Theme color for browser UI -->
  <meta name="theme-color" content="#3b82f6">
  
  <!-- iOS specific -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="default">
  <meta name="apple-mobile-web-app-title" content="MyApp">
  <link rel="apple-touch-icon" href="/icons/icon-152.png">
  
  <!-- Splash screens for iOS -->
  <link rel="apple-touch-startup-image" href="/splash/splash-640x1136.png" media="(device-width: 320px) and (device-height: 568px)">
  
  <!-- Windows -->
  <meta name="msapplication-TileImage" content="/icons/icon-144.png">
  <meta name="msapplication-TileColor" content="#3b82f6">
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

---

## 2. Service Workers

```typescript
// ════════════════════════════════════════════════════════════════
// SERVICE WORKER LIFECYCLE
// ════════════════════════════════════════════════════════════════

// sw.ts - Basic service worker
const CACHE_NAME = 'my-app-v1';
const STATIC_ASSETS = [
  '/',
  '/index.html',
  '/styles/main.css',
  '/scripts/main.js',
  '/icons/icon-192.png',
];

// INSTALL - Precache static assets
self.addEventListener('install', (event: ExtendableEvent) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      console.log('Caching static assets');
      return cache.addAll(STATIC_ASSETS);
    })
  );
  // Activate immediately (skip waiting)
  self.skipWaiting();
});

// ACTIVATE - Clean up old caches
self.addEventListener('activate', (event: ExtendableEvent) => {
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames
          .filter((name) => name !== CACHE_NAME)
          .map((name) => caches.delete(name))
      );
    })
  );
  // Take control immediately
  self.clients.claim();
});

// FETCH - Intercept network requests
self.addEventListener('fetch', (event: FetchEvent) => {
  event.respondWith(
    caches.match(event.request).then((cachedResponse) => {
      if (cachedResponse) {
        return cachedResponse;
      }
      return fetch(event.request);
    })
  );
});

// ════════════════════════════════════════════════════════════════
// REGISTERING SERVICE WORKER
// ════════════════════════════════════════════════════════════════

// src/registerSW.ts
export async function registerServiceWorker() {
  if ('serviceWorker' in navigator) {
    try {
      const registration = await navigator.serviceWorker.register('/sw.js', {
        scope: '/',
      });

      // Check for updates
      registration.addEventListener('updatefound', () => {
        const newWorker = registration.installing;
        if (newWorker) {
          newWorker.addEventListener('statechange', () => {
            if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
              // New version available
              showUpdateNotification();
            }
          });
        }
      });

      console.log('SW registered:', registration.scope);
    } catch (error) {
      console.error('SW registration failed:', error);
    }
  }
}

// Show update notification
function showUpdateNotification() {
  const shouldUpdate = confirm('New version available! Reload to update?');
  if (shouldUpdate) {
    window.location.reload();
  }
}

// Skip waiting and reload (trigger from UI)
export function skipWaitingAndReload() {
  navigator.serviceWorker.ready.then((registration) => {
    registration.waiting?.postMessage({ type: 'SKIP_WAITING' });
  });

  // Reload when new SW takes over
  navigator.serviceWorker.addEventListener('controllerchange', () => {
    window.location.reload();
  });
}

// In service worker, listen for skip waiting message
self.addEventListener('message', (event) => {
  if (event.data?.type === 'SKIP_WAITING') {
    self.skipWaiting();
  }
});
```

---

## 3. Caching Strategies

```typescript
// ════════════════════════════════════════════════════════════════
// WORKBOX (Recommended for production)
// ════════════════════════════════════════════════════════════════

// sw.ts using Workbox
import { precacheAndRoute, cleanupOutdatedCaches } from 'workbox-precaching';
import { registerRoute } from 'workbox-routing';
import { 
  CacheFirst, 
  NetworkFirst, 
  StaleWhileRevalidate 
} from 'workbox-strategies';
import { CacheableResponsePlugin } from 'workbox-cacheable-response';
import { ExpirationPlugin } from 'workbox-expiration';

// Precache static assets (injected at build time)
precacheAndRoute(self.__WB_MANIFEST);

// Clean up old caches
cleanupOutdatedCaches();

// Cache First - Static assets (images, fonts)
registerRoute(
  ({ request }) => 
    request.destination === 'image' ||
    request.destination === 'font',
  new CacheFirst({
    cacheName: 'static-assets',
    plugins: [
      new CacheableResponsePlugin({
        statuses: [0, 200],
      }),
      new ExpirationPlugin({
        maxEntries: 100,
        maxAgeSeconds: 60 * 60 * 24 * 30, // 30 days
      }),
    ],
  })
);

// Network First - API calls
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  new NetworkFirst({
    cacheName: 'api-cache',
    networkTimeoutSeconds: 10,
    plugins: [
      new CacheableResponsePlugin({
        statuses: [0, 200],
      }),
      new ExpirationPlugin({
        maxEntries: 50,
        maxAgeSeconds: 60 * 60 * 24, // 24 hours
      }),
    ],
  })
);

// Stale While Revalidate - JS/CSS
registerRoute(
  ({ request }) =>
    request.destination === 'script' ||
    request.destination === 'style',
  new StaleWhileRevalidate({
    cacheName: 'static-resources',
    plugins: [
      new CacheableResponsePlugin({
        statuses: [0, 200],
      }),
    ],
  })
);

// ════════════════════════════════════════════════════════════════
// VITE PWA PLUGIN
// ════════════════════════════════════════════════════════════════

// vite.config.ts
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    VitePWA({
      registerType: 'autoUpdate',
      includeAssets: ['favicon.ico', 'apple-touch-icon.png'],
      manifest: {
        name: 'My App',
        short_name: 'MyApp',
        theme_color: '#3b82f6',
        icons: [
          {
            src: 'pwa-192x192.png',
            sizes: '192x192',
            type: 'image/png',
          },
          {
            src: 'pwa-512x512.png',
            sizes: '512x512',
            type: 'image/png',
          },
        ],
      },
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff2}'],
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/api\.example\.com\/.*/i,
            handler: 'NetworkFirst',
            options: {
              cacheName: 'api-cache',
              expiration: {
                maxEntries: 100,
                maxAgeSeconds: 60 * 60 * 24, // 24 hours
              },
            },
          },
        ],
      },
    }),
  ],
});

// ════════════════════════════════════════════════════════════════
// OFFLINE FALLBACK
// ════════════════════════════════════════════════════════════════

// Serve offline page when network fails
import { setCatchHandler } from 'workbox-routing';

// Offline fallback page
setCatchHandler(async ({ event }) => {
  if (event.request.destination === 'document') {
    return caches.match('/offline.html');
  }
  
  // For images, return placeholder
  if (event.request.destination === 'image') {
    return caches.match('/placeholder.svg');
  }
  
  return Response.error();
});
```

---

## 4. Install Experience

```tsx
// ════════════════════════════════════════════════════════════════
// CUSTOM INSTALL PROMPT
// ════════════════════════════════════════════════════════════════

import { useState, useEffect } from 'react';

interface BeforeInstallPromptEvent extends Event {
  prompt(): Promise<void>;
  userChoice: Promise<{ outcome: 'accepted' | 'dismissed' }>;
}

function useInstallPrompt() {
  const [installPrompt, setInstallPrompt] = useState<BeforeInstallPromptEvent | null>(null);
  const [isInstalled, setIsInstalled] = useState(false);

  useEffect(() => {
    // Check if already installed
    if (window.matchMedia('(display-mode: standalone)').matches) {
      setIsInstalled(true);
      return;
    }

    // Capture the install prompt
    const handleBeforeInstallPrompt = (e: Event) => {
      e.preventDefault(); // Prevent automatic prompt
      setInstallPrompt(e as BeforeInstallPromptEvent);
    };

    // Track successful install
    const handleAppInstalled = () => {
      setIsInstalled(true);
      setInstallPrompt(null);
    };

    window.addEventListener('beforeinstallprompt', handleBeforeInstallPrompt);
    window.addEventListener('appinstalled', handleAppInstalled);

    return () => {
      window.removeEventListener('beforeinstallprompt', handleBeforeInstallPrompt);
      window.removeEventListener('appinstalled', handleAppInstalled);
    };
  }, []);

  const install = async () => {
    if (!installPrompt) return;

    // Show the native prompt
    await installPrompt.prompt();

    // Wait for user choice
    const { outcome } = await installPrompt.userChoice;
    
    if (outcome === 'accepted') {
      setInstallPrompt(null);
    }

    return outcome;
  };

  return {
    canInstall: !!installPrompt && !isInstalled,
    isInstalled,
    install,
  };
}

// Install Banner Component
function InstallBanner() {
  const { canInstall, install } = useInstallPrompt();
  const [dismissed, setDismissed] = useState(false);

  if (!canInstall || dismissed) return null;

  return (
    <div className="install-banner">
      <div className="install-banner-content">
        <img src="/icons/icon-72.png" alt="" className="install-icon" />
        <div>
          <h3>Install Our App</h3>
          <p>Add to home screen for the best experience</p>
        </div>
      </div>
      <div className="install-banner-actions">
        <button onClick={() => setDismissed(true)}>Not now</button>
        <button onClick={install} className="primary">Install</button>
      </div>
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// DETECT INSTALL STATE
// ════════════════════════════════════════════════════════════════

function useDisplayMode() {
  const [displayMode, setDisplayMode] = useState<'browser' | 'standalone' | 'minimal-ui'>('browser');

  useEffect(() => {
    const updateDisplayMode = () => {
      if (window.matchMedia('(display-mode: standalone)').matches) {
        setDisplayMode('standalone');
      } else if (window.matchMedia('(display-mode: minimal-ui)').matches) {
        setDisplayMode('minimal-ui');
      } else {
        setDisplayMode('browser');
      }
    };

    updateDisplayMode();

    // Listen for changes (e.g., install)
    window.matchMedia('(display-mode: standalone)').addEventListener('change', updateDisplayMode);

    return () => {
      window.matchMedia('(display-mode: standalone)').removeEventListener('change', updateDisplayMode);
    };
  }, []);

  return displayMode;
}

// Usage: Show different UI based on display mode
function App() {
  const displayMode = useDisplayMode();

  return (
    <div>
      {displayMode === 'browser' && <InstallBanner />}
      {displayMode === 'standalone' && <AppHeader />}
    </div>
  );
}
```

---

## 5. Push Notifications

```typescript
// ════════════════════════════════════════════════════════════════
// PUSH NOTIFICATION SETUP
// ════════════════════════════════════════════════════════════════

// Request permission and subscribe
async function subscribeToPush() {
  // Check support
  if (!('PushManager' in window)) {
    throw new Error('Push notifications not supported');
  }

  // Request permission
  const permission = await Notification.requestPermission();
  if (permission !== 'granted') {
    throw new Error('Permission denied');
  }

  // Get service worker registration
  const registration = await navigator.serviceWorker.ready;

  // Subscribe to push
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true, // Required
    applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY),
  });

  // Send subscription to server
  await fetch('/api/push/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(subscription),
  });

  return subscription;
}

// Convert VAPID key
function urlBase64ToUint8Array(base64String: string) {
  const padding = '='.repeat((4 - (base64String.length % 4)) % 4);
  const base64 = (base64String + padding)
    .replace(/-/g, '+')
    .replace(/_/g, '/');
  const rawData = window.atob(base64);
  return Uint8Array.from([...rawData].map((char) => char.charCodeAt(0)));
}

// ════════════════════════════════════════════════════════════════
// SERVICE WORKER PUSH HANDLER
// ════════════════════════════════════════════════════════════════

// sw.ts
self.addEventListener('push', (event: PushEvent) => {
  const data = event.data?.json() ?? {
    title: 'New Notification',
    body: 'You have a new notification',
  };

  const options: NotificationOptions = {
    body: data.body,
    icon: '/icons/icon-192.png',
    badge: '/icons/badge-72.png',
    vibrate: [100, 50, 100],
    data: {
      url: data.url || '/',
    },
    actions: data.actions || [],
    tag: data.tag, // Group notifications
    renotify: data.renotify || false,
  };

  event.waitUntil(
    self.registration.showNotification(data.title, options)
  );
});

// Handle notification click
self.addEventListener('notificationclick', (event: NotificationEvent) => {
  event.notification.close();

  const url = event.notification.data?.url || '/';

  // Handle action buttons
  if (event.action === 'view') {
    // Open specific URL
  }

  event.waitUntil(
    clients.matchAll({ type: 'window', includeUncontrolled: true })
      .then((windowClients) => {
        // Focus existing window or open new
        for (const client of windowClients) {
          if (client.url === url && 'focus' in client) {
            return client.focus();
          }
        }
        return clients.openWindow(url);
      })
  );
});

// ════════════════════════════════════════════════════════════════
// SERVER-SIDE PUSH (Node.js)
// ════════════════════════════════════════════════════════════════

// server/push.ts
import webpush from 'web-push';

webpush.setVapidDetails(
  'mailto:admin@example.com',
  process.env.VAPID_PUBLIC_KEY!,
  process.env.VAPID_PRIVATE_KEY!
);

async function sendPushNotification(
  subscription: webpush.PushSubscription,
  payload: { title: string; body: string; url?: string }
) {
  try {
    await webpush.sendNotification(
      subscription,
      JSON.stringify(payload)
    );
  } catch (error) {
    if (error.statusCode === 410) {
      // Subscription expired, remove from database
      await removeSubscription(subscription.endpoint);
    }
    throw error;
  }
}
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# PWA PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Caching everything forever
# Bad
cache.addAll(ALL_ASSETS);  # Never updates

# Good
# Use versioned cache names
# Clean old caches on activate
# Set appropriate expiration

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Not handling SW updates
# Bad
# User stuck on old version forever

# Good
# Detect updatefound
# Prompt user to refresh
# Or use autoUpdate with Vite PWA

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Caching API responses incorrectly
# Bad
# Cache POST requests
# Cache personalized data

# Good
# Only cache GET requests
# Don't cache user-specific data
# Use NetworkFirst for fresh data

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Missing offline fallback
# Bad
# Network fails = blank screen

# Good
# Serve offline.html for navigation
# Show placeholder for images
# Queue requests for later

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Aggressive install prompts
# Bad
# Show install prompt immediately on load

# Good
# Wait for user engagement
# Show after positive interaction
# Respect "not now" dismissal
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is a Progressive Web App?"**
> "A PWA uses modern web technologies to deliver app-like experiences. Key features: works offline (service worker), installable (manifest + HTTPS), fast (caching). It's a website that can be installed and works like a native app while being a web app."

**Q: "What are the minimum requirements for a PWA?"**
> "HTTPS (or localhost), a web app manifest with name, icons (192px and 512px minimum), start_url, and display mode. Plus a registered service worker. These enable the browser to offer installation."

**Q: "What is a service worker?"**
> "A JavaScript file that runs in the background, separate from the web page. It intercepts network requests, enabling offline support and caching. Also handles push notifications and background sync. Has its own lifecycle: install, activate, fetch."

### Intermediate Questions

**Q: "Explain the different caching strategies."**
> "Cache First: serve from cache, fallback to network - fast, potentially stale. Network First: try network, fallback to cache - fresh data, offline support. Stale While Revalidate: serve cache immediately, update in background - instant + eventually fresh."

**Q: "How do you handle service worker updates?"**
> "New SW installs in background, waits until old one is inactive. Detect via updatefound event. Options: 1) skipWaiting to activate immediately (may break). 2) Prompt user to refresh. 3) Wait for all tabs to close. Vite PWA autoUpdate handles this."

**Q: "What is the app shell architecture?"**
> "Precache minimal UI (shell) - HTML, CSS, core JS. Shell loads instantly from cache. Content loads dynamically. First visit: cache shell. Subsequent: instant shell, then fetch content. Feels like a native app."

### Advanced Questions

**Q: "How do you implement push notifications?"**
> "1) Request Notification permission. 2) Get PushManager subscription with VAPID key. 3) Send subscription to server. 4) Server sends push via web-push library. 5) SW push event shows notification. Handle notificationclick to open app."

**Q: "How do you handle offline form submissions?"**
> "Background Sync API: register sync event, service worker handles when online. Or: store in IndexedDB, show queued indicator, send when online. Queue management: retry logic, expiration, conflict resolution."

**Q: "What are the limitations of PWAs?"**
> "iOS limitations: no push on iOS (until iOS 16.4), limited background sync, Safari install less prominent. No access to: contacts, SMS, certain sensors. App store distribution requires wrapper. Battery/memory management not as optimized."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              PWA CHECKLIST                                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  MANIFEST:                                                      │
│  □ name and short_name                                         │
│  □ Icons (192px, 512px)                                        │
│  □ start_url                                                   │
│  □ display: standalone                                         │
│  □ theme_color                                                 │
│                                                                 │
│  SERVICE WORKER:                                                │
│  □ Registered successfully                                     │
│  □ Precaches app shell                                         │
│  □ Handles fetch events                                        │
│  □ Offline fallback                                            │
│  □ Update handling                                             │
│                                                                 │
│  PERFORMANCE:                                                   │
│  □ Cache static assets                                         │
│  □ Appropriate cache strategies                                │
│  □ Cache expiration policies                                   │
│                                                                 │
│  USER EXPERIENCE:                                               │
│  □ Custom install prompt                                       │
│  □ Offline indicator                                           │
│  □ Update notification                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

CACHING STRATEGIES:
┌────────────────────────────────────────────────────────────────┐
│ Cache First:        Static assets, fonts, images               │
│ Network First:      API calls, frequently changing data        │
│ Stale-Revalidate:   JS/CSS, occasionally updating assets       │
│ Cache Only:         Precached static files                     │
│ Network Only:       Analytics, non-cacheable requests          │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

