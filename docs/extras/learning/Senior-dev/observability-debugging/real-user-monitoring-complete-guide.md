# Real User Monitoring (RUM) - Complete Guide

> **MUST REMEMBER**: RUM captures actual user experience data - page loads, interactions, errors, and performance from real browsers/devices. Key metrics: Core Web Vitals (LCP, FID, CLS), Time to Interactive, and error rates. Unlike synthetic monitoring, RUM shows real-world conditions: slow networks, old devices, geographic latency. Sample data to control costs; segment by user type, device, and geography.

---

## How to Explain Like a Senior Developer

"RUM tells you what your REAL users actually experience. Synthetic tests run from controlled environments, but real users have slow phones, congested networks, and ad blockers. RUM captures Core Web Vitals (LCP for load speed, FID for interactivity, CLS for visual stability), JavaScript errors, API latency, and user flows. You inject a script that reports metrics back to your analytics service. The key is segmentation - your p50 might look good, but users on 3G in India might have terrible experience. Use RUM to find performance regressions, identify problematic user segments, and correlate performance with business metrics like conversion rates."

---

## Core Implementation

### Basic RUM Setup

```typescript
// rum/client.ts

interface PerformanceMetrics {
  // Navigation Timing
  dns: number;
  tcp: number;
  ttfb: number;        // Time to First Byte
  domLoad: number;
  fullLoad: number;
  
  // Core Web Vitals
  lcp?: number;        // Largest Contentful Paint
  fid?: number;        // First Input Delay
  cls?: number;        // Cumulative Layout Shift
  inp?: number;        // Interaction to Next Paint (replacing FID)
  
  // Custom metrics
  tti?: number;        // Time to Interactive
  fcp?: number;        // First Contentful Paint
}

interface RUMEvent {
  type: 'pageview' | 'performance' | 'error' | 'interaction' | 'api_call';
  timestamp: number;
  sessionId: string;
  userId?: string;
  url: string;
  data: object;
}

class RUMClient {
  private sessionId: string;
  private userId?: string;
  private buffer: RUMEvent[] = [];
  private flushInterval: number;
  private endpoint: string;
  
  constructor(config: { endpoint: string; flushIntervalMs?: number }) {
    this.sessionId = this.generateSessionId();
    this.endpoint = config.endpoint;
    this.flushInterval = config.flushIntervalMs || 5000;
    
    this.init();
  }
  
  private generateSessionId(): string {
    return `${Date.now()}-${Math.random().toString(36).slice(2)}`;
  }
  
  private init(): void {
    // Track page views
    this.trackPageView();
    
    // Track performance metrics
    this.observePerformance();
    
    // Track errors
    this.setupErrorTracking();
    
    // Track interactions
    this.observeInteractions();
    
    // Flush buffer periodically
    setInterval(() => this.flush(), this.flushInterval);
    
    // Flush on page unload
    window.addEventListener('visibilitychange', () => {
      if (document.visibilityState === 'hidden') {
        this.flush();
      }
    });
  }
  
  setUser(userId: string): void {
    this.userId = userId;
  }
  
  private trackPageView(): void {
    this.addEvent({
      type: 'pageview',
      timestamp: Date.now(),
      sessionId: this.sessionId,
      userId: this.userId,
      url: window.location.href,
      data: {
        referrer: document.referrer,
        userAgent: navigator.userAgent,
        screenWidth: window.screen.width,
        screenHeight: window.screen.height,
        devicePixelRatio: window.devicePixelRatio,
        connection: (navigator as any).connection?.effectiveType,
      },
    });
  }
  
  private observePerformance(): void {
    // Navigation timing
    window.addEventListener('load', () => {
      setTimeout(() => {
        const timing = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
        
        this.addEvent({
          type: 'performance',
          timestamp: Date.now(),
          sessionId: this.sessionId,
          userId: this.userId,
          url: window.location.href,
          data: {
            metric: 'navigation',
            dns: timing.domainLookupEnd - timing.domainLookupStart,
            tcp: timing.connectEnd - timing.connectStart,
            ttfb: timing.responseStart - timing.requestStart,
            domLoad: timing.domContentLoadedEventEnd - timing.fetchStart,
            fullLoad: timing.loadEventEnd - timing.fetchStart,
          },
        });
      }, 0);
    });
    
    // Core Web Vitals using web-vitals library
    this.observeCoreWebVitals();
  }
  
  private observeCoreWebVitals(): void {
    // LCP - Largest Contentful Paint
    const lcpObserver = new PerformanceObserver((list) => {
      const entries = list.getEntries();
      const lastEntry = entries[entries.length - 1] as any;
      
      this.addEvent({
        type: 'performance',
        timestamp: Date.now(),
        sessionId: this.sessionId,
        userId: this.userId,
        url: window.location.href,
        data: {
          metric: 'lcp',
          value: lastEntry.renderTime || lastEntry.loadTime,
          element: lastEntry.element?.tagName,
        },
      });
    });
    lcpObserver.observe({ type: 'largest-contentful-paint', buffered: true });
    
    // FID - First Input Delay
    const fidObserver = new PerformanceObserver((list) => {
      const firstInput = list.getEntries()[0] as any;
      
      this.addEvent({
        type: 'performance',
        timestamp: Date.now(),
        sessionId: this.sessionId,
        userId: this.userId,
        url: window.location.href,
        data: {
          metric: 'fid',
          value: firstInput.processingStart - firstInput.startTime,
          eventType: firstInput.name,
        },
      });
    });
    fidObserver.observe({ type: 'first-input', buffered: true });
    
    // CLS - Cumulative Layout Shift
    let clsValue = 0;
    const clsObserver = new PerformanceObserver((list) => {
      for (const entry of list.getEntries() as any[]) {
        if (!entry.hadRecentInput) {
          clsValue += entry.value;
        }
      }
    });
    clsObserver.observe({ type: 'layout-shift', buffered: true });
    
    // Report CLS on page hide
    window.addEventListener('visibilitychange', () => {
      if (document.visibilityState === 'hidden') {
        this.addEvent({
          type: 'performance',
          timestamp: Date.now(),
          sessionId: this.sessionId,
          userId: this.userId,
          url: window.location.href,
          data: {
            metric: 'cls',
            value: clsValue,
          },
        });
      }
    });
  }
  
  private setupErrorTracking(): void {
    window.addEventListener('error', (event) => {
      this.addEvent({
        type: 'error',
        timestamp: Date.now(),
        sessionId: this.sessionId,
        userId: this.userId,
        url: window.location.href,
        data: {
          message: event.message,
          filename: event.filename,
          lineno: event.lineno,
          colno: event.colno,
          stack: event.error?.stack,
        },
      });
    });
    
    window.addEventListener('unhandledrejection', (event) => {
      this.addEvent({
        type: 'error',
        timestamp: Date.now(),
        sessionId: this.sessionId,
        userId: this.userId,
        url: window.location.href,
        data: {
          message: event.reason?.message || String(event.reason),
          stack: event.reason?.stack,
          type: 'unhandledrejection',
        },
      });
    });
  }
  
  private observeInteractions(): void {
    // Track clicks on interactive elements
    document.addEventListener('click', (event) => {
      const target = event.target as HTMLElement;
      
      if (target.matches('button, a, [role="button"], input[type="submit"]')) {
        this.addEvent({
          type: 'interaction',
          timestamp: Date.now(),
          sessionId: this.sessionId,
          userId: this.userId,
          url: window.location.href,
          data: {
            action: 'click',
            element: target.tagName,
            text: target.textContent?.slice(0, 50),
            id: target.id,
            className: target.className,
          },
        });
      }
    });
  }
  
  // Track API calls
  trackAPICall(
    url: string,
    method: string,
    duration: number,
    status: number,
    success: boolean
  ): void {
    this.addEvent({
      type: 'api_call',
      timestamp: Date.now(),
      sessionId: this.sessionId,
      userId: this.userId,
      url: window.location.href,
      data: {
        apiUrl: url,
        method,
        duration,
        status,
        success,
      },
    });
  }
  
  // Custom event tracking
  trackCustomEvent(name: string, data: object): void {
    this.addEvent({
      type: 'interaction',
      timestamp: Date.now(),
      sessionId: this.sessionId,
      userId: this.userId,
      url: window.location.href,
      data: {
        action: 'custom',
        name,
        ...data,
      },
    });
  }
  
  private addEvent(event: RUMEvent): void {
    this.buffer.push(event);
    
    // Flush if buffer is getting large
    if (this.buffer.length >= 10) {
      this.flush();
    }
  }
  
  private async flush(): Promise<void> {
    if (this.buffer.length === 0) return;
    
    const events = [...this.buffer];
    this.buffer = [];
    
    try {
      // Use sendBeacon for reliability on page unload
      const blob = new Blob([JSON.stringify(events)], { type: 'application/json' });
      navigator.sendBeacon(this.endpoint, blob);
    } catch (error) {
      // Fallback to fetch
      fetch(this.endpoint, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(events),
        keepalive: true,
      }).catch(() => {
        // Re-add events to buffer on failure
        this.buffer.unshift(...events);
      });
    }
  }
}

// Initialize
const rum = new RUMClient({
  endpoint: '/api/rum/events',
  flushIntervalMs: 5000,
});

export { rum, RUMClient };
```

### Fetch/XHR Interceptor for API Tracking

```typescript
// rum/api-interceptor.ts
import { rum } from './client';

// Intercept fetch
const originalFetch = window.fetch;

window.fetch = async function(...args) {
  const url = typeof args[0] === 'string' ? args[0] : args[0].url;
  const method = args[1]?.method || 'GET';
  const start = performance.now();
  
  try {
    const response = await originalFetch.apply(this, args);
    const duration = performance.now() - start;
    
    rum.trackAPICall(url, method, duration, response.status, response.ok);
    
    return response;
  } catch (error) {
    const duration = performance.now() - start;
    rum.trackAPICall(url, method, duration, 0, false);
    throw error;
  }
};

// Intercept XMLHttpRequest
const originalXHROpen = XMLHttpRequest.prototype.open;
const originalXHRSend = XMLHttpRequest.prototype.send;

XMLHttpRequest.prototype.open = function(method: string, url: string, ...rest: any[]) {
  (this as any)._rumMethod = method;
  (this as any)._rumUrl = url;
  return originalXHROpen.apply(this, [method, url, ...rest] as any);
};

XMLHttpRequest.prototype.send = function(...args) {
  const start = performance.now();
  const method = (this as any)._rumMethod;
  const url = (this as any)._rumUrl;
  
  this.addEventListener('loadend', () => {
    const duration = performance.now() - start;
    rum.trackAPICall(url, method, duration, this.status, this.status >= 200 && this.status < 400);
  });
  
  return originalXHRSend.apply(this, args);
};
```

### Server-Side RUM Endpoint

```typescript
// api/rum/events.ts
import { Request, Response } from 'express';
import { logger } from '../../logging/structured-logger';

interface RUMEvent {
  type: string;
  timestamp: number;
  sessionId: string;
  userId?: string;
  url: string;
  data: object;
}

// Store events (in production, send to analytics service)
export async function handleRUMEvents(req: Request, res: Response): Promise<void> {
  const events: RUMEvent[] = req.body;
  
  for (const event of events) {
    // Enrich with server-side data
    const enrichedEvent = {
      ...event,
      receivedAt: Date.now(),
      clientIP: req.ip,
      country: req.headers['cf-ipcountry'], // Cloudflare
    };
    
    // Log for analysis (or send to analytics service)
    logger.info('RUM event', enrichedEvent);
    
    // Forward to analytics service
    // await analyticsService.track(enrichedEvent);
  }
  
  res.status(204).send();
}

// Aggregation for dashboards
interface AggregatedMetrics {
  period: string;
  pageviews: number;
  uniqueSessions: number;
  avgLCP: number;
  avgFID: number;
  avgCLS: number;
  p75LCP: number;
  p75FID: number;
  p75CLS: number;
  errorCount: number;
  errorRate: number;
}

export function aggregateMetrics(events: RUMEvent[]): AggregatedMetrics {
  const lcpValues: number[] = [];
  const fidValues: number[] = [];
  const clsValues: number[] = [];
  const sessions = new Set<string>();
  let errorCount = 0;
  
  for (const event of events) {
    sessions.add(event.sessionId);
    
    if (event.type === 'performance') {
      const data = event.data as any;
      if (data.metric === 'lcp') lcpValues.push(data.value);
      if (data.metric === 'fid') fidValues.push(data.value);
      if (data.metric === 'cls') clsValues.push(data.value);
    }
    
    if (event.type === 'error') {
      errorCount++;
    }
  }
  
  const avg = (arr: number[]) => arr.length ? arr.reduce((a, b) => a + b, 0) / arr.length : 0;
  const percentile = (arr: number[], p: number) => {
    const sorted = [...arr].sort((a, b) => a - b);
    const index = Math.ceil((p / 100) * sorted.length) - 1;
    return sorted[index] || 0;
  };
  
  return {
    period: new Date().toISOString(),
    pageviews: events.filter(e => e.type === 'pageview').length,
    uniqueSessions: sessions.size,
    avgLCP: avg(lcpValues),
    avgFID: avg(fidValues),
    avgCLS: avg(clsValues),
    p75LCP: percentile(lcpValues, 75),
    p75FID: percentile(fidValues, 75),
    p75CLS: percentile(clsValues, 75),
    errorCount,
    errorRate: errorCount / Math.max(sessions.size, 1),
  };
}
```

### React Integration

```typescript
// rum/react-integration.tsx
import React, { useEffect, useRef } from 'react';
import { rum } from './client';

// Track component render performance
export function useRenderTracking(componentName: string): void {
  const renderCount = useRef(0);
  const firstRender = useRef(performance.now());
  
  useEffect(() => {
    renderCount.current++;
    
    // Track first render
    if (renderCount.current === 1) {
      rum.trackCustomEvent('component_first_render', {
        component: componentName,
        duration: performance.now() - firstRender.current,
      });
    }
  });
}

// Track user journey/funnel
export function useJourneyTracking(stepName: string): void {
  useEffect(() => {
    rum.trackCustomEvent('journey_step', {
      step: stepName,
      timestamp: Date.now(),
    });
  }, [stepName]);
}

// Error boundary with RUM
interface Props {
  children: React.ReactNode;
  fallback: React.ReactNode;
}

interface State {
  hasError: boolean;
}

export class RUMErrorBoundary extends React.Component<Props, State> {
  state = { hasError: false };
  
  static getDerivedStateFromError(): State {
    return { hasError: true };
  }
  
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo): void {
    rum.trackCustomEvent('react_error_boundary', {
      error: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
    });
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}

// Performance timing HOC
export function withPerformanceTracking<P extends object>(
  WrappedComponent: React.ComponentType<P>,
  componentName: string
): React.FC<P> {
  return function PerformanceTrackedComponent(props: P) {
    useRenderTracking(componentName);
    return <WrappedComponent {...props} />;
  };
}

// Usage
function CheckoutPage() {
  useJourneyTracking('checkout_started');
  useRenderTracking('CheckoutPage');
  
  return (
    <div>
      <h1>Checkout</h1>
      {/* ... */}
    </div>
  );
}
```

---

## Real-World Scenarios

### Scenario 1: Identifying Performance Issues by Segment

```typescript
// analysis/segmentation.ts

interface SegmentedMetrics {
  segment: string;
  count: number;
  avgLCP: number;
  avgFID: number;
  avgCLS: number;
  errorRate: number;
}

function analyzeBySegment(
  events: any[],
  segmentKey: string
): SegmentedMetrics[] {
  const segments: Map<string, any[]> = new Map();
  
  // Group by segment
  for (const event of events) {
    const segmentValue = event.data[segmentKey] || 'unknown';
    if (!segments.has(segmentValue)) {
      segments.set(segmentValue, []);
    }
    segments.get(segmentValue)!.push(event);
  }
  
  // Calculate metrics per segment
  const results: SegmentedMetrics[] = [];
  
  for (const [segment, segmentEvents] of segments) {
    const lcpValues = segmentEvents
      .filter(e => e.data.metric === 'lcp')
      .map(e => e.data.value);
    
    const errorEvents = segmentEvents.filter(e => e.type === 'error');
    
    results.push({
      segment,
      count: segmentEvents.length,
      avgLCP: average(lcpValues),
      avgFID: 0, // Similar calculation
      avgCLS: 0,
      errorRate: errorEvents.length / segmentEvents.length,
    });
  }
  
  return results.sort((a, b) => b.avgLCP - a.avgLCP);
}

function average(arr: number[]): number {
  return arr.length ? arr.reduce((a, b) => a + b, 0) / arr.length : 0;
}

// Example: Find worst performing segments
const byDevice = analyzeBySegment(events, 'deviceType');
// Result: [{ segment: 'mobile', avgLCP: 4500 }, { segment: 'desktop', avgLCP: 2100 }]

const byCountry = analyzeBySegment(events, 'country');
// Result: [{ segment: 'IN', avgLCP: 5200 }, { segment: 'US', avgLCP: 1800 }]

const byConnection = analyzeBySegment(events, 'connection');
// Result: [{ segment: '3g', avgLCP: 6000 }, { segment: '4g', avgLCP: 2500 }]

const events: any[] = [];
```

### Scenario 2: Correlating Performance with Business Metrics

```typescript
// analysis/business-correlation.ts

interface ConversionData {
  sessionId: string;
  converted: boolean;
  lcp: number;
  fid: number;
}

function analyzePerformanceConversionCorrelation(
  data: ConversionData[]
): object {
  // Group by LCP buckets
  const lcpBuckets = [
    { name: 'fast', max: 2500, sessions: [] as ConversionData[] },
    { name: 'moderate', max: 4000, sessions: [] as ConversionData[] },
    { name: 'slow', max: Infinity, sessions: [] as ConversionData[] },
  ];
  
  for (const session of data) {
    const bucket = lcpBuckets.find(b => session.lcp <= b.max)!;
    bucket.sessions.push(session);
  }
  
  const analysis = lcpBuckets.map(bucket => ({
    lcpRange: bucket.name,
    sessions: bucket.sessions.length,
    conversions: bucket.sessions.filter(s => s.converted).length,
    conversionRate: (
      bucket.sessions.filter(s => s.converted).length / 
      bucket.sessions.length * 100
    ).toFixed(2) + '%',
  }));
  
  return {
    analysis,
    insight: `Sessions with LCP < 2.5s have ${
      analysis[0].conversionRate
    } conversion rate vs ${
      analysis[2].conversionRate
    } for slow sessions`,
  };
}

// Example output:
// {
//   analysis: [
//     { lcpRange: 'fast', sessions: 5000, conversionRate: '4.2%' },
//     { lcpRange: 'moderate', sessions: 3000, conversionRate: '2.8%' },
//     { lcpRange: 'slow', sessions: 2000, conversionRate: '1.5%' }
//   ],
//   insight: 'Sessions with LCP < 2.5s have 4.2% conversion rate vs 1.5% for slow sessions'
// }
```

---

## Common Pitfalls

### 1. Not Sampling High-Traffic Sites

```typescript
// ❌ BAD: Track every user
const rum = new RUMClient({ endpoint: '/api/rum' }); // Millions of events!

// ✅ GOOD: Sample intelligently
const SAMPLE_RATE = 0.1; // 10%

const shouldTrack = Math.random() < SAMPLE_RATE || isImportantUser();

if (shouldTrack) {
  const rum = new RUMClient({ endpoint: '/api/rum' });
}

function isImportantUser(): boolean {
  return false; // Check if premium user, beta tester, etc.
}
```

### 2. Missing Context in Error Reports

```typescript
// ❌ BAD: Just the error
rum.trackError(error.message);

// ✅ GOOD: Rich context
rum.trackCustomEvent('error', {
  message: error.message,
  stack: error.stack,
  url: window.location.href,
  userAction: lastUserAction,
  apiCallsInFlight: pendingAPICalls,
  sessionDuration: Date.now() - sessionStart,
});

function trackError(msg: string): void {}
const error = new Error();
let lastUserAction = '';
let pendingAPICalls = 0;
let sessionStart = Date.now();
```

### 3. Not Tracking User Flows

```typescript
// ❌ BAD: Only page views
rum.trackPageView('/checkout');

// ✅ GOOD: Full funnel tracking
rum.trackCustomEvent('funnel', { step: 'cart_viewed', products: 3 });
rum.trackCustomEvent('funnel', { step: 'checkout_started' });
rum.trackCustomEvent('funnel', { step: 'payment_entered' });
rum.trackCustomEvent('funnel', { step: 'order_completed', value: 99.99 });

function trackPageView(path: string): void {}
```

---

## Interview Questions

### Q1: What are Core Web Vitals and why do they matter?

**A:** LCP (Largest Contentful Paint) measures loading - how fast the main content appears. FID (First Input Delay) / INP measures interactivity - how fast the page responds to user input. CLS (Cumulative Layout Shift) measures visual stability - do elements jump around? They matter because Google uses them for search ranking, and they correlate with user engagement and conversion rates.

### Q2: How is RUM different from synthetic monitoring?

**A:** RUM captures data from real users on real devices with real network conditions. Synthetic runs controlled tests from specific locations. RUM shows actual user experience distribution; synthetic ensures baseline functionality. Use both: synthetic for regression detection and uptime, RUM for real user experience insights.

### Q3: How do you handle RUM data at scale?

**A:** Sample traffic (track 10% of sessions), aggregate on the client before sending, batch events, use sendBeacon for reliable delivery, store aggregated metrics not raw events, set retention policies, and use specialized time-series databases.

### Q4: How do you identify which users are having a bad experience?

**A:** Segment by device type, browser, geography, network type. Look at percentiles (p75, p90) not just averages. Set up alerting on metric thresholds. Correlate with user attributes (plan type, account age). Build dashboards showing worst-performing segments.

---

## Quick Reference Checklist

### Implementation
- [ ] Track Core Web Vitals (LCP, FID/INP, CLS)
- [ ] Track JavaScript errors
- [ ] Track API call performance
- [ ] Include session and user identifiers
- [ ] Use sendBeacon for reliable delivery

### Data Collection
- [ ] Sample at high traffic
- [ ] Batch events before sending
- [ ] Include device/network context
- [ ] Track user journeys/funnels
- [ ] Respect privacy (no PII in URLs)

### Analysis
- [ ] Segment by device, geography, network
- [ ] Look at percentiles, not just averages
- [ ] Correlate with business metrics
- [ ] Alert on threshold breaches
- [ ] Track trends over time

### Core Web Vitals Targets
- [ ] LCP: < 2.5 seconds (good), < 4s (needs improvement)
- [ ] FID: < 100ms (good), < 300ms (needs improvement)
- [ ] CLS: < 0.1 (good), < 0.25 (needs improvement)

---

*Last updated: February 2026*

