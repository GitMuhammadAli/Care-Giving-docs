# 📊 Core Web Vitals - Complete Guide

> A comprehensive guide to Core Web Vitals - LCP, FID/INP, CLS, performance budgets, measuring, and improving scores for better user experience and SEO.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Core Web Vitals are Google's metrics for user experience: LCP measures loading (largest content visible), INP measures interactivity (response to clicks), and CLS measures visual stability (layout shifts) - they directly impact SEO rankings."

### The Core Web Vitals Model
```
CORE WEB VITALS OVERVIEW:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  LCP - Largest Contentful Paint                                │
│  "How fast does the main content appear?"                      │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │  GOOD        │  NEEDS IMPROVEMENT  │  POOR              │  │
│  │  ≤2.5s       │  2.5s - 4.0s        │  >4.0s             │  │
│  │  🟢          │  🟡                 │  🔴                │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│  INP - Interaction to Next Paint (replaced FID)               │
│  "How fast does the page respond to user input?"              │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │  GOOD        │  NEEDS IMPROVEMENT  │  POOR              │  │
│  │  ≤200ms      │  200ms - 500ms      │  >500ms            │  │
│  │  🟢          │  🟡                 │  🔴                │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                                  │
│  CLS - Cumulative Layout Shift                                 │
│  "How much does content jump around?"                          │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │  GOOD        │  NEEDS IMPROVEMENT  │  POOR              │  │
│  │  ≤0.1        │  0.1 - 0.25         │  >0.25             │  │
│  │  🟢          │  🟡                 │  🔴                │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Numbers to Remember
| Metric | Good | Poor | What It Measures |
|--------|------|------|------------------|
| **LCP** | ≤2.5s | >4.0s | Loading - largest visible element |
| **INP** | ≤200ms | >500ms | Interactivity - input response |
| **CLS** | ≤0.1 | >0.25 | Stability - layout shifts |
| **FCP** | ≤1.8s | >3.0s | First content visible |
| **TTFB** | ≤800ms | >1.8s | Server response time |
| **TBT** | ≤200ms | >600ms | Main thread blocking |

### The "Wow" Statement
> "Our e-commerce site had poor Core Web Vitals - LCP 5.2s, CLS 0.35, INP 450ms - killing our SEO and conversions. I systematically addressed each: For LCP, I preloaded the hero image, inlined critical CSS, and added priority hints. LCP dropped to 2.1s. For CLS, I added explicit dimensions to images and reserved space for ads. CLS dropped to 0.05. For INP, I moved heavy JavaScript to web workers and implemented virtualization for the product list. INP dropped to 120ms. After these fixes, we moved from page 2 to top 3 Google results for key terms, and conversion rate increased 15%."

### Key Terms to Drop
| Term | Use It Like This |
|------|------------------|
| **"LCP element"** | "The LCP element was our hero image - needed to preload it" |
| **"Layout shift"** | "Images without dimensions cause layout shifts when they load" |
| **"Long tasks"** | "Breaking up long tasks improved INP by letting browser respond faster" |
| **"75th percentile"** | "Core Web Vitals are measured at 75th percentile of real users" |
| **"Field data vs lab data"** | "PageSpeed shows lab data, but Google ranks on field data from CrUX" |

---

## 📚 Core Concepts

### LCP - Largest Contentful Paint

```
LCP ELEMENT CANDIDATES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  What counts as LCP:                                           │
│  • <img> elements                                              │
│  • <image> inside SVG                                          │
│  • <video> poster image                                        │
│  • Element with background-image                               │
│  • Block-level text elements (<h1>, <p>, etc.)                 │
│                                                                  │
│  COMMON LCP ELEMENTS:                                           │
│  • Hero image/banner                                           │
│  • Featured product image                                      │
│  • Main headline text                                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```tsx
// ════════════════════════════════════════════════════════════════
// IMPROVING LCP
// ════════════════════════════════════════════════════════════════

// 1. PRELOAD LCP IMAGE
// In <head>
<link 
    rel="preload" 
    as="image" 
    href="/hero.webp"
    imagesrcset="/hero-400.webp 400w, /hero-800.webp 800w"
    imagesizes="100vw"
/>

// 2. PRIORITY HINTS (Fetch Priority API)
<img 
    src="/hero.jpg" 
    fetchpriority="high"  // Tell browser this is important
    alt="Hero"
/>

// Low priority for below-fold images
<img 
    src="/footer-logo.jpg" 
    fetchpriority="low"
    loading="lazy"
    alt="Logo"
/>

// 3. INLINE CRITICAL CSS
// Extract CSS needed for above-the-fold and inline it
<style>
    /* Critical CSS inlined */
    .hero { background: #f0f0f0; height: 400px; }
    .hero-title { font-size: 2rem; }
</style>

// Load rest of CSS async
<link rel="preload" href="/styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'" />
<noscript><link rel="stylesheet" href="/styles.css" /></noscript>

// 4. OPTIMIZE SERVER RESPONSE (TTFB)
// - Use CDN for static assets
// - Enable caching headers
// - Use streaming SSR
// - Database query optimization

// 5. NEXT.JS: Priority for LCP image
import Image from 'next/image';

<Image
    src="/hero.jpg"
    alt="Hero"
    width={1200}
    height={600}
    priority  // Disables lazy loading, preloads
/>
```

### INP - Interaction to Next Paint

```tsx
// ════════════════════════════════════════════════════════════════
// IMPROVING INP (Interaction to Next Paint)
// ════════════════════════════════════════════════════════════════

// INP measures WORST interaction response during page lifecycle
// Every click, tap, keypress is measured

// 1. BREAK UP LONG TASKS
// ❌ BAD: Blocks main thread
function handleClick() {
    // 500ms of work - blocks UI
    for (let i = 0; i < 10000000; i++) {
        process(data[i]);
    }
}

// ✅ GOOD: Yield to main thread
async function handleClick() {
    for (let i = 0; i < 10000000; i++) {
        process(data[i]);
        
        // Yield every 100 items
        if (i % 100 === 0) {
            await scheduler.yield?.() ?? new Promise(r => setTimeout(r, 0));
        }
    }
}

// 2. USE WEB WORKERS FOR HEAVY COMPUTATION
// worker.js
self.onmessage = function(e) {
    const result = heavyComputation(e.data);
    self.postMessage(result);
};

// main.js
const worker = new Worker('worker.js');

function handleClick() {
    // Instant response - work happens in background
    worker.postMessage(data);
    worker.onmessage = (e) => updateUI(e.data);
}

// 3. VIRTUALIZE LONG LISTS
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualizedList({ items }) {
    const parentRef = useRef(null);
    
    const virtualizer = useVirtualizer({
        count: items.length,
        getScrollElement: () => parentRef.current,
        estimateSize: () => 50,
    });
    
    return (
        <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
            <div style={{ height: virtualizer.getTotalSize() }}>
                {virtualizer.getVirtualItems().map((virtualItem) => (
                    <div
                        key={virtualItem.key}
                        style={{
                            position: 'absolute',
                            top: virtualItem.start,
                            height: virtualItem.size,
                        }}
                    >
                        {items[virtualItem.index].name}
                    </div>
                ))}
            </div>
        </div>
    );
}

// 4. DEBOUNCE/THROTTLE USER INPUT
function SearchInput() {
    const [query, setQuery] = useState('');
    
    // Debounce search to avoid processing every keystroke
    const debouncedSearch = useMemo(
        () => debounce((q) => performSearch(q), 300),
        []
    );
    
    return (
        <input
            value={query}
            onChange={(e) => {
                setQuery(e.target.value);
                debouncedSearch(e.target.value);
            }}
        />
    );
}
```

### CLS - Cumulative Layout Shift

```tsx
// ════════════════════════════════════════════════════════════════
// PREVENTING CLS (Layout Shifts)
// ════════════════════════════════════════════════════════════════

// 1. ALWAYS SET IMAGE DIMENSIONS
// ❌ BAD: No dimensions - shifts when image loads
<img src="photo.jpg" alt="Photo" />

// ✅ GOOD: Explicit dimensions
<img src="photo.jpg" alt="Photo" width="800" height="600" />

// ✅ GOOD: CSS aspect-ratio
<img 
    src="photo.jpg" 
    alt="Photo"
    style={{ aspectRatio: '16/9', width: '100%', height: 'auto' }}
/>

// 2. RESERVE SPACE FOR DYNAMIC CONTENT
// ❌ BAD: Ad loads and pushes content down
<div id="ad-container"></div>

// ✅ GOOD: Reserve space
<div 
    id="ad-container" 
    style={{ minHeight: '250px', background: '#f0f0f0' }}
></div>

// 3. AVOID INSERTING CONTENT ABOVE EXISTING CONTENT
// ❌ BAD: Banner appears at top, pushes everything down
function Page() {
    const [showBanner, setShowBanner] = useState(false);
    
    useEffect(() => {
        checkPromotion().then(setShowBanner);
    }, []);
    
    return (
        <>
            {showBanner && <PromoBanner />}  {/* Shifts content! */}
            <MainContent />
        </>
    );
}

// ✅ GOOD: Reserve space or use transform
function Page() {
    const [showBanner, setShowBanner] = useState(false);
    
    return (
        <>
            <div style={{ minHeight: showBanner ? 'auto' : '60px' }}>
                {showBanner && <PromoBanner />}
            </div>
            <MainContent />
        </>
    );
}

// 4. USE CSS TRANSFORM FOR ANIMATIONS
// ❌ BAD: Changing height/width causes layout shift
.accordion-content {
    height: 0;
    transition: height 0.3s;
}
.accordion-content.open {
    height: 200px;  /* Layout shift! */
}

// ✅ GOOD: Use transform (doesn't trigger layout)
.accordion-content {
    transform: scaleY(0);
    transform-origin: top;
    transition: transform 0.3s;
}
.accordion-content.open {
    transform: scaleY(1);
}

// 5. FONT LOADING
// ❌ BAD: FOUT (Flash of Unstyled Text)
@font-face {
    font-family: 'CustomFont';
    src: url('font.woff2') format('woff2');
}

// ✅ GOOD: Font display swap with fallback matching
@font-face {
    font-family: 'CustomFont';
    src: url('font.woff2') format('woff2');
    font-display: swap;  /* Show fallback immediately */
}

// Match fallback metrics to reduce shift
body {
    font-family: 'CustomFont', Arial, sans-serif;
    /* Adjust fallback to match custom font metrics */
}
```

### Measuring Core Web Vitals

```typescript
// ════════════════════════════════════════════════════════════════
// MEASURING IN CODE
// ════════════════════════════════════════════════════════════════

import { onLCP, onINP, onCLS, onFCP, onTTFB } from 'web-vitals';

// Report to analytics
function sendToAnalytics(metric) {
    console.log(metric.name, metric.value);
    
    // Send to your analytics
    fetch('/api/metrics', {
        method: 'POST',
        body: JSON.stringify({
            name: metric.name,
            value: metric.value,
            id: metric.id,
            page: window.location.pathname
        })
    });
}

// Track all Core Web Vitals
onLCP(sendToAnalytics);
onINP(sendToAnalytics);
onCLS(sendToAnalytics);
onFCP(sendToAnalytics);
onTTFB(sendToAnalytics);

// ════════════════════════════════════════════════════════════════
// NEXT.JS BUILT-IN REPORTING
// ════════════════════════════════════════════════════════════════

// pages/_app.js
export function reportWebVitals(metric) {
    console.log(metric);
    
    // Send to analytics
    if (metric.label === 'web-vital') {
        analytics.track('Web Vital', {
            name: metric.name,
            value: metric.value,
        });
    }
}
```

---

## Common Pitfalls

```
CORE WEB VITALS PITFALLS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. OPTIMIZING LAB DATA ONLY                                    │
│     Problem: Lab looks good, field data is poor                │
│     Solution: Monitor real user metrics (CrUX, RUM)            │
│                                                                  │
│  2. LAZY LOADING LCP IMAGE                                      │
│     Problem: Hero loads late, kills LCP                        │
│     Solution: priority/eager for above-fold images             │
│                                                                  │
│  3. RENDER-BLOCKING JS                                          │
│     Problem: Scripts block first paint                         │
│     Solution: async/defer, move to end, code split             │
│                                                                  │
│  4. NO IMAGE DIMENSIONS                                         │
│     Problem: Images cause layout shifts                        │
│     Solution: Always set width/height or aspect-ratio          │
│                                                                  │
│  5. THIRD-PARTY SCRIPTS                                         │
│     Problem: Ads, analytics block main thread                  │
│     Solution: Load async, defer non-critical                   │
│                                                                  │
│  6. SYNCHRONOUS EVENT HANDLERS                                  │
│     Problem: Long handlers block INP                           │
│     Solution: Yield to main thread, use workers                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "What are Core Web Vitals and why do they matter?"**
> "Three metrics measuring user experience: LCP (loading - largest content in 2.5s), INP (interactivity - respond to input in 200ms), CLS (stability - minimal layout shifts, under 0.1). They matter because Google uses them for search ranking, and they directly correlate with user engagement and conversion rates."

**Q: "How do you improve LCP?"**
> "Identify the LCP element first (usually hero image or headline). Then: 1) Preload critical resources with `<link rel='preload'>`. 2) Use fetchpriority='high' on LCP image. 3) Inline critical CSS. 4) Optimize images (WebP/AVIF, right size). 5) Reduce server response time (CDN, caching). 6) Eliminate render-blocking resources."

**Q: "What causes CLS and how do you fix it?"**
> "CLS happens when content shifts unexpectedly. Common causes: images without dimensions, ads loading, fonts loading, dynamic content inserted above. Fixes: always set image dimensions, reserve space for dynamic content, use font-display: swap with matched fallback, avoid inserting content above existing content."

---

## Quick Reference

```
CORE WEB VITALS CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  TARGETS (Good):                                                │
│  • LCP: ≤2.5s (loading)                                        │
│  • INP: ≤200ms (interactivity)                                 │
│  • CLS: ≤0.1 (stability)                                       │
│                                                                  │
│  LCP FIXES:                                                     │
│  • Preload hero image                                          │
│  • fetchpriority="high"                                        │
│  • Inline critical CSS                                         │
│  • Optimize images                                             │
│                                                                  │
│  INP FIXES:                                                     │
│  • Break long tasks (yield)                                    │
│  • Use web workers                                             │
│  • Virtualize lists                                            │
│  • Debounce input handlers                                     │
│                                                                  │
│  CLS FIXES:                                                     │
│  • Set image dimensions                                        │
│  • Reserve space for ads/dynamic                               │
│  • font-display: swap                                          │
│  • Avoid inserting content above                               │
│                                                                  │
│  MEASURING:                                                     │
│  • web-vitals library                                          │
│  • PageSpeed Insights                                          │
│  • Chrome DevTools Performance                                 │
│  • CrUX (real user data)                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


