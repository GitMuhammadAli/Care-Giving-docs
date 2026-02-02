# Synthetic Monitoring - Complete Guide

> **MUST REMEMBER**: Synthetic monitoring runs automated tests from controlled locations to verify availability, performance, and functionality 24/7. Use it for uptime checks, critical user flows, API health, SSL certificate monitoring, and performance baselines. It catches issues before users do. Combine with RUM - synthetic tells you IF it works, RUM tells you HOW users experience it.

---

## How to Explain Like a Senior Developer

"Synthetic monitoring is your proactive watchdog. Instead of waiting for users to report issues, you simulate user interactions from multiple locations around the world, continuously. Basic synthetic checks verify your site responds (uptime monitoring). Advanced checks walk through critical flows like login → add to cart → checkout. You can test from different geographies to catch regional issues, verify SSL certificates before they expire, and establish performance baselines. The key insight: synthetic monitoring catches problems during low-traffic hours when there are no real users to generate RUM data. Use it for SLA measurement, regression detection, and as the first line of defense in incident detection."

---

## Core Implementation

### Basic Uptime Monitoring

```typescript
// synthetic/uptime-checker.ts
import axios from 'axios';
import { logger } from '../logging/structured-logger';

interface UptimeCheck {
  name: string;
  url: string;
  method: 'GET' | 'POST' | 'HEAD';
  expectedStatus: number;
  timeoutMs: number;
  intervalMs: number;
  headers?: Record<string, string>;
  body?: object;
  assertions?: Array<{
    type: 'status' | 'responseTime' | 'bodyContains' | 'headerContains';
    value: any;
  }>;
}

interface CheckResult {
  check: UptimeCheck;
  success: boolean;
  statusCode?: number;
  responseTimeMs: number;
  timestamp: Date;
  error?: string;
  assertions: Array<{ name: string; passed: boolean; details?: string }>;
}

const uptimeChecks: UptimeCheck[] = [
  {
    name: 'API Health',
    url: 'https://api.example.com/health',
    method: 'GET',
    expectedStatus: 200,
    timeoutMs: 5000,
    intervalMs: 60000, // Every minute
    assertions: [
      { type: 'responseTime', value: 2000 },
      { type: 'bodyContains', value: '"status":"healthy"' },
    ],
  },
  {
    name: 'Homepage',
    url: 'https://example.com',
    method: 'GET',
    expectedStatus: 200,
    timeoutMs: 10000,
    intervalMs: 60000,
    assertions: [
      { type: 'responseTime', value: 3000 },
      { type: 'bodyContains', value: '<title>' },
    ],
  },
  {
    name: 'Login API',
    url: 'https://api.example.com/auth/login',
    method: 'POST',
    expectedStatus: 401, // Should reject invalid credentials
    timeoutMs: 5000,
    intervalMs: 300000, // Every 5 minutes
    headers: { 'Content-Type': 'application/json' },
    body: { email: 'synthetic@test.com', password: 'invalid' },
  },
];

async function runCheck(check: UptimeCheck): Promise<CheckResult> {
  const start = Date.now();
  const assertions: CheckResult['assertions'] = [];
  
  try {
    const response = await axios({
      method: check.method,
      url: check.url,
      timeout: check.timeoutMs,
      headers: check.headers,
      data: check.body,
      validateStatus: () => true, // Don't throw on any status
    });
    
    const responseTimeMs = Date.now() - start;
    
    // Run assertions
    if (check.assertions) {
      for (const assertion of check.assertions) {
        switch (assertion.type) {
          case 'status':
            assertions.push({
              name: `Status is ${assertion.value}`,
              passed: response.status === assertion.value,
              details: `Got ${response.status}`,
            });
            break;
            
          case 'responseTime':
            assertions.push({
              name: `Response time < ${assertion.value}ms`,
              passed: responseTimeMs < assertion.value,
              details: `Got ${responseTimeMs}ms`,
            });
            break;
            
          case 'bodyContains':
            const bodyString = typeof response.data === 'string' 
              ? response.data 
              : JSON.stringify(response.data);
            assertions.push({
              name: `Body contains "${assertion.value}"`,
              passed: bodyString.includes(assertion.value),
            });
            break;
            
          case 'headerContains':
            const [headerName, headerValue] = assertion.value.split(':');
            assertions.push({
              name: `Header ${headerName} contains ${headerValue}`,
              passed: response.headers[headerName.toLowerCase()]?.includes(headerValue),
            });
            break;
        }
      }
    }
    
    // Check expected status
    const statusMatch = response.status === check.expectedStatus;
    assertions.push({
      name: `Status is ${check.expectedStatus}`,
      passed: statusMatch,
      details: `Got ${response.status}`,
    });
    
    const allPassed = assertions.every(a => a.passed);
    
    return {
      check,
      success: allPassed,
      statusCode: response.status,
      responseTimeMs,
      timestamp: new Date(),
      assertions,
    };
    
  } catch (error: any) {
    return {
      check,
      success: false,
      responseTimeMs: Date.now() - start,
      timestamp: new Date(),
      error: error.message,
      assertions: [{
        name: 'Request succeeded',
        passed: false,
        details: error.message,
      }],
    };
  }
}

// Run all checks and handle results
async function runAllChecks(): Promise<void> {
  for (const check of uptimeChecks) {
    const result = await runCheck(check);
    
    // Log result
    logger.info('Synthetic check completed', {
      checkName: check.name,
      success: result.success,
      statusCode: result.statusCode,
      responseTimeMs: result.responseTimeMs,
      failedAssertions: result.assertions.filter(a => !a.passed),
    });
    
    // Alert on failure
    if (!result.success) {
      await sendAlert(result);
    }
    
    // Record metrics
    await recordMetrics(result);
  }
}

async function sendAlert(result: CheckResult): Promise<void> {
  // Integration with alerting system
}

async function recordMetrics(result: CheckResult): Promise<void> {
  // Send to metrics system
}

export { runCheck, runAllChecks, UptimeCheck };
```

### Browser-Based Synthetic Tests (Playwright)

```typescript
// synthetic/browser-tests.ts
import { chromium, Browser, Page } from 'playwright';
import { logger } from '../logging/structured-logger';

interface BrowserTestResult {
  testName: string;
  success: boolean;
  durationMs: number;
  steps: Array<{
    name: string;
    durationMs: number;
    success: boolean;
    screenshot?: string;
    error?: string;
  }>;
  metrics?: {
    lcp?: number;
    fcp?: number;
    ttfb?: number;
  };
}

class SyntheticBrowserTest {
  private browser: Browser | null = null;
  
  async setup(): Promise<void> {
    this.browser = await chromium.launch({
      headless: true,
    });
  }
  
  async teardown(): Promise<void> {
    await this.browser?.close();
  }
  
  async runTest(
    testName: string,
    testFn: (page: Page) => Promise<void>
  ): Promise<BrowserTestResult> {
    const start = Date.now();
    const steps: BrowserTestResult['steps'] = [];
    
    const context = await this.browser!.newContext({
      viewport: { width: 1920, height: 1080 },
      userAgent: 'SyntheticMonitor/1.0',
    });
    
    const page = await context.newPage();
    
    // Collect performance metrics
    let metrics: BrowserTestResult['metrics'] = {};
    
    page.on('load', async () => {
      try {
        const perfMetrics = await page.evaluate(() => {
          const paint = performance.getEntriesByType('paint');
          const navigation = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
          
          return {
            fcp: paint.find(p => p.name === 'first-contentful-paint')?.startTime,
            ttfb: navigation?.responseStart,
          };
        });
        metrics = { ...metrics, ...perfMetrics };
      } catch (e) {
        // Ignore errors collecting metrics
      }
    });
    
    try {
      await testFn(page);
      
      return {
        testName,
        success: true,
        durationMs: Date.now() - start,
        steps,
        metrics,
      };
    } catch (error: any) {
      // Take screenshot on failure
      const screenshot = await page.screenshot({ 
        path: `/tmp/synthetic-failure-${Date.now()}.png`,
        fullPage: true,
      });
      
      return {
        testName,
        success: false,
        durationMs: Date.now() - start,
        steps: [...steps, {
          name: 'Test failed',
          durationMs: 0,
          success: false,
          error: error.message,
          screenshot: screenshot.toString('base64'),
        }],
        metrics,
      };
    } finally {
      await context.close();
    }
  }
}

// Example: Login flow test
async function loginFlowTest(page: Page): Promise<void> {
  // Navigate to login page
  await page.goto('https://example.com/login');
  await page.waitForLoadState('networkidle');
  
  // Fill login form
  await page.fill('input[name="email"]', 'synthetic@test.com');
  await page.fill('input[name="password"]', process.env.SYNTHETIC_TEST_PASSWORD!);
  
  // Click login button
  await page.click('button[type="submit"]');
  
  // Wait for redirect to dashboard
  await page.waitForURL('**/dashboard', { timeout: 10000 });
  
  // Verify dashboard loaded
  await page.waitForSelector('h1:has-text("Dashboard")', { timeout: 5000 });
}

// Example: Checkout flow test
async function checkoutFlowTest(page: Page): Promise<void> {
  // Navigate to product page
  await page.goto('https://example.com/products/test-product');
  await page.waitForLoadState('networkidle');
  
  // Add to cart
  await page.click('button:has-text("Add to Cart")');
  await page.waitForSelector('.cart-count:has-text("1")');
  
  // Go to cart
  await page.click('a[href="/cart"]');
  await page.waitForURL('**/cart');
  
  // Proceed to checkout
  await page.click('button:has-text("Checkout")');
  await page.waitForURL('**/checkout');
  
  // Verify checkout page loaded
  await page.waitForSelector('form[data-testid="checkout-form"]');
}

// Run synthetic browser tests
async function runBrowserTests(): Promise<void> {
  const tester = new SyntheticBrowserTest();
  await tester.setup();
  
  const tests = [
    { name: 'Login Flow', fn: loginFlowTest },
    { name: 'Checkout Flow', fn: checkoutFlowTest },
  ];
  
  for (const test of tests) {
    const result = await tester.runTest(test.name, test.fn);
    
    logger.info('Browser synthetic test completed', {
      testName: result.testName,
      success: result.success,
      durationMs: result.durationMs,
      metrics: result.metrics,
    });
    
    if (!result.success) {
      // Alert on failure
      logger.error('Synthetic test failed', {
        testName: result.testName,
        steps: result.steps,
      });
    }
  }
  
  await tester.teardown();
}

export { SyntheticBrowserTest, runBrowserTests };
```

### Multi-Location Monitoring

```typescript
// synthetic/multi-location.ts

interface MonitoringLocation {
  id: string;
  name: string;
  region: string;
  endpoint: string; // Worker endpoint in that region
}

const locations: MonitoringLocation[] = [
  { id: 'us-east', name: 'US East (Virginia)', region: 'us-east-1', endpoint: 'https://us-east.monitors.example.com' },
  { id: 'us-west', name: 'US West (Oregon)', region: 'us-west-2', endpoint: 'https://us-west.monitors.example.com' },
  { id: 'eu-west', name: 'Europe (Ireland)', region: 'eu-west-1', endpoint: 'https://eu-west.monitors.example.com' },
  { id: 'ap-south', name: 'Asia Pacific (Mumbai)', region: 'ap-south-1', endpoint: 'https://ap-south.monitors.example.com' },
  { id: 'ap-northeast', name: 'Asia Pacific (Tokyo)', region: 'ap-northeast-1', endpoint: 'https://ap-northeast.monitors.example.com' },
];

interface MultiLocationResult {
  checkName: string;
  timestamp: Date;
  results: Array<{
    location: MonitoringLocation;
    success: boolean;
    responseTimeMs: number;
    error?: string;
  }>;
  summary: {
    allPassed: boolean;
    avgResponseTime: number;
    slowestLocation: string;
    fastestLocation: string;
    failedLocations: string[];
  };
}

async function runFromLocation(
  location: MonitoringLocation,
  check: { url: string; timeoutMs: number }
): Promise<{ success: boolean; responseTimeMs: number; error?: string }> {
  try {
    // Call the monitoring worker in that region
    const response = await fetch(`${location.endpoint}/check`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(check),
    });
    
    const result = await response.json();
    return result;
  } catch (error: any) {
    return {
      success: false,
      responseTimeMs: 0,
      error: error.message,
    };
  }
}

async function runMultiLocationCheck(
  checkName: string,
  url: string
): Promise<MultiLocationResult> {
  const check = { url, timeoutMs: 10000 };
  
  // Run from all locations in parallel
  const results = await Promise.all(
    locations.map(async (location) => ({
      location,
      ...await runFromLocation(location, check),
    }))
  );
  
  // Calculate summary
  const successfulResults = results.filter(r => r.success);
  const avgResponseTime = successfulResults.length
    ? successfulResults.reduce((sum, r) => sum + r.responseTimeMs, 0) / successfulResults.length
    : 0;
  
  const sortedByTime = [...successfulResults].sort((a, b) => a.responseTimeMs - b.responseTimeMs);
  
  return {
    checkName,
    timestamp: new Date(),
    results,
    summary: {
      allPassed: results.every(r => r.success),
      avgResponseTime,
      slowestLocation: sortedByTime[sortedByTime.length - 1]?.location.name || 'N/A',
      fastestLocation: sortedByTime[0]?.location.name || 'N/A',
      failedLocations: results.filter(r => !r.success).map(r => r.location.name),
    },
  };
}

export { runMultiLocationCheck, locations };
```

### SSL Certificate Monitoring

```typescript
// synthetic/ssl-monitor.ts
import * as tls from 'tls';
import { URL } from 'url';

interface SSLCertInfo {
  domain: string;
  issuer: string;
  validFrom: Date;
  validTo: Date;
  daysUntilExpiry: number;
  isValid: boolean;
  errors: string[];
}

async function checkSSLCertificate(domain: string): Promise<SSLCertInfo> {
  return new Promise((resolve, reject) => {
    const options = {
      host: domain,
      port: 443,
      servername: domain,
      rejectUnauthorized: false, // We want to inspect even invalid certs
    };
    
    const socket = tls.connect(options, () => {
      const cert = socket.getPeerCertificate();
      const errors: string[] = [];
      
      // Check if authorized
      if (!socket.authorized) {
        errors.push(socket.authorizationError || 'Certificate not authorized');
      }
      
      const validFrom = new Date(cert.valid_from);
      const validTo = new Date(cert.valid_to);
      const now = new Date();
      
      // Check validity period
      if (now < validFrom) {
        errors.push('Certificate not yet valid');
      }
      if (now > validTo) {
        errors.push('Certificate has expired');
      }
      
      const daysUntilExpiry = Math.floor(
        (validTo.getTime() - now.getTime()) / (1000 * 60 * 60 * 24)
      );
      
      socket.end();
      
      resolve({
        domain,
        issuer: cert.issuer?.O || 'Unknown',
        validFrom,
        validTo,
        daysUntilExpiry,
        isValid: errors.length === 0,
        errors,
      });
    });
    
    socket.on('error', (err) => {
      reject(err);
    });
    
    socket.setTimeout(10000, () => {
      socket.destroy();
      reject(new Error('Connection timeout'));
    });
  });
}

// Monitor multiple domains
async function monitorSSLCertificates(
  domains: string[],
  warningDays: number = 30,
  criticalDays: number = 7
): Promise<{
  results: SSLCertInfo[];
  warnings: SSLCertInfo[];
  critical: SSLCertInfo[];
}> {
  const results = await Promise.all(
    domains.map(domain => checkSSLCertificate(domain).catch(err => ({
      domain,
      issuer: 'Unknown',
      validFrom: new Date(),
      validTo: new Date(),
      daysUntilExpiry: -1,
      isValid: false,
      errors: [err.message],
    })))
  );
  
  const warnings = results.filter(r => 
    r.daysUntilExpiry > criticalDays && r.daysUntilExpiry <= warningDays
  );
  
  const critical = results.filter(r => 
    r.daysUntilExpiry <= criticalDays || !r.isValid
  );
  
  return { results, warnings, critical };
}

// Alert on expiring certificates
async function alertOnExpiringCerts(): Promise<void> {
  const domains = [
    'api.example.com',
    'www.example.com',
    'admin.example.com',
  ];
  
  const { warnings, critical } = await monitorSSLCertificates(domains);
  
  for (const cert of critical) {
    console.error('CRITICAL: SSL certificate issue', {
      domain: cert.domain,
      daysUntilExpiry: cert.daysUntilExpiry,
      errors: cert.errors,
    });
  }
  
  for (const cert of warnings) {
    console.warn('WARNING: SSL certificate expiring soon', {
      domain: cert.domain,
      daysUntilExpiry: cert.daysUntilExpiry,
      expiresOn: cert.validTo,
    });
  }
}

export { checkSSLCertificate, monitorSSLCertificates };
```

### API Contract Testing

```typescript
// synthetic/api-contract.ts
import Ajv from 'ajv';

interface ContractTest {
  name: string;
  endpoint: string;
  method: 'GET' | 'POST' | 'PUT' | 'DELETE';
  headers?: Record<string, string>;
  body?: object;
  expectedStatus: number;
  responseSchema: object;
}

const ajv = new Ajv();

const contractTests: ContractTest[] = [
  {
    name: 'Get User API',
    endpoint: 'https://api.example.com/users/1',
    method: 'GET',
    expectedStatus: 200,
    responseSchema: {
      type: 'object',
      required: ['id', 'email', 'name'],
      properties: {
        id: { type: 'number' },
        email: { type: 'string', format: 'email' },
        name: { type: 'string' },
        createdAt: { type: 'string', format: 'date-time' },
      },
    },
  },
  {
    name: 'List Products API',
    endpoint: 'https://api.example.com/products',
    method: 'GET',
    expectedStatus: 200,
    responseSchema: {
      type: 'object',
      required: ['data', 'pagination'],
      properties: {
        data: {
          type: 'array',
          items: {
            type: 'object',
            required: ['id', 'name', 'price'],
            properties: {
              id: { type: 'number' },
              name: { type: 'string' },
              price: { type: 'number' },
            },
          },
        },
        pagination: {
          type: 'object',
          required: ['page', 'totalPages'],
          properties: {
            page: { type: 'number' },
            totalPages: { type: 'number' },
          },
        },
      },
    },
  },
];

async function runContractTest(test: ContractTest): Promise<{
  passed: boolean;
  errors: string[];
}> {
  const errors: string[] = [];
  
  try {
    const response = await fetch(test.endpoint, {
      method: test.method,
      headers: {
        'Content-Type': 'application/json',
        ...test.headers,
      },
      body: test.body ? JSON.stringify(test.body) : undefined,
    });
    
    // Check status
    if (response.status !== test.expectedStatus) {
      errors.push(`Expected status ${test.expectedStatus}, got ${response.status}`);
    }
    
    // Validate response schema
    const data = await response.json();
    const validate = ajv.compile(test.responseSchema);
    const valid = validate(data);
    
    if (!valid) {
      errors.push(...(validate.errors?.map(e => 
        `Schema error: ${e.instancePath} ${e.message}`
      ) || []));
    }
    
  } catch (error: any) {
    errors.push(`Request failed: ${error.message}`);
  }
  
  return {
    passed: errors.length === 0,
    errors,
  };
}

// Run all contract tests
async function runContractTests(): Promise<void> {
  for (const test of contractTests) {
    const result = await runContractTest(test);
    
    if (result.passed) {
      console.log(`✓ ${test.name}`);
    } else {
      console.error(`✗ ${test.name}:`);
      result.errors.forEach(e => console.error(`  - ${e}`));
    }
  }
}

export { runContractTest, runContractTests };
```

---

## Real-World Scenarios

### Scenario 1: Setting Up Comprehensive Monitoring

```typescript
// synthetic/comprehensive-setup.ts

interface MonitoringPlan {
  uptimeChecks: Array<{
    name: string;
    url: string;
    interval: string;
    locations: string[];
  }>;
  browserTests: Array<{
    name: string;
    flow: string;
    interval: string;
    criticalPath: boolean;
  }>;
  sslChecks: Array<{
    domain: string;
    warningDays: number;
  }>;
  apiContracts: Array<{
    name: string;
    endpoint: string;
    interval: string;
  }>;
}

const monitoringPlan: MonitoringPlan = {
  uptimeChecks: [
    // Critical endpoints - check every minute from multiple locations
    { name: 'API Health', url: '/health', interval: '1m', locations: ['us-east', 'eu-west', 'ap-south'] },
    { name: 'Homepage', url: '/', interval: '1m', locations: ['us-east', 'eu-west'] },
    
    // Important endpoints - check every 5 minutes
    { name: 'Login Page', url: '/login', interval: '5m', locations: ['us-east'] },
    { name: 'Search API', url: '/api/search?q=test', interval: '5m', locations: ['us-east'] },
  ],
  
  browserTests: [
    // Critical user flows - test every 15 minutes
    { name: 'Login Flow', flow: 'login', interval: '15m', criticalPath: true },
    { name: 'Checkout Flow', flow: 'checkout', interval: '15m', criticalPath: true },
    
    // Important flows - test every hour
    { name: 'Search and Filter', flow: 'search', interval: '1h', criticalPath: false },
    { name: 'Account Settings', flow: 'settings', interval: '1h', criticalPath: false },
  ],
  
  sslChecks: [
    { domain: 'api.example.com', warningDays: 30 },
    { domain: 'www.example.com', warningDays: 30 },
    { domain: 'admin.example.com', warningDays: 30 },
  ],
  
  apiContracts: [
    { name: 'User API', endpoint: '/api/users', interval: '1h' },
    { name: 'Products API', endpoint: '/api/products', interval: '1h' },
    { name: 'Orders API', endpoint: '/api/orders', interval: '1h' },
  ],
};
```

### Scenario 2: Alerting Configuration

```typescript
// synthetic/alerting-config.ts

interface AlertConfig {
  check: string;
  conditions: Array<{
    metric: string;
    operator: '>' | '<' | '==' | '!=';
    threshold: number;
    duration?: string;
  }>;
  severity: 'critical' | 'warning' | 'info';
  notifications: string[];
  escalation?: {
    afterMinutes: number;
    to: string[];
  };
}

const alertConfigs: AlertConfig[] = [
  {
    check: 'API Health',
    conditions: [
      { metric: 'success', operator: '==', threshold: 0 },
    ],
    severity: 'critical',
    notifications: ['pagerduty', 'slack-incidents'],
    escalation: {
      afterMinutes: 5,
      to: ['engineering-manager'],
    },
  },
  {
    check: 'API Health',
    conditions: [
      { metric: 'responseTime', operator: '>', threshold: 2000, duration: '5m' },
    ],
    severity: 'warning',
    notifications: ['slack-alerts'],
  },
  {
    check: 'Login Flow',
    conditions: [
      { metric: 'success', operator: '==', threshold: 0 },
    ],
    severity: 'critical',
    notifications: ['pagerduty', 'slack-incidents'],
  },
  {
    check: 'SSL Certificate',
    conditions: [
      { metric: 'daysUntilExpiry', operator: '<', threshold: 7 },
    ],
    severity: 'critical',
    notifications: ['pagerduty', 'email-ops'],
  },
  {
    check: 'SSL Certificate',
    conditions: [
      { metric: 'daysUntilExpiry', operator: '<', threshold: 30 },
    ],
    severity: 'warning',
    notifications: ['slack-alerts', 'email-ops'],
  },
];
```

---

## Common Pitfalls

### 1. Not Testing from Multiple Locations

```typescript
// ❌ BAD: Only testing from one location
const result = await runCheck('https://api.example.com/health');
// Misses regional outages!

// ✅ GOOD: Test from multiple regions
const results = await runMultiLocationCheck(
  'API Health',
  'https://api.example.com/health'
);
// Catches regional issues
```

### 2. Alert Fatigue from Flaky Tests

```typescript
// ❌ BAD: Alert on every failure
if (!result.success) {
  sendAlert(result); // Flaky tests cause alert fatigue
}

// ✅ GOOD: Require multiple failures before alerting
const recentResults = getRecentResults(check.name, 3);
const failureCount = recentResults.filter(r => !r.success).length;

if (failureCount >= 2) { // 2 out of 3 failures
  sendAlert(result);
}

function getRecentResults(name: string, count: number): any[] { return []; }
function sendAlert(result: any): void {}
const result = { success: false };
const check = { name: '' };
```

### 3. Not Testing the Full User Flow

```typescript
// ❌ BAD: Only checking if page loads
await page.goto('https://example.com/checkout');
// Page loads but checkout button might be broken!

// ✅ GOOD: Test actual user interactions
await page.goto('https://example.com/products/1');
await page.click('button:has-text("Add to Cart")');
await page.click('a:has-text("Checkout")');
await page.fill('input[name="card"]', '4242424242424242');
await page.click('button:has-text("Pay")');
await page.waitForSelector('text=Order confirmed');

const page = { 
  goto: async (url: string) => {},
  click: async (selector: string) => {},
  fill: async (selector: string, value: string) => {},
  waitForSelector: async (selector: string) => {},
};
```

### 4. Ignoring Performance Baselines

```typescript
// ❌ BAD: Only check if it works
if (response.status === 200) {
  // Success!
}

// ✅ GOOD: Also check performance
if (response.status === 200) {
  if (responseTime > baselineP95 * 1.5) {
    alertPerformanceRegression({
      current: responseTime,
      baseline: baselineP95,
    });
  }
}

const response = { status: 200 };
const responseTime = 0;
const baselineP95 = 1000;
function alertPerformanceRegression(data: object): void {}
```

---

## Interview Questions

### Q1: When should you use synthetic monitoring vs RUM?

**A:** Use synthetic for: baseline performance, availability checking, pre-production testing, SLA measurement, 24/7 monitoring during low traffic. Use RUM for: real user experience, geographic/device distribution, performance correlation with business metrics, identifying edge cases. Use both together for complete observability.

### Q2: How do you handle flaky synthetic tests?

**A:** Implement retry logic with exponential backoff, require multiple consecutive failures before alerting, review and fix tests that fail > X% of the time, separate "flaky" vs "reliable" test pools, investigate root causes (network, test environment, actual issues).

### Q3: What should you monitor with synthetic tests?

**A:** 1) Uptime/availability of critical endpoints, 2) Critical user flows (login, checkout, signup), 3) API contract compliance, 4) SSL certificate expiry, 5) Performance baselines, 6) Multi-region availability, 7) Third-party integrations.

### Q4: How do you determine test frequency?

**A:** Based on criticality and cost. Critical paths (login, checkout): every 5-15 minutes. Important pages: every hour. Lower-priority: daily. Balance between detection speed, cost, and load on your systems. More frequent in production, less frequent in staging.

---

## Quick Reference Checklist

### Uptime Monitoring
- [ ] Health endpoints every 1 minute
- [ ] Multiple geographic locations
- [ ] Response time thresholds
- [ ] Status code verification

### Browser Testing
- [ ] Critical user flows (login, checkout)
- [ ] Form submissions work
- [ ] JavaScript errors captured
- [ ] Performance metrics collected

### SSL Monitoring
- [ ] All production domains covered
- [ ] 30-day warning alerts
- [ ] 7-day critical alerts
- [ ] Auto-renewal verification

### Alerting
- [ ] Require multiple failures before alert
- [ ] Escalation paths defined
- [ ] On-call rotation integrated
- [ ] False positive rate tracked

### Best Practices
- [ ] Test from user perspective
- [ ] Include performance assertions
- [ ] Document test purposes
- [ ] Review and update regularly

---

*Last updated: February 2026*

