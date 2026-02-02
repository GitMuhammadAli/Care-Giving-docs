# 🧩 Micro-Frontends - Complete Guide

> A comprehensive guide to micro-frontends - Module Federation, iframe integration, web components, routing, and building scalable frontend architectures.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Micro-frontends extend microservices principles to the frontend, allowing independent teams to develop, deploy, and scale separate portions of a web application that compose together at runtime or build time."

### The 7 Key Concepts (Remember These!)
```
1. MODULE FEDERATION  → Webpack 5 runtime sharing of code
2. BUILD-TIME         → Combine at build (npm packages)
3. RUN-TIME           → Load dynamically at runtime
4. SHELL/HOST         → Container app that loads remotes
5. REMOTE             → Independent micro-frontend module
6. SHARED DEPS        → React, libraries shared between MFEs
7. ROUTING            → Navigation between micro-frontends
```

### Integration Approaches
```
┌─────────────────────────────────────────────────────────────────┐
│              MICRO-FRONTEND INTEGRATION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  MODULE FEDERATION (Webpack 5)                                 │
│  ─────────────────────────────                                  │
│  • Share code at runtime                                       │
│  • Independent deploys                                         │
│  • Shared dependencies                                         │
│  ✅ Best DX, smallest bundle                                   │
│  ❌ Webpack-specific (Vite plugin available)                   │
│  Use: Modern setups, same framework                            │
│                                                                 │
│  IFRAME                                                        │
│  ──────                                                         │
│  • Complete isolation                                          │
│  • Any framework                                               │
│  • Separate DOM/JS context                                     │
│  ✅ Maximum isolation                                          │
│  ❌ Performance, communication, styling                        │
│  Use: Third-party widgets, legacy integration                  │
│                                                                 │
│  WEB COMPONENTS                                                │
│  ──────────────                                                 │
│  • Framework agnostic                                          │
│  • Custom elements                                             │
│  • Shadow DOM encapsulation                                    │
│  ✅ Standards-based, isolation                                 │
│  ❌ React integration quirks                                   │
│  Use: Shared components across frameworks                      │
│                                                                 │
│  NPM PACKAGES                                                  │
│  ────────────                                                   │
│  • Build-time integration                                      │
│  • Version controlled                                          │
│  • Tree-shakeable                                              │
│  ✅ Simple, type-safe                                          │
│  ❌ Coupled deploys                                            │
│  Use: Shared libraries, slower release cycle                   │
│                                                                 │
│  SERVER-SIDE COMPOSITION                                       │
│  ────────────────────────                                       │
│  • Edge/server assembles HTML                                  │
│  • SSI, ESI, server includes                                   │
│  ✅ SEO-friendly, fast initial load                            │
│  ❌ Complex infrastructure                                     │
│  Use: Content-heavy sites, SEO critical                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Architecture Overview
```
┌─────────────────────────────────────────────────────────────────┐
│              MICRO-FRONTEND ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                    ┌─────────────────────┐                     │
│                    │     SHELL/HOST      │                     │
│                    │   (Container App)   │                     │
│                    │  • Routing          │                     │
│                    │  • Auth             │                     │
│                    │  • Layout           │                     │
│                    └──────────┬──────────┘                     │
│                               │                                │
│           ┌───────────────────┼───────────────────┐            │
│           │                   │                   │            │
│           ▼                   ▼                   ▼            │
│    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │
│    │   REMOTE 1  │    │   REMOTE 2  │    │   REMOTE 3  │      │
│    │  (Products) │    │   (Cart)    │    │  (Checkout) │      │
│    │  Team A     │    │  Team B     │    │  Team C     │      │
│    └─────────────┘    └─────────────┘    └─────────────┘      │
│                                                                 │
│    Each remote:                                                │
│    • Own repo                                                  │
│    • Own CI/CD                                                 │
│    • Independent deploy                                        │
│    • Own technology choices                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Module Federation"** | "We use Webpack Module Federation for runtime integration" |
| **"Shell application"** | "The shell handles routing and loads micro-frontends" |
| **"Remote entry"** | "Each MFE exposes a remoteEntry.js for the shell" |
| **"Shared singleton"** | "React is a shared singleton to avoid multiple instances" |
| **"Vertical slice"** | "Each team owns a vertical slice including frontend" |
| **"Composition"** | "Micro-frontends compose at runtime in the shell" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Team size | **5-9 devs** | Per micro-frontend |
| Deploy frequency | **Independent** | No coordination needed |
| Shared deps | **React, router** | Singleton to avoid conflicts |
| Communication | **Events/props** | Loose coupling |

### The "Wow" Statement (Memorize This!)
> "We use Module Federation with a shell app that loads product, cart, and checkout micro-frontends at runtime. Each team owns their vertical slice - repo, CI/CD, deploys independently. React is shared as singleton to avoid conflicts. Teams can even use different React versions with version ranges. Communication uses custom events for loose coupling. The shell handles auth and routing, passing user context via props. We lazy-load MFEs on route change for performance. This lets our 30-person team deploy independently without coordination. Build times dropped from 20 to 5 minutes. New features ship within team boundaries."

---

## 📚 Table of Contents

1. [Module Federation](#1-module-federation)
2. [Shell Application](#2-shell-application)
3. [Communication Patterns](#3-communication-patterns)
4. [Routing](#4-routing)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Module Federation

```typescript
// ════════════════════════════════════════════════════════════════
// REMOTE APPLICATION (Products MFE)
// ════════════════════════════════════════════════════════════════

// webpack.config.js - Remote (products-mfe)
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  entry: './src/index',
  mode: 'development',
  devServer: {
    port: 3001,
  },
  output: {
    publicPath: 'auto',
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'products',
      filename: 'remoteEntry.js',
      exposes: {
        // What this MFE exports
        './ProductList': './src/components/ProductList',
        './ProductDetail': './src/components/ProductDetail',
        './store': './src/store',
      },
      shared: {
        react: {
          singleton: true,
          requiredVersion: '^18.0.0',
        },
        'react-dom': {
          singleton: true,
          requiredVersion: '^18.0.0',
        },
        'react-router-dom': {
          singleton: true,
          requiredVersion: '^6.0.0',
        },
      },
    }),
  ],
};

// src/components/ProductList.tsx
import React from 'react';

interface ProductListProps {
  onProductSelect?: (productId: string) => void;
  category?: string;
}

const ProductList: React.FC<ProductListProps> = ({ onProductSelect, category }) => {
  const products = useProducts(category);

  return (
    <div className="product-list">
      {products.map(product => (
        <div 
          key={product.id} 
          onClick={() => onProductSelect?.(product.id)}
          className="product-card"
        >
          <img src={product.image} alt={product.name} />
          <h3>{product.name}</h3>
          <p>${product.price}</p>
        </div>
      ))}
    </div>
  );
};

export default ProductList;

// src/bootstrap.tsx (Async bootstrap for shared deps)
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root')!);
root.render(<App />);

// src/index.ts (Entry point - dynamic import for async)
import('./bootstrap');

// ════════════════════════════════════════════════════════════════
// HOST APPLICATION (Shell)
// ════════════════════════════════════════════════════════════════

// webpack.config.js - Host (shell)
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  entry: './src/index',
  mode: 'development',
  devServer: {
    port: 3000,
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'shell',
      remotes: {
        // Load remotes at runtime
        products: 'products@http://localhost:3001/remoteEntry.js',
        cart: 'cart@http://localhost:3002/remoteEntry.js',
        checkout: 'checkout@http://localhost:3003/remoteEntry.js',
      },
      shared: {
        react: {
          singleton: true,
          requiredVersion: '^18.0.0',
        },
        'react-dom': {
          singleton: true,
          requiredVersion: '^18.0.0',
        },
        'react-router-dom': {
          singleton: true,
          requiredVersion: '^6.0.0',
        },
      },
    }),
  ],
};

// src/remotes.d.ts - TypeScript declarations
declare module 'products/ProductList' {
  import { FC } from 'react';
  interface ProductListProps {
    onProductSelect?: (productId: string) => void;
    category?: string;
  }
  const ProductList: FC<ProductListProps>;
  export default ProductList;
}

declare module 'products/ProductDetail' {
  import { FC } from 'react';
  interface ProductDetailProps {
    productId: string;
  }
  const ProductDetail: FC<ProductDetailProps>;
  export default ProductDetail;
}

declare module 'cart/MiniCart' {
  import { FC } from 'react';
  const MiniCart: FC;
  export default MiniCart;
}

declare module 'checkout/CheckoutFlow' {
  import { FC } from 'react';
  const CheckoutFlow: FC;
  export default CheckoutFlow;
}
```

---

## 2. Shell Application

```tsx
// ════════════════════════════════════════════════════════════════
// SHELL APPLICATION
// ════════════════════════════════════════════════════════════════

// src/App.tsx
import React, { Suspense, lazy } from 'react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';
import { AuthProvider } from './contexts/AuthContext';
import { ErrorBoundary } from './components/ErrorBoundary';
import Layout from './components/Layout';
import Loading from './components/Loading';

// Lazy load micro-frontends
const ProductList = lazy(() => import('products/ProductList'));
const ProductDetail = lazy(() => import('products/ProductDetail'));
const Cart = lazy(() => import('cart/CartPage'));
const Checkout = lazy(() => import('checkout/CheckoutFlow'));

// Fallback if MFE fails to load
const MFEErrorFallback = ({ error, resetErrorBoundary }) => (
  <div className="mfe-error">
    <h2>Failed to load this section</h2>
    <p>{error.message}</p>
    <button onClick={resetErrorBoundary}>Try again</button>
  </div>
);

function App() {
  return (
    <BrowserRouter>
      <AuthProvider>
        <Layout>
          <ErrorBoundary FallbackComponent={MFEErrorFallback}>
            <Suspense fallback={<Loading />}>
              <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/products" element={<ProductList />} />
                <Route path="/products/:id" element={<ProductDetail />} />
                <Route path="/cart" element={<Cart />} />
                <Route path="/checkout/*" element={<Checkout />} />
              </Routes>
            </Suspense>
          </ErrorBoundary>
        </Layout>
      </AuthProvider>
    </BrowserRouter>
  );
}

// src/components/Layout.tsx
import React from 'react';
import Header from './Header';
import { MiniCart } from './MiniCart';

const Layout: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return (
    <div className="app-layout">
      <Header />
      <main className="main-content">
        {children}
      </main>
      <MiniCart />
    </div>
  );
};

// src/components/MiniCart.tsx - Load cart MFE
import React, { Suspense, lazy } from 'react';

const RemoteMiniCart = lazy(() => import('cart/MiniCart'));

export const MiniCart = () => (
  <Suspense fallback={<div className="cart-loading">...</div>}>
    <RemoteMiniCart />
  </Suspense>
);

// ════════════════════════════════════════════════════════════════
// DYNAMIC REMOTE LOADING
// ════════════════════════════════════════════════════════════════

// For dynamic/runtime remote URLs
const loadRemote = async (scope: string, module: string) => {
  // Initialize the remote container
  await __webpack_init_sharing__('default');
  const container = window[scope];
  
  // Initialize the container
  await container.init(__webpack_share_scopes__.default);
  
  // Get the module factory
  const factory = await container.get(module);
  
  return factory();
};

// Dynamic component loader
const DynamicRemote: React.FC<{
  scope: string;
  module: string;
  url: string;
  props?: Record<string, any>;
}> = ({ scope, module, url, props = {} }) => {
  const [Component, setComponent] = React.useState<React.ComponentType | null>(null);
  const [error, setError] = React.useState<Error | null>(null);

  React.useEffect(() => {
    const loadComponent = async () => {
      try {
        // Load remote script
        if (!window[scope]) {
          await new Promise((resolve, reject) => {
            const script = document.createElement('script');
            script.src = url;
            script.onload = resolve;
            script.onerror = reject;
            document.head.appendChild(script);
          });
        }

        const { default: LoadedComponent } = await loadRemote(scope, module);
        setComponent(() => LoadedComponent);
      } catch (err) {
        setError(err as Error);
      }
    };

    loadComponent();
  }, [scope, module, url]);

  if (error) return <div>Failed to load: {error.message}</div>;
  if (!Component) return <div>Loading...</div>;

  return <Component {...props} />;
};

// Usage with config
const remoteConfig = {
  products: {
    url: process.env.PRODUCTS_MFE_URL || 'http://localhost:3001/remoteEntry.js',
    scope: 'products',
  },
  cart: {
    url: process.env.CART_MFE_URL || 'http://localhost:3002/remoteEntry.js',
    scope: 'cart',
  },
};

<DynamicRemote
  scope={remoteConfig.products.scope}
  module="./ProductList"
  url={remoteConfig.products.url}
  props={{ category: 'electronics' }}
/>
```

---

## 3. Communication Patterns

```tsx
// ════════════════════════════════════════════════════════════════
// CUSTOM EVENTS (Loose Coupling)
// ════════════════════════════════════════════════════════════════

// shared/events.ts - Shared event types
export interface AppEvents {
  'cart:add': { productId: string; quantity: number };
  'cart:remove': { productId: string };
  'cart:update': { productId: string; quantity: number };
  'user:login': { userId: string; name: string };
  'user:logout': {};
  'navigation:change': { path: string };
}

// Event emitter utility
class EventBus {
  private listeners: Map<string, Set<Function>> = new Map();

  emit<K extends keyof AppEvents>(event: K, data: AppEvents[K]) {
    const customEvent = new CustomEvent(event, { detail: data });
    window.dispatchEvent(customEvent);
  }

  on<K extends keyof AppEvents>(
    event: K,
    callback: (data: AppEvents[K]) => void
  ): () => void {
    const handler = (e: CustomEvent<AppEvents[K]>) => callback(e.detail);
    window.addEventListener(event, handler as EventListener);
    return () => window.removeEventListener(event, handler as EventListener);
  }
}

export const eventBus = new EventBus();

// ════════════════════════════════════════════════════════════════
// USAGE IN MICRO-FRONTENDS
// ════════════════════════════════════════════════════════════════

// Products MFE - Emit cart add event
import { eventBus } from '@shared/events';

function ProductCard({ product }) {
  const handleAddToCart = () => {
    eventBus.emit('cart:add', {
      productId: product.id,
      quantity: 1,
    });
  };

  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <button onClick={handleAddToCart}>Add to Cart</button>
    </div>
  );
}

// Cart MFE - Listen for cart events
import { useEffect } from 'react';
import { eventBus } from '@shared/events';

function useCartEvents() {
  const { addItem, removeItem, updateQuantity } = useCartStore();

  useEffect(() => {
    const unsubAdd = eventBus.on('cart:add', ({ productId, quantity }) => {
      addItem(productId, quantity);
    });

    const unsubRemove = eventBus.on('cart:remove', ({ productId }) => {
      removeItem(productId);
    });

    const unsubUpdate = eventBus.on('cart:update', ({ productId, quantity }) => {
      updateQuantity(productId, quantity);
    });

    return () => {
      unsubAdd();
      unsubRemove();
      unsubUpdate();
    };
  }, [addItem, removeItem, updateQuantity]);
}

// ════════════════════════════════════════════════════════════════
// PROPS PASSING (Direct Communication)
// ════════════════════════════════════════════════════════════════

// Shell passes props to remotes
function ProductsPage() {
  const { user } = useAuth();
  const navigate = useNavigate();

  const handleProductSelect = (productId: string) => {
    navigate(`/products/${productId}`);
  };

  const handleAddToCart = (productId: string, quantity: number) => {
    eventBus.emit('cart:add', { productId, quantity });
  };

  return (
    <Suspense fallback={<Loading />}>
      <ProductList
        user={user}
        onProductSelect={handleProductSelect}
        onAddToCart={handleAddToCart}
      />
    </Suspense>
  );
}

// ════════════════════════════════════════════════════════════════
// SHARED STATE (Zustand across MFEs)
// ════════════════════════════════════════════════════════════════

// shared/stores/cartStore.ts
import { create } from 'zustand';

interface CartItem {
  productId: string;
  quantity: number;
  price: number;
}

interface CartStore {
  items: CartItem[];
  addItem: (productId: string, quantity: number, price: number) => void;
  removeItem: (productId: string) => void;
  clearCart: () => void;
  total: () => number;
}

// Exposed from Cart MFE
export const useCartStore = create<CartStore>((set, get) => ({
  items: [],
  addItem: (productId, quantity, price) =>
    set((state) => ({
      items: [...state.items, { productId, quantity, price }],
    })),
  removeItem: (productId) =>
    set((state) => ({
      items: state.items.filter((i) => i.productId !== productId),
    })),
  clearCart: () => set({ items: [] }),
  total: () => get().items.reduce((sum, i) => sum + i.price * i.quantity, 0),
}));

// Cart MFE webpack.config.js
exposes: {
  './MiniCart': './src/components/MiniCart',
  './CartPage': './src/pages/CartPage',
  './store': './src/stores/cartStore',  // Expose store
}

// Products MFE uses Cart store
import { useCartStore } from 'cart/store';

function ProductCard({ product }) {
  const addItem = useCartStore((state) => state.addItem);
  
  return (
    <button onClick={() => addItem(product.id, 1, product.price)}>
      Add to Cart
    </button>
  );
}
```

---

## 4. Routing

```tsx
// ════════════════════════════════════════════════════════════════
// SHELL ROUTING
// ════════════════════════════════════════════════════════════════

// Shell owns top-level routes
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';

function ShellRoutes() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Shell-owned routes */}
        <Route path="/" element={<HomePage />} />
        <Route path="/login" element={<LoginPage />} />
        
        {/* Products MFE routes */}
        <Route path="/products/*" element={
          <ErrorBoundary>
            <Suspense fallback={<Loading />}>
              <ProductRoutes />
            </Suspense>
          </ErrorBoundary>
        } />
        
        {/* Cart MFE routes */}
        <Route path="/cart/*" element={
          <Suspense fallback={<Loading />}>
            <CartRoutes />
          </Suspense>
        } />
        
        {/* Checkout MFE routes */}
        <Route path="/checkout/*" element={
          <ProtectedRoute>
            <Suspense fallback={<Loading />}>
              <CheckoutRoutes />
            </Suspense>
          </ProtectedRoute>
        } />
        
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

// ════════════════════════════════════════════════════════════════
// REMOTE ROUTING (Products MFE)
// ════════════════════════════════════════════════════════════════

// Products MFE - Internal routes
const ProductRoutes = lazy(() => import('products/Routes'));

// products-mfe/src/Routes.tsx
import { Routes, Route } from 'react-router-dom';
import ProductList from './pages/ProductList';
import ProductDetail from './pages/ProductDetail';
import CategoryPage from './pages/CategoryPage';

export default function ProductRoutes() {
  return (
    <Routes>
      <Route index element={<ProductList />} />
      <Route path=":productId" element={<ProductDetail />} />
      <Route path="category/:categoryId" element={<CategoryPage />} />
    </Routes>
  );
}

// ════════════════════════════════════════════════════════════════
// NAVIGATION BETWEEN MFEs
// ════════════════════════════════════════════════════════════════

// Products MFE navigates to Cart
import { useNavigate } from 'react-router-dom';

function AddToCartButton({ productId }) {
  const navigate = useNavigate();
  
  const handleAddToCart = async () => {
    // Add to cart via event
    eventBus.emit('cart:add', { productId, quantity: 1 });
    
    // Navigate to cart (handled by shell's router)
    navigate('/cart');
  };
  
  return <button onClick={handleAddToCart}>Add to Cart</button>;
}

// ════════════════════════════════════════════════════════════════
// MEMORY ROUTER FOR STANDALONE DEVELOPMENT
// ════════════════════════════════════════════════════════════════

// Products MFE standalone mode
import { MemoryRouter, BrowserRouter } from 'react-router-dom';

const Router = process.env.STANDALONE 
  ? BrowserRouter 
  : MemoryRouter;

function ProductsApp() {
  return (
    <Router>
      <ProductRoutes />
    </Router>
  );
}

// Detect if running standalone or in shell
const isStandalone = !window.__SHELL__;

export default function App() {
  if (isStandalone) {
    return (
      <BrowserRouter>
        <ProductRoutes />
      </BrowserRouter>
    );
  }
  
  // When in shell, routes are provided by shell
  return <ProductRoutes />;
}
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# MICRO-FRONTEND BEST PRACTICES
# ════════════════════════════════════════════════════════════════

team_organization:
  vertical_slices:
    - Each team owns full vertical slice
    - Frontend + backend + database
    - Independent deploy pipeline
    - Clear domain boundaries
    
  team_size:
    - 5-9 developers per MFE
    - Cross-functional (FE, BE, QA)
    - Own their domain end-to-end

shared_code:
  minimize:
    - Share only what's necessary
    - React, router (singletons)
    - Design system components
    - Utility libraries
    
  versioning:
    - Shared deps use version ranges
    - Design system has breaking change policy
    - Independent upgrade path

communication:
  patterns:
    - Custom events for loose coupling
    - Props for parent-child
    - Shared state only when necessary
    - URL for shareable state
    
  avoid:
    - Direct function calls between MFEs
    - Tight coupling to internal APIs
    - Shared mutable state

deployment:
  independence:
    - Each MFE deploys independently
    - No coordination required
    - Feature flags for gradual rollout
    
  versioning:
    - Semantic versioning for remoteEntry
    - Backward compatible changes
    - Deprecation strategy

styling:
  isolation:
    - Scoped CSS (modules, styled-components)
    - BEM naming convention
    - CSS custom properties for theming
    - Avoid global styles

# ════════════════════════════════════════════════════════════════
# PROJECT STRUCTURE
# ════════════════════════════════════════════════════════════════

project_structure:
  monorepo:
    packages/
      shell/           # Host application
      products-mfe/    # Products micro-frontend
      cart-mfe/        # Cart micro-frontend
      shared/          # Shared utilities, types
      design-system/   # Shared components
      
  separate_repos:
    shell-app/
    products-mfe/
    cart-mfe/
    shared-libs/       # Published to npm
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# MICRO-FRONTEND PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Shared state between MFEs
# Bad
# Global Redux store shared between all MFEs
# Tight coupling, coordination required

# Good
# Each MFE has own state
# Communication via events
# Shared state only for truly global (auth)

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Multiple React instances
# Bad
# Each MFE bundles its own React
# Multiple React instances = hooks don't work

# Good
shared: {
  react: { singleton: true, requiredVersion: '^18.0.0' },
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: CSS conflicts
# Bad
.button { ... }  # Global styles conflict

# Good
# Use CSS modules, scoped styles
.products_button__abc123 { ... }

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Too many micro-frontends
# Bad
# 20 micro-frontends for small team
# Overhead exceeds benefits

# Good
# Start with 2-3 for 15-20 devs
# Split when team grows
# Each MFE = 1-2 teams

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Tight version coupling
# Bad
# All MFEs must use exact same React version
# One upgrade = all upgrade

# Good
# Use version ranges
# Test compatibility
# Independent upgrade paths

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: No fallback for failed MFE
# Bad
# MFE fails to load = blank screen

# Good
<ErrorBoundary fallback={<MFEFallback />}>
  <Suspense fallback={<Loading />}>
    <RemoteMFE />
  </Suspense>
</ErrorBoundary>
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What are micro-frontends?"**
> "Micro-frontends extend microservices to frontend - independent teams develop, deploy, and scale separate portions of a web application. Each MFE is owned by a team, has its own repo and CI/CD, and composes into a larger application. Benefits: independent deploys, team autonomy, technology flexibility."

**Q: "What is Module Federation?"**
> "Webpack 5 feature for sharing code at runtime between separately built applications. A shell app loads remote micro-frontends via their remoteEntry.js. Shared dependencies (React) are singletons to avoid conflicts. Enables independent builds and deploys while sharing code efficiently."

**Q: "What are the integration approaches for micro-frontends?"**
> "Module Federation: runtime sharing, best for same framework. Iframes: complete isolation, any framework, performance overhead. Web Components: standards-based, shadow DOM. NPM packages: build-time, version controlled. Server composition: SSI/ESI, SEO-friendly. Choose based on isolation needs and framework constraints."

### Intermediate Questions

**Q: "How do micro-frontends communicate?"**
> "Custom events for loose coupling (eventBus pattern). Props for parent-child (shell passes to remote). Shared state sparingly (Zustand store exposed by one MFE). URL state for shareable. Avoid: direct function calls, tight coupling. Events are preferred - MFEs don't need to know about each other."

**Q: "How do you handle routing with micro-frontends?"**
> "Shell owns top-level routes, delegates to MFEs via wildcard routes (/products/*). Each MFE has internal routes. Shared router (react-router) as singleton. MFEs can navigate using useNavigate. Memory router for standalone development. URL changes handled by shell."

**Q: "How do you share dependencies?"**
> "Module Federation's shared config marks dependencies as singleton. React, react-dom, router must be singletons - multiple instances break hooks. Use version ranges for flexibility. Non-critical deps can duplicate. Design system published to npm, consumed by all MFEs."

### Advanced Questions

**Q: "When should you NOT use micro-frontends?"**
> "Small team (< 10 devs) - overhead exceeds benefits. Simple application - monolith is simpler. Tight coupling between features - artificial boundaries cause pain. Performance-critical - added network requests. Start with monolith, split when team/complexity grows. Not a default architecture."

**Q: "How do you handle authentication across MFEs?"**
> "Shell handles auth, stores token. Passes user context via props or shared state. Each MFE's API calls include token from shared location. Logout event clears all MFE state. Don't duplicate auth logic in each MFE. Consider: token refresh, session timeout coordination."

**Q: "How do you deploy micro-frontends?"**
> "Each MFE has own CI/CD pipeline. Deploy remoteEntry.js to CDN. Shell loads remotes by URL (can be from config/env). Blue-green or canary per MFE. Feature flags for gradual rollout. No coordination needed - shell loads latest remote. Version remoteEntry if needed."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              MICRO-FRONTEND CHECKLIST                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ARCHITECTURE:                                                  │
│  □ Shell/host for routing, auth, layout                        │
│  □ Clear domain boundaries per MFE                             │
│  □ Independent repos and CI/CD                                 │
│  □ Vertical team ownership                                     │
│                                                                 │
│  MODULE FEDERATION:                                             │
│  □ React as shared singleton                                   │
│  □ Async bootstrap (dynamic import)                            │
│  □ TypeScript declarations for remotes                         │
│  □ Error boundaries and Suspense                               │
│                                                                 │
│  COMMUNICATION:                                                 │
│  □ Custom events for loose coupling                            │
│  □ Props for direct parent-child                               │
│  □ Minimal shared state                                        │
│                                                                 │
│  STYLING:                                                       │
│  □ Scoped CSS (modules, styled-components)                     │
│  □ Shared design system                                        │
│  □ CSS custom properties for theming                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

INTEGRATION COMPARISON:
┌────────────────────────────────────────────────────────────────┐
│ Module Federation: Runtime, shared deps, Webpack 5             │
│ Iframe:           Complete isolation, any framework            │
│ Web Components:   Standards-based, shadow DOM                  │
│ NPM Packages:     Build-time, coupled deploys                  │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

