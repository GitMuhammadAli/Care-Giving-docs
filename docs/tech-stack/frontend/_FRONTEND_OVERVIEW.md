# Frontend Architecture Overview

> Understanding how all the frontend pieces fit together.

---

## The Mental Model

Think of the frontend like a **newspaper publishing system**:

- **React** = The printing press (renders content)
- **Next.js** = The editor-in-chief (decides what goes where, when)
- **Zustand** = The notepad on your desk (things you're actively working on)
- **TanStack Query** = The archive room (data from the server, cached)
- **Tailwind** = The style guide (consistent visual language)
- **Framer Motion** = The illustrator (makes things move beautifully)

---

## Data Flow Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           USER INTERACTION                                   │
│                                  │                                           │
│                    ┌─────────────▼─────────────┐                            │
│                    │        COMPONENT          │                            │
│                    │    (React + Hooks)        │                            │
│                    └─────────────┬─────────────┘                            │
│                                  │                                           │
│           ┌──────────────────────┼──────────────────────┐                   │
│           │                      │                      │                   │
│           ▼                      ▼                      ▼                   │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐          │
│  │     ZUSTAND     │   │  TANSTACK QUERY │   │   LOCAL STATE   │          │
│  │                 │   │                 │   │                 │          │
│  │  Global client  │   │  Server state   │   │  Component-only │          │
│  │     state       │   │    (cached)     │   │     state       │          │
│  │                 │   │                 │   │                 │          │
│  │  • User auth    │   │  • API data     │   │  • Form inputs  │          │
│  │  • UI settings  │   │  • Medications  │   │  • Modal open   │          │
│  │  • Family ctx   │   │  • Appointments │   │  • Tab selected │          │
│  └────────┬────────┘   └────────┬────────┘   └─────────────────┘          │
│           │                     │                                           │
│           │                     ▼                                           │
│           │            ┌─────────────────┐                                  │
│           │            │      API        │                                  │
│           │            │   (fetch/axios) │                                  │
│           │            └────────┬────────┘                                  │
│           │                     │                                           │
└───────────┼─────────────────────┼───────────────────────────────────────────┘
            │                     │
            │                     ▼
            │            ┌─────────────────┐
            │            │  Backend API    │
            │            │   (NestJS)      │
            │            └─────────────────┘
            │
            ▼
    ┌─────────────────┐
    │   WebSocket     │
    │   (Socket.io)   │
    │                 │
    │  Real-time      │
    │  updates        │
    └─────────────────┘
```

---

## State Management Decision Tree

```
                    ┌─────────────────────────────────────┐
                    │     Where should this data live?    │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │   Does it come from the server?     │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────YES─────┴─────NO────────────┐
                    │                                      │
                    ▼                                      ▼
        ┌───────────────────┐              ┌───────────────────────┐
        │  TanStack Query   │              │  Does it need to be   │
        │                   │              │  shared across pages? │
        │  It handles:      │              └───────────┬───────────┘
        │  • Caching        │                          │
        │  • Refetching     │          ┌────YES────────┴────NO────────┐
        │  • Loading states │          │                              │
        │  • Error states   │          ▼                              ▼
        └───────────────────┘  ┌───────────────┐         ┌─────────────────┐
                               │    Zustand    │         │   useState()    │
                               │               │         │                 │
                               │  Examples:    │         │  Examples:      │
                               │  • User auth  │         │  • Form inputs  │
                               │  • Theme      │         │  • Modal state  │
                               │  • Family ctx │         │  • Temp UI      │
                               └───────────────┘         └─────────────────┘
```

---

## Component Architecture

### The Component Hierarchy Philosophy

```
Pages (Route Components)
  │
  ├── Responsible for:
  │   • Data fetching (useQuery)
  │   • Page-level layout
  │   • Error boundaries
  │   • Loading states
  │
  └── Contains...
      │
      Feature Components
        │
        ├── Responsible for:
        │   • Business logic
        │   • Local state management
        │   • Feature-specific behavior
        │
        └── Contains...
            │
            UI Components
              │
              ├── Responsible for:
              │   • Visual presentation
              │   • Styling
              │   • Accessibility
              │   • NO business logic
              │
              └── Examples: Button, Card, Input, Modal
```

### When to Create a New Component?

Ask yourself:

1. **Is it reusable?** Will this exact UI appear elsewhere?
2. **Is it complex?** Does it have its own state or logic?
3. **Is it testable?** Would isolating it make testing easier?
4. **Is it readable?** Is the parent component getting too long?

If **any** answer is yes, extract to a new component.

---

## Server vs Client Components (Next.js 14)

### The Mental Model

Think of it like a **restaurant**:

- **Server Components** = Kitchen staff (prep work, heavy lifting)
- **Client Components** = Waiters (interact with customers, respond to actions)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    COMPONENT BOUNDARY RULE                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   SERVER COMPONENTS (default)        │   CLIENT COMPONENTS           │
│   ─────────────────────────          │   ─────────────────           │
│                                      │                               │
│   ✅ Fetch data                      │   ✅ useState, useEffect      │
│   ✅ Access backend resources        │   ✅ Event handlers (onClick) │
│   ✅ Keep sensitive data server-side │   ✅ Browser APIs             │
│   ✅ Large dependencies              │   ✅ Custom hooks with state  │
│   ✅ SEO-critical content            │   ✅ Interactive elements     │
│                                      │                               │
│   ❌ Cannot use hooks                │   ❌ Heavier JS bundle        │
│   ❌ Cannot use browser APIs         │   ❌ No direct DB access      │
│   ❌ Cannot handle events            │                               │
│                                      │                               │
└─────────────────────────────────────────────────────────────────────┘
```

### The Boundary Pattern

```tsx
// page.tsx (Server Component)
async function MedicationsPage() {
  const medications = await fetchMedications(); // Server-side fetch
  
  return (
    <div>
      <h1>Medications</h1>  {/* Static, server-rendered */}
      <MedicationList data={medications} />  {/* Pass data down */}
      <AddMedicationButton />  {/* Client component for interactivity */}
    </div>
  );
}

// AddMedicationButton.tsx (Client Component)
'use client';  // This marks the boundary

function AddMedicationButton() {
  const [isOpen, setIsOpen] = useState(false);  // Now we can use hooks
  return <button onClick={() => setIsOpen(true)}>Add</button>;
}
```

---

## Styling Philosophy

### Why Tailwind CSS?

| Traditional CSS Problem | Tailwind Solution |
|-------------------------|-------------------|
| "What should I name this class?" | No naming needed, use utilities |
| "Is this class used elsewhere?" | Styles are co-located with markup |
| "CSS file growing forever" | Only used styles in final bundle |
| "Inconsistent spacing/colors" | Constrained design tokens |

### The Tailwind Mental Model

Think of Tailwind classes as **building blocks**:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TAILWIND CLASS CATEGORIES                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  LAYOUT          SPACING         TYPOGRAPHY      COLORS             │
│  ──────          ───────         ──────────      ──────             │
│  flex            p-4             text-lg         bg-white           │
│  grid            m-2             font-bold       text-gray-900      │
│  block           gap-4           leading-6       border-blue-500    │
│  hidden          space-y-2       tracking-wide   hover:bg-gray-100  │
│                                                                      │
│  SIZING          BORDERS         EFFECTS         RESPONSIVE         │
│  ──────          ───────         ───────         ──────────         │
│  w-full          rounded-lg      shadow-md       md:flex            │
│  h-screen        border-2        opacity-50      lg:grid-cols-3     │
│  max-w-md        divide-y        blur-sm         xl:text-xl         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### When to Extract to a Component vs Custom Class?

```
IF you're repeating the same combination 3+ times
  AND it represents a meaningful UI concept
  THEN extract to a component (preferred)
  
IF you're repeating 3+ times
  AND it's a utility pattern, not a concept
  THEN create @apply in globals.css

ELSE just use inline Tailwind classes
```

---

## Form Handling Strategy

### The Stack

```
React Hook Form  →  Zod Schema  →  API Submission
    (state)         (validation)     (mutation)
```

### Why This Combination?

| Library | Responsibility | Why Not Alternatives? |
|---------|----------------|----------------------|
| **React Hook Form** | Form state, submission | Formik is heavier, more re-renders |
| **Zod** | Validation schemas | Type inference, composable |
| **TanStack Query (mutation)** | API submission | Automatic error/loading states |

### The Pattern

```
1. Define Zod schema (single source of truth for validation)
2. Infer TypeScript type from schema
3. Pass schema to React Hook Form via zodResolver
4. Use useMutation for submission
5. Handle success/error states
```

---

## Real-Time Updates Strategy

### When to Use What?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REAL-TIME DECISION MATRIX                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Polling (setInterval + refetch)                                    │
│  ─────────────────────────────                                      │
│  ✅ Simple to implement                                             │
│  ✅ Works everywhere                                                │
│  ❌ Wastes bandwidth                                                │
│  ❌ Delayed updates                                                 │
│  → Use for: Dashboard metrics, non-critical data                    │
│                                                                      │
│  WebSocket (Socket.io)                                              │
│  ───────────────────                                                │
│  ✅ Instant updates                                                 │
│  ✅ Bidirectional                                                   │
│  ❌ More complex                                                    │
│  ❌ Connection management                                           │
│  → Use for: Chat, emergency alerts, collaboration                   │
│                                                                      │
│  Server-Sent Events (SSE)                                           │
│  ──────────────────────                                             │
│  ✅ Simple server push                                              │
│  ✅ Auto-reconnect                                                  │
│  ❌ One-way only                                                    │
│  → Use for: Notifications, activity feeds                           │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Our Approach

CareCircle uses **Socket.io** for:
- Emergency alerts (critical, need instant delivery)
- Medication logs (family coordination)
- Shift handoffs (caregiver communication)

And **TanStack Query refetching** for:
- Dashboard data (acceptable 30s staleness)
- Lists that don't need instant updates

---

## Error Handling Philosophy

### The Error Boundary Strategy

```
App Layout
  │
  └── Error Boundary (catches unexpected errors)
        │
        └── Page
              │
              ├── Query Error (handled by TanStack Query)
              │     → Show error UI, offer retry
              │
              ├── Form Error (handled by React Hook Form)
              │     → Show field-level errors
              │
              └── Network Error (caught by fetch wrapper)
                    → Show toast notification
```

### What Goes Where?

| Error Type | Where to Handle | User Experience |
|------------|-----------------|-----------------|
| 404 Not Found | Next.js `not-found.tsx` | Full page "not found" |
| 500 Server Error | Error boundary | "Something went wrong" + retry |
| Validation Error | Form component | Inline field errors |
| Auth Error (401) | API interceptor | Redirect to login |
| Network Error | Toast notification | Non-blocking alert |

---

## Performance Mental Model

### What Actually Matters

```
┌─────────────────────────────────────────────────────────────────────┐
│                    FRONTEND PERFORMANCE PRIORITIES                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. PERCEIVED PERFORMANCE (most important)                          │
│     → Users care about FEELING fast                                 │
│     → Loading skeletons > spinners                                  │
│     → Optimistic updates > wait for server                          │
│                                                                      │
│  2. BUNDLE SIZE                                                     │
│     → Lazy load routes and heavy components                         │
│     → Tree-shake unused code                                        │
│     → Analyze with next/bundle-analyzer                             │
│                                                                      │
│  3. RE-RENDERS                                                      │
│     → Often over-optimized prematurely                              │
│     → Profile before optimizing                                     │
│     → React.memo, useMemo, useCallback when MEASURED needed         │
│                                                                      │
│  4. NETWORK REQUESTS                                                │
│     → Batch where possible                                          │
│     → Cache aggressively (TanStack Query does this)                 │
│     → Prefetch predictable navigation                               │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Quick Reference

### File Naming Conventions

```
components/
  ui/
    button.tsx          # Base UI components (lowercase)
    input.tsx
  medications/
    MedicationCard.tsx  # Feature components (PascalCase)
    MedicationList.tsx
hooks/
  useAuth.ts           # Custom hooks (camelCase with 'use' prefix)
  useMedications.ts
stores/
  authStore.ts         # Zustand stores (camelCase with 'Store' suffix)
  familyStore.ts
lib/
  api.ts               # Utilities (lowercase)
  utils.ts
```

### Import Order Convention

```typescript
// 1. React/Next imports
import { useState, useEffect } from 'react';
import { useRouter } from 'next/navigation';

// 2. Third-party libraries
import { useQuery } from '@tanstack/react-query';
import { motion } from 'framer-motion';

// 3. Internal utilities/hooks
import { useAuth } from '@/hooks/useAuth';
import { cn } from '@/lib/utils';

// 4. Components
import { Button } from '@/components/ui/button';
import { MedicationCard } from '@/components/medications/MedicationCard';

// 5. Types
import type { Medication } from '@/types';
```

---

*Next: Deep dive into [React Concepts](react.md)*


