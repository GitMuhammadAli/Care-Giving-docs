# 🧠 Memoization Patterns - Complete Guide

> A comprehensive guide to memoization in React and JavaScript - React.memo, useMemo, useCallback, computation caching, and knowing when NOT to memoize.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Memoization caches the result of expensive computations or component renders, returning the cached value when inputs haven't changed - trading memory for CPU cycles."

### The Memoization Mental Model
```
WITHOUT MEMOIZATION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Parent re-renders                                              │
│       ↓                                                         │
│  Child re-renders (even if props unchanged)                    │
│       ↓                                                         │
│  Expensive calculation runs again                              │
│       ↓                                                         │
│  UI jank / slow performance                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

WITH MEMOIZATION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Parent re-renders                                              │
│       ↓                                                         │
│  React.memo checks: props changed?                             │
│       ↓                                                         │
│  NO → Return cached render (skip re-render)                    │
│  YES → Re-render child                                         │
│                                                                  │
│  useMemo checks: deps changed?                                 │
│       ↓                                                         │
│  NO → Return cached value                                      │
│  YES → Run expensive calculation                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Numbers to Remember
| Metric | Value | Context |
|--------|-------|---------|
| React.memo overhead | **~0.1ms** | Shallow comparison cost |
| Worth memoizing if | **>1ms saved** | Rule of thumb |
| useMemo threshold | **>100 items** | For list operations |
| Reference equality | **O(1)** | vs deep compare O(n) |

### The "Wow" Statement (Memorize This!)
> "We had a dashboard re-rendering 500ms on every keystroke because a parent component's state change caused all children to re-render, including a component with expensive data transformation. I wrapped it with React.memo and moved the transformation into useMemo with proper deps. Render time dropped to 5ms. The key insight was that useMemo alone isn't enough - if the parent passes a new object/function reference each render, React.memo's shallow compare fails. So I also wrapped the callback prop in useCallback. But I'm careful not to over-memoize - premature memoization adds complexity and memory overhead for components that render fast anyway."

### Key Terms to Drop
| Term | Use It Like This |
|------|------------------|
| **"Referential equality"** | "useCallback preserves referential equality so React.memo's shallow compare works" |
| **"Shallow comparison"** | "React.memo does shallow comparison - won't detect nested object changes" |
| **"Stable reference"** | "Need a stable reference for the dependency array" |
| **"Memoization overhead"** | "The memoization overhead exceeds the render cost - not worth it" |

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "When should you use React.memo, useMemo, and useCallback?"

**Junior Answer:**
> "Use them everywhere to make things faster."

**Senior Answer:**
> "Memoization has overhead - memory and comparison costs. Use it strategically:

**React.memo** - Wrap component when:
- Parent re-renders frequently with same props
- Component render is expensive (complex UI, many children)
- Props are primitives or stable references

**useMemo** - Cache computation when:
- Filtering/sorting large arrays (>100 items)
- Complex calculations that don't need to run every render
- Creating objects/arrays passed to memoized children

**useCallback** - Stabilize function when:
- Passing callback to memoized child (prevents breaking React.memo)
- Function is dependency of useEffect/useMemo
- NOT for every function - only when reference stability matters

**Don't memoize when:**
- Component is already fast (<1ms render)
- Props change every render anyway
- Premature optimization - measure first!"

---

## 📚 Core Concepts

### React.memo

```tsx
// ════════════════════════════════════════════════════════════════
// REACT.MEMO: Prevent re-renders when props unchanged
// ════════════════════════════════════════════════════════════════

// ❌ WITHOUT MEMO: Re-renders every time parent re-renders
const ExpensiveList = ({ items, onSelect }) => {
    console.log('Rendering ExpensiveList'); // Logs on EVERY parent render
    return (
        <ul>
            {items.map(item => (
                <li key={item.id} onClick={() => onSelect(item)}>
                    {item.name}
                </li>
            ))}
        </ul>
    );
};

// ✅ WITH MEMO: Only re-renders when props actually change
const ExpensiveList = React.memo(({ items, onSelect }) => {
    console.log('Rendering ExpensiveList'); // Only logs when items/onSelect change
    return (
        <ul>
            {items.map(item => (
                <li key={item.id} onClick={() => onSelect(item)}>
                    {item.name}
                </li>
            ))}
        </ul>
    );
});

// ════════════════════════════════════════════════════════════════
// CUSTOM COMPARISON FUNCTION
// ════════════════════════════════════════════════════════════════

const UserCard = React.memo(
    ({ user, onEdit }) => {
        return (
            <div>
                <h2>{user.name}</h2>
                <button onClick={onEdit}>Edit</button>
            </div>
        );
    },
    // Custom comparison: only re-render if user.id changes
    (prevProps, nextProps) => {
        return prevProps.user.id === nextProps.user.id;
    }
);

// ⚠️ WARNING: Custom comparison can cause bugs if you forget to compare important props!
```

### useMemo

```tsx
// ════════════════════════════════════════════════════════════════
// USEMEMO: Cache expensive computations
// ════════════════════════════════════════════════════════════════

function ProductList({ products, filter, sortBy }) {
    // ❌ BAD: Runs on EVERY render, even if products/filter unchanged
    const filteredProducts = products
        .filter(p => p.category === filter)
        .sort((a, b) => a[sortBy] - b[sortBy]);
    
    // ✅ GOOD: Only recalculates when dependencies change
    const filteredProducts = useMemo(() => {
        console.log('Filtering and sorting...'); // Only logs when deps change
        return products
            .filter(p => p.category === filter)
            .sort((a, b) => a[sortBy] - b[sortBy]);
    }, [products, filter, sortBy]);
    
    return <List items={filteredProducts} />;
}

// ════════════════════════════════════════════════════════════════
// USEMEMO FOR REFERENTIAL EQUALITY (passing to memoized child)
// ════════════════════════════════════════════════════════════════

function Parent() {
    const [count, setCount] = useState(0);
    
    // ❌ BAD: New object every render → breaks child's React.memo
    const config = { theme: 'dark', size: 'large' };
    
    // ✅ GOOD: Same reference if values unchanged
    const config = useMemo(() => ({ 
        theme: 'dark', 
        size: 'large' 
    }), []); // Empty deps = never changes
    
    return (
        <>
            <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
            <MemoizedChild config={config} />
        </>
    );
}

// ════════════════════════════════════════════════════════════════
// WHEN NOT TO USE USEMEMO
// ════════════════════════════════════════════════════════════════

function SimpleComponent({ name }) {
    // ❌ UNNECESSARY: String concatenation is cheap
    const greeting = useMemo(() => `Hello, ${name}!`, [name]);
    
    // ✅ JUST DO IT: No memoization needed for simple operations
    const greeting = `Hello, ${name}!`;
    
    return <h1>{greeting}</h1>;
}
```

### useCallback

```tsx
// ════════════════════════════════════════════════════════════════
// USECALLBACK: Stabilize function references
// ════════════════════════════════════════════════════════════════

function Parent() {
    const [count, setCount] = useState(0);
    const [items, setItems] = useState([]);
    
    // ❌ BAD: New function every render → MemoizedChild re-renders
    const handleSelect = (item) => {
        console.log('Selected:', item);
    };
    
    // ✅ GOOD: Same function reference across renders
    const handleSelect = useCallback((item) => {
        console.log('Selected:', item);
    }, []); // Empty deps if no external dependencies
    
    // ✅ WITH DEPENDENCIES: New function only when setItems changes
    const handleAdd = useCallback((item) => {
        setItems(prev => [...prev, item]);
    }, []); // setItems is stable, no need in deps
    
    return (
        <>
            <button onClick={() => setCount(c => c + 1)}>
                Count: {count}
            </button>
            <MemoizedList items={items} onSelect={handleSelect} />
        </>
    );
}

// ════════════════════════════════════════════════════════════════
// USECALLBACK + USEMEMO TOGETHER
// ════════════════════════════════════════════════════════════════

function SearchableList({ items }) {
    const [query, setQuery] = useState('');
    const [sortOrder, setSortOrder] = useState('asc');
    
    // Memoize filtered results
    const filteredItems = useMemo(() => {
        return items
            .filter(item => item.name.includes(query))
            .sort((a, b) => 
                sortOrder === 'asc' 
                    ? a.name.localeCompare(b.name)
                    : b.name.localeCompare(a.name)
            );
    }, [items, query, sortOrder]);
    
    // Stabilize callback
    const handleItemClick = useCallback((item) => {
        console.log('Clicked:', item.id);
    }, []);
    
    return (
        <>
            <input 
                value={query} 
                onChange={e => setQuery(e.target.value)} 
            />
            <MemoizedItemList 
                items={filteredItems} 
                onItemClick={handleItemClick} 
            />
        </>
    );
}
```

### General Computation Memoization

```typescript
// ════════════════════════════════════════════════════════════════
// MEMOIZATION OUTSIDE REACT
// ════════════════════════════════════════════════════════════════

// Simple memoization function
function memoize<T extends (...args: any[]) => any>(fn: T): T {
    const cache = new Map();
    
    return ((...args: Parameters<T>) => {
        const key = JSON.stringify(args);
        
        if (cache.has(key)) {
            return cache.get(key);
        }
        
        const result = fn(...args);
        cache.set(key, result);
        return result;
    }) as T;
}

// Usage
const expensiveCalculation = memoize((n: number) => {
    console.log('Computing...');
    let result = 0;
    for (let i = 0; i < n * 1000000; i++) {
        result += i;
    }
    return result;
});

expensiveCalculation(100); // Computing... (calculates)
expensiveCalculation(100); // Returns cached (instant)
expensiveCalculation(200); // Computing... (new input)

// ════════════════════════════════════════════════════════════════
// LRU CACHE (Limited memory)
// ════════════════════════════════════════════════════════════════

class LRUCache<K, V> {
    private cache = new Map<K, V>();
    private maxSize: number;
    
    constructor(maxSize: number) {
        this.maxSize = maxSize;
    }
    
    get(key: K): V | undefined {
        if (!this.cache.has(key)) return undefined;
        
        // Move to end (most recently used)
        const value = this.cache.get(key)!;
        this.cache.delete(key);
        this.cache.set(key, value);
        return value;
    }
    
    set(key: K, value: V): void {
        if (this.cache.has(key)) {
            this.cache.delete(key);
        } else if (this.cache.size >= this.maxSize) {
            // Delete oldest (first item)
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
        }
        this.cache.set(key, value);
    }
}

// Usage with memoization
function memoizeWithLRU<T extends (...args: any[]) => any>(
    fn: T, 
    maxSize = 100
): T {
    const cache = new LRUCache<string, ReturnType<T>>(maxSize);
    
    return ((...args: Parameters<T>) => {
        const key = JSON.stringify(args);
        const cached = cache.get(key);
        
        if (cached !== undefined) {
            return cached;
        }
        
        const result = fn(...args);
        cache.set(key, result);
        return result;
    }) as T;
}
```

---

## Common Pitfalls

```
MEMOIZATION PITFALLS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. MEMOIZING EVERYTHING                                        │
│     Problem: Overhead exceeds benefit for simple components    │
│     Solution: Measure first, memoize expensive renders only    │
│                                                                  │
│  2. FORGETTING USECALLBACK WITH REACT.MEMO                     │
│     Problem: New function reference breaks memoization         │
│     Solution: Wrap callbacks in useCallback                    │
│                                                                  │
│  3. WRONG DEPENDENCIES                                          │
│     Problem: Missing deps = stale values, extra deps = no cache│
│     Solution: Use ESLint exhaustive-deps rule                  │
│                                                                  │
│  4. OBJECT/ARRAY IN DEPS                                        │
│     Problem: New reference every render breaks cache           │
│     Solution: Memoize objects or use primitive deps            │
│                                                                  │
│  5. INLINE OBJECTS AS PROPS                                     │
│     Problem: style={{color:'red'}} = new object each render   │
│     Solution: Define outside component or use useMemo          │
│                                                                  │
│  6. MEMORY LEAKS                                                │
│     Problem: Unlimited cache grows forever                     │
│     Solution: Use LRU cache with max size                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "When should you NOT use React.memo?"**
> "When component is already fast (<1ms render), when props change every render anyway (memoization overhead wasted), when component is simple with few children, or when optimizing prematurely before measuring. Always profile first - React DevTools Profiler shows render times."

**Q: "What's the difference between useMemo and useCallback?"**
> "useMemo caches a computed VALUE, useCallback caches a FUNCTION. useCallback(fn, deps) is equivalent to useMemo(() => fn, deps). Use useMemo for expensive calculations, useCallback for stabilizing function references passed to memoized children."

**Q: "Why would React.memo not work even though you wrapped the component?"**
> "Common causes: 1) Passing new object/array/function reference each render - use useMemo/useCallback. 2) Props actually are changing - verify with React DevTools. 3) Children prop changes - JSX children are objects. 4) Context value changes - context updates bypass memo."

**Q: "How do you decide what to memoize?"**
> "I follow this process: 1) Identify slow renders with React DevTools Profiler. 2) Check if component re-renders unnecessarily (props unchanged). 3) If yes, add React.memo. 4) If still slow, check for expensive computations → useMemo. 5) Check if passing callbacks to memoized children → useCallback. Don't memoize speculatively."

---

## Quick Reference

```
MEMOIZATION CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  REACT.MEMO:                                                    │
│  • Prevents re-render when props unchanged                     │
│  • Uses shallow comparison by default                          │
│  • Requires stable prop references (objects, functions)        │
│                                                                  │
│  USEMEMO:                                                       │
│  • Caches computed values                                      │
│  • Use for: filtering, sorting, complex calculations           │
│  • Use for: creating stable object references                  │
│                                                                  │
│  USECALLBACK:                                                   │
│  • Caches function references                                  │
│  • Use for: callbacks passed to memoized children              │
│  • Use for: functions in useEffect/useMemo deps                │
│                                                                  │
│  WHEN TO MEMOIZE:                                               │
│  ✓ Expensive renders (>1ms)                                    │
│  ✓ Large list operations (>100 items)                          │
│  ✓ Callbacks to memoized children                              │
│                                                                  │
│  WHEN NOT TO MEMOIZE:                                           │
│  ✗ Simple components                                           │
│  ✗ Props change every render                                   │
│  ✗ Before measuring performance                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

*Guide covers React 18+ patterns. Some patterns differ in older versions.*


