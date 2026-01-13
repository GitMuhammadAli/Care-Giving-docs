# Chapter 06: Distributed Systems

> "A distributed system is one where a machine you've never heard of can cause your program to fail."

---

## 🌐 Why Distributed Systems?

```
Reasons:
1. Scalability - Handle more load
2. Reliability - No single point of failure
3. Latency - Data closer to users
4. Compliance - Data sovereignty requirements

Challenges:
1. Network is unreliable
2. Latency is non-zero
3. Bandwidth is limited
4. Network is insecure
5. Topology changes
6. There are multiple administrators
7. Transport cost is non-zero
8. Network is heterogeneous
```

---

## 🔄 Consensus Algorithms

### The Problem

```
Three servers need to agree on a value.
What if one crashes? Network partitions?

Server A: "Value is 5"
Server B: "Value is 5"
Server C: (no response - crashed? slow? partition?)

How do we proceed?
```

### Paxos

```
Roles:
- Proposers: Suggest values
- Acceptors: Vote on proposals
- Learners: Learn decided value

Phase 1 (Prepare):
┌──────────┐ prepare(n)  ┌──────────┐
│ Proposer │────────────►│ Acceptor │
│          │◄────────────│          │
│          │ promise(n)  │          │
└──────────┘             └──────────┘

Phase 2 (Accept):
┌──────────┐ accept(n,v) ┌──────────┐
│ Proposer │────────────►│ Acceptor │
│          │◄────────────│          │
│          │ accepted    │          │
└──────────┘             └──────────┘

Guarantees:
- Safety: Only one value chosen
- Liveness: Eventually a value is chosen (if majority available)
```

### Raft (Understandable Consensus)

```
Leader Election:
┌─────────────────────────────────────────────────────────────────┐
│ All nodes start as FOLLOWERS                                    │
│                                                                 │
│ If no heartbeat from leader → become CANDIDATE                  │
│                                                                 │
│ CANDIDATE requests votes from others                            │
│                                                                 │
│ If majority votes → become LEADER                               │
│                                                                 │
│ LEADER sends heartbeats and replicates log                      │
└─────────────────────────────────────────────────────────────────┘

Log Replication:
┌────────────────────────────────────────────────────────────────┐
│ Leader                                                          │
│ Log: [cmd1] [cmd2] [cmd3] [cmd4]                               │
│         │      │      │      │                                  │
│         ▼      ▼      ▼      ▼                                  │
│ Follower 1: [cmd1] [cmd2] [cmd3] [cmd4]  ✓                     │
│ Follower 2: [cmd1] [cmd2] [cmd3]         (catching up)         │
│                                                                 │
│ Entry committed when majority has it                            │
└────────────────────────────────────────────────────────────────┘

Used by: etcd, Consul, CockroachDB
```

---

## ⏰ Time in Distributed Systems

### The Problem with Clocks

```
Physical clocks are unreliable:
- Clock drift (crystals aren't perfect)
- NTP can jump forward or backward
- Leap seconds

Server A: 10:00:00.000
Server B: 10:00:00.050  (50ms drift)
Server C: 10:00:00.100  (100ms drift)

"Who wrote first?" is hard to answer!
```

### Logical Clocks

**Lamport Timestamps:**
```
Rule: On each event, increment counter
      On receive, max(local, received) + 1

Process A:     Process B:     Process C:
    │              │              │
   (1)───send────►(2)            │
    │              │              │
   (3)            (4)───send───►(5)
    │              │              │
   (6)◄──────────(7)             │
    │              │              │

If event a → b (happens before), then L(a) < L(b)
But L(a) < L(b) doesn't mean a → b
```

**Vector Clocks:**
```
Each process maintains vector of all process clocks

Process A:      Process B:      Process C:
[1,0,0]         [0,0,0]         [0,0,0]
    │               │               │
[2,0,0]──send─►[2,1,0]             │
    │               │               │
    │           [2,2,0]──send──►[2,2,1]
    │               │               │
[3,2,0]◄───────[2,3,0]             │

Can determine:
- a happened before b: V(a) < V(b)
- a and b concurrent: neither < other
```

### Hybrid Logical Clocks (HLC)

```
Combines physical + logical time

HLC = (physical_time, logical_counter, node_id)

Benefits:
- Bounded difference from real time
- Total ordering of events
- Works across datacenters

Used by: CockroachDB, YugabyteDB
```

---

## 🔒 Distributed Locking

### The Problem

```
Two processes try to modify same resource:

Process A: read balance → 100
Process B: read balance → 100
Process A: write balance → 100 - 50 = 50
Process B: write balance → 100 - 30 = 70  (lost update!)

Need mutual exclusion!
```

### Redis-Based Lock (Redlock)

```javascript
async function acquireLock(resource, ttl = 10000) {
  const lockKey = `lock:${resource}`;
  const lockValue = uuid();
  
  // SET with NX (only if not exists) and PX (expiry)
  const acquired = await redis.set(lockKey, lockValue, 'NX', 'PX', ttl);
  
  if (acquired) {
    return {
      release: async () => {
        // Only release if we own the lock (check value)
        const script = `
          if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
          else
            return 0
          end
        `;
        await redis.eval(script, 1, lockKey, lockValue);
      }
    };
  }
  return null;
}

// Usage
const lock = await acquireLock('order:123');
if (lock) {
  try {
    await processOrder();
  } finally {
    await lock.release();
  }
}
```

### Distributed Lock Challenges

```
1. Clock drift:
   Lock expires before work done
   Solution: Fencing tokens
   
2. Network partition:
   Client thinks it has lock, but it expired
   Solution: Heartbeat renewal
   
3. Split brain:
   Two clients think they have lock
   Solution: Redlock across multiple Redis instances

Fencing tokens:
┌────────────────────────────────────────────────────────────┐
│ Lock grant includes incrementing token                     │
│                                                            │
│ Client A gets lock (token: 34)                             │
│ Client A takes too long, lock expires                      │
│ Client B gets lock (token: 35)                             │
│ Client A tries to write with token 34                      │
│ Storage rejects: 34 < 35 (stale)                          │
└────────────────────────────────────────────────────────────┘
```

---

## 📊 Consistency Models

### Strong Consistency

```
After write completes, all reads see new value

Write(x=5) ──────────────────────────────────►
                     │
                     ▼ (write completes)
Read(x) ───────────────────────────────────► returns 5

Expensive: Requires synchronous replication
Used for: Financial transactions, inventory
```

### Eventual Consistency

```
After write, reads EVENTUALLY see new value

Write(x=5) ─────►
                  
Read(x) ───► returns 3 (stale)
Read(x) ───► returns 3 (still stale)
Read(x) ───► returns 5 (converged!)

Cheap: Asynchronous replication
Used for: Social media feeds, analytics
```

### Causal Consistency

```
If event A causes event B, everyone sees A before B

User posts: "I'm engaged!"              (A)
Friend comments: "Congratulations!"     (B, caused by A)

Causal consistency guarantees:
- No one sees comment without seeing post
- But might see other unrelated posts in different order

Stronger than eventual, weaker than strong
```

### Read Your Writes

```
After your write, YOUR reads see it
Other users might not yet

User A: Write(x=5)
User A: Read(x) → 5 (guaranteed)
User B: Read(x) → 3 (might be stale)

Implementation: Route reads to same replica as writes
Or: Track write timestamp, read from up-to-date replica
```

---

## 🔄 Replication Strategies

### Single Leader

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│              ┌────────────┐                                  │
│ Writes ────► │   Leader   │                                  │
│              └─────┬──────┘                                  │
│                    │ Replication                             │
│         ┌──────────┼──────────┐                              │
│         ▼          ▼          ▼                              │
│    ┌─────────┐┌─────────┐┌─────────┐                        │
│    │Follower1││Follower2││Follower3│ ◄──── Reads            │
│    └─────────┘└─────────┘└─────────┘                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘

Pros: Simple, no conflicts
Cons: Write bottleneck, leader failure = downtime
```

### Multi-Leader

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│    ┌────────────┐  ◄──►  ┌────────────┐                     │
│    │  Leader 1  │        │  Leader 2  │                     │
│    │ (US-East)  │        │ (EU-West)  │                     │
│    └─────┬──────┘        └──────┬─────┘                     │
│          │                      │                            │
│    ┌─────┴──────┐        ┌──────┴─────┐                     │
│    │ Followers  │        │ Followers  │                     │
│    └────────────┘        └────────────┘                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘

Pros: Write scalability, multi-region
Cons: Conflict resolution needed
```

### Conflict Resolution

```
Last Write Wins (LWW):
- Highest timestamp wins
- Simple but loses data

Merge / CRDT:
- Conflict-free Replicated Data Types
- Automatically merge without conflicts
- Example: Counter, Set, Register

Application Resolution:
- Store all versions
- Let application/user decide
- Example: Git merge conflicts
```

---

## 🌐 Distributed Transactions

### Two-Phase Commit (2PC)

```
Phase 1: Prepare
┌─────────────┐    prepare?    ┌─────────────┐
│ Coordinator │ ─────────────► │Participant 1│
│             │ ◄───── yes ─── │             │
│             │                └─────────────┘
│             │    prepare?    ┌─────────────┐
│             │ ─────────────► │Participant 2│
│             │ ◄───── yes ─── │             │
└─────────────┘                └─────────────┘

Phase 2: Commit (if all said yes)
┌─────────────┐    commit      ┌─────────────┐
│ Coordinator │ ─────────────► │Participant 1│
│             │ ◄───── ack ─── │             │
│             │                └─────────────┘
│             │    commit      ┌─────────────┐
│             │ ─────────────► │Participant 2│
│             │ ◄───── ack ─── │             │
└─────────────┘                └─────────────┘

Problems:
- Blocking: If coordinator crashes after prepare
- Latency: Multiple round trips
```

### Saga Pattern (Alternative)

```
Instead of distributed transaction:
Sequence of local transactions with compensating actions

T1 → T2 → T3 → T4 (success path)

If T3 fails:
T1 → T2 → T3(fail) → C2 → C1 (compensation)

Each step has undo action (compensation)
Eventually consistent, not atomic
```

---

## 📖 Further Reading

- "Designing Data-Intensive Applications" Ch. 8-9
- "Distributed Systems" by Van Steen
- MIT 6.824 Distributed Systems Course
- Jepsen.io (distributed systems testing)

---

**Next:** [Chapter 07: Networking Deep Dive →](./07-networking-deep-dive.md)


