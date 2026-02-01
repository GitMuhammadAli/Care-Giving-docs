# React Concepts

> Understanding React's component model and rendering philosophy.

---

## 1. What Is React?

### Plain English Explanation

React is a **JavaScript library for building user interfaces** through composable components.

Think of it like **LEGO blocks**:
- Each component is a block
- Blocks can contain other blocks
- You describe what the final structure should look like
- React figures out how to build it efficiently

### The Core Problem React Solves

**DOM manipulation is painful and error-prone.**

Without React, updating the UI requires manually finding elements, changing them, and keeping everything in sync. React lets you describe *what* the UI should look like, and it handles *how* to update it.

### Mental Model: Declarative vs Imperative

```
IMPERATIVE (manual instructions):
─────────────────────────────────
"Walk to the fridge. Open the door. Look for milk. Pick it up.
Close the door. Walk back. Pour milk into glass."

DECLARATIVE (desired outcome):
──────────────────────────────
"I want a glass of milk."
(The 'how' is someone else's problem)

React is DECLARATIVE:
  You describe WHAT the UI should look like
  React figures out HOW to make it happen
```

---

## 2. Core Concepts & Terminology

### The Component Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         COMPONENT MENTAL MODEL                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  COMPONENT = FUNCTION that returns UI                                        │
│                                                                              │
│  Input (Props) ──► Component ──► Output (UI)                                │
│                        │                                                     │
│                        └── State (internal memory)                           │
│                                                                              │
│  FORMULA:                                                                    │
│  UI = f(state, props)                                                        │
│                                                                              │
│  Same inputs = Same output (predictable)                                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Terminology

| Term | Definition | Analogy |
|------|------------|---------|
| **Component** | Reusable UI building block | LEGO brick |
| **Props** | Data passed from parent to child | Function arguments |
| **State** | Data that changes over time | Memory |
| **Render** | Component function running | Taking a photo |
| **Re-render** | Component function running again | Taking another photo |
| **Virtual DOM** | React's internal UI representation | Blueprint |
| **Hook** | Function that lets components use React features | Power-up |

---

## 3. How Rendering Works Under the Hood

### The Render Cycle

```
1. TRIGGER
   • Initial render (first time)
   • State update (setState called)
   • Props change (parent re-rendered)

2. RENDER PHASE (Pure, no side effects)
   • React calls your component function
   • Component returns JSX
   • React builds new Virtual DOM tree

3. RECONCILIATION
   • React compares new tree with previous
   • Calculates minimal changes needed (diffing)

4. COMMIT PHASE (Side effects allowed)
   • React updates the real DOM
   • Runs useLayoutEffect
   • Browser paints
   • Runs useEffect
```

### Why Virtual DOM?

Direct DOM manipulation is slow because the browser must recalculate styles, reflow layout, and repaint. React's Virtual DOM batches changes into one minimal update, resulting in fewer, smarter DOM operations.

---

## 4. Why React for CareCircle

| Requirement | Why React Fits |
|-------------|----------------|
| Complex UI state | Component state + hooks handle complexity |
| Real-time updates | Efficient re-rendering for live data |
| Team familiarity | Largest community, easier hiring |
| Next.js integration | React is required for Next.js |
| Mobile future | React Native shares concepts |

---

## 5. When to Use React Patterns ✅

### Use State When:
- Data changes over time
- Data affects what the user sees
- User interactions modify the data

### Use Props When:
- Passing data from parent to child
- Configuring child component behavior
- Passing callback functions

### Use Effects When:
- Synchronizing with external systems (APIs, subscriptions)
- Fetching data on mount
- Manually interacting with the DOM

### Use Refs When:
- Accessing DOM elements directly
- Storing values that don't trigger re-renders

---

## 6. When to AVOID Patterns ❌

### DON'T Put Derived Data in State

```
❌ BAD: total is derived from items
const [items, setItems] = useState([]);
const [total, setTotal] = useState(0);  // Redundant!

✅ GOOD: Calculate derived values
const [items, setItems] = useState([]);
const total = items.reduce((sum, item) => sum + item.price, 0);
```

### DON'T Mutate State

```
❌ BAD: user.name = 'Jane'; setUser(user);
   (Same reference, React doesn't see change)

✅ GOOD: setUser({ ...user, name: 'Jane' });
   (New object, React re-renders)
```

### DON'T Overuse useEffect

If you can calculate something during render, do it during render—not in an effect.

---

## 7. Best Practices & Recommendations

### Component Composition

Prefer composition over prop drilling. Pass children or use context for truly global data.

### Keep Components Focused

One component, one job. If a component does multiple unrelated things, split it.

### Naming Conventions

```
COMPONENTS: PascalCase (MedicationCard.tsx)
HOOKS: camelCase with 'use' prefix (useMedications.ts)
EVENT HANDLERS: handle + Event (handleClick, handleSubmit)
BOOLEAN PROPS: is/has/can prefix (isLoading, hasError)
```

---

## 8. Common Mistakes & How to Avoid Them

### Mistake 1: Infinite Loops

Setting state inside useEffect without proper dependencies causes infinite loops.

### Mistake 2: Missing Keys

Always use stable, unique keys when rendering lists (item.id, not array index).

### Mistake 3: Stale Closures

Use functional updates (`setCount(c => c + 1)`) when the new value depends on the previous.

---

## 9. Performance Considerations

### When to Optimize

```
DON'T OPTIMIZE PREMATURELY!

1. Build it correctly first
2. Measure if there's a problem
3. Only then optimize

React is fast by default.
```

### React.memo, useMemo, useCallback

Only useful when:
- Component re-renders frequently with same props
- Calculation is truly expensive
- Value is passed to memoized children

---

## 10. Quick Reference

### Hook Summary

| Hook | Purpose |
|------|---------|
| `useState` | Store component state |
| `useEffect` | Side effects (fetch, subscribe) |
| `useRef` | DOM access, mutable values |
| `useMemo` | Memoize expensive calculations |
| `useCallback` | Memoize functions |
| `useContext` | Access context values |

---

## 11. Learning Resources

- [React.dev](https://react.dev) - Official docs
- [React.dev/learn](https://react.dev/learn) - Interactive tutorials
- Dan Abramov's blog (overreacted.io)

---

*Next: [State Management](state-management.md) | [Next.js](nextjs.md)*
