# 🔄 SSR vs SSG vs CSR vs ISR - Complete Guide

> A comprehensive guide to rendering strategies - Server-Side Rendering, Static Site Generation, Client-Side Rendering, Incremental Static Regeneration, and choosing the right approach.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Rendering strategies determine when and where HTML is generated: CSR renders in browser (SPA), SSR renders on server per-request, SSG pre-renders at build time, and ISR combines SSG with background revalidation for dynamic static content."

### The 7 Key Concepts (Remember These!)
```
1. CSR (Client-Side)     → Browser renders, SPA, dynamic
2. SSR (Server-Side)     → Server renders per request
3. SSG (Static)          → Pre-rendered at build time
4. ISR (Incremental)     → Static + background revalidation
5. HYDRATION            → Making server HTML interactive
6. STREAMING            → Progressive server rendering
7. RSC (Server Components) → Components that stay on server
```

### Rendering Strategy Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│              RENDERING STRATEGIES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CSR (Client-Side Rendering)                                   │
│  ─────────────────────────────                                  │
│  • Browser downloads JS, renders content                       │
│  • Initial: blank page → loading → content                     │
│  ✅ Rich interactivity, no server needed                       │
│  ❌ Slow FCP, poor SEO, JS required                            │
│  Use: Dashboards, authenticated apps, SPAs                     │
│                                                                 │
│  SSR (Server-Side Rendering)                                   │
│  ──────────────────────────────                                 │
│  • Server generates HTML per request                           │
│  • Initial: full HTML → hydrate → interactive                  │
│  ✅ Fast FCP, good SEO, dynamic content                        │
│  ❌ Server load, TTFB latency, expensive                       │
│  Use: Personalized content, frequently changing data           │
│                                                                 │
│  SSG (Static Site Generation)                                  │
│  ─────────────────────────────                                  │
│  • HTML generated at build time                                │
│  • Initial: cached HTML → hydrate → interactive                │
│  ✅ Fastest, CDN cacheable, cheapest                           │
│  ❌ Stale data, long builds, no personalization               │
│  Use: Blogs, docs, marketing pages                             │
│                                                                 │
│  ISR (Incremental Static Regeneration)                         │
│  ──────────────────────────────────────                         │
│  • Static + background revalidation                            │
│  • Stale-while-revalidate for pages                            │
│  ✅ Static performance + fresh data                            │
│  ❌ Still shows stale briefly, Next.js specific                │
│  Use: E-commerce, news, product pages                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### When to Use What
```
┌─────────────────────────────────────────────────────────────────┐
│              DECISION MATRIX                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CONTENT TYPE              │ STRATEGY                          │
│  ──────────────────────────│─────────────────────────────────  │
│  Static content            │ SSG                               │
│  (blog, docs, marketing)   │                                   │
│                            │                                   │
│  Changes often, SEO needed │ ISR or SSR                        │
│  (product pages, news)     │                                   │
│                            │                                   │
│  Personalized, dynamic     │ SSR                               │
│  (user dashboard, feed)    │                                   │
│                            │                                   │
│  Behind login, no SEO      │ CSR                               │
│  (admin, internal tools)   │                                   │
│                            │                                   │
│  Mixed content             │ Hybrid (per-page strategy)        │
│                            │                                   │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  QUESTIONS TO ASK:                                             │
│  1. Does this page need SEO? → SSR/SSG/ISR                     │
│  2. Is content the same for all users? → SSG/ISR               │
│  3. How often does content change? → ISR/SSR                   │
│  4. Is it behind authentication? → CSR often fine              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Hydration"** | "The server HTML hydrates on client with React" |
| **"TTFB"** | "SSG has better TTFB than SSR since it's pre-built" |
| **"Revalidation"** | "ISR revalidates pages in the background" |
| **"Streaming"** | "We use streaming SSR to reduce TTFB" |
| **"Static generation"** | "Marketing pages use static generation at build" |
| **"On-demand ISR"** | "Webhook triggers on-demand ISR when CMS updates" |

### Key Numbers to Remember
| Metric | CSR | SSR | SSG | ISR |
|--------|-----|-----|-----|-----|
| FCP | Slow | Fast | Fastest | Fastest |
| TTFB | Fast | Slow | Fastest | Fastest |
| SEO | Poor | Good | Good | Good |
| Server | None | High | None | Low |

### The "Wow" Statement (Memorize This!)
> "We use a hybrid approach: marketing pages are SSG for instant load from CDN, product pages use ISR with 60-second revalidation so they're fast but fresh, user dashboards are SSR for personalized content. The product catalog has 50,000 pages - SSG would take hours, so we use ISR with on-demand revalidation triggered by CMS webhooks. User-specific components within SSR pages are client components that fetch after hydration. We stream SSR with Suspense boundaries so users see content progressively. This gives us sub-second FCP across the site while keeping data fresh."

---

## 📚 Table of Contents

1. [CSR (Client-Side Rendering)](#1-csr-client-side-rendering)
2. [SSR (Server-Side Rendering)](#2-ssr-server-side-rendering)
3. [SSG (Static Site Generation)](#3-ssg-static-site-generation)
4. [ISR (Incremental Static Regeneration)](#4-isr-incremental-static-regeneration)
5. [Hydration & Streaming](#5-hydration--streaming)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. CSR (Client-Side Rendering)

```tsx
// ════════════════════════════════════════════════════════════════
// CSR - TRADITIONAL REACT SPA
// ════════════════════════════════════════════════════════════════

// index.html - Empty shell
<!DOCTYPE html>
<html>
<head>
  <title>My App</title>
</head>
<body>
  <div id="root"></div> <!-- Empty, JS fills it -->
  <script src="/bundle.js"></script>
</body>
</html>

// App.tsx - Client-side data fetching
import { useQuery } from '@tanstack/react-query';

function ProductPage({ productId }: { productId: string }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['product', productId],
    queryFn: () => fetchProduct(productId),
  });

  if (isLoading) return <ProductSkeleton />;
  if (error) return <Error error={error} />;

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
      <p>${data.price}</p>
    </div>
  );
}

// Timeline:
// 1. Browser requests page
// 2. Server sends empty HTML with JS bundle
// 3. Browser downloads and parses JS
// 4. React renders loading state
// 5. fetch() calls API
// 6. React renders content

// Pros:
// - Rich interactivity
// - No server needed (static hosting)
// - Fast subsequent navigations

// Cons:
// - Slow initial load (download + parse JS)
// - Poor SEO (empty HTML)
// - Requires JavaScript
// - Loading spinners everywhere
```

---

## 2. SSR (Server-Side Rendering)

```tsx
// ════════════════════════════════════════════════════════════════
// SSR - NEXT.JS SERVER-SIDE RENDERING
// ════════════════════════════════════════════════════════════════

// app/products/[id]/page.tsx (Next.js App Router)
async function ProductPage({ params }: { params: { id: string } }) {
  // Runs on server for every request
  const product = await fetchProduct(params.id);

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>${product.price}</p>
      
      {/* Client component for interactivity */}
      <AddToCartButton productId={product.id} />
    </div>
  );
}

export default ProductPage;

// Force dynamic rendering (no caching)
export const dynamic = 'force-dynamic';

// ════════════════════════════════════════════════════════════════
// SSR WITH CACHING (Next.js)
// ════════════════════════════════════════════════════════════════

async function ProductPage({ params }: { params: { id: string } }) {
  // Cache for 60 seconds
  const product = await fetch(`${API}/products/${params.id}`, {
    next: { revalidate: 60 },
  }).then(r => r.json());

  return <ProductDetails product={product} />;
}

// ════════════════════════════════════════════════════════════════
// SSR - PAGES ROUTER (getServerSideProps)
// ════════════════════════════════════════════════════════════════

// pages/products/[id].tsx
export async function getServerSideProps(context: GetServerSidePropsContext) {
  const { id } = context.params!;
  
  try {
    const product = await fetchProduct(id);
    
    return {
      props: { product },
    };
  } catch (error) {
    return {
      notFound: true,
    };
  }
}

function ProductPage({ product }: { product: Product }) {
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
}

export default ProductPage;

// Timeline:
// 1. Browser requests page
// 2. Server fetches data
// 3. Server renders React to HTML
// 4. Server sends complete HTML
// 5. Browser shows content immediately
// 6. JS downloads and hydrates
// 7. Page becomes interactive

// Pros:
// - Fast FCP (content visible immediately)
// - Good SEO (full HTML)
// - Always fresh data
// - Works without JS (basic functionality)

// Cons:
// - Slower TTFB (server must render)
// - Server load on every request
// - More expensive infrastructure
```

---

## 3. SSG (Static Site Generation)

```tsx
// ════════════════════════════════════════════════════════════════
// SSG - NEXT.JS STATIC GENERATION
// ════════════════════════════════════════════════════════════════

// app/blog/[slug]/page.tsx (App Router)
import { notFound } from 'next/navigation';

// Generate static pages at build time
export async function generateStaticParams() {
  const posts = await getAllPosts();
  
  return posts.map((post) => ({
    slug: post.slug,
  }));
}

async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug);
  
  if (!post) notFound();

  return (
    <article>
      <h1>{post.title}</h1>
      <time>{post.date}</time>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

export default BlogPost;

// ════════════════════════════════════════════════════════════════
// SSG - PAGES ROUTER (getStaticProps + getStaticPaths)
// ════════════════════════════════════════════════════════════════

// pages/blog/[slug].tsx
export async function getStaticPaths() {
  const posts = await getAllPosts();
  
  return {
    paths: posts.map((post) => ({
      params: { slug: post.slug },
    })),
    fallback: false, // 404 for unknown paths
  };
}

export async function getStaticProps({ params }: GetStaticPropsContext) {
  const post = await getPost(params!.slug);
  
  if (!post) {
    return { notFound: true };
  }

  return {
    props: { post },
  };
}

function BlogPost({ post }: { post: Post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}

export default BlogPost;

// ════════════════════════════════════════════════════════════════
// SSG FALLBACK STRATEGIES
// ════════════════════════════════════════════════════════════════

export async function getStaticPaths() {
  const popularPosts = await getPopularPosts();
  
  return {
    paths: popularPosts.map((post) => ({
      params: { slug: post.slug },
    })),
    
    // fallback options:
    // false: 404 for paths not in paths array
    // true: Generate on first request, show fallback
    // 'blocking': Generate on first request, no fallback (like SSR)
    
    fallback: 'blocking', // Best for ISR-like behavior
  };
}

// With fallback: true, check if page is ready
function BlogPost({ post }: { post: Post }) {
  const router = useRouter();
  
  // Show loading while generating
  if (router.isFallback) {
    return <PostSkeleton />;
  }

  return <article>{/* ... */}</article>;
}

// Timeline:
// 1. Build time: fetch data, render HTML
// 2. Deploy static files to CDN
// 3. Browser requests page
// 4. CDN serves pre-built HTML instantly
// 5. JS downloads and hydrates

// Pros:
// - Fastest possible (pre-built)
// - Cheapest (CDN, no server)
// - Best SEO
// - Very reliable

// Cons:
// - Stale data until rebuild
// - Long build times for many pages
// - No personalization
```

---

## 4. ISR (Incremental Static Regeneration)

```tsx
// ════════════════════════════════════════════════════════════════
// ISR - NEXT.JS INCREMENTAL STATIC REGENERATION
// ════════════════════════════════════════════════════════════════

// app/products/[id]/page.tsx (App Router)
async function ProductPage({ params }: { params: { id: string } }) {
  const product = await fetch(`${API}/products/${params.id}`, {
    next: { revalidate: 60 }, // Revalidate every 60 seconds
  }).then(r => r.json());

  return (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
      <p>Stock: {product.stock}</p>
    </div>
  );
}

// Or at page level
export const revalidate = 60; // Revalidate every 60 seconds

// ════════════════════════════════════════════════════════════════
// ISR - PAGES ROUTER
// ════════════════════════════════════════════════════════════════

// pages/products/[id].tsx
export async function getStaticProps({ params }: GetStaticPropsContext) {
  const product = await fetchProduct(params!.id);

  return {
    props: { product },
    revalidate: 60, // Regenerate at most every 60 seconds
  };
}

// How ISR works:
// 1. First request: serve cached page
// 2. If stale (>60s), trigger background regeneration
// 3. Next request: serve NEW page (or still old if regen not done)
// 4. Stale-while-revalidate pattern

// ════════════════════════════════════════════════════════════════
// ON-DEMAND ISR (Webhook trigger)
// ════════════════════════════════════════════════════════════════

// app/api/revalidate/route.ts (App Router)
import { revalidatePath, revalidateTag } from 'next/cache';
import { NextRequest } from 'next/server';

export async function POST(request: NextRequest) {
  const { secret, path, tag } = await request.json();

  // Verify secret
  if (secret !== process.env.REVALIDATION_SECRET) {
    return Response.json({ error: 'Invalid secret' }, { status: 401 });
  }

  // Revalidate by path
  if (path) {
    revalidatePath(path);
    return Response.json({ revalidated: true, path });
  }

  // Revalidate by tag
  if (tag) {
    revalidateTag(tag);
    return Response.json({ revalidated: true, tag });
  }

  return Response.json({ error: 'Path or tag required' }, { status: 400 });
}

// Usage in fetch:
const product = await fetch(`${API}/products/${id}`, {
  next: { tags: ['products', `product-${id}`] },
});

// CMS webhook calls:
// POST /api/revalidate { secret: "xxx", tag: "product-123" }

// ════════════════════════════════════════════════════════════════
// ISR - PAGES ROUTER ON-DEMAND
// ════════════════════════════════════════════════════════════════

// pages/api/revalidate.ts
export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.query.secret !== process.env.REVALIDATION_SECRET) {
    return res.status(401).json({ message: 'Invalid token' });
  }

  try {
    const path = req.query.path as string;
    await res.revalidate(path);
    return res.json({ revalidated: true });
  } catch (err) {
    return res.status(500).send('Error revalidating');
  }
}

// Timeline:
// 1. Build: generate static pages
// 2. First request: serve from cache (fast!)
// 3. If stale: regenerate in background
// 4. On webhook: regenerate immediately

// Pros:
// - Static performance
// - Data can be relatively fresh
// - Handles many pages (generate on-demand)
// - Webhook support for instant updates

// Cons:
// - Briefly stale after change
// - Next.js specific
// - More complex than pure SSG
```

---

## 5. Hydration & Streaming

```tsx
// ════════════════════════════════════════════════════════════════
// HYDRATION
// ════════════════════════════════════════════════════════════════

// Hydration: Attaching React event handlers to server-rendered HTML

// Server sends:
<div id="root">
  <button>Count: 0</button> <!-- Static HTML -->
</div>

// After hydration, button becomes interactive
// React attaches onClick handler to existing DOM

// Hydration mismatch: Server/client HTML differs
// ❌ BAD: Will cause hydration error
function Clock() {
  return <span>{new Date().toLocaleTimeString()}</span>;
  // Server time ≠ client time
}

// ✅ GOOD: Render on client only
function Clock() {
  const [time, setTime] = useState<string>();
  
  useEffect(() => {
    setTime(new Date().toLocaleTimeString());
    const interval = setInterval(() => {
      setTime(new Date().toLocaleTimeString());
    }, 1000);
    return () => clearInterval(interval);
  }, []);

  return <span>{time ?? '--:--:--'}</span>;
}

// ════════════════════════════════════════════════════════════════
// STREAMING SSR
// ════════════════════════════════════════════════════════════════

// Without streaming: Wait for ALL data, then send HTML
// With streaming: Send HTML progressively as ready

// Next.js App Router - Streaming with Suspense
async function ProductPage({ params }: { params: { id: string } }) {
  return (
    <div>
      {/* Sent immediately */}
      <Header />
      
      {/* Streams in when ready */}
      <Suspense fallback={<ProductSkeleton />}>
        <ProductDetails id={params.id} />
      </Suspense>
      
      {/* Streams in when ready */}
      <Suspense fallback={<ReviewsSkeleton />}>
        <ProductReviews id={params.id} />
      </Suspense>
      
      {/* Sent immediately */}
      <Footer />
    </div>
  );
}

async function ProductDetails({ id }: { id: string }) {
  // This fetch blocks only this component
  const product = await fetchProduct(id);
  return <div>{product.name}</div>;
}

async function ProductReviews({ id }: { id: string }) {
  // Slow API doesn't block product details
  const reviews = await fetchReviews(id); // 2 seconds
  return <ReviewList reviews={reviews} />;
}

// Timeline with streaming:
// 0ms: Header, navigation, skeletons sent
// 100ms: ProductDetails streams in, replaces skeleton
// 2000ms: ProductReviews streams in, replaces skeleton

// ════════════════════════════════════════════════════════════════
// PARTIAL PRERENDERING (PPR) - Next.js 14+
// ════════════════════════════════════════════════════════════════

// Combines static shell with streaming dynamic content
// next.config.js
module.exports = {
  experimental: {
    ppr: true,
  },
};

// Static shell + dynamic holes
async function ProductPage({ params }: { params: { id: string } }) {
  return (
    <div>
      {/* Static - prerendered at build */}
      <Header />
      <ProductInfo id={params.id} />
      
      {/* Dynamic - streamed at request */}
      <Suspense fallback={<PriceSkeleton />}>
        <DynamicPrice id={params.id} />
      </Suspense>
      
      <Suspense fallback={<StockSkeleton />}>
        <LiveStock id={params.id} />
      </Suspense>
    </div>
  );
}

// Benefits:
// - Static shell loads instantly
// - Dynamic content streams in
// - Best of SSG + SSR
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# RENDERING STRATEGY PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: SSR for everything
# Bad
# All pages server-rendered
# High server costs, slow TTFB

# Good
# Marketing pages: SSG
# Product pages: ISR
# User dashboard: SSR or CSR

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Hydration mismatches
# Bad
<div>Current time: {new Date().toString()}</div>
# Server time ≠ client time = error

# Good
const [time, setTime] = useState<string>();
useEffect(() => setTime(new Date().toString()), []);
<div>Current time: {time ?? 'Loading...'}</div>

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: SSG for dynamic content
# Bad
# E-commerce prices in SSG
# User sees stale prices

# Good
# Use ISR with short revalidation
# Or SSR for real-time prices

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Not using streaming
# Bad
# One slow API blocks entire page

# Good
<Suspense fallback={<Skeleton />}>
  <SlowComponent />
</Suspense>
# Rest of page loads immediately

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: CSR for SEO-critical pages
# Bad
# Product pages are CSR
# Search engines see empty page

# Good
# Product pages are SSR/SSG/ISR
# Good SEO, fast FCP
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is the difference between SSR and SSG?"**
> "SSG generates HTML at build time - pages are pre-built and served from CDN. SSR generates HTML at request time - server renders fresh HTML for each request. SSG is faster and cheaper but data can be stale. SSR is always fresh but has server cost and TTFB."

**Q: "When would you use CSR?"**
> "CSR works for apps behind authentication where SEO doesn't matter: dashboards, admin panels, internal tools. Also for highly dynamic content where server rendering provides no benefit. Avoid for public pages that need SEO or fast initial load."

**Q: "What is hydration?"**
> "Hydration is the process of making server-rendered HTML interactive. Server sends static HTML, browser displays it immediately, then React attaches event handlers to existing DOM. Important: server and client HTML must match or hydration fails."

### Intermediate Questions

**Q: "What is ISR and when would you use it?"**
> "ISR is Incremental Static Regeneration - combines static generation with background revalidation. Pages are pre-built but regenerate after a time interval. Use for content that changes but doesn't need real-time updates: product pages, blog posts, news. On-demand ISR via webhooks for instant updates."

**Q: "How does streaming SSR improve performance?"**
> "Traditional SSR waits for all data before sending anything. Streaming sends HTML progressively - shell first, then components as they're ready. Wrapped in Suspense boundaries. Reduces TTFB and FCP. Slow components don't block fast ones."

**Q: "How do you handle different rendering strategies in one app?"**
> "Modern frameworks like Next.js support per-page strategies. Marketing pages: SSG. Product pages: ISR with 60s revalidation. User dashboard: SSR. Admin: CSR. Mixing strategies optimizes each page type."

### Advanced Questions

**Q: "What is Partial Prerendering (PPR)?"**
> "PPR (Next.js 14+) combines static shell with streaming dynamic holes. Static parts prerender at build, dynamic parts stream at request. Get SSG speed for most of page, SSR freshness where needed. Best of both worlds without choosing one strategy per page."

**Q: "How do you handle authentication with SSR/SSG?"**
> "Public content: SSG/ISR. Personalized content: SSR with auth check or client-side fetch. Pattern: SSG shell with client components for user-specific data. Or use middleware to redirect unauthenticated users before SSR runs."

**Q: "What are the tradeoffs of each rendering strategy?"**
> "CSR: Best interactivity, worst SEO/initial load. SSG: Best performance/SEO, worst freshness. SSR: Best for dynamic personalized content, highest server cost. ISR: Balance of SSG speed and freshness, but briefly stale. Choose based on: SEO needs, update frequency, personalization."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              RENDERING STRATEGY CHECKLIST                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  USE SSG WHEN:                                                  │
│  □ Content rarely changes                                      │
│  □ Same content for all users                                  │
│  □ SEO important                                               │
│  Example: Blog, docs, marketing                                │
│                                                                 │
│  USE ISR WHEN:                                                  │
│  □ Content changes periodically                                │
│  □ Can tolerate brief staleness                                │
│  □ Many pages (can't rebuild all)                              │
│  Example: E-commerce, news                                     │
│                                                                 │
│  USE SSR WHEN:                                                  │
│  □ Real-time data critical                                     │
│  □ Personalized content                                        │
│  □ SEO needed                                                  │
│  Example: Social feeds, search results                         │
│                                                                 │
│  USE CSR WHEN:                                                  │
│  □ Behind authentication                                       │
│  □ SEO not needed                                              │
│  □ Rich interactivity                                          │
│  Example: Dashboard, admin                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

COMPARISON TABLE:
┌───────────┬────────┬────────┬────────┬────────┐
│           │  CSR   │  SSR   │  SSG   │  ISR   │
├───────────┼────────┼────────┼────────┼────────┤
│ FCP       │ Slow   │ Fast   │ Fast   │ Fast   │
│ TTFB      │ Fast   │ Slow   │ Fast   │ Fast   │
│ SEO       │ Poor   │ Good   │ Good   │ Good   │
│ Freshness │ Real   │ Real   │ Stale  │ ~Fresh │
│ Cost      │ Low    │ High   │ Low    │ Low    │
└───────────┴────────┴────────┴────────┴────────┘
```

---

*Last updated: February 2026*

