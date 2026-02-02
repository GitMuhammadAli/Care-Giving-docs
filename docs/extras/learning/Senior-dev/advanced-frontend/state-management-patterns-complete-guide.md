# 🔄 State Management Patterns - Complete Guide

> A comprehensive guide to frontend state management - Redux, Zustand, Jotai, Context, and choosing the right tool for your application.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "State management is the practice of organizing and synchronizing application data across components, choosing between global solutions (Redux, Zustand), atomic state (Jotai, Recoil), or built-in options (Context, useState) based on complexity, performance needs, and developer experience."

### The 7 Key Concepts (Remember These!)
```
1. LOCAL STATE       → useState, useReducer - component-specific
2. GLOBAL STATE      → Redux, Zustand - app-wide shared state
3. SERVER STATE      → React Query, SWR - cached API data
4. ATOMIC STATE      → Jotai, Recoil - bottom-up state atoms
5. CONTEXT           → React Context - prop drilling solution
6. DERIVED STATE     → Computed/selector values from base state
7. PERSISTENCE       → localStorage, sessionStorage sync
```

### State Management Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│              STATE MANAGEMENT COMPARISON                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  REDUX                                                         │
│  ─────                                                          │
│  • Single global store                                         │
│  • Unidirectional data flow                                    │
│  • Time-travel debugging                                       │
│  • Large ecosystem (RTK, Saga, Thunk)                          │
│  ✅ Predictable, debuggable                                    │
│  ❌ Boilerplate, learning curve                                │
│  Use: Large apps, complex state, team standardization          │
│                                                                 │
│  ZUSTAND                                                       │
│  ───────                                                        │
│  • Minimal API, hooks-based                                    │
│  • No providers needed                                         │
│  • TypeScript-first                                            │
│  • Middleware support                                          │
│  ✅ Simple, small bundle                                       │
│  ❌ Less structured than Redux                                 │
│  Use: Medium apps, simpler global state                        │
│                                                                 │
│  JOTAI                                                         │
│  ─────                                                          │
│  • Atomic state model                                          │
│  • Bottom-up approach                                          │
│  • Automatic code splitting                                    │
│  • Primitive and derived atoms                                 │
│  ✅ Granular re-renders, composable                            │
│  ❌ Mental model shift                                         │
│  Use: Complex derived state, fine-grained updates              │
│                                                                 │
│  CONTEXT                                                       │
│  ───────                                                        │
│  • Built into React                                            │
│  • No external dependencies                                    │
│  • Provider pattern                                            │
│  ✅ No library needed                                          │
│  ❌ Re-render issues, not optimized for frequent updates       │
│  Use: Theme, auth, infrequent updates                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### When to Use What
```
┌─────────────────────────────────────────────────────────────────┐
│                   DECISION MATRIX                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  NEED                           │ SOLUTION                      │
│  ──────────────────────────────│─────────────────────────────  │
│  Component-local state          │ useState, useReducer          │
│  Share between siblings         │ Lift state up                 │
│  Share across tree (infrequent) │ Context                       │
│  Share across tree (frequent)   │ Zustand, Jotai, Redux         │
│  Server data caching            │ React Query, SWR              │
│  Complex derived state          │ Jotai, Recoil, Redux selectors│
│  Form state                     │ React Hook Form, Formik       │
│  URL state                      │ URL params, search params     │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  PROJECT SIZE:                                                 │
│  Small (1-5 devs)  → useState + Context or Zustand             │
│  Medium (5-15 devs) → Zustand or Redux Toolkit                 │
│  Large (15+ devs)   → Redux Toolkit (standardization)          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Single source of truth"** | "Redux provides a single source of truth for app state" |
| **"Atomic state"** | "Jotai's atomic model enables granular re-renders" |
| **"Selector memoization"** | "We memoize selectors to prevent unnecessary re-renders" |
| **"Derived state"** | "Computed values are derived state from base atoms" |
| **"Flux pattern"** | "Redux follows the Flux pattern with unidirectional flow" |
| **"State colocation"** | "We colocate state near where it's used" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Redux bundle | **~4KB** | RTK is tree-shakable |
| Zustand bundle | **~1KB** | Minimal footprint |
| Jotai bundle | **~3KB** | Atom-based |
| Context re-render | **All consumers** | No selector support |

### The "Wow" Statement (Memorize This!)
> "We use a layered state management approach: useState for component-local, React Query for server state, and Zustand for UI state that needs sharing. We avoid Context for frequently-updated state because it re-renders all consumers. For complex forms, React Hook Form handles local form state efficiently. Zustand gives us Redux-like patterns without the boilerplate - we get devtools, persistence middleware, and selectors. We colocate state as close to usage as possible, lifting only when necessary. Server state is separate from client state - React Query handles caching, deduplication, and background refetching. This separation keeps our state predictable and our components focused."

---

## 📚 Table of Contents

1. [useState & useReducer](#1-usestate--usereducer)
2. [React Context](#2-react-context)
3. [Redux Toolkit](#3-redux-toolkit)
4. [Zustand](#4-zustand)
5. [Jotai](#5-jotai)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. useState & useReducer

```tsx
// ════════════════════════════════════════════════════════════════
// useState - SIMPLE LOCAL STATE
// ════════════════════════════════════════════════════════════════

import { useState, useCallback } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  // Functional update for state based on previous value
  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);
  
  return (
    <div>
      <span>{count}</span>
      <button onClick={increment}>+</button>
    </div>
  );
}

// Complex state with object
function UserForm() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    role: 'user',
  });

  // Update single field
  const updateField = (field: string, value: string) => {
    setUser(prev => ({ ...prev, [field]: value }));
  };

  return (
    <form>
      <input 
        value={user.name} 
        onChange={e => updateField('name', e.target.value)} 
      />
      <input 
        value={user.email} 
        onChange={e => updateField('email', e.target.value)} 
      />
    </form>
  );
}

// ════════════════════════════════════════════════════════════════
// useReducer - COMPLEX STATE LOGIC
// ════════════════════════════════════════════════════════════════

import { useReducer } from 'react';

// State type
interface CartState {
  items: Array<{ id: string; name: string; quantity: number; price: number }>;
  total: number;
  isLoading: boolean;
  error: string | null;
}

// Action types
type CartAction =
  | { type: 'ADD_ITEM'; payload: { id: string; name: string; price: number } }
  | { type: 'REMOVE_ITEM'; payload: { id: string } }
  | { type: 'UPDATE_QUANTITY'; payload: { id: string; quantity: number } }
  | { type: 'CLEAR_CART' }
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'SET_ERROR'; payload: string };

const initialState: CartState = {
  items: [],
  total: 0,
  isLoading: false,
  error: null,
};

// Reducer function
function cartReducer(state: CartState, action: CartAction): CartState {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existingItem = state.items.find(item => item.id === action.payload.id);
      
      if (existingItem) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          ),
          total: state.total + action.payload.price,
        };
      }
      
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }],
        total: state.total + action.payload.price,
      };
    }
    
    case 'REMOVE_ITEM': {
      const item = state.items.find(i => i.id === action.payload.id);
      return {
        ...state,
        items: state.items.filter(i => i.id !== action.payload.id),
        total: state.total - (item ? item.price * item.quantity : 0),
      };
    }
    
    case 'UPDATE_QUANTITY': {
      const item = state.items.find(i => i.id === action.payload.id);
      if (!item) return state;
      
      const quantityDiff = action.payload.quantity - item.quantity;
      return {
        ...state,
        items: state.items.map(i =>
          i.id === action.payload.id
            ? { ...i, quantity: action.payload.quantity }
            : i
        ),
        total: state.total + (item.price * quantityDiff),
      };
    }
    
    case 'CLEAR_CART':
      return initialState;
    
    case 'SET_LOADING':
      return { ...state, isLoading: action.payload };
    
    case 'SET_ERROR':
      return { ...state, error: action.payload, isLoading: false };
    
    default:
      return state;
  }
}

// Usage
function ShoppingCart() {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  
  const addItem = (product: { id: string; name: string; price: number }) => {
    dispatch({ type: 'ADD_ITEM', payload: product });
  };
  
  const removeItem = (id: string) => {
    dispatch({ type: 'REMOVE_ITEM', payload: { id } });
  };
  
  return (
    <div>
      <h2>Cart ({state.items.length} items)</h2>
      {state.items.map(item => (
        <div key={item.id}>
          <span>{item.name} x {item.quantity}</span>
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}
      <p>Total: ${state.total.toFixed(2)}</p>
    </div>
  );
}
```

---

## 2. React Context

```tsx
// ════════════════════════════════════════════════════════════════
// CONTEXT FOR GLOBAL STATE
// ════════════════════════════════════════════════════════════════

import { createContext, useContext, useState, useCallback, useMemo, ReactNode } from 'react';

// ════════════════════════════════════════════════════════════════
// THEME CONTEXT (Good use case - infrequent updates)
// ════════════════════════════════════════════════════════════════

type Theme = 'light' | 'dark';

interface ThemeContextValue {
  theme: Theme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);

function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');
  
  const toggleTheme = useCallback(() => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  }, []);
  
  // Memoize value to prevent unnecessary re-renders
  const value = useMemo(() => ({
    theme,
    toggleTheme,
  }), [theme, toggleTheme]);
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// ════════════════════════════════════════════════════════════════
// AUTH CONTEXT (Good use case)
// ════════════════════════════════════════════════════════════════

interface User {
  id: string;
  name: string;
  email: string;
  role: string;
}

interface AuthContextValue {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextValue | null>(null);

function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  const login = useCallback(async (email: string, password: string) => {
    setIsLoading(true);
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        body: JSON.stringify({ email, password }),
      });
      const data = await response.json();
      setUser(data.user);
    } finally {
      setIsLoading(false);
    }
  }, []);

  const logout = useCallback(() => {
    setUser(null);
  }, []);

  const value = useMemo(() => ({
    user,
    isAuthenticated: !!user,
    isLoading,
    login,
    logout,
  }), [user, isLoading, login, logout]);

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// ════════════════════════════════════════════════════════════════
// CONTEXT PERFORMANCE OPTIMIZATION
// ════════════════════════════════════════════════════════════════

// Problem: All consumers re-render when ANY value changes
// Solution: Split contexts by update frequency

// Separate state and dispatch contexts
const CountStateContext = createContext<number>(0);
const CountDispatchContext = createContext<React.Dispatch<React.SetStateAction<number>>>(() => {});

function CountProvider({ children }: { children: ReactNode }) {
  const [count, setCount] = useState(0);
  
  return (
    <CountStateContext.Provider value={count}>
      <CountDispatchContext.Provider value={setCount}>
        {children}
      </CountDispatchContext.Provider>
    </CountStateContext.Provider>
  );
}

// Components that only dispatch don't re-render when state changes
function IncrementButton() {
  const setCount = useContext(CountDispatchContext);
  console.log('IncrementButton render'); // Only renders once
  
  return (
    <button onClick={() => setCount(c => c + 1)}>
      Increment
    </button>
  );
}

// Only components reading state re-render
function CountDisplay() {
  const count = useContext(CountStateContext);
  console.log('CountDisplay render'); // Re-renders on count change
  
  return <span>Count: {count}</span>;
}
```

---

## 3. Redux Toolkit

```tsx
// ════════════════════════════════════════════════════════════════
// REDUX TOOLKIT SETUP
// ════════════════════════════════════════════════════════════════

// store/slices/userSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserState {
  currentUser: User | null;
  users: User[];
  status: 'idle' | 'loading' | 'succeeded' | 'failed';
  error: string | null;
}

const initialState: UserState = {
  currentUser: null,
  users: [],
  status: 'idle',
  error: null,
};

// Async thunk for API calls
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async (_, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/users');
      if (!response.ok) throw new Error('Failed to fetch');
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

export const createUser = createAsyncThunk(
  'users/createUser',
  async (userData: Omit<User, 'id'>, { rejectWithValue }) => {
    try {
      const response = await fetch('/api/users', {
        method: 'POST',
        body: JSON.stringify(userData),
      });
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const userSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {
    setCurrentUser: (state, action: PayloadAction<User | null>) => {
      state.currentUser = action.payload;
    },
    updateUser: (state, action: PayloadAction<Partial<User> & { id: string }>) => {
      const index = state.users.findIndex(u => u.id === action.payload.id);
      if (index !== -1) {
        state.users[index] = { ...state.users[index], ...action.payload };
      }
    },
    removeUser: (state, action: PayloadAction<string>) => {
      state.users = state.users.filter(u => u.id !== action.payload);
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.users = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload as string;
      })
      .addCase(createUser.fulfilled, (state, action) => {
        state.users.push(action.payload);
      });
  },
});

export const { setCurrentUser, updateUser, removeUser } = userSlice.actions;
export default userSlice.reducer;

// ════════════════════════════════════════════════════════════════
// STORE CONFIGURATION
// ════════════════════════════════════════════════════════════════

// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './slices/userSlice';
import cartReducer from './slices/cartSlice';

export const store = configureStore({
  reducer: {
    users: userReducer,
    cart: cartReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        // Ignore these paths in serializable check
        ignoredActions: ['persist/PERSIST'],
      },
    }),
  devTools: process.env.NODE_ENV !== 'production',
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// Typed hooks
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;

// ════════════════════════════════════════════════════════════════
// SELECTORS WITH MEMOIZATION
// ════════════════════════════════════════════════════════════════

import { createSelector } from '@reduxjs/toolkit';

// Simple selectors
const selectUsers = (state: RootState) => state.users.users;
const selectSearchTerm = (state: RootState) => state.users.searchTerm;

// Memoized selector - only recalculates when inputs change
export const selectFilteredUsers = createSelector(
  [selectUsers, selectSearchTerm],
  (users, searchTerm) => {
    if (!searchTerm) return users;
    return users.filter(user =>
      user.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }
);

// Parameterized selector
export const selectUserById = (userId: string) =>
  createSelector(
    [selectUsers],
    (users) => users.find(u => u.id === userId)
  );

// ════════════════════════════════════════════════════════════════
// COMPONENT USAGE
// ════════════════════════════════════════════════════════════════

function UserList() {
  const dispatch = useAppDispatch();
  const users = useAppSelector(selectFilteredUsers);
  const status = useAppSelector(state => state.users.status);

  useEffect(() => {
    if (status === 'idle') {
      dispatch(fetchUsers());
    }
  }, [status, dispatch]);

  if (status === 'loading') return <Spinner />;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name}
          <button onClick={() => dispatch(removeUser(user.id))}>
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```

---

## 4. Zustand

```tsx
// ════════════════════════════════════════════════════════════════
// ZUSTAND STORE
// ════════════════════════════════════════════════════════════════

import { create } from 'zustand';
import { devtools, persist, subscribeWithSelector } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

// Store types
interface User {
  id: string;
  name: string;
  email: string;
}

interface UserStore {
  // State
  users: User[];
  currentUser: User | null;
  isLoading: boolean;
  error: string | null;
  
  // Actions
  setUsers: (users: User[]) => void;
  addUser: (user: User) => void;
  updateUser: (id: string, updates: Partial<User>) => void;
  removeUser: (id: string) => void;
  setCurrentUser: (user: User | null) => void;
  fetchUsers: () => Promise<void>;
}

// Create store
export const useUserStore = create<UserStore>()(
  devtools(
    persist(
      immer((set, get) => ({
        // Initial state
        users: [],
        currentUser: null,
        isLoading: false,
        error: null,

        // Actions
        setUsers: (users) => set({ users }),
        
        addUser: (user) =>
          set((state) => {
            state.users.push(user); // Immer allows mutation
          }),
        
        updateUser: (id, updates) =>
          set((state) => {
            const user = state.users.find((u) => u.id === id);
            if (user) {
              Object.assign(user, updates);
            }
          }),
        
        removeUser: (id) =>
          set((state) => {
            state.users = state.users.filter((u) => u.id !== id);
          }),
        
        setCurrentUser: (user) => set({ currentUser: user }),
        
        fetchUsers: async () => {
          set({ isLoading: true, error: null });
          try {
            const response = await fetch('/api/users');
            const users = await response.json();
            set({ users, isLoading: false });
          } catch (error) {
            set({ error: (error as Error).message, isLoading: false });
          }
        },
      })),
      {
        name: 'user-storage', // localStorage key
        partialize: (state) => ({ currentUser: state.currentUser }), // Only persist currentUser
      }
    ),
    { name: 'UserStore' } // DevTools name
  )
);

// ════════════════════════════════════════════════════════════════
// SELECTORS (Prevent re-renders)
// ════════════════════════════════════════════════════════════════

// ❌ BAD - Entire store subscribed, re-renders on any change
// const { users, addUser } = useUserStore();

// ✅ GOOD - Only subscribe to what you need
function UserList() {
  const users = useUserStore((state) => state.users);
  const removeUser = useUserStore((state) => state.removeUser);
  
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>
          {user.name}
          <button onClick={() => removeUser(user.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}

// Derived/computed selectors
const selectAdminUsers = (state: UserStore) =>
  state.users.filter((u) => u.role === 'admin');

function AdminList() {
  const admins = useUserStore(selectAdminUsers);
  // Only re-renders when admin users change
  return <ul>{admins.map((a) => <li key={a.id}>{a.name}</li>)}</ul>;
}

// ════════════════════════════════════════════════════════════════
// CART STORE EXAMPLE
// ════════════════════════════════════════════════════════════════

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartStore {
  items: CartItem[];
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
  
  // Computed getters
  totalItems: () => number;
  totalPrice: () => number;
}

export const useCartStore = create<CartStore>()(
  devtools(
    persist(
      (set, get) => ({
        items: [],
        
        addItem: (item) =>
          set((state) => {
            const existingItem = state.items.find((i) => i.id === item.id);
            if (existingItem) {
              return {
                items: state.items.map((i) =>
                  i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
                ),
              };
            }
            return { items: [...state.items, { ...item, quantity: 1 }] };
          }),
        
        removeItem: (id) =>
          set((state) => ({
            items: state.items.filter((i) => i.id !== id),
          })),
        
        updateQuantity: (id, quantity) =>
          set((state) => ({
            items: state.items.map((i) =>
              i.id === id ? { ...i, quantity } : i
            ),
          })),
        
        clearCart: () => set({ items: [] }),
        
        // Computed values (called as functions)
        totalItems: () => get().items.reduce((sum, i) => sum + i.quantity, 0),
        totalPrice: () =>
          get().items.reduce((sum, i) => sum + i.price * i.quantity, 0),
      }),
      { name: 'cart-storage' }
    )
  )
);

// Usage
function CartSummary() {
  const totalItems = useCartStore((state) => state.totalItems());
  const totalPrice = useCartStore((state) => state.totalPrice());
  
  return (
    <div>
      <span>{totalItems} items</span>
      <span>${totalPrice.toFixed(2)}</span>
    </div>
  );
}
```

---

## 5. Jotai

```tsx
// ════════════════════════════════════════════════════════════════
// JOTAI - ATOMIC STATE
// ════════════════════════════════════════════════════════════════

import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';
import { atomWithStorage, atomWithReset, RESET } from 'jotai/utils';
import { atomWithQuery } from 'jotai-tanstack-query';

// ════════════════════════════════════════════════════════════════
// PRIMITIVE ATOMS
// ════════════════════════════════════════════════════════════════

// Simple atom
const countAtom = atom(0);

// Atom with localStorage persistence
const themeAtom = atomWithStorage<'light' | 'dark'>('theme', 'light');

// Resettable atom
const formAtom = atomWithReset({
  name: '',
  email: '',
  message: '',
});

// Usage
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  return (
    <button onClick={() => setCount((c) => c + 1)}>
      Count: {count}
    </button>
  );
}

// Read-only hook (no setter)
function CountDisplay() {
  const count = useAtomValue(countAtom);
  return <span>{count}</span>;
}

// Write-only hook (no value)
function IncrementButton() {
  const setCount = useSetAtom(countAtom);
  return <button onClick={() => setCount((c) => c + 1)}>+</button>;
}

// ════════════════════════════════════════════════════════════════
// DERIVED ATOMS
// ════════════════════════════════════════════════════════════════

// Read-only derived atom
const doubleCountAtom = atom((get) => get(countAtom) * 2);

// Read-write derived atom
const countWithMinAtom = atom(
  (get) => get(countAtom),
  (get, set, newValue: number) => {
    set(countAtom, Math.max(0, newValue)); // Minimum of 0
  }
);

// Derived from multiple atoms
const userAtom = atom({ name: 'John', age: 30 });
const settingsAtom = atom({ theme: 'dark', language: 'en' });

const userSummaryAtom = atom((get) => {
  const user = get(userAtom);
  const settings = get(settingsAtom);
  return `${user.name} (${user.age}) - ${settings.theme} theme`;
});

// ════════════════════════════════════════════════════════════════
// ASYNC ATOMS
// ════════════════════════════════════════════════════════════════

// Atom that fetches data
const userDataAtom = atom(async () => {
  const response = await fetch('/api/user');
  return response.json();
});

// Atom with dependency
const userIdAtom = atom(1);

const userByIdAtom = atom(async (get) => {
  const userId = get(userIdAtom);
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
});

// With React Query integration
const usersQueryAtom = atomWithQuery(() => ({
  queryKey: ['users'],
  queryFn: async () => {
    const response = await fetch('/api/users');
    return response.json();
  },
}));

// Usage with Suspense
function UserProfile() {
  const [user] = useAtom(userDataAtom);
  return <div>{user.name}</div>;
}

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <UserProfile />
    </Suspense>
  );
}

// ════════════════════════════════════════════════════════════════
// ATOM FAMILY (Parameterized atoms)
// ════════════════════════════════════════════════════════════════

import { atomFamily } from 'jotai/utils';

// Create atoms dynamically based on parameter
const todoAtomFamily = atomFamily((id: string) =>
  atom({ id, text: '', completed: false })
);

const todoIdsAtom = atom<string[]>([]);

// Derived atom for all todos
const todosAtom = atom((get) => {
  const ids = get(todoIdsAtom);
  return ids.map((id) => get(todoAtomFamily(id)));
});

// Usage
function TodoItem({ id }: { id: string }) {
  const [todo, setTodo] = useAtom(todoAtomFamily(id));
  
  return (
    <div>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={(e) => setTodo({ ...todo, completed: e.target.checked })}
      />
      <span>{todo.text}</span>
    </div>
  );
}

function TodoList() {
  const todos = useAtomValue(todosAtom);
  
  return (
    <ul>
      {todos.map((todo) => (
        <TodoItem key={todo.id} id={todo.id} />
      ))}
    </ul>
  );
}

// ════════════════════════════════════════════════════════════════
// FORM STATE WITH JOTAI
// ════════════════════════════════════════════════════════════════

// Form atoms
const nameAtom = atom('');
const emailAtom = atom('');
const passwordAtom = atom('');

// Derived validation atom
const formValidAtom = atom((get) => {
  const name = get(nameAtom);
  const email = get(emailAtom);
  const password = get(passwordAtom);
  
  return {
    isValid: name.length > 0 && email.includes('@') && password.length >= 8,
    errors: {
      name: name.length === 0 ? 'Name is required' : null,
      email: !email.includes('@') ? 'Invalid email' : null,
      password: password.length < 8 ? 'Password too short' : null,
    },
  };
});

// Reset all form atoms
const resetFormAtom = atom(null, (get, set) => {
  set(nameAtom, '');
  set(emailAtom, '');
  set(passwordAtom, '');
});

function RegistrationForm() {
  const [name, setName] = useAtom(nameAtom);
  const [email, setEmail] = useAtom(emailAtom);
  const [password, setPassword] = useAtom(passwordAtom);
  const { isValid, errors } = useAtomValue(formValidAtom);
  const resetForm = useSetAtom(resetFormAtom);
  
  return (
    <form>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      {errors.name && <span>{errors.name}</span>}
      
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      {errors.email && <span>{errors.email}</span>}
      
      <input 
        type="password" 
        value={password} 
        onChange={(e) => setPassword(e.target.value)} 
      />
      {errors.password && <span>{errors.password}</span>}
      
      <button type="submit" disabled={!isValid}>Submit</button>
      <button type="button" onClick={resetForm}>Reset</button>
    </form>
  );
}
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# STATE MANAGEMENT PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Using Context for frequently updated state
# Bad
<CounterContext.Provider value={{ count, setCount }}>
  {children}  {/* ALL children re-render on count change */}
</CounterContext.Provider>

# Good
# Use Zustand/Jotai for frequent updates
# Or split Context into state/dispatch

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Not using selectors in Zustand
# Bad
const { users, addUser, removeUser, settings, theme } = useStore();
# Component re-renders when ANY state changes

# Good
const users = useStore(state => state.users);
const addUser = useStore(state => state.addUser);
# Only re-renders when users change

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Storing server data in Redux/Zustand
# Bad
# Fetching API data and putting in Redux
# Manually handling loading, errors, caching, refetching

# Good
# Use React Query/SWR for server state
# Redux/Zustand for client-only UI state

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Mutating state directly
# Bad
state.users.push(newUser);  // Direct mutation

# Good
return [...state.users, newUser];  // Immutable update
# Or use Immer middleware

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Prop drilling when not needed
# Bad
<App>
  <Layout user={user} onLogout={onLogout}>
    <Sidebar user={user}>
      <UserInfo user={user} onLogout={onLogout} />
    </Sidebar>
  </Layout>
</App>

# Good
# Use Context or global state for widely-used data
<UserProvider>
  <App>
    <Layout>
      <Sidebar>
        <UserInfo />  {/* Gets user from context */}
      </Sidebar>
    </Layout>
  </App>
</UserProvider>

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Over-engineering with Redux for simple apps
# Bad
# Using Redux for a todo app with 3 components

# Good
# Start with useState/useReducer
# Add global state only when needed
# "Use the simplest solution that works"
```

---

## 7. Interview Questions

### Basic Questions

**Q: "When would you use useState vs useReducer?"**
> "useState for simple, independent state values. useReducer when: state has multiple related values, next state depends on previous, complex update logic, or when you want to extract logic into a reducer. useReducer also gives you dispatch which is stable across renders."

**Q: "What's wrong with using Context for everything?"**
> "Context re-renders ALL consumers when any value changes - no selector support. Fine for infrequent updates (theme, auth), but causes performance issues for frequently-changing state. Use Zustand/Jotai/Redux for that. Also, Context nesting can get messy."

**Q: "What is prop drilling and how do you solve it?"**
> "Passing props through multiple component levels just to reach a deeply nested child. Solutions: 1) Context for widely-used data. 2) Component composition (children prop). 3) Global state management. Choose based on update frequency and scope of data."

### Intermediate Questions

**Q: "Redux vs Zustand - when to use each?"**
> "Redux: Large teams (standardization), complex state, need for time-travel debugging, extensive middleware ecosystem. Zustand: Simpler apps, less boilerplate, when you want Redux patterns without Redux complexity. Zustand is ~1KB, Redux Toolkit ~4KB. Both work well; choose based on team size and complexity."

**Q: "What is selector memoization and why is it important?"**
> "Selectors compute derived state from store. Without memoization, they recalculate on every render. createSelector (Redux) or shallow comparison (Zustand) memoizes - only recalculates when inputs change. Prevents expensive computations and unnecessary re-renders."

**Q: "How do you decide what goes in global state vs local state?"**
> "Global: Shared across distant components, persisted data, app-wide settings. Local: Form inputs before submit, UI state like open/closed, component-specific data. Rule: Keep state as local as possible, lift/globalize only when needed. Server state belongs in React Query, not global store."

### Advanced Questions

**Q: "Explain Jotai's atomic model vs Redux's single store."**
> "Redux: Top-down, single store, slices combine into one tree. Jotai: Bottom-up, independent atoms compose into derived atoms. Atoms enable granular subscriptions - components only re-render when their specific atoms change. Better for complex derived state and code splitting. Trade-off: less predictable than single store."

**Q: "How would you handle state that needs to persist across page refreshes?"**
> "Options: 1) Zustand/Jotai persistence middleware to localStorage. 2) URL state for shareable state (search params). 3) Cookies for auth tokens. 4) IndexedDB for large data. Consider: what needs persistence, security implications, sync on multiple tabs (storage events)."

**Q: "How do you structure state in a large application?"**
> "Separate concerns: 1) Server state → React Query (caching, deduplication). 2) Global UI state → Zustand/Redux (modals, sidebar, theme). 3) Form state → React Hook Form (validation, submission). 4) URL state → router search params. 5) Component state → useState. Each tool for its purpose."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              STATE MANAGEMENT CHECKLIST                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CHOOSING A SOLUTION:                                           │
│  □ Component-local? → useState/useReducer                      │
│  □ Share between siblings? → Lift state up                     │
│  □ Theme/auth (infrequent)? → Context                          │
│  □ Frequent updates shared? → Zustand/Jotai/Redux              │
│  □ Server data? → React Query/SWR                              │
│  □ Forms? → React Hook Form                                    │
│                                                                 │
│  PERFORMANCE:                                                   │
│  □ Use selectors (don't subscribe to entire store)             │
│  □ Memoize derived state                                       │
│  □ Split Context by update frequency                           │
│  □ Colocate state near usage                                   │
│                                                                 │
│  ORGANIZATION:                                                  │
│  □ Separate server state from client state                     │
│  □ Keep state as local as possible                             │
│  □ Use appropriate tool for each type                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

COMPARISON:
┌────────────────────────────────────────────────────────────────┐
│ Context:  Built-in, no selectors, re-renders all consumers    │
│ Redux:    Single store, middleware, devtools, ~4KB            │
│ Zustand:  Simple API, selectors, middleware, ~1KB             │
│ Jotai:    Atomic, bottom-up, granular updates, ~3KB           │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

