# 🧹 Memory Management - Complete Guide

> A comprehensive guide to memory management in JavaScript/Node.js - memory leaks, garbage collection, heap analysis, profiling, and common leak patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Memory management in JavaScript means understanding how garbage collection works, identifying memory leaks through heap snapshots and allocation timelines, and avoiding common patterns like forgotten timers, detached DOM nodes, and closure-captured references."

### The Memory Mental Model
```
JAVASCRIPT MEMORY LIFECYCLE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. ALLOCATE                                                    │
│     const obj = { data: 'hello' };  // Memory allocated        │
│     const arr = new Array(1000000); // More memory             │
│                                                                  │
│  2. USE                                                         │
│     console.log(obj.data);          // Memory in use           │
│                                                                  │
│  3. RELEASE (Garbage Collection)                                │
│     obj = null;  // No more references                         │
│     // GC eventually frees the memory                          │
│                                                                  │
│  MEMORY LEAK = Step 3 never happens                            │
│  Object is no longer needed but still referenced               │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  HEAP MEMORY                                              │  │
│  │                                                           │  │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐              │  │
│  │  │ Obj │ │ Obj │ │LEAK │ │ Obj │ │LEAK │ ← Growing!   │  │
│  │  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘              │  │
│  │                                                           │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Numbers to Remember
| Metric | Value | Context |
|--------|-------|---------|
| V8 heap limit (default) | **~1.5GB** | Node.js, can increase |
| Browser tab limit | **~2-4GB** | Varies by browser |
| Healthy heap growth | **Sawtooth pattern** | Up, GC, down, repeat |
| Memory leak sign | **Staircase pattern** | Keeps growing after GC |
| Major GC pause | **10-100ms** | Can cause jank |

### The "Wow" Statement
> "We had a Node.js service that crashed every 8 hours with OOM. I took heap snapshots at startup and after 4 hours, compared them in Chrome DevTools - found 500MB of objects retained by event listeners that were never removed. The pattern: we subscribed to events in a class constructor but never unsubscribed in cleanup. After adding proper removeListener calls in destroy(), memory stayed flat. The key debugging technique was the 'Comparison' view in heap snapshots showing objects allocated between snapshots that weren't collected."

### Key Terms to Drop
| Term | Use It Like This |
|------|------------------|
| **"Heap snapshot"** | "I took heap snapshots to compare memory allocation over time" |
| **"Retained size"** | "The retained size shows this closure is keeping 50MB alive" |
| **"Detached DOM"** | "Found detached DOM nodes - removed from page but still referenced" |
| **"GC roots"** | "Traced the retainer path back to GC roots to find the leak" |
| **"Allocation timeline"** | "Used allocation timeline to see objects created during the leak" |

---

## 📚 Core Concepts

### Common Memory Leak Patterns

```javascript
// ════════════════════════════════════════════════════════════════
// LEAK PATTERN 1: Forgotten Timers
// ════════════════════════════════════════════════════════════════

// ❌ LEAK: Timer keeps reference to largeData even after component unmounts
function startPolling() {
    const largeData = new Array(1000000).fill('data');
    
    setInterval(() => {
        console.log(largeData.length); // Closure captures largeData
    }, 1000);
}
// Timer runs forever, largeData never freed

// ✅ FIX: Clear timer on cleanup
function startPolling() {
    const largeData = new Array(1000000).fill('data');
    
    const intervalId = setInterval(() => {
        console.log(largeData.length);
    }, 1000);
    
    // Return cleanup function
    return () => clearInterval(intervalId);
}

// React component example
useEffect(() => {
    const cleanup = startPolling();
    return cleanup; // Clear on unmount
}, []);

// ════════════════════════════════════════════════════════════════
// LEAK PATTERN 2: Event Listeners Never Removed
// ════════════════════════════════════════════════════════════════

// ❌ LEAK: Listener holds reference, accumulates on every call
class DataFetcher {
    constructor() {
        this.data = new Array(1000000).fill('data');
        window.addEventListener('resize', this.handleResize);
    }
    
    handleResize = () => {
        console.log('Resized, data:', this.data.length);
    }
}

// Creating new instances leaks old ones
let fetcher = new DataFetcher();
fetcher = new DataFetcher(); // Old fetcher's listener still active!

// ✅ FIX: Remove listener on destroy
class DataFetcher {
    constructor() {
        this.data = new Array(1000000).fill('data');
        window.addEventListener('resize', this.handleResize);
    }
    
    handleResize = () => {
        console.log('Resized, data:', this.data.length);
    }
    
    destroy() {
        window.removeEventListener('resize', this.handleResize);
    }
}

// ════════════════════════════════════════════════════════════════
// LEAK PATTERN 3: Closures Capturing Large Objects
// ════════════════════════════════════════════════════════════════

// ❌ LEAK: Closure captures entire scope
function processData(hugeArray) {
    const processed = hugeArray.map(x => x * 2);
    
    return function getLength() {
        // Only needs processed.length, but closure captures 'hugeArray' too!
        return processed.length;
    };
}

// ✅ FIX: Only capture what you need
function processData(hugeArray) {
    const processed = hugeArray.map(x => x * 2);
    const length = processed.length;
    
    return function getLength() {
        return length; // Only captures the number
    };
}

// ════════════════════════════════════════════════════════════════
// LEAK PATTERN 4: Detached DOM Nodes
// ════════════════════════════════════════════════════════════════

// ❌ LEAK: DOM removed but reference kept
let elements = [];

function addElement() {
    const el = document.createElement('div');
    el.innerHTML = '<p>Lots of content...</p>'.repeat(1000);
    document.body.appendChild(el);
    elements.push(el); // Reference stored
}

function removeElements() {
    elements.forEach(el => el.remove()); // Removed from DOM
    // But elements array still holds references!
}

// ✅ FIX: Clear references
function removeElements() {
    elements.forEach(el => el.remove());
    elements = []; // Clear references
}

// ════════════════════════════════════════════════════════════════
// LEAK PATTERN 5: Growing Collections
// ════════════════════════════════════════════════════════════════

// ❌ LEAK: Cache grows forever
const cache = new Map();

function getData(key) {
    if (!cache.has(key)) {
        cache.set(key, expensiveComputation(key));
    }
    return cache.get(key);
}
// Cache never cleared, grows until OOM

// ✅ FIX: Use LRU cache or WeakMap
const cache = new Map();
const MAX_CACHE_SIZE = 1000;

function getData(key) {
    if (!cache.has(key)) {
        if (cache.size >= MAX_CACHE_SIZE) {
            // Delete oldest entry
            const firstKey = cache.keys().next().value;
            cache.delete(firstKey);
        }
        cache.set(key, expensiveComputation(key));
    }
    return cache.get(key);
}
```

### WeakMap and WeakSet

```javascript
// ════════════════════════════════════════════════════════════════
// WEAKMAP: References don't prevent garbage collection
// ════════════════════════════════════════════════════════════════

// Regular Map holds strong references
const strongMap = new Map();
let obj = { data: 'important' };
strongMap.set(obj, 'metadata');
obj = null; // Object STILL exists - Map holds reference

// WeakMap holds weak references
const weakMap = new WeakMap();
let obj2 = { data: 'important' };
weakMap.set(obj2, 'metadata');
obj2 = null; // Object CAN be garbage collected!

// ════════════════════════════════════════════════════════════════
// USE CASE: Caching data for DOM elements
// ════════════════════════════════════════════════════════════════

const elementData = new WeakMap();

function storeElementData(element, data) {
    elementData.set(element, data);
}

function getElementData(element) {
    return elementData.get(element);
}

// When element is removed from DOM and dereferenced,
// its cached data is automatically garbage collected!

// ════════════════════════════════════════════════════════════════
// USE CASE: Private data
// ════════════════════════════════════════════════════════════════

const privateData = new WeakMap();

class User {
    constructor(name, password) {
        this.name = name;
        privateData.set(this, { password }); // Private!
    }
    
    checkPassword(input) {
        return privateData.get(this).password === input;
    }
}
```

### Profiling Memory

```javascript
// ════════════════════════════════════════════════════════════════
// NODE.JS MEMORY MONITORING
// ════════════════════════════════════════════════════════════════

// Check memory usage
function logMemory() {
    const used = process.memoryUsage();
    console.log({
        heapUsed: Math.round(used.heapUsed / 1024 / 1024) + ' MB',
        heapTotal: Math.round(used.heapTotal / 1024 / 1024) + ' MB',
        external: Math.round(used.external / 1024 / 1024) + ' MB',
        rss: Math.round(used.rss / 1024 / 1024) + ' MB'
    });
}

// Monitor over time
setInterval(logMemory, 5000);

// Force garbage collection (requires --expose-gc flag)
// node --expose-gc app.js
if (global.gc) {
    global.gc();
    console.log('Manual GC triggered');
}

// ════════════════════════════════════════════════════════════════
// HEAP SNAPSHOT IN NODE.JS
// ════════════════════════════════════════════════════════════════

const v8 = require('v8');
const fs = require('fs');

function takeHeapSnapshot() {
    const snapshotStream = v8.writeHeapSnapshot();
    console.log(`Heap snapshot written to ${snapshotStream}`);
    // Open .heapsnapshot file in Chrome DevTools
}

// Take snapshot on demand via signal
process.on('SIGUSR2', takeHeapSnapshot);

// ════════════════════════════════════════════════════════════════
// CHROME DEVTOOLS PROFILING
// ════════════════════════════════════════════════════════════════

/*
1. Open DevTools → Memory tab

2. HEAP SNAPSHOT
   - Take snapshot at different times
   - Compare snapshots to find leaks
   - Look at "Objects allocated between Snapshot 1 and 2"
   
3. ALLOCATION TIMELINE
   - Record while performing action
   - Blue bars = allocated, gray = freed
   - Persistent blue bars = potential leaks
   
4. ALLOCATION SAMPLING
   - Lower overhead profiling
   - Shows where memory is allocated

KEY VIEWS:
- Summary: Objects by constructor
- Comparison: Diff between snapshots
- Containment: Object hierarchy
- Dominators: What's keeping objects alive

FINDING LEAKS:
1. Take snapshot 1
2. Perform suspected leaky action multiple times
3. Take snapshot 2
4. Select "Objects allocated between Snapshot 1 and 2"
5. Look for objects that shouldn't exist
6. Check "Retainers" to see what's holding reference
*/
```

---

## Common Pitfalls

```
MEMORY MANAGEMENT PITFALLS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. FORGOTTEN CLEANUP                                           │
│     Problem: Event listeners, timers, subscriptions            │
│     Solution: Always clean up in destroy/unmount               │
│                                                                  │
│  2. CLOSURE SCOPE CAPTURE                                       │
│     Problem: Closures keep entire scope alive                  │
│     Solution: Only reference what you need                     │
│                                                                  │
│  3. GLOBAL VARIABLES                                            │
│     Problem: Never garbage collected                           │
│     Solution: Avoid globals, use modules                       │
│                                                                  │
│  4. INFINITE CACHES                                             │
│     Problem: Maps/objects grow forever                         │
│     Solution: LRU cache, WeakMap, TTL                          │
│                                                                  │
│  5. CONSOLE.LOG IN PRODUCTION                                   │
│     Problem: Console retains logged objects                    │
│     Solution: Remove or disable in production                  │
│                                                                  │
│  6. CIRCULAR REFERENCES                                         │
│     Problem: Objects reference each other                      │
│     Note: Modern GC handles this, but complicates debugging    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "How do you identify and fix memory leaks?"**
> "I use Chrome DevTools Memory tab. Take heap snapshots before and after suspected leak, use Comparison view to see objects allocated that weren't freed. Check Retainers panel to see what's holding references. Common culprits: event listeners not removed, timers not cleared, closures capturing large objects. Fix by ensuring proper cleanup."

**Q: "Explain garbage collection in JavaScript"**
> "V8 uses generational GC. New objects go to 'young generation', collected frequently with Scavenge algorithm. Surviving objects promote to 'old generation', collected less often with Mark-Sweep. Objects are collected when unreachable from GC roots (global object, stack). Memory leaks happen when objects are still reachable but no longer needed."

**Q: "What's the difference between Map and WeakMap?"**
> "Map holds strong references - objects used as keys won't be garbage collected. WeakMap holds weak references - if nothing else references the key object, it can be collected along with its value. Use WeakMap for caching data associated with objects that might be removed, like DOM elements."

---

## Quick Reference

```
MEMORY MANAGEMENT CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  COMMON LEAKS:                                                  │
│  • Timers: Always clearInterval/clearTimeout                   │
│  • Events: Always removeEventListener                          │
│  • Closures: Don't capture unnecessary scope                   │
│  • Collections: Use LRU/WeakMap, set limits                    │
│  • DOM: Clear references when removing elements                │
│                                                                  │
│  DEBUGGING TOOLS:                                               │
│  • Heap Snapshot: Compare before/after                         │
│  • Allocation Timeline: See what's allocated                   │
│  • Retainers: Find what holds references                       │
│  • process.memoryUsage(): Node.js monitoring                   │
│                                                                  │
│  PATTERNS:                                                      │
│  • Healthy: Sawtooth (grow, GC, drop)                          │
│  • Leak: Staircase (keeps growing)                             │
│                                                                  │
│  NODE.JS:                                                       │
│  • --max-old-space-size=4096 (increase heap)                   │
│  • --expose-gc (manual GC for testing)                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


