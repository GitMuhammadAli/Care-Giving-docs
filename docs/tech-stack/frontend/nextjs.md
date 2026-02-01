# Next.js Concepts

> Understanding the full-stack React framework.

---

## 1. What Is Next.js?

### Plain English Explanation

Next.js is a **React framework** that adds structure, routing, and server-side capabilities to React applications.

Think of it like **React with superpowers**:
- React = Engine
- Next.js = Complete car (with steering, GPS, automatic transmission)

### The Core Problem Next.js Solves

React alone doesn't provide:
- Routing (navigating between pages)
- Server-side rendering (SEO, performance)
- API routes (backend in the same project)
- File-based structure

Next.js adds all of these with sensible defaults.

---

## 2. Core Concepts & Terminology

### The App Router Model (Next.js 14+)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         APP ROUTER STRUCTURE                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  app/                                                                        │
│  ├── page.tsx              → Route: /                                       │
│  ├── layout.tsx            → Wraps all pages                                │
│  ├── loading.tsx           → Loading UI (Suspense)                          │
│  ├── error.tsx             → Error UI (Error Boundary)                      │
│  ├── not-found.tsx         → 404 page                                       │
│  │                                                                           │
│  ├── dashboard/                                                              │
│  │   ├── page.tsx          → Route: /dashboard                              │
│  │   └── layout.tsx        → Nested layout                                  │
│  │                                                                           │
│  ├── medications/                                                            │
│  │   ├── page.tsx          → Route: /medications                            │
│  │   └── [id]/                                                               │
│  │       └── page.tsx      → Route: /medications/:id (dynamic)              │
│  │                                                                           │
│  └── api/                                                                    │
│      └── health/                                                             │
│          └── route.ts      → API: /api/health                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Terminology

| Term | Definition |
|------|------------|
| **Server Component** | Renders on server, no client JS |
| **Client Component** | Renders on client, interactive |
| **Route Handler** | API endpoint in Next.js |
| **Layout** | Shared UI wrapper for routes |
| **Loading UI** | Shown while page loads |
| **Streaming** | Progressive rendering |

---

## 3. Server vs Client Components

### The Mental Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SERVER vs CLIENT COMPONENTS                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  SERVER COMPONENTS (default)          │  CLIENT COMPONENTS                  │
│  ─────────────────────────            │  ─────────────────                  │
│                                       │                                     │
│  ✅ Fetch data directly               │  ✅ useState, useEffect             │
│  ✅ Access backend resources          │  ✅ Event handlers (onClick)        │
│  ✅ Keep secrets server-side          │  ✅ Browser APIs                    │
│  ✅ Reduce client bundle              │  ✅ Interactive UI                  │
│                                       │                                     │
│  ❌ No hooks                          │  ❌ Bigger JS bundle                │
│  ❌ No event handlers                 │  ❌ No direct server access         │
│                                       │                                     │
│  MARKER: (none, default)              │  MARKER: 'use client'               │
│                                       │                                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

### The Boundary Rule

```tsx
// page.tsx (Server Component - default)
async function MedicationsPage() {
  const meds = await getMedications();  // Server-side data fetch
  
  return (
    <div>
      <h1>Medications</h1>              {/* Server-rendered */}
      <MedicationList data={meds} />    {/* Can be server or client */}
      <AddButton />                     {/* Must be client (has onClick) */}
    </div>
  );
}

// AddButton.tsx (Client Component)
'use client';  // ← This marks the boundary

function AddButton() {
  const [open, setOpen] = useState(false);  // Now we can use hooks
  return <button onClick={() => setOpen(true)}>Add</button>;
}
```

---

## 4. Why Next.js for CareCircle

| Requirement | Why Next.js Fits |
|-------------|-----------------|
| SEO | Server-side rendering |
| Performance | Automatic code splitting |
| API routes | Co-located frontend/backend |
| TypeScript | First-class support |
| Vercel deployment | Optimized hosting |

---

## 5. When to Use Next.js Patterns ✅

### Use Server Components When:
- Fetching data from database/API
- Rendering static content
- Using large dependencies (keeps them off client)
- SEO-critical content

### Use Client Components When:
- User interaction required (clicks, forms)
- Using React hooks
- Accessing browser APIs
- Real-time updates

### Use Route Handlers When:
- Need API endpoints
- Webhook receivers
- Server-side form processing

---

## 6. When to AVOID Patterns ❌

### DON'T Use Client Components for Everything

```
❌ BAD: Everything is 'use client'
// Makes bundle huge, loses SSR benefits

✅ GOOD: Minimal client boundary
// Only mark interactive parts as client
```

### DON'T Fetch in Client Components

```tsx
❌ BAD: Fetch in useEffect
'use client';
function Page() {
  useEffect(() => { fetch('/api/data') }, []);
}

✅ GOOD: Fetch in Server Component
async function Page() {
  const data = await fetch('...');  // Server-side
  return <ClientComponent data={data} />;
}
```

---

## 7. Best Practices & Recommendations

### File Conventions

```
app/
├── (marketing)/           # Route group (no URL segment)
│   ├── about/
│   └── contact/
├── (app)/                 # Another group
│   ├── dashboard/
│   └── settings/
├── _components/           # Private folder (not a route)
├── loading.tsx            # Global loading
└── error.tsx              # Global error
```

### Data Fetching Patterns

```tsx
// Server Component - Direct fetch
async function Page() {
  const data = await prisma.user.findMany();
  return <UserList users={data} />;
}

// Server Component - API call
async function Page() {
  const res = await fetch('https://api.example.com/data', {
    cache: 'force-cache',      // Static
    // cache: 'no-store',      // Dynamic
    // next: { revalidate: 60 } // ISR
  });
  const data = await res.json();
  return <DataView data={data} />;
}
```

### Caching Strategies

| Strategy | When | How |
|----------|------|-----|
| Static | Data never changes | `cache: 'force-cache'` |
| Dynamic | Always fresh | `cache: 'no-store'` |
| ISR | Revalidate periodically | `next: { revalidate: 60 }` |

---

## 8. Common Mistakes & How to Avoid Them

### Mistake 1: Importing Server Code in Client

```tsx
❌ BAD: Importing prisma in client component
'use client';
import { prisma } from '@/lib/prisma';  // Error! Server-only

✅ GOOD: Pass data as props
// page.tsx (server)
const data = await prisma.user.findMany();
return <ClientComponent data={data} />;
```

### Mistake 2: Not Using Loading UI

```tsx
❌ BAD: No loading state
// Page shows blank while data loads

✅ GOOD: Add loading.tsx
// app/medications/loading.tsx
export default function Loading() {
  return <Skeleton />;
}
```

### Mistake 3: Unnecessary Client Components

Ask: "Does this NEED interactivity?"
- If no → Keep as Server Component
- If yes → Make only the interactive part client

---

## 9. Routing Patterns

### Dynamic Routes

```
app/
├── medications/
│   ├── [id]/           → /medications/123
│   │   └── page.tsx
│   └── [...slug]/      → /medications/a/b/c (catch-all)
│       └── page.tsx
```

### Route Groups

```
app/
├── (auth)/             # Grouped but no /auth in URL
│   ├── login/
│   └── register/
├── (dashboard)/        # Different layout
│   └── ...
```

### Parallel Routes

```
app/
├── @sidebar/           # Slot
│   └── page.tsx
├── @main/              # Another slot
│   └── page.tsx
└── layout.tsx          # Uses both: { sidebar, main }
```

---

## 10. Quick Reference

### Special Files

| File | Purpose |
|------|---------|
| `page.tsx` | Route UI |
| `layout.tsx` | Shared wrapper |
| `loading.tsx` | Loading UI |
| `error.tsx` | Error boundary |
| `not-found.tsx` | 404 page |
| `route.ts` | API endpoint |

### Metadata

```tsx
// Static
export const metadata = {
  title: 'Medications',
  description: 'Manage medications',
};

// Dynamic
export async function generateMetadata({ params }) {
  return { title: `Medication ${params.id}` };
}
```

### Navigation

```tsx
import Link from 'next/link';
import { useRouter } from 'next/navigation';

// Declarative
<Link href="/medications">View</Link>

// Programmatic
const router = useRouter();
router.push('/medications');
```

---

## 11. Learning Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Next.js Learn Course](https://nextjs.org/learn)
- Vercel's YouTube tutorials

---

*Next: [React](react.md) | [State Management](state-management.md)*
