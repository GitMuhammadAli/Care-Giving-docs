# 🎭 E2E Testing - Complete Guide

> A comprehensive guide to end-to-end testing - Playwright, Cypress, visual regression, handling flaky tests, and testing real user flows.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "E2E (End-to-End) tests validate complete user workflows through the actual application UI, simulating real user behavior from start to finish - clicking buttons, filling forms, navigating pages - to ensure the entire system works together correctly."

### The 7 Key Concepts (Remember These!)
```
1. USER FLOWS       → Test complete journeys (signup → checkout)
2. REAL BROWSER     → Tests run in actual Chrome, Firefox, Safari
3. PAGE OBJECTS     → Encapsulate page interactions
4. SELECTORS        → Find elements (data-testid preferred)
5. WAITING          → Handle async operations properly
6. VISUAL TESTING   → Screenshot comparisons
7. FLAKY TESTS      → Unreliable tests that pass/fail randomly
```

### E2E vs Other Test Types
```
┌─────────────────────────────────────────────────────────────────┐
│                    E2E TEST SCOPE                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Unit Test:    [Function] ─────────────────────────────────    │
│                                                                 │
│  Integration:  [Service] → [Database] ─────────────────────    │
│                                                                 │
│  E2E:          [Browser] → [Frontend] → [API] → [DB]           │
│                    ↓           ↓          ↓       ↓            │
│                  User       React      Express  PostgreSQL      │
│                  clicks     renders    handles   stores         │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  E2E TESTS:                                                    │
│  • Highest confidence (tests what users experience)            │
│  • Slowest (minutes per test)                                  │
│  • Most brittle (UI changes break tests)                       │
│  • Most expensive (real infrastructure)                        │
│  • Used sparingly (critical paths only)                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### When to E2E Test
```
┌─────────────────────────────────────────────────────────────────┐
│                   WHEN TO E2E TEST                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ DO E2E TEST (Critical Paths):                              │
│  • User registration / login / logout                          │
│  • Checkout / payment flows                                    │
│  • Core business workflows                                     │
│  • Permission-protected pages                                  │
│  • Multi-step wizards / forms                                  │
│                                                                 │
│  ❌ DON'T E2E TEST:                                            │
│  • Every form field validation (unit test it)                  │
│  • API error handling (integration test it)                    │
│  • Component styling (visual regression)                       │
│  • Edge cases (unit test them)                                 │
│  • Performance (load test it)                                  │
│                                                                 │
│  RULE: E2E tests = 5-10% of test suite                         │
│  Focus on HAPPY PATHS of CRITICAL FLOWS                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Page Object Model"** | "We use POM to encapsulate page interactions" |
| **"Test isolation"** | "Each E2E test is isolated with fresh data" |
| **"Flaky test"** | "We quarantine flaky tests and fix root causes" |
| **"Visual regression"** | "Visual regression catches unintended UI changes" |
| **"Headed/headless"** | "Tests run headless in CI, headed for debugging" |
| **"Data-testid"** | "We use data-testid selectors for stability" |

### Key Numbers to Remember
| Metric | Target | Why |
|--------|--------|-----|
| E2E coverage | **Critical paths only** | 5-10% of suite |
| Test duration | **< 2 minutes each** | Reasonable CI time |
| Parallelization | **Yes** | Speed up suite |
| Flaky rate | **< 1%** | Reliability |
| Retry count | **2-3 max** | Don't mask issues |

### The "Wow" Statement (Memorize This!)
> "We use Playwright for E2E testing, covering only critical user journeys - registration, login, checkout, core workflows. Tests use Page Object Model for maintainability - page classes encapsulate selectors and interactions. We use data-testid attributes for stable selectors that survive refactoring. Tests run in parallel across Chromium, Firefox, and WebKit for cross-browser coverage. Visual regression with Percy catches unintended UI changes. We avoid arbitrary waits - always wait for specific conditions. Flaky tests go into quarantine immediately - we fix root causes, not add retries. Tests run headless in CI, headed locally for debugging. Our suite covers 15 critical flows in under 10 minutes."

---

## 📚 Table of Contents

1. [Playwright](#1-playwright)
2. [Cypress](#2-cypress)
3. [Page Object Model](#3-page-object-model)
4. [Handling Async](#4-handling-async)
5. [Visual Regression](#5-visual-regression)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Playwright

```typescript
// ════════════════════════════════════════════════════════════════
// PLAYWRIGHT CONFIGURATION
// playwright.config.ts
// ════════════════════════════════════════════════════════════════

import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI, // Fail if test.only in CI
  retries: process.env.CI ? 2 : 0, // Retry in CI only
  workers: process.env.CI ? 1 : undefined, // Parallel workers
  reporter: [
    ['html'],
    ['junit', { outputFile: 'results/junit.xml' }],
  ],
  
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry', // Collect trace on retry
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'mobile',
      use: { ...devices['iPhone 13'] },
    },
  ],

  // Start dev server before tests
  webServer: {
    command: 'npm run start:test',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
  },
});

// ════════════════════════════════════════════════════════════════
// PLAYWRIGHT TESTS
// ════════════════════════════════════════════════════════════════

import { test, expect, Page } from '@playwright/test';

test.describe('Authentication Flow', () => {
  test('user can register and login', async ({ page }) => {
    // Generate unique email for this test
    const email = `test-${Date.now()}@example.com`;
    const password = 'SecurePass123!';

    // Navigate to registration
    await page.goto('/register');

    // Fill registration form
    await page.fill('[data-testid="email-input"]', email);
    await page.fill('[data-testid="password-input"]', password);
    await page.fill('[data-testid="confirm-password-input"]', password);
    await page.fill('[data-testid="name-input"]', 'Test User');

    // Submit
    await page.click('[data-testid="register-button"]');

    // Should redirect to dashboard
    await expect(page).toHaveURL(/\/dashboard/);
    await expect(page.locator('[data-testid="welcome-message"]'))
      .toContainText('Welcome, Test User');

    // Logout
    await page.click('[data-testid="user-menu"]');
    await page.click('[data-testid="logout-button"]');
    await expect(page).toHaveURL('/login');

    // Login with new account
    await page.fill('[data-testid="email-input"]', email);
    await page.fill('[data-testid="password-input"]', password);
    await page.click('[data-testid="login-button"]');

    // Back to dashboard
    await expect(page).toHaveURL(/\/dashboard/);
  });

  test('shows error for invalid credentials', async ({ page }) => {
    await page.goto('/login');

    await page.fill('[data-testid="email-input"]', 'wrong@example.com');
    await page.fill('[data-testid="password-input"]', 'wrongpassword');
    await page.click('[data-testid="login-button"]');

    await expect(page.locator('[data-testid="error-message"]'))
      .toHaveText('Invalid email or password');
    await expect(page).toHaveURL('/login'); // Still on login page
  });
});

test.describe('Checkout Flow', () => {
  // Use authenticated state
  test.use({ storageState: 'e2e/.auth/user.json' });

  test('complete purchase', async ({ page }) => {
    // Add product to cart
    await page.goto('/products');
    await page.click('[data-testid="product-card"]:first-child [data-testid="add-to-cart"]');
    
    // Verify cart updated
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1');

    // Go to cart
    await page.click('[data-testid="cart-icon"]');
    await expect(page).toHaveURL('/cart');

    // Proceed to checkout
    await page.click('[data-testid="checkout-button"]');

    // Fill shipping
    await page.fill('[data-testid="shipping-address"]', '123 Main St');
    await page.fill('[data-testid="shipping-city"]', 'New York');
    await page.fill('[data-testid="shipping-zip"]', '10001');
    await page.click('[data-testid="continue-button"]');

    // Fill payment (test card)
    await page.fill('[data-testid="card-number"]', '4242424242424242');
    await page.fill('[data-testid="card-expiry"]', '12/25');
    await page.fill('[data-testid="card-cvc"]', '123');

    // Place order
    await page.click('[data-testid="place-order-button"]');

    // Confirmation page
    await expect(page).toHaveURL(/\/order-confirmation/);
    await expect(page.locator('[data-testid="order-number"]')).toBeVisible();
    await expect(page.locator('[data-testid="success-message"]'))
      .toContainText('Thank you for your order');
  });
});
```

---

## 2. Cypress

```typescript
// ════════════════════════════════════════════════════════════════
// CYPRESS CONFIGURATION
// cypress.config.ts
// ════════════════════════════════════════════════════════════════

import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: true,
    screenshotOnRunFailure: true,
    retries: {
      runMode: 2,
      openMode: 0,
    },
    setupNodeEvents(on, config) {
      // Database tasks
      on('task', {
        async seedDatabase() {
          // Seed test data
          return null;
        },
        async clearDatabase() {
          // Clear test data
          return null;
        },
      });
    },
  },
});

// ════════════════════════════════════════════════════════════════
// CYPRESS TESTS
// cypress/e2e/auth.cy.ts
// ════════════════════════════════════════════════════════════════

describe('Authentication', () => {
  beforeEach(() => {
    cy.task('clearDatabase');
    cy.task('seedDatabase');
  });

  it('allows user to register', () => {
    const email = `test-${Date.now()}@example.com`;

    cy.visit('/register');

    cy.get('[data-testid="email-input"]').type(email);
    cy.get('[data-testid="password-input"]').type('SecurePass123!');
    cy.get('[data-testid="name-input"]').type('Test User');
    cy.get('[data-testid="register-button"]').click();

    cy.url().should('include', '/dashboard');
    cy.get('[data-testid="welcome-message"]').should('contain', 'Test User');
  });

  it('shows validation errors', () => {
    cy.visit('/register');

    cy.get('[data-testid="register-button"]').click();

    cy.get('[data-testid="email-error"]').should('be.visible');
    cy.get('[data-testid="password-error"]').should('be.visible');
  });
});

describe('Shopping Cart', () => {
  beforeEach(() => {
    cy.login('test@example.com', 'password123'); // Custom command
  });

  it('adds item to cart', () => {
    cy.visit('/products');

    cy.get('[data-testid="product-card"]').first().within(() => {
      cy.get('[data-testid="add-to-cart"]').click();
    });

    cy.get('[data-testid="cart-count"]').should('have.text', '1');
    cy.get('[data-testid="toast"]').should('contain', 'Added to cart');
  });

  it('completes checkout', () => {
    // Add item to cart
    cy.visit('/products');
    cy.get('[data-testid="add-to-cart"]').first().click();

    // Go to checkout
    cy.get('[data-testid="cart-icon"]').click();
    cy.get('[data-testid="checkout-button"]').click();

    // Shipping
    cy.get('[data-testid="address"]').type('123 Main St');
    cy.get('[data-testid="city"]').type('NYC');
    cy.get('[data-testid="zip"]').type('10001');
    cy.get('[data-testid="continue"]').click();

    // Payment
    cy.getStripeElement('cardNumber').type('4242424242424242');
    cy.getStripeElement('cardExpiry').type('1225');
    cy.getStripeElement('cardCvc').type('123');

    cy.get('[data-testid="place-order"]').click();

    // Confirmation
    cy.url().should('include', '/order-confirmation');
    cy.get('[data-testid="order-number"]').should('exist');
  });
});

// ════════════════════════════════════════════════════════════════
// CYPRESS CUSTOM COMMANDS
// cypress/support/commands.ts
// ════════════════════════════════════════════════════════════════

Cypress.Commands.add('login', (email: string, password: string) => {
  cy.session([email, password], () => {
    cy.visit('/login');
    cy.get('[data-testid="email-input"]').type(email);
    cy.get('[data-testid="password-input"]').type(password);
    cy.get('[data-testid="login-button"]').click();
    cy.url().should('include', '/dashboard');
  });
});

Cypress.Commands.add('getStripeElement', (fieldName: string) => {
  return cy
    .get(`iframe[name*="__stripe"]`)
    .its('0.contentDocument.body')
    .should('not.be.empty')
    .then(cy.wrap)
    .find(`input[name="${fieldName}"]`);
});

// Type definitions
declare global {
  namespace Cypress {
    interface Chainable {
      login(email: string, password: string): Chainable<void>;
      getStripeElement(fieldName: string): Chainable<JQuery<HTMLElement>>;
    }
  }
}
```

---

## 3. Page Object Model

```typescript
// ════════════════════════════════════════════════════════════════
// PAGE OBJECT MODEL - PLAYWRIGHT
// ════════════════════════════════════════════════════════════════

// e2e/pages/login.page.ts
import { Page, Locator, expect } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;
  readonly errorMessage: Locator;
  readonly forgotPasswordLink: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.locator('[data-testid="email-input"]');
    this.passwordInput = page.locator('[data-testid="password-input"]');
    this.loginButton = page.locator('[data-testid="login-button"]');
    this.errorMessage = page.locator('[data-testid="error-message"]');
    this.forgotPasswordLink = page.locator('[data-testid="forgot-password"]');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  async expectError(message: string) {
    await expect(this.errorMessage).toContainText(message);
  }

  async expectLoggedIn() {
    await expect(this.page).toHaveURL(/\/dashboard/);
  }
}

// e2e/pages/checkout.page.ts
export class CheckoutPage {
  readonly page: Page;

  // Shipping section
  readonly addressInput: Locator;
  readonly cityInput: Locator;
  readonly zipInput: Locator;
  readonly continueToPaymentButton: Locator;

  // Payment section
  readonly cardNumberInput: Locator;
  readonly cardExpiryInput: Locator;
  readonly cardCvcInput: Locator;
  readonly placeOrderButton: Locator;

  // Confirmation
  readonly orderNumber: Locator;
  readonly successMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.addressInput = page.locator('[data-testid="shipping-address"]');
    this.cityInput = page.locator('[data-testid="shipping-city"]');
    this.zipInput = page.locator('[data-testid="shipping-zip"]');
    this.continueToPaymentButton = page.locator('[data-testid="continue-to-payment"]');
    this.cardNumberInput = page.locator('[data-testid="card-number"]');
    this.cardExpiryInput = page.locator('[data-testid="card-expiry"]');
    this.cardCvcInput = page.locator('[data-testid="card-cvc"]');
    this.placeOrderButton = page.locator('[data-testid="place-order"]');
    this.orderNumber = page.locator('[data-testid="order-number"]');
    this.successMessage = page.locator('[data-testid="success-message"]');
  }

  async fillShipping(address: string, city: string, zip: string) {
    await this.addressInput.fill(address);
    await this.cityInput.fill(city);
    await this.zipInput.fill(zip);
    await this.continueToPaymentButton.click();
  }

  async fillPayment(cardNumber: string, expiry: string, cvc: string) {
    await this.cardNumberInput.fill(cardNumber);
    await this.cardExpiryInput.fill(expiry);
    await this.cardCvcInput.fill(cvc);
  }

  async placeOrder() {
    await this.placeOrderButton.click();
  }

  async getOrderNumber(): Promise<string> {
    return (await this.orderNumber.textContent()) || '';
  }

  async expectOrderConfirmed() {
    await expect(this.successMessage).toContainText('Thank you');
    await expect(this.orderNumber).toBeVisible();
  }
}

// ════════════════════════════════════════════════════════════════
// USING PAGE OBJECTS IN TESTS
// ════════════════════════════════════════════════════════════════

import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/login.page';
import { CheckoutPage } from './pages/checkout.page';

test.describe('E-commerce Flow', () => {
  test('complete purchase journey', async ({ page }) => {
    const loginPage = new LoginPage(page);
    const checkoutPage = new CheckoutPage(page);

    // Login
    await loginPage.goto();
    await loginPage.login('buyer@example.com', 'password123');
    await loginPage.expectLoggedIn();

    // Add item to cart (could be another page object)
    await page.goto('/products');
    await page.click('[data-testid="add-to-cart"]:first-child');
    await page.click('[data-testid="go-to-checkout"]');

    // Checkout
    await checkoutPage.fillShipping('123 Main St', 'New York', '10001');
    await checkoutPage.fillPayment('4242424242424242', '12/25', '123');
    await checkoutPage.placeOrder();
    
    // Verify
    await checkoutPage.expectOrderConfirmed();
    const orderNumber = await checkoutPage.getOrderNumber();
    expect(orderNumber).toMatch(/^ORD-\d+$/);
  });
});
```

---

## 4. Handling Async

```typescript
// ════════════════════════════════════════════════════════════════
// PROPER ASYNC HANDLING
// ════════════════════════════════════════════════════════════════

// ❌ BAD: Arbitrary waits
test('bad async handling', async ({ page }) => {
  await page.click('[data-testid="submit"]');
  await page.waitForTimeout(3000); // Don't do this!
  expect(await page.textContent('.result')).toBe('Success');
});

// ✅ GOOD: Wait for specific conditions
test('good async handling', async ({ page }) => {
  await page.click('[data-testid="submit"]');
  
  // Wait for element to appear
  await expect(page.locator('[data-testid="result"]'))
    .toHaveText('Success', { timeout: 5000 });
});

// ════════════════════════════════════════════════════════════════
// WAITING STRATEGIES
// ════════════════════════════════════════════════════════════════

// Wait for navigation
await page.click('[data-testid="submit"]');
await page.waitForURL(/\/success/);

// Wait for network request to complete
await Promise.all([
  page.waitForResponse(response => 
    response.url().includes('/api/orders') && response.status() === 200
  ),
  page.click('[data-testid="submit"]'),
]);

// Wait for element state
await expect(page.locator('[data-testid="modal"]')).toBeVisible();
await expect(page.locator('[data-testid="loading"]')).toBeHidden();
await expect(page.locator('[data-testid="button"]')).toBeEnabled();

// Wait for network to be idle
await page.goto('/dashboard', { waitUntil: 'networkidle' });

// Wait for specific content
await expect(page.locator('table tbody tr')).toHaveCount(10);
await expect(page.locator('[data-testid="total"]')).toContainText('$100');

// Retry assertions automatically (Playwright)
await expect(async () => {
  const count = await page.locator('.item').count();
  expect(count).toBeGreaterThan(0);
}).toPass({ timeout: 10000 });

// ════════════════════════════════════════════════════════════════
// HANDLING LOADING STATES
// ════════════════════════════════════════════════════════════════

async function submitAndWaitForResult(page: Page) {
  // Click submit and wait for loading to appear then disappear
  await page.click('[data-testid="submit"]');
  
  // Wait for loading indicator
  const loading = page.locator('[data-testid="loading"]');
  await expect(loading).toBeVisible({ timeout: 1000 }).catch(() => {
    // Loading might be so fast it's not visible - that's okay
  });
  
  // Wait for loading to disappear
  await expect(loading).toBeHidden({ timeout: 30000 });
  
  // Now check result
  return page.locator('[data-testid="result"]');
}

// ════════════════════════════════════════════════════════════════
// HANDLING ANIMATIONS
// ════════════════════════════════════════════════════════════════

// Disable animations for tests
test.use({
  // Playwright: reduce motion
  contextOptions: {
    reducedMotion: 'reduce',
  },
});

// Or in CSS
/* test-specific.css */
*, *::before, *::after {
  animation-duration: 0s !important;
  transition-duration: 0s !important;
}
```

---

## 5. Visual Regression

```typescript
// ════════════════════════════════════════════════════════════════
// VISUAL REGRESSION TESTING - PLAYWRIGHT
// ════════════════════════════════════════════════════════════════

import { test, expect } from '@playwright/test';

test.describe('Visual Regression', () => {
  test('homepage matches snapshot', async ({ page }) => {
    await page.goto('/');
    
    // Wait for content to load
    await expect(page.locator('[data-testid="hero"]')).toBeVisible();
    
    // Full page screenshot
    await expect(page).toHaveScreenshot('homepage.png', {
      fullPage: true,
      maxDiffPixels: 100, // Allow small differences
    });
  });

  test('product card matches snapshot', async ({ page }) => {
    await page.goto('/products');
    
    // Screenshot specific element
    const productCard = page.locator('[data-testid="product-card"]').first();
    await expect(productCard).toHaveScreenshot('product-card.png');
  });

  test('checkout form matches snapshot', async ({ page }) => {
    await page.goto('/checkout');
    
    // Mask dynamic content
    await expect(page).toHaveScreenshot('checkout-form.png', {
      mask: [
        page.locator('[data-testid="timestamp"]'),
        page.locator('[data-testid="order-id"]'),
      ],
    });
  });
});

// ════════════════════════════════════════════════════════════════
// VISUAL TESTING WITH PERCY
// ════════════════════════════════════════════════════════════════

import percySnapshot from '@percy/playwright';

test('visual test with Percy', async ({ page }) => {
  await page.goto('/dashboard');
  
  // Percy handles comparison in cloud
  await percySnapshot(page, 'Dashboard');
  
  // With options
  await percySnapshot(page, 'Dashboard - Mobile', {
    widths: [375, 768],
    percyCSS: `.dynamic-content { visibility: hidden; }`,
  });
});

// ════════════════════════════════════════════════════════════════
// HANDLING DYNAMIC CONTENT IN SCREENSHOTS
// ════════════════════════════════════════════════════════════════

test('screenshot with dynamic content', async ({ page }) => {
  await page.goto('/profile');
  
  // Hide dynamic elements via CSS
  await page.addStyleTag({
    content: `
      [data-testid="timestamp"],
      [data-testid="avatar"],
      .dynamic-ad { visibility: hidden !important; }
    `,
  });
  
  // Or replace with static content
  await page.evaluate(() => {
    document.querySelectorAll('[data-testid="timestamp"]').forEach(el => {
      el.textContent = '2024-01-15 10:00:00';
    });
  });
  
  await expect(page).toHaveScreenshot('profile.png');
});
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# E2E TESTING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Arbitrary waits
# Bad
await page.click('#submit');
await page.waitForTimeout(5000);

# Good
await page.click('#submit');
await expect(page.locator('.success')).toBeVisible();

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Brittle selectors
# Bad
await page.click('div.container > div:nth-child(2) > button.blue');
# Breaks if CSS changes

# Good
await page.click('[data-testid="submit-button"]');
# Stable, survives refactoring

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Not isolating tests
# Bad
# Test 1 creates user
# Test 2 assumes user exists
# Tests fail when run in different order

# Good
beforeEach(async () => {
  await resetDatabase();
  await seedTestData();
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Testing too much with E2E
# Bad
# 200 E2E tests covering every feature
# Takes 2 hours to run, constantly failing

# Good
# 20 E2E tests covering critical paths
# Unit/integration tests for everything else

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Ignoring flaky tests
# Bad
# Test fails sometimes
# Add retry: 5
# Move on

# Good
# Investigate root cause
# Fix timing issues, add proper waits
# Quarantine until fixed

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: No test data management
# Bad
# Tests modify production-like data
# State bleeds between tests
# Unpredictable failures

# Good
# Seed known test data
# Clean up after tests
# Use unique identifiers per test
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is E2E testing?"**
> "End-to-end tests validate complete user workflows through the actual UI - real browser, real clicks, real navigation. Tests simulate what users do: register, login, checkout. They test the entire stack working together. Used for critical paths only (5-10% of tests) because they're slow and brittle."

**Q: "Playwright vs Cypress?"**
> "Playwright: Multiple browsers (Chrome, Firefox, Safari), multiple languages, parallel by default, better async handling. Cypress: JavaScript only, great DevX and debugging, automatic waiting, time-travel debugging. Playwright better for cross-browser, Cypress better for developer experience."

**Q: "What is Page Object Model?"**
> "Design pattern that encapsulates page interactions into classes. Each page has a class with selectors and methods (loginPage.login(email, pass)). Benefits: Reusability, maintainability, readability. When UI changes, update one place, not every test."

### Intermediate Questions

**Q: "How do you handle flaky tests?"**
> "First, investigate root cause - usually timing issues. Add proper waits (wait for element, not arbitrary timeout). Ensure test isolation (clean data). Check for race conditions. If truly random, quarantine and fix. Don't just add retries - that masks the problem."

**Q: "How do you handle authentication in E2E tests?"**
> "Don't login via UI for every test (slow). Options: 1) API login and inject token. 2) Use storageState to save/restore auth. 3) cy.session() in Cypress. 4) Bypass auth in test environment. Login UI should be tested once, then skip for other tests."

**Q: "How do you choose selectors?"**
> "Priority: data-testid > aria roles > text content > CSS class. data-testid is most stable - survives refactoring, CSS changes, text changes. Never use dynamic IDs or generated classes. Add testid to components specifically for testing."

### Advanced Questions

**Q: "How do you test third-party integrations (Stripe, OAuth)?"**
> "Don't test their code. Options: 1) Use sandbox/test mode (Stripe test cards). 2) Mock at network level. 3) Use fake implementations. 4) Test up to the boundary, mock the response. Verify your code handles their responses correctly."

**Q: "How do you structure E2E tests for a large application?"**
> "Organize by user journey, not by page. Critical flows first: auth, core business, checkout. Page objects for maintainability. Shared utilities for common actions. Parallel execution. Run smoke tests on every PR, full suite nightly. Tag tests by priority."

**Q: "How do you handle visual regression testing?"**
> "Screenshot comparison tools (Playwright snapshots, Percy, Chromatic). Challenges: dynamic content (mask or mock), animations (disable), cross-browser differences (per-browser baselines). Review changes in PR. Accept intentional changes, reject unintended."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                    E2E TESTING CHECKLIST                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TEST COVERAGE:                                                 │
│  □ Critical user journeys only                                 │
│  □ Registration / login / logout                               │
│  □ Core business flows                                         │
│  □ Checkout / payment                                          │
│                                                                 │
│  IMPLEMENTATION:                                                │
│  □ Page Object Model                                           │
│  □ data-testid selectors                                       │
│  □ Proper async handling (no arbitrary waits)                  │
│  □ Test isolation (clean state)                                │
│                                                                 │
│  STABILITY:                                                     │
│  □ < 1% flaky rate                                             │
│  □ Root cause investigation for failures                       │
│  □ No excessive retries                                        │
│                                                                 │
│  CI/CD:                                                         │
│  □ Parallel execution                                          │
│  □ Screenshots/videos on failure                               │
│  □ Cross-browser testing                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

SELECTOR PRIORITY:
┌────────────────────────────────────────────────────────────────┐
│ 1. data-testid="submit"          ← BEST (stable)               │
│ 2. role="button" name="Submit"   ← Good (accessible)           │
│ 3. text="Submit"                 ← OK (may change)             │
│ 4. .btn-primary                  ← Avoid (CSS can change)      │
│ 5. div > div:nth-child(2)        ← NEVER (extremely brittle)   │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

