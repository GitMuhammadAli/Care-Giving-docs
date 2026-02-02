# 📊 Code Coverage - Complete Guide

> A comprehensive guide to code coverage - meaningful coverage metrics, coverage reports, setting goals, and avoiding the coverage trap.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Code coverage measures what percentage of your code is executed during tests - including line, branch, function, and statement coverage - but high coverage doesn't guarantee good tests, it only shows what was run, not what was verified."

### The 7 Key Concepts (Remember These!)
```
1. LINE COVERAGE      → % of lines executed
2. BRANCH COVERAGE    → % of if/else branches taken
3. FUNCTION COVERAGE  → % of functions called
4. STATEMENT COVERAGE → % of statements executed
5. COVERAGE REPORT    → Visual breakdown of coverage
6. COVERAGE GAP       → Untested code sections
7. COVERAGE TRAP      → High coverage ≠ quality tests
```

### Coverage Types Explained
```
┌─────────────────────────────────────────────────────────────────┐
│                    COVERAGE TYPES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  function processOrder(order) {          LINE  BRANCH FUNCTION │
│    if (order.items.length === 0) {        ✓      ✓              │
│      throw new Error('Empty order');      ✗      ✗              │
│    }                                                            │
│                                                                 │
│    const total = calculateTotal(order);   ✓      -       ✓     │
│                                                                 │
│    if (total > 1000) {                    ✓      ✓              │
│      applyDiscount(order);                ✗      ✗              │
│    }                                                            │
│                                                                 │
│    return total;                          ✓      -              │
│  }                                               Function: ✓    │
│                                                                 │
│  ───────────────────────────────────────────────────────────── │
│                                                                 │
│  With ONE test: processOrder({ items: [{...}] })               │
│                                                                 │
│  Line Coverage:     5/7 = 71%                                  │
│  Branch Coverage:   2/4 = 50%  (empty order, high total)       │
│  Function Coverage: 2/3 = 67%  (applyDiscount not called)      │
│                                                                 │
│  MISSING:                                                      │
│  • Empty order branch (line 3)                                 │
│  • High total branch (line 9 - applyDiscount)                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### The Coverage Trap
```
┌─────────────────────────────────────────────────────────────────┐
│                   THE COVERAGE TRAP                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ❌ HIGH COVERAGE, BAD TESTS:                                  │
│                                                                 │
│  function add(a, b) { return a + b; }                          │
│                                                                 │
│  test('add', () => {                                           │
│    add(2, 3);  // 100% coverage, 0% verification!              │
│  });                                                            │
│                                                                 │
│  The test runs the code but doesn't check the result.          │
│  A bug (a - b instead of a + b) would NOT be caught.           │
│                                                                 │
│  ───────────────────────────────────────────────────────────── │
│                                                                 │
│  ✓ MEANINGFUL COVERAGE:                                        │
│                                                                 │
│  test('add returns sum of two numbers', () => {                │
│    expect(add(2, 3)).toBe(5);   // Verified!                   │
│    expect(add(-1, 1)).toBe(0);  // Edge case!                  │
│    expect(add(0, 0)).toBe(0);   // Boundary!                   │
│  });                                                            │
│                                                                 │
│  COVERAGE WITHOUT ASSERTIONS = FALSE CONFIDENCE                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Branch coverage"** | "We track branch coverage, not just lines" |
| **"Coverage gap"** | "The report showed coverage gaps in error handling" |
| **"Meaningful coverage"** | "We focus on meaningful coverage with assertions" |
| **"Coverage ratchet"** | "We use a coverage ratchet - can't decrease coverage" |
| **"Uncovered lines"** | "Review uncovered lines before merging" |
| **"Coverage threshold"** | "CI fails below 80% coverage threshold" |

### Key Numbers to Remember
| Metric | Target | Notes |
|--------|--------|-------|
| Line coverage | **80%** | Industry standard |
| Branch coverage | **70%** | Harder to achieve |
| Function coverage | **90%** | Functions should be tested |
| Critical code | **90%+** | Payment, auth, core logic |
| New code | **80%+** | Don't add untested code |

### The "Wow" Statement (Memorize This!)
> "We use Istanbul/nyc for coverage with 80% line and 70% branch thresholds. But we don't obsess over the number - high coverage without assertions is meaningless. We focus on meaningful coverage: tests that verify behavior, not just execute code. Coverage reports highlight gaps, especially in error handling and edge cases, which we review in PRs. We use a coverage ratchet - coverage can only increase, preventing regression. Critical paths (payments, auth) require 90%+ coverage. We combine coverage with mutation testing for true test quality - coverage shows what ran, mutation testing shows what was tested."

---

## 📚 Table of Contents

1. [Coverage Tools](#1-coverage-tools)
2. [Coverage Reports](#2-coverage-reports)
3. [Setting Coverage Goals](#3-setting-coverage-goals)
4. [CI Integration](#4-ci-integration)
5. [Improving Coverage](#5-improving-coverage)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Coverage Tools

```javascript
// ════════════════════════════════════════════════════════════════
// JEST COVERAGE CONFIGURATION
// jest.config.js
// ════════════════════════════════════════════════════════════════

module.exports = {
  // Enable coverage
  collectCoverage: true,
  
  // Files to collect coverage from
  collectCoverageFrom: [
    'src/**/*.{js,ts}',
    '!src/**/*.d.ts',
    '!src/**/*.test.{js,ts}',
    '!src/**/*.spec.{js,ts}',
    '!src/**/index.{js,ts}',
    '!src/types/**',
  ],
  
  // Coverage output directory
  coverageDirectory: 'coverage',
  
  // Coverage reporters
  coverageReporters: [
    'text',           // Console output
    'text-summary',   // Summary in console
    'html',           // HTML report
    'lcov',           // For CI tools
    'json',           // Machine-readable
  ],
  
  // Coverage thresholds (fail if below)
  coverageThreshold: {
    // Global thresholds
    global: {
      branches: 70,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    // Per-file thresholds
    './src/services/payment/': {
      branches: 90,
      functions: 95,
      lines: 90,
      statements: 90,
    },
    // Specific file
    './src/utils/validation.ts': {
      branches: 100,
      functions: 100,
      lines: 100,
    },
  },
};

// ════════════════════════════════════════════════════════════════
// NYC (ISTANBUL) CONFIGURATION
// .nycrc.json
// ════════════════════════════════════════════════════════════════

{
  "all": true,
  "include": ["src/**/*.ts"],
  "exclude": [
    "**/*.test.ts",
    "**/*.spec.ts",
    "**/node_modules/**",
    "**/dist/**"
  ],
  "reporter": ["text", "html", "lcov"],
  "report-dir": "./coverage",
  "temp-dir": "./.nyc_output",
  "check-coverage": true,
  "branches": 70,
  "lines": 80,
  "functions": 80,
  "statements": 80,
  "watermarks": {
    "lines": [60, 80],
    "functions": [60, 80],
    "branches": [50, 70],
    "statements": [60, 80]
  }
}

// package.json scripts
{
  "scripts": {
    "test": "jest",
    "test:coverage": "jest --coverage",
    "test:coverage:watch": "jest --coverage --watchAll",
    "coverage:report": "open coverage/lcov-report/index.html"
  }
}
```

---

## 2. Coverage Reports

```
═══════════════════════════════════════════════════════════════════
UNDERSTANDING COVERAGE REPORTS
═══════════════════════════════════════════════════════════════════

Console Output (Jest):

------------------------|---------|----------|---------|---------|
File                    | % Stmts | % Branch | % Funcs | % Lines |
------------------------|---------|----------|---------|---------|
All files               |   82.35 |    71.42 |   87.50 |   82.14 |
 src/services           |   85.00 |    75.00 |   90.00 |   85.00 |
  orderService.ts       |   90.00 |    80.00 |  100.00 |   90.00 |
  userService.ts        |   80.00 |    70.00 |   80.00 |   80.00 |
 src/utils              |   75.00 |    60.00 |   80.00 |   75.00 |
  validation.ts         |   60.00 |    40.00 |   60.00 |   60.00 | 15-20,35
  helpers.ts            |   90.00 |    80.00 |  100.00 |   90.00 |
------------------------|---------|----------|---------|---------|

Reading the Report:
• "15-20,35" = Uncovered lines in validation.ts
• Red/Yellow in HTML = Below threshold
• Branch % usually lowest (harder to cover all if/else)


═══════════════════════════════════════════════════════════════════
HTML REPORT SECTIONS
═══════════════════════════════════════════════════════════════════

1. FILE LISTING
   • Click file to see line-by-line coverage
   • Sort by coverage to find gaps

2. LINE COVERAGE VIEW
   • Green: Covered
   • Red: Not covered
   • Yellow: Partially covered (some branches)
   • Number: Times line was executed

3. BRANCH INDICATORS
   • "I" = If branch not taken
   • "E" = Else branch not taken
   • Shows which conditions need tests


Example Line View:
┌─────────────────────────────────────────────────────────────────┐
│ 10  1x │ function validateEmail(email) {                       │
│ 11  1x │   if (!email) {                              [I]      │
│ 12  0x │     return { valid: false, error: 'Required' };       │
│ 13     │   }                                                   │
│ 14  1x │   if (!email.includes('@')) {                [E]      │
│ 15  1x │     return { valid: false, error: 'Invalid' };        │
│ 16     │   }                                                   │
│ 17  0x │   return { valid: true };                             │
│ 18     │ }                                                     │
└─────────────────────────────────────────────────────────────────┘

[I] = if branch not taken (no test with !email)
[E] = else branch not taken (no test with valid email)
0x  = Line never executed
```

---

## 3. Setting Coverage Goals

```yaml
# ════════════════════════════════════════════════════════════════
# COVERAGE GOAL STRATEGY
# ════════════════════════════════════════════════════════════════

coverage_philosophy:
  # Coverage is a TOOL, not a GOAL
  # High coverage doesn't guarantee quality
  # Low coverage definitely indicates gaps

  principles:
    - Coverage shows what was run, not what was tested
    - Assertions matter more than execution
    - 100% coverage is usually not worth the effort
    - Focus on meaningful coverage over numbers

recommended_targets:
  overall:
    lines: 80%
    branches: 70%
    functions: 80%
    statements: 80%

  by_code_type:
    critical_business_logic:
      target: 90%+
      examples:
        - Payment processing
        - Authentication/authorization
        - Financial calculations
        - Data validation

    core_application:
      target: 80%
      examples:
        - Services
        - Repositories
        - Controllers

    utilities:
      target: 70%
      examples:
        - Helpers
        - Formatters
        - Generic utilities

    configuration:
      target: 50%
      examples:
        - Config files
        - Constants
        - Type definitions

  exempt_from_coverage:
    - Generated code
    - Type definitions (.d.ts)
    - Index files (re-exports)
    - Test utilities
    - Mocks

# ════════════════════════════════════════════════════════════════
# COVERAGE RATCHET
# ════════════════════════════════════════════════════════════════

# Prevent coverage regression - can only go up
coverage_ratchet:
  concept: |
    Store current coverage, fail if new coverage is lower.
    Gradually improve coverage over time.
    
  implementation:
    # Store in file committed to repo
    .coverage-thresholds.json:
      branches: 72.5
      functions: 85.2
      lines: 81.3
      statements: 81.0
      
    # CI script
    script: |
      CURRENT_COVERAGE=$(get-current-coverage)
      STORED_COVERAGE=$(cat .coverage-thresholds.json)
      
      if [ $CURRENT_COVERAGE -lt $STORED_COVERAGE ]; then
        echo "Coverage regression! $CURRENT_COVERAGE < $STORED_COVERAGE"
        exit 1
      fi
      
      # Update threshold if coverage improved
      if [ $CURRENT_COVERAGE -gt $STORED_COVERAGE ]; then
        echo $CURRENT_COVERAGE > .coverage-thresholds.json
      fi
```

---

## 4. CI Integration

```yaml
# ════════════════════════════════════════════════════════════════
# GITHUB ACTIONS - COVERAGE WORKFLOW
# ════════════════════════════════════════════════════════════════

name: Test Coverage

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests with coverage
        run: npm run test:coverage

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info
          fail_ci_if_error: true

      - name: Coverage report in PR
        uses: 5monkeys/cobertura-action@master
        if: github.event_name == 'pull_request'
        with:
          path: coverage/cobertura-coverage.xml
          minimum_coverage: 80
          show_line: true
          show_branch: true
          show_missing: true

      - name: Check coverage thresholds
        run: |
          npm run test:coverage -- --coverageThreshold='{"global":{"lines":80,"branches":70}}'

# ════════════════════════════════════════════════════════════════
# CODECOV CONFIGURATION
# codecov.yml
# ════════════════════════════════════════════════════════════════

coverage:
  status:
    project:
      default:
        target: 80%
        threshold: 1%
        if_ci_failed: error
    patch:
      default:
        target: 80%
        threshold: 5%

  # Different targets per folder
  flags:
    services:
      paths:
        - src/services/
      target: 85%
    utils:
      paths:
        - src/utils/
      target: 70%

comment:
  layout: "reach,diff,flags,files"
  behavior: default
  require_changes: true

ignore:
  - "**/*.test.ts"
  - "**/*.spec.ts"
  - "**/types/**"
  - "**/mocks/**"
```

```typescript
// ════════════════════════════════════════════════════════════════
// PR COVERAGE COMMENT EXAMPLE
// ════════════════════════════════════════════════════════════════

/*
## Coverage Report

| Totals | Coverage |
|--------|----------|
| Lines  | 82.5% (+1.2%) ✅ |
| Branches | 71.3% (-0.5%) ⚠️ |
| Functions | 88.0% (+2.0%) ✅ |

### Changed Files

| File | Lines | Branches | Diff |
|------|-------|----------|------|
| src/services/order.ts | 95% | 85% | +10 |
| src/utils/validate.ts | 60% | 40% | +5 |

### Uncovered Lines

`src/utils/validate.ts`: 45-50, 67

⚠️ Branch coverage decreased. Please add tests for:
- Line 67: else branch in `validateInput()`
*/
```

---

## 5. Improving Coverage

```typescript
// ════════════════════════════════════════════════════════════════
// FINDING AND FIXING COVERAGE GAPS
// ════════════════════════════════════════════════════════════════

// ORIGINAL CODE with coverage gaps
function processPayment(order: Order, payment: Payment): Result {
  // Line 1: Covered
  if (!order.items.length) {
    // Line 2-3: NOT COVERED - no test for empty order
    return { success: false, error: 'Empty order' };
  }

  // Line 4: Covered
  const total = calculateTotal(order);

  // Line 5: Covered
  if (total !== payment.amount) {
    // Line 6-7: NOT COVERED - no test for amount mismatch
    return { success: false, error: 'Amount mismatch' };
  }

  // Line 8: Covered
  try {
    // Line 9: Covered
    const result = chargeCard(payment);
    // Line 10: Covered
    return { success: true, transactionId: result.id };
  } catch (error) {
    // Line 11-12: NOT COVERED - no test for payment failure
    return { success: false, error: 'Payment failed' };
  }
}

// TESTS that leave gaps
describe('processPayment', () => {
  it('processes valid payment', () => {
    const result = processPayment(validOrder, validPayment);
    expect(result.success).toBe(true);
  });
});
// Coverage: ~50% - only happy path

// ════════════════════════════════════════════════════════════════
// IMPROVED TESTS - Full Coverage
// ════════════════════════════════════════════════════════════════

describe('processPayment', () => {
  // Happy path
  it('processes valid payment', () => {
    const result = processPayment(validOrder, validPayment);
    expect(result.success).toBe(true);
    expect(result.transactionId).toBeDefined();
  });

  // Cover empty order branch
  it('fails for empty order', () => {
    const emptyOrder = { items: [] };
    const result = processPayment(emptyOrder, validPayment);
    
    expect(result.success).toBe(false);
    expect(result.error).toBe('Empty order');
  });

  // Cover amount mismatch branch
  it('fails when payment amount does not match total', () => {
    const wrongPayment = { ...validPayment, amount: 999 };
    const result = processPayment(validOrder, wrongPayment);
    
    expect(result.success).toBe(false);
    expect(result.error).toBe('Amount mismatch');
  });

  // Cover error handling
  it('handles payment processing failure', () => {
    jest.spyOn(paymentService, 'chargeCard')
      .mockRejectedValue(new Error('Card declined'));
    
    const result = processPayment(validOrder, validPayment);
    
    expect(result.success).toBe(false);
    expect(result.error).toBe('Payment failed');
  });
});
// Coverage: 100%

// ════════════════════════════════════════════════════════════════
// STRATEGIES FOR INCREASING COVERAGE
// ════════════════════════════════════════════════════════════════

strategies:
  1. Error Paths:
     - Test every throw statement
     - Test catch blocks
     - Test validation failures

  2. Branch Coverage:
     - Test both if and else
     - Test all switch cases
     - Test ternary both ways

  3. Edge Cases:
     - Empty arrays/strings
     - Null/undefined
     - Zero values
     - Boundary values

  4. Async Paths:
     - Success responses
     - Error responses
     - Timeout scenarios
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# COVERAGE PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Chasing 100% coverage
# Bad
# Spend days testing every line
# Test obvious getters/setters
# Test framework code

# Good
# Focus on critical code (90%+)
# Accept 80% overall
# Don't test trivial code

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Coverage without assertions
# Bad
test('runs processOrder', () => {
  processOrder(order);  // Just runs, doesn't check
});
# 100% coverage, 0% confidence

# Good
test('processOrder calculates correct total', () => {
  const result = processOrder(order);
  expect(result.total).toBe(150);
  expect(result.status).toBe('processed');
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Testing implementation details for coverage
# Bad
test('calls internal _calculateTax method', () => {
  const spy = jest.spyOn(service, '_calculateTax');
  service.processOrder(order);
  expect(spy).toHaveBeenCalled();
});
# Brittle, tests implementation not behavior

# Good
test('order total includes tax', () => {
  const result = service.processOrder(order);
  expect(result.total).toBe(subtotal * 1.1);  // 10% tax
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Ignoring branch coverage
# Bad
# Only tracking line coverage
# 90% lines but 50% branches

# Good
# Track and threshold branch coverage
# If/else paths matter
# Usually hardest metric to improve

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Static thresholds that never change
# Bad
threshold: 50%  # Set 2 years ago, never updated

# Good
# Use coverage ratchet
# Gradually increase thresholds
# Review and update quarterly

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Excluding too much code
# Bad
collectCoverageFrom: ['src/services/**']
# Only services covered, utils/helpers ignored

# Good
# Include everything by default
# Explicitly exclude only: types, mocks, test files
# Review exclusions periodically
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is code coverage?"**
> "Code coverage measures what percentage of your code is executed during tests. Types: line coverage (lines run), branch coverage (if/else paths), function coverage (functions called). It shows what was run, not what was tested - high coverage without assertions is meaningless."

**Q: "What coverage percentage should you aim for?"**
> "80% line coverage is industry standard. 70% branch coverage (harder to achieve). 90%+ for critical code (payments, auth). But don't obsess over numbers - meaningful tests with assertions matter more than coverage percentage. 100% is usually not worth the effort."

**Q: "What's the difference between line and branch coverage?"**
> "Line coverage: Was this line executed? Branch coverage: Were all paths through conditionals taken? You can have 100% line coverage but 50% branch if you only test one side of an if/else. Branch coverage is more thorough and usually harder to achieve."

### Intermediate Questions

**Q: "Why is high coverage not enough?"**
> "Coverage only shows execution, not verification. Tests without assertions have high coverage but catch no bugs. Example: `add(2, 3)` gives 100% coverage but doesn't verify the result. A bug (a - b instead of a + b) wouldn't be caught. Always pair coverage with meaningful assertions."

**Q: "How do you use coverage reports effectively?"**
> "Review reports to find gaps, not to chase numbers. Focus on: uncovered error handling, missing branch coverage, critical paths without tests. In PRs, review coverage diff for new code. Use HTML reports to see exactly which lines and branches are missing."

**Q: "What is a coverage ratchet?"**
> "A mechanism that prevents coverage regression. Store current coverage as threshold, fail CI if coverage drops. Coverage can only stay same or increase. Gradually improves coverage over time without allowing regression. Good for legacy codebases."

### Advanced Questions

**Q: "How do you balance coverage vs. speed?"**
> "Don't aim for 100% everywhere - diminishing returns. High coverage (90%+) for critical paths. Acceptable coverage (80%) for core logic. Lower coverage (70%) okay for utilities. Skip trivial code. Focus testing effort where bugs are costly. Use mutation testing for true quality."

**Q: "Coverage vs. mutation testing?"**
> "Coverage: What code ran? Mutation: Were changes detected? You can have 100% coverage with 60% mutation score. Mutation testing reveals weak tests that run code but don't verify it. Use both: coverage for quantity, mutation for quality. Coverage is faster, mutation more thorough."

**Q: "How do you handle legacy code with low coverage?"**
> "Use coverage ratchet - can't decrease coverage. Add tests for new code (80%+ threshold). When modifying legacy code, add tests for changed areas. Identify critical paths, prioritize coverage there. Don't try to cover everything at once - incremental improvement."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  COVERAGE CHECKLIST                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SETUP:                                                         │
│  □ Configure coverage tool (Istanbul/Jest)                     │
│  □ Set thresholds (80% lines, 70% branches)                    │
│  □ Configure CI coverage reports                               │
│  □ Set up Codecov/Coveralls                                    │
│                                                                 │
│  MEANINGFUL COVERAGE:                                           │
│  □ Tests have assertions                                       │
│  □ Error paths are tested                                      │
│  □ All branches covered                                        │
│  □ Edge cases included                                         │
│                                                                 │
│  TARGETS BY CODE TYPE:                                          │
│  □ Critical (payment, auth): 90%+                              │
│  □ Core logic: 80%+                                            │
│  □ Utilities: 70%+                                             │
│  □ Config/types: Exclude                                       │
│                                                                 │
│  AVOID:                                                         │
│  □ Chasing 100%                                                │
│  □ Coverage without assertions                                 │
│  □ Testing implementation details                              │
│  □ Static thresholds forever                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

COVERAGE TYPES:
┌────────────────────────────────────────────────────────────────┐
│ Lines:      % of code lines executed                           │
│ Branches:   % of if/else paths taken                           │
│ Functions:  % of functions called                              │
│ Statements: % of statements executed                           │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

