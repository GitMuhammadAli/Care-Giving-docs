# Code Review Best Practices - Complete Guide

> **MUST REMEMBER**: Code reviews are about code, not people. Focus on: correctness, maintainability, security, performance. Give constructive feedback with suggestions (not commands). As author: small PRs, good descriptions, self-review first. As reviewer: timely response, ask questions, praise good code. Use checklists for consistency. Automate what you can (linting, formatting, tests).

---

## How to Explain Like a Senior Developer

"Code review is one of the highest-leverage activities in engineering. Done well, it catches bugs, spreads knowledge, maintains quality standards, and mentors developers. Done poorly, it creates friction and blocks progress. The key is making it about the code, not the person. Instead of 'You did this wrong,' say 'This approach might cause X issue - what about Y?' As an author, make reviewers' lives easy: small PRs, clear descriptions, self-review first. As a reviewer, be timely (blocking someone for days is worse than imperfect feedback), ask questions instead of making demands, and acknowledge good work. Automate the nitpicks (formatting, linting) so humans focus on logic and design."

---

## Core Practices

### Creating Reviewable Pull Requests

```markdown
## PR Template

### What does this PR do?
[Clear description of the change and WHY it's needed]

### Type of change
- [ ] Bug fix (non-breaking change that fixes an issue)
- [ ] New feature (non-breaking change that adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to change)
- [ ] Refactoring (no functional changes)
- [ ] Documentation update

### How to test
1. [Step-by-step testing instructions]
2. [Include relevant test data or scenarios]

### Screenshots (if applicable)
[Before/After for UI changes]

### Checklist
- [ ] I have performed a self-review
- [ ] I have added tests that prove my fix/feature works
- [ ] New and existing tests pass locally
- [ ] I have updated documentation as needed
- [ ] My changes don't introduce new warnings

### Related issues
Closes #123
```

### Self-Review Checklist

```typescript
// self-review-checklist.ts

/**
 * Before requesting review, ask yourself:
 */

const selfReviewChecklist = {
  
  // Correctness
  correctness: [
    'Does the code do what the ticket/description says?',
    'Have I tested the happy path AND edge cases?',
    'Are there any obvious bugs I can spot?',
    'Does it work with existing data/state?',
  ],
  
  // Code Quality
  quality: [
    'Is the code readable without my context?',
    'Are variable/function names clear?',
    'Is there any duplicated code I can extract?',
    'Are functions small and focused?',
    'Did I remove console.logs and debug code?',
  ],
  
  // Testing
  testing: [
    'Did I add/update tests?',
    'Do tests cover edge cases?',
    'Are tests readable and maintainable?',
    'Do all tests pass?',
  ],
  
  // Security
  security: [
    'Am I exposing sensitive data?',
    'Is user input validated?',
    'Are there any SQL injection / XSS vulnerabilities?',
    'Are permissions checked correctly?',
  ],
  
  // Performance
  performance: [
    'Are there any obvious N+1 queries?',
    'Am I loading more data than needed?',
    'Are there any memory leaks?',
    'Could this cause performance issues at scale?',
  ],
  
  // Documentation
  documentation: [
    'Are complex parts commented?',
    'Did I update relevant documentation?',
    'Are API changes documented?',
  ],
};
```

### Giving Effective Feedback

```markdown
## Feedback Guidelines

### ❌ Bad Feedback Examples

"This is wrong."
→ Not constructive, doesn't explain why or suggest alternative

"Why would you do it this way?"
→ Sounds accusatory, could be genuine question but tone unclear

"Just use X instead."
→ Commanding, doesn't explain reasoning

"This code is messy."
→ Vague, subjective, not actionable


### ✅ Good Feedback Examples

"This could cause a race condition when two users submit simultaneously. 
Consider using a database transaction or optimistic locking. Here's an example: [link]"
→ Explains the issue, suggests solution, provides resources

"I'm not sure I understand the intent here - could you add a comment 
explaining why we need this check?"
→ Asks for clarification, assumes positive intent

"Nit: This could be simplified using optional chaining: `user?.profile?.name`"
→ Labels as minor (nit), shows the alternative

"I love how you extracted this into a reusable hook! 
One suggestion: consider adding error handling for the API call."
→ Acknowledges good work, then adds constructive feedback

"Question: What happens if `items` is empty here? 
I see we're accessing `items[0]` without checking."
→ Frames as question, identifies specific concern
```

### Code Review Response Types

```typescript
// review-comment-types.ts

/**
 * Prefix your comments to set expectations
 */

type CommentType = {
  prefix: string;
  meaning: string;
  expectation: string;
};

const commentTypes: CommentType[] = [
  {
    prefix: 'Blocker:',
    meaning: 'Must be fixed before merge',
    expectation: 'Author should address and re-request review',
  },
  {
    prefix: 'Suggestion:',
    meaning: 'Consider this improvement',
    expectation: 'Author can accept, discuss, or decline with reason',
  },
  {
    prefix: 'Nit:',
    meaning: 'Minor issue, optional to fix',
    expectation: 'Author can fix or ignore; don\'t block merge',
  },
  {
    prefix: 'Question:',
    meaning: 'I need clarification to review properly',
    expectation: 'Author should explain (comment or in-code)',
  },
  {
    prefix: 'FYI:',
    meaning: 'Information for author, no action needed',
    expectation: 'Acknowledge or continue',
  },
  {
    prefix: 'Praise:',
    meaning: 'This is great!',
    expectation: 'Keep doing this',
  },
];

// Example usage in review:
/*
Blocker: This SQL query is vulnerable to injection. 
Use parameterized queries instead.

Suggestion: This logic is duplicated in UserService. 
Consider extracting to a shared utility.

Nit: Prefer `const` over `let` here since the value never changes.

Question: Is there a reason we're not using the existing 
`validateEmail()` helper?

Praise: Great test coverage! The edge cases you've considered
are exactly what we need.
*/
```

---

## Real-World Scenarios

### Scenario 1: Reviewing a Large PR

```typescript
// large-pr-strategy.ts

/**
 * When you receive a 500+ line PR:
 */

const largeePRStrategy = {
  
  // 1. Don't review it all at once
  approach: 'Review in passes, not all at once',
  
  passes: [
    {
      pass: 1,
      focus: 'High-level architecture',
      questions: [
        'Does the overall approach make sense?',
        'Are there major design issues?',
        'Is this the right place for this code?',
      ],
      action: 'Comment on architecture before diving into details',
    },
    {
      pass: 2,
      focus: 'Logic and correctness',
      questions: [
        'Does the logic handle all cases?',
        'Are there edge cases missing?',
        'Any obvious bugs?',
      ],
    },
    {
      pass: 3,
      focus: 'Code quality and style',
      questions: [
        'Is the code readable?',
        'Are names clear?',
        'Any code smells?',
      ],
    },
  ],
  
  // Alternative: Ask author to split PR
  splitRequest: `
    This PR touches several concerns (auth, user service, database). 
    Would you be open to splitting into smaller PRs? It would help:
    - Get faster reviews for each piece
    - Easier to revert if needed
    - Better git history
    
    Suggested split:
    1. Database migration
    2. User service changes
    3. Auth updates
  `,
};
```

### Scenario 2: Disagreeing with Approach

```typescript
// handling-disagreement.ts

/**
 * When you fundamentally disagree with the approach:
 */

// ❌ DON'T: Block without discussion
// "This approach is wrong. Don't merge."

// ✅ DO: Express concerns constructively
const constructiveDisagreement = `
I have some concerns about this approach that I'd like to discuss 
before we proceed.

My understanding of the requirements:
- We need to sync user data in real-time
- Performance is critical (< 100ms latency)

My concern with the polling approach:
- Polling every second means 86,400 requests/day per user
- At scale (10k users), that's 864M requests/day
- Database load could become a bottleneck

Alternative I'd suggest exploring:
- WebSocket for real-time updates (push instead of pull)
- Or CDC (Change Data Capture) for database changes

Happy to discuss in a call if that's easier! I might be missing 
context that makes polling the right choice here.
`;

// Key principles:
// 1. Assume you might be wrong
// 2. Explain your reasoning
// 3. Offer alternatives
// 4. Offer to discuss synchronously
// 5. Stay respectful
```

### Scenario 3: Receiving Critical Feedback

```typescript
// receiving-feedback.ts

/**
 * How to respond to review comments professionally
 */

const respondingToFeedback = {
  
  // Pause before responding if frustrated
  rule1: 'Take a breath. They\'re reviewing code, not you.',
  
  // ❌ Defensive responses
  bad: [
    'It works, doesn\'t it?',
    'That\'s how I\'ve always done it.',
    'The deadline is tomorrow, no time to change.',
    'You just don\'t understand the requirements.',
  ],
  
  // ✅ Professional responses
  good: [
    // Agree and fix
    'Good catch! Fixed in the latest commit.',
    
    // Clarify
    'I chose this approach because [reason]. Does that context change your suggestion?',
    
    // Discuss
    'Interesting point. I see the tradeoff differently - can we discuss?',
    
    // Accept nits selectively
    'I\'ll address the blocker comments now. I\'ll create a follow-up ticket for the style nits to not block this release.',
    
    // Ask for help
    'I\'m not sure how to implement your suggestion. Could you point me to an example?',
  ],
  
  // When you disagree
  disagree: `
    Thanks for the feedback. I see where you're coming from, but 
    I'd like to share my perspective:
    
    [Explain your reasoning with specific points]
    
    I'm open to changing if you still think your approach is better 
    after hearing this. Or should we get a third opinion?
  `,
};
```

---

## Review Checklist by Area

### Security Review

```markdown
## Security Checklist

### Authentication & Authorization
- [ ] Are all endpoints properly authenticated?
- [ ] Are authorization checks in place for sensitive operations?
- [ ] Are session tokens handled securely?

### Input Validation
- [ ] Is all user input validated?
- [ ] Are there SQL injection vulnerabilities?
- [ ] Are there XSS vulnerabilities in rendered content?
- [ ] Is file upload validated (type, size)?

### Data Protection
- [ ] Is sensitive data encrypted at rest?
- [ ] Is sensitive data masked in logs?
- [ ] Are API responses leaking unnecessary data?
- [ ] Are secrets in environment variables (not code)?

### Dependencies
- [ ] Are new dependencies from trusted sources?
- [ ] Are there known vulnerabilities in dependencies?
```

### Performance Review

```markdown
## Performance Checklist

### Database
- [ ] Are queries using indexes appropriately?
- [ ] Are there N+1 query problems?
- [ ] Is data fetched lazily when appropriate?
- [ ] Are expensive queries cached?

### API
- [ ] Is pagination implemented for list endpoints?
- [ ] Are responses appropriately sized?
- [ ] Is caching used where beneficial?

### Frontend
- [ ] Are images optimized?
- [ ] Is code splitting used for large bundles?
- [ ] Are there unnecessary re-renders?
```

---

## Common Pitfalls

### 1. Nitpicking When There Are Bigger Issues

```typescript
// ❌ BAD: 20 comments about naming when there's a security bug
// "This variable should be camelCase"
// "Add a space here"
// ... meanwhile, SQL injection ignored

// ✅ GOOD: Prioritize by impact
// Pass 1: Security & correctness issues
// Pass 2: Architecture & design
// Pass 3: Nits (or skip if time-sensitive)
```

### 2. Review as Gatekeeper, Not Collaborator

```typescript
// ❌ BAD: "No, do it my way"
// Reviewing to impose preferences, not improve code

// ✅ GOOD: "Here's why I suggest this approach..."
// Focus on:
// - Teaching, not commanding
// - Explaining reasoning
// - Being open to author's perspective
```

### 3. Delayed Reviews

```typescript
// ❌ BAD: Review after 3 days
// - Context is lost
// - Author has moved on mentally
// - Creates merge conflicts

// ✅ GOOD: Review within 1 business day
// - Set aside review time daily
// - Smaller PRs = faster reviews
// - Async communication for blockers
```

---

## Interview Questions

### Q1: How do you approach code review?

**A:** I approach reviews with three goals: catch bugs, maintain quality, and share knowledge. I start with a high-level pass (does the overall approach make sense?), then dive into logic/correctness, then style. I use prefixes (Blocker/Suggestion/Nit) to set expectations. I focus on explaining *why* something is an issue, not just *what* to change. I try to review within a day to not block teammates.

### Q2: How do you give constructive feedback?

**A:** I follow the principle of "critique the code, not the person." I phrase feedback as suggestions or questions, not commands. I explain my reasoning and offer alternatives. I acknowledge good work, not just problems. For major disagreements, I offer to discuss synchronously. I use "Blocker" sparingly and label nits as optional.

### Q3: How do you handle receiving critical feedback?

**A:** I remind myself that reviews are about code quality, not personal criticism. I read comments without immediately responding if frustrated. I try to understand the reviewer's perspective - they usually have a valid point. If I disagree, I explain my reasoning and offer to discuss. I thank reviewers for their time, especially for thorough reviews.

### Q4: How do you review a very large PR?

**A:** First, I check if it can be split - multiple concerns should be separate PRs. If it can't be split, I review in passes: architecture first, then logic, then style. I give early feedback on approach before diving into details (to avoid wasted effort if approach needs change). I communicate that the review will take time.

---

## Quick Reference Checklist

### As PR Author
- [ ] Keep PRs small (< 400 lines ideal)
- [ ] Write clear description (what & why)
- [ ] Self-review before requesting
- [ ] Add tests
- [ ] Respond to feedback promptly

### As Reviewer
- [ ] Review within 1 business day
- [ ] Start with high-level, then details
- [ ] Use prefixes (Blocker/Suggestion/Nit)
- [ ] Explain reasoning, not just issues
- [ ] Acknowledge good work

### Team Level
- [ ] Automate formatting/linting
- [ ] Establish review guidelines
- [ ] Rotate reviewers for knowledge sharing
- [ ] Track review metrics (time, iterations)

---

*Last updated: February 2026*

