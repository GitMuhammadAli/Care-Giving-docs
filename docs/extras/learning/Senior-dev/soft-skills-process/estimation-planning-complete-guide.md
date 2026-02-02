# Estimation & Planning - Complete Guide

> **MUST REMEMBER**: Estimation is about communication, not precision. Story points measure complexity, not time. Break large tasks into smaller pieces (< 3 days). Use ranges (1-3 days), not single numbers. Track velocity over time to improve accuracy. Common techniques: Planning Poker, T-shirt sizing, reference stories. Under-estimation causes: optimism bias, forgetting integration/testing/edge cases, scope creep.

---

## How to Explain Like a Senior Developer

"Estimation is one of the hardest skills in software - we're predicting the future for something we've never built before. The key insight: estimates are for communication, not contracts. They help the team and stakeholders plan. Story points measure relative complexity, not hours - a 5-point story is roughly double a 2-point story's complexity. Break work into small chunks because accuracy drops dramatically past 3 days. Always estimate in ranges (2-5 days, not 3 days) to communicate uncertainty. Track your velocity over sprints - this tells you how many points you actually complete, not how many you thought you would. The biggest mistakes: estimating without understanding the work, forgetting testing and edge cases, not accounting for integration, and optimism bias (everything takes longer than you think)."

---

## Core Estimation Techniques

### Story Points with Planning Poker

```typescript
// estimation/story-points.ts

/**
 * Fibonacci-based story points
 * Why Fibonacci: Forces thinking about relative size
 * The gap between 8 and 13 is intentional - big stories are hard to estimate
 */
const STORY_POINTS = [1, 2, 3, 5, 8, 13, 21, '?', '☕'];

interface Story {
  id: string;
  title: string;
  description: string;
  acceptanceCriteria: string[];
}

interface EstimationRound {
  story: Story;
  votes: Map<string, number | string>;
  finalEstimate?: number;
  discussion: string[];
}

/**
 * Reference stories for calibration
 * Team should agree on these benchmarks
 */
const referenceStories = {
  1: 'Add a new field to an existing API response (no logic changes)',
  2: 'Create a simple CRUD endpoint with validation',
  3: 'Add authentication to an existing endpoint',
  5: 'Build a new feature with database schema changes',
  8: 'Integrate with a third-party API (with error handling, retries)',
  13: 'Major refactoring of a core service',
  21: 'This should probably be broken down',
};

/**
 * Planning Poker process
 */
class PlanningPoker {
  private team: string[];
  private rounds: EstimationRound[] = [];
  
  constructor(team: string[]) {
    this.team = team;
  }
  
  /**
   * Run estimation for a story
   */
  async estimate(story: Story): Promise<number> {
    let round = 1;
    let consensus = false;
    
    while (!consensus && round <= 3) {
      console.log(`\nRound ${round} for: ${story.title}`);
      
      // Everyone votes simultaneously
      const votes = await this.collectVotes(story);
      
      // Check for consensus
      const uniqueVotes = new Set(votes.values());
      
      if (uniqueVotes.size === 1) {
        // Everyone agrees
        consensus = true;
        return Array.from(votes.values())[0] as number;
      }
      
      if (this.isCloseEnough(votes)) {
        // Close enough - take the higher
        consensus = true;
        return Math.max(...Array.from(votes.values()) as number[]);
      }
      
      // Discussion needed
      const { lowest, highest } = this.getExtremes(votes);
      console.log(`Votes range from ${lowest.vote} to ${highest.vote}`);
      console.log(`${lowest.member}: Why low?`);
      console.log(`${highest.member}: Why high?`);
      
      // After discussion, vote again
      round++;
    }
    
    // No consensus after 3 rounds - take the higher estimate
    console.log('No consensus - using higher estimate to be safe');
    return Math.max(...Array.from(this.rounds[this.rounds.length - 1].votes.values()) as number[]);
  }
  
  private async collectVotes(story: Story): Promise<Map<string, number | string>> {
    const votes = new Map<string, number | string>();
    // In real implementation, votes collected simultaneously
    // to avoid anchoring bias
    return votes;
  }
  
  private isCloseEnough(votes: Map<string, number | string>): boolean {
    const numericVotes = Array.from(votes.values()).filter(v => typeof v === 'number') as number[];
    const max = Math.max(...numericVotes);
    const min = Math.min(...numericVotes);
    
    // Adjacent Fibonacci numbers are "close enough"
    const adjacentPairs = [[1, 2], [2, 3], [3, 5], [5, 8]];
    return adjacentPairs.some(([a, b]) => 
      (min === a && max === b) || (min === b && max === a)
    );
  }
  
  private getExtremes(votes: Map<string, number | string>) {
    let lowest = { member: '', vote: Infinity };
    let highest = { member: '', vote: -Infinity };
    
    for (const [member, vote] of votes.entries()) {
      if (typeof vote === 'number') {
        if (vote < lowest.vote) lowest = { member, vote };
        if (vote > highest.vote) highest = { member, vote };
      }
    }
    
    return { lowest, highest };
  }
}
```

### T-Shirt Sizing for Roadmap Planning

```typescript
// estimation/tshirt-sizing.ts

/**
 * T-shirt sizing for high-level/roadmap estimation
 * Less precise than story points, better for early planning
 */

type TShirtSize = 'XS' | 'S' | 'M' | 'L' | 'XL' | 'XXL';

interface TShirtMapping {
  size: TShirtSize;
  description: string;
  typicalDuration: string;
  typicalPoints: string;
  examples: string[];
}

const tshirtMappings: TShirtMapping[] = [
  {
    size: 'XS',
    description: 'Trivial change, well understood',
    typicalDuration: '< 1 day',
    typicalPoints: '1-2 points',
    examples: [
      'Fix typo in UI',
      'Add config flag',
      'Update dependency version',
    ],
  },
  {
    size: 'S',
    description: 'Small, well-defined task',
    typicalDuration: '1-2 days',
    typicalPoints: '2-3 points',
    examples: [
      'Add new API field',
      'Create simple component',
      'Write missing tests',
    ],
  },
  {
    size: 'M',
    description: 'Medium complexity, some unknowns',
    typicalDuration: '3-5 days',
    typicalPoints: '5-8 points',
    examples: [
      'New feature with DB changes',
      'API integration',
      'Moderate refactoring',
    ],
  },
  {
    size: 'L',
    description: 'Large feature, significant complexity',
    typicalDuration: '1-2 weeks',
    typicalPoints: '13-21 points',
    examples: [
      'New service/module',
      'Major feature',
      'Complex integration',
    ],
  },
  {
    size: 'XL',
    description: 'Very large, should be broken down',
    typicalDuration: '2-4 weeks',
    typicalPoints: '20+ points',
    examples: [
      'New subsystem',
      'Major migration',
      'Architecture change',
    ],
  },
  {
    size: 'XXL',
    description: 'Too big to estimate - needs breakdown',
    typicalDuration: '> 1 month',
    typicalPoints: 'Unknown',
    examples: [
      'Major initiative',
      'Platform rewrite',
      'Should be an epic',
    ],
  },
];

/**
 * When to use T-shirt vs Story Points
 */
const estimationContext = {
  tshirtSizing: {
    when: [
      'Roadmap planning (quarterly)',
      'Initial backlog grooming',
      'High-level project estimates',
      'Comparing initiatives',
    ],
    benefits: [
      'Fast',
      'Good for unknowns',
      'Non-technical stakeholder friendly',
    ],
  },
  storyPoints: {
    when: [
      'Sprint planning',
      'Well-defined stories',
      'Tracking velocity',
    ],
    benefits: [
      'More precise',
      'Velocity tracking',
      'Team calibration',
    ],
  },
};
```

### Three-Point Estimation

```typescript
// estimation/three-point.ts

/**
 * Three-Point Estimation (PERT)
 * Use for higher-stakes estimates where uncertainty matters
 */

interface ThreePointEstimate {
  optimistic: number;    // Best case (10% chance)
  mostLikely: number;    // Most probable
  pessimistic: number;   // Worst case (90% includes this)
}

function calculatePERT(estimate: ThreePointEstimate): {
  expected: number;
  standardDeviation: number;
  range: { low: number; high: number };
} {
  const { optimistic, mostLikely, pessimistic } = estimate;
  
  // PERT formula: (O + 4M + P) / 6
  const expected = (optimistic + 4 * mostLikely + pessimistic) / 6;
  
  // Standard deviation: (P - O) / 6
  const standardDeviation = (pessimistic - optimistic) / 6;
  
  // 95% confidence interval: expected ± 2*stdDev
  const range = {
    low: Math.max(0, expected - 2 * standardDeviation),
    high: expected + 2 * standardDeviation,
  };
  
  return { expected, standardDeviation, range };
}

// Example usage
const featureEstimate: ThreePointEstimate = {
  optimistic: 3,    // Best case: 3 days
  mostLikely: 5,    // Probably: 5 days
  pessimistic: 12,  // Worst case: 12 days (unknown integrations, etc.)
};

const result = calculatePERT(featureEstimate);
// expected: 5.8 days
// range: 2.8 - 8.8 days (95% confidence)

console.log(`Expected: ${result.expected.toFixed(1)} days`);
console.log(`Range: ${result.range.low.toFixed(1)} - ${result.range.high.toFixed(1)} days`);
```

---

## Real-World Scenarios

### Scenario 1: Sprint Planning

```typescript
// estimation/sprint-planning.ts

interface Sprint {
  number: number;
  startDate: Date;
  endDate: Date;
  capacity: number;  // Story points
  committed: Story[];
  completed: Story[];
}

interface TeamMetrics {
  averageVelocity: number;
  velocityHistory: number[];
  availabilityPercent: number;  // Account for PTO, meetings
}

class SprintPlanner {
  private team: TeamMetrics;
  private sprints: Sprint[] = [];
  
  /**
   * Calculate sprint capacity
   */
  calculateCapacity(teamSize: number, sprintDays: number): number {
    // Historical velocity is most reliable
    if (this.team.velocityHistory.length >= 3) {
      const recentVelocity = this.team.velocityHistory.slice(-3);
      const avgVelocity = recentVelocity.reduce((a, b) => a + b, 0) / recentVelocity.length;
      
      // Adjust for availability
      return Math.floor(avgVelocity * this.team.availabilityPercent);
    }
    
    // New team heuristic: ~8-10 points per dev per 2-week sprint
    return teamSize * 8 * this.team.availabilityPercent;
  }
  
  /**
   * Sprint planning meeting flow
   */
  planSprint(backlog: Story[], capacity: number): Story[] {
    const committed: Story[] = [];
    let totalPoints = 0;
    
    for (const story of backlog) {
      // Check if story is ready
      if (!this.isStoryReady(story)) {
        console.log(`Story ${story.id} not ready - needs refinement`);
        continue;
      }
      
      // Check capacity
      if (totalPoints + (story as any).points > capacity) {
        console.log(`Capacity reached at ${totalPoints} points`);
        break;
      }
      
      // Commit to story
      committed.push(story);
      totalPoints += (story as any).points;
    }
    
    // Leave buffer (don't fill to 100%)
    const utilizationPercent = (totalPoints / capacity) * 100;
    if (utilizationPercent > 85) {
      console.log(`Warning: ${utilizationPercent}% utilization - consider removing a story`);
    }
    
    return committed;
  }
  
  private isStoryReady(story: Story): boolean {
    return (
      story.description.length > 0 &&
      story.acceptanceCriteria.length > 0 &&
      (story as any).points !== undefined &&
      (story as any).points <= 8  // Large stories should be split
    );
  }
  
  /**
   * After sprint - update velocity
   */
  completeSprint(sprint: Sprint): void {
    const completedPoints = sprint.completed.reduce(
      (sum, story) => sum + ((story as any).points || 0),
      0
    );
    
    this.team.velocityHistory.push(completedPoints);
    
    // Keep last 6 sprints for velocity calculation
    if (this.team.velocityHistory.length > 6) {
      this.team.velocityHistory.shift();
    }
    
    // Calculate new average
    this.team.averageVelocity = 
      this.team.velocityHistory.reduce((a, b) => a + b, 0) / 
      this.team.velocityHistory.length;
    
    console.log(`Sprint ${sprint.number} completed: ${completedPoints} points`);
    console.log(`New average velocity: ${this.team.averageVelocity.toFixed(1)}`);
  }
}
```

### Scenario 2: Project Timeline Estimation

```typescript
// estimation/project-timeline.ts

interface Epic {
  id: string;
  name: string;
  stories: Story[];
  dependencies: string[];  // Epic IDs
}

interface ProjectEstimate {
  epics: Epic[];
  totalPoints: number;
  estimatedSprints: number;
  estimatedWeeks: { min: number; max: number };
  risks: string[];
}

class ProjectEstimator {
  private teamVelocity: number;
  private sprintLengthWeeks: number = 2;
  
  constructor(teamVelocity: number) {
    this.teamVelocity = teamVelocity;
  }
  
  /**
   * Estimate project timeline
   */
  estimateProject(epics: Epic[]): ProjectEstimate {
    // Sum all story points
    const totalPoints = epics.reduce((sum, epic) => {
      return sum + epic.stories.reduce(
        (s, story) => s + ((story as any).points || 0),
        0
      );
    }, 0);
    
    // Calculate sprints needed
    const sprintsNeeded = Math.ceil(totalPoints / this.teamVelocity);
    
    // Add buffer for unknowns (20-50% depending on uncertainty)
    const uncertaintyMultiplier = this.calculateUncertainty(epics);
    
    const minWeeks = sprintsNeeded * this.sprintLengthWeeks;
    const maxWeeks = Math.ceil(minWeeks * uncertaintyMultiplier);
    
    // Identify risks
    const risks = this.identifyRisks(epics);
    
    return {
      epics,
      totalPoints,
      estimatedSprints: sprintsNeeded,
      estimatedWeeks: { min: minWeeks, max: maxWeeks },
      risks,
    };
  }
  
  private calculateUncertainty(epics: Epic[]): number {
    let uncertainty = 1.2;  // Base 20% buffer
    
    // Large stories increase uncertainty
    const largeStories = epics.flatMap(e => e.stories)
      .filter(s => ((s as any).points || 0) >= 8);
    uncertainty += largeStories.length * 0.05;
    
    // Dependencies increase uncertainty
    const hasDependencies = epics.some(e => e.dependencies.length > 0);
    if (hasDependencies) uncertainty += 0.1;
    
    // Cap at 1.5x
    return Math.min(uncertainty, 1.5);
  }
  
  private identifyRisks(epics: Epic[]): string[] {
    const risks: string[] = [];
    
    // Large stories
    const largeStoryCount = epics.flatMap(e => e.stories)
      .filter(s => ((s as any).points || 0) >= 8).length;
    if (largeStoryCount > 0) {
      risks.push(`${largeStoryCount} large stories (8+ points) - consider breaking down`);
    }
    
    // Circular dependencies
    // ... (complex check omitted)
    
    // Unestimated stories
    const unestimated = epics.flatMap(e => e.stories)
      .filter(s => (s as any).points === undefined).length;
    if (unestimated > 0) {
      risks.push(`${unestimated} unestimated stories`);
    }
    
    return risks;
  }
  
  /**
   * Present estimate to stakeholders
   */
  formatForStakeholders(estimate: ProjectEstimate): string {
    return `
## Project Estimate

**Scope:** ${estimate.epics.length} epics, ${estimate.totalPoints} story points

**Timeline:** ${estimate.estimatedWeeks.min}-${estimate.estimatedWeeks.max} weeks
- Best case: ${estimate.estimatedWeeks.min} weeks
- Expected: ${Math.round((estimate.estimatedWeeks.min + estimate.estimatedWeeks.max) / 2)} weeks
- Worst case: ${estimate.estimatedWeeks.max} weeks

**Assumptions:**
- Team velocity of ${this.teamVelocity} points/sprint maintained
- No significant scope changes
- Full team availability

**Risks:**
${estimate.risks.map(r => `- ${r}`).join('\n')}

**Recommendation:** Plan for ${estimate.estimatedWeeks.max} weeks to allow for unknowns.
    `.trim();
  }
}
```

---

## Common Pitfalls

### 1. Estimating Without Understanding

```typescript
// ❌ BAD: Quick estimate without investigation
// "Yeah, that's probably a 3-pointer"

// ✅ GOOD: Understand before estimating
const estimationChecklist = [
  'Have I read the requirements/acceptance criteria?',
  'Do I understand the technical approach?',
  'Have I identified the main risks/unknowns?',
  'Have I checked for dependencies?',
  'Have I considered testing effort?',
];
```

### 2. Forgetting Non-Coding Work

```typescript
// ❌ BAD: Estimate only coding time
const badEstimate = {
  coding: 2, // days
  total: 2,  // Missing everything else!
};

// ✅ GOOD: Include all work
const goodEstimate = {
  coding: 2,
  codeReview: 0.5,
  testing: 1,
  documentation: 0.5,
  deployment: 0.25,
  buffer: 0.5,  // Unknowns, meetings, context switching
  total: 4.75,
};
```

### 3. Single-Point Estimates

```typescript
// ❌ BAD: "It'll take 3 days"
// (Really means: best case, if nothing goes wrong)

// ✅ GOOD: "It'll take 3-5 days"
// (Communicates uncertainty)

// Even better for important estimates:
const rangedEstimate = {
  optimistic: 3,  // If everything goes smoothly
  likely: 4,      // Realistic expectation
  pessimistic: 7, // If we hit problems
  confidence: 'Medium - depends on API stability',
};
```

---

## Interview Questions

### Q1: How do you estimate a feature you've never built before?

**A:** First, break it into smaller pieces I can reason about. Research similar implementations or ask teammates. Use reference stories (is this bigger or smaller than X?). Give a range, not a single number. Identify the unknowns and timebox a spike if needed. Track my estimates vs actuals to improve over time.

### Q2: What do you do when stakeholders push back on estimates?

**A:** I explain the breakdown - here's what the estimate includes. I ask what tradeoffs they'd accept (reduced scope, less testing, more risk). I don't negotiate the estimate down without reducing scope. I offer options: "We can do A in 2 weeks, or A+B in 4 weeks." I'm transparent about uncertainty.

### Q3: How do you improve estimation accuracy over time?

**A:** Track actuals vs estimates for every story. Do retrospectives on misses - why was it different? Build a library of reference stories. Measure team velocity over multiple sprints. Account for systemic biases (we always forget testing, integration takes longer than expected).

### Q4: Story points vs hours - which do you prefer?

**A:** Story points for sprint planning - they measure complexity, not time, and everyone's hour is different. Hours for time-sensitive commitments or when stakeholders need calendar dates. The key is: story points for team planning, convert to time ranges for external communication using velocity.

---

## Quick Reference Checklist

### Before Estimating
- [ ] Understand requirements and acceptance criteria
- [ ] Identify technical approach
- [ ] Consider dependencies
- [ ] Account for testing
- [ ] Identify unknowns

### Estimation Meeting
- [ ] Use reference stories for calibration
- [ ] Vote simultaneously (avoid anchoring)
- [ ] Discuss outliers
- [ ] Include all work (not just coding)
- [ ] Give ranges for uncertainty

### After Sprint
- [ ] Compare estimates vs actuals
- [ ] Update velocity
- [ ] Retrospect on big misses
- [ ] Update reference stories

---

*Last updated: February 2026*

