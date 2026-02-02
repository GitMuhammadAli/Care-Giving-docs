# ⚙️ Virtual DOM & Reconciliation - Complete Guide

> A comprehensive guide to Virtual DOM - how React works under the hood, the reconciliation algorithm, keys, rendering optimization, and building performant applications.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "The Virtual DOM is a lightweight JavaScript representation of the actual DOM; React uses a reconciliation algorithm to efficiently diff the old and new virtual trees and apply only the minimum necessary DOM changes."

### The 7 Key Concepts (Remember These!)
```
1. VIRTUAL DOM     → JS object tree representing UI
2. RECONCILIATION  → Algorithm to diff old vs new VDOM
3. FIBER           → React's internal work unit
4. KEYS            → Help identify which items changed in lists
5. BATCHING        → Group multiple updates into one render
6. CONCURRENT      → Interruptible rendering (React 18+)
7. COMMIT PHASE    → When React actually updates the DOM
```

### React Rendering Phases
```
┌─────────────────────────────────────────────────────────────────┐
│              REACT RENDERING PHASES                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TRIGGER                                                       │
│  ───────                                                        │
│  • Initial render (createRoot)                                 │
│  • State update (setState, useState setter)                    │
│  • Parent re-render                                            │
│  • Context change                                              │
│                                                                 │
│           ↓                                                    │
│                                                                 │
│  RENDER PHASE (Pure, can be interrupted)                       │
│  ──────────────────────────────────────                         │
│  • Call function components                                    │
│  • Create new Virtual DOM tree                                 │
│  • Diff with previous tree (reconciliation)                    │
│  • Determine what needs to change                              │
│  • NO side effects here!                                       │
│                                                                 │
│           ↓                                                    │
│                                                                 │
│  COMMIT PHASE (Synchronous, cannot interrupt)                  │
│  ─────────────────────────────────────────────                  │
│  • Apply DOM changes                                           │
│  • Run layout effects (useLayoutEffect)                        │
│  • Run effects (useEffect) - after paint                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Reconciliation Algorithm
```
┌─────────────────────────────────────────────────────────────────┐
│              RECONCILIATION RULES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  RULE 1: Different types = rebuild                             │
│  ─────────────────────────────────                              │
│  <div> → <span>  : Destroy div tree, build span tree           │
│  <ComponentA> → <ComponentB> : Unmount A, mount B              │
│                                                                 │
│  RULE 2: Same type = update                                    │
│  ───────────────────────────                                    │
│  <div className="a"> → <div className="b">                     │
│  Only update className attribute                               │
│                                                                 │
│  RULE 3: Lists need keys                                       │
│  ────────────────────────                                       │
│  Without keys: Rebuild entire list                             │
│  With keys: Only update changed items                          │
│                                                                 │
│  ALGORITHM COMPLEXITY:                                         │
│  ──────────────────────                                         │
│  General diff: O(n³) - too slow                                │
│  React's heuristic: O(n) - fast enough                         │
│  Assumptions make it possible:                                 │
│  1. Different types = different trees                          │
│  2. Keys identify stable elements                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Fiber"** | "React Fiber enabled concurrent rendering by making work interruptible" |
| **"Reconciliation"** | "Reconciliation diffs the virtual DOM trees" |
| **"Commit phase"** | "DOM mutations happen in the commit phase" |
| **"Work unit"** | "Each fiber is a unit of work that can be paused" |
| **"Batching"** | "React 18 batches all state updates automatically" |
| **"Time slicing"** | "Concurrent mode uses time slicing to stay responsive" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Diff complexity | **O(n)** | Heuristic-based algorithm |
| Frame budget | **16ms** | 60fps target |
| Batching | **Automatic** | React 18+ |
| Priority levels | **5** | Lane priorities in Fiber |

### The "Wow" Statement (Memorize This!)
> "React maintains a virtual DOM - a lightweight JS object tree. On state change, React creates a new virtual tree and diffs it against the old one using reconciliation. The algorithm is O(n) thanks to two assumptions: different types mean different trees, and keys identify stable elements. React Fiber made this interruptible - work is broken into units (fibers) that can pause for higher priority updates. The render phase is pure and can run in concurrent mode. The commit phase is synchronous - that's when DOM actually updates. React 18's automatic batching groups all state updates, even in async callbacks. Keys are critical for list performance - without them, React can't identify which items moved."

---

## 📚 Table of Contents

1. [Virtual DOM Basics](#1-virtual-dom-basics)
2. [Reconciliation Algorithm](#2-reconciliation-algorithm)
3. [Keys](#3-keys)
4. [React Fiber](#4-react-fiber)
5. [Rendering Optimization](#5-rendering-optimization)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Virtual DOM Basics

```tsx
// ════════════════════════════════════════════════════════════════
// WHAT IS VIRTUAL DOM?
// ════════════════════════════════════════════════════════════════

// Virtual DOM is just JavaScript objects representing DOM structure

// This JSX:
<div className="container">
  <h1>Hello</h1>
  <p>World</p>
</div>

// Becomes this Virtual DOM (simplified):
{
  type: 'div',
  props: {
    className: 'container',
    children: [
      {
        type: 'h1',
        props: {
          children: 'Hello'
        }
      },
      {
        type: 'p',
        props: {
          children: 'World'
        }
      }
    ]
  }
}

// ════════════════════════════════════════════════════════════════
// REACT ELEMENTS vs COMPONENTS
// ════════════════════════════════════════════════════════════════

// React Element - Plain object describing what to render
const element = {
  type: 'button',
  props: {
    className: 'btn',
    children: 'Click'
  }
};

// Same as JSX:
const element = <button className="btn">Click</button>;

// React.createElement call:
const element = React.createElement(
  'button',
  { className: 'btn' },
  'Click'
);

// Component - Function that returns elements
function Button({ label }) {
  return <button className="btn">{label}</button>;
}

// Component element:
const componentElement = {
  type: Button,  // Reference to function
  props: {
    label: 'Click'
  }
};

// ════════════════════════════════════════════════════════════════
// WHY VIRTUAL DOM?
// ════════════════════════════════════════════════════════════════

// Direct DOM manipulation is expensive
document.getElementById('root').innerHTML = '<div>...</div>';
// Rebuilds entire DOM tree!

// Virtual DOM approach:
// 1. Build new virtual tree (fast - just JS objects)
// 2. Diff against old virtual tree (fast - O(n))
// 3. Apply only changed nodes to real DOM (minimal changes)

// Example: Updating one item in a list of 1000
// Without VDOM: Rebuild all 1000 DOM nodes
// With VDOM: Update only 1 DOM node

// ════════════════════════════════════════════════════════════════
// RENDER FUNCTION VISUALIZATION
// ════════════════════════════════════════════════════════════════

function App() {
  const [count, setCount] = useState(0);
  
  // Every render creates NEW elements
  // But they're cheap JS objects
  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}

// Render 1: count = 0
// Virtual DOM tree with "Count: 0"

// Click button, setCount(1)
// Render 2: count = 1
// NEW Virtual DOM tree with "Count: 1"

// React diffs trees:
// - div: same type, check children
// - h1: same type, children changed "Count: 0" → "Count: 1"
// - button: same type, no changes

// Commit: Only update text node in h1
```

---

## 2. Reconciliation Algorithm

```tsx
// ════════════════════════════════════════════════════════════════
// RECONCILIATION RULES
// ════════════════════════════════════════════════════════════════

// RULE 1: Elements of different types produce different trees

// Before
<div>
  <Counter />
</div>

// After
<span>
  <Counter />
</span>

// React will:
// 1. Destroy <div> and all children (unmount Counter)
// 2. Build <span> and all children (mount new Counter)
// Counter's state is LOST!

// ════════════════════════════════════════════════════════════════

// RULE 2: Same type - update attributes/props

// Before
<div className="before" title="old" />

// After  
<div className="after" title="old" />

// React will:
// 1. Keep the same DOM node
// 2. Update only className attribute
// Component instances stay mounted, state is preserved

// ════════════════════════════════════════════════════════════════

// RULE 3: Same component type - update props

// Before
<Counter count={1} />

// After
<Counter count={2} />

// React will:
// 1. Keep the same Counter instance
// 2. Call component with new props
// State is preserved!

// ════════════════════════════════════════════════════════════════
// RECURSION ON CHILDREN
// ════════════════════════════════════════════════════════════════

// Simple case - children compared in order

// Before
<ul>
  <li>Apple</li>
  <li>Banana</li>
</ul>

// After
<ul>
  <li>Apple</li>
  <li>Banana</li>
  <li>Cherry</li>  {/* Added at end */}
</ul>

// React compares:
// 1. <li>Apple</li> = <li>Apple</li> ✓ No change
// 2. <li>Banana</li> = <li>Banana</li> ✓ No change
// 3. <li>Cherry</li> = ??? → Insert new node

// Efficient! Only one DOM insertion

// ════════════════════════════════════════════════════════════════

// Problem case - insertion at beginning

// Before
<ul>
  <li>Apple</li>
  <li>Banana</li>
</ul>

// After
<ul>
  <li>Cherry</li>  {/* Added at beginning */}
  <li>Apple</li>
  <li>Banana</li>
</ul>

// Without keys, React compares by position:
// 1. <li>Apple</li> vs <li>Cherry</li> → Update text
// 2. <li>Banana</li> vs <li>Apple</li> → Update text
// 3. ??? vs <li>Banana</li> → Insert new node

// TERRIBLE! Updated 2 nodes + inserted 1 = 3 mutations
// Should have been: Insert 1 node

// Solution: KEYS
```

---

## 3. Keys

```tsx
// ════════════════════════════════════════════════════════════════
// WHY KEYS MATTER
// ════════════════════════════════════════════════════════════════

// With keys - React can identify items

// Before
<ul>
  <li key="apple">Apple</li>
  <li key="banana">Banana</li>
</ul>

// After
<ul>
  <li key="cherry">Cherry</li>  {/* New item */}
  <li key="apple">Apple</li>    {/* Same key */}
  <li key="banana">Banana</li>  {/* Same key */}
</ul>

// React's reconciliation with keys:
// 1. key="cherry" - new, insert at beginning
// 2. key="apple" - exists, keep as is, move position
// 3. key="banana" - exists, keep as is, move position

// Only 1 DOM insertion! Much better.

// ════════════════════════════════════════════════════════════════
// KEY REQUIREMENTS
// ════════════════════════════════════════════════════════════════

// 1. Keys must be UNIQUE among siblings
// ❌ BAD - duplicate keys
{items.map(item => (
  <li key="same">{item.name}</li>  // All same key!
))}

// ✅ GOOD - unique keys
{items.map(item => (
  <li key={item.id}>{item.name}</li>
))}

// 2. Keys must be STABLE (same item = same key)
// ❌ BAD - random keys
{items.map(item => (
  <li key={Math.random()}>{item.name}</li>  // Different every render!
))}

// ❌ BAD - index as key for reorderable lists
{items.map((item, index) => (
  <li key={index}>{item.name}</li>
))}
// Problem: If items reorder, index stays same but item is different
// React thinks same item, won't re-render properly

// ✅ GOOD - stable unique identifier
{items.map(item => (
  <li key={item.id}>{item.name}</li>  // Stable ID from data
))}

// 3. When index as key IS okay:
// - List is static (never reordered)
// - Items have no state
// - Items are never inserted/deleted in middle

// ════════════════════════════════════════════════════════════════
// KEY PITFALL: STATE LOSS
// ════════════════════════════════════════════════════════════════

function TodoList({ todos }) {
  return (
    <ul>
      {todos.map((todo, index) => (
        // ❌ Using index as key
        <TodoItem key={index} todo={todo} />
      ))}
    </ul>
  );
}

function TodoItem({ todo }) {
  // This component has internal state
  const [isEditing, setIsEditing] = useState(false);
  
  return (
    <li>
      {isEditing ? <input /> : <span>{todo.text}</span>}
    </li>
  );
}

// Scenario:
// 1. User is editing item at index 1
// 2. Item at index 0 gets deleted
// 3. Items shift: old index 1 is now index 0
// 4. React sees key=0 still exists, keeps state
// 5. BUG: Wrong item now shows editing state!

// ✅ FIX: Use stable keys
{todos.map(todo => (
  <TodoItem key={todo.id} todo={todo} />
))}

// ════════════════════════════════════════════════════════════════
// KEY TRICK: FORCE REMOUNT
// ════════════════════════════════════════════════════════════════

// Sometimes you WANT to destroy and recreate a component

function UserProfile({ userId }) {
  // Change key to force remount when userId changes
  return <Profile key={userId} userId={userId} />;
}

// Without key: Profile updates with new userId
// With key: Profile unmounts, new Profile mounts
// Useful to reset all internal state
```

---

## 4. React Fiber

```tsx
// ════════════════════════════════════════════════════════════════
// WHAT IS FIBER?
// ════════════════════════════════════════════════════════════════

// Fiber is React's reconciliation engine (since React 16)
// Key innovation: Work can be interrupted and resumed

// Before Fiber (Stack Reconciler):
// - Recursive, synchronous
// - Once started, must complete
// - Long renders block main thread (janky UI)

// After Fiber:
// - Iterative, can pause
// - Work broken into units
// - Can yield to browser for high-priority work

// ════════════════════════════════════════════════════════════════
// FIBER NODE STRUCTURE
// ════════════════════════════════════════════════════════════════

// Each React element has a corresponding Fiber node
interface FiberNode {
  // Identity
  type: any;           // Component function or 'div', 'span', etc.
  key: string | null;
  
  // Tree structure (like linked list)
  child: FiberNode | null;      // First child
  sibling: FiberNode | null;    // Next sibling
  return: FiberNode | null;     // Parent
  
  // State
  pendingProps: any;
  memoizedProps: any;
  memoizedState: any;
  
  // Effects
  flags: number;       // What work to do (update, delete, etc.)
  subtreeFlags: number;
  
  // Priority
  lanes: number;       // Priority of this work
}

// ════════════════════════════════════════════════════════════════
// WORK LOOP
// ════════════════════════════════════════════════════════════════

// Simplified work loop
function workLoop(deadline: IdleDeadline) {
  let shouldYield = false;
  
  while (nextUnitOfWork && !shouldYield) {
    // Process one fiber
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    
    // Check if we should yield to browser
    shouldYield = deadline.timeRemaining() < 1;
  }
  
  // If more work, schedule next chunk
  if (nextUnitOfWork) {
    requestIdleCallback(workLoop);
  } else {
    // All work done, commit to DOM
    commitRoot();
  }
}

// ════════════════════════════════════════════════════════════════
// CONCURRENT FEATURES (React 18+)
// ════════════════════════════════════════════════════════════════

// Interruptible rendering enables:

// 1. Transitions - Low priority updates
function SearchResults() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const [isPending, startTransition] = useTransition();
  
  function handleChange(e) {
    // High priority - update input immediately
    setQuery(e.target.value);
    
    // Low priority - can be interrupted
    startTransition(() => {
      setResults(searchDatabase(e.target.value));
    });
  }
  
  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending ? <Spinner /> : <ResultList results={results} />}
    </>
  );
}

// 2. Suspense for data fetching
function Profile({ id }) {
  return (
    <Suspense fallback={<Skeleton />}>
      <ProfileDetails id={id} />
    </Suspense>
  );
}

// 3. useDeferredValue - Defer non-urgent updates
function SearchResults({ query }) {
  const deferredQuery = useDeferredValue(query);
  
  // UI stays responsive while search processes
  const results = useMemo(
    () => heavySearch(deferredQuery),
    [deferredQuery]
  );
  
  return <ResultList results={results} />;
}
```

---

## 5. Rendering Optimization

```tsx
// ════════════════════════════════════════════════════════════════
// WHEN DOES A COMPONENT RE-RENDER?
// ════════════════════════════════════════════════════════════════

// 1. Its state changes
// 2. Its props change
// 3. Its parent re-renders (even if props are same!)
// 4. Context it consumes changes

// ════════════════════════════════════════════════════════════════
// MEMO - PREVENT UNNECESSARY RE-RENDERS
// ════════════════════════════════════════════════════════════════

// Without memo - re-renders when parent re-renders
function ExpensiveList({ items }) {
  console.log('ExpensiveList render');
  return items.map(item => <Item key={item.id} item={item} />);
}

// With memo - only re-renders when props change
const ExpensiveList = memo(function ExpensiveList({ items }) {
  console.log('ExpensiveList render');
  return items.map(item => <Item key={item.id} item={item} />);
});

// Custom comparison function
const ExpensiveList = memo(
  function ExpensiveList({ items, filter }) {
    // ...
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    return prevProps.items.length === nextProps.items.length &&
           prevProps.filter === nextProps.filter;
  }
);

// ════════════════════════════════════════════════════════════════
// USEMEMO - MEMOIZE EXPENSIVE CALCULATIONS
// ════════════════════════════════════════════════════════════════

function SearchResults({ items, query }) {
  // ❌ Filters on every render
  const filtered = items.filter(item => 
    item.name.includes(query)
  );
  
  // ✅ Only recalculates when items or query change
  const filtered = useMemo(
    () => items.filter(item => item.name.includes(query)),
    [items, query]
  );
  
  return <List items={filtered} />;
}

// ════════════════════════════════════════════════════════════════
// USECALLBACK - MEMOIZE FUNCTIONS
// ════════════════════════════════════════════════════════════════

function Parent() {
  const [count, setCount] = useState(0);
  
  // ❌ New function every render - Child re-renders
  const handleClick = () => {
    console.log('clicked');
  };
  
  // ✅ Same function reference - Child can skip re-render
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);
  
  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Parent count: {count}
      </button>
      <Child onClick={handleClick} />
    </>
  );
}

const Child = memo(function Child({ onClick }) {
  console.log('Child render');
  return <button onClick={onClick}>Child</button>;
});

// ════════════════════════════════════════════════════════════════
// COMPOSITION TO AVOID RE-RENDERS
// ════════════════════════════════════════════════════════════════

// ❌ All children re-render when value changes
function App() {
  const [value, setValue] = useState('');
  
  return (
    <div>
      <input value={value} onChange={e => setValue(e.target.value)} />
      <ExpensiveTree /> {/* Re-renders on every keystroke! */}
    </div>
  );
}

// ✅ Lift state to separate component
function App() {
  return (
    <div>
      <SearchInput />
      <ExpensiveTree /> {/* Never re-renders from input changes */}
    </div>
  );
}

function SearchInput() {
  const [value, setValue] = useState('');
  return <input value={value} onChange={e => setValue(e.target.value)} />;
}

// ✅ Or pass children as props (children are created in parent scope)
function InputWrapper({ children }) {
  const [value, setValue] = useState('');
  
  return (
    <div>
      <input value={value} onChange={e => setValue(e.target.value)} />
      {children} {/* Not re-created, doesn't re-render */}
    </div>
  );
}

function App() {
  return (
    <InputWrapper>
      <ExpensiveTree />
    </InputWrapper>
  );
}
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# VIRTUAL DOM PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Using index as key for dynamic lists
# Bad
{items.map((item, index) => (
  <Item key={index} item={item} />
))}
# Items reorder = bugs with state

# Good
{items.map(item => (
  <Item key={item.id} item={item} />
))}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Creating objects/arrays in render
# Bad
<Child style={{ color: 'red' }} />  # New object every render
<Child items={[1, 2, 3]} />          # New array every render
<Child onClick={() => {}} />          # New function every render
# Child re-renders even with memo

# Good
const style = useMemo(() => ({ color: 'red' }), []);
const items = useMemo(() => [1, 2, 3], []);
const onClick = useCallback(() => {}, []);
<Child style={style} items={items} onClick={onClick} />

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Unnecessary memo everywhere
# Bad
const Button = memo(({ onClick, children }) => (
  <button onClick={onClick}>{children}</button>
));
# memo has cost, small components don't benefit

# Good
# Only memo expensive components
# Or components that receive same props often

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: State too high in tree
# Bad
# App holds all state, everything re-renders

# Good
# Colocate state with components that need it
# Split components to isolate state

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Expecting component to unmount
# Bad
<div>
  {isLoggedIn ? <Dashboard /> : <Login />}
</div>
# Same position = same instance (state preserved)

# Good - use key to force unmount
<div>
  {isLoggedIn ? (
    <Dashboard key="dashboard" />
  ) : (
    <Login key="login" />
  )}
</div>
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is the Virtual DOM?"**
> "The Virtual DOM is a lightweight JavaScript object representation of the actual DOM. React creates a new virtual tree on each render, diffs it against the previous one (reconciliation), and applies only the necessary changes to the real DOM. This is more efficient than manipulating the DOM directly."

**Q: "Why do we need keys in lists?"**
> "Keys help React identify which items changed, were added, or removed. Without keys, React compares elements by position, which is inefficient if items are inserted or reordered. Keys must be unique among siblings and stable (same item = same key). Never use array index for dynamic lists."

**Q: "What happens when a component's state changes?"**
> "React marks the component for re-render. In the render phase, it calls the component function to create a new virtual DOM tree. It diffs against the previous tree using reconciliation. In the commit phase, it applies the minimal DOM changes. Then effects run."

### Intermediate Questions

**Q: "Explain React's reconciliation algorithm."**
> "React uses heuristics to achieve O(n) diffing: 1) Elements of different types produce different trees - full rebuild. 2) Same type elements update attributes and recurse on children. 3) Keys identify stable elements in lists. These assumptions make efficient diffing possible."

**Q: "What is React Fiber?"**
> "Fiber is React's reconciliation engine since React 16. Key feature: work can be interrupted and resumed. Work is broken into fiber units (linked list structure). This enables concurrent features: transitions, Suspense, time slicing. The render phase is interruptible; commit phase is synchronous."

**Q: "How do memo, useMemo, and useCallback differ?"**
> "memo wraps a component to skip re-render if props haven't changed. useMemo memoizes a computed value. useCallback memoizes a function reference. All prevent unnecessary work but have overhead - use when the cost of re-computing/re-rendering exceeds memoization cost."

### Advanced Questions

**Q: "When would a component re-render unnecessarily?"**
> "When parent re-renders (even with same props), when prop objects/arrays are recreated each render, when context changes (all consumers re-render). Solutions: memo for expensive components, useMemo/useCallback for stable references, composition to isolate state, context selectors."

**Q: "Explain the render and commit phases."**
> "Render phase: call function components, create virtual DOM, diff trees, determine changes. It's pure (no side effects) and can be interrupted in concurrent mode. Commit phase: apply DOM changes, run useLayoutEffect synchronously, then run useEffect after paint. Commit cannot be interrupted."

**Q: "How does concurrent rendering work?"**
> "React Fiber breaks work into units that can pause. When higher priority work arrives (user interaction), current work can be interrupted. Low priority updates (transitions) can be deferred. useDeferredValue and startTransition mark work as interruptible. This keeps the UI responsive during expensive renders."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              VIRTUAL DOM CHECKLIST                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  KEYS:                                                          │
│  □ Use stable unique IDs, not index                            │
│  □ Same item = same key across renders                         │
│  □ Use key to force remount when needed                        │
│                                                                 │
│  OPTIMIZATION:                                                  │
│  □ memo for expensive components                               │
│  □ useMemo for expensive calculations                          │
│  □ useCallback for stable function references                  │
│  □ Colocate state to minimize re-renders                       │
│                                                                 │
│  AVOID:                                                         │
│  □ Creating objects/arrays in render                           │
│  □ Index as key for dynamic lists                              │
│  □ Unnecessary memoization overhead                            │
│                                                                 │
│  CONCURRENT (React 18+):                                        │
│  □ useTransition for low-priority updates                      │
│  □ useDeferredValue for expensive derived state                │
│  □ Suspense for data fetching                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

RENDERING PHASES:
┌────────────────────────────────────────────────────────────────┐
│ Trigger:  State change, props change, parent render            │
│ Render:   Create VDOM, diff, determine changes (interruptible) │
│ Commit:   Apply DOM changes, run effects (synchronous)         │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

