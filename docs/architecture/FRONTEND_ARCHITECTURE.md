# 🎨 CareCircle Frontend Architecture - Complete Guide

> Deep dive into the Next.js 14 frontend, React patterns, and the "Warm Hearth" design system.

---

## Table of Contents

1. [Project Structure](#1-project-structure)
2. [App Router & Layouts](#2-app-router--layouts)
3. [Component Architecture](#3-component-architecture)
4. [State Management](#4-state-management)
5. [API Integration](#5-api-integration)
6. [Real-Time Updates](#6-real-time-updates)
7. [PWA & Offline Support](#7-pwa--offline-support)
8. [Design System](#8-design-system)

---

## 1. Project Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      FRONTEND PROJECT STRUCTURE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   apps/web/src/                                                             │
│   │                                                                         │
│   ├── app/                           # Next.js App Router                   │
│   │   ├── layout.tsx                 # Root layout (providers)              │
│   │   ├── page.tsx                   # Landing page (/)                     │
│   │   ├── globals.css                # Global styles                        │
│   │   │                                                                     │
│   │   ├── (auth)/                    # Auth route group                     │
│   │   │   ├── layout.tsx             # Centered layout                      │
│   │   │   ├── login/page.tsx         # /login                               │
│   │   │   ├── register/page.tsx      # /register                            │
│   │   │   ├── forgot-password/page.tsx                                      │
│   │   │   └── accept-invite/                                                │
│   │   │       └── [token]/page.tsx   # /accept-invite/:token                │
│   │   │                                                                     │
│   │   └── (app)/                     # Protected app routes                 │
│   │       ├── layout.tsx             # App shell (sidebar + nav)            │
│   │       ├── page.tsx               # Dashboard (/)                        │
│   │       ├── care/                                                         │
│   │       │   └── [id]/              # Care recipient routes                │
│   │       │       ├── page.tsx       # Overview                             │
│   │       │       ├── medications/page.tsx                                  │
│   │       │       ├── calendar/page.tsx                                     │
│   │       │       ├── documents/page.tsx                                    │
│   │       │       ├── emergency/page.tsx                                    │
│   │       │       └── timeline/page.tsx                                     │
│   │       ├── family/page.tsx                                               │
│   │       └── settings/page.tsx                                             │
│   │                                                                         │
│   ├── components/                    # React Components                     │
│   │   ├── ui/                        # Base UI components                   │
│   │   │   ├── button.tsx                                                    │
│   │   │   ├── input.tsx                                                     │
│   │   │   ├── card.tsx                                                      │
│   │   │   ├── modal.tsx                                                     │
│   │   │   ├── toast.tsx                                                     │
│   │   │   ├── badge.tsx                                                     │
│   │   │   ├── skeleton.tsx                                                  │
│   │   │   └── index.ts                                                      │
│   │   │                                                                     │
│   │   ├── layout/                    # Layout components                    │
│   │   │   ├── app-shell.tsx                                                 │
│   │   │   ├── sidebar.tsx                                                   │
│   │   │   ├── mobile-nav.tsx                                                │
│   │   │   └── header.tsx                                                    │
│   │   │                                                                     │
│   │   ├── care/                      # Domain components                    │
│   │   │   ├── care-recipient-card.tsx                                       │
│   │   │   ├── medication-card.tsx                                           │
│   │   │   ├── appointment-card.tsx                                          │
│   │   │   ├── timeline-entry.tsx                                            │
│   │   │   └── emergency-button.tsx                                          │
│   │   │                                                                     │
│   │   ├── forms/                     # Form components                      │
│   │   │   ├── login-form.tsx                                                │
│   │   │   ├── register-form.tsx                                             │
│   │   │   ├── medication-form.tsx                                           │
│   │   │   └── appointment-form.tsx                                          │
│   │   │                                                                     │
│   │   └── providers/                 # Context providers                    │
│   │       ├── auth-provider.tsx                                             │
│   │       ├── query-provider.tsx                                            │
│   │       └── websocket-provider.tsx                                        │
│   │                                                                         │
│   ├── hooks/                         # Custom React hooks                   │
│   │   ├── use-auth.ts                                                       │
│   │   ├── use-medications.ts                                                │
│   │   ├── use-appointments.ts                                               │
│   │   ├── use-websocket.ts                                                  │
│   │   ├── use-offline.ts                                                    │
│   │   └── use-media-query.ts                                                │
│   │                                                                         │
│   ├── lib/                           # Utilities & API                      │
│   │   ├── api/                                                              │
│   │   │   ├── client.ts              # Axios instance                       │
│   │   │   ├── auth.ts                                                       │
│   │   │   ├── medications.ts                                                │
│   │   │   └── ...                                                           │
│   │   ├── utils.ts                   # Helper functions                     │
│   │   ├── constants.ts                                                      │
│   │   └── offline-storage.ts         # LocalForage wrapper                  │
│   │                                                                         │
│   └── styles/                                                               │
│       └── design-tokens.ts           # Theme configuration                  │
│                                                                             │
│   public/                                                                   │
│   ├── sw.js                          # Service worker                       │
│   ├── manifest.json                  # PWA manifest                         │
│   └── icons/                         # App icons                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. App Router & Layouts

### Route Groups

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      ROUTE GROUPS EXPLAINED                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Route groups use (parentheses) to organize routes without affecting URLs: │
│                                                                             │
│   app/                                                                      │
│   ├── (auth)/          ← Group: shared centered layout                      │
│   │   ├── layout.tsx   ← Centered card layout                               │
│   │   ├── login/       → URL: /login                                        │
│   │   └── register/    → URL: /register                                     │
│   │                                                                         │
│   └── (app)/           ← Group: shared app shell layout                     │
│       ├── layout.tsx   ← Sidebar + navigation                               │
│       ├── page.tsx     → URL: / (dashboard)                                 │
│       └── care/[id]/   → URL: /care/cr_abc123                               │
│                                                                             │
│   Benefits:                                                                 │
│   • Different layouts for auth vs app pages                                 │
│   • URL structure unchanged                                                 │
│   • Shared state within groups                                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Layout Hierarchy

```tsx
// app/layout.tsx - ROOT LAYOUT
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body className="font-sans bg-bg-base text-text-primary">
        <QueryProvider>
          <AuthProvider>
            <WebSocketProvider>
              <ToastProvider>
                {children}
              </ToastProvider>
            </WebSocketProvider>
          </AuthProvider>
        </QueryProvider>
      </body>
    </html>
  );
}

// app/(auth)/layout.tsx - AUTH LAYOUT
export default function AuthLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-screen flex items-center justify-center bg-bg-base">
      <div className="w-full max-w-md p-8">
        <div className="text-center mb-8">
          <Logo className="w-12 h-12 mx-auto" />
          <h1 className="text-2xl font-semibold mt-4">CareCircle</h1>
        </div>
        <Card className="p-8">
          {children}
        </Card>
      </div>
    </div>
  );
}

// app/(app)/layout.tsx - APP LAYOUT
export default function AppLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-screen flex">
      {/* Desktop Sidebar */}
      <aside className="hidden lg:flex w-64 flex-col border-r border-border-subtle">
        <Sidebar />
      </aside>
      
      {/* Main Content */}
      <main className="flex-1 flex flex-col">
        <Header />
        <div className="flex-1 p-6 pb-20 lg:pb-6">
          {children}
        </div>
      </main>
      
      {/* Mobile Navigation */}
      <MobileNav className="lg:hidden" />
      
      {/* Emergency Button - Always visible */}
      <EmergencyButton />
    </div>
  );
}
```

### Dynamic Routes

```tsx
// app/(app)/care/[id]/medications/page.tsx

interface PageProps {
  params: { id: string };  // Care recipient ID
  searchParams: { date?: string };
}

export default async function MedicationsPage({ params, searchParams }: PageProps) {
  const { id: careRecipientId } = params;
  const date = searchParams.date || new Date().toISOString().split('T')[0];

  return (
    <div className="space-y-6">
      <PageHeader
        title="Medications"
        actions={
          <Button onClick={() => openModal('add-medication')}>
            <Plus className="w-4 h-4 mr-2" />
            Add Medication
          </Button>
        }
      />
      
      <MedicationSchedule
        careRecipientId={careRecipientId}
        date={date}
      />
      
      <AllMedicationsList careRecipientId={careRecipientId} />
    </div>
  );
}
```

---

## 3. Component Architecture

### Component Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      COMPONENT HIERARCHY                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Level 1: UI PRIMITIVES (components/ui/)                                   │
│   ─────────────────────────────────────────                                 │
│   Pure, reusable, no business logic                                         │
│   • Button, Input, Card, Badge, Modal, Toast                                │
│   • Follow design system tokens exactly                                     │
│   • No API calls or state management                                        │
│                                                                             │
│   Level 2: LAYOUT COMPONENTS (components/layout/)                           │
│   ───────────────────────────────────────────────                           │
│   Structure and navigation                                                  │
│   • AppShell, Sidebar, Header, MobileNav                                    │
│   • May use auth context for user info                                      │
│   • No domain-specific logic                                                │
│                                                                             │
│   Level 3: DOMAIN COMPONENTS (components/care/)                             │
│   ─────────────────────────────────────────────                             │
│   Business logic, composed of primitives                                    │
│   • MedicationCard, AppointmentCard, TimelineEntry                          │
│   • Use React Query hooks                                                   │
│   • Handle mutations (optimistic updates)                                   │
│                                                                             │
│   Level 4: FORM COMPONENTS (components/forms/)                              │
│   ─────────────────────────────────────────────                             │
│   Complete forms with validation                                            │
│   • LoginForm, MedicationForm, AppointmentForm                              │
│   • Use react-hook-form + Zod                                               │
│   • Handle submission and errors                                            │
│                                                                             │
│   Level 5: PAGE COMPONENTS (app/**/page.tsx)                                │
│   ──────────────────────────────────────────                                │
│   Full page layouts                                                         │
│   • Compose domain and form components                                      │
│   • Handle page-level state                                                 │
│   • Server components where possible                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### UI Component Example

```tsx
// components/ui/button.tsx

import { cva, type VariantProps } from 'class-variance-authority';
import { Loader2 } from 'lucide-react';
import { forwardRef } from 'react';
import { cn } from '@/lib/utils';

const buttonVariants = cva(
  // Base styles
  'inline-flex items-center justify-center rounded-lg font-medium transition-colors ' +
  'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-border-focus ' +
  'disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        primary: 'bg-accent-primary text-text-inverse hover:bg-accent-primary-hover',
        secondary: 'bg-bg-surface text-text-primary border border-border-default hover:bg-bg-subtle',
        ghost: 'hover:bg-bg-subtle text-text-secondary',
        warm: 'bg-accent-warm text-text-inverse hover:bg-accent-warm-hover',
        danger: 'bg-error text-text-inverse hover:bg-error/90',
        emergency: 'bg-emergency text-text-inverse hover:bg-emergency-dark animate-pulse',
      },
      size: {
        sm: 'h-9 px-3.5 text-sm',
        md: 'h-11 px-4.5 text-base',
        lg: 'h-13 px-6 text-lg',
        xl: 'h-15 px-8 text-xl',
        icon: 'h-11 w-11',
      },
    },
    defaultVariants: {
      variant: 'primary',
      size: 'md',
    },
  }
);

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  isLoading?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, isLoading, leftIcon, rightIcon, children, ...props }, ref) => {
    return (
      <button
        ref={ref}
        className={cn(buttonVariants({ variant, size, className }))}
        disabled={isLoading || props.disabled}
        {...props}
      >
        {isLoading ? (
          <Loader2 className="w-4 h-4 mr-2 animate-spin" />
        ) : leftIcon ? (
          <span className="mr-2">{leftIcon}</span>
        ) : null}
        {children}
        {rightIcon && !isLoading && <span className="ml-2">{rightIcon}</span>}
      </button>
    );
  }
);

Button.displayName = 'Button';
```

### Domain Component Example

```tsx
// components/care/medication-card.tsx

'use client';

import { useState } from 'react';
import { Check, X, Clock, AlertCircle } from 'lucide-react';
import { Button, Card, Badge, Modal } from '@/components/ui';
import { useLogMedication } from '@/hooks/use-medications';
import { cn } from '@/lib/utils';
import type { MedicationScheduleItem } from '@/lib/api/medications';

interface MedicationCardProps {
  item: MedicationScheduleItem;
}

export function MedicationCard({ item }: MedicationCardProps) {
  const [showSkipModal, setShowSkipModal] = useState(false);
  const { mutate: logMed, isPending } = useLogMedication();

  const { medication, scheduledTime, status, log } = item;

  const handleGiven = () => {
    logMed({
      medicationId: medication.id,
      status: 'GIVEN',
      scheduledTime,
    });
  };

  const handleSkip = (reason: string) => {
    logMed({
      medicationId: medication.id,
      status: 'SKIPPED',
      scheduledTime,
      skipReason: reason,
    });
    setShowSkipModal(false);
  };

  const statusStyles = {
    PENDING: 'border-l-border-default',
    GIVEN: 'border-l-success bg-success-light/30',
    SKIPPED: 'border-l-warning bg-warning-light/30',
    MISSED: 'border-l-error bg-error-light/30',
  };

  return (
    <>
      <Card className={cn('border-l-4', statusStyles[status])}>
        <div className="flex items-start justify-between">
          <div className="space-y-1">
            <div className="flex items-center gap-2">
              <h3 className="font-semibold text-text-primary">
                {medication.name}
              </h3>
              <Badge variant="default">{medication.dosage}</Badge>
            </div>
            
            <p className="text-sm text-text-secondary">
              {medication.instructions}
            </p>
            
            <div className="flex items-center gap-2 text-sm text-text-tertiary">
              <Clock className="w-4 h-4" />
              <span>
                {new Date(scheduledTime).toLocaleTimeString([], {
                  hour: '2-digit',
                  minute: '2-digit',
                })}
              </span>
            </div>
          </div>

          {status === 'PENDING' && (
            <div className="flex gap-2">
              <Button
                size="sm"
                variant="primary"
                onClick={handleGiven}
                isLoading={isPending}
                leftIcon={<Check className="w-4 h-4" />}
              >
                Given
              </Button>
              <Button
                size="sm"
                variant="secondary"
                onClick={() => setShowSkipModal(true)}
                leftIcon={<X className="w-4 h-4" />}
              >
                Skip
              </Button>
            </div>
          )}

          {status === 'GIVEN' && log && (
            <div className="text-sm text-success">
              <Check className="w-5 h-5 inline mr-1" />
              Given by {log.loggedBy.fullName}
            </div>
          )}

          {status === 'SKIPPED' && log && (
            <div className="text-sm text-warning">
              <AlertCircle className="w-5 h-5 inline mr-1" />
              Skipped: {log.skipReason}
            </div>
          )}
        </div>
      </Card>

      <SkipReasonModal
        isOpen={showSkipModal}
        onClose={() => setShowSkipModal(false)}
        onConfirm={handleSkip}
        medicationName={medication.name}
      />
    </>
  );
}
```

---

## 4. State Management

### State Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      STATE MANAGEMENT STRATEGY                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   We use different tools for different types of state:                      │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ SERVER STATE (API data)                                             │   │
│   │ ═════════════════════════                                           │   │
│   │ Tool: TanStack Query (React Query)                                  │   │
│   │                                                                     │   │
│   │ • Medications, appointments, care recipients                        │   │
│   │ • Cached with automatic invalidation                                │   │
│   │ • Background refetching                                             │   │
│   │ • Optimistic updates for mutations                                  │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ CLIENT STATE (UI state)                                             │   │
│   │ ══════════════════════                                              │   │
│   │ Tool: React useState / useReducer                                   │   │
│   │                                                                     │   │
│   │ • Modal open/closed                                                 │   │
│   │ • Form inputs                                                       │   │
│   │ • Selected items                                                    │   │
│   │ • Temporary UI state                                                │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ GLOBAL STATE (shared across app)                                    │   │
│   │ ═══════════════════════════════                                     │   │
│   │ Tool: React Context                                                 │   │
│   │                                                                     │   │
│   │ • Current user (AuthContext)                                        │   │
│   │ • WebSocket connection (WebSocketContext)                           │   │
│   │ • Theme/preferences                                                 │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ PERSISTENT STATE (survives refresh)                                 │   │
│   │ ═══════════════════════════════════                                 │   │
│   │ Tool: LocalForage (IndexedDB)                                       │   │
│   │                                                                     │   │
│   │ • Offline data cache                                                │   │
│   │ • Pending offline actions                                           │   │
│   │ • Emergency info                                                    │   │
│   │ • User preferences                                                  │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Auth Context

```tsx
// components/providers/auth-provider.tsx

'use client';

import { createContext, useContext, useEffect, useState } from 'react';
import { useRouter } from 'next/navigation';
import { authApi, type User } from '@/lib/api/auth';

interface AuthContextValue {
  user: User | null;
  isLoading: boolean;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  register: (data: RegisterData) => Promise<void>;
  logout: () => Promise<void>;
  refreshUser: () => Promise<void>;
}

const AuthContext = createContext<AuthContextValue | null>(null);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const router = useRouter();

  // Check auth status on mount
  useEffect(() => {
    checkAuth();
  }, []);

  const checkAuth = async () => {
    try {
      const user = await authApi.getMe();
      setUser(user);
    } catch {
      setUser(null);
    } finally {
      setIsLoading(false);
    }
  };

  const login = async (email: string, password: string) => {
    const { user } = await authApi.login({ email, password });
    setUser(user);
    router.push('/');
  };

  const register = async (data: RegisterData) => {
    const { user } = await authApi.register(data);
    setUser(user);
    router.push('/');
  };

  const logout = async () => {
    await authApi.logout();
    setUser(null);
    router.push('/login');
  };

  const refreshUser = async () => {
    const user = await authApi.getMe();
    setUser(user);
  };

  return (
    <AuthContext.Provider
      value={{
        user,
        isLoading,
        isAuthenticated: !!user,
        login,
        register,
        logout,
        refreshUser,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

---

## 5. API Integration

### API Client Setup

```tsx
// lib/api/client.ts

import axios, { AxiosError } from 'axios';

const API_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:3001/api/v1';

export const api = axios.create({
  baseURL: API_URL,
  withCredentials: true, // Send cookies with every request
  headers: {
    'Content-Type': 'application/json',
  },
});

// Response interceptor for token refresh
let isRefreshing = false;
let failedQueue: Array<{
  resolve: (value?: unknown) => void;
  reject: (reason?: any) => void;
}> = [];

const processQueue = (error: Error | null) => {
  failedQueue.forEach((prom) => {
    if (error) {
      prom.reject(error);
    } else {
      prom.resolve();
    }
  });
  failedQueue = [];
};

api.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    const originalRequest = error.config as any;

    // If 401 and not already retrying
    if (error.response?.status === 401 && !originalRequest._retry) {
      if (isRefreshing) {
        // Queue the request while refreshing
        return new Promise((resolve, reject) => {
          failedQueue.push({ resolve, reject });
        }).then(() => api(originalRequest));
      }

      originalRequest._retry = true;
      isRefreshing = true;

      try {
        await api.post('/auth/refresh');
        processQueue(null);
        return api(originalRequest);
      } catch (refreshError) {
        processQueue(refreshError as Error);
        // Redirect to login
        if (typeof window !== 'undefined') {
          window.location.href = '/login';
        }
        return Promise.reject(refreshError);
      } finally {
        isRefreshing = false;
      }
    }

    return Promise.reject(error);
  }
);

export default api;
```

### React Query Hooks

```tsx
// hooks/use-medications.ts

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { medicationsApi } from '@/lib/api/medications';
import { toast } from '@/components/ui/toast';

// Query keys factory
export const medicationKeys = {
  all: ['medications'] as const,
  lists: () => [...medicationKeys.all, 'list'] as const,
  list: (recipientId: string) => [...medicationKeys.lists(), recipientId] as const,
  schedules: () => [...medicationKeys.all, 'schedule'] as const,
  schedule: (recipientId: string, date: string) =>
    [...medicationKeys.schedules(), recipientId, date] as const,
  details: () => [...medicationKeys.all, 'detail'] as const,
  detail: (id: string) => [...medicationKeys.details(), id] as const,
};

// Fetch medications for a care recipient
export function useMedications(recipientId: string) {
  return useQuery({
    queryKey: medicationKeys.list(recipientId),
    queryFn: () => medicationsApi.getAll(recipientId),
    staleTime: 1000 * 60 * 5, // 5 minutes
  });
}

// Fetch today's medication schedule
export function useMedicationSchedule(recipientId: string, date: string) {
  return useQuery({
    queryKey: medicationKeys.schedule(recipientId, date),
    queryFn: () => medicationsApi.getSchedule(recipientId, date),
    staleTime: 1000 * 60, // 1 minute (schedule changes more often)
    refetchInterval: 1000 * 60, // Refetch every minute
  });
}

// Log medication mutation with optimistic update
export function useLogMedication() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: medicationsApi.logMedication,

    // Optimistic update
    onMutate: async (variables) => {
      const { medicationId, status, scheduledTime } = variables;

      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: medicationKeys.schedules() });

      // Snapshot current state
      const previousSchedules = queryClient.getQueriesData({
        queryKey: medicationKeys.schedules(),
      });

      // Optimistically update
      queryClient.setQueriesData(
        { queryKey: medicationKeys.schedules() },
        (old: any) => {
          if (!old) return old;
          return old.map((item: any) =>
            item.medication.id === medicationId &&
            item.scheduledTime === scheduledTime
              ? { ...item, status, isPending: true }
              : item
          );
        }
      );

      return { previousSchedules };
    },

    // Rollback on error
    onError: (err, variables, context) => {
      if (context?.previousSchedules) {
        context.previousSchedules.forEach(([queryKey, data]) => {
          queryClient.setQueryData(queryKey, data);
        });
      }
      toast.error('Failed to log medication. Please try again.');
    },

    // Refetch on success
    onSuccess: (data, variables) => {
      toast.success(
        variables.status === 'GIVEN'
          ? 'Medication marked as given!'
          : 'Medication skipped'
      );
    },

    // Always refetch after mutation
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: medicationKeys.schedules() });
    },
  });
}

// Create medication mutation
export function useCreateMedication() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: medicationsApi.create,
    onSuccess: (data, variables) => {
      queryClient.invalidateQueries({
        queryKey: medicationKeys.list(variables.careRecipientId),
      });
      toast.success('Medication added successfully!');
    },
    onError: () => {
      toast.error('Failed to add medication.');
    },
  });
}
```

---

## 6. Real-Time Updates

### WebSocket Provider

```tsx
// components/providers/websocket-provider.tsx

'use client';

import { createContext, useContext, useEffect, useState } from 'react';
import { io, Socket } from 'socket.io-client';
import { useQueryClient } from '@tanstack/react-query';
import { useAuth } from './auth-provider';
import { toast } from '@/components/ui/toast';
import { medicationKeys } from '@/hooks/use-medications';

interface WebSocketContextValue {
  socket: Socket | null;
  isConnected: boolean;
}

const WebSocketContext = createContext<WebSocketContextValue>({
  socket: null,
  isConnected: false,
});

export function WebSocketProvider({ children }: { children: React.ReactNode }) {
  const [socket, setSocket] = useState<Socket | null>(null);
  const [isConnected, setIsConnected] = useState(false);
  const { user, isAuthenticated } = useAuth();
  const queryClient = useQueryClient();

  useEffect(() => {
    if (!isAuthenticated) {
      socket?.disconnect();
      setSocket(null);
      return;
    }

    const newSocket = io(process.env.NEXT_PUBLIC_WS_URL || 'http://localhost:3001', {
      path: '/carecircle',
      withCredentials: true,
      transports: ['websocket', 'polling'],
    });

    newSocket.on('connect', () => {
      console.log('WebSocket connected');
      setIsConnected(true);
    });

    newSocket.on('disconnect', () => {
      console.log('WebSocket disconnected');
      setIsConnected(false);
    });

    // Domain event handlers
    newSocket.on('medication.logged', (data) => {
      // Invalidate medication queries
      queryClient.invalidateQueries({ queryKey: medicationKeys.schedules() });

      // Show toast if it wasn't me
      if (data.loggedById !== user?.id) {
        toast.info(
          `${data.loggedByName} ${data.status === 'GIVEN' ? 'gave' : 'skipped'} ${data.medicationName}`
        );
      }
    });

    newSocket.on('appointment.created', (data) => {
      queryClient.invalidateQueries({ queryKey: ['appointments'] });
      if (data.createdById !== user?.id) {
        toast.info(`New appointment: ${data.title}`);
      }
    });

    newSocket.on('emergency.alert.created', (data) => {
      // Critical: Show emergency modal
      showEmergencyAlert(data);
    });

    newSocket.on('shift.update', (data) => {
      queryClient.invalidateQueries({ queryKey: ['shifts'] });
    });

    setSocket(newSocket);

    return () => {
      newSocket.disconnect();
    };
  }, [isAuthenticated, user?.id, queryClient]);

  return (
    <WebSocketContext.Provider value={{ socket, isConnected }}>
      {children}
    </WebSocketContext.Provider>
  );
}

export function useWebSocket() {
  return useContext(WebSocketContext);
}
```

### Connection Status Indicator

```tsx
// components/layout/connection-status.tsx

'use client';

import { Wifi, WifiOff } from 'lucide-react';
import { useWebSocket } from '@/components/providers/websocket-provider';
import { useOnlineStatus } from '@/hooks/use-online-status';

export function ConnectionStatus() {
  const { isConnected } = useWebSocket();
  const isOnline = useOnlineStatus();

  if (!isOnline) {
    return (
      <div className="flex items-center gap-2 text-warning text-sm">
        <WifiOff className="w-4 h-4" />
        <span>Offline - Changes will sync when back online</span>
      </div>
    );
  }

  if (!isConnected) {
    return (
      <div className="flex items-center gap-2 text-warning text-sm">
        <div className="w-2 h-2 rounded-full bg-warning animate-pulse" />
        <span>Reconnecting...</span>
      </div>
    );
  }

  return (
    <div className="flex items-center gap-2 text-success text-sm">
      <div className="w-2 h-2 rounded-full bg-success" />
      <span>Connected</span>
    </div>
  );
}
```

---

## 7. PWA & Offline Support

### Service Worker

```javascript
// public/sw.js

const CACHE_NAME = 'carecircle-v1';
const OFFLINE_URLS = [
  '/',
  '/offline',
  '/manifest.json',
  '/icons/icon-192.png',
  '/icons/icon-512.png',
];

// Install: Cache critical assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll(OFFLINE_URLS);
    })
  );
  self.skipWaiting();
});

// Activate: Clean old caches
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((keys) => {
      return Promise.all(
        keys.filter((key) => key !== CACHE_NAME).map((key) => caches.delete(key))
      );
    })
  );
  self.clients.claim();
});

// Fetch: Network-first with cache fallback
self.addEventListener('fetch', (event) => {
  // Skip non-GET requests
  if (event.request.method !== 'GET') return;

  // Skip API requests (handled differently)
  if (event.request.url.includes('/api/')) return;

  event.respondWith(
    fetch(event.request)
      .then((response) => {
        // Cache successful responses
        if (response.status === 200) {
          const clone = response.clone();
          caches.open(CACHE_NAME).then((cache) => {
            cache.put(event.request, clone);
          });
        }
        return response;
      })
      .catch(() => {
        // Return cached version or offline page
        return caches.match(event.request).then((cached) => {
          return cached || caches.match('/offline');
        });
      })
  );
});

// Push notifications
self.addEventListener('push', (event) => {
  const data = event.data?.json() || {};

  const options = {
    body: data.body || 'New notification',
    icon: '/icons/icon-192.png',
    badge: '/icons/badge-72.png',
    vibrate: [200, 100, 200],
    tag: data.tag || 'default',
    data: data.data || {},
    actions: data.actions || [],
  };

  // Emergency notifications get priority
  if (data.type === 'emergency') {
    options.requireInteraction = true;
    options.vibrate = [500, 200, 500, 200, 500];
  }

  event.waitUntil(
    self.registration.showNotification(data.title || 'CareCircle', options)
  );
});

// Notification click handler
self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  const url = event.notification.data?.url || '/';

  event.waitUntil(
    clients.matchAll({ type: 'window' }).then((windowClients) => {
      // Focus existing window or open new one
      for (const client of windowClients) {
        if (client.url === url && 'focus' in client) {
          return client.focus();
        }
      }
      return clients.openWindow(url);
    })
  );
});
```

### Offline Hook

```tsx
// hooks/use-offline.ts

import { useState, useEffect } from 'react';
import localforage from 'localforage';

// Configure localforage
localforage.config({
  name: 'CareCircle',
  storeName: 'offline_data',
});

interface OfflineAction {
  id: string;
  type: string;
  payload: any;
  queuedAt: number;
}

export function useOfflineSync() {
  const [pendingActions, setPendingActions] = useState<OfflineAction[]>([]);
  const [isSyncing, setIsSyncing] = useState(false);

  // Load pending actions on mount
  useEffect(() => {
    loadPendingActions();
  }, []);

  // Sync when back online
  useEffect(() => {
    const handleOnline = () => {
      syncPendingActions();
    };

    window.addEventListener('online', handleOnline);
    return () => window.removeEventListener('online', handleOnline);
  }, [pendingActions]);

  const loadPendingActions = async () => {
    const actions = await localforage.getItem<OfflineAction[]>('pending_actions');
    setPendingActions(actions || []);
  };

  const queueAction = async (type: string, payload: any) => {
    const action: OfflineAction = {
      id: crypto.randomUUID(),
      type,
      payload,
      queuedAt: Date.now(),
    };

    const updated = [...pendingActions, action];
    await localforage.setItem('pending_actions', updated);
    setPendingActions(updated);

    // Try to sync immediately if online
    if (navigator.onLine) {
      syncPendingActions();
    }
  };

  const syncPendingActions = async () => {
    if (isSyncing || pendingActions.length === 0) return;

    setIsSyncing(true);

    const remaining: OfflineAction[] = [];

    for (const action of pendingActions) {
      try {
        await executeAction(action);
      } catch (error) {
        // Keep failed actions for retry
        remaining.push(action);
      }
    }

    await localforage.setItem('pending_actions', remaining);
    setPendingActions(remaining);
    setIsSyncing(false);
  };

  const executeAction = async (action: OfflineAction) => {
    // Execute based on action type
    switch (action.type) {
      case 'LOG_MEDICATION':
        await medicationsApi.logMedication(action.payload);
        break;
      case 'CREATE_TIMELINE_ENTRY':
        await timelineApi.create(action.payload);
        break;
      // Add more action types...
    }
  };

  return {
    pendingActions,
    queueAction,
    syncPendingActions,
    isSyncing,
    hasPendingActions: pendingActions.length > 0,
  };
}

// Cache emergency info for offline access
export async function cacheEmergencyInfo(recipientId: string, info: EmergencyInfo) {
  await localforage.setItem(`emergency_${recipientId}`, info);
}

export async function getOfflineEmergencyInfo(recipientId: string) {
  return localforage.getItem<EmergencyInfo>(`emergency_${recipientId}`);
}
```

---

## 8. Design System

### Tailwind Configuration

```typescript
// tailwind.config.ts

import type { Config } from 'tailwindcss';

const config: Config = {
  content: ['./src/**/*.{js,ts,jsx,tsx,mdx}'],
  theme: {
    extend: {
      colors: {
        // Background colors
        bg: {
          base: '#FAFAF8',
          surface: '#FFFFFF',
          muted: '#F5F5F3',
          subtle: '#EEEDE9',
          inverse: '#1A1A18',
        },
        // Border colors
        border: {
          subtle: '#E8E7E3',
          DEFAULT: '#D9D8D4',
          strong: '#C4C3BF',
          focus: '#2D5A4A',
        },
        // Text colors
        text: {
          primary: '#1A1A18',
          secondary: '#5C5C58',
          tertiary: '#8A8A86',
          disabled: '#B5B5B1',
          inverse: '#FAFAF8',
          link: '#2D5A4A',
        },
        // Accent colors
        accent: {
          primary: '#2D5A4A',
          'primary-hover': '#234839',
          'primary-light': '#E8F0ED',
          warm: '#C4725C',
          'warm-hover': '#A85E4A',
          'warm-light': '#F9EFEC',
        },
        // Semantic colors
        success: {
          DEFAULT: '#3D8B6E',
          light: '#E8F5EF',
        },
        warning: {
          DEFAULT: '#C49A3D',
          light: '#FBF5E8',
        },
        error: {
          DEFAULT: '#C45C5C',
          light: '#FCEAEA',
        },
        info: {
          DEFAULT: '#5C8AC4',
          light: '#EDF3FB',
        },
        // Emergency
        emergency: {
          DEFAULT: '#D32F2F',
          light: '#FFEBEE',
          dark: '#B71C1C',
        },
      },
      fontFamily: {
        sans: ['Inter', '-apple-system', 'sans-serif'],
        serif: ['Source Serif 4', 'Georgia', 'serif'],
      },
      borderRadius: {
        sm: '6px',
        md: '8px',
        lg: '12px',
        xl: '16px',
        '2xl': '24px',
      },
      boxShadow: {
        xs: '0 1px 2px rgba(26, 26, 24, 0.04)',
        sm: '0 2px 4px rgba(26, 26, 24, 0.06)',
        md: '0 4px 12px rgba(26, 26, 24, 0.08)',
        lg: '0 8px 24px rgba(26, 26, 24, 0.12)',
        xl: '0 16px 48px rgba(26, 26, 24, 0.16)',
        focus: '0 0 0 3px rgba(45, 90, 74, 0.2)',
      },
      animation: {
        'fade-in': 'fadeIn 200ms ease-out',
        'slide-up': 'slideUp 250ms cubic-bezier(0.34, 1.56, 0.64, 1)',
        'pulse-slow': 'pulse 2s ease-in-out infinite',
      },
    },
  },
  plugins: [],
};

export default config;
```

### Global Styles

```css
/* app/globals.css */

@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  /* Focus visible for keyboard navigation */
  *:focus-visible {
    @apply outline-none ring-2 ring-border-focus ring-offset-2;
  }

  /* Default body styles */
  body {
    @apply bg-bg-base text-text-primary antialiased;
  }

  /* Smooth scrolling */
  html {
    scroll-behavior: smooth;
  }

  /* Selection color */
  ::selection {
    @apply bg-accent-primary-light text-accent-primary;
  }
}

@layer components {
  /* Card hover effect */
  .card-interactive {
    @apply transition-all duration-200;
    @apply hover:shadow-md hover:-translate-y-0.5;
  }

  /* Touch target minimum size */
  .touch-target {
    @apply min-h-[44px] min-w-[44px];
  }

  /* Screen reader only */
  .sr-only {
    @apply absolute w-px h-px p-0 -m-px overflow-hidden whitespace-nowrap border-0;
    clip: rect(0, 0, 0, 0);
  }
}

/* Reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## Summary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          KEY TAKEAWAYS                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   1. APP ROUTER STRUCTURE                                                   │
│      • Route groups for different layouts                                   │
│      • Dynamic routes for care recipients                                   │
│      • Server and client components                                         │
│                                                                             │
│   2. COMPONENT ARCHITECTURE                                                 │
│      • UI primitives → Layout → Domain → Forms → Pages                      │
│      • Design system with Tailwind + CVA                                    │
│      • Consistent prop patterns                                             │
│                                                                             │
│   3. STATE MANAGEMENT                                                       │
│      • React Query for server state                                         │
│      • Context for global client state                                      │
│      • LocalForage for offline persistence                                  │
│                                                                             │
│   4. API INTEGRATION                                                        │
│      • Axios with interceptors for auth                                     │
│      • React Query hooks with optimistic updates                            │
│      • Error handling and retries                                           │
│                                                                             │
│   5. REAL-TIME FEATURES                                                     │
│      • Socket.io for WebSocket                                              │
│      • Cache invalidation on events                                         │
│      • Connection status indicator                                          │
│                                                                             │
│   6. PWA & OFFLINE                                                          │
│      • Service worker for caching                                           │
│      • Offline action queue                                                 │
│      • Push notifications                                                   │
│                                                                             │
│   7. DESIGN SYSTEM                                                          │
│      • "Warm Hearth" aesthetic                                              │
│      • Tailwind design tokens                                               │
│      • Accessible, mobile-first                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

**Related Guides:**

- [COMPLETE_LEARNING_GUIDE.md](./COMPLETE_LEARNING_GUIDE.md) - Project overview
- [API_ARCHITECTURE.md](./API_ARCHITECTURE.md) - Backend architecture
- [AUTH_COMPLETE_GUIDE.md](./AUTH_COMPLETE_GUIDE.md) - Authentication details

---

_CareCircle Frontend: Warm, accessible, real-time. 🎨_

