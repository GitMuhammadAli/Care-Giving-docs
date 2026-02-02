# System Design Interviews - Complete Guide

> **MUST REMEMBER**: System design = structured conversation about building large-scale systems. Framework: Requirements → Estimation → High-level design → Deep dive → Trade-offs. Always clarify requirements first. State assumptions explicitly. Discuss trade-offs (there's no perfect solution). Practice: design URL shortener, Twitter feed, chat system, etc. Focus on communication, not just technical knowledge.

---

## How to Explain Like a Senior Developer

"System design interviews test how you approach ambiguous, large-scale problems - more about your thinking process than arriving at a 'correct' answer. The framework I use: start by clarifying requirements (functional and non-functional), do back-of-envelope calculations to understand scale, sketch a high-level design, then deep dive into the most critical or complex components. Throughout, I discuss trade-offs - every choice has pros and cons. The interviewer wants to see structured thinking, awareness of scale, ability to communicate complex ideas, and knowledge of common patterns. Don't jump into solutions. Don't ignore scale. Don't present one option - show you've considered alternatives."

---

## The Framework

### Step-by-Step Approach

```typescript
// system-design/framework.ts

/**
 * System Design Interview Framework
 * Time: 45-60 minutes total
 */

const interviewFramework = {
  
  step1_requirements: {
    time: '5-10 minutes',
    goal: 'Clarify scope and constraints',
    
    functionalRequirements: {
      questions: [
        'What are the core features?',
        'Who are the users?',
        'What are the main use cases?',
      ],
      example: {
        system: 'URL Shortener',
        functional: [
          'Shorten a long URL',
          'Redirect short URL to original',
          'Custom aliases (optional)',
          'Analytics (optional)',
        ],
      },
    },
    
    nonFunctionalRequirements: {
      questions: [
        'What scale are we designing for?',
        'What are latency requirements?',
        'What are availability requirements?',
        'Is consistency or availability more important?',
      ],
      example: {
        system: 'URL Shortener',
        nonFunctional: [
          '100M URLs created per month',
          '10:1 read:write ratio (1B redirects/month)',
          '< 100ms redirect latency',
          '99.9% availability',
          'URLs never expire (unless user deletes)',
        ],
      },
    },
  },
  
  step2_estimation: {
    time: '5 minutes',
    goal: 'Understand scale to drive design decisions',
    
    calculations: [
      'Traffic: requests per second',
      'Storage: total data over 5 years',
      'Bandwidth: data transfer per second',
      'Memory: cache size needed',
    ],
    
    example: {
      system: 'URL Shortener',
      traffic: {
        writes: '100M / 30 days / 86400 sec ≈ 40 URLs/sec',
        reads: '40 × 10 = 400 redirects/sec',
        peak: '400 × 3 = 1200 redirects/sec (3x peak)',
      },
      storage: {
        perUrl: '500 bytes (URL + metadata)',
        yearly: '100M × 12 × 500B = 600GB/year',
        fiveYears: '3TB',
      },
      cache: {
        hotUrls: 'Top 20% of URLs = 80% of traffic',
        cacheSize: '20% × 600GB = 120GB (fits in memory)',
      },
    },
  },
  
  step3_highLevel: {
    time: '10-15 minutes',
    goal: 'Design the main components and data flow',
    
    components: [
      'Clients (web, mobile)',
      'Load balancer',
      'Application servers',
      'Database(s)',
      'Cache',
      'CDN (if applicable)',
      'Message queue (if async needed)',
    ],
    
    diagram: `
      Client → Load Balancer → App Servers → Cache → Database
                                    ↓
                             Key Generation Service
    `,
  },
  
  step4_deepDive: {
    time: '15-20 minutes',
    goal: 'Detail critical components',
    
    areas: [
      'Database schema and choice',
      'API design',
      'Core algorithms',
      'Scaling strategies',
      'Failure handling',
    ],
  },
  
  step5_tradeoffs: {
    time: '5-10 minutes',
    goal: 'Discuss alternatives and limitations',
    
    questions: [
      'What are the bottlenecks?',
      'What happens if X fails?',
      'How would this change for 10x scale?',
      'What would you do differently with more time?',
    ],
  },
};
```

### Back-of-Envelope Calculations

```typescript
// system-design/calculations.ts

/**
 * Common numbers for estimation
 * Memorize these order of magnitudes
 */
const referenceNumbers = {
  
  time: {
    secondsPerDay: 86_400,
    secondsPerMonth: 2_500_000,  // ~2.5M
    secondsPerYear: 31_000_000,  // ~31M
  },
  
  storage: {
    KB: 1_000,
    MB: 1_000_000,
    GB: 1_000_000_000,
    TB: 1_000_000_000_000,
    
    // Typical sizes
    tweetSize: '300 bytes',
    imageMetadata: '200 bytes',
    userRecord: '1 KB',
    averageImage: '200 KB',
    averageVideo: '5 MB/min',
  },
  
  throughput: {
    ssdRead: '500 MB/s',
    ssdWrite: '200 MB/s',
    hddRead: '100 MB/s',
    networkInternal: '1-10 Gbps',
    networkPublic: '100 Mbps - 1 Gbps',
  },
  
  latency: {
    l1Cache: '0.5 ns',
    l2Cache: '7 ns',
    ram: '100 ns',
    ssd: '100 µs',
    hdd: '10 ms',
    sameDatacenter: '0.5 ms',
    crossCountry: '50-100 ms',
    crossContinent: '100-200 ms',
  },
  
  scale: {
    millionPerMonth: 0.4,  // requests per second
    millionPerDay: 12,     // requests per second  
    billionPerMonth: 400,  // requests per second
  },
};

/**
 * Example calculation: Twitter-like feed
 */
const twitterCalculation = {
  assumptions: {
    users: '500M total users',
    dau: '200M daily active users',
    avgTweetsPerUser: '2 per day',
    avgFollowing: '200 people',
    avgTweetSize: '300 bytes',
  },
  
  traffic: {
    tweetsPerDay: '200M × 2 = 400M tweets/day',
    tweetsPerSecond: '400M / 86400 ≈ 4600 tweets/sec',
    
    feedReads: '200M users × 10 feed views = 2B reads/day',
    readsPerSecond: '2B / 86400 ≈ 23000 reads/sec',
  },
  
  storage: {
    tweetsPerYear: '400M × 365 = 146B tweets',
    tweetStorage: '146B × 300B = 44TB/year (just text)',
  },
  
  fanout: {
    avgFollowers: 200,
    fanoutPerTweet: 'Each tweet → 200 feed updates',
    totalFanout: '4600 × 200 = 920K feed updates/sec',
  },
};
```

---

## Common System Design Problems

### URL Shortener

```typescript
// system-design/url-shortener.ts

/**
 * Design a URL shortener (like bit.ly)
 */

// API Design
const urlShortenerAPI = {
  createShortUrl: {
    method: 'POST',
    path: '/api/urls',
    body: {
      longUrl: 'https://example.com/very/long/path',
      customAlias: 'my-link',  // optional
      expiresAt: '2025-01-01', // optional
    },
    response: {
      shortUrl: 'https://short.ly/abc123',
      shortCode: 'abc123',
    },
  },
  redirect: {
    method: 'GET',
    path: '/:shortCode',
    response: '302 Redirect to long URL',
  },
};

// Database Schema
const urlSchema = {
  table: 'urls',
  columns: {
    id: 'BIGINT PRIMARY KEY',
    shortCode: 'VARCHAR(10) UNIQUE INDEX',
    longUrl: 'VARCHAR(2048)',
    userId: 'BIGINT INDEX',  // optional, for registered users
    createdAt: 'TIMESTAMP',
    expiresAt: 'TIMESTAMP NULL',
    clickCount: 'INT DEFAULT 0',
  },
};

// Key Generation Strategies
const keyGenStrategies = {
  
  // Option 1: Hash the URL
  hashBased: {
    approach: 'MD5/SHA hash of URL, take first 7 chars',
    pros: ['Deterministic', 'Same URL = same short code'],
    cons: ['Collisions possible', 'Need to handle duplicates'],
    collision: 'Base62 with 7 chars = 62^7 = 3.5 trillion combinations',
  },
  
  // Option 2: Counter-based
  counterBased: {
    approach: 'Increment counter, convert to Base62',
    pros: ['No collisions', 'Simple'],
    cons: ['Counter is bottleneck', 'Predictable codes'],
    scaling: 'Use multiple ranges per server (1-1M, 1M-2M, etc.)',
  },
  
  // Option 3: Pre-generated keys (Recommended)
  preGenerated: {
    approach: 'Generate keys in advance, store in key DB',
    implementation: `
      1. Key Generation Service generates random keys
      2. Store in key_db table (key, used: boolean)
      3. App server requests batch of keys
      4. Mark keys as used atomically
    `,
    pros: ['No collision handling needed', 'Scales well'],
    cons: ['Separate service needed', 'Key synchronization'],
  },
};

// Scaling Considerations
const scalingUrlShortener = {
  
  database: {
    strategy: 'Shard by shortCode (first char)',
    reason: 'Distributes reads and writes evenly',
    implementation: 'shortCode starts with a-z → 26 shards',
  },
  
  caching: {
    strategy: 'Cache popular URLs in Redis',
    eviction: 'LRU (Least Recently Used)',
    ttl: '24 hours',
    hitRate: 'Expect 90%+ hit rate for top URLs',
  },
  
  availability: {
    database: 'Primary-replica setup with auto-failover',
    appServers: 'Multiple instances behind load balancer',
    keyService: 'Pre-generate keys, cache batch per app server',
  },
};
```

### Design a Chat System

```typescript
// system-design/chat-system.ts

/**
 * Design a real-time chat system (like WhatsApp/Slack)
 */

// Requirements clarification
const chatRequirements = {
  functional: [
    'Send/receive messages in real-time',
    '1-on-1 and group chats',
    'Message history',
    'Online/offline status',
    'Read receipts',
    'Media sharing (images, files)',
  ],
  
  nonFunctional: {
    users: '500M total, 50M DAU',
    latency: '< 100ms message delivery',
    availability: '99.99%',
    storage: 'Keep messages forever',
  },
};

// High-level architecture
const chatArchitecture = `
  ┌─────────┐     ┌──────────────┐     ┌─────────────┐
  │  Client │────▶│ API Gateway  │────▶│ Chat Service│
  │(Mobile/ │     │(Auth, Rate   │     │(HTTP APIs)  │
  │  Web)   │     │ Limiting)    │     └─────────────┘
  └────┬────┘     └──────────────┘            │
       │                                       ▼
       │          ┌──────────────┐     ┌─────────────┐
       └─────────▶│ WebSocket    │────▶│ Message     │
        realtime  │ Servers      │     │ Queue       │
                  └──────────────┘     └──────┬──────┘
                         ▲                     │
                         │              ┌──────▼──────┐
                  ┌──────┴──────┐       │ Message     │
                  │ Presence    │       │ Storage     │
                  │ Service     │       │(Cassandra)  │
                  └─────────────┘       └─────────────┘
`;

// Key components
const chatComponents = {
  
  webSocketService: {
    purpose: 'Maintain persistent connections for real-time messaging',
    implementation: `
      - Each user maintains WebSocket connection
      - Server tracks: userId → connectionId mapping
      - When user sends message:
        1. Receive via WebSocket
        2. Store in message queue
        3. Deliver to recipients via their WebSocket
    `,
    scaling: `
      - Multiple WS servers (stateless)
      - Use Redis Pub/Sub for cross-server messaging
      - User can be connected to any server
    `,
  },
  
  messageDelivery: {
    online: 'Push via WebSocket immediately',
    offline: 'Store in pending queue, deliver when online',
    pushNotification: 'Send FCM/APNs when recipient offline',
  },
  
  groupChat: {
    smallGroups: 'Fan-out on write (store per recipient)',
    largeGroups: 'Fan-out on read (store once, deliver on read)',
    threshold: '~100 members = switch from write to read fan-out',
  },
  
  messageStorage: {
    choice: 'Cassandra (wide-column store)',
    reason: 'High write throughput, easy sharding, time-series queries',
    schema: `
      Partition key: chat_id
      Clustering key: timestamp
      
      This groups messages by conversation, ordered by time
    `,
  },
  
  presence: {
    online: 'Heartbeat every 5 seconds',
    offline: 'No heartbeat for 30 seconds = offline',
    storage: 'Redis with TTL',
    fanout: 'Only notify close contacts, not all followers',
  },
};
```

---

## Common Patterns

### Scaling Patterns to Know

```typescript
// system-design/patterns.ts

const scalingPatterns = {
  
  horizontalScaling: {
    what: 'Add more machines',
    when: 'Stateless services',
    how: 'Load balancer distributes traffic',
  },
  
  verticalScaling: {
    what: 'Bigger machine',
    when: 'Quick fix, database (initially)',
    limit: 'Hardware limits, single point of failure',
  },
  
  caching: {
    what: 'Store frequently accessed data in memory',
    layers: ['Client', 'CDN', 'Application', 'Database'],
    patterns: {
      cacheAside: 'App checks cache, falls back to DB',
      writeThrough: 'Write to cache and DB together',
      writeBehind: 'Write to cache, async write to DB',
    },
  },
  
  databaseSharding: {
    what: 'Split data across multiple databases',
    strategies: {
      range: 'userId 1-1M on shard1, 1M-2M on shard2',
      hash: 'hash(userId) % numShards',
      directory: 'Lookup table maps key to shard',
    },
    challenges: ['Joins', 'Rebalancing', 'Hot spots'],
  },
  
  loadBalancing: {
    algorithms: {
      roundRobin: 'Equal distribution',
      leastConnections: 'Route to least busy server',
      ipHash: 'Same client → same server (sticky)',
      weighted: 'More traffic to bigger servers',
    },
  },
  
  messageQueues: {
    what: 'Async communication between services',
    when: 'Decouple services, handle spikes, retry logic',
    tools: ['Kafka', 'RabbitMQ', 'SQS'],
    patterns: ['Pub/Sub', 'Work queue', 'Request/reply'],
  },
  
  cdn: {
    what: 'Serve static content from edge locations',
    when: 'Static assets, geographically distributed users',
    benefits: ['Lower latency', 'Reduced origin load'],
  },
};
```

---

## Interview Tips

### Communication Framework

```typescript
// system-design/communication.ts

const communicationTips = {
  
  thinkAloud: {
    why: 'Interviewer wants to see your thought process',
    how: `
      "Let me think about the trade-offs here..."
      "I'm considering two approaches..."
      "One concern with this approach is..."
    `,
  },
  
  askQuestions: {
    why: 'Requirements are intentionally ambiguous',
    examples: [
      'What features are in scope?',
      'How many users are we designing for?',
      'What\'s more important: consistency or availability?',
      'Can we assume a specific geographic region?',
    ],
  },
  
  stateAssumptions: {
    why: 'Shows awareness of constraints',
    how: `
      "I'll assume we have 100M users..."
      "For simplicity, I'll start with a single region..."
      "I'm assuming we can tolerate eventual consistency..."
    `,
  },
  
  discussTradeoffs: {
    why: 'Shows depth of understanding',
    how: `
      "SQL gives us ACID but is harder to scale horizontally"
      "Caching improves latency but adds complexity for invalidation"
      "Push model is real-time but expensive at scale"
    `,
  },
  
  structureYourTime: {
    '45min': {
      requirements: '5 min',
      estimation: '5 min',
      highLevel: '10 min',
      deepDive: '20 min',
      wrapUp: '5 min',
    },
  },
};
```

---

## Common Pitfalls

### 1. Jumping to Solutions

```markdown
❌ BAD: "We'll use MongoDB and WebSockets"
(Without understanding requirements)

✅ GOOD: "Before I design, let me clarify:
- How many users?
- Real-time or near-real-time?
- What's the read/write ratio?"
```

### 2. Ignoring Scale

```markdown
❌ BAD: Design for small scale, then say "and we'll scale later"

✅ GOOD: Estimate scale upfront, design for it
"At 1B requests/day, that's ~12K QPS.
A single database won't handle this, so let's discuss sharding..."
```

### 3. Not Discussing Trade-offs

```markdown
❌ BAD: "We'll use X" (as if it's obviously correct)

✅ GOOD: "We could use X or Y. X gives us [benefits] 
but has [drawbacks]. Y is better for [scenario].
Given our requirements for [requirement], I'd choose X."
```

---

## Interview Questions

### Q1: How do you approach a system design problem?

**A:** I follow a structured framework: First, clarify requirements (functional and non-functional). Then, do back-of-envelope calculations to understand scale. Next, sketch high-level architecture. Then deep dive into critical components. Throughout, I discuss trade-offs. I make sure to think aloud and ask clarifying questions.

### Q2: How do you handle ambiguity in design problems?

**A:** Ambiguity is intentional - it tests how you handle uncertainty. I ask clarifying questions, state assumptions explicitly, and design for the most likely scenario while noting how the design would change for other scenarios. I don't try to design for every possible requirement.

### Q3: What do you do when you don't know something?

**A:** I'm honest about gaps and reason from first principles. "I haven't used Cassandra directly, but I know it's a wide-column store optimized for write-heavy workloads, so I'd consider it for our messaging use case." Interviewers value honesty and reasoning over pretending to know everything.

---

## Quick Reference Checklist

### During Interview
- [ ] Clarify requirements first
- [ ] State assumptions explicitly
- [ ] Do rough calculations
- [ ] Draw high-level diagram
- [ ] Discuss trade-offs
- [ ] Think aloud

### Common Components
- [ ] Load balancer
- [ ] CDN
- [ ] API gateway
- [ ] Application servers
- [ ] Cache (Redis)
- [ ] Database (SQL/NoSQL)
- [ ] Message queue
- [ ] Search (Elasticsearch)

### Numbers to Know
- [ ] 86,400 seconds/day
- [ ] 2.5M seconds/month
- [ ] 1M req/day ≈ 12 req/sec
- [ ] 1B req/day ≈ 12K req/sec

---

*Last updated: February 2026*

