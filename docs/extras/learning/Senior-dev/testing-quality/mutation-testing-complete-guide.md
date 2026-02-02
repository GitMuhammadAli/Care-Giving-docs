# 🧬 Mutation Testing - Complete Guide

> A comprehensive guide to mutation testing - measuring test quality, mutation score, and ensuring your tests can actually catch bugs.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Mutation testing evaluates test quality by introducing small bugs (mutations) into your code and checking if tests detect them - a test suite that can't catch intentional bugs certainly can't catch real ones."

### The 7 Key Concepts (Remember These!)
```
1. MUTANT         → Code with intentionally introduced bug
2. MUTATION       → The change made to create a mutant
3. KILLED         → Mutant detected by failing test ✓
4. SURVIVED       → Mutant NOT detected by tests ✗
5. MUTATION SCORE → % of mutants killed (quality metric)
6. EQUIVALENT     → Mutant that doesn't change behavior
7. MUTATION OPERATOR → Type of change (e.g., > to >=)
```

### Mutation Testing Flow
```
┌─────────────────────────────────────────────────────────────────┐
│                  MUTATION TESTING FLOW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. ORIGINAL CODE                                              │
│     ┌─────────────────────────────────────┐                    │
│     │ if (age >= 18) {                    │                    │
│     │   return "adult";                   │                    │
│     │ }                                   │                    │
│     └─────────────────────────────────────┘                    │
│                      │                                          │
│                      ▼                                          │
│  2. CREATE MUTANTS (introduce bugs)                            │
│     ┌─────────────────────────────────────┐                    │
│     │ Mutant 1: if (age > 18)   // >= → > │                    │
│     │ Mutant 2: if (age <= 18)  // >= → <=│                    │
│     │ Mutant 3: if (age >= 19)  // 18 → 19│                    │
│     │ Mutant 4: if (true)       // remove │                    │
│     └─────────────────────────────────────┘                    │
│                      │                                          │
│                      ▼                                          │
│  3. RUN TESTS AGAINST EACH MUTANT                              │
│     ┌─────────────────────────────────────┐                    │
│     │ Mutant 1 (age > 18):                │                    │
│     │   Test: verify(17) → "minor" ✓      │                    │
│     │   Test: verify(18) → "adult"        │                    │
│     │   MUTANT SURVIVES! Tests pass! ✗    │                    │
│     │                                     │                    │
│     │ Mutant 2 (age <= 18):               │                    │
│     │   Test: verify(18) → "adult"        │                    │
│     │   MUTANT KILLED! Test fails! ✓     │                    │
│     └─────────────────────────────────────┘                    │
│                      │                                          │
│                      ▼                                          │
│  4. CALCULATE MUTATION SCORE                                   │
│     ┌─────────────────────────────────────┐                    │
│     │ Killed: 3 / Total: 4 = 75% score    │                    │
│     │                                     │                    │
│     │ Missing test for boundary:          │                    │
│     │ test("age exactly 18 is adult")     │                    │
│     └─────────────────────────────────────┘                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Common Mutation Operators
```
┌─────────────────────────────────────────────────────────────────┐
│                  MUTATION OPERATORS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  RELATIONAL OPERATORS                                          │
│  • > → >= , < , <= , ==                                        │
│  • >= → > , < , <= , ==                                        │
│  • == → != , < , > , <= , >=                                   │
│                                                                 │
│  ARITHMETIC OPERATORS                                          │
│  • + → - , * , /                                               │
│  • * → / , + , -                                               │
│  • ++ → --                                                     │
│                                                                 │
│  LOGICAL OPERATORS                                             │
│  • && → ||                                                     │
│  • || → &&                                                     │
│  • ! → (remove)                                                │
│                                                                 │
│  LITERAL MUTATIONS                                             │
│  • true → false                                                │
│  • 0 → 1, -1                                                   │
│  • "string" → ""                                               │
│                                                                 │
│  STATEMENT MUTATIONS                                           │
│  • Remove statement entirely                                   │
│  • Remove function call                                        │
│  • Empty return (return;)                                      │
│                                                                 │
│  CONDITIONAL MUTATIONS                                         │
│  • if (cond) → if (true)                                       │
│  • if (cond) → if (false)                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Mutation score"** | "We maintain 80%+ mutation score on critical code" |
| **"Killed mutant"** | "Tests killed 95 of 100 mutants" |
| **"Survived mutant"** | "5 survived mutants indicate missing test cases" |
| **"Equivalent mutant"** | "Some survivors are equivalent mutants - no behavior change" |
| **"Test strength"** | "Mutation testing measures actual test strength" |
| **"Stryker"** | "We use Stryker for JavaScript mutation testing" |

### Key Numbers to Remember
| Metric | Target | Notes |
|--------|--------|-------|
| Mutation score | **> 80%** | For critical code |
| Runtime | **10-100x** slower | Than normal tests |
| Mutants per file | **50-500** | Depends on code complexity |
| Score vs coverage | **Score < Coverage** | Typically 10-20% lower |

### The "Wow" Statement (Memorize This!)
> "We use mutation testing with Stryker on critical business logic. While our code coverage is 90%, mutation score revealed it was only 72% - tests were covering lines without actually validating behavior. Survived mutants showed boundary conditions we missed: changing > to >= survived because we never tested the exact boundary value. We added targeted tests for each survivor, bringing mutation score to 85%. We run mutation tests nightly (too slow for every PR) and track score over time. It's invaluable for payment processing, authorization logic, and calculations where bugs are costly. Coverage tells you what you ran, mutation testing tells you what you tested."

---

## 📚 Table of Contents

1. [Stryker Setup](#1-stryker-setup)
2. [Understanding Results](#2-understanding-results)
3. [Improving Mutation Score](#3-improving-mutation-score)
4. [CI Integration](#4-ci-integration)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Stryker Setup

```javascript
// ════════════════════════════════════════════════════════════════
// STRYKER CONFIGURATION
// stryker.conf.mjs
// ════════════════════════════════════════════════════════════════

/** @type {import('@stryker-mutator/api/core').PartialStrykerOptions} */
export default {
  // Files to mutate
  mutate: [
    'src/**/*.ts',
    '!src/**/*.spec.ts',     // Exclude test files
    '!src/**/*.test.ts',
    '!src/**/*.d.ts',        // Exclude type definitions
    '!src/index.ts',         // Exclude entry points
  ],

  // Test runner
  testRunner: 'jest',
  jest: {
    configFile: 'jest.config.js',
    enableFindRelatedTests: true, // Speed up by running related tests only
  },

  // Mutation operators to use
  mutator: {
    excludedMutations: [
      'StringLiteral',  // Don't mutate strings (often not important)
    ],
  },

  // Reporter options
  reporters: ['html', 'progress', 'clear-text'],
  htmlReporter: {
    fileName: 'reports/mutation.html',
  },

  // Performance options
  concurrency: 4, // Parallel test runs
  timeoutMS: 60000, // Timeout per mutant
  timeoutFactor: 1.5, // Multiplier for slow tests

  // Coverage analysis for speed
  coverageAnalysis: 'perTest', // Only run tests that cover mutated code

  // Thresholds
  thresholds: {
    high: 80,
    low: 60,
    break: 50, // Fail if below this
  },

  // Incremental mode (cache results)
  incremental: true,
  incrementalFile: '.stryker-cache/incremental.json',
};

// ════════════════════════════════════════════════════════════════
// PACKAGE.JSON SCRIPTS
// ════════════════════════════════════════════════════════════════

{
  "scripts": {
    "test:mutation": "stryker run",
    "test:mutation:incremental": "stryker run --incremental",
    "test:mutation:file": "stryker run --mutate 'src/services/payment.ts'"
  },
  "devDependencies": {
    "@stryker-mutator/core": "^7.0.0",
    "@stryker-mutator/jest-runner": "^7.0.0",
    "@stryker-mutator/typescript-checker": "^7.0.0"
  }
}
```

---

## 2. Understanding Results

```typescript
// ════════════════════════════════════════════════════════════════
// EXAMPLE: ANALYZING MUTATION RESULTS
// ════════════════════════════════════════════════════════════════

// Original code
function calculateDiscount(price: number, customerType: string): number {
  if (price <= 0) {
    throw new Error('Price must be positive');
  }

  let discount = 0;

  if (customerType === 'premium') {
    discount = 0.2;
  } else if (customerType === 'regular' && price > 100) {
    discount = 0.1;
  }

  return price * (1 - discount);
}

// Tests
describe('calculateDiscount', () => {
  it('returns full price for basic customer', () => {
    expect(calculateDiscount(100, 'basic')).toBe(100);
  });

  it('gives 20% discount for premium', () => {
    expect(calculateDiscount(100, 'premium')).toBe(80);
  });

  it('throws for zero price', () => {
    expect(() => calculateDiscount(0, 'basic')).toThrow();
  });
});

// ════════════════════════════════════════════════════════════════
// STRYKER OUTPUT
// ════════════════════════════════════════════════════════════════

/*
All mutants:
┌─────────────────────────────────────────────────────────────────┐
│ #1 KILLED   - price <= 0 → price < 0                           │
│ #2 SURVIVED - price > 100 → price >= 100 ← PROBLEM!            │
│ #3 KILLED   - discount = 0.2 → discount = 0                    │
│ #4 SURVIVED - discount = 0.1 → discount = 0 ← PROBLEM!         │
│ #5 KILLED   - customerType === 'premium' → false               │
│ #6 SURVIVED - && → || ← PROBLEM!                               │
│ #7 KILLED   - return price * (1 - discount) → return 0         │
└─────────────────────────────────────────────────────────────────┘

Mutation score: 57% (4/7 killed)

SURVIVED MUTANTS ANALYSIS:

#2: price > 100 → price >= 100
    Missing test: What happens when price is exactly 100?
    Need: test("regular customer at $100 gets no discount")

#4: discount = 0.1 → discount = 0
    Missing test: No test verifies regular customer discount
    Need: test("regular customer over $100 gets 10% off")

#6: && → ||
    Missing test: Doesn't test the combined condition
    Need: test("regular customer under $100 gets no discount")
*/

// ════════════════════════════════════════════════════════════════
// IMPROVED TESTS (Kill Survivors)
// ════════════════════════════════════════════════════════════════

describe('calculateDiscount - comprehensive', () => {
  // Existing tests...

  // Kill #2: Test boundary condition
  it('regular customer at exactly $100 gets no discount', () => {
    expect(calculateDiscount(100, 'regular')).toBe(100);
  });

  // Kill #4: Verify regular discount actually applies
  it('regular customer over $100 gets 10% discount', () => {
    expect(calculateDiscount(150, 'regular')).toBe(135);
  });

  // Kill #6: Test && condition separately
  it('regular customer under $100 gets no discount', () => {
    expect(calculateDiscount(50, 'regular')).toBe(50);
  });
});

// New mutation score: 100% (7/7 killed)
```

---

## 3. Improving Mutation Score

```typescript
// ════════════════════════════════════════════════════════════════
// STRATEGIES FOR KILLING MUTANTS
// ════════════════════════════════════════════════════════════════

// 1. BOUNDARY VALUE TESTING
// Mutants: > → >=, < → <=, etc.

// Bad: Only tests well inside boundaries
test('adult check', () => {
  expect(isAdult(25)).toBe(true);
  expect(isAdult(10)).toBe(false);
});

// Good: Tests exact boundaries
test('adult check - boundaries', () => {
  expect(isAdult(17)).toBe(false);  // Just below
  expect(isAdult(18)).toBe(true);   // Exact boundary
  expect(isAdult(19)).toBe(true);   // Just above
});

// 2. ASSERTION SPECIFICITY
// Mutants: Change return values

// Bad: Only checks truthiness
test('calculate total', () => {
  expect(calculateTotal(items)).toBeTruthy();
});

// Good: Checks exact value
test('calculate total', () => {
  expect(calculateTotal([
    { price: 10, qty: 2 },
    { price: 5, qty: 3 },
  ])).toBe(35);
});

// 3. CONDITIONAL COVERAGE
// Mutants: && → ||, if(x) → if(true)

// Bad: Only tests one path
test('premium with high value', () => {
  expect(getDiscount('premium', 1000)).toBe(0.2);
});

// Good: Tests all combinations
test('discount logic', () => {
  expect(getDiscount('premium', 1000)).toBe(0.2);  // premium, high value
  expect(getDiscount('premium', 50)).toBe(0.2);    // premium, low value
  expect(getDiscount('regular', 1000)).toBe(0.1);  // regular, high value
  expect(getDiscount('regular', 50)).toBe(0);      // regular, low value
});

// 4. ARITHMETIC MUTATIONS
// Mutants: + → -, * → /

// Bad: Doesn't validate calculation
test('applies tax', () => {
  const result = applyTax(100, 0.1);
  expect(result).toBeGreaterThan(100);
});

// Good: Validates exact calculation
test('applies tax correctly', () => {
  expect(applyTax(100, 0.1)).toBe(110);  // 100 * 1.1
  expect(applyTax(100, 0.2)).toBe(120);  // Verify multiplier
  expect(applyTax(50, 0.1)).toBe(55);    // Verify base
});

// 5. STRING MUTATIONS
// Mutants: "value" → ""

// Bad: Checks existence only
test('generates greeting', () => {
  expect(greet('John')).toBeTruthy();
});

// Good: Verifies content
test('generates greeting', () => {
  expect(greet('John')).toBe('Hello, John!');
  expect(greet('Jane')).toContain('Jane');
});
```

---

## 4. CI Integration

```yaml
# ════════════════════════════════════════════════════════════════
# GITHUB ACTIONS - MUTATION TESTING
# ════════════════════════════════════════════════════════════════

name: Mutation Testing

on:
  # Run nightly (too slow for every PR)
  schedule:
    - cron: '0 2 * * *'
  # Manual trigger
  workflow_dispatch:
  # Run on main branch merges
  push:
    branches: [main]

jobs:
  mutation-test:
    runs-on: ubuntu-latest
    timeout-minutes: 60
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Restore Stryker cache
        uses: actions/cache@v3
        with:
          path: .stryker-cache
          key: stryker-${{ hashFiles('src/**/*.ts') }}
          restore-keys: stryker-

      - name: Run mutation tests
        run: npm run test:mutation -- --incremental

      - name: Upload mutation report
        uses: actions/upload-artifact@v3
        with:
          name: mutation-report
          path: reports/mutation.html

      - name: Check mutation score
        run: |
          SCORE=$(cat reports/mutation.json | jq '.mutationScore')
          if (( $(echo "$SCORE < 70" | bc -l) )); then
            echo "Mutation score $SCORE% is below threshold of 70%"
            exit 1
          fi

      - name: Post results to PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const report = JSON.parse(fs.readFileSync('reports/mutation.json'));
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: `## Mutation Testing Results
              - Score: ${report.mutationScore.toFixed(2)}%
              - Killed: ${report.killed}
              - Survived: ${report.survived}
              - Timeout: ${report.timeout}`
            });

# ════════════════════════════════════════════════════════════════
# INCREMENTAL MUTATION TESTING FOR PRs
# ════════════════════════════════════════════════════════════════

name: PR Mutation Check

on:
  pull_request:
    paths:
      - 'src/services/payment/**'
      - 'src/services/auth/**'

jobs:
  mutation-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Get changed files
        id: changed
        run: |
          FILES=$(git diff --name-only origin/main...HEAD | grep -E '^src/.*\.ts$' | grep -v '.spec.ts' | tr '\n' ',')
          echo "files=$FILES" >> $GITHUB_OUTPUT

      - name: Run mutation on changed files
        if: steps.changed.outputs.files != ''
        run: |
          npm run test:mutation -- --mutate '${{ steps.changed.outputs.files }}'
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# MUTATION TESTING BEST PRACTICES
# ════════════════════════════════════════════════════════════════

when_to_use:
  recommended:
    - Critical business logic (payments, auth)
    - Complex algorithms
    - Financial calculations
    - Security-sensitive code
    - Core domain logic

  not_recommended:
    - UI components (visual testing better)
    - Simple CRUD operations
    - Generated code
    - Third-party integrations
    - Entire codebase (too slow)

optimization_strategies:
  speed:
    - Use incremental mode
    - Run on changed files only
    - Configure test timeout
    - Limit concurrency based on CI resources
    - Exclude trivial mutations (StringLiteral)

  focus:
    - Target high-risk code
    - Set realistic thresholds per area
    - Don't aim for 100% everywhere

  workflow:
    - Nightly runs for full suite
    - PR checks for changed files
    - Track score over time
    - Review survived mutants, not just score

handling_equivalent_mutants:
  # Equivalent mutants: behavior identical to original
  # Example: x = x * 1 → x = x / 1 (same result)
  
  strategies:
    - Mark as ignored if truly equivalent
    - Review if mutation reveals dead code
    - Some survived mutants are acceptable
    
threshold_guidelines:
  critical_code: 80%+
  core_logic: 70%+
  utilities: 60%+
  overall: 60%+ (don't block below)
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# MUTATION TESTING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Running on entire codebase
# Bad
# Mutate everything, takes 4 hours
stryker run

# Good
# Target critical code
stryker run --mutate 'src/services/payment/**'

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Chasing 100% mutation score
# Bad
# Every mutation must be killed!
# Waste time on trivial mutations

# Good
# Focus on meaningful mutations
# 80% on critical, 60% elsewhere
# Accept some equivalent mutants

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Ignoring survived mutants
# Bad
# "Score is 75%, good enough"
# Don't investigate survivors

# Good
# Review each survivor
# Ask: "Is this a real gap?"
# Either add test or mark equivalent

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Slow tests make mutation testing painful
# Bad
# Each test takes 30 seconds
# 500 mutants × 30 seconds = 4+ hours

# Good
# Optimize test speed first
# Use mocks appropriately
# Enable perTest coverage analysis

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Blocking PRs on mutation score
# Bad
# PR fails if mutation score drops
# Developers frustrated, disable mutation testing

# Good
# Run nightly, report results
# Block only for critical code
# Use as guidance, not gate

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Testing mutations, not behavior
# Bad
# Write test specifically to kill mutant
test('kill > to >= mutation', () => {
  expect(isAdult(18)).toBe(true);  // Only for mutation
});

# Good
# Write meaningful tests that happen to kill mutants
test('18 year old is considered adult', () => {
  expect(isAdult(18)).toBe(true);  // Business requirement
});
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is mutation testing?"**
> "Mutation testing introduces intentional bugs (mutations) into code and checks if tests detect them. A killed mutant means tests caught the bug. A surviving mutant indicates a gap in test coverage. Mutation score = killed / total mutants. It measures test quality, not just coverage."

**Q: "How is mutation score different from code coverage?"**
> "Code coverage tells you which lines were executed. Mutation score tells you if those lines were actually tested. You can have 100% coverage with weak assertions that miss bugs. Mutation testing verifies tests can detect changes. Typically mutation score is 10-20% lower than coverage."

**Q: "What is a killed vs survived mutant?"**
> "Killed: A test failed when mutation was introduced - the test detected the bug. Good! Survived: All tests passed despite the mutation - tests didn't detect the bug. Bad - indicates missing or weak test. Goal is to kill all meaningful mutants."

### Intermediate Questions

**Q: "What are equivalent mutants?"**
> "Mutations that don't actually change behavior. Example: `x * 1` → `x / 1` produces same result. These survive but aren't test gaps. They're noise in mutation testing. Tools try to detect them, but some manual review needed. Don't chase killing equivalent mutants."

**Q: "When should you use mutation testing?"**
> "For critical code: payment processing, authentication, business rules, algorithms. Not for everything - too slow and expensive. Run nightly, not on every PR. Target high-risk areas. 80%+ score for critical code, 60%+ elsewhere. Use to find gaps, not as a gate."

**Q: "How do you handle slow mutation tests?"**
> "Optimization: Use incremental mode (cache results). Run only on changed files. Configure proper timeouts. Enable perTest coverage (only run relevant tests). Exclude trivial mutations (strings). Parallelize. Run nightly, not on every commit."

### Advanced Questions

**Q: "How do you integrate mutation testing in CI/CD?"**
> "Nightly scheduled runs for full suite. Upload reports as artifacts. Track score over time. For PRs, run only on changed critical files. Don't block PRs on score (causes frustration). Alert on significant drops. Store historical data for trends."

**Q: "How do you improve mutation score effectively?"**
> "Review surviving mutants individually. Focus on: boundary conditions (> vs >=), return value assertions (exact values, not just truthy), conditional logic (all branches). Write tests for business requirements - they naturally kill mutants. Don't write tests just to kill mutants."

**Q: "Mutation testing vs other test quality metrics?"**
> "Coverage: Lines executed (quantity). Mutation: Bugs detected (quality). Both needed. High coverage + low mutation = weak tests. Branch coverage: Paths taken. Mutation goes deeper - tests logic correctness. Use mutation for critical code, coverage for general."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│               MUTATION TESTING CHECKLIST                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SETUP:                                                         │
│  □ Install Stryker (or equivalent)                             │
│  □ Configure mutate patterns                                   │
│  □ Set up incremental caching                                  │
│  □ Configure thresholds                                        │
│                                                                 │
│  STRATEGY:                                                      │
│  □ Target critical code first                                  │
│  □ Run nightly, not every PR                                   │
│  □ Review survivors, don't just track score                    │
│                                                                 │
│  IMPROVING SCORE:                                               │
│  □ Test boundary values                                        │
│  □ Assert exact values, not truthy                             │
│  □ Cover all conditional branches                              │
│  □ Write meaningful tests, not mutation killers                │
│                                                                 │
│  TARGETS:                                                       │
│  □ Critical code: 80%+                                         │
│  □ Core logic: 70%+                                            │
│  □ General: 60%+                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

STRYKER QUICK START:
┌────────────────────────────────────────────────────────────────┐
│ npm i -D @stryker-mutator/core @stryker-mutator/jest-runner   │
│ npx stryker init                                               │
│ npx stryker run                                                │
│ npx stryker run --mutate 'src/critical/**'                     │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

