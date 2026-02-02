# Error Tracking - Complete Guide

> **MUST REMEMBER**: Error tracking services (Sentry, Bugsnag) automatically capture, group, and alert on application errors. Key features: source maps for readable stack traces, error grouping to reduce noise, breadcrumbs showing events before the error, and release tracking to identify regressions. Essential for production debugging.

---

## How to Explain Like a Senior Developer

"Console.error doesn't cut it in production. You need to know when errors happen, how often, which users are affected, and what they were doing. Sentry captures all this automatically - stack traces, browser info, the user's journey leading up to the crash. The killer feature is error grouping: instead of 10,000 alerts for the same bug, you see '1 issue affecting 10,000 users'. Source maps decode minified code so you see your actual source. Release tracking shows 'this bug started with version 2.3.1'. Set it up on day one - it's how you maintain quality at scale."

---

## Core Implementation

### Sentry Setup for Node.js

```typescript
// sentry/init.ts
import * as Sentry from '@sentry/node';
import { ProfilingIntegration } from '@sentry/profiling-node';
import { Express } from 'express';

export function initSentry(app: Express): void {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV || 'development',
    release: `${process.env.SERVICE_NAME}@${process.env.SERVICE_VERSION}`,
    
    // Performance monitoring
    tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
    
    // Profiling
    profilesSampleRate: 0.1,
    
    integrations: [
      // Express integration
      new Sentry.Integrations.Express({ app }),
      // HTTP integration
      new Sentry.Integrations.Http({ tracing: true }),
      // Profiling
      new ProfilingIntegration(),
    ],
    
    // Filter sensitive data
    beforeSend(event, hint) {
      // Remove sensitive data
      if (event.request?.data) {
        const data = event.request.data as any;
        if (data.password) data.password = '[REDACTED]';
        if (data.token) data.token = '[REDACTED]';
      }
      
      // Don't send certain errors
      const error = hint.originalException as Error;
      if (error?.message?.includes('canceled')) {
        return null; // Don't send canceled requests
      }
      
      return event;
    },
    
    // Breadcrumb filtering
    beforeBreadcrumb(breadcrumb) {
      // Filter out noisy breadcrumbs
      if (breadcrumb.category === 'console' && breadcrumb.level === 'debug') {
        return null;
      }
      return breadcrumb;
    },
    
    // Ignore certain errors
    ignoreErrors: [
      'Network Error',
      'ECONNREFUSED',
      /^Loading chunk \d+ failed/,
    ],
  });
}

// Express middleware setup
export function setupSentryMiddleware(app: Express): void {
  // RequestHandler creates a separate execution context
  app.use(Sentry.Handlers.requestHandler());
  
  // TracingHandler creates a trace for every incoming request
  app.use(Sentry.Handlers.tracingHandler());
}

// Error handler (must be after all routes)
export function setupSentryErrorHandler(app: Express): void {
  app.use(Sentry.Handlers.errorHandler({
    shouldHandleError(error: any) {
      // Capture 4xx and 5xx errors
      if (error.statusCode >= 400) {
        return true;
      }
      return true;
    },
  }));
}
```

### Sentry Setup for React

```typescript
// sentry/browser.ts
import * as Sentry from '@sentry/react';
import { BrowserTracing } from '@sentry/tracing';

export function initSentryBrowser(): void {
  Sentry.init({
    dsn: process.env.REACT_APP_SENTRY_DSN,
    environment: process.env.NODE_ENV,
    release: `frontend@${process.env.REACT_APP_VERSION}`,
    
    integrations: [
      new BrowserTracing({
        // Trace requests to these origins
        tracePropagationTargets: [
          'localhost',
          /^https:\/\/api\.example\.com/,
        ],
        // Create spans for route changes
        routingInstrumentation: Sentry.reactRouterV6Instrumentation(
          React.useEffect,
          useLocation,
          useNavigationType,
          createRoutesFromChildren,
          matchRoutes
        ),
      }),
      new Sentry.Replay({
        // Session replay for errors
        maskAllText: true,
        blockAllMedia: true,
      }),
    ],
    
    // Performance sampling
    tracesSampleRate: 0.1,
    
    // Replay sampling
    replaysSessionSampleRate: 0.1, // Sample 10% of sessions
    replaysOnErrorSampleRate: 1.0, // Always capture on error
    
    // Filter errors
    beforeSend(event, hint) {
      // Filter out browser extension errors
      if (event.exception?.values?.[0]?.stacktrace?.frames?.some(
        (frame) => frame.filename?.includes('chrome-extension')
      )) {
        return null;
      }
      return event;
    },
  });
}

// Error Boundary component
import { ErrorBoundary } from '@sentry/react';
import React from 'react';

export function AppErrorBoundary({ children }: { children: React.ReactNode }) {
  return (
    <ErrorBoundary
      fallback={({ error, resetError }) => (
        <div className="error-page">
          <h1>Something went wrong</h1>
          <p>{error.message}</p>
          <button onClick={resetError}>Try again</button>
        </div>
      )}
      onError={(error, componentStack) => {
        console.error('Error caught by boundary:', error, componentStack);
      }}
    >
      {children}
    </ErrorBoundary>
  );
}

// Mock imports for type checking
import { useLocation, useNavigationType, createRoutesFromChildren, matchRoutes } from 'react-router-dom';
```

### Custom Error Capturing

```typescript
// error-capture.ts
import * as Sentry from '@sentry/node';

// Capture error with context
export function captureError(
  error: Error,
  context?: {
    user?: { id: string; email?: string };
    tags?: Record<string, string>;
    extra?: Record<string, unknown>;
    level?: Sentry.SeverityLevel;
  }
): string {
  return Sentry.withScope((scope) => {
    // Set user context
    if (context?.user) {
      scope.setUser({
        id: context.user.id,
        email: context.user.email,
      });
    }
    
    // Set tags for filtering
    if (context?.tags) {
      Object.entries(context.tags).forEach(([key, value]) => {
        scope.setTag(key, value);
      });
    }
    
    // Set extra context
    if (context?.extra) {
      Object.entries(context.extra).forEach(([key, value]) => {
        scope.setExtra(key, value);
      });
    }
    
    // Set severity level
    if (context?.level) {
      scope.setLevel(context.level);
    }
    
    return Sentry.captureException(error);
  });
}

// Capture message (for non-exception events)
export function captureMessage(
  message: string,
  level: Sentry.SeverityLevel = 'info',
  context?: Record<string, unknown>
): string {
  return Sentry.withScope((scope) => {
    if (context) {
      Object.entries(context).forEach(([key, value]) => {
        scope.setExtra(key, value);
      });
    }
    return Sentry.captureMessage(message, level);
  });
}

// Usage examples
async function handlePayment(orderId: string, userId: string): Promise<void> {
  try {
    await processPayment(orderId);
  } catch (error) {
    captureError(error as Error, {
      user: { id: userId },
      tags: {
        feature: 'payment',
        payment_provider: 'stripe',
      },
      extra: {
        orderId,
        timestamp: new Date().toISOString(),
      },
      level: 'error',
    });
    throw error;
  }
}

async function processPayment(orderId: string): Promise<void> {}
```

### Breadcrumbs

```typescript
// breadcrumbs.ts
import * as Sentry from '@sentry/node';

// Add custom breadcrumb
export function addBreadcrumb(
  message: string,
  category: string,
  data?: Record<string, unknown>,
  level: Sentry.SeverityLevel = 'info'
): void {
  Sentry.addBreadcrumb({
    message,
    category,
    level,
    data,
    timestamp: Date.now() / 1000,
  });
}

// Navigation breadcrumb
export function trackNavigation(from: string, to: string): void {
  addBreadcrumb(`Navigated from ${from} to ${to}`, 'navigation', {
    from,
    to,
  });
}

// User action breadcrumb
export function trackUserAction(action: string, element?: string): void {
  addBreadcrumb(action, 'user', {
    element,
  });
}

// API request breadcrumb
export function trackApiRequest(
  method: string,
  url: string,
  statusCode: number
): void {
  addBreadcrumb(`${method} ${url}`, 'http', {
    method,
    url,
    status_code: statusCode,
  }, statusCode >= 400 ? 'warning' : 'info');
}

// Example: Axios interceptor for automatic breadcrumbs
import axios from 'axios';

axios.interceptors.response.use(
  (response) => {
    trackApiRequest(
      response.config.method?.toUpperCase() || 'GET',
      response.config.url || '',
      response.status
    );
    return response;
  },
  (error) => {
    if (error.response) {
      trackApiRequest(
        error.config?.method?.toUpperCase() || 'GET',
        error.config?.url || '',
        error.response.status
      );
    }
    return Promise.reject(error);
  }
);
```

### Release Tracking and Source Maps

```typescript
// release-tracking.ts
import * as Sentry from '@sentry/node';

// Set release in Sentry config
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  release: `my-app@${process.env.npm_package_version}`,
});

// Upload source maps in CI/CD
// package.json script:
// "sentry:sourcemaps": "sentry-cli releases files $VERSION upload-sourcemaps ./dist"

/*
# .sentryclirc
[defaults]
url=https://sentry.io/
org=your-org
project=your-project

[auth]
token=your-auth-token
*/

// CI/CD script (GitHub Actions)
/*
- name: Create Sentry release
  env:
    SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
    SENTRY_ORG: your-org
    SENTRY_PROJECT: your-project
    VERSION: ${{ github.sha }}
  run: |
    # Install Sentry CLI
    npm install -g @sentry/cli
    
    # Create release
    sentry-cli releases new $VERSION
    
    # Upload source maps
    sentry-cli releases files $VERSION upload-sourcemaps ./dist --url-prefix '~/dist'
    
    # Associate commits
    sentry-cli releases set-commits $VERSION --auto
    
    # Finalize release
    sentry-cli releases finalize $VERSION
    
    # Create deploy
    sentry-cli releases deploys $VERSION new -e production
*/

// Webpack config for source maps
/*
module.exports = {
  devtool: 'source-map', // Generate source maps
  plugins: [
    new SentryWebpackPlugin({
      authToken: process.env.SENTRY_AUTH_TOKEN,
      org: 'your-org',
      project: 'your-project',
      release: process.env.VERSION,
      include: './dist',
      ignore: ['node_modules'],
      urlPrefix: '~/dist',
    }),
  ],
};
*/
```

### Performance Monitoring with Sentry

```typescript
// performance.ts
import * as Sentry from '@sentry/node';

// Manual transaction
async function processOrder(orderId: string): Promise<void> {
  const transaction = Sentry.startTransaction({
    op: 'order.process',
    name: 'Process Order',
    data: { orderId },
  });
  
  Sentry.getCurrentHub().configureScope((scope) => {
    scope.setSpan(transaction);
  });
  
  try {
    // Create child spans
    const validationSpan = transaction.startChild({
      op: 'order.validate',
      description: 'Validate order',
    });
    await validateOrder(orderId);
    validationSpan.finish();
    
    const paymentSpan = transaction.startChild({
      op: 'payment.process',
      description: 'Process payment',
    });
    await processPayment(orderId);
    paymentSpan.finish();
    
    const fulfillmentSpan = transaction.startChild({
      op: 'order.fulfill',
      description: 'Fulfill order',
    });
    await fulfillOrder(orderId);
    fulfillmentSpan.finish();
    
    transaction.setStatus('ok');
  } catch (error) {
    transaction.setStatus('internal_error');
    throw error;
  } finally {
    transaction.finish();
  }
}

// Database query spans
async function queryDatabase(query: string): Promise<any[]> {
  const span = Sentry.getCurrentHub().getScope()?.getSpan()?.startChild({
    op: 'db.query',
    description: query.substring(0, 100),
  });
  
  try {
    const result = await db.query(query);
    span?.setStatus('ok');
    return result;
  } catch (error) {
    span?.setStatus('internal_error');
    throw error;
  } finally {
    span?.finish();
  }
}

async function validateOrder(orderId: string): Promise<void> {}
async function processPayment(orderId: string): Promise<void> {}
async function fulfillOrder(orderId: string): Promise<void> {}
const db = { query: async (q: string) => [] };
```

---

## Real-World Scenarios

### Scenario 1: User Context and Session Replay

```typescript
// user-context.ts
import * as Sentry from '@sentry/node';

interface User {
  id: string;
  email: string;
  name: string;
  plan: 'free' | 'pro' | 'enterprise';
}

// Set user context on login
export function setUserContext(user: User): void {
  Sentry.setUser({
    id: user.id,
    email: user.email,
    username: user.name,
  });
  
  Sentry.setTag('user.plan', user.plan);
}

// Clear on logout
export function clearUserContext(): void {
  Sentry.setUser(null);
}

// Express middleware
import { Request, Response, NextFunction } from 'express';

export function sentryUserMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
): void {
  const user = (req as any).user;
  
  if (user) {
    Sentry.setUser({
      id: user.id,
      email: user.email,
    });
    Sentry.setTag('user.role', user.role);
  }
  
  next();
}

// Set custom context for better debugging
export function setOrderContext(order: {
  id: string;
  total: number;
  items: number;
}): void {
  Sentry.setContext('order', {
    id: order.id,
    total: order.total,
    itemCount: order.items,
  });
}
```

### Scenario 2: Alert Rules and Issue Management

```typescript
// alerts.ts
/*
Sentry Alert Rules (configured in UI or via API):

1. Error Spike Alert:
   - Trigger: Number of events > 100 in 1 hour
   - Action: Slack notification to #errors channel

2. New Issue Alert:
   - Trigger: First time seeing this issue
   - Action: Email to engineering team

3. Regression Alert:
   - Trigger: Issue marked resolved reappears
   - Action: Slack notification + PagerDuty

4. Specific Error Alert:
   - Trigger: Error message contains "payment"
   - Action: Immediate PagerDuty + Slack

5. User Impact Alert:
   - Trigger: > 100 unique users affected
   - Action: Slack notification to #incidents
*/

// Programmatic issue management
import * as Sentry from '@sentry/node';

// Mark handled errors
export function captureHandledError(error: Error): void {
  Sentry.withScope((scope) => {
    scope.setTag('handled', 'true');
    scope.setLevel('warning'); // Lower severity for handled errors
    Sentry.captureException(error);
  });
}

// Fingerprinting for better grouping
export function captureWithFingerprint(
  error: Error,
  fingerprint: string[]
): void {
  Sentry.withScope((scope) => {
    scope.setFingerprint(fingerprint);
    Sentry.captureException(error);
  });
}

// Example: Group all payment provider errors together
async function handlePaymentError(error: Error, provider: string): Promise<void> {
  Sentry.withScope((scope) => {
    // Custom fingerprint to group by provider
    scope.setFingerprint(['payment-error', provider]);
    scope.setTag('payment.provider', provider);
    Sentry.captureException(error);
  });
}
```

---

## Common Pitfalls

### 1. Capturing Too Many Errors

```typescript
// ❌ BAD: Capturing expected errors
try {
  const user = await findUser(id);
} catch (error) {
  Sentry.captureException(error); // NotFoundError is expected!
  return null;
}

// ✅ GOOD: Only capture unexpected errors
try {
  const user = await findUser(id);
} catch (error) {
  if (error instanceof NotFoundError) {
    return null; // Expected, don't report
  }
  Sentry.captureException(error); // Unexpected, report it
  throw error;
}

class NotFoundError extends Error {}
async function findUser(id: string): Promise<any> { return null; }
```

### 2. Missing Source Maps

```typescript
// ❌ BAD: Minified stack traces are useless
// Error: n is not a function
//   at e.r (main.min.js:1:2345)

// ✅ GOOD: Upload source maps in CI/CD
// package.json
{
  "scripts": {
    "build": "webpack --mode production",
    "postbuild": "sentry-cli releases files $VERSION upload-sourcemaps ./dist"
  }
}
```

### 3. Not Setting User Context

```typescript
// ❌ BAD: Errors without user context
Sentry.captureException(error);
// Hard to identify affected users

// ✅ GOOD: Set user context on auth
Sentry.setUser({
  id: user.id,
  email: user.email,
});
// Now all errors include user info
```

---

## Interview Questions

### Q1: How does error grouping work in Sentry?

**A:** Sentry groups errors by "fingerprint" - a hash of stack trace, error type, and message. Same stack trace = same issue. You can customize fingerprinting for better grouping (e.g., group all "database connection" errors regardless of exact message). This reduces noise: instead of 10,000 alerts, you see "1 issue, 10,000 events".

### Q2: What are breadcrumbs and why are they useful?

**A:** Breadcrumbs are a trail of events (clicks, navigation, API calls, console logs) leading up to an error. They answer "what was the user doing before the crash?". Automatically captured for HTTP requests, console output, and DOM events. Essential for reproducing bugs that depend on specific user journeys.

### Q3: How do source maps improve error debugging?

**A:** Production code is minified/bundled, making stack traces unreadable (`main.js:1:2345`). Source maps map minified code back to original source files with line numbers. Upload them to Sentry, and errors show your actual source code. Keep source maps private (don't serve to browsers).

### Q4: How do you handle high error volume in production?

**A:** 1) Filter known/expected errors before sending, 2) Use sampling for high-volume issues, 3) Set up alert thresholds instead of per-error alerts, 4) Customize fingerprinting to reduce duplicates, 5) Prioritize by user impact (unique users affected).

---

## Quick Reference Checklist

### Setup
- [ ] Install SDK (Sentry/Bugsnag)
- [ ] Configure DSN and environment
- [ ] Set up source map uploads
- [ ] Configure release tracking

### Integration
- [ ] Add error boundaries (React)
- [ ] Set up Express/framework middleware
- [ ] Configure user context
- [ ] Add custom breadcrumbs

### Configuration
- [ ] Filter sensitive data
- [ ] Ignore expected errors
- [ ] Set sampling rates
- [ ] Configure custom fingerprinting

### Operations
- [ ] Set up alert rules
- [ ] Create issue assignment rules
- [ ] Configure integrations (Slack, PagerDuty)
- [ ] Review and triage regularly

---

*Last updated: February 2026*

