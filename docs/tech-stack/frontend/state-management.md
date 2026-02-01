# State Management Concepts

> Understanding when and how to manage state in React applications.

---

## 1. What Is State? (Conceptual Definition)

### Plain English Explanation

**State is data that changes over time and affects what the user sees.**

Think of it like a **whiteboard** in a meeting room:
- The whiteboard displays current information
- When someone writes something new, everyone sees the change
- The whiteboard's content determines what people discuss

In React:
- State is the "whiteboard"
- Changing state is "writing on the whiteboard"
- Components "looking at the whiteboard" re-render when it changes

### The Core Problem State Solves

React components are just functions. Every time they run, local variables reset:

```
Without State:
  function Counter() {
    let count = 0;        // Resets to 0 every render!
    return <span>{count}</span>;
  }

With State:
  function Counter() {
    const [count, setCount] = useState(0);  // React remembers this
    return <span>{count}</span>;
  }
```

State is React's way of giving functions a **memory**.

---

## 2. Core Concepts & Terminology

### Types of State

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         STATE TAXONOMY                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  LOCAL STATE                          │  GLOBAL STATE                    │
│  ───────────                          │  ────────────                    │
│  Belongs to ONE component             │  Shared across MANY components   │
│                                       │                                  │
│  Examples:                            │  Examples:                       │
│  • Form input values                  │  • Current user                  │
│  • Modal open/closed                  │  • Theme preference              │
│  • Dropdown expanded                  │  • Shopping cart                 │
│  • Tab selection                      │  • Selected family               │
│                                       │                                  │
│  Tool: useState()                     │  Tool: Zustand, Context          │
│                                       │                                  │
├───────────────────────────────────────┼──────────────────────────────────┤
│                                       │                                  │
│  SERVER STATE                         │  URL STATE                       │
│  ────────────                         │  ─────────                       │
│  Data that lives on the server        │  State encoded in the URL        │
│                                       │                                  │
│  Examples:                            │  Examples:                       │
│  • User list                          │  • Current page                  │
│  • Medication records                 │  • Search filters                │
│  • Appointments                       │  • Sort order                    │
│                                       │  • Selected item ID              │
│  Tool: TanStack Query                 │                                  │
│                                       │  Tool: URL params, searchParams  │
│  Characteristics:                     │                                  │
│  • Cached                             │  Benefits:                       │
│  • Can become stale                   │  • Shareable links               │
│  • May be shared by users             │  • Browser history works         │
│  • Needs sync strategy                │  • Bookmarkable                  │
│                                       │                                  │
└─────────────────────────────────────────────────────────────────────────┘
```

### Key Terminology

| Term | Definition | Example |
|------|------------|---------|
| **Render** | React running your component function | Opening a page |
| **Re-render** | Running the function again due to change | Clicking a button |
| **Derived State** | Values calculated from other state | `fullName = first + last` |
| **Lifting State** | Moving state to a common parent | Two siblings need same data |
| **State Colocation** | Keeping state close to where it's used | Form state in form component |
| **Stale State** | Cached data that's out of date | Old API response |

---

## 3. How State Works Under the Hood

### The Re-render Cycle

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      REACT'S STATE UPDATE CYCLE                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. USER ACTION                                                          │
│     │                                                                    │
│     │  onClick={() => setCount(count + 1)}                              │
│     │                                                                    │
│     ▼                                                                    │
│  2. STATE UPDATE SCHEDULED                                               │
│     │                                                                    │
│     │  React queues the update (batching)                               │
│     │  Multiple updates in same event → one re-render                   │
│     │                                                                    │
│     ▼                                                                    │
│  3. COMPONENT RE-RENDERS                                                 │
│     │                                                                    │
│     │  React calls your function again                                  │
│     │  useState returns the NEW value                                   │
│     │                                                                    │
│     ▼                                                                    │
│  4. VIRTUAL DOM DIFFING                                                  │
│     │                                                                    │
│     │  React compares old vs new output                                 │
│     │  Calculates minimal changes needed                                │
│     │                                                                    │
│     ▼                                                                    │
│  5. DOM UPDATE                                                           │
│     │                                                                    │
│     │  Only changed parts update in real DOM                            │
│     │  User sees the new UI                                             │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Why State Updates Feel "Asynchronous"

State updates are **batched** for performance:

```
// This schedules TWO updates, but only ONE re-render
function handleClick() {
  setCount(count + 1);  // Scheduled
  setName('Alice');     // Scheduled
  // Re-render happens AFTER this function completes
  console.log(count);   // Still shows OLD value!
}
```

**Mental Model**: Think of `setState` as "requesting a change" not "making a change."

---

## 4. Why We Use These Tools in CareCircle

### Our State Management Stack

| Tool | What It Manages | Why This Choice |
|------|-----------------|-----------------|
| **useState** | Component-local state | Built-in, simple |
| **Zustand** | Global client state | Minimal boilerplate, TypeScript-first |
| **TanStack Query** | Server state | Caching, background updates, deduplication |
| **URL params** | Navigational state | Shareable, back button works |

### Why NOT Other Options?

| Alternative | Why We Didn't Choose It |
|-------------|-------------------------|
| **Redux** | Too much boilerplate for our scale |
| **Context** (for global state) | Re-renders entire tree on any change |
| **MobX** | Different paradigm, team less familiar |
| **SWR** | TanStack Query has more features |
| **Jotai/Recoil** | Zustand simpler for our use cases |

---

## 5. When to Use Each Type ✅

### useState - Local Component State

**Perfect for:**
- Form input values
- Toggle states (open/closed, expanded/collapsed)
- UI-only state (selected tab, hover state)
- Temporary data during user interaction

**The Test:** "Does any OTHER component need to know about this?"
- If NO → useState
- If YES → Consider lifting or global state

### Zustand - Global Client State

**Perfect for:**
- User authentication state
- Theme/preferences
- Shopping cart
- Currently selected family/context
- Any state that:
  - Survives page navigation
  - Is needed by unrelated components
  - Doesn't come from the server

### TanStack Query - Server State

**Perfect for:**
- Any data from an API
- Data that multiple components might need
- Data that can become stale
- Data that needs refresh strategies

**The Rule:** If it comes from `fetch()` or an API, use Query.

### URL State

**Perfect for:**
- Current page/route
- Search filters
- Sort order
- Selected item (when deep-linkable)
- Pagination page number

**The Test:** "Should this be shareable via URL?"
- If YES → URL state
- If NO → Other state type

---

## 6. When to AVOID Each Type ❌

### useState Anti-patterns

```
❌ WRONG: Global state in useState

// Bad: This state is needed everywhere
function App() {
  const [user, setUser] = useState(null);
  // Now you have to prop-drill user to every component
}

✅ RIGHT: Use Zustand for global state

const useAuthStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));
```

```
❌ WRONG: Server data in useState

// Bad: You're managing caching, loading, errors manually
const [medications, setMedications] = useState([]);
const [loading, setLoading] = useState(true);
const [error, setError] = useState(null);

useEffect(() => {
  fetchMedications()
    .then(setMedications)
    .catch(setError)
    .finally(() => setLoading(false));
}, []);

✅ RIGHT: Use TanStack Query

const { data: medications, isLoading, error } = useQuery({
  queryKey: ['medications'],
  queryFn: fetchMedications,
});
```

### Zustand Anti-patterns

```
❌ WRONG: Server data in Zustand

// Bad: Now you're fighting TanStack Query
const useStore = create((set) => ({
  medications: [],
  fetchMedications: async () => {
    const data = await api.getMedications();
    set({ medications: data });
  },
}));

✅ RIGHT: Zustand for CLIENT state, Query for SERVER state
```

```
❌ WRONG: Frequently changing data

// Bad: Causes re-renders across all consumers
const useStore = create((set) => ({
  mousePosition: { x: 0, y: 0 },  // Updates 60 times/second!
}));

✅ RIGHT: Use a ref for high-frequency updates
```

### TanStack Query Anti-patterns

```
❌ WRONG: Derived data as separate query

// Bad: Fetching when you could calculate
const { data: medications } = useQuery(['medications'], ...);
const { data: activeMeds } = useQuery(['activeMedications'], ...);

✅ RIGHT: Derive from existing data

const { data: medications } = useQuery(['medications'], ...);
const activeMeds = medications?.filter(m => m.isActive);
```

---

## 7. Best Practices & Recommendations

### 1. State Should Be Minimal

```
❌ WRONG: Redundant state

const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [fullName, setFullName] = useState('');  // Redundant!

✅ RIGHT: Derive what you can

const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const fullName = `${firstName} ${lastName}`;  // Derived!
```

### 2. Colocate State

Keep state as close as possible to where it's used:

```
❌ WRONG: State too high

// In App.tsx
const [searchQuery, setSearchQuery] = useState('');
// Prop-drill through 5 components to the search input

✅ RIGHT: State where it's needed

// In SearchComponent.tsx
const [searchQuery, setSearchQuery] = useState('');
// Used right here, no drilling
```

### 3. Use Selectors with Zustand

```typescript
❌ WRONG: Subscribe to entire store

const store = useAuthStore();  // Re-renders on ANY store change

✅ RIGHT: Subscribe to specific slices

const user = useAuthStore((state) => state.user);  // Only re-renders when user changes
const isAdmin = useAuthStore((state) => state.user?.role === 'ADMIN');
```

### 4. Structure Query Keys Hierarchically

```typescript
// Enables targeted invalidation
const queryKeys = {
  all: ['medications'] as const,
  lists: () => [...queryKeys.all, 'list'] as const,
  list: (filters: string) => [...queryKeys.lists(), filters] as const,
  details: () => [...queryKeys.all, 'detail'] as const,
  detail: (id: string) => [...queryKeys.details(), id] as const,
};

// Now you can invalidate precisely
queryClient.invalidateQueries({ queryKey: queryKeys.lists() });
```

---

## 8. Common Mistakes & How to Avoid Them

### Mistake 1: State Updates That Don't Trigger Re-renders

```javascript
❌ WRONG: Mutating state directly

const [items, setItems] = useState([1, 2, 3]);

function addItem() {
  items.push(4);  // Mutating! React doesn't see the change
  setItems(items);  // Same reference, no re-render
}

✅ RIGHT: Create new reference

function addItem() {
  setItems([...items, 4]);  // New array, React sees the change
}
```

### Mistake 2: Stale Closure

```javascript
❌ WRONG: Stale value in async code

function Counter() {
  const [count, setCount] = useState(0);
  
  function handleClick() {
    setTimeout(() => {
      setCount(count + 1);  // 'count' is stale (captured value)
    }, 1000);
  }
}

✅ RIGHT: Use functional update

function handleClick() {
  setTimeout(() => {
    setCount(prev => prev + 1);  // Always uses current value
  }, 1000);
}
```

### Mistake 3: Infinite useEffect Loops

```javascript
❌ WRONG: Object/array in dependencies

useEffect(() => {
  fetchData(filters);
}, [filters]);  // If 'filters' is { name: 'x' }, re-runs every render!

✅ RIGHT: Stable dependencies

// Option 1: Stringify
useEffect(() => {
  fetchData(filters);
}, [JSON.stringify(filters)]);

// Option 2: Primitive values
useEffect(() => {
  fetchData({ name: filterName });
}, [filterName]);

// Option 3: useMemo the object
const stableFilters = useMemo(() => ({ name }), [name]);
```

### Mistake 4: Overusing Global State

```
❌ Pattern: Put everything in Zustand "just in case"

✅ Principle: Default to LOCAL state, lift/globalize only when NEEDED
```

---

## 9. Performance Considerations

### What Actually Causes Performance Issues?

```
MYTH: "Re-renders are expensive"
TRUTH: Re-renders are usually fast. UNNECESSARY re-renders are the problem.

MYTH: "Always use useMemo and useCallback"
TRUTH: These have costs too. Measure before optimizing.
```

### When to Optimize State Performance

```
1. MEASURE FIRST
   Use React DevTools Profiler to identify actual slow components

2. CONSIDER THESE (in order):
   a. Are you rendering too much data? → Virtualize lists
   b. Is computation expensive? → useMemo the result
   c. Are callbacks causing child re-renders? → useCallback if children are memoized
   d. Is global state too coarse? → Use selectors

3. LAST RESORT:
   React.memo on components (but measure the improvement!)
```

### Zustand Performance Tips

```typescript
// ✅ Good: Atomic selectors
const userName = useStore(state => state.user?.name);

// ❌ Bad: Selecting objects
const user = useStore(state => state.user);  // Re-renders if ANY user property changes

// ✅ Good: Shallow comparison for objects
import { shallow } from 'zustand/shallow';
const { user, settings } = useStore(
  state => ({ user: state.user, settings: state.settings }),
  shallow
);
```

---

## 10. Decision Flowchart

```
                    ┌─────────────────────────────────────┐
                    │       I have some data...           │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │    Does it come from an API/server? │
                    └─────────────────┬───────────────────┘
                                      │
              ┌───────────YES─────────┴─────────NO────────────┐
              │                                                │
              ▼                                                ▼
    ┌─────────────────┐                      ┌─────────────────────────────┐
    │  TanStack Query │                      │  Should it persist across   │
    │                 │                      │  page navigation?           │
    │  useQuery()     │                      └──────────────┬──────────────┘
    │  useMutation()  │                                     │
    └─────────────────┘              ┌──────YES─────────────┴──────NO──────┐
                                     │                                     │
                                     ▼                                     ▼
                         ┌─────────────────────┐         ┌─────────────────────┐
                         │  Should it be in    │         │     useState()      │
                         │  the URL?           │         │                     │
                         └──────────┬──────────┘         │  Keep it local to   │
                                    │                    │  the component      │
                         ┌────YES───┴───NO────┐          └─────────────────────┘
                         │                    │
                         ▼                    ▼
              ┌─────────────────┐   ┌─────────────────┐
              │  URL Params     │   │     Zustand     │
              │                 │   │                 │
              │  searchParams   │   │  Global store   │
              │  pathname       │   │  for client     │
              └─────────────────┘   │  state          │
                                    └─────────────────┘
```

---

## 11. Quick Reference Cheatsheet

### State Type Quick Guide

| Data Type | Tool | Example |
|-----------|------|---------|
| Form input | `useState` | Search box text |
| Toggle UI | `useState` | Modal open/closed |
| API data | `useQuery` | Medication list |
| Mutation | `useMutation` | Create medication |
| Auth user | Zustand | Current logged-in user |
| Theme | Zustand | Dark/light mode |
| Filters | URL params | `?status=active` |
| Sort | URL params | `?sort=name` |

### Common Patterns

```typescript
// Pattern: Boolean toggle
const [isOpen, setIsOpen] = useState(false);
const toggle = () => setIsOpen(prev => !prev);

// Pattern: Form state
const [form, setForm] = useState({ name: '', email: '' });
const updateField = (field: string, value: string) =>
  setForm(prev => ({ ...prev, [field]: value }));

// Pattern: Array manipulation
const [items, setItems] = useState<Item[]>([]);
const addItem = (item: Item) => setItems(prev => [...prev, item]);
const removeItem = (id: string) => setItems(prev => prev.filter(i => i.id !== id));
const updateItem = (id: string, updates: Partial<Item>) =>
  setItems(prev => prev.map(i => i.id === id ? { ...i, ...updates } : i));
```

---

## 12. Learning Resources

### Official Docs
- [React useState](https://react.dev/reference/react/useState)
- [TanStack Query Docs](https://tanstack.com/query/latest)
- [Zustand GitHub](https://github.com/pmndrs/zustand)

### Recommended Reading
- "Thinking in React" (React docs)
- "Practical React Query" by TkDodo (blog series)
- "Working with Zustand" by the pmndrs team

### Video Resources
- Theo's "Stop Using useState for EVERYTHING" (YouTube)
- Jack Herrington's React state management series

---

*Next: [TanStack Query Deep Dive](tanstack-query.md) | [Zustand Patterns](zustand.md)*


