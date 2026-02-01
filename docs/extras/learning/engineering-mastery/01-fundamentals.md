# Chapter 01: Computer Science Fundamentals

> "If you don't understand the fundamentals, you'll never build great systems."

---

## 🎯 Why This Matters

Every great system is built on solid fundamentals. When you understand:
- **How memory works** → You write efficient code
- **How algorithms scale** → You design for millions
- **How data structures work** → You choose the right tools

---

## 📊 Big O Notation - The Language of Scale

Big O describes how an algorithm's performance changes as input grows.

### Time Complexity Chart

```
Operations
    │
10⁹ │                                          O(n!)
    │                                        O(2ⁿ)
10⁶ │                                    O(n²)
    │                               O(n log n)
10³ │                          O(n)
    │                    O(log n)
  1 │ ─────────── O(1)
    └────────────────────────────────────────────
      1     10    100   1K    10K   100K   1M    Elements
```

### Common Complexities

| Complexity | Name | Example | 1M items |
|------------|------|---------|----------|
| O(1) | Constant | Hash lookup | 1 op |
| O(log n) | Logarithmic | Binary search | 20 ops |
| O(n) | Linear | Array scan | 1M ops |
| O(n log n) | Linearithmic | Merge sort | 20M ops |
| O(n²) | Quadratic | Nested loops | 1T ops ❌ |
| O(2ⁿ) | Exponential | Power set | ∞ ❌ |

### Real-World Implications

```javascript
// O(1) - HashMap lookup
const user = users.get(userId);  // Always fast, regardless of size

// O(n) - Finding in array
const user = users.find(u => u.id === userId);  // Slow at scale

// At 1 million users:
// HashMap: 1 operation = ~0.001ms
// Array:   1,000,000 operations = ~1000ms (1 second!)
```

---

## 🏗️ Essential Data Structures

### 1. Arrays & Linked Lists

```
Array (contiguous memory):
┌───┬───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │ 5 │  → Fast random access O(1)
└───┴───┴───┴───┴───┘  → Slow insert/delete O(n)

Linked List (scattered memory):
┌───┐   ┌───┐   ┌───┐
│ 1 │──►│ 2 │──►│ 3 │  → Fast insert/delete O(1)
└───┘   └───┘   └───┘  → Slow random access O(n)
```

**When to use:**
- Array: Read-heavy, fixed size, cache-friendly
- Linked List: Write-heavy, frequent insertions

### 2. Hash Tables (The Most Important)

```
Key "user_123" ──► hash() ──► index 7

Bucket Array:
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────────────┐
│  0  │  1  │  2  │  3  │  4  │  5  │  6  │ 7: user_123 │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────────────┘

Collision handling (chaining):
Bucket 7: user_123 ──► user_456 ──► user_789
```

**Hash Table Operations:**
| Operation | Average | Worst |
|-----------|---------|-------|
| Insert | O(1) | O(n) |
| Lookup | O(1) | O(n) |
| Delete | O(1) | O(n) |

**Real-world uses:**
- Database indexes
- Caching (Redis)
- Session storage
- Deduplication

### 3. Trees

```
Binary Search Tree:
           8
         /   \
        3     10
       / \      \
      1   6      14
         / \    /
        4   7  13

Properties:
- Left child < Parent < Right child
- Lookup: O(log n) average, O(n) worst
- Insert: O(log n) average
```

**Self-Balancing Trees (AVL, Red-Black):**
```
Unbalanced (bad):     Balanced (good):
    1                      4
     \                   /   \
      2                 2     6
       \               / \   / \
        3             1   3 5   7
         \
          4 → O(n) lookup!  → O(log n) guaranteed
```

**B-Trees (Used in Databases):**
```
        [10, 20, 30]
       /    |    \    \
   [1-9] [11-19] [21-29] [31-40]
   
- Each node has multiple keys
- Optimized for disk reads (fewer I/O operations)
- Used in PostgreSQL, MySQL indexes
```

### 4. Heaps (Priority Queues)

```
Min-Heap:
           1
         /   \
        3     2
       / \   / \
      7   5 4   6

Properties:
- Parent ≤ Children (min-heap)
- Get minimum: O(1)
- Insert/Delete: O(log n)
```

**Uses:**
- Task scheduling (process with highest priority)
- Dijkstra's algorithm
- Median finding
- Top-K problems

### 5. Graphs

```
Directed Graph:            Adjacency List:
    A ────► B              A: [B, C]
    │       │              B: [D]
    ▼       ▼              C: [D]
    C ────► D              D: []
```

**Traversal Algorithms:**

```
BFS (Breadth-First Search):        DFS (Depth-First Search):
Level by level                     Go deep first

    1 ─ Level 0                    Visit: 1 → 2 → 4 → 5 → 3 → 6
   / \                             
  2   3 ─ Level 1                       1
 / \   \                               /|\
4   5   6 ─ Level 2                   2 │ 3
                                     /| │  \
Visit: 1 → 2 → 3 → 4 → 5 → 6        4 5 ─  6
```

---

## 🧮 Essential Algorithms

### 1. Sorting Algorithms

| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Tim Sort | O(n) | O(n log n) | O(n log n) | O(n) | Yes |

**Tim Sort** (used in Python, Java): Hybrid of merge + insertion sort

### 2. Searching

```javascript
// Binary Search - O(log n)
function binarySearch(arr, target) {
  let left = 0, right = arr.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) left = mid + 1;
    else right = mid - 1;
  }
  
  return -1;
}

// At 1 billion items: ~30 comparisons!
```

### 3. Graph Algorithms

**Dijkstra's (Shortest Path):**
```
Find shortest path A → E:

    A ──2── B
    │       │
    1       3
    │       │
    C ──1── D ──1── E
    
Distances from A:
A=0, B=2, C=1, D=2, E=3
Path: A → C → D → E (cost: 3)
```

**Topological Sort (Dependency Resolution):**
```
Build order for:  A depends on B, B depends on C

    C → B → A
    
Build: C first, then B, then A
Used in: Package managers (npm), build systems
```

---

## 💾 Memory & Storage Fundamentals

### Memory Hierarchy

```
┌─────────────────────────────────────────────────────────┐
│              CPU Registers (1 cycle, bytes)              │
├─────────────────────────────────────────────────────────┤
│              L1 Cache (4 cycles, 64KB)                   │
├─────────────────────────────────────────────────────────┤
│              L2 Cache (10 cycles, 256KB)                 │
├─────────────────────────────────────────────────────────┤
│              L3 Cache (50 cycles, 8MB)                   │
├─────────────────────────────────────────────────────────┤
│              Main RAM (200 cycles, 16GB)                 │
├─────────────────────────────────────────────────────────┤
│              SSD (10,000 cycles, 500GB)                  │
├─────────────────────────────────────────────────────────┤
│              HDD (10,000,000 cycles, 2TB)                │
└─────────────────────────────────────────────────────────┘
              ↑ Speed                    ↓ Cost
```

### Cache-Friendly Code

```javascript
// BAD: Column-major access (cache misses)
for (let col = 0; col < 1000; col++) {
  for (let row = 0; row < 1000; row++) {
    matrix[row][col] = 0;  // Jumps in memory
  }
}

// GOOD: Row-major access (cache-friendly)
for (let row = 0; row < 1000; row++) {
  for (let col = 0; col < 1000; col++) {
    matrix[row][col] = 0;  // Sequential memory access
  }
}

// 10-100x faster due to cache hits!
```

### Stack vs Heap

```
Stack (automatic, fast):          Heap (manual, slower):
┌─────────────────────┐           ┌─────────────────────┐
│ Function call frame │           │    Large objects    │
├─────────────────────┤           │    Dynamic arrays   │
│ Local variables     │           │    Objects          │
├─────────────────────┤           │                     │
│ Return address      │           │  Fragmented memory  │
└─────────────────────┘           └─────────────────────┘

- Fixed size                      - Grows dynamically
- LIFO order                      - Random access
- Auto cleanup                    - Need GC/manual free
```

---

## 🔢 Numbers Every Programmer Should Know

```
L1 cache reference                           0.5 ns
Branch mispredict                            5   ns
L2 cache reference                           7   ns
Mutex lock/unlock                           25   ns
Main memory reference                      100   ns
Compress 1K bytes with Zippy             3,000   ns
Send 1K bytes over 1 Gbps network       10,000   ns
Read 4K randomly from SSD              150,000   ns
Read 1 MB sequentially from memory     250,000   ns
Round trip within same datacenter      500,000   ns
Read 1 MB sequentially from SSD      1,000,000   ns
Disk seek                           10,000,000   ns
Read 1 MB sequentially from disk    20,000,000   ns
Send packet CA→Netherlands→CA      150,000,000   ns
```

### Implications for System Design

| Operation | Time | Implication |
|-----------|------|-------------|
| Memory read | 100ns | Keep hot data in memory |
| SSD read | 150μs | Use SSDs for databases |
| Network (datacenter) | 500μs | Minimize service calls |
| Network (cross-region) | 150ms | Cache aggressively, async |

---

## 🧠 Problem-Solving Patterns

### 1. Two Pointers
```javascript
// Find pair that sums to target in sorted array
function twoSum(arr, target) {
  let left = 0, right = arr.length - 1;
  
  while (left < right) {
    const sum = arr[left] + arr[right];
    if (sum === target) return [left, right];
    if (sum < target) left++;
    else right--;
  }
  return null;
}
// O(n) instead of O(n²)
```

### 2. Sliding Window
```javascript
// Maximum sum of k consecutive elements
function maxSumWindow(arr, k) {
  let windowSum = arr.slice(0, k).reduce((a, b) => a + b);
  let maxSum = windowSum;
  
  for (let i = k; i < arr.length; i++) {
    windowSum += arr[i] - arr[i - k];  // Slide window
    maxSum = Math.max(maxSum, windowSum);
  }
  return maxSum;
}
// O(n) instead of O(n*k)
```

### 3. Divide & Conquer
```javascript
// Merge Sort
function mergeSort(arr) {
  if (arr.length <= 1) return arr;
  
  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));
  
  return merge(left, right);
}
// Divide problem, solve subproblems, combine
```

### 4. Dynamic Programming
```javascript
// Fibonacci with memoization
function fib(n, memo = {}) {
  if (n in memo) return memo[n];
  if (n <= 2) return 1;
  
  memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
  return memo[n];
}
// O(n) instead of O(2^n)

// Bottom-up (tabulation)
function fibTab(n) {
  const dp = [0, 1, 1];
  for (let i = 3; i <= n; i++) {
    dp[i] = dp[i-1] + dp[i-2];
  }
  return dp[n];
}
```

---

## 🎯 Practice Problems

| Problem | Concepts | Difficulty |
|---------|----------|------------|
| Two Sum | Hash Map | Easy |
| Valid Parentheses | Stack | Easy |
| Merge Intervals | Sorting | Medium |
| LRU Cache | Hash + DLL | Medium |
| Word Search | DFS/Backtrack | Medium |
| Median of Two Arrays | Binary Search | Hard |
| Serialize Binary Tree | Tree, Design | Hard |

**Resources:**
- LeetCode (focus on top 100)
- NeetCode (structured roadmap)
- Cracking the Coding Interview

---

## 📖 Further Reading

- "Introduction to Algorithms" (CLRS)
- "Algorithm Design Manual" by Skiena
- "Grokking Algorithms" (beginner-friendly)

---

**Next:** [Chapter 02: System Design Principles →](./02-system-design.md)


