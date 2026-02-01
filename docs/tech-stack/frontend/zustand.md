# Zustand

> Lightweight global state management for React.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | Bear-bones state management |
| **Why** | Simple API, TypeScript support, no boilerplate |
| **Version** | 4.x |
| **Location** | `apps/web/src/stores/` |

## Basic Usage

### Creating a Store
```typescript
// stores/authStore.ts
import { create } from 'zustand';

interface User {
  id: string;
  email: string;
  fullName: string;
}

interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  setUser: (user: User | null) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  isAuthenticated: false,
  setUser: (user) => set({ user, isAuthenticated: !!user }),
  logout: () => set({ user: null, isAuthenticated: false }),
}));
```

### Using in Components
```tsx
'use client';

import { useAuthStore } from '@/stores/authStore';

function Header() {
  // Subscribe to specific state
  const user = useAuthStore((state) => state.user);
  const logout = useAuthStore((state) => state.logout);

  return (
    <header>
      {user ? (
        <>
          <span>Welcome, {user.fullName}</span>
          <button onClick={logout}>Logout</button>
        </>
      ) : (
        <Link href="/login">Login</Link>
      )}
    </header>
  );
}
```

## Store Patterns

### UI State Store
```typescript
// stores/uiStore.ts
interface UIState {
  sidebarOpen: boolean;
  theme: 'light' | 'dark' | 'system';
  notifications: Notification[];
  
  toggleSidebar: () => void;
  setTheme: (theme: 'light' | 'dark' | 'system') => void;
  addNotification: (notification: Notification) => void;
  removeNotification: (id: string) => void;
}

export const useUIStore = create<UIState>((set) => ({
  sidebarOpen: true,
  theme: 'system',
  notifications: [],

  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
  
  setTheme: (theme) => set({ theme }),
  
  addNotification: (notification) =>
    set((state) => ({
      notifications: [...state.notifications, notification],
    })),
    
  removeNotification: (id) =>
    set((state) => ({
      notifications: state.notifications.filter((n) => n.id !== id),
    })),
}));
```

### Family Context Store
```typescript
// stores/familyStore.ts
interface FamilyState {
  currentFamily: Family | null;
  currentCareRecipient: CareRecipient | null;
  
  setCurrentFamily: (family: Family | null) => void;
  setCurrentCareRecipient: (recipient: CareRecipient | null) => void;
  reset: () => void;
}

export const useFamilyStore = create<FamilyState>((set) => ({
  currentFamily: null,
  currentCareRecipient: null,

  setCurrentFamily: (currentFamily) => set({ currentFamily }),
  setCurrentCareRecipient: (currentCareRecipient) => set({ currentCareRecipient }),
  reset: () => set({ currentFamily: null, currentCareRecipient: null }),
}));
```

## Middleware

### Persist Middleware
```typescript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

export const useSettingsStore = create<SettingsState>()(
  persist(
    (set) => ({
      theme: 'system',
      language: 'en',
      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
    }),
    {
      name: 'carecircle-settings',
      storage: createJSONStorage(() => localStorage),
    }
  )
);
```

### DevTools Middleware
```typescript
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';

export const useAuthStore = create<AuthState>()(
  devtools(
    (set) => ({
      user: null,
      setUser: (user) => set({ user }, false, 'setUser'),
    }),
    { name: 'AuthStore' }
  )
);
```

### Combining Middleware
```typescript
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

export const useStore = create<State>()(
  devtools(
    persist(
      (set) => ({
        // state and actions
      }),
      { name: 'app-storage' }
    ),
    { name: 'AppStore' }
  )
);
```

## Advanced Patterns

### Selectors
```typescript
// Prevent unnecessary re-renders with selectors
const userName = useAuthStore((state) => state.user?.fullName);
const isAdmin = useAuthStore((state) => state.user?.role === 'ADMIN');

// Shallow equality for objects
import { shallow } from 'zustand/shallow';

const { user, isAuthenticated } = useAuthStore(
  (state) => ({ user: state.user, isAuthenticated: state.isAuthenticated }),
  shallow
);
```

### Computed Values
```typescript
interface CartState {
  items: CartItem[];
  addItem: (item: CartItem) => void;
}

export const useCartStore = create<CartState>((set, get) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
}));

// Computed selector (outside store)
export const useCartTotal = () =>
  useCartStore((state) =>
    state.items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  );
```

### Actions Outside Components
```typescript
// Access store outside React
const { setUser, logout } = useAuthStore.getState();

// Subscribe to changes
const unsubscribe = useAuthStore.subscribe(
  (state) => state.user,
  (user) => console.log('User changed:', user)
);
```

### Async Actions
```typescript
interface AuthState {
  user: User | null;
  isLoading: boolean;
  error: string | null;
  login: (credentials: LoginCredentials) => Promise<void>;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  isLoading: false,
  error: null,

  login: async (credentials) => {
    set({ isLoading: true, error: null });
    try {
      const response = await api.login(credentials);
      set({ user: response.user, isLoading: false });
    } catch (error) {
      set({ error: error.message, isLoading: false });
    }
  },
}));
```

## Best Practices

### 1. Keep Stores Focused
```typescript
// ❌ Bad: One giant store
const useAppStore = create((set) => ({
  user: null,
  theme: 'light',
  cart: [],
  notifications: [],
  // ... 50 more properties
}));

// ✅ Good: Separate stores by domain
const useAuthStore = create((set) => ({ user: null, ... }));
const useUIStore = create((set) => ({ theme: 'light', ... }));
const useCartStore = create((set) => ({ items: [], ... }));
```

### 2. Use Selectors
```typescript
// ❌ Bad: Re-renders on any state change
const state = useAuthStore();

// ✅ Good: Only re-renders when user changes
const user = useAuthStore((state) => state.user);
```

### 3. Separate Server vs Client State
```typescript
// Server state → TanStack Query
const { data: medications } = useQuery({ ... });

// Client state → Zustand
const selectedMedication = useUIStore((state) => state.selectedMedication);
```

## Troubleshooting

### Hydration Mismatch
```typescript
// For SSR/Next.js, handle hydration:
const useStore = create<State>()(
  persist(
    (set) => ({ ... }),
    {
      name: 'storage',
      skipHydration: true, // Handle manually
    }
  )
);

// In component
useEffect(() => {
  useStore.persist.rehydrate();
}, []);
```

### Stale Closures
```typescript
// ❌ Bad: Stale state in closure
set((state) => {
  setTimeout(() => {
    console.log(state.count); // May be stale
  }, 1000);
  return state;
});

// ✅ Good: Use get() for current state
setTimeout(() => {
  const currentCount = get().count;
  console.log(currentCount);
}, 1000);
```

---

*See also: [TanStack Query](tanstack-query.md), [React](react.md)*


