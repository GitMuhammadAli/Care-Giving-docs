# 📦 Bundle Optimization - Complete Guide

> A comprehensive guide to frontend bundle optimization - tree shaking, code splitting, lazy loading, and reducing bundle size for faster applications.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Bundle optimization reduces JavaScript shipped to users through tree shaking (removing unused code), code splitting (loading code on demand), and lazy loading (deferring non-critical resources) to improve initial load time and runtime performance."

### The 7 Key Concepts (Remember These!)
```
1. TREE SHAKING     → Remove unused exports (dead code elimination)
2. CODE SPLITTING   → Break bundle into chunks loaded on demand
3. LAZY LOADING     → Load resources when needed (routes, components)
4. MINIFICATION     → Remove whitespace, shorten names
5. COMPRESSION      → Gzip/Brotli encoding
6. CHUNKING         → Group modules into logical bundles
7. CACHING          → Content-hash filenames for long-term cache
```

### Bundle Size Impact
```
┌─────────────────────────────────────────────────────────────────┐
│              BUNDLE SIZE IMPACT                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SIZE vs LOAD TIME (3G connection):                            │
│  ─────────────────────────────────                              │
│  100KB  →  ~0.6s download                                      │
│  200KB  →  ~1.3s download                                      │
│  500KB  →  ~3.2s download                                      │
│  1MB    →  ~6.5s download                                      │
│                                                                 │
│  TYPICAL BUNDLE SIZES:                                         │
│  ─────────────────────                                          │
│  React          →  ~42KB  (gzipped)                            │
│  React DOM      →  ~40KB  (gzipped)                            │
│  Lodash (full)  →  ~70KB  (gzipped)                            │
│  Lodash (es)    →  Tree-shakeable                              │
│  Moment.js      →  ~67KB  (gzipped) - use date-fns/dayjs       │
│  date-fns       →  ~6KB   (only what you use)                  │
│                                                                 │
│  TARGETS:                                                      │
│  ────────                                                       │
│  Initial JS:    < 100KB (gzipped)                              │
│  Initial CSS:   < 50KB  (gzipped)                              │
│  Total:         < 200KB (gzipped)                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Optimization Techniques
```
┌─────────────────────────────────────────────────────────────────┐
│              OPTIMIZATION TECHNIQUES                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BUILD TIME:                                                   │
│  ───────────                                                    │
│  • Tree shaking (remove unused exports)                        │
│  • Minification (terser, esbuild)                              │
│  • Dead code elimination                                       │
│  • Scope hoisting                                              │
│                                                                 │
│  TRANSFER:                                                     │
│  ─────────                                                      │
│  • Gzip compression (~70% reduction)                           │
│  • Brotli compression (~80% reduction)                         │
│  • CDN delivery                                                │
│                                                                 │
│  RUNTIME:                                                      │
│  ────────                                                       │
│  • Code splitting (dynamic imports)                            │
│  • Route-based splitting                                       │
│  • Component lazy loading                                      │
│  • Preloading critical chunks                                  │
│                                                                 │
│  CACHING:                                                      │
│  ────────                                                       │
│  • Content-hash filenames                                      │
│  • Vendor chunk separation                                     │
│  • Long-term browser caching                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Tree shaking"** | "We use ES modules for tree shaking dead code" |
| **"Code splitting"** | "Routes are code split for smaller initial bundle" |
| **"Vendor chunk"** | "Vendor chunk is cached separately from app code" |
| **"Side effects"** | "Package marked sideEffects: false enables tree shaking" |
| **"Dynamic import"** | "We use dynamic imports for lazy loading" |
| **"Bundle analyzer"** | "Bundle analyzer showed lodash was 30% of our bundle" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Initial JS target | **< 100KB** | gzipped |
| Gzip reduction | **~70%** | Typical compression |
| Brotli reduction | **~80%** | Better than gzip |
| Parse time | **~1ms/KB** | Mobile parse cost |

### The "Wow" Statement (Memorize This!)
> "Our initial bundle is under 100KB gzipped. We code split by route using React.lazy, so users only download code for the current page. Vendor dependencies are in a separate chunk with content-hash for long-term caching - React updates don't invalidate vendor cache. We analyzed the bundle and found moment.js was 70KB - replaced with date-fns which tree-shakes to 6KB. All imports are ES modules for tree shaking. Heavy components like charts load lazily with Suspense. We preload next route on link hover. The result: TTI dropped from 4s to 1.5s. We monitor bundle size in CI - PRs that increase it significantly require justification."

---

## 📚 Table of Contents

1. [Tree Shaking](#1-tree-shaking)
2. [Code Splitting](#2-code-splitting)
3. [Lazy Loading](#3-lazy-loading)
4. [Chunking Strategies](#4-chunking-strategies)
5. [Analysis & Monitoring](#5-analysis--monitoring)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Tree Shaking

```typescript
// ════════════════════════════════════════════════════════════════
// TREE SHAKING BASICS
// ════════════════════════════════════════════════════════════════

// Tree shaking removes unused exports from the final bundle
// REQUIRES: ES modules (import/export), not CommonJS (require)

// ❌ BAD: CommonJS - Can't tree shake
const _ = require('lodash');
_.map(array, fn); // Entire lodash included (~70KB)

// ✅ GOOD: ES modules - Tree shakeable
import { map } from 'lodash-es';
map(array, fn); // Only `map` included (~2KB)

// ════════════════════════════════════════════════════════════════
// NAMED EXPORTS vs DEFAULT EXPORTS
// ════════════════════════════════════════════════════════════════

// Named exports are better for tree shaking
// utils.ts
export function formatDate(date: Date) { /* ... */ }
export function formatCurrency(amount: number) { /* ... */ }
export function formatNumber(num: number) { /* ... */ }

// Consumer - only formatDate is bundled
import { formatDate } from './utils';

// ════════════════════════════════════════════════════════════════
// BARREL FILES (Index re-exports)
// ════════════════════════════════════════════════════════════════

// ⚠️ CAUTION: Barrel files can break tree shaking

// components/index.ts (barrel)
export { Button } from './Button';
export { Input } from './Input';
export { Modal } from './Modal';
export { DatePicker } from './DatePicker'; // Heavy component

// ❌ This might include ALL components depending on bundler
import { Button } from './components';

// ✅ Direct imports are safer
import { Button } from './components/Button';

// Modern bundlers handle this better, but verify with analyzer

// ════════════════════════════════════════════════════════════════
// SIDE EFFECTS
// ════════════════════════════════════════════════════════════════

// Side effects prevent tree shaking
// A module with side effects runs code on import

// ❌ Has side effects - can't be tree shaken
// analytics.ts
import { trackPageView } from './tracker';
trackPageView(); // Runs on import!
export function trackEvent() { /* ... */ }

// ✅ No side effects - tree shakeable
// analytics.ts
export function trackPageView() { /* ... */ }
export function trackEvent() { /* ... */ }
// Consumer calls trackPageView() explicitly

// package.json - declare no side effects
{
  "name": "my-library",
  "sideEffects": false // All files are pure
}

// Or specify files with side effects
{
  "sideEffects": [
    "*.css",
    "./src/polyfills.js"
  ]
}

// ════════════════════════════════════════════════════════════════
// LIBRARY BEST PRACTICES
// ════════════════════════════════════════════════════════════════

// Choose tree-shakeable libraries
// ❌ Avoid
import moment from 'moment'; // 67KB, not tree-shakeable
import _ from 'lodash'; // 70KB, not tree-shakeable

// ✅ Use
import { format } from 'date-fns'; // Only what you use
import { map, filter } from 'lodash-es'; // ES modules version
// Or write your own for simple utilities

// Check if library is tree-shakeable:
// 1. Uses ES modules (import/export)
// 2. package.json has "module" or "exports" field
// 3. sideEffects: false in package.json
```

---

## 2. Code Splitting

```tsx
// ════════════════════════════════════════════════════════════════
// ROUTE-BASED CODE SPLITTING
// ════════════════════════════════════════════════════════════════

import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// Lazy load route components
const Home = lazy(() => import('./pages/Home'));
const Products = lazy(() => import('./pages/Products'));
const ProductDetail = lazy(() => import('./pages/ProductDetail'));
const Cart = lazy(() => import('./pages/Cart'));
const Checkout = lazy(() => import('./pages/Checkout'));
const Admin = lazy(() => import('./pages/Admin'));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/products" element={<Products />} />
        <Route path="/products/:id" element={<ProductDetail />} />
        <Route path="/cart" element={<Cart />} />
        <Route path="/checkout" element={<Checkout />} />
        <Route path="/admin/*" element={<Admin />} />
      </Routes>
    </Suspense>
  );
}

// ════════════════════════════════════════════════════════════════
// NAMED CHUNKS
// ════════════════════════════════════════════════════════════════

// Add webpack magic comments for better chunk names
const Home = lazy(() => import(/* webpackChunkName: "home" */ './pages/Home'));
const Admin = lazy(() => import(/* webpackChunkName: "admin" */ './pages/Admin'));

// Vite uses file name by default, or:
const Admin = lazy(() => import('./pages/Admin')); // admin.js chunk

// ════════════════════════════════════════════════════════════════
// PRELOADING
// ════════════════════════════════════════════════════════════════

// Preload on hover for faster navigation
const ProductDetail = lazy(() => import('./pages/ProductDetail'));

function ProductCard({ product }: { product: Product }) {
  const preloadProductDetail = () => {
    // Triggers the lazy import
    import('./pages/ProductDetail');
  };

  return (
    <Link
      to={`/products/${product.id}`}
      onMouseEnter={preloadProductDetail}
      onFocus={preloadProductDetail}
    >
      {product.name}
    </Link>
  );
}

// Or use preload hint in HTML
// <link rel="prefetch" href="/static/js/product-detail.chunk.js">

// Preload function helper
function preloadComponent(importFn: () => Promise<any>) {
  return importFn();
}

// Usage
<Link
  onMouseEnter={() => preloadComponent(() => import('./pages/Checkout'))}
>
  Checkout
</Link>

// ════════════════════════════════════════════════════════════════
// COMPONENT-LEVEL SPLITTING
// ════════════════════════════════════════════════════════════════

// Split heavy components
const HeavyChart = lazy(() => import('./components/HeavyChart'));
const RichTextEditor = lazy(() => import('./components/RichTextEditor'));
const DataGrid = lazy(() => import('./components/DataGrid'));

function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* Chart loads lazily */}
      <Suspense fallback={<ChartSkeleton />}>
        <HeavyChart data={chartData} />
      </Suspense>
      
      {/* Editor loads only when needed */}
      {showEditor && (
        <Suspense fallback={<EditorSkeleton />}>
          <RichTextEditor />
        </Suspense>
      )}
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// DYNAMIC IMPORTS IN EVENT HANDLERS
// ════════════════════════════════════════════════════════════════

async function handleExport() {
  // Load xlsx library only when user clicks export
  const XLSX = await import('xlsx');
  
  const workbook = XLSX.utils.book_new();
  const worksheet = XLSX.utils.json_to_sheet(data);
  XLSX.utils.book_append_sheet(workbook, worksheet, 'Data');
  XLSX.writeFile(workbook, 'export.xlsx');
}

async function handleShare() {
  // Load share functionality on demand
  const { shareToSocial } = await import('./utils/sharing');
  shareToSocial(data);
}
```

---

## 3. Lazy Loading

```tsx
// ════════════════════════════════════════════════════════════════
// LAZY LOADING IMAGES
// ════════════════════════════════════════════════════════════════

// Native lazy loading
function ProductImage({ src, alt }: { src: string; alt: string }) {
  return (
    <img
      src={src}
      alt={alt}
      loading="lazy" // Native browser lazy loading
      decoding="async" // Don't block main thread
    />
  );
}

// With placeholder/blur
function OptimizedImage({ src, alt, placeholder }: ImageProps) {
  const [isLoaded, setIsLoaded] = useState(false);

  return (
    <div className="image-container">
      {!isLoaded && (
        <img
          src={placeholder}
          alt=""
          className="image-placeholder"
          aria-hidden="true"
        />
      )}
      <img
        src={src}
        alt={alt}
        loading="lazy"
        onLoad={() => setIsLoaded(true)}
        className={`image ${isLoaded ? 'loaded' : ''}`}
      />
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// INTERSECTION OBSERVER LAZY LOADING
// ════════════════════════════════════════════════════════════════

function useLazyLoad<T extends HTMLElement>() {
  const ref = useRef<T>(null);
  const [isVisible, setIsVisible] = useState(false);

  useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.disconnect();
        }
      },
      {
        rootMargin: '100px', // Start loading 100px before visible
        threshold: 0,
      }
    );

    observer.observe(element);

    return () => observer.disconnect();
  }, []);

  return { ref, isVisible };
}

// Usage
function LazyComponent({ children }: { children: React.ReactNode }) {
  const { ref, isVisible } = useLazyLoad<HTMLDivElement>();

  return (
    <div ref={ref}>
      {isVisible ? children : <Placeholder />}
    </div>
  );
}

// Lazy load heavy component
function LazyChart({ data }: { data: ChartData }) {
  const { ref, isVisible } = useLazyLoad<HTMLDivElement>();

  return (
    <div ref={ref} style={{ minHeight: 300 }}>
      {isVisible && (
        <Suspense fallback={<ChartSkeleton />}>
          <HeavyChart data={data} />
        </Suspense>
      )}
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// LAZY LOADING THIRD-PARTY SCRIPTS
// ════════════════════════════════════════════════════════════════

function useScript(src: string, options: { defer?: boolean } = {}) {
  const [status, setStatus] = useState<'loading' | 'ready' | 'error'>('loading');

  useEffect(() => {
    const script = document.createElement('script');
    script.src = src;
    script.async = true;
    if (options.defer) script.defer = true;

    script.onload = () => setStatus('ready');
    script.onerror = () => setStatus('error');

    document.body.appendChild(script);

    return () => {
      document.body.removeChild(script);
    };
  }, [src, options.defer]);

  return status;
}

// Load analytics only after interaction
function useDelayedAnalytics() {
  const [shouldLoad, setShouldLoad] = useState(false);

  useEffect(() => {
    // Load after idle or interaction
    const timeout = setTimeout(() => setShouldLoad(true), 3000);
    
    const handleInteraction = () => {
      setShouldLoad(true);
      clearTimeout(timeout);
    };

    window.addEventListener('scroll', handleInteraction, { once: true });
    window.addEventListener('click', handleInteraction, { once: true });

    return () => {
      clearTimeout(timeout);
      window.removeEventListener('scroll', handleInteraction);
      window.removeEventListener('click', handleInteraction);
    };
  }, []);

  const status = useScript(
    shouldLoad ? 'https://www.google-analytics.com/analytics.js' : ''
  );

  return status;
}
```

---

## 4. Chunking Strategies

```typescript
// ════════════════════════════════════════════════════════════════
// VITE CHUNKING CONFIGURATION
// ════════════════════════════════════════════════════════════════

// vite.config.ts
import { defineConfig } from 'vite';

export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Vendor chunk for framework
          'vendor-react': ['react', 'react-dom', 'react-router-dom'],
          
          // UI library in separate chunk
          'vendor-ui': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
          
          // Utilities
          'vendor-utils': ['date-fns', 'lodash-es'],
        },
      },
    },
    // Target chunk size
    chunkSizeWarningLimit: 500, // KB
  },
});

// Advanced: Function-based chunking
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          // Put node_modules in vendor chunks
          if (id.includes('node_modules')) {
            // React ecosystem
            if (id.includes('react')) {
              return 'vendor-react';
            }
            // UI components
            if (id.includes('@radix-ui') || id.includes('@headlessui')) {
              return 'vendor-ui';
            }
            // Everything else from node_modules
            return 'vendor';
          }
          
          // Admin routes in separate chunk
          if (id.includes('/pages/admin/')) {
            return 'admin';
          }
        },
      },
    },
  },
});

// ════════════════════════════════════════════════════════════════
// WEBPACK CHUNKING CONFIGURATION
// ════════════════════════════════════════════════════════════════

// webpack.config.js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        // Vendor chunk
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          priority: 20,
        },
        // React ecosystem
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom|react-router)[\\/]/,
          name: 'react',
          chunks: 'all',
          priority: 30,
        },
        // Common code used by multiple chunks
        common: {
          minChunks: 2,
          priority: 10,
          reuseExistingChunk: true,
        },
      },
    },
    // Separate runtime chunk for long-term caching
    runtimeChunk: 'single',
  },
  output: {
    // Content hash for cache busting
    filename: '[name].[contenthash].js',
    chunkFilename: '[name].[contenthash].chunk.js',
  },
};

// ════════════════════════════════════════════════════════════════
// CACHE OPTIMIZATION
// ════════════════════════════════════════════════════════════════

/*
  CHUNKING STRATEGY FOR CACHING:
  
  1. runtime.js      - Webpack runtime, changes rarely
  2. vendor.js       - node_modules, changes on dependency update
  3. react.js        - React ecosystem, very stable
  4. common.js       - Shared app code
  5. [page].js       - Page-specific code
  
  Benefits:
  - User updates are small (only app chunks change)
  - Vendor chunk cached for weeks/months
  - React chunk cached until React version changes
*/

// Content hash in filename ensures cache invalidation
// main.a1b2c3d4.js -> main.e5f6g7h8.js on change
```

---

## 5. Analysis & Monitoring

```typescript
// ════════════════════════════════════════════════════════════════
// BUNDLE ANALYZER
// ════════════════════════════════════════════════════════════════

// Vite
// npm install -D rollup-plugin-visualizer
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    visualizer({
      filename: 'dist/stats.html',
      open: true,
      gzipSize: true,
    }),
  ],
});

// Webpack
// npm install -D webpack-bundle-analyzer
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      reportFilename: 'bundle-report.html',
    }),
  ],
};

// ════════════════════════════════════════════════════════════════
// BUNDLE SIZE BUDGETS
// ════════════════════════════════════════════════════════════════

// Webpack performance hints
module.exports = {
  performance: {
    maxAssetSize: 244 * 1024, // 244 KB
    maxEntrypointSize: 244 * 1024,
    hints: 'error', // Fail build if exceeded
  },
};

// Or use bundlesize package
// package.json
{
  "bundlesize": [
    {
      "path": "./dist/main.*.js",
      "maxSize": "100 kB"
    },
    {
      "path": "./dist/vendor.*.js",
      "maxSize": "150 kB"
    }
  ]
}

// CI script
// npx bundlesize

// ════════════════════════════════════════════════════════════════
// IMPORT COST (VS Code Extension)
// ════════════════════════════════════════════════════════════════

// Shows inline size of imports
import { format } from 'date-fns'; // 2.3K (gzipped)
import moment from 'moment'; // 72.1K (gzipped) ❌

// ════════════════════════════════════════════════════════════════
// CUSTOM SIZE TRACKING
// ════════════════════════════════════════════════════════════════

// scripts/check-bundle-size.js
const fs = require('fs');
const path = require('path');
const gzipSize = require('gzip-size');

const BUDGET = {
  'main': 100 * 1024, // 100KB
  'vendor': 150 * 1024,
};

async function checkBundleSize() {
  const distPath = path.join(__dirname, '../dist/assets');
  const files = fs.readdirSync(distPath);
  
  const results = [];
  
  for (const file of files) {
    if (!file.endsWith('.js')) continue;
    
    const filePath = path.join(distPath, file);
    const content = fs.readFileSync(filePath);
    const gzipped = await gzipSize(content);
    
    const chunkName = file.split('.')[0];
    const budget = BUDGET[chunkName];
    
    results.push({
      file,
      size: gzipped,
      budget,
      overBudget: budget ? gzipped > budget : false,
    });
  }
  
  console.table(results);
  
  if (results.some(r => r.overBudget)) {
    process.exit(1);
  }
}

checkBundleSize();

// ════════════════════════════════════════════════════════════════
// LIGHTHOUSE CI
// ════════════════════════════════════════════════════════════════

// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      url: ['http://localhost:3000'],
      numberOfRuns: 3,
    },
    assert: {
      assertions: {
        'categories:performance': ['error', { minScore: 0.9 }],
        'total-byte-weight': ['error', { maxNumericValue: 500000 }],
        'mainthread-work-breakdown': ['warn', { maxNumericValue: 4000 }],
      },
    },
    upload: {
      target: 'temporary-public-storage',
    },
  },
};

// Run: npx lhci autorun
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# BUNDLE OPTIMIZATION PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Importing entire libraries
# Bad
import _ from 'lodash';  # 70KB
import moment from 'moment';  # 67KB
import * as Icons from 'react-icons';  # All icons!

# Good
import { debounce } from 'lodash-es';  # 2KB
import { format } from 'date-fns';  # 2KB
import { FiMenu } from 'react-icons/fi';  # Single icon

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Not code splitting routes
# Bad
import Home from './pages/Home';
import Products from './pages/Products';
import Admin from './pages/Admin';  # 200KB, all users pay

# Good
const Home = lazy(() => import('./pages/Home'));
const Admin = lazy(() => import('./pages/Admin'));  # Only admin users

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Large images in bundle
# Bad
import heroImage from './hero.png';  # 2MB in JS bundle!

# Good
# Use public folder or CDN
<img src="/images/hero.webp" loading="lazy" />
# Or use image optimization (Next.js Image, etc.)

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Barrel imports breaking tree shaking
# Bad
import { Button } from '@/components';  # May import all

# Good
import { Button } from '@/components/Button';  # Direct import

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Not separating vendor chunks
# Bad
# Everything in one bundle
# App change = download React again

# Good
# Separate vendor chunk
# App changes don't invalidate React cache

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Importing dev-only code in production
# Bad
import { DevTools } from '@tanstack/react-query-devtools';
# Always in bundle

# Good
const DevTools = lazy(() =>
  import('@tanstack/react-query-devtools').then(m => ({
    default: m.ReactQueryDevtools
  }))
);
# Only in dev, or use process.env.NODE_ENV check
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is tree shaking?"**
> "Tree shaking removes unused code from the final bundle. It works with ES modules (import/export) - bundlers analyze which exports are used and eliminate dead code. Requires: ES modules, no side effects, proper library design. CommonJS (require) can't be tree shaken."

**Q: "What is code splitting?"**
> "Code splitting breaks your bundle into smaller chunks loaded on demand. Route-based splitting loads page code only when navigating there. Dynamic import() creates split points. Benefits: smaller initial bundle, faster first load. React.lazy and Suspense make this easy."

**Q: "What's the difference between lazy loading and code splitting?"**
> "Code splitting is the build-time process of creating separate chunks. Lazy loading is the runtime decision of when to load them. They work together: code splitting creates chunks, lazy loading (via dynamic import, IntersectionObserver) decides when to fetch them."

### Intermediate Questions

**Q: "How do you analyze bundle size?"**
> "Use bundle analyzer plugins (webpack-bundle-analyzer, rollup-plugin-visualizer) to visualize what's in your bundle. VS Code Import Cost extension shows size inline. Set up bundle budgets in CI to prevent regression. Check gzipped size, not raw size."

**Q: "What makes a library tree-shakeable?"**
> "Three things: 1) Uses ES modules (export/import), not CommonJS (module.exports). 2) Has sideEffects: false in package.json. 3) Proper module structure - named exports, no import-time side effects. Check package.json for 'module' or 'exports' field."

**Q: "How do you optimize vendor caching?"**
> "Separate vendor code into its own chunk with content-hash filename. Vendor chunk changes less often than app code. Splitting further (react, ui-lib, utils) means even smaller invalidation. Runtime chunk should be separate. Result: most users hit cache on revisit."

### Advanced Questions

**Q: "How do you handle large dependencies?"**
> "First, question if you need it - maybe native API or smaller alternative exists. If needed: lazy load (dynamic import on user action), use CDN with async loading, check for tree-shakeable version (lodash-es not lodash). For charts/editors, load only when visible."

**Q: "How do you set up bundle budgets?"**
> "Define size limits in build config (webpack performance hints) or CI tool (bundlesize, Lighthouse CI). Typical: 100KB initial JS, 150KB vendor. Fail build when exceeded. Track over time. PRs that increase size need justification. Alert on budget breach."

**Q: "How do you optimize for repeat visitors?"**
> "Long-term caching with content-hash filenames. Separate vendor chunk (rarely changes) from app code. Use service worker for offline caching. Preload next routes. CDN for static assets. Result: returning users load mostly from cache, only fetch changed chunks."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              BUNDLE OPTIMIZATION CHECKLIST                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TREE SHAKING:                                                  │
│  □ Use ES modules (import/export)                              │
│  □ Choose tree-shakeable libraries                             │
│  □ Direct imports, avoid barrel re-exports                     │
│  □ Check sideEffects in package.json                           │
│                                                                 │
│  CODE SPLITTING:                                                │
│  □ Split by route (React.lazy)                                 │
│  □ Split heavy components                                      │
│  □ Dynamic import for on-demand features                       │
│  □ Preload on hover/intent                                     │
│                                                                 │
│  CHUNKING:                                                      │
│  □ Separate vendor chunk                                       │
│  □ Content-hash filenames                                      │
│  □ Separate runtime chunk                                      │
│                                                                 │
│  MONITORING:                                                    │
│  □ Bundle analyzer in CI                                       │
│  □ Bundle size budgets                                         │
│  □ Track size over time                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

TARGETS:
┌────────────────────────────────────────────────────────────────┐
│ Initial JS:    < 100KB (gzipped)                               │
│ Vendor chunk:  < 150KB (gzipped)                               │
│ Total:         < 300KB (gzipped)                               │
│ LCP:           < 2.5s                                          │
│ TTI:           < 3.5s                                          │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

