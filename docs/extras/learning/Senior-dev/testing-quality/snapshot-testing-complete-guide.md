# 📸 Snapshot Testing - Complete Guide

> A comprehensive guide to snapshot testing - when to use, maintenance strategies, best practices, and avoiding snapshot test anti-patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Snapshot testing captures the output of a component or function, saves it to a file, and compares future runs against that 'snapshot' - making it easy to detect unintended changes but requiring discipline to avoid blindly accepting updates."

### The 7 Key Concepts (Remember These!)
```
1. SNAPSHOT       → Saved output for comparison
2. BASELINE       → The "known good" snapshot
3. UPDATE         → Regenerate snapshot intentionally
4. INLINE         → Snapshot in test file (not external)
5. SERIALIZER     → Custom formatting for output
6. DIFF           → Changes between snapshot and current
7. REVIEW         → Must review changes, not blindly accept
```

### How Snapshot Testing Works
```
┌─────────────────────────────────────────────────────────────────┐
│                HOW SNAPSHOT TESTING WORKS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  FIRST RUN (No snapshot exists)                                │
│  ─────────────────────────────────                              │
│  1. Test executes: render(<Button>Click</Button>)              │
│  2. Output captured: <button class="btn">Click</button>        │
│  3. Snapshot created: __snapshots__/Button.test.tsx.snap       │
│  4. Test PASSES (baseline established)                         │
│                                                                 │
│  SUBSEQUENT RUNS                                                │
│  ────────────────                                               │
│  1. Test executes: render(<Button>Click</Button>)              │
│  2. Output captured: <button class="btn">Click</button>        │
│  3. Compared with saved snapshot                               │
│  4. Match → PASS / Mismatch → FAIL                             │
│                                                                 │
│  WHEN OUTPUT CHANGES                                           │
│  ───────────────────                                            │
│  Before: <button class="btn">Click</button>                    │
│  After:  <button class="btn primary">Click</button>            │
│                                                                 │
│  ┌─────────────────────────────────────────────┐               │
│  │ Snapshot test FAILS!                        │               │
│  │                                             │               │
│  │ - <button class="btn">Click</button>        │               │
│  │ + <button class="btn primary">Click</button>│               │
│  │                                             │               │
│  │ Options:                                    │               │
│  │ • Fix bug if unintended                     │               │
│  │ • Update snapshot if intentional            │               │
│  └─────────────────────────────────────────────┘               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### When to Use Snapshots
```
┌─────────────────────────────────────────────────────────────────┐
│                 WHEN TO USE SNAPSHOTS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ GOOD USE CASES:                                            │
│  • UI component structure                                      │
│  • Serialized data (JSON, XML)                                 │
│  • Error messages                                              │
│  • CLI output                                                  │
│  • Email templates                                             │
│  • API response structure                                      │
│  • Config file generation                                      │
│                                                                 │
│  ❌ BAD USE CASES:                                             │
│  • Random/dynamic data (timestamps, IDs)                       │
│  • Large snapshots (>50 lines)                                 │
│  • Frequently changing output                                  │
│  • Performance-critical assertions                             │
│  • Business logic verification                                 │
│  • Replacing all other test types                              │
│                                                                 │
│  ⚠️ USE WITH CAUTION:                                          │
│  • Entire page snapshots (too large)                           │
│  • Third-party component output                                │
│  • Styled components (CSS changes often)                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Snapshot regression"** | "Snapshot tests catch unintended regressions" |
| **"Snapshot review"** | "We require careful snapshot review in PRs" |
| **"Inline snapshots"** | "We use inline snapshots for small outputs" |
| **"Property matchers"** | "Property matchers handle dynamic values" |
| **"Snapshot pollution"** | "Large snapshots lead to snapshot pollution" |
| **"False positives"** | "Dynamic data causes false positive failures" |

### Key Numbers to Remember
| Metric | Guideline | Notes |
|--------|-----------|-------|
| Snapshot size | **< 50 lines** | Larger = harder to review |
| Update frequency | **Low** | Frequent updates = code smell |
| Coverage | **Supplement, not replace** | Use with other tests |
| Review time | **Actually read diff** | Don't blindly update |

### The "Wow" Statement (Memorize This!)
> "We use snapshot tests strategically - for component structure, serialization output, and error messages. Key rules: snapshots under 50 lines (focused on specific output), property matchers for dynamic values like IDs and timestamps, and mandatory diff review in PRs. We never blindly update snapshots - every update requires explanation. Inline snapshots for small outputs keep context in the test file. We combine snapshots with specific assertions: snapshot verifies structure, unit tests verify behavior. This catches unintended UI changes without the brittleness of large page snapshots. Our snapshot update rate is low - frequent updates indicate either unstable code or misuse of snapshots."

---

## 📚 Table of Contents

1. [Jest Snapshots](#1-jest-snapshots)
2. [React Component Snapshots](#2-react-component-snapshots)
3. [Handling Dynamic Data](#3-handling-dynamic-data)
4. [Maintenance Strategies](#4-maintenance-strategies)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Jest Snapshots

```typescript
// ════════════════════════════════════════════════════════════════
// BASIC SNAPSHOT TESTING
// ════════════════════════════════════════════════════════════════

// Simple value snapshot
test('user object matches snapshot', () => {
  const user = createUser({ name: 'John', role: 'admin' });
  expect(user).toMatchSnapshot();
});

// Generated snapshot file (__snapshots__/user.test.ts.snap):
// exports[`user object matches snapshot 1`] = `
// Object {
//   "id": "user-123",
//   "name": "John",
//   "role": "admin",
//   "createdAt": "2024-01-15T10:00:00.000Z",
// }
// `;

// ════════════════════════════════════════════════════════════════
// INLINE SNAPSHOTS
// ════════════════════════════════════════════════════════════════

// Inline snapshot - stored in test file
test('error message format', () => {
  const error = new ValidationError('email', 'Invalid format');
  
  expect(error.message).toMatchInlineSnapshot(`"Validation failed: email - Invalid format"`);
});

// Benefits: 
// - See expected value in test file
// - No separate snapshot file
// - Better for small outputs

// ════════════════════════════════════════════════════════════════
// SNAPSHOT SERIALIZATION
// ════════════════════════════════════════════════════════════════

// JSON snapshot
test('API response structure', () => {
  const response = {
    data: { users: [{ id: 1, name: 'John' }] },
    meta: { total: 1, page: 1 },
  };
  
  expect(response).toMatchSnapshot();
});

// Custom serializer for specific types
expect.addSnapshotSerializer({
  test: (val) => val instanceof Date,
  print: (val) => `Date: ${val.toISOString()}`,
});

// ════════════════════════════════════════════════════════════════
// UPDATING SNAPSHOTS
// ════════════════════════════════════════════════════════════════

// When output changes intentionally:
// jest --updateSnapshot
// jest -u

// Update specific test:
// jest -u --testNamePattern="error message format"

// Interactive update:
// jest --watch
// Press 'u' to update failing snapshots
```

---

## 2. React Component Snapshots

```tsx
// ════════════════════════════════════════════════════════════════
// REACT COMPONENT SNAPSHOTS
// ════════════════════════════════════════════════════════════════

import { render } from '@testing-library/react';

// Basic component snapshot
test('Button renders correctly', () => {
  const { container } = render(<Button>Click me</Button>);
  expect(container.firstChild).toMatchSnapshot();
});

// Snapshot file:
// exports[`Button renders correctly 1`] = `
// <button
//   class="btn btn-primary"
//   type="button"
// >
//   Click me
// </button>
// `;

// ════════════════════════════════════════════════════════════════
// FOCUSED COMPONENT SNAPSHOTS
// ════════════════════════════════════════════════════════════════

// ❌ Bad: Full component with all variations
test('UserCard renders', () => {
  const { container } = render(
    <UserCard user={user} showDetails expanded onEdit={fn} onDelete={fn} />
  );
  expect(container).toMatchSnapshot(); // 200+ lines!
});

// ✅ Good: Focused snapshots for specific states
describe('UserCard', () => {
  test('renders name and avatar', () => {
    const { getByText, getByRole } = render(<UserCard user={user} />);
    
    expect(getByText(user.name)).toBeInTheDocument();
    expect(getByRole('img')).toHaveAttribute('src', user.avatarUrl);
  });
  
  test('collapsed state matches snapshot', () => {
    const { container } = render(<UserCard user={user} expanded={false} />);
    expect(container.firstChild).toMatchSnapshot();
  });
  
  test('expanded state shows additional info', () => {
    const { getByText } = render(<UserCard user={user} expanded={true} />);
    expect(getByText(user.email)).toBeInTheDocument();
    expect(getByText(user.phone)).toBeInTheDocument();
  });
});

// ════════════════════════════════════════════════════════════════
// REACT TESTING LIBRARY + SNAPSHOTS
// ════════════════════════════════════════════════════════════════

import { render, screen } from '@testing-library/react';

describe('ProductCard', () => {
  const product = {
    id: '123',
    name: 'Test Product',
    price: 99.99,
    description: 'A great product',
  };

  test('renders product information', () => {
    render(<ProductCard product={product} />);
    
    // Specific assertions (preferred)
    expect(screen.getByText('Test Product')).toBeInTheDocument();
    expect(screen.getByText('$99.99')).toBeInTheDocument();
    
    // Snapshot for structure (supplementary)
    expect(screen.getByRole('article')).toMatchSnapshot();
  });

  test('renders out of stock state', () => {
    render(<ProductCard product={{ ...product, inStock: false }} />);
    
    expect(screen.getByText('Out of Stock')).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /add to cart/i })).toBeDisabled();
  });
});

// ════════════════════════════════════════════════════════════════
// STYLED COMPONENTS SNAPSHOTS
// ════════════════════════════════════════════════════════════════

import 'jest-styled-components';

test('Button has correct styles', () => {
  const { container } = render(<Button variant="primary" />);
  
  expect(container.firstChild).toHaveStyleRule('background-color', '#007bff');
  expect(container.firstChild).toHaveStyleRule('color', '#ffffff');
  
  // Or snapshot the styled output
  expect(container.firstChild).toMatchSnapshot();
});
```

---

## 3. Handling Dynamic Data

```typescript
// ════════════════════════════════════════════════════════════════
// PROPERTY MATCHERS FOR DYNAMIC VALUES
// ════════════════════════════════════════════════════════════════

test('user has dynamic fields', () => {
  const user = createUser({ name: 'John' });
  
  // ❌ Bad: Will fail due to dynamic id and timestamp
  expect(user).toMatchSnapshot();
  
  // ✅ Good: Property matchers for dynamic values
  expect(user).toMatchSnapshot({
    id: expect.any(String),
    createdAt: expect.any(Date),
    updatedAt: expect.any(Date),
  });
});

// Snapshot stores:
// exports[`user has dynamic fields 1`] = `
// Object {
//   "createdAt": Any<Date>,
//   "id": Any<String>,
//   "name": "John",
//   "role": "user",
//   "updatedAt": Any<Date>,
// }
// `;

// ════════════════════════════════════════════════════════════════
// MOCKING DYNAMIC VALUES
// ════════════════════════════════════════════════════════════════

// Mock date for consistent snapshots
beforeAll(() => {
  jest.useFakeTimers();
  jest.setSystemTime(new Date('2024-01-15T10:00:00Z'));
});

afterAll(() => {
  jest.useRealTimers();
});

// Mock UUID generation
jest.mock('uuid', () => ({
  v4: () => 'mock-uuid-12345',
}));

// ════════════════════════════════════════════════════════════════
// REPLACING DYNAMIC CONTENT
// ════════════════════════════════════════════════════════════════

test('email template snapshot', () => {
  const email = generateWelcomeEmail({
    name: 'John',
    signupDate: new Date(),
    verificationCode: 'ABC123',
  });
  
  // Replace dynamic values before snapshot
  const normalized = email
    .replace(/\d{4}-\d{2}-\d{2}/g, 'YYYY-MM-DD')
    .replace(/[A-Z0-9]{6}/g, 'XXXXXX');
  
  expect(normalized).toMatchSnapshot();
});

// ════════════════════════════════════════════════════════════════
// CUSTOM SNAPSHOT SERIALIZERS
// ════════════════════════════════════════════════════════════════

// Serializer that redacts sensitive/dynamic data
expect.addSnapshotSerializer({
  test: (val) => val && typeof val === 'object' && 'password' in val,
  print: (val, serialize) => {
    const { password, ...rest } = val;
    return serialize({ ...rest, password: '[REDACTED]' });
  },
});

// Serializer for dates
expect.addSnapshotSerializer({
  test: (val) => val instanceof Date,
  print: () => 'Date<mock>',
});

// Usage
test('user response redacts password', () => {
  const response = {
    id: '123',
    email: 'test@example.com',
    password: 'secret123',
    createdAt: new Date(),
  };
  
  expect(response).toMatchSnapshot();
});

// Snapshot:
// Object {
//   "createdAt": Date<mock>,
//   "email": "test@example.com",
//   "id": "123",
//   "password": "[REDACTED]",
// }
```

---

## 4. Maintenance Strategies

```typescript
// ════════════════════════════════════════════════════════════════
// SNAPSHOT MAINTENANCE BEST PRACTICES
// ════════════════════════════════════════════════════════════════

/*
 * 1. REVIEW EVERY SNAPSHOT UPDATE
 * ────────────────────────────────
 * Never run `jest -u` blindly!
 * 
 * Before updating:
 * - Read the diff carefully
 * - Understand WHY it changed
 * - Verify the change is intentional
 * - If unclear, investigate the code change
 */

// 2. USE DESCRIPTIVE TEST NAMES
// ─────────────────────────────────

// ❌ Bad
test('snapshot 1', () => {
  expect(render(<Component />)).toMatchSnapshot();
});

// ✅ Good
test('UserAvatar renders initials when no image provided', () => {
  expect(render(<UserAvatar name="John Doe" />)).toMatchSnapshot();
});

// 3. KEEP SNAPSHOTS SMALL AND FOCUSED
// ────────────────────────────────────

// ❌ Bad: Entire page snapshot
test('dashboard page', () => {
  const { container } = render(<DashboardPage />);
  expect(container).toMatchSnapshot(); // 500+ lines!
});

// ✅ Good: Specific component snapshot
test('dashboard header shows user name', () => {
  const { getByTestId } = render(<DashboardPage user={mockUser} />);
  expect(getByTestId('dashboard-header')).toMatchSnapshot();
});

// 4. COMMIT SNAPSHOTS WITH RELATED CODE
// ──────────────────────────────────────

/*
 * Commit message example:
 * 
 * feat: Add loading state to UserCard
 * 
 * - Added isLoading prop
 * - Shows skeleton when loading
 * - Updated snapshots to reflect new loading state
 */

// 5. REGULARLY CLEAN UP OBSOLETE SNAPSHOTS
// ─────────────────────────────────────────

// Run periodically:
// jest --ci --detectOpenHandles --updateSnapshot --testPathIgnorePatterns=[]
// This removes orphaned snapshots

// 6. USE SNAPSHOT TESTING JUDICIOUSLY
// ────────────────────────────────────

describe('NotificationBanner', () => {
  // Behavior test (specific assertion)
  test('shows correct icon for error type', () => {
    render(<NotificationBanner type="error" message="Failed" />);
    expect(screen.getByRole('img')).toHaveAttribute('aria-label', 'error icon');
  });

  // Structure test (snapshot)
  test('error banner structure', () => {
    const { container } = render(
      <NotificationBanner type="error" message="Failed" />
    );
    expect(container.firstChild).toMatchSnapshot();
  });

  // Interaction test (behavior)
  test('calls onDismiss when close button clicked', () => {
    const onDismiss = jest.fn();
    render(<NotificationBanner message="Test" onDismiss={onDismiss} />);
    fireEvent.click(screen.getByRole('button', { name: /close/i }));
    expect(onDismiss).toHaveBeenCalled();
  });
});
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# SNAPSHOT TESTING BEST PRACTICES
# ════════════════════════════════════════════════════════════════

dos:
  - Use for UI structure verification
  - Keep snapshots small (<50 lines)
  - Use inline snapshots for small outputs
  - Use property matchers for dynamic values
  - Review every snapshot update carefully
  - Combine with specific assertions
  - Use descriptive test names
  - Commit snapshots with code changes

donts:
  - Don't snapshot entire pages
  - Don't blindly update snapshots
  - Don't replace all tests with snapshots
  - Don't snapshot frequently changing output
  - Don't snapshot implementation details
  - Don't ignore large snapshot diffs
  - Don't test business logic with snapshots

when_to_use:
  ideal:
    - Component structure
    - Serialized output (JSON, XML)
    - Error messages
    - CLI output
    - Config generation
    - Email templates
    
  avoid:
    - Large page renders
    - Outputs with dates/IDs
    - Third-party components
    - Frequently updated code
    - Logic verification

review_checklist:
  - Is the change intentional?
  - Does the diff match the code change?
  - Is the snapshot still readable?
  - Should this be a specific assertion instead?
  - Are dynamic values properly handled?
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# SNAPSHOT TESTING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Blindly updating snapshots
# Bad
$ jest -u  # Without reviewing what changed
# Just make the tests pass!

# Good
# Review each diff in snapshot
# Understand why it changed
# Update only if intentional

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Huge snapshots
# Bad
test('page snapshot', () => {
  expect(render(<EntirePage />).container).toMatchSnapshot();
});
# 500+ line snapshot no one reviews

# Good
test('header structure', () => {
  expect(render(<Header />).container).toMatchSnapshot();
});
# Small, focused, reviewable

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Dynamic values causing flaky tests
# Bad
test('user snapshot', () => {
  expect(createUser()).toMatchSnapshot();
});
# Fails because ID and timestamp change

# Good
test('user snapshot', () => {
  expect(createUser()).toMatchSnapshot({
    id: expect.any(String),
    createdAt: expect.any(Date),
  });
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Testing behavior with snapshots
# Bad
test('calculates total correctly', () => {
  expect(calculateTotal(items)).toMatchSnapshot();
});
# Doesn't clearly test the expectation

# Good
test('calculates total correctly', () => {
  expect(calculateTotal(items)).toBe(150.00);
});
# Clear expectation

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Not cleaning obsolete snapshots
# Bad
# Deleted test but snapshot file remains
# Stale snapshots accumulate

# Good
# Run: jest --ci --updateSnapshot
# Removes orphaned snapshots
# Do periodically or in CI

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Snapshots as only testing strategy
# Bad
# All tests are snapshots
# No behavior verification

# Good
# Snapshots for structure
# Unit tests for logic
# Integration tests for workflows
# E2E for critical paths
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is snapshot testing?"**
> "Snapshot testing captures output (component render, JSON, etc.), saves it to a file, and compares future runs against that baseline. If output changes, test fails. Good for catching unintended changes. Must review updates carefully - don't blindly accept."

**Q: "When should you use snapshot testing?"**
> "Use for: UI component structure, serialized data, error messages, config output. Avoid for: business logic, large pages, frequently changing code, dynamic data (timestamps, IDs). Snapshots supplement specific assertions, don't replace them."

**Q: "What's the risk of snapshot testing?"**
> "Main risk: blindly updating snapshots. Developer runs `jest -u` without reviewing diff, accepting bugs as 'new baseline'. Also: large snapshots no one reads, false failures from dynamic data, tests that pass but don't verify behavior."

### Intermediate Questions

**Q: "How do you handle dynamic values in snapshots?"**
> "Use property matchers: `expect(data).toMatchSnapshot({ id: expect.any(String), date: expect.any(Date) })`. Or mock time/IDs, or use custom serializers that replace dynamic values with placeholders. Never let random data into snapshots."

**Q: "What's the difference between snapshots and assertions?"**
> "Snapshots: 'Did output change?' - catches any change, intended or not. Assertions: 'Is X correct?' - verifies specific expectations. Use snapshots for structure/regression, assertions for behavior. Combine both: snapshot verifies shape, assertion verifies values."

**Q: "How should snapshot updates be handled in PRs?"**
> "Require review of all snapshot changes. PR should explain why snapshot changed. Large changes are red flag - investigate. Reviewer should: read the diff, verify it matches code change, check if specific assertion is better. Never approve blind updates."

### Advanced Questions

**Q: "How do you avoid snapshot test pollution?"**
> "Keep snapshots small (<50 lines). One assertion per test. Snapshot specific elements, not entire pages. Use inline snapshots for small outputs. Clean orphaned snapshots regularly. Don't snapshot third-party or frequently changing components."

**Q: "Snapshots vs visual regression testing?"**
> "Snapshots: Test DOM structure (text-based). Fast, deterministic. Catches structural changes. Visual regression: Test rendered appearance (screenshot comparison). Catches styling issues, browser rendering differences. Both valuable: snapshots for structure, visual for appearance."

**Q: "How do you test components with snapshot AND behavior tests?"**
> "Combine approaches: Snapshot verifies structure exists and is stable. Unit tests verify specific behaviors (button click calls handler). Integration tests verify component works in context. Example: snapshot for 'renders header', assertion for 'header shows user name'."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                 SNAPSHOT TESTING CHECKLIST                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  WHEN TO USE:                                                   │
│  □ UI component structure                                      │
│  □ Serialized output (JSON, XML)                               │
│  □ Error messages                                              │
│  □ CLI/config output                                           │
│                                                                 │
│  AVOID:                                                         │
│  □ Large page snapshots                                        │
│  □ Business logic testing                                      │
│  □ Frequently changing output                                  │
│  □ Dynamic data without matchers                               │
│                                                                 │
│  MAINTENANCE:                                                   │
│  □ Review every update                                         │
│  □ Keep snapshots < 50 lines                                   │
│  □ Use property matchers for dynamic values                    │
│  □ Clean orphaned snapshots                                    │
│  □ Commit with related code changes                            │
│                                                                 │
│  COMBINE WITH:                                                  │
│  □ Specific assertions for behavior                            │
│  □ Unit tests for logic                                        │
│  □ Visual tests for appearance                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

JEST SNAPSHOT COMMANDS:
┌────────────────────────────────────────────────────────────────┐
│ jest                           # Run tests, fail on mismatch   │
│ jest -u                        # Update all snapshots          │
│ jest -u --testNamePattern="X"  # Update specific test          │
│ jest --watch → press 'u'       # Interactive update            │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

