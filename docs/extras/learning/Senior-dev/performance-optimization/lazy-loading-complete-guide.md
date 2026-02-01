# ⏳ Lazy Loading - Complete Guide

> A comprehensive guide to lazy loading - code splitting, dynamic imports, React.lazy, Suspense, intersection observer, and loading content on demand.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Lazy loading defers loading of non-critical resources until they're needed - splitting your bundle into chunks loaded on demand, reducing initial page load from seconds to milliseconds."

### The Lazy Loading Mental Model
```
WITHOUT LAZY LOADING (Eager):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Initial Load: Download EVERYTHING                              │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    main.js (5MB)                          │   │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐       │   │
│  │  │Home │ │About│ │Admin│ │Chart│ │PDF  │ │Modal│       │   │
│  │  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └─────┘       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  Time to Interactive: 5 seconds (download + parse 5MB)         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

WITH LAZY LOADING (Code Splitting):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Initial Load: Only what's needed for first page               │
│                                                                  │
│  ┌─────────────────┐                                           │
│  │ main.js (500KB) │  ← Fast initial load!                    │
│  │  ┌─────┐        │                                          │
│  │  │Home │        │                                          │
│  │  └─────┘        │                                          │
│  └─────────────────┘                                           │
│                                                                  │
│  Loaded on demand when needed:                                 │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐                     │
│  │About│ │Admin│ │Chart│ │PDF  │ │Modal│                     │
│  └─────┘ └─────┘ └─────┘ └─────┘ └─────┘                     │
│                                                                  │
│  Time to Interactive: 1 second                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Numbers to Remember
| Metric | Value | Context |
|--------|-------|---------|
| Ideal initial bundle | **<200KB** | For fast Time to Interactive |
| Chunk size target | **50-150KB** | Balance between requests and size |
| Above-the-fold | **Load immediately** | Critical rendering path |
| Below-the-fold | **Lazy load** | Not visible initially |
| Intersection threshold | **0.1-0.5** | Load before fully in view |

### The "Wow" Statement
> "Our React app had a 3MB bundle causing 6-second load times. I analyzed with webpack-bundle-analyzer and found chart libraries, PDF generation, and admin pages in the main bundle - stuff most users never use. Implemented route-based code splitting with React.lazy for admin routes and dynamic imports for heavy libraries. Initial bundle dropped to 400KB, load time to 1.2 seconds. Added Intersection Observer for below-the-fold images and components. Key insight: the analytics dashboard was loading chart.js for every user even though only 5% visited it."

### Key Terms to Drop
| Term | Use It Like This |
|------|------------------|
| **"Code splitting"** | "Route-based code splitting reduced our initial bundle by 70%" |
| **"Dynamic import"** | "Using dynamic import() for heavy libraries loaded on demand" |
| **"Suspense boundary"** | "Wrapped lazy components in Suspense with a skeleton fallback" |
| **"Intersection Observer"** | "Using Intersection Observer to lazy load images as they enter viewport" |
| **"Preload/prefetch"** | "Prefetching next route on hover for instant navigation" |

---

## 📚 Core Concepts

### Dynamic Imports

```typescript
// ════════════════════════════════════════════════════════════════
// DYNAMIC IMPORT BASICS
// ════════════════════════════════════════════════════════════════

// ❌ STATIC IMPORT: Always loaded in main bundle
import { generatePDF } from './pdfGenerator';
import ChartLibrary from 'heavy-chart-library';

// ✅ DYNAMIC IMPORT: Loaded only when needed
const generatePDF = async (data) => {
    const { generatePDF } = await import('./pdfGenerator');
    return generatePDF(data);
};

// Button click handler - library loaded on demand
async function handleExportPDF() {
    setLoading(true);
    const { generatePDF } = await import('./pdfGenerator');
    const pdf = await generatePDF(reportData);
    downloadPDF(pdf);
    setLoading(false);
}

// ════════════════════════════════════════════════════════════════
// WEBPACK MAGIC COMMENTS
// ════════════════════════════════════════════════════════════════

// Named chunk (for debugging/caching)
const AdminPanel = () => import(
    /* webpackChunkName: "admin" */ 
    './AdminPanel'
);

// Prefetch (load during idle time)
const Settings = () => import(
    /* webpackPrefetch: true */
    './Settings'
);

// Preload (load with current chunk)
const CriticalModule = () => import(
    /* webpackPreload: true */
    './CriticalModule'
);

// Multiple modules in one chunk
const ChartA = () => import(/* webpackChunkName: "charts" */ './ChartA');
const ChartB = () => import(/* webpackChunkName: "charts" */ './ChartB');
```

### React.lazy and Suspense

```tsx
// ════════════════════════════════════════════════════════════════
// REACT.LAZY: Code splitting for components
// ════════════════════════════════════════════════════════════════

import React, { Suspense, lazy } from 'react';

// Lazy load components
const AdminDashboard = lazy(() => import('./AdminDashboard'));
const UserProfile = lazy(() => import('./UserProfile'));
const Analytics = lazy(() => import('./Analytics'));

// Loading fallback component
function LoadingSpinner() {
    return (
        <div className="loading-container">
            <div className="spinner" />
            <p>Loading...</p>
        </div>
    );
}

// Skeleton loader (better UX)
function DashboardSkeleton() {
    return (
        <div className="dashboard-skeleton">
            <div className="skeleton-header" />
            <div className="skeleton-card" />
            <div className="skeleton-card" />
        </div>
    );
}

// App with Suspense boundaries
function App() {
    return (
        <Router>
            <Suspense fallback={<LoadingSpinner />}>
                <Routes>
                    <Route path="/" element={<Home />} />
                    
                    {/* Lazy loaded routes */}
                    <Route 
                        path="/admin" 
                        element={
                            <Suspense fallback={<DashboardSkeleton />}>
                                <AdminDashboard />
                            </Suspense>
                        } 
                    />
                    <Route path="/profile" element={<UserProfile />} />
                    <Route path="/analytics" element={<Analytics />} />
                </Routes>
            </Suspense>
        </Router>
    );
}

// ════════════════════════════════════════════════════════════════
// ERROR BOUNDARY FOR LAZY COMPONENTS
// ════════════════════════════════════════════════════════════════

class LazyLoadErrorBoundary extends React.Component {
    state = { hasError: false };
    
    static getDerivedStateFromError(error) {
        return { hasError: true };
    }
    
    render() {
        if (this.state.hasError) {
            return (
                <div>
                    <p>Failed to load component.</p>
                    <button onClick={() => window.location.reload()}>
                        Retry
                    </button>
                </div>
            );
        }
        return this.props.children;
    }
}

// Usage
<LazyLoadErrorBoundary>
    <Suspense fallback={<Loading />}>
        <LazyComponent />
    </Suspense>
</LazyLoadErrorBoundary>
```

### Intersection Observer

```tsx
// ════════════════════════════════════════════════════════════════
// INTERSECTION OBSERVER: Load when visible
// ════════════════════════════════════════════════════════════════

import { useEffect, useRef, useState } from 'react';

// Custom hook for lazy loading
function useLazyLoad(options = {}) {
    const [isVisible, setIsVisible] = useState(false);
    const [hasLoaded, setHasLoaded] = useState(false);
    const ref = useRef<HTMLDivElement>(null);
    
    useEffect(() => {
        const observer = new IntersectionObserver(
            ([entry]) => {
                if (entry.isIntersecting && !hasLoaded) {
                    setIsVisible(true);
                    setHasLoaded(true);
                    observer.disconnect(); // Only load once
                }
            },
            {
                rootMargin: '100px', // Load 100px before visible
                threshold: 0.1,
                ...options
            }
        );
        
        if (ref.current) {
            observer.observe(ref.current);
        }
        
        return () => observer.disconnect();
    }, [hasLoaded, options]);
    
    return { ref, isVisible };
}

// Lazy loaded component
function LazySection({ children }: { children: React.ReactNode }) {
    const { ref, isVisible } = useLazyLoad();
    
    return (
        <div ref={ref} style={{ minHeight: '200px' }}>
            {isVisible ? children : <div className="placeholder" />}
        </div>
    );
}

// Usage
function HomePage() {
    return (
        <>
            <HeroSection /> {/* Always loaded */}
            
            <LazySection>
                <FeaturesSection /> {/* Loaded when scrolled to */}
            </LazySection>
            
            <LazySection>
                <TestimonialsSection />
            </LazySection>
            
            <LazySection>
                <ContactForm />
            </LazySection>
        </>
    );
}

// ════════════════════════════════════════════════════════════════
// LAZY LOADING IMAGES
// ════════════════════════════════════════════════════════════════

function LazyImage({ src, alt, ...props }) {
    const [isLoaded, setIsLoaded] = useState(false);
    const { ref, isVisible } = useLazyLoad();
    
    return (
        <div ref={ref} className="image-container">
            {isVisible ? (
                <>
                    {!isLoaded && <div className="image-skeleton" />}
                    <img
                        src={src}
                        alt={alt}
                        onLoad={() => setIsLoaded(true)}
                        style={{ opacity: isLoaded ? 1 : 0 }}
                        {...props}
                    />
                </>
            ) : (
                <div className="image-placeholder" />
            )}
        </div>
    );
}

// Native lazy loading (modern browsers)
<img src="image.jpg" alt="Description" loading="lazy" />
```

### Route-Based Code Splitting

```tsx
// ════════════════════════════════════════════════════════════════
// NEXT.JS: Automatic route-based splitting
// ════════════════════════════════════════════════════════════════

// pages/admin/dashboard.tsx → Only loaded when visiting /admin/dashboard
// Each page is automatically a separate chunk

// Dynamic import for components within pages
import dynamic from 'next/dynamic';

const HeavyChart = dynamic(
    () => import('../components/HeavyChart'),
    {
        loading: () => <ChartSkeleton />,
        ssr: false // Don't render on server
    }
);

// ════════════════════════════════════════════════════════════════
// REACT ROUTER: Manual route splitting
// ════════════════════════════════════════════════════════════════

import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// Lazy load all routes
const routes = [
    { path: '/', component: lazy(() => import('./pages/Home')) },
    { path: '/products', component: lazy(() => import('./pages/Products')) },
    { path: '/product/:id', component: lazy(() => import('./pages/ProductDetail')) },
    { path: '/admin/*', component: lazy(() => import('./pages/Admin')) },
    { path: '/checkout', component: lazy(() => import('./pages/Checkout')) },
];

function App() {
    return (
        <Suspense fallback={<PageLoader />}>
            <Routes>
                {routes.map(({ path, component: Component }) => (
                    <Route key={path} path={path} element={<Component />} />
                ))}
            </Routes>
        </Suspense>
    );
}

// ════════════════════════════════════════════════════════════════
// PREFETCHING ON HOVER
// ════════════════════════════════════════════════════════════════

function NavLink({ to, children }) {
    const prefetchRoute = () => {
        // Trigger dynamic import on hover
        if (to === '/admin') {
            import('./pages/Admin');
        }
    };
    
    return (
        <Link 
            to={to} 
            onMouseEnter={prefetchRoute}
            onFocus={prefetchRoute}
        >
            {children}
        </Link>
    );
}
```

---

## Common Pitfalls

```
LAZY LOADING PITFALLS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. TOO MANY SMALL CHUNKS                                       │
│     Problem: Many HTTP requests hurt performance               │
│     Solution: Group related modules, target 50-150KB chunks    │
│                                                                  │
│  2. NO LOADING STATE                                            │
│     Problem: UI flashes/shifts when chunk loads                │
│     Solution: Suspense fallback with skeleton matching layout  │
│                                                                  │
│  3. LAZY LOADING CRITICAL PATH                                  │
│     Problem: Above-the-fold content is lazy loaded             │
│     Solution: Only lazy load below-the-fold and routes         │
│                                                                  │
│  4. NOT HANDLING ERRORS                                         │
│     Problem: Network failure breaks app                        │
│     Solution: Error boundary with retry option                 │
│                                                                  │
│  5. NO PREFETCHING                                              │
│     Problem: User waits on navigation                          │
│     Solution: Prefetch on hover/focus                          │
│                                                                  │
│  6. LAYOUT SHIFT                                                │
│     Problem: Content jumps when lazy content loads             │
│     Solution: Reserve space with min-height/aspect-ratio       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "How do you implement code splitting in React?"**
> "Three main approaches: 1) Route-based with React.lazy - each route is a separate chunk, loaded with Suspense. 2) Component-based - lazy load heavy components like charts or modals. 3) Library-based - dynamic import() for heavy libraries only used in certain features. I analyze with webpack-bundle-analyzer to find good split points."

**Q: "What's the difference between preload and prefetch?"**
> "Preload loads resources needed for current page with high priority - use for critical resources. Prefetch loads resources that might be needed for future navigation with low priority during idle time - use for next likely route. In webpack: webpackPreload for current, webpackPrefetch for future."

**Q: "How do you lazy load images?"**
> "Native approach: `loading='lazy'` attribute (modern browsers). Custom approach: Intersection Observer to load when image enters viewport. Best practice: use placeholder/blur hash for layout stability, load slightly before visible (rootMargin), and use srcset for responsive images."

---

## Quick Reference

```
LAZY LOADING CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  CODE SPLITTING:                                                │
│  • React.lazy(() => import('./Component'))                     │
│  • Wrap in <Suspense fallback={...}>                          │
│  • Use Error Boundary for failures                             │
│                                                                  │
│  DYNAMIC IMPORT:                                                │
│  • const module = await import('./module')                     │
│  • webpackChunkName for naming                                 │
│  • webpackPrefetch/Preload for loading hints                   │
│                                                                  │
│  INTERSECTION OBSERVER:                                         │
│  • Load when element enters viewport                           │
│  • rootMargin for early loading                                │
│  • Reserve space to prevent layout shift                       │
│                                                                  │
│  WHAT TO LAZY LOAD:                                             │
│  ✓ Routes (especially admin, settings)                         │
│  ✓ Heavy libraries (charts, PDF, editors)                      │
│  ✓ Modals and dialogs                                          │
│  ✓ Below-the-fold content                                      │
│  ✗ Above-the-fold critical content                             │
│  ✗ Small frequently-used components                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


