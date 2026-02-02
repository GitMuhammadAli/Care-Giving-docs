# ⚛️ React Server Components (RSC) - Complete Guide

> A comprehensive guide to React Server Components - server/client boundaries, data fetching, streaming, and building modern full-stack React applications.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "React Server Components are components that render exclusively on the server, sending only their HTML output to the client - enabling direct database access, smaller bundles, and automatic code splitting while seamlessly integrating with interactive Client Components."

### The 7 Key Concepts (Remember These!)
```
1. SERVER COMPONENTS  → Run only on server, zero JS to client
2. CLIENT COMPONENTS  → Run on client, interactive, 'use client'
3. BOUNDARY           → 'use client' marks server/client split
4. COMPOSITION        → Server can render Client, not vice versa
5. STREAMING          → Progressive HTML delivery with Suspense
6. DATA FETCHING      → Direct DB/API access in Server Components
7. SERIALIZATION      → Props must be serializable across boundary
```

### Server vs Client Components
```
┌─────────────────────────────────────────────────────────────────┐
│              SERVER vs CLIENT COMPONENTS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SERVER COMPONENTS (Default in App Router)                     │
│  ──────────────────────────────────────────                     │
│  ✅ Direct database/API access                                 │
│  ✅ Access server-only resources (fs, env vars)                │
│  ✅ Zero JavaScript sent to client                             │
│  ✅ Automatic code splitting                                   │
│  ✅ Keep sensitive logic on server                             │
│  ❌ No useState, useEffect                                     │
│  ❌ No browser APIs                                            │
│  ❌ No event handlers                                          │
│                                                                 │
│  CLIENT COMPONENTS ('use client')                              │
│  ─────────────────────────────────                              │
│  ✅ useState, useEffect, useContext                            │
│  ✅ Event handlers (onClick, onChange)                         │
│  ✅ Browser APIs (window, localStorage)                        │
│  ✅ Custom hooks with state/effects                            │
│  ❌ Adds to bundle size                                        │
│  ❌ No direct server resources                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### When to Use Which
```
┌─────────────────────────────────────────────────────────────────┐
│              DECISION GUIDE                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  USE SERVER COMPONENT:                                         │
│  • Fetching data                                               │
│  • Accessing backend resources                                 │
│  • Rendering static content                                    │
│  • Sensitive logic (API keys, auth)                            │
│  • Large dependencies (markdown, syntax highlighting)          │
│                                                                 │
│  USE CLIENT COMPONENT:                                         │
│  • Interactivity (onClick, onChange)                           │
│  • State (useState, useReducer)                                │
│  • Effects (useEffect, useLayoutEffect)                        │
│  • Browser APIs (window, localStorage)                         │
│  • Refs to DOM elements                                        │
│  • Third-party components with client features                 │
│                                                                 │
│  PATTERN: Keep client boundary as low as possible              │
│  • Page = Server Component                                     │
│  • Interactive widget = Client Component                       │
│  • Data display = Server Component                             │
│  • Form inputs = Client Component                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Server boundary"** | "The 'use client' directive marks the server/client boundary" |
| **"Zero bundle"** | "Server Components have zero bundle impact" |
| **"Serializable props"** | "Props crossing the boundary must be serializable" |
| **"Progressive rendering"** | "RSC enables progressive rendering with streaming" |
| **"Server-only"** | "Database queries stay server-only, never exposed to client" |
| **"Composition pattern"** | "Server components compose client components, not the reverse" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Client JS reduction | **30-50%** | Typical RSC migration |
| Bundle impact | **0 bytes** | Server Component JS |
| Default | **Server** | All components in App Router |
| Suspense boundaries | **Multiple** | For streaming |

### The "Wow" Statement (Memorize This!)
> "With React Server Components, our page components fetch data directly from the database - no API layer needed. The query code never ships to the client, bundle dropped 40%. Interactive parts are Client Components, kept as leaves in the tree. Server Components pass serializable props down. We compose them: Server renders the page shell, passes data to Client Components for interactivity. Large dependencies like markdown parsers stay server-only. Streaming with Suspense means users see content progressively. The boundary is clear: 'use client' for interactive pieces, everything else is Server by default."

---

## 📚 Table of Contents

1. [Server Components Basics](#1-server-components-basics)
2. [Client Components](#2-client-components)
3. [Composition Patterns](#3-composition-patterns)
4. [Data Fetching](#4-data-fetching)
5. [Streaming & Suspense](#5-streaming--suspense)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Server Components Basics

```tsx
// ════════════════════════════════════════════════════════════════
// SERVER COMPONENTS (Default in Next.js App Router)
// ════════════════════════════════════════════════════════════════

// app/products/page.tsx - Server Component (no directive needed)
import { db } from '@/lib/db';

// This function only runs on the server
async function ProductsPage() {
  // Direct database access!
  const products = await db.product.findMany({
    orderBy: { createdAt: 'desc' },
    take: 20,
  });

  // This code NEVER reaches the client
  // - db import not bundled
  // - SQL query not exposed
  
  return (
    <div>
      <h1>Products</h1>
      <ul>
        {products.map(product => (
          <li key={product.id}>
            {product.name} - ${product.price}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default ProductsPage;

// ════════════════════════════════════════════════════════════════
// SERVER COMPONENT CAPABILITIES
// ════════════════════════════════════════════════════════════════

// Access environment variables (server secrets)
async function ConfigDisplay() {
  const apiKey = process.env.SECRET_API_KEY; // Safe - never sent to client
  const data = await fetchFromAPI(apiKey);
  
  return <div>{data.result}</div>;
}

// Read files from filesystem
import { readFile } from 'fs/promises';

async function MarkdownPage({ slug }: { slug: string }) {
  const content = await readFile(`./content/${slug}.md`, 'utf-8');
  const html = await markdownToHtml(content);
  
  return <article dangerouslySetInnerHTML={{ __html: html }} />;
}

// Use heavy libraries without bundle cost
import { highlight } from 'prismjs'; // Only on server

async function CodeBlock({ code, language }: CodeBlockProps) {
  const highlighted = highlight(code, language);
  
  return (
    <pre>
      <code dangerouslySetInnerHTML={{ __html: highlighted }} />
    </pre>
  );
}

// ════════════════════════════════════════════════════════════════
// WHAT SERVER COMPONENTS CAN'T DO
// ════════════════════════════════════════════════════════════════

// ❌ No React hooks that use state/effects
function ServerComponent() {
  // These will ERROR in Server Component:
  const [count, setCount] = useState(0);    // ❌
  useEffect(() => {}, []);                   // ❌
  useContext(SomeContext);                   // ❌ (use client context)
  
  // These are fine:
  const id = useId();                        // ✅ (generates unique ID)
}

// ❌ No event handlers
function ServerComponent() {
  return (
    // This won't work - no interactivity
    <button onClick={() => alert('Hi')}>  // ❌ onClick ignored
      Click me
    </button>
  );
}

// ❌ No browser APIs
function ServerComponent() {
  // These don't exist on server:
  window.localStorage;  // ❌
  document.cookie;      // ❌
  navigator.userAgent;  // ❌
}
```

---

## 2. Client Components

```tsx
// ════════════════════════════════════════════════════════════════
// CLIENT COMPONENTS ('use client' directive)
// ════════════════════════════════════════════════════════════════

// components/Counter.tsx
'use client'; // This directive marks the client boundary

import { useState } from 'react';

export function Counter() {
  // Now we can use hooks!
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// USING CLIENT COMPONENTS IN SERVER COMPONENTS
// ════════════════════════════════════════════════════════════════

// app/page.tsx - Server Component
import { Counter } from '@/components/Counter';
import { AddToCart } from '@/components/AddToCart';
import { db } from '@/lib/db';

async function HomePage() {
  const products = await db.product.findMany();

  return (
    <main>
      <h1>Welcome</h1>
      
      {/* Client Component for interactivity */}
      <Counter />
      
      {/* Server-rendered data passed to Client Component */}
      {products.map(product => (
        <div key={product.id}>
          <h2>{product.name}</h2>
          <p>{product.description}</p>
          {/* Client Component receives serializable props */}
          <AddToCart productId={product.id} />
        </div>
      ))}
    </main>
  );
}

// ════════════════════════════════════════════════════════════════
// PROPS MUST BE SERIALIZABLE
// ════════════════════════════════════════════════════════════════

// ❌ BAD: Can't pass functions across boundary
async function ServerParent() {
  const handleClick = () => console.log('clicked');
  
  return <ClientChild onClick={handleClick} />; // ❌ Error!
}

// ❌ BAD: Can't pass class instances
async function ServerParent() {
  const date = new Date();
  
  return <ClientChild date={date} />; // ❌ Error!
}

// ✅ GOOD: Pass serializable data
async function ServerParent() {
  const data = await fetchData();
  
  return (
    <ClientChild 
      // Primitives ✅
      count={42}
      name="John"
      isActive={true}
      
      // Serializable objects ✅
      user={{ id: 1, name: 'John' }}
      items={['a', 'b', 'c']}
      
      // Dates as strings ✅
      createdAt={new Date().toISOString()}
    />
  );
}

// ════════════════════════════════════════════════════════════════
// CLIENT COMPONENT BOUNDARIES
// ════════════════════════════════════════════════════════════════

// 'use client' creates a boundary
// Everything imported into a Client Component becomes client code

// components/InteractiveSection.tsx
'use client';

import { useState } from 'react';
import { Button } from './Button'; // Button becomes client code too!
import { helpers } from './utils'; // utils becomes client code too!

export function InteractiveSection() {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <div>
      <Button onClick={() => setIsOpen(!isOpen)}>Toggle</Button>
      {isOpen && <p>Content</p>}
    </div>
  );
}

// TIP: Keep client boundaries as low as possible
// Only mark the interactive leaf components as 'use client'
```

---

## 3. Composition Patterns

```tsx
// ════════════════════════════════════════════════════════════════
// PATTERN 1: SERVER PARENT WITH CLIENT CHILDREN
// ════════════════════════════════════════════════════════════════

// app/products/[id]/page.tsx - Server Component
import { ProductDetails } from './ProductDetails'; // Server
import { AddToCartButton } from './AddToCartButton'; // Client

async function ProductPage({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id);

  return (
    <article>
      {/* Server Component */}
      <ProductDetails product={product} />
      
      {/* Client Component for interactivity */}
      <AddToCartButton 
        productId={product.id} 
        price={product.price} 
      />
    </article>
  );
}

// ════════════════════════════════════════════════════════════════
// PATTERN 2: PASSING SERVER COMPONENTS AS CHILDREN
// ════════════════════════════════════════════════════════════════

// components/Modal.tsx - Client Component
'use client';

import { useState } from 'react';

export function Modal({ 
  trigger, 
  children 
}: { 
  trigger: React.ReactNode;
  children: React.ReactNode;
}) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsOpen(true)}>{trigger}</button>
      {isOpen && (
        <div className="modal">
          {children} {/* Server Component can be passed here! */}
          <button onClick={() => setIsOpen(false)}>Close</button>
        </div>
      )}
    </>
  );
}

// app/page.tsx - Server Component
import { Modal } from '@/components/Modal';
import { ProductDetails } from './ProductDetails'; // Server Component

async function Page() {
  const product = await getProduct('123');

  return (
    <Modal trigger="View Details">
      {/* Server Component passed as children to Client Component */}
      <ProductDetails product={product} />
    </Modal>
  );
}

// ════════════════════════════════════════════════════════════════
// PATTERN 3: SLOT PATTERN FOR MIXED COMPOSITION
// ════════════════════════════════════════════════════════════════

// components/Card.tsx - Client Component
'use client';

interface CardProps {
  header: React.ReactNode;   // Slot for server content
  footer: React.ReactNode;   // Slot for server content
  children: React.ReactNode;
  onExpand?: () => void;
}

export function Card({ header, footer, children, onExpand }: CardProps) {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <div className="card">
      <div className="card-header">{header}</div>
      
      <div className={`card-body ${isExpanded ? 'expanded' : ''}`}>
        {children}
      </div>
      
      {onExpand && (
        <button onClick={() => setIsExpanded(!isExpanded)}>
          {isExpanded ? 'Collapse' : 'Expand'}
        </button>
      )}
      
      <div className="card-footer">{footer}</div>
    </div>
  );
}

// app/page.tsx
async function Page() {
  const product = await getProduct('123');
  const reviews = await getReviews('123');

  return (
    <Card
      header={<ProductHeader product={product} />}  // Server Component
      footer={<ProductFooter price={product.price} />}  // Server Component
    >
      <ReviewsList reviews={reviews} />  {/* Server Component */}
    </Card>
  );
}

// ════════════════════════════════════════════════════════════════
// PATTERN 4: CONTEXT PROVIDERS
// ════════════════════════════════════════════════════════════════

// providers/ThemeProvider.tsx - Client Component
'use client';

import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext<{
  theme: 'light' | 'dark';
  toggle: () => void;
} | null>(null);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  return (
    <ThemeContext.Provider 
      value={{ 
        theme, 
        toggle: () => setTheme(t => t === 'light' ? 'dark' : 'light') 
      }}
    >
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}

// app/layout.tsx - Server Component wrapping Client Provider
import { ThemeProvider } from '@/providers/ThemeProvider';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <ThemeProvider>
          {children} {/* Server Components can be children */}
        </ThemeProvider>
      </body>
    </html>
  );
}
```

---

## 4. Data Fetching

```tsx
// ════════════════════════════════════════════════════════════════
// DATA FETCHING IN SERVER COMPONENTS
// ════════════════════════════════════════════════════════════════

// Direct database access
import { prisma } from '@/lib/prisma';

async function UsersPage() {
  // Runs on server only - prisma never bundled for client
  const users = await prisma.user.findMany({
    include: { posts: true },
  });

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name} - {user.posts.length} posts
        </li>
      ))}
    </ul>
  );
}

// ════════════════════════════════════════════════════════════════
// FETCH WITH CACHING (Next.js)
// ════════════════════════════════════════════════════════════════

async function ProductPage({ id }: { id: string }) {
  // Cached by default
  const product = await fetch(`${API}/products/${id}`).then(r => r.json());
  
  // Revalidate every 60 seconds
  const reviews = await fetch(`${API}/products/${id}/reviews`, {
    next: { revalidate: 60 },
  }).then(r => r.json());
  
  // No caching (always fresh)
  const stock = await fetch(`${API}/products/${id}/stock`, {
    cache: 'no-store',
  }).then(r => r.json());

  return (
    <div>
      <h1>{product.name}</h1>
      <p>Stock: {stock.quantity}</p>
      <ReviewList reviews={reviews} />
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// PARALLEL DATA FETCHING
// ════════════════════════════════════════════════════════════════

async function DashboardPage() {
  // Parallel fetching - faster than sequential
  const [user, orders, notifications] = await Promise.all([
    getUser(),
    getOrders(),
    getNotifications(),
  ]);

  return (
    <div>
      <UserProfile user={user} />
      <OrderList orders={orders} />
      <NotificationList notifications={notifications} />
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// DATA FETCHING WITH SERVER ACTIONS
// ════════════════════════════════════════════════════════════════

// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  await prisma.post.create({
    data: { title, content },
  });

  revalidatePath('/posts');
}

// app/posts/new/page.tsx - Server Component with form
import { createPost } from '@/app/actions';

function NewPostPage() {
  return (
    <form action={createPost}>
      <input name="title" required />
      <textarea name="content" required />
      <button type="submit">Create Post</button>
    </form>
  );
}

// components/CreatePostForm.tsx - Client Component for better UX
'use client';

import { createPost } from '@/app/actions';
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Creating...' : 'Create Post'}
    </button>
  );
}

export function CreatePostForm() {
  return (
    <form action={createPost}>
      <input name="title" required />
      <textarea name="content" required />
      <SubmitButton />
    </form>
  );
}
```

---

## 5. Streaming & Suspense

```tsx
// ════════════════════════════════════════════════════════════════
// STREAMING WITH SUSPENSE
// ════════════════════════════════════════════════════════════════

import { Suspense } from 'react';

async function ProductPage({ id }: { id: string }) {
  return (
    <div>
      {/* Sent immediately */}
      <h1>Product Details</h1>
      
      {/* Streams in when ready */}
      <Suspense fallback={<ProductSkeleton />}>
        <ProductDetails id={id} />
      </Suspense>
      
      {/* Streams in when ready (can be slow) */}
      <Suspense fallback={<ReviewsSkeleton />}>
        <ProductReviews id={id} />
      </Suspense>
      
      {/* Streams in when ready */}
      <Suspense fallback={<RecommendationsSkeleton />}>
        <Recommendations productId={id} />
      </Suspense>
    </div>
  );
}

// Each async component streams independently
async function ProductDetails({ id }: { id: string }) {
  const product = await getProduct(id); // 100ms
  return <div>{product.name}</div>;
}

async function ProductReviews({ id }: { id: string }) {
  const reviews = await getReviews(id); // 2000ms - slow!
  return <div>{reviews.length} reviews</div>;
}

async function Recommendations({ productId }: { productId: string }) {
  const recommendations = await getRecommendations(productId); // 500ms
  return <div>{recommendations.length} recommendations</div>;
}

// ════════════════════════════════════════════════════════════════
// NESTED SUSPENSE
// ════════════════════════════════════════════════════════════════

async function UserDashboard() {
  return (
    <Suspense fallback={<DashboardSkeleton />}>
      <Dashboard />
    </Suspense>
  );
}

async function Dashboard() {
  const user = await getUser(); // Needed for everything

  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      
      {/* Nested suspense for independent sections */}
      <div className="grid">
        <Suspense fallback={<StatsSkeleton />}>
          <Stats userId={user.id} />
        </Suspense>
        
        <Suspense fallback={<OrdersSkeleton />}>
          <RecentOrders userId={user.id} />
        </Suspense>
        
        <Suspense fallback={<NotificationsSkeleton />}>
          <Notifications userId={user.id} />
        </Suspense>
      </div>
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// LOADING.TSX (File-based Suspense)
// ════════════════════════════════════════════════════════════════

// app/products/loading.tsx
export default function Loading() {
  return (
    <div className="loading">
      <Spinner />
      <p>Loading products...</p>
    </div>
  );
}

// app/products/page.tsx
async function ProductsPage() {
  const products = await getProducts(); // loading.tsx shows while this runs
  
  return (
    <ul>
      {products.map(p => <ProductCard key={p.id} product={p} />)}
    </ul>
  );
}

// Equivalent to:
// <Suspense fallback={<Loading />}>
//   <ProductsPage />
// </Suspense>
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# RSC PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Using hooks in Server Components
# Bad
async function ServerComponent() {
  const [count, setCount] = useState(0);  # Error!
}

# Good
# Add 'use client' or move hook to Client Component

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Passing non-serializable props
# Bad
async function Server() {
  const handleClick = () => {};
  return <Client onClick={handleClick} />;  # Error!
}

# Good
# Define event handlers in Client Component
# Or use Server Actions for forms

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Importing server code in client
# Bad
'use client';
import { db } from '@/lib/db';  # db bundled for client!

# Good
# Keep server imports in Server Components
# Use Server Actions for mutations

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Making everything 'use client'
# Bad
'use client';
async function Page() {  # Lost all RSC benefits!
  const data = await fetchData();
}

# Good
# Keep 'use client' for leaf components only
# Most components should be Server Components

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Client Component importing Server Component
# Bad
'use client';
import { ServerThing } from './ServerThing';  # Won't work as expected

# Good
# Server Component passes Server children to Client
<ClientWrapper>
  <ServerThing />  # Passed as children
</ClientWrapper>
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What are React Server Components?"**
> "Server Components render only on the server, sending HTML to the client with zero JavaScript. They can directly access databases, filesystem, and server resources. Client Components (marked with 'use client') handle interactivity. This enables smaller bundles and secure server-side logic."

**Q: "What's the difference between Server and Client Components?"**
> "Server Components: run on server, can be async, access backend directly, no hooks/event handlers, zero bundle cost. Client Components: run in browser, can use useState/useEffect, handle events, add to bundle. Default is Server in Next.js App Router."

**Q: "What are the rules for passing props between Server and Client Components?"**
> "Props crossing the server/client boundary must be serializable: primitives, plain objects, arrays. No functions, class instances, or Dates. Convert Date to ISO string. Functions like event handlers must be defined in Client Components."

### Intermediate Questions

**Q: "How do you compose Server and Client Components?"**
> "Server Components can render Client Components and pass serializable props. Client Components can receive Server Components as children (slot pattern). This lets you keep interactivity at the leaves while maximizing server rendering. The children are pre-rendered on server."

**Q: "How does data fetching differ in Server Components?"**
> "Server Components can directly await async operations - database queries, file reads, API calls. No useEffect or loading states needed at component level. Data is fetched at render time. Use Suspense boundaries for streaming and loading UI."

**Q: "What is streaming with Server Components?"**
> "Streaming progressively sends HTML as Server Components resolve. Wrap slow components in Suspense - the fallback shows immediately, actual content streams in when ready. Multiple Suspense boundaries enable parallel streaming. Users see content faster."

### Advanced Questions

**Q: "How do Server Actions work?"**
> "Server Actions are async functions marked with 'use server' that run on the server. They can be called from Client Components for mutations. Forms can use them as action prop. They enable RPC-like calls without API routes. Use revalidatePath/revalidateTag to update cached data."

**Q: "What are the bundle size implications?"**
> "Server Components contribute zero JS to client bundle - their code, dependencies, and data fetching stay on server. Only Client Components add to bundle. Keep client boundaries small and at leaves. Large dependencies like markdown parsers can stay server-only."

**Q: "How do you decide what should be Server vs Client?"**
> "Server: data fetching, database access, large dependencies, static content, sensitive logic. Client: interactivity, state, effects, browser APIs. Pattern: pages and layouts are Server, interactive widgets are Client. Push client boundary down as far as possible."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              RSC CHECKLIST                                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SERVER COMPONENTS (Default):                                   │
│  □ Data fetching                                               │
│  □ Database/API access                                         │
│  □ Large dependencies                                          │
│  □ Static content                                              │
│  □ Sensitive logic                                             │
│                                                                 │
│  CLIENT COMPONENTS ('use client'):                              │
│  □ useState, useEffect                                         │
│  □ Event handlers                                              │
│  □ Browser APIs                                                │
│  □ Third-party interactive libraries                           │
│                                                                 │
│  COMPOSITION:                                                   │
│  □ Server renders Client (props must serialize)                │
│  □ Client receives Server as children (slot pattern)           │
│  □ Context providers are Client, wrap Server children          │
│                                                                 │
│  STREAMING:                                                     │
│  □ Suspense boundaries for async components                    │
│  □ loading.tsx for route-level loading                         │
│  □ Parallel data fetching with Promise.all                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

QUICK RULES:
┌────────────────────────────────────────────────────────────────┐
│ Default:        Server Component                               │
│ Interactivity:  Add 'use client'                               │
│ Props:          Must be serializable across boundary           │
│ Composition:    Server → Client ✓, Client → Server (as child)  │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

