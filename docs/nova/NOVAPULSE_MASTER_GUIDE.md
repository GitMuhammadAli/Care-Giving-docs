# NovaPulse: The Only Guide You Need

**One document. Follow it top to bottom. That's it.**

---

## What You're Building

**NovaPulse** = Production-ready SaaS Admin & Operations Platform (like Linear / Notion / Vercel)

```
┌─────────────────────────────────────────────────────────────────┐
│                     NovaPulse Platform                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐        │
│   │   Company   │    │   Company   │    │   Company   │        │
│   │      A      │    │      B      │    │      C      │        │
│   └──────┬──────┘    └──────┬──────┘    └──────┬──────┘        │
│          │                  │                  │                │
│   ┌──────┴──────┐    ┌──────┴──────┐    ┌──────┴──────┐        │
│   │ Admin       │    │ Admin       │    │ Admin       │        │
│   │ Manager     │    │ Manager     │    │ Manager     │        │
│   │ User        │    │ User        │    │ User        │        │
│   └─────────────┘    └─────────────┘    └─────────────┘        │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │                    Shared Services                       │  │
│   │  • Auth & Sessions     • Billing & Subscriptions        │  │
│   │  • File Storage        • Background Jobs                │  │
│   │  • Audit Logs          • Analytics                      │  │
│   │  • Invitations         • Automation Workflows           │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Core Features

| Feature | Description |
|---------|-------------|
| Multi-Tenant | Complete company isolation with RLS |
| Role-Based Access | Admin, Manager, User with granular permissions |
| User Management | CRUD, invitations, department assignment |
| Billing | Stripe integration, subscriptions, webhooks |
| File Storage | Secure uploads with Supabase Storage |
| Background Jobs | DB-backed async workers |
| Audit Logs | Complete activity tracking |
| Analytics | Dashboards and reports |
| Automation | Visual workflow builder |

---

## Tech Stack (Final)

| What | Technology | Learn First |
|------|------------|-------------|
| **Frontend Framework** | React 18 + TypeScript | https://react.dev/learn |
| **Build Tool** | Vite | https://vitejs.dev/guide/ |
| **Routing** | React Router v6 | https://reactrouter.com/en/main/start/tutorial |
| **Styling** | Tailwind CSS | https://tailwindcss.com/docs |
| **UI Components** | shadcn/ui (Radix) | https://ui.shadcn.com/docs |
| **Data Fetching** | TanStack Query v5 | https://tanstack.com/query/latest |
| **Forms** | React Hook Form + Zod | https://react-hook-form.com/get-started |
| **Animations** | Framer Motion | https://www.framer.com/motion/ |
| **Charts** | Recharts | https://recharts.org/en-US/guide |
| **Workflows** | React Flow | https://reactflow.dev/learn |
| **Backend** | Supabase | https://supabase.com/docs |
| **Database** | PostgreSQL | https://www.postgresql.org/docs/ |
| **Auth** | Supabase Auth | https://supabase.com/docs/guides/auth |
| **Storage** | Supabase Storage | https://supabase.com/docs/guides/storage |
| **Edge Functions** | Deno | https://supabase.com/docs/guides/functions |

---

## Architecture Decision: Keep It Simple

```
┌──────────────────────────────────────────────────────────────┐
│                    PHASE 1-10 ARCHITECTURE                   │
│                                                              │
│   ┌──────────────┐         ┌──────────────────────────────┐ │
│   │   React App  │ ◄─────► │        Supabase              │ │
│   │   (Vite)     │         │  ┌──────────────────────┐   │ │
│   └──────────────┘         │  │ PostgreSQL + RLS     │   │ │
│                            │  ├──────────────────────┤   │ │
│                            │  │ Auth                 │   │ │
│                            │  ├──────────────────────┤   │ │
│                            │  │ Storage              │   │ │
│                            │  ├──────────────────────┤   │ │
│                            │  │ Edge Functions       │   │ │
│                            │  ├──────────────────────┤   │ │
│                            │  │ Realtime             │   │ │
│                            │  └──────────────────────┘   │ │
│                            └──────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

### What We DON'T Need Yet

| Technology | When to Add | Why Not Now |
|------------|-------------|-------------|
| Redis | Phase 11+ | Supabase handles caching |
| Kubernetes | Phase 12+ | Supabase auto-scales |
| Multi-region DB | Phase 13+ | Single region is fine |
| Microservices | Phase 14+ | Monolith first |
| Load Balancers | Never (manual) | Supabase provides this |

**Rule:** Supabase already provides load balancing, horizontal scaling, managed Postgres, and edge execution. Don't over-engineer.

---

## The Phases

```
Phase 1:  Foundation & Auth        (Days 1-3)
Phase 2:  Multi-Tenancy & RBAC     (Days 4-6)
Phase 2.5: Invitations             (Days 7-8)
Phase 3:  Core Product Features    (Days 9-14)
Phase 4:  Billing & Background     (Days 15-19)
Phase 5:  Performance              (Days 20-22)
Phase 6:  Security Hardening       (Days 23-25)
Phase 7:  Observability            (Days 26-28)
Phase 8:  Customization & UX       (Days 29-32)
Phase 9:  Testing & QA             (Days 33-36)
Phase 10: Deployment & Release     (Days 37-40)
```

---

## Core Security Model (Non-Negotiable)

Before you write ANY code, understand these rules:

```
┌─────────────────────────────────────────────────────────────┐
│                  SECURITY ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   1. Roles stored in SEPARATE user_roles table              │
│      ❌ NEVER in profile.role                               │
│                                                             │
│   2. ALL tables have RLS enabled                            │
│      ❌ No exceptions                                       │
│                                                             │
│   3. Role checks via SECURITY DEFINER functions             │
│      ❌ Never trust frontend for permissions                │
│                                                             │
│   4. Audit logs for ALL sensitive actions                   │
│      ❌ No silent mutations                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

```sql
-- ✅ CORRECT: Role in separate table
CREATE TABLE user_roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  company_id UUID REFERENCES companies(id) ON DELETE CASCADE,
  role TEXT NOT NULL CHECK (role IN ('admin', 'manager', 'user')),
  UNIQUE(user_id, company_id)
);

-- ❌ WRONG: Role in profile
CREATE TABLE profiles (
  id UUID PRIMARY KEY,
  role TEXT  -- NEVER DO THIS
);
```

---

## Phase 1: Foundation & Auth

**Branch:** `main → develop → phase-1/foundation`

### Learn First (2-3 hours)

| Topic | Resource |
|-------|----------|
| Supabase Setup | https://supabase.com/docs/guides/getting-started |
| Supabase Auth | https://supabase.com/docs/guides/auth |
| Row Level Security | https://supabase.com/docs/guides/auth/row-level-security |
| React + Supabase | https://supabase.com/docs/guides/getting-started/quickstarts/reactjs |

### Do

#### 1. Create Supabase Project

```bash
# Go to https://supabase.com/dashboard
# Create new project: "novapulse"
# Save your credentials:
#   - Project URL
#   - Anon Key
#   - Service Role Key (keep secret!)
```

#### 2. Create React App

```bash
mkdir novapulse && cd novapulse
npm create vite@latest web -- --template react-ts
cd web
npm install
```

#### 3. Install Dependencies

```bash
# Core
npm install @supabase/supabase-js @tanstack/react-query react-router-dom

# UI
npm install tailwindcss postcss autoprefixer
npm install @radix-ui/react-slot class-variance-authority clsx tailwind-merge
npm install lucide-react

# Forms
npm install react-hook-form @hookform/resolvers zod

# Utils
npm install date-fns sonner
```

#### 4. Initialize Tailwind

```bash
npx tailwindcss init -p
```

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss'

const config: Config = {
  darkMode: ['class'],
  content: [
    './index.html',
    './src/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
    },
  },
  plugins: [],
}

export default config
```

#### 5. Database Schema

Run this in Supabase SQL Editor:

```sql
-- ============================================
-- PHASE 1: FOUNDATION SCHEMA
-- ============================================

-- Enable necessary extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- ============================================
-- PROFILES TABLE
-- ============================================
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT NOT NULL,
  full_name TEXT,
  avatar_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- Policies
CREATE POLICY "Users can view own profile"
  ON profiles FOR SELECT
  USING (auth.uid() = id);

CREATE POLICY "Users can update own profile"
  ON profiles FOR UPDATE
  USING (auth.uid() = id);

-- ============================================
-- USER ROLES TABLE (CRITICAL: Separate from profile!)
-- ============================================
CREATE TABLE user_roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  company_id UUID, -- Will reference companies table in Phase 2
  role TEXT NOT NULL DEFAULT 'user' CHECK (role IN ('admin', 'manager', 'user')),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, company_id)
);

-- Enable RLS
ALTER TABLE user_roles ENABLE ROW LEVEL SECURITY;

-- ============================================
-- AUDIT LOGS TABLE
-- ============================================
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id),
  action TEXT NOT NULL,
  entity_type TEXT NOT NULL,
  entity_id UUID,
  old_data JSONB,
  new_data JSONB,
  ip_address INET,
  user_agent TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE audit_logs ENABLE ROW LEVEL SECURITY;

-- ============================================
-- HELPER FUNCTIONS
-- ============================================

-- Get current user's role (SECURITY DEFINER = runs with elevated privileges)
CREATE OR REPLACE FUNCTION get_user_role(p_user_id UUID, p_company_id UUID DEFAULT NULL)
RETURNS TEXT
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
DECLARE
  v_role TEXT;
BEGIN
  SELECT role INTO v_role
  FROM user_roles
  WHERE user_id = p_user_id
    AND (company_id = p_company_id OR (p_company_id IS NULL AND company_id IS NULL))
  LIMIT 1;
  
  RETURN COALESCE(v_role, 'user');
END;
$$;

-- Check if user is admin
CREATE OR REPLACE FUNCTION is_admin(p_user_id UUID, p_company_id UUID DEFAULT NULL)
RETURNS BOOLEAN
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  RETURN get_user_role(p_user_id, p_company_id) = 'admin';
END;
$$;

-- Log audit event
CREATE OR REPLACE FUNCTION log_audit(
  p_action TEXT,
  p_entity_type TEXT,
  p_entity_id UUID DEFAULT NULL,
  p_old_data JSONB DEFAULT NULL,
  p_new_data JSONB DEFAULT NULL
)
RETURNS UUID
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
DECLARE
  v_id UUID;
BEGIN
  INSERT INTO audit_logs (user_id, action, entity_type, entity_id, old_data, new_data)
  VALUES (auth.uid(), p_action, p_entity_type, p_entity_id, p_old_data, p_new_data)
  RETURNING id INTO v_id;
  
  RETURN v_id;
END;
$$;

-- ============================================
-- TRIGGERS
-- ============================================

-- Auto-create profile on user signup
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS TRIGGER
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  -- Create profile
  INSERT INTO profiles (id, email, full_name)
  VALUES (
    NEW.id,
    NEW.email,
    COALESCE(NEW.raw_user_meta_data->>'full_name', split_part(NEW.email, '@', 1))
  );
  
  -- Create default role (no company yet)
  INSERT INTO user_roles (user_id, role)
  VALUES (NEW.id, 'user');
  
  -- Log the event
  PERFORM log_audit('user_created', 'user', NEW.id, NULL, jsonb_build_object('email', NEW.email));
  
  RETURN NEW;
END;
$$;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION handle_new_user();

-- Auto-update updated_at
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$;

CREATE TRIGGER update_profiles_updated_at
  BEFORE UPDATE ON profiles
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

#### 6. Supabase Client

```typescript
// src/lib/supabase.ts
import { createClient } from '@supabase/supabase-js'
import type { Database } from './database.types'

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY

if (!supabaseUrl || !supabaseAnonKey) {
  throw new Error('Missing Supabase environment variables')
}

export const supabase = createClient<Database>(supabaseUrl, supabaseAnonKey, {
  auth: {
    autoRefreshToken: true,
    persistSession: true,
    detectSessionInUrl: true,
  },
})
```

#### 7. Auth Context

```typescript
// src/contexts/AuthContext.tsx
import { createContext, useContext, useEffect, useState, ReactNode } from 'react'
import { User, Session } from '@supabase/supabase-js'
import { supabase } from '@/lib/supabase'

interface AuthContextType {
  user: User | null
  session: Session | null
  loading: boolean
  signUp: (email: string, password: string, fullName: string) => Promise<void>
  signIn: (email: string, password: string) => Promise<void>
  signOut: () => Promise<void>
}

const AuthContext = createContext<AuthContextType | undefined>(undefined)

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null)
  const [session, setSession] = useState<Session | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    // Get initial session
    supabase.auth.getSession().then(({ data: { session } }) => {
      setSession(session)
      setUser(session?.user ?? null)
      setLoading(false)
    })

    // Listen for auth changes
    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      async (event, session) => {
        setSession(session)
        setUser(session?.user ?? null)
        setLoading(false)
      }
    )

    return () => subscription.unsubscribe()
  }, [])

  const signUp = async (email: string, password: string, fullName: string) => {
    const { error } = await supabase.auth.signUp({
      email,
      password,
      options: {
        data: { full_name: fullName },
      },
    })
    if (error) throw error
  }

  const signIn = async (email: string, password: string) => {
    const { error } = await supabase.auth.signInWithPassword({
      email,
      password,
    })
    if (error) throw error
  }

  const signOut = async () => {
    const { error } = await supabase.auth.signOut()
    if (error) throw error
  }

  return (
    <AuthContext.Provider value={{ user, session, loading, signUp, signIn, signOut }}>
      {children}
    </AuthContext.Provider>
  )
}

export function useAuth() {
  const context = useContext(AuthContext)
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider')
  }
  return context
}
```

#### 8. Protected Route Component

```typescript
// src/components/ProtectedRoute.tsx
import { Navigate, useLocation } from 'react-router-dom'
import { useAuth } from '@/contexts/AuthContext'

interface ProtectedRouteProps {
  children: React.ReactNode
}

export function ProtectedRoute({ children }: ProtectedRouteProps) {
  const { user, loading } = useAuth()
  const location = useLocation()

  if (loading) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-primary" />
      </div>
    )
  }

  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />
  }

  return <>{children}</>
}
```

#### 9. Login Page

```typescript
// src/pages/auth/LoginPage.tsx
import { useState } from 'react'
import { Link, useNavigate, useLocation } from 'react-router-dom'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { useAuth } from '@/contexts/AuthContext'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { toast } from 'sonner'

const loginSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(1, 'Password is required'),
})

type LoginFormData = z.infer<typeof loginSchema>

export function LoginPage() {
  const { signIn } = useAuth()
  const navigate = useNavigate()
  const location = useLocation()
  const [isLoading, setIsLoading] = useState(false)

  const from = location.state?.from?.pathname || '/dashboard'

  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  })

  const onSubmit = async (data: LoginFormData) => {
    setIsLoading(true)
    try {
      await signIn(data.email, data.password)
      toast.success('Welcome back!')
      navigate(from, { replace: true })
    } catch (error: any) {
      toast.error(error.message || 'Failed to sign in')
    } finally {
      setIsLoading(false)
    }
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-background">
      <Card className="w-full max-w-md">
        <CardHeader className="space-y-1">
          <CardTitle className="text-2xl font-bold">Welcome back</CardTitle>
          <CardDescription>
            Enter your credentials to access your account
          </CardDescription>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
            <div className="space-y-2">
              <Label htmlFor="email">Email</Label>
              <Input
                id="email"
                type="email"
                placeholder="name@example.com"
                {...register('email')}
              />
              {errors.email && (
                <p className="text-sm text-destructive">{errors.email.message}</p>
              )}
            </div>

            <div className="space-y-2">
              <Label htmlFor="password">Password</Label>
              <Input
                id="password"
                type="password"
                {...register('password')}
              />
              {errors.password && (
                <p className="text-sm text-destructive">{errors.password.message}</p>
              )}
            </div>

            <Button type="submit" className="w-full" disabled={isLoading}>
              {isLoading ? 'Signing in...' : 'Sign in'}
            </Button>

            <p className="text-center text-sm text-muted-foreground">
              Don't have an account?{' '}
              <Link to="/register" className="text-primary hover:underline">
                Sign up
              </Link>
            </p>
          </form>
        </CardContent>
      </Card>
    </div>
  )
}
```

#### 10. Register Page

```typescript
// src/pages/auth/RegisterPage.tsx
import { useState } from 'react'
import { Link, useNavigate } from 'react-router-dom'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { useAuth } from '@/contexts/AuthContext'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { toast } from 'sonner'

const registerSchema = z.object({
  fullName: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email address'),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain an uppercase letter')
    .regex(/[a-z]/, 'Password must contain a lowercase letter')
    .regex(/[0-9]/, 'Password must contain a number'),
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: "Passwords don't match",
  path: ['confirmPassword'],
})

type RegisterFormData = z.infer<typeof registerSchema>

export function RegisterPage() {
  const { signUp } = useAuth()
  const navigate = useNavigate()
  const [isLoading, setIsLoading] = useState(false)

  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<RegisterFormData>({
    resolver: zodResolver(registerSchema),
  })

  const onSubmit = async (data: RegisterFormData) => {
    setIsLoading(true)
    try {
      await signUp(data.email, data.password, data.fullName)
      toast.success('Account created! Please check your email to verify.')
      navigate('/login')
    } catch (error: any) {
      toast.error(error.message || 'Failed to create account')
    } finally {
      setIsLoading(false)
    }
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-background">
      <Card className="w-full max-w-md">
        <CardHeader className="space-y-1">
          <CardTitle className="text-2xl font-bold">Create an account</CardTitle>
          <CardDescription>
            Enter your information to get started
          </CardDescription>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
            <div className="space-y-2">
              <Label htmlFor="fullName">Full Name</Label>
              <Input
                id="fullName"
                placeholder="John Doe"
                {...register('fullName')}
              />
              {errors.fullName && (
                <p className="text-sm text-destructive">{errors.fullName.message}</p>
              )}
            </div>

            <div className="space-y-2">
              <Label htmlFor="email">Email</Label>
              <Input
                id="email"
                type="email"
                placeholder="name@example.com"
                {...register('email')}
              />
              {errors.email && (
                <p className="text-sm text-destructive">{errors.email.message}</p>
              )}
            </div>

            <div className="space-y-2">
              <Label htmlFor="password">Password</Label>
              <Input
                id="password"
                type="password"
                {...register('password')}
              />
              {errors.password && (
                <p className="text-sm text-destructive">{errors.password.message}</p>
              )}
            </div>

            <div className="space-y-2">
              <Label htmlFor="confirmPassword">Confirm Password</Label>
              <Input
                id="confirmPassword"
                type="password"
                {...register('confirmPassword')}
              />
              {errors.confirmPassword && (
                <p className="text-sm text-destructive">{errors.confirmPassword.message}</p>
              )}
            </div>

            <Button type="submit" className="w-full" disabled={isLoading}>
              {isLoading ? 'Creating account...' : 'Create account'}
            </Button>

            <p className="text-center text-sm text-muted-foreground">
              Already have an account?{' '}
              <Link to="/login" className="text-primary hover:underline">
                Sign in
              </Link>
            </p>
          </form>
        </CardContent>
      </Card>
    </div>
  )
}
```

#### 11. App Router Setup

```typescript
// src/App.tsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { Toaster } from 'sonner'
import { AuthProvider } from '@/contexts/AuthContext'
import { ProtectedRoute } from '@/components/ProtectedRoute'
import { LoginPage } from '@/pages/auth/LoginPage'
import { RegisterPage } from '@/pages/auth/RegisterPage'
import { DashboardPage } from '@/pages/DashboardPage'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000,
      retry: 1,
    },
  },
})

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <AuthProvider>
        <BrowserRouter>
          <Routes>
            {/* Public routes */}
            <Route path="/login" element={<LoginPage />} />
            <Route path="/register" element={<RegisterPage />} />

            {/* Protected routes */}
            <Route
              path="/dashboard"
              element={
                <ProtectedRoute>
                  <DashboardPage />
                </ProtectedRoute>
              }
            />

            {/* Redirect root to dashboard */}
            <Route path="/" element={<Navigate to="/dashboard" replace />} />
          </Routes>
        </BrowserRouter>
        <Toaster position="top-right" />
      </AuthProvider>
    </QueryClientProvider>
  )
}

export default App
```

#### 12. Environment Variables

```bash
# .env.local
VITE_SUPABASE_URL=your-supabase-url
VITE_SUPABASE_ANON_KEY=your-anon-key
```

### Test It

```bash
npm run dev

# 1. Go to http://localhost:5173/register
# 2. Create an account
# 3. Check email for verification (or disable in Supabase dashboard for dev)
# 4. Login
# 5. Should redirect to /dashboard
```

### Done When

- [ ] User can register with email/password
- [ ] User can login
- [ ] Session persists on refresh
- [ ] Protected routes redirect to login
- [ ] Profile created automatically on signup
- [ ] Default role assigned on signup
- [ ] Audit log records user creation
- [ ] RLS policies working (test in Supabase dashboard)

### Commit

```bash
git add .
git commit -m "feat(auth): add foundation and authentication"
git checkout develop
git merge phase-1/foundation
```

---

## Phase 2: Multi-Tenancy & RBAC

**Branch:** `develop → phase-2/multi-tenancy`

### Learn First (1-2 hours)

| Topic | Resource |
|-------|----------|
| Multi-tenant patterns | https://supabase.com/docs/guides/auth/row-level-security |
| RLS with joins | https://supabase.com/docs/guides/auth/row-level-security#policies-with-joins |

### What You're Building

```
┌─────────────────────────────────────────────────────────────┐
│                    MULTI-TENANT MODEL                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Company A                    Company B                    │
│   ┌─────────────────┐         ┌─────────────────┐          │
│   │ Admin: alice@   │         │ Admin: bob@     │          │
│   │ Manager: carol@ │         │ Manager: dave@  │          │
│   │ User: eve@      │         │ User: frank@    │          │
│   │                 │         │                 │          │
│   │ Departments:    │         │ Departments:    │          │
│   │ - Engineering   │         │ - Sales         │          │
│   │ - Marketing     │         │ - Support       │          │
│   └─────────────────┘         └─────────────────┘          │
│                                                             │
│   ❌ Company A CANNOT see Company B data                    │
│   ❌ User cannot access other user's data                   │
│   ✅ Admin can manage all company users                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Do

#### 1. Database Schema - Companies & Departments

```sql
-- ============================================
-- PHASE 2: MULTI-TENANCY SCHEMA
-- ============================================

-- ============================================
-- COMPANIES TABLE
-- ============================================
CREATE TABLE companies (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  logo_url TEXT,
  settings JSONB DEFAULT '{}',
  subscription_tier TEXT DEFAULT 'free' CHECK (subscription_tier IN ('free', 'starter', 'pro', 'enterprise')),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE companies ENABLE ROW LEVEL SECURITY;

-- ============================================
-- DEPARTMENTS TABLE
-- ============================================
CREATE TABLE departments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_id UUID NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  description TEXT,
  parent_id UUID REFERENCES departments(id) ON DELETE SET NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(company_id, name)
);

-- Enable RLS
ALTER TABLE departments ENABLE ROW LEVEL SECURITY;

-- ============================================
-- COMPANY MEMBERS (Links users to companies)
-- ============================================
CREATE TABLE company_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_id UUID NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  department_id UUID REFERENCES departments(id) ON DELETE SET NULL,
  role TEXT NOT NULL DEFAULT 'user' CHECK (role IN ('admin', 'manager', 'user')),
  job_title TEXT,
  is_active BOOLEAN DEFAULT true,
  joined_at TIMESTAMPTZ DEFAULT NOW(),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(company_id, user_id)
);

-- Enable RLS
ALTER TABLE company_members ENABLE ROW LEVEL SECURITY;

-- ============================================
-- UPDATE USER_ROLES TO LINK TO COMPANIES
-- ============================================
ALTER TABLE user_roles 
  ADD CONSTRAINT fk_company 
  FOREIGN KEY (company_id) 
  REFERENCES companies(id) 
  ON DELETE CASCADE;

-- ============================================
-- HELPER FUNCTIONS
-- ============================================

-- Get user's company memberships
CREATE OR REPLACE FUNCTION get_user_companies(p_user_id UUID)
RETURNS TABLE(company_id UUID, role TEXT)
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  RETURN QUERY
  SELECT cm.company_id, cm.role
  FROM company_members cm
  WHERE cm.user_id = p_user_id AND cm.is_active = true;
END;
$$;

-- Check if user belongs to company
CREATE OR REPLACE FUNCTION is_company_member(p_user_id UUID, p_company_id UUID)
RETURNS BOOLEAN
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  RETURN EXISTS (
    SELECT 1 FROM company_members
    WHERE user_id = p_user_id 
      AND company_id = p_company_id 
      AND is_active = true
  );
END;
$$;

-- Check if user is company admin
CREATE OR REPLACE FUNCTION is_company_admin(p_user_id UUID, p_company_id UUID)
RETURNS BOOLEAN
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  RETURN EXISTS (
    SELECT 1 FROM company_members
    WHERE user_id = p_user_id 
      AND company_id = p_company_id 
      AND role = 'admin'
      AND is_active = true
  );
END;
$$;

-- Check if user is company admin or manager
CREATE OR REPLACE FUNCTION is_company_admin_or_manager(p_user_id UUID, p_company_id UUID)
RETURNS BOOLEAN
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  RETURN EXISTS (
    SELECT 1 FROM company_members
    WHERE user_id = p_user_id 
      AND company_id = p_company_id 
      AND role IN ('admin', 'manager')
      AND is_active = true
  );
END;
$$;

-- ============================================
-- RLS POLICIES
-- ============================================

-- Companies: Members can view their companies
CREATE POLICY "Members can view their companies"
  ON companies FOR SELECT
  USING (is_company_member(auth.uid(), id));

-- Companies: Only admins can update
CREATE POLICY "Admins can update company"
  ON companies FOR UPDATE
  USING (is_company_admin(auth.uid(), id));

-- Departments: Members can view
CREATE POLICY "Members can view departments"
  ON departments FOR SELECT
  USING (is_company_member(auth.uid(), company_id));

-- Departments: Admins can manage
CREATE POLICY "Admins can insert departments"
  ON departments FOR INSERT
  WITH CHECK (is_company_admin(auth.uid(), company_id));

CREATE POLICY "Admins can update departments"
  ON departments FOR UPDATE
  USING (is_company_admin(auth.uid(), company_id));

CREATE POLICY "Admins can delete departments"
  ON departments FOR DELETE
  USING (is_company_admin(auth.uid(), company_id));

-- Company Members: View policy
CREATE POLICY "Members can view other members"
  ON company_members FOR SELECT
  USING (is_company_member(auth.uid(), company_id));

-- Company Members: Admin management
CREATE POLICY "Admins can insert members"
  ON company_members FOR INSERT
  WITH CHECK (is_company_admin(auth.uid(), company_id));

CREATE POLICY "Admins can update members"
  ON company_members FOR UPDATE
  USING (is_company_admin(auth.uid(), company_id));

CREATE POLICY "Admins can delete members"
  ON company_members FOR DELETE
  USING (is_company_admin(auth.uid(), company_id));

-- ============================================
-- TRIGGERS
-- ============================================

-- Update timestamps
CREATE TRIGGER update_companies_updated_at
  BEFORE UPDATE ON companies
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_departments_updated_at
  BEFORE UPDATE ON departments
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_company_members_updated_at
  BEFORE UPDATE ON company_members
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

#### 2. Company Registration Flow

```typescript
// src/lib/api/companies.ts
import { supabase } from '@/lib/supabase'

export interface CreateCompanyData {
  name: string
  slug: string
}

export async function createCompany(data: CreateCompanyData) {
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) throw new Error('Not authenticated')

  // Start transaction-like operation
  // 1. Create company
  const { data: company, error: companyError } = await supabase
    .from('companies')
    .insert({
      name: data.name,
      slug: data.slug.toLowerCase().replace(/[^a-z0-9-]/g, '-'),
    })
    .select()
    .single()

  if (companyError) throw companyError

  // 2. Add current user as admin
  const { error: memberError } = await supabase
    .from('company_members')
    .insert({
      company_id: company.id,
      user_id: user.id,
      role: 'admin',
    })

  if (memberError) {
    // Rollback: delete company
    await supabase.from('companies').delete().eq('id', company.id)
    throw memberError
  }

  // 3. Update user_roles
  const { error: roleError } = await supabase
    .from('user_roles')
    .upsert({
      user_id: user.id,
      company_id: company.id,
      role: 'admin',
    })

  if (roleError) console.error('Failed to update user_roles:', roleError)

  return company
}

export async function getUserCompanies() {
  const { data, error } = await supabase
    .from('company_members')
    .select(`
      company_id,
      role,
      joined_at,
      companies (
        id,
        name,
        slug,
        logo_url,
        subscription_tier
      )
    `)
    .eq('is_active', true)

  if (error) throw error
  return data
}

export async function getCompanyById(companyId: string) {
  const { data, error } = await supabase
    .from('companies')
    .select('*')
    .eq('id', companyId)
    .single()

  if (error) throw error
  return data
}
```

#### 3. Company Context

```typescript
// src/contexts/CompanyContext.tsx
import { createContext, useContext, useState, useEffect, ReactNode } from 'react'
import { useAuth } from '@/contexts/AuthContext'
import { getUserCompanies } from '@/lib/api/companies'

interface Company {
  id: string
  name: string
  slug: string
  logo_url: string | null
  subscription_tier: string
}

interface CompanyMembership {
  company_id: string
  role: 'admin' | 'manager' | 'user'
  joined_at: string
  companies: Company
}

interface CompanyContextType {
  companies: CompanyMembership[]
  currentCompany: Company | null
  currentRole: 'admin' | 'manager' | 'user' | null
  setCurrentCompany: (company: Company) => void
  loading: boolean
  refetch: () => Promise<void>
}

const CompanyContext = createContext<CompanyContextType | undefined>(undefined)

const STORAGE_KEY = 'novapulse_current_company'

export function CompanyProvider({ children }: { children: ReactNode }) {
  const { user } = useAuth()
  const [companies, setCompanies] = useState<CompanyMembership[]>([])
  const [currentCompany, setCurrentCompanyState] = useState<Company | null>(null)
  const [currentRole, setCurrentRole] = useState<'admin' | 'manager' | 'user' | null>(null)
  const [loading, setLoading] = useState(true)

  const fetchCompanies = async () => {
    if (!user) {
      setCompanies([])
      setCurrentCompanyState(null)
      setCurrentRole(null)
      setLoading(false)
      return
    }

    try {
      const data = await getUserCompanies()
      setCompanies(data as CompanyMembership[])

      // Restore from localStorage or use first company
      const savedCompanyId = localStorage.getItem(STORAGE_KEY)
      const savedMembership = data.find(m => m.company_id === savedCompanyId)
      
      if (savedMembership) {
        setCurrentCompanyState(savedMembership.companies as Company)
        setCurrentRole(savedMembership.role)
      } else if (data.length > 0) {
        const first = data[0]
        setCurrentCompanyState(first.companies as Company)
        setCurrentRole(first.role)
        localStorage.setItem(STORAGE_KEY, first.company_id)
      }
    } catch (error) {
      console.error('Failed to fetch companies:', error)
    } finally {
      setLoading(false)
    }
  }

  useEffect(() => {
    fetchCompanies()
  }, [user])

  const setCurrentCompany = (company: Company) => {
    const membership = companies.find(m => m.company_id === company.id)
    if (membership) {
      setCurrentCompanyState(company)
      setCurrentRole(membership.role)
      localStorage.setItem(STORAGE_KEY, company.id)
    }
  }

  return (
    <CompanyContext.Provider
      value={{
        companies,
        currentCompany,
        currentRole,
        setCurrentCompany,
        loading,
        refetch: fetchCompanies,
      }}
    >
      {children}
    </CompanyContext.Provider>
  )
}

export function useCompany() {
  const context = useContext(CompanyContext)
  if (context === undefined) {
    throw new Error('useCompany must be used within a CompanyProvider')
  }
  return context
}
```

#### 4. Role-Based Component

```typescript
// src/components/RoleGuard.tsx
import { useCompany } from '@/contexts/CompanyContext'

interface RoleGuardProps {
  children: React.ReactNode
  allowedRoles: ('admin' | 'manager' | 'user')[]
  fallback?: React.ReactNode
}

export function RoleGuard({ children, allowedRoles, fallback = null }: RoleGuardProps) {
  const { currentRole } = useCompany()

  if (!currentRole || !allowedRoles.includes(currentRole)) {
    return <>{fallback}</>
  }

  return <>{children}</>
}

// Hook version
export function useHasRole(allowedRoles: ('admin' | 'manager' | 'user')[]) {
  const { currentRole } = useCompany()
  return currentRole ? allowedRoles.includes(currentRole) : false
}

// Specific hooks
export function useIsAdmin() {
  return useHasRole(['admin'])
}

export function useIsAdminOrManager() {
  return useHasRole(['admin', 'manager'])
}
```

#### 5. Company Switcher Component

```typescript
// src/components/CompanySwitcher.tsx
import { Check, ChevronsUpDown, Plus } from 'lucide-react'
import { cn } from '@/lib/utils'
import { Button } from '@/components/ui/button'
import {
  Command,
  CommandEmpty,
  CommandGroup,
  CommandInput,
  CommandItem,
  CommandList,
  CommandSeparator,
} from '@/components/ui/command'
import {
  Popover,
  PopoverContent,
  PopoverTrigger,
} from '@/components/ui/popover'
import { useCompany } from '@/contexts/CompanyContext'
import { useState } from 'react'
import { useNavigate } from 'react-router-dom'

export function CompanySwitcher() {
  const { companies, currentCompany, setCurrentCompany } = useCompany()
  const [open, setOpen] = useState(false)
  const navigate = useNavigate()

  return (
    <Popover open={open} onOpenChange={setOpen}>
      <PopoverTrigger asChild>
        <Button
          variant="outline"
          role="combobox"
          aria-expanded={open}
          className="w-[200px] justify-between"
        >
          {currentCompany?.name ?? 'Select company...'}
          <ChevronsUpDown className="ml-2 h-4 w-4 shrink-0 opacity-50" />
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-[200px] p-0">
        <Command>
          <CommandInput placeholder="Search company..." />
          <CommandList>
            <CommandEmpty>No company found.</CommandEmpty>
            <CommandGroup heading="Companies">
              {companies.map((membership) => (
                <CommandItem
                  key={membership.company_id}
                  value={membership.companies.name}
                  onSelect={() => {
                    setCurrentCompany(membership.companies)
                    setOpen(false)
                  }}
                >
                  <Check
                    className={cn(
                      'mr-2 h-4 w-4',
                      currentCompany?.id === membership.company_id
                        ? 'opacity-100'
                        : 'opacity-0'
                    )}
                  />
                  <div className="flex flex-col">
                    <span>{membership.companies.name}</span>
                    <span className="text-xs text-muted-foreground capitalize">
                      {membership.role}
                    </span>
                  </div>
                </CommandItem>
              ))}
            </CommandGroup>
            <CommandSeparator />
            <CommandGroup>
              <CommandItem
                onSelect={() => {
                  setOpen(false)
                  navigate('/create-company')
                }}
              >
                <Plus className="mr-2 h-4 w-4" />
                Create Company
              </CommandItem>
            </CommandGroup>
          </CommandList>
        </Command>
      </PopoverContent>
    </Popover>
  )
}
```

#### 6. Create Company Page

```typescript
// src/pages/CreateCompanyPage.tsx
import { useState } from 'react'
import { useNavigate } from 'react-router-dom'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { createCompany } from '@/lib/api/companies'
import { useCompany } from '@/contexts/CompanyContext'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { toast } from 'sonner'

const companySchema = z.object({
  name: z.string().min(2, 'Company name must be at least 2 characters'),
  slug: z
    .string()
    .min(3, 'URL slug must be at least 3 characters')
    .regex(/^[a-z0-9-]+$/, 'Only lowercase letters, numbers, and hyphens'),
})

type CompanyFormData = z.infer<typeof companySchema>

export function CreateCompanyPage() {
  const navigate = useNavigate()
  const { refetch } = useCompany()
  const [isLoading, setIsLoading] = useState(false)

  const {
    register,
    handleSubmit,
    watch,
    setValue,
    formState: { errors },
  } = useForm<CompanyFormData>({
    resolver: zodResolver(companySchema),
  })

  const name = watch('name')

  // Auto-generate slug from name
  const generateSlug = (name: string) => {
    return name
      .toLowerCase()
      .replace(/[^a-z0-9]+/g, '-')
      .replace(/^-|-$/g, '')
  }

  const onSubmit = async (data: CompanyFormData) => {
    setIsLoading(true)
    try {
      const company = await createCompany(data)
      await refetch()
      toast.success('Company created successfully!')
      navigate(`/dashboard`)
    } catch (error: any) {
      if (error.code === '23505') {
        toast.error('A company with this URL already exists')
      } else {
        toast.error(error.message || 'Failed to create company')
      }
    } finally {
      setIsLoading(false)
    }
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-background">
      <Card className="w-full max-w-md">
        <CardHeader>
          <CardTitle>Create a Company</CardTitle>
          <CardDescription>
            Set up your organization to start inviting team members
          </CardDescription>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
            <div className="space-y-2">
              <Label htmlFor="name">Company Name</Label>
              <Input
                id="name"
                placeholder="Acme Inc."
                {...register('name')}
                onChange={(e) => {
                  register('name').onChange(e)
                  setValue('slug', generateSlug(e.target.value))
                }}
              />
              {errors.name && (
                <p className="text-sm text-destructive">{errors.name.message}</p>
              )}
            </div>

            <div className="space-y-2">
              <Label htmlFor="slug">URL Slug</Label>
              <div className="flex items-center">
                <span className="text-sm text-muted-foreground mr-1">
                  novapulse.app/
                </span>
                <Input
                  id="slug"
                  placeholder="acme-inc"
                  {...register('slug')}
                />
              </div>
              {errors.slug && (
                <p className="text-sm text-destructive">{errors.slug.message}</p>
              )}
            </div>

            <Button type="submit" className="w-full" disabled={isLoading}>
              {isLoading ? 'Creating...' : 'Create Company'}
            </Button>
          </form>
        </CardContent>
      </Card>
    </div>
  )
}
```

### Done When

- [ ] Companies table created with RLS
- [ ] Departments table created with RLS
- [ ] Company members linking table with roles
- [ ] User can create a company (becomes admin)
- [ ] User can belong to multiple companies
- [ ] Company switcher works
- [ ] RLS prevents cross-company data access
- [ ] Role-based UI components working

### Commit

```bash
git add .
git commit -m "feat(multi-tenancy): add companies and RBAC"
git checkout develop
git merge phase-2/multi-tenancy
```

---

## Phase 2.5: Invitations

**Branch:** `develop → phase-2.5/invitations`

### Learn First (30 mins)

| Topic | Resource |
|-------|----------|
| Supabase Edge Functions | https://supabase.com/docs/guides/functions |
| Email templates | https://supabase.com/docs/guides/auth/auth-email-templates |

### Do

#### 1. Invitations Table

```sql
-- ============================================
-- PHASE 2.5: INVITATIONS
-- ============================================

CREATE TABLE invitations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_id UUID NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  email TEXT NOT NULL,
  role TEXT NOT NULL DEFAULT 'user' CHECK (role IN ('admin', 'manager', 'user')),
  department_id UUID REFERENCES departments(id) ON DELETE SET NULL,
  token TEXT UNIQUE NOT NULL DEFAULT encode(gen_random_bytes(32), 'hex'),
  invited_by UUID NOT NULL REFERENCES auth.users(id),
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'accepted', 'expired', 'revoked')),
  expires_at TIMESTAMPTZ NOT NULL DEFAULT (NOW() + INTERVAL '7 days'),
  accepted_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(company_id, email, status) -- Prevent duplicate pending invites
);

-- Enable RLS
ALTER TABLE invitations ENABLE ROW LEVEL SECURITY;

-- Policies
CREATE POLICY "Admins can view invitations"
  ON invitations FOR SELECT
  USING (is_company_admin(auth.uid(), company_id));

CREATE POLICY "Admins can create invitations"
  ON invitations FOR INSERT
  WITH CHECK (is_company_admin(auth.uid(), company_id));

CREATE POLICY "Admins can update invitations"
  ON invitations FOR UPDATE
  USING (is_company_admin(auth.uid(), company_id));

-- Public policy for accepting invites (by token)
CREATE POLICY "Anyone can view invite by token"
  ON invitations FOR SELECT
  USING (true); -- Token-based lookup, additional validation in function

-- Function to accept invitation
CREATE OR REPLACE FUNCTION accept_invitation(p_token TEXT)
RETURNS JSONB
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
DECLARE
  v_invitation RECORD;
  v_user_id UUID;
BEGIN
  -- Get current user
  v_user_id := auth.uid();
  IF v_user_id IS NULL THEN
    RETURN jsonb_build_object('success', false, 'error', 'Not authenticated');
  END IF;

  -- Get invitation
  SELECT * INTO v_invitation
  FROM invitations
  WHERE token = p_token
    AND status = 'pending'
    AND expires_at > NOW();

  IF NOT FOUND THEN
    RETURN jsonb_build_object('success', false, 'error', 'Invalid or expired invitation');
  END IF;

  -- Check if user email matches
  IF (SELECT email FROM auth.users WHERE id = v_user_id) != v_invitation.email THEN
    RETURN jsonb_build_object('success', false, 'error', 'Invitation is for a different email');
  END IF;

  -- Check if already a member
  IF EXISTS (
    SELECT 1 FROM company_members 
    WHERE company_id = v_invitation.company_id AND user_id = v_user_id
  ) THEN
    RETURN jsonb_build_object('success', false, 'error', 'Already a member of this company');
  END IF;

  -- Add as company member
  INSERT INTO company_members (company_id, user_id, department_id, role)
  VALUES (v_invitation.company_id, v_user_id, v_invitation.department_id, v_invitation.role);

  -- Update user_roles
  INSERT INTO user_roles (user_id, company_id, role)
  VALUES (v_user_id, v_invitation.company_id, v_invitation.role)
  ON CONFLICT (user_id, company_id) DO UPDATE SET role = v_invitation.role;

  -- Mark invitation as accepted
  UPDATE invitations
  SET status = 'accepted', accepted_at = NOW()
  WHERE id = v_invitation.id;

  -- Log audit event
  PERFORM log_audit('invitation_accepted', 'invitation', v_invitation.id, NULL, 
    jsonb_build_object('company_id', v_invitation.company_id, 'role', v_invitation.role));

  RETURN jsonb_build_object(
    'success', true, 
    'company_id', v_invitation.company_id,
    'role', v_invitation.role
  );
END;
$$;
```

#### 2. Invitations API

```typescript
// src/lib/api/invitations.ts
import { supabase } from '@/lib/supabase'

export interface CreateInvitationData {
  companyId: string
  email: string
  role: 'admin' | 'manager' | 'user'
  departmentId?: string
}

export async function createInvitation(data: CreateInvitationData) {
  const { data: invitation, error } = await supabase
    .from('invitations')
    .insert({
      company_id: data.companyId,
      email: data.email.toLowerCase(),
      role: data.role,
      department_id: data.departmentId,
      invited_by: (await supabase.auth.getUser()).data.user?.id,
    })
    .select(`
      *,
      companies (name),
      departments (name)
    `)
    .single()

  if (error) throw error
  return invitation
}

export async function getCompanyInvitations(companyId: string) {
  const { data, error } = await supabase
    .from('invitations')
    .select(`
      *,
      departments (name),
      profiles!invited_by (full_name, email)
    `)
    .eq('company_id', companyId)
    .order('created_at', { ascending: false })

  if (error) throw error
  return data
}

export async function revokeInvitation(invitationId: string) {
  const { error } = await supabase
    .from('invitations')
    .update({ status: 'revoked' })
    .eq('id', invitationId)

  if (error) throw error
}

export async function resendInvitation(invitationId: string) {
  // Generate new token and reset expiry
  const newToken = crypto.randomUUID().replace(/-/g, '') + crypto.randomUUID().replace(/-/g, '')
  
  const { data, error } = await supabase
    .from('invitations')
    .update({
      token: newToken,
      expires_at: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000).toISOString(),
      status: 'pending',
    })
    .eq('id', invitationId)
    .select()
    .single()

  if (error) throw error
  return data
}

export async function getInvitationByToken(token: string) {
  const { data, error } = await supabase
    .from('invitations')
    .select(`
      *,
      companies (id, name, logo_url)
    `)
    .eq('token', token)
    .eq('status', 'pending')
    .gt('expires_at', new Date().toISOString())
    .single()

  if (error) throw error
  return data
}

export async function acceptInvitation(token: string) {
  const { data, error } = await supabase.rpc('accept_invitation', {
    p_token: token,
  })

  if (error) throw error
  if (!data.success) throw new Error(data.error)
  
  return data
}
```

#### 3. Invite Team Page

```typescript
// src/pages/admin/InviteTeamPage.tsx
import { useState } from 'react'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { useCompany } from '@/contexts/CompanyContext'
import { createInvitation, getCompanyInvitations, revokeInvitation } from '@/lib/api/invitations'
import { getDepartments } from '@/lib/api/departments'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table'
import { Badge } from '@/components/ui/badge'
import { toast } from 'sonner'
import { Copy, Trash2, RefreshCw } from 'lucide-react'
import { formatDistanceToNow } from 'date-fns'

const inviteSchema = z.object({
  email: z.string().email('Invalid email address'),
  role: z.enum(['admin', 'manager', 'user']),
  departmentId: z.string().optional(),
})

type InviteFormData = z.infer<typeof inviteSchema>

export function InviteTeamPage() {
  const { currentCompany } = useCompany()
  const queryClient = useQueryClient()

  const { data: invitations, isLoading } = useQuery({
    queryKey: ['invitations', currentCompany?.id],
    queryFn: () => getCompanyInvitations(currentCompany!.id),
    enabled: !!currentCompany,
  })

  const { data: departments } = useQuery({
    queryKey: ['departments', currentCompany?.id],
    queryFn: () => getDepartments(currentCompany!.id),
    enabled: !!currentCompany,
  })

  const createMutation = useMutation({
    mutationFn: createInvitation,
    onSuccess: (data) => {
      queryClient.invalidateQueries({ queryKey: ['invitations'] })
      toast.success('Invitation sent!')
      // Copy invite link to clipboard
      const inviteUrl = `${window.location.origin}/invite/${data.token}`
      navigator.clipboard.writeText(inviteUrl)
      toast.info('Invite link copied to clipboard')
      reset()
    },
    onError: (error: any) => {
      if (error.code === '23505') {
        toast.error('This person already has a pending invitation')
      } else {
        toast.error(error.message || 'Failed to send invitation')
      }
    },
  })

  const revokeMutation = useMutation({
    mutationFn: revokeInvitation,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['invitations'] })
      toast.success('Invitation revoked')
    },
  })

  const {
    register,
    handleSubmit,
    setValue,
    reset,
    formState: { errors },
  } = useForm<InviteFormData>({
    resolver: zodResolver(inviteSchema),
    defaultValues: { role: 'user' },
  })

  const onSubmit = (data: InviteFormData) => {
    if (!currentCompany) return
    createMutation.mutate({
      companyId: currentCompany.id,
      email: data.email,
      role: data.role,
      departmentId: data.departmentId,
    })
  }

  const copyInviteLink = (token: string) => {
    const url = `${window.location.origin}/invite/${token}`
    navigator.clipboard.writeText(url)
    toast.success('Link copied!')
  }

  return (
    <div className="space-y-6">
      <Card>
        <CardHeader>
          <CardTitle>Invite Team Member</CardTitle>
          <CardDescription>
            Send an invitation to join {currentCompany?.name}
          </CardDescription>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
            <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
              <div className="space-y-2">
                <Label htmlFor="email">Email</Label>
                <Input
                  id="email"
                  type="email"
                  placeholder="colleague@company.com"
                  {...register('email')}
                />
                {errors.email && (
                  <p className="text-sm text-destructive">{errors.email.message}</p>
                )}
              </div>

              <div className="space-y-2">
                <Label>Role</Label>
                <Select onValueChange={(v) => setValue('role', v as any)} defaultValue="user">
                  <SelectTrigger>
                    <SelectValue />
                  </SelectTrigger>
                  <SelectContent>
                    <SelectItem value="admin">Admin</SelectItem>
                    <SelectItem value="manager">Manager</SelectItem>
                    <SelectItem value="user">User</SelectItem>
                  </SelectContent>
                </Select>
              </div>

              <div className="space-y-2">
                <Label>Department (optional)</Label>
                <Select onValueChange={(v) => setValue('departmentId', v)}>
                  <SelectTrigger>
                    <SelectValue placeholder="Select department" />
                  </SelectTrigger>
                  <SelectContent>
                    {departments?.map((dept) => (
                      <SelectItem key={dept.id} value={dept.id}>
                        {dept.name}
                      </SelectItem>
                    ))}
                  </SelectContent>
                </Select>
              </div>
            </div>

            <Button type="submit" disabled={createMutation.isPending}>
              {createMutation.isPending ? 'Sending...' : 'Send Invitation'}
            </Button>
          </form>
        </CardContent>
      </Card>

      <Card>
        <CardHeader>
          <CardTitle>Pending Invitations</CardTitle>
        </CardHeader>
        <CardContent>
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead>Email</TableHead>
                <TableHead>Role</TableHead>
                <TableHead>Department</TableHead>
                <TableHead>Status</TableHead>
                <TableHead>Sent</TableHead>
                <TableHead>Actions</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              {invitations?.map((invite) => (
                <TableRow key={invite.id}>
                  <TableCell>{invite.email}</TableCell>
                  <TableCell className="capitalize">{invite.role}</TableCell>
                  <TableCell>{invite.departments?.name || '-'}</TableCell>
                  <TableCell>
                    <Badge
                      variant={
                        invite.status === 'accepted'
                          ? 'default'
                          : invite.status === 'pending'
                          ? 'secondary'
                          : 'destructive'
                      }
                    >
                      {invite.status}
                    </Badge>
                  </TableCell>
                  <TableCell>
                    {formatDistanceToNow(new Date(invite.created_at), { addSuffix: true })}
                  </TableCell>
                  <TableCell>
                    <div className="flex gap-2">
                      {invite.status === 'pending' && (
                        <>
                          <Button
                            variant="ghost"
                            size="sm"
                            onClick={() => copyInviteLink(invite.token)}
                          >
                            <Copy className="h-4 w-4" />
                          </Button>
                          <Button
                            variant="ghost"
                            size="sm"
                            onClick={() => revokeMutation.mutate(invite.id)}
                          >
                            <Trash2 className="h-4 w-4 text-destructive" />
                          </Button>
                        </>
                      )}
                    </div>
                  </TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </CardContent>
      </Card>
    </div>
  )
}
```

#### 4. Accept Invitation Page

```typescript
// src/pages/AcceptInvitePage.tsx
import { useEffect, useState } from 'react'
import { useParams, useNavigate, Link } from 'react-router-dom'
import { useQuery, useMutation } from '@tanstack/react-query'
import { useAuth } from '@/contexts/AuthContext'
import { useCompany } from '@/contexts/CompanyContext'
import { getInvitationByToken, acceptInvitation } from '@/lib/api/invitations'
import { Button } from '@/components/ui/button'
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { toast } from 'sonner'
import { Building2, CheckCircle, XCircle } from 'lucide-react'

export function AcceptInvitePage() {
  const { token } = useParams<{ token: string }>()
  const { user } = useAuth()
  const { refetch } = useCompany()
  const navigate = useNavigate()

  const { data: invitation, isLoading, error } = useQuery({
    queryKey: ['invitation', token],
    queryFn: () => getInvitationByToken(token!),
    enabled: !!token,
    retry: false,
  })

  const acceptMutation = useMutation({
    mutationFn: () => acceptInvitation(token!),
    onSuccess: async () => {
      await refetch()
      toast.success('Welcome to the team!')
      navigate('/dashboard')
    },
    onError: (error: any) => {
      toast.error(error.message || 'Failed to accept invitation')
    },
  })

  if (isLoading) {
    return (
      <div className="min-h-screen flex items-center justify-center">
        <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-primary" />
      </div>
    )
  }

  if (error || !invitation) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-background">
        <Card className="w-full max-w-md">
          <CardContent className="pt-6 text-center">
            <XCircle className="h-12 w-12 text-destructive mx-auto mb-4" />
            <h2 className="text-xl font-semibold mb-2">Invalid Invitation</h2>
            <p className="text-muted-foreground mb-4">
              This invitation link is invalid, expired, or has already been used.
            </p>
            <Button asChild>
              <Link to="/login">Go to Login</Link>
            </Button>
          </CardContent>
        </Card>
      </div>
    )
  }

  // User not logged in - show login prompt
  if (!user) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-background">
        <Card className="w-full max-w-md">
          <CardHeader className="text-center">
            <Building2 className="h-12 w-12 mx-auto mb-4 text-primary" />
            <CardTitle>You're Invited!</CardTitle>
            <CardDescription>
              {invitation.companies.name} has invited you to join as {invitation.role}
            </CardDescription>
          </CardHeader>
          <CardContent className="space-y-4">
            <p className="text-center text-muted-foreground">
              Please sign in or create an account with <strong>{invitation.email}</strong> to accept this invitation.
            </p>
            <div className="flex gap-4">
              <Button asChild className="flex-1">
                <Link to={`/login?redirect=/invite/${token}`}>Sign In</Link>
              </Button>
              <Button asChild variant="outline" className="flex-1">
                <Link to={`/register?email=${encodeURIComponent(invitation.email)}&redirect=/invite/${token}`}>
                  Sign Up
                </Link>
              </Button>
            </div>
          </CardContent>
        </Card>
      </div>
    )
  }

  // User logged in with different email
  if (user.email?.toLowerCase() !== invitation.email.toLowerCase()) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-background">
        <Card className="w-full max-w-md">
          <CardContent className="pt-6 text-center">
            <XCircle className="h-12 w-12 text-destructive mx-auto mb-4" />
            <h2 className="text-xl font-semibold mb-2">Wrong Account</h2>
            <p className="text-muted-foreground mb-4">
              This invitation was sent to <strong>{invitation.email}</strong>, but you're signed in as <strong>{user.email}</strong>.
            </p>
            <p className="text-muted-foreground mb-4">
              Please sign in with the correct account.
            </p>
          </CardContent>
        </Card>
      </div>
    )
  }

  // Ready to accept
  return (
    <div className="min-h-screen flex items-center justify-center bg-background">
      <Card className="w-full max-w-md">
        <CardHeader className="text-center">
          {invitation.companies.logo_url ? (
            <img
              src={invitation.companies.logo_url}
              alt={invitation.companies.name}
              className="h-16 w-16 mx-auto mb-4 rounded-lg"
            />
          ) : (
            <Building2 className="h-12 w-12 mx-auto mb-4 text-primary" />
          )}
          <CardTitle>Join {invitation.companies.name}</CardTitle>
          <CardDescription>
            You've been invited to join as <strong className="capitalize">{invitation.role}</strong>
          </CardDescription>
        </CardHeader>
        <CardContent className="space-y-4">
          <Button
            className="w-full"
            onClick={() => acceptMutation.mutate()}
            disabled={acceptMutation.isPending}
          >
            {acceptMutation.isPending ? 'Joining...' : 'Accept Invitation'}
          </Button>
          <Button variant="ghost" className="w-full" asChild>
            <Link to="/dashboard">Decline</Link>
          </Button>
        </CardContent>
      </Card>
    </div>
  )
}
```

### Done When

- [ ] Invitations table with token-based access
- [ ] Admin can create invitations
- [ ] Invite link can be copied
- [ ] User can accept invite (logged in)
- [ ] User prompted to login/register (logged out)
- [ ] Email validation (invite email = user email)
- [ ] Invitations can be revoked
- [ ] Expiry enforced
- [ ] Audit log for accepted invites

### Commit

```bash
git add .
git commit -m "feat(invitations): add team invitation system"
git checkout develop
git merge phase-2.5/invitations
```

---

## Phase 3: Core Product Features

**Branch:** `develop → phase-3/core-features`

### Learn First (2-3 hours)

| Topic | Resource |
|-------|----------|
| TanStack Query Mutations | https://tanstack.com/query/latest/docs/react/guides/mutations |
| React Hook Form Advanced | https://react-hook-form.com/advanced-usage |
| Recharts | https://recharts.org/en-US/examples |

### What You're Building

```
┌─────────────────────────────────────────────────────────────────┐
│                    ROLE-BASED DASHBOARDS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ADMIN DASHBOARD                                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ • User Management (CRUD)                                 │   │
│  │ • Department Management                                  │   │
│  │ • Team Management                                        │   │
│  │ • Company Settings                                       │   │
│  │ • Permissions & Roles                                    │   │
│  │ • Audit Logs                                            │   │
│  │ • Analytics Dashboard                                    │   │
│  │ • Billing & Subscription                                 │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  MANAGER DASHBOARD                                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ • Team Overview                                          │   │
│  │ • Assign Tasks                                           │   │
│  │ • View Reports                                           │   │
│  │ • Limited User Control                                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  USER DASHBOARD                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ • Personal Dashboard                                     │   │
│  │ • Profile Settings                                       │   │
│  │ • Assigned Work                                          │   │
│  │ • Activity Feed                                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Do

#### 1. Database Schema - Tasks & Activity

```sql
-- ============================================
-- PHASE 3: CORE FEATURES SCHEMA
-- ============================================

-- ============================================
-- TEAMS TABLE
-- ============================================
CREATE TABLE teams (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_id UUID NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  department_id UUID REFERENCES departments(id) ON DELETE SET NULL,
  name TEXT NOT NULL,
  description TEXT,
  lead_id UUID REFERENCES auth.users(id) ON DELETE SET NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(company_id, name)
);

ALTER TABLE teams ENABLE ROW LEVEL SECURITY;

-- ============================================
-- TEAM MEMBERS
-- ============================================
CREATE TABLE team_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  role TEXT DEFAULT 'member' CHECK (role IN ('lead', 'member')),
  joined_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(team_id, user_id)
);

ALTER TABLE team_members ENABLE ROW LEVEL SECURITY;

-- ============================================
-- TASKS TABLE
-- ============================================
CREATE TABLE tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_id UUID NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  team_id UUID REFERENCES teams(id) ON DELETE SET NULL,
  assigned_to UUID REFERENCES auth.users(id) ON DELETE SET NULL,
  created_by UUID NOT NULL REFERENCES auth.users(id),
  title TEXT NOT NULL,
  description TEXT,
  status TEXT DEFAULT 'todo' CHECK (status IN ('todo', 'in_progress', 'review', 'done', 'cancelled')),
  priority TEXT DEFAULT 'medium' CHECK (priority IN ('low', 'medium', 'high', 'urgent')),
  due_date DATE,
  completed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;

-- ============================================
-- ACTIVITY FEED
-- ============================================
CREATE TABLE activities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_id UUID NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  action TEXT NOT NULL,
  entity_type TEXT NOT NULL,
  entity_id UUID,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE activities ENABLE ROW LEVEL SECURITY;

-- ============================================
-- RLS POLICIES
-- ============================================

-- Teams
CREATE POLICY "Members can view teams"
  ON teams FOR SELECT
  USING (is_company_member(auth.uid(), company_id));

CREATE POLICY "Admins/Managers can create teams"
  ON teams FOR INSERT
  WITH CHECK (is_company_admin_or_manager(auth.uid(), company_id));

CREATE POLICY "Admins/Managers can update teams"
  ON teams FOR UPDATE
  USING (is_company_admin_or_manager(auth.uid(), company_id));

CREATE POLICY "Admins can delete teams"
  ON teams FOR DELETE
  USING (is_company_admin(auth.uid(), company_id));

-- Team Members
CREATE POLICY "Members can view team members"
  ON team_members FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM teams t
      WHERE t.id = team_id
        AND is_company_member(auth.uid(), t.company_id)
    )
  );

CREATE POLICY "Admins/Managers can manage team members"
  ON team_members FOR ALL
  USING (
    EXISTS (
      SELECT 1 FROM teams t
      WHERE t.id = team_id
        AND is_company_admin_or_manager(auth.uid(), t.company_id)
    )
  );

-- Tasks
CREATE POLICY "Members can view company tasks"
  ON tasks FOR SELECT
  USING (is_company_member(auth.uid(), company_id));

CREATE POLICY "Members can create tasks"
  ON tasks FOR INSERT
  WITH CHECK (is_company_member(auth.uid(), company_id));

CREATE POLICY "Assigned user or admin/manager can update tasks"
  ON tasks FOR UPDATE
  USING (
    assigned_to = auth.uid() 
    OR created_by = auth.uid()
    OR is_company_admin_or_manager(auth.uid(), company_id)
  );

CREATE POLICY "Admins can delete tasks"
  ON tasks FOR DELETE
  USING (is_company_admin(auth.uid(), company_id));

-- Activities
CREATE POLICY "Members can view activities"
  ON activities FOR SELECT
  USING (is_company_member(auth.uid(), company_id));

CREATE POLICY "System can insert activities"
  ON activities FOR INSERT
  WITH CHECK (is_company_member(auth.uid(), company_id));

-- ============================================
-- ACTIVITY LOGGING FUNCTION
-- ============================================
CREATE OR REPLACE FUNCTION log_activity(
  p_company_id UUID,
  p_action TEXT,
  p_entity_type TEXT,
  p_entity_id UUID DEFAULT NULL,
  p_metadata JSONB DEFAULT '{}'
)
RETURNS UUID
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
DECLARE
  v_id UUID;
BEGIN
  INSERT INTO activities (company_id, user_id, action, entity_type, entity_id, metadata)
  VALUES (p_company_id, auth.uid(), p_action, p_entity_type, p_entity_id, p_metadata)
  RETURNING id INTO v_id;
  
  RETURN v_id;
END;
$$;

-- ============================================
-- TRIGGERS FOR AUTO ACTIVITY LOGGING
-- ============================================
CREATE OR REPLACE FUNCTION log_task_activity()
RETURNS TRIGGER
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  IF TG_OP = 'INSERT' THEN
    PERFORM log_activity(
      NEW.company_id,
      'task_created',
      'task',
      NEW.id,
      jsonb_build_object('title', NEW.title)
    );
  ELSIF TG_OP = 'UPDATE' THEN
    IF OLD.status != NEW.status THEN
      PERFORM log_activity(
        NEW.company_id,
        'task_status_changed',
        'task',
        NEW.id,
        jsonb_build_object('from', OLD.status, 'to', NEW.status, 'title', NEW.title)
      );
    END IF;
    IF OLD.assigned_to IS DISTINCT FROM NEW.assigned_to THEN
      PERFORM log_activity(
        NEW.company_id,
        'task_assigned',
        'task',
        NEW.id,
        jsonb_build_object('assigned_to', NEW.assigned_to, 'title', NEW.title)
      );
    END IF;
  END IF;
  RETURN NEW;
END;
$$;

CREATE TRIGGER task_activity_trigger
  AFTER INSERT OR UPDATE ON tasks
  FOR EACH ROW EXECUTE FUNCTION log_task_activity();

-- Timestamps
CREATE TRIGGER update_teams_updated_at
  BEFORE UPDATE ON teams
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_tasks_updated_at
  BEFORE UPDATE ON tasks
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

#### 2. Dashboard Layout

```typescript
// src/components/layout/DashboardLayout.tsx
import { Outlet, Link, useLocation, useNavigate } from 'react-router-dom'
import {
  LayoutDashboard,
  Users,
  Building2,
  FolderKanban,
  Settings,
  Bell,
  FileText,
  BarChart3,
  CreditCard,
  LogOut,
  Menu,
} from 'lucide-react'
import { cn } from '@/lib/utils'
import { useAuth } from '@/contexts/AuthContext'
import { useCompany } from '@/contexts/CompanyContext'
import { CompanySwitcher } from '@/components/CompanySwitcher'
import { Button } from '@/components/ui/button'
import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu'
import { useState } from 'react'

interface NavItem {
  href: string
  label: string
  icon: React.ComponentType<{ className?: string }>
  roles?: ('admin' | 'manager' | 'user')[]
}

const navItems: NavItem[] = [
  { href: '/dashboard', label: 'Overview', icon: LayoutDashboard },
  { href: '/dashboard/users', label: 'Users', icon: Users, roles: ['admin'] },
  { href: '/dashboard/departments', label: 'Departments', icon: Building2, roles: ['admin'] },
  { href: '/dashboard/teams', label: 'Teams', icon: FolderKanban, roles: ['admin', 'manager'] },
  { href: '/dashboard/tasks', label: 'Tasks', icon: FileText },
  { href: '/dashboard/analytics', label: 'Analytics', icon: BarChart3, roles: ['admin', 'manager'] },
  { href: '/dashboard/billing', label: 'Billing', icon: CreditCard, roles: ['admin'] },
  { href: '/dashboard/settings', label: 'Settings', icon: Settings, roles: ['admin'] },
]

export function DashboardLayout() {
  const { pathname } = useLocation()
  const { user, signOut } = useAuth()
  const { currentCompany, currentRole } = useCompany()
  const navigate = useNavigate()
  const [sidebarOpen, setSidebarOpen] = useState(true)

  const filteredNavItems = navItems.filter(
    (item) => !item.roles || (currentRole && item.roles.includes(currentRole))
  )

  const handleSignOut = async () => {
    await signOut()
    navigate('/login')
  }

  return (
    <div className="min-h-screen bg-background">
      {/* Sidebar */}
      <aside
        className={cn(
          'fixed left-0 top-0 z-40 h-screen bg-card border-r transition-all duration-300',
          sidebarOpen ? 'w-64' : 'w-16'
        )}
      >
        {/* Logo */}
        <div className="flex h-16 items-center justify-between px-4 border-b">
          {sidebarOpen && (
            <Link to="/dashboard" className="flex items-center gap-2">
              <div className="h-8 w-8 rounded-lg bg-primary flex items-center justify-center">
                <span className="text-primary-foreground font-bold">N</span>
              </div>
              <span className="font-bold text-lg">NovaPulse</span>
            </Link>
          )}
          <Button
            variant="ghost"
            size="sm"
            onClick={() => setSidebarOpen(!sidebarOpen)}
          >
            <Menu className="h-5 w-5" />
          </Button>
        </div>

        {/* Company Switcher */}
        {sidebarOpen && (
          <div className="p-4 border-b">
            <CompanySwitcher />
          </div>
        )}

        {/* Navigation */}
        <nav className="p-4 space-y-1">
          {filteredNavItems.map((item) => {
            const Icon = item.icon
            const isActive = pathname === item.href || pathname.startsWith(item.href + '/')
            
            return (
              <Link
                key={item.href}
                to={item.href}
                className={cn(
                  'flex items-center gap-3 px-3 py-2 rounded-md transition-colors',
                  isActive
                    ? 'bg-primary text-primary-foreground'
                    : 'text-muted-foreground hover:bg-muted hover:text-foreground',
                  !sidebarOpen && 'justify-center'
                )}
              >
                <Icon className="h-5 w-5 flex-shrink-0" />
                {sidebarOpen && <span>{item.label}</span>}
              </Link>
            )
          })}
        </nav>
      </aside>

      {/* Main Content */}
      <div className={cn('transition-all duration-300', sidebarOpen ? 'ml-64' : 'ml-16')}>
        {/* Header */}
        <header className="sticky top-0 z-30 h-16 bg-card border-b flex items-center justify-between px-6">
          <div className="flex items-center gap-4">
            <h1 className="text-lg font-semibold">
              {currentCompany?.name || 'Select a company'}
            </h1>
            {currentRole && (
              <span className="text-xs bg-muted px-2 py-1 rounded capitalize">
                {currentRole}
              </span>
            )}
          </div>

          <div className="flex items-center gap-4">
            <Button variant="ghost" size="icon">
              <Bell className="h-5 w-5" />
            </Button>

            <DropdownMenu>
              <DropdownMenuTrigger asChild>
                <Button variant="ghost" className="relative h-10 w-10 rounded-full">
                  <Avatar className="h-10 w-10">
                    <AvatarImage src="" />
                    <AvatarFallback>
                      {user?.email?.charAt(0).toUpperCase()}
                    </AvatarFallback>
                  </Avatar>
                </Button>
              </DropdownMenuTrigger>
              <DropdownMenuContent align="end" className="w-56">
                <div className="flex items-center justify-start gap-2 p-2">
                  <div className="flex flex-col space-y-1 leading-none">
                    <p className="font-medium">{user?.email}</p>
                    <p className="text-xs text-muted-foreground capitalize">
                      {currentRole}
                    </p>
                  </div>
                </div>
                <DropdownMenuSeparator />
                <DropdownMenuItem asChild>
                  <Link to="/dashboard/profile">Profile</Link>
                </DropdownMenuItem>
                <DropdownMenuItem asChild>
                  <Link to="/dashboard/settings">Settings</Link>
                </DropdownMenuItem>
                <DropdownMenuSeparator />
                <DropdownMenuItem onClick={handleSignOut}>
                  <LogOut className="mr-2 h-4 w-4" />
                  Sign out
                </DropdownMenuItem>
              </DropdownMenuContent>
            </DropdownMenu>
          </div>
        </header>

        {/* Page Content */}
        <main className="p-6">
          <Outlet />
        </main>
      </div>
    </div>
  )
}
```

#### 3. Admin Dashboard Overview

```typescript
// src/pages/dashboard/OverviewPage.tsx
import { useQuery } from '@tanstack/react-query'
import { useCompany } from '@/contexts/CompanyContext'
import { supabase } from '@/lib/supabase'
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { Users, Building2, FolderKanban, CheckCircle, Clock, AlertTriangle } from 'lucide-react'
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, PieChart, Pie, Cell } from 'recharts'
import { RoleGuard } from '@/components/RoleGuard'
import { ActivityFeed } from '@/components/ActivityFeed'

interface DashboardStats {
  totalUsers: number
  totalDepartments: number
  totalTeams: number
  tasksCompleted: number
  tasksPending: number
  tasksOverdue: number
}

async function getDashboardStats(companyId: string): Promise<DashboardStats> {
  const [users, departments, teams, tasks] = await Promise.all([
    supabase.from('company_members').select('id', { count: 'exact' }).eq('company_id', companyId).eq('is_active', true),
    supabase.from('departments').select('id', { count: 'exact' }).eq('company_id', companyId),
    supabase.from('teams').select('id', { count: 'exact' }).eq('company_id', companyId),
    supabase.from('tasks').select('status, due_date').eq('company_id', companyId),
  ])

  const today = new Date().toISOString().split('T')[0]
  const taskData = tasks.data || []

  return {
    totalUsers: users.count || 0,
    totalDepartments: departments.count || 0,
    totalTeams: teams.count || 0,
    tasksCompleted: taskData.filter(t => t.status === 'done').length,
    tasksPending: taskData.filter(t => t.status !== 'done' && t.status !== 'cancelled').length,
    tasksOverdue: taskData.filter(t => t.due_date && t.due_date < today && t.status !== 'done').length,
  }
}

const COLORS = ['#22c55e', '#3b82f6', '#f59e0b', '#ef4444']

export function OverviewPage() {
  const { currentCompany, currentRole } = useCompany()

  const { data: stats, isLoading } = useQuery({
    queryKey: ['dashboard-stats', currentCompany?.id],
    queryFn: () => getDashboardStats(currentCompany!.id),
    enabled: !!currentCompany,
  })

  if (!currentCompany) {
    return (
      <div className="flex items-center justify-center h-64">
        <p className="text-muted-foreground">Please select or create a company to continue.</p>
      </div>
    )
  }

  if (isLoading) {
    return (
      <div className="flex items-center justify-center h-64">
        <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-primary" />
      </div>
    )
  }

  const taskPieData = [
    { name: 'Completed', value: stats?.tasksCompleted || 0 },
    { name: 'Pending', value: stats?.tasksPending || 0 },
    { name: 'Overdue', value: stats?.tasksOverdue || 0 },
  ]

  return (
    <div className="space-y-6">
      <div>
        <h2 className="text-2xl font-bold">Welcome back!</h2>
        <p className="text-muted-foreground">
          Here's what's happening in {currentCompany.name}
        </p>
      </div>

      {/* Stats Cards - Admin/Manager only */}
      <RoleGuard allowedRoles={['admin', 'manager']}>
        <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
          <Card>
            <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
              <CardTitle className="text-sm font-medium">Total Users</CardTitle>
              <Users className="h-4 w-4 text-muted-foreground" />
            </CardHeader>
            <CardContent>
              <div className="text-2xl font-bold">{stats?.totalUsers}</div>
              <p className="text-xs text-muted-foreground">Active team members</p>
            </CardContent>
          </Card>

          <Card>
            <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
              <CardTitle className="text-sm font-medium">Departments</CardTitle>
              <Building2 className="h-4 w-4 text-muted-foreground" />
            </CardHeader>
            <CardContent>
              <div className="text-2xl font-bold">{stats?.totalDepartments}</div>
              <p className="text-xs text-muted-foreground">Organization units</p>
            </CardContent>
          </Card>

          <Card>
            <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
              <CardTitle className="text-sm font-medium">Teams</CardTitle>
              <FolderKanban className="h-4 w-4 text-muted-foreground" />
            </CardHeader>
            <CardContent>
              <div className="text-2xl font-bold">{stats?.totalTeams}</div>
              <p className="text-xs text-muted-foreground">Active teams</p>
            </CardContent>
          </Card>

          <Card>
            <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
              <CardTitle className="text-sm font-medium">Tasks Completed</CardTitle>
              <CheckCircle className="h-4 w-4 text-muted-foreground" />
            </CardHeader>
            <CardContent>
              <div className="text-2xl font-bold">{stats?.tasksCompleted}</div>
              <p className="text-xs text-muted-foreground">
                {stats?.tasksOverdue ? (
                  <span className="text-destructive">
                    {stats.tasksOverdue} overdue
                  </span>
                ) : (
                  'All on track'
                )}
              </p>
            </CardContent>
          </Card>
        </div>
      </RoleGuard>

      <div className="grid gap-6 md:grid-cols-2">
        {/* Task Status Chart */}
        <Card>
          <CardHeader>
            <CardTitle>Task Status</CardTitle>
            <CardDescription>Overview of all tasks</CardDescription>
          </CardHeader>
          <CardContent>
            <div className="h-[300px]">
              <ResponsiveContainer width="100%" height="100%">
                <PieChart>
                  <Pie
                    data={taskPieData}
                    cx="50%"
                    cy="50%"
                    innerRadius={60}
                    outerRadius={100}
                    paddingAngle={5}
                    dataKey="value"
                    label={({ name, value }) => `${name}: ${value}`}
                  >
                    {taskPieData.map((entry, index) => (
                      <Cell key={`cell-${index}`} fill={COLORS[index % COLORS.length]} />
                    ))}
                  </Pie>
                  <Tooltip />
                </PieChart>
              </ResponsiveContainer>
            </div>
          </CardContent>
        </Card>

        {/* Recent Activity */}
        <Card>
          <CardHeader>
            <CardTitle>Recent Activity</CardTitle>
            <CardDescription>Latest actions in your company</CardDescription>
          </CardHeader>
          <CardContent>
            <ActivityFeed companyId={currentCompany.id} limit={5} />
          </CardContent>
        </Card>
      </div>

      {/* My Tasks Section - For regular users */}
      {currentRole === 'user' && (
        <Card>
          <CardHeader>
            <CardTitle>My Tasks</CardTitle>
            <CardDescription>Tasks assigned to you</CardDescription>
          </CardHeader>
          <CardContent>
            <MyTasksList companyId={currentCompany.id} />
          </CardContent>
        </Card>
      )}
    </div>
  )
}

// My Tasks component for users
function MyTasksList({ companyId }: { companyId: string }) {
  const { data: tasks, isLoading } = useQuery({
    queryKey: ['my-tasks', companyId],
    queryFn: async () => {
      const { data: { user } } = await supabase.auth.getUser()
      const { data, error } = await supabase
        .from('tasks')
        .select('*')
        .eq('company_id', companyId)
        .eq('assigned_to', user?.id)
        .neq('status', 'done')
        .order('due_date', { ascending: true })
        .limit(10)
      
      if (error) throw error
      return data
    },
  })

  if (isLoading) return <div>Loading...</div>

  if (!tasks?.length) {
    return <p className="text-muted-foreground">No tasks assigned to you.</p>
  }

  return (
    <div className="space-y-3">
      {tasks.map((task) => (
        <div
          key={task.id}
          className="flex items-center justify-between p-3 border rounded-lg"
        >
          <div>
            <p className="font-medium">{task.title}</p>
            <p className="text-sm text-muted-foreground">
              Due: {task.due_date || 'No due date'}
            </p>
          </div>
          <span
            className={cn(
              'text-xs px-2 py-1 rounded capitalize',
              task.priority === 'urgent' && 'bg-red-100 text-red-700',
              task.priority === 'high' && 'bg-orange-100 text-orange-700',
              task.priority === 'medium' && 'bg-yellow-100 text-yellow-700',
              task.priority === 'low' && 'bg-green-100 text-green-700'
            )}
          >
            {task.priority}
          </span>
        </div>
      ))}
    </div>
  )
}
```

#### 4. User Management Page (Admin Only)

```typescript
// src/pages/dashboard/UsersPage.tsx
import { useState } from 'react'
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { useCompany } from '@/contexts/CompanyContext'
import { supabase } from '@/lib/supabase'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table'
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select'
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
} from '@/components/ui/dialog'
import { Badge } from '@/components/ui/badge'
import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar'
import { toast } from 'sonner'
import { Search, MoreHorizontal, UserPlus, Mail, Trash2 } from 'lucide-react'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu'
import { formatDistanceToNow } from 'date-fns'
import { Link } from 'react-router-dom'

interface CompanyMember {
  id: string
  user_id: string
  role: 'admin' | 'manager' | 'user'
  department_id: string | null
  job_title: string | null
  is_active: boolean
  joined_at: string
  profiles: {
    email: string
    full_name: string
    avatar_url: string | null
  }
  departments: {
    name: string
  } | null
}

async function getCompanyMembers(companyId: string): Promise<CompanyMember[]> {
  const { data, error } = await supabase
    .from('company_members')
    .select(`
      *,
      profiles!user_id (email, full_name, avatar_url),
      departments (name)
    `)
    .eq('company_id', companyId)
    .order('joined_at', { ascending: false })

  if (error) throw error
  return data as CompanyMember[]
}

export function UsersPage() {
  const { currentCompany } = useCompany()
  const queryClient = useQueryClient()
  const [search, setSearch] = useState('')
  const [roleFilter, setRoleFilter] = useState<string>('all')

  const { data: members, isLoading } = useQuery({
    queryKey: ['company-members', currentCompany?.id],
    queryFn: () => getCompanyMembers(currentCompany!.id),
    enabled: !!currentCompany,
  })

  const updateRoleMutation = useMutation({
    mutationFn: async ({ memberId, role }: { memberId: string; role: string }) => {
      const { error } = await supabase
        .from('company_members')
        .update({ role })
        .eq('id', memberId)

      if (error) throw error
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['company-members'] })
      toast.success('Role updated successfully')
    },
    onError: () => {
      toast.error('Failed to update role')
    },
  })

  const toggleActiveMutation = useMutation({
    mutationFn: async ({ memberId, isActive }: { memberId: string; isActive: boolean }) => {
      const { error } = await supabase
        .from('company_members')
        .update({ is_active: isActive })
        .eq('id', memberId)

      if (error) throw error
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['company-members'] })
      toast.success('User status updated')
    },
  })

  const filteredMembers = members?.filter((member) => {
    const matchesSearch =
      member.profiles.full_name?.toLowerCase().includes(search.toLowerCase()) ||
      member.profiles.email.toLowerCase().includes(search.toLowerCase())
    const matchesRole = roleFilter === 'all' || member.role === roleFilter
    return matchesSearch && matchesRole
  })

  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <div>
          <h2 className="text-2xl font-bold">Users</h2>
          <p className="text-muted-foreground">
            Manage team members and their roles
          </p>
        </div>
        <Button asChild>
          <Link to="/dashboard/invite">
            <UserPlus className="mr-2 h-4 w-4" />
            Invite User
          </Link>
        </Button>
      </div>

      {/* Filters */}
      <div className="flex gap-4">
        <div className="relative flex-1 max-w-sm">
          <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
          <Input
            placeholder="Search users..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="pl-9"
          />
        </div>
        <Select value={roleFilter} onValueChange={setRoleFilter}>
          <SelectTrigger className="w-[180px]">
            <SelectValue placeholder="Filter by role" />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="all">All roles</SelectItem>
            <SelectItem value="admin">Admin</SelectItem>
            <SelectItem value="manager">Manager</SelectItem>
            <SelectItem value="user">User</SelectItem>
          </SelectContent>
        </Select>
      </div>

      {/* Users Table */}
      <div className="border rounded-lg">
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>User</TableHead>
              <TableHead>Role</TableHead>
              <TableHead>Department</TableHead>
              <TableHead>Status</TableHead>
              <TableHead>Joined</TableHead>
              <TableHead className="w-[50px]"></TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {isLoading ? (
              <TableRow>
                <TableCell colSpan={6} className="text-center py-8">
                  Loading...
                </TableCell>
              </TableRow>
            ) : filteredMembers?.length === 0 ? (
              <TableRow>
                <TableCell colSpan={6} className="text-center py-8">
                  No users found
                </TableCell>
              </TableRow>
            ) : (
              filteredMembers?.map((member) => (
                <TableRow key={member.id}>
                  <TableCell>
                    <div className="flex items-center gap-3">
                      <Avatar className="h-9 w-9">
                        <AvatarImage src={member.profiles.avatar_url || ''} />
                        <AvatarFallback>
                          {member.profiles.full_name?.charAt(0) || member.profiles.email.charAt(0)}
                        </AvatarFallback>
                      </Avatar>
                      <div>
                        <p className="font-medium">{member.profiles.full_name}</p>
                        <p className="text-sm text-muted-foreground">{member.profiles.email}</p>
                      </div>
                    </div>
                  </TableCell>
                  <TableCell>
                    <Select
                      value={member.role}
                      onValueChange={(value) =>
                        updateRoleMutation.mutate({ memberId: member.id, role: value })
                      }
                    >
                      <SelectTrigger className="w-[110px]">
                        <SelectValue />
                      </SelectTrigger>
                      <SelectContent>
                        <SelectItem value="admin">Admin</SelectItem>
                        <SelectItem value="manager">Manager</SelectItem>
                        <SelectItem value="user">User</SelectItem>
                      </SelectContent>
                    </Select>
                  </TableCell>
                  <TableCell>{member.departments?.name || '-'}</TableCell>
                  <TableCell>
                    <Badge variant={member.is_active ? 'default' : 'secondary'}>
                      {member.is_active ? 'Active' : 'Inactive'}
                    </Badge>
                  </TableCell>
                  <TableCell className="text-muted-foreground">
                    {formatDistanceToNow(new Date(member.joined_at), { addSuffix: true })}
                  </TableCell>
                  <TableCell>
                    <DropdownMenu>
                      <DropdownMenuTrigger asChild>
                        <Button variant="ghost" size="icon">
                          <MoreHorizontal className="h-4 w-4" />
                        </Button>
                      </DropdownMenuTrigger>
                      <DropdownMenuContent align="end">
                        <DropdownMenuItem>
                          <Mail className="mr-2 h-4 w-4" />
                          Send Email
                        </DropdownMenuItem>
                        <DropdownMenuSeparator />
                        <DropdownMenuItem
                          onClick={() =>
                            toggleActiveMutation.mutate({
                              memberId: member.id,
                              isActive: !member.is_active,
                            })
                          }
                        >
                          {member.is_active ? 'Deactivate' : 'Activate'}
                        </DropdownMenuItem>
                        <DropdownMenuItem className="text-destructive">
                          <Trash2 className="mr-2 h-4 w-4" />
                          Remove
                        </DropdownMenuItem>
                      </DropdownMenuContent>
                    </DropdownMenu>
                  </TableCell>
                </TableRow>
              ))
            )}
          </TableBody>
        </Table>
      </div>
    </div>
  )
}
```

#### 5. Activity Feed Component

```typescript
// src/components/ActivityFeed.tsx
import { useQuery } from '@tanstack/react-query'
import { supabase } from '@/lib/supabase'
import { formatDistanceToNow } from 'date-fns'
import { Avatar, AvatarFallback } from '@/components/ui/avatar'
import {
  UserPlus,
  CheckCircle,
  Edit,
  Trash2,
  FolderPlus,
  Users,
  ArrowRight,
} from 'lucide-react'

interface Activity {
  id: string
  user_id: string
  action: string
  entity_type: string
  entity_id: string | null
  metadata: Record<string, any>
  created_at: string
  profiles: {
    full_name: string
    email: string
  }
}

const actionIcons: Record<string, React.ComponentType<{ className?: string }>> = {
  task_created: FolderPlus,
  task_status_changed: ArrowRight,
  task_assigned: Users,
  task_completed: CheckCircle,
  user_invited: UserPlus,
  invitation_accepted: UserPlus,
  default: Edit,
}

const actionLabels: Record<string, (metadata: Record<string, any>) => string> = {
  task_created: (m) => `created task "${m.title}"`,
  task_status_changed: (m) => `moved task "${m.title}" from ${m.from} to ${m.to}`,
  task_assigned: (m) => `assigned task "${m.title}"`,
  user_invited: (m) => `invited ${m.email}`,
  invitation_accepted: () => `joined the company`,
  default: () => 'performed an action',
}

export function ActivityFeed({ companyId, limit = 10 }: { companyId: string; limit?: number }) {
  const { data: activities, isLoading } = useQuery({
    queryKey: ['activities', companyId, limit],
    queryFn: async () => {
      const { data, error } = await supabase
        .from('activities')
        .select(`
          *,
          profiles!user_id (full_name, email)
        `)
        .eq('company_id', companyId)
        .order('created_at', { ascending: false })
        .limit(limit)

      if (error) throw error
      return data as Activity[]
    },
  })

  if (isLoading) {
    return <div className="animate-pulse space-y-3">
      {[...Array(3)].map((_, i) => (
        <div key={i} className="flex gap-3">
          <div className="h-8 w-8 bg-muted rounded-full" />
          <div className="flex-1 space-y-2">
            <div className="h-4 bg-muted rounded w-3/4" />
            <div className="h-3 bg-muted rounded w-1/4" />
          </div>
        </div>
      ))}
    </div>
  }

  if (!activities?.length) {
    return <p className="text-muted-foreground text-sm">No recent activity</p>
  }

  return (
    <div className="space-y-4">
      {activities.map((activity) => {
        const Icon = actionIcons[activity.action] || actionIcons.default
        const getLabel = actionLabels[activity.action] || actionLabels.default

        return (
          <div key={activity.id} className="flex gap-3">
            <Avatar className="h-8 w-8">
              <AvatarFallback className="text-xs">
                {activity.profiles.full_name?.charAt(0) || '?'}
              </AvatarFallback>
            </Avatar>
            <div className="flex-1 min-w-0">
              <p className="text-sm">
                <span className="font-medium">{activity.profiles.full_name}</span>{' '}
                {getLabel(activity.metadata)}
              </p>
              <p className="text-xs text-muted-foreground">
                {formatDistanceToNow(new Date(activity.created_at), { addSuffix: true })}
              </p>
            </div>
            <Icon className="h-4 w-4 text-muted-foreground flex-shrink-0 mt-1" />
          </div>
        )
      })}
    </div>
  )
}
```

#### 6. Update Router

```typescript
// src/App.tsx - Add new routes
import { DashboardLayout } from '@/components/layout/DashboardLayout'
import { OverviewPage } from '@/pages/dashboard/OverviewPage'
import { UsersPage } from '@/pages/dashboard/UsersPage'
import { DepartmentsPage } from '@/pages/dashboard/DepartmentsPage'
import { TeamsPage } from '@/pages/dashboard/TeamsPage'
import { TasksPage } from '@/pages/dashboard/TasksPage'
import { SettingsPage } from '@/pages/dashboard/SettingsPage'
import { InviteTeamPage } from '@/pages/admin/InviteTeamPage'
import { AcceptInvitePage } from '@/pages/AcceptInvitePage'
import { CreateCompanyPage } from '@/pages/CreateCompanyPage'

// In Routes:
<Route path="/invite/:token" element={<AcceptInvitePage />} />
<Route path="/create-company" element={
  <ProtectedRoute>
    <CreateCompanyPage />
  </ProtectedRoute>
} />

<Route path="/dashboard" element={
  <ProtectedRoute>
    <CompanyProvider>
      <DashboardLayout />
    </CompanyProvider>
  </ProtectedRoute>
}>
  <Route index element={<OverviewPage />} />
  <Route path="users" element={<UsersPage />} />
  <Route path="departments" element={<DepartmentsPage />} />
  <Route path="teams" element={<TeamsPage />} />
  <Route path="tasks" element={<TasksPage />} />
  <Route path="invite" element={<InviteTeamPage />} />
  <Route path="settings" element={<SettingsPage />} />
  <Route path="profile" element={<ProfilePage />} />
</Route>
```

### Done When

- [ ] Dashboard layout with sidebar navigation
- [ ] Role-based navigation (Admin sees more items)
- [ ] Overview page with stats cards
- [ ] Charts showing task status
- [ ] Activity feed working
- [ ] User management page (Admin only)
- [ ] Can change user roles
- [ ] Can activate/deactivate users
- [ ] Users can see their assigned tasks
- [ ] All data scoped to current company

### Commit

```bash
git add .
git commit -m "feat(core): add dashboard, users, and activity feed"
git checkout develop
git merge phase-3/core-features
```

---

## Phase 4: Billing, Files & Background Jobs

**Branch:** `develop → phase-4/billing`

### Learn First (3-4 hours)

| Topic | Resource |
|-------|----------|
| Stripe Subscriptions | https://stripe.com/docs/billing/subscriptions/overview |
| Stripe Webhooks | https://stripe.com/docs/webhooks |
| Supabase Storage | https://supabase.com/docs/guides/storage |
| Supabase Edge Functions | https://supabase.com/docs/guides/functions |

### What You're Building

```
┌─────────────────────────────────────────────────────────────────┐
│                    BILLING ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────────┐      │
│  │  User    │───▶│  Stripe      │───▶│  Webhook         │      │
│  │  Checkout│    │  Checkout    │    │  Handler         │      │
│  └──────────┘    └──────────────┘    └────────┬─────────┘      │
│                                               │                 │
│                                               ▼                 │
│                                      ┌──────────────────┐      │
│                                      │  Update DB       │      │
│                                      │  subscription    │      │
│                                      │  status          │      │
│                                      └──────────────────┘      │
│                                                                 │
│  Plans: Free → Starter → Pro → Enterprise                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Do

#### 1. Database Schema - Subscriptions & Jobs

```sql
-- ============================================
-- PHASE 4: BILLING & JOBS SCHEMA
-- ============================================

-- ============================================
-- SUBSCRIPTIONS TABLE
-- ============================================
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_id UUID NOT NULL REFERENCES companies(id) ON DELETE CASCADE UNIQUE,
  stripe_customer_id TEXT UNIQUE,
  stripe_subscription_id TEXT UNIQUE,
  plan TEXT NOT NULL DEFAULT 'free' CHECK (plan IN ('free', 'starter', 'pro', 'enterprise')),
  status TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'past_due', 'cancelled', 'trialing')),
  current_period_start TIMESTAMPTZ,
  current_period_end TIMESTAMPTZ,
  cancel_at_period_end BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE subscriptions ENABLE ROW LEVEL SECURITY;

-- Only admins can view/manage subscriptions
CREATE POLICY "Admins can view subscription"
  ON subscriptions FOR SELECT
  USING (is_company_admin(auth.uid(), company_id));

CREATE POLICY "System can manage subscriptions"
  ON subscriptions FOR ALL
  USING (true)  -- Webhook uses service role
  WITH CHECK (true);

-- ============================================
-- PLAN LIMITS (Configuration)
-- ============================================
CREATE TABLE plan_limits (
  plan TEXT PRIMARY KEY,
  max_users INT NOT NULL,
  max_storage_gb INT NOT NULL,
  max_departments INT NOT NULL,
  max_teams INT NOT NULL,
  features JSONB DEFAULT '{}'
);

INSERT INTO plan_limits (plan, max_users, max_storage_gb, max_departments, max_teams, features) VALUES
  ('free', 5, 1, 2, 3, '{"api_access": false, "analytics": false, "audit_logs": false}'),
  ('starter', 15, 10, 5, 10, '{"api_access": true, "analytics": false, "audit_logs": true}'),
  ('pro', 50, 50, 20, 50, '{"api_access": true, "analytics": true, "audit_logs": true}'),
  ('enterprise', -1, -1, -1, -1, '{"api_access": true, "analytics": true, "audit_logs": true, "sso": true}');

-- ============================================
-- BACKGROUND JOBS TABLE (DB-backed queue)
-- ============================================
CREATE TABLE background_jobs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  type TEXT NOT NULL,
  payload JSONB NOT NULL DEFAULT '{}',
  status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'processing', 'completed', 'failed')),
  attempts INT DEFAULT 0,
  max_attempts INT DEFAULT 3,
  scheduled_at TIMESTAMPTZ DEFAULT NOW(),
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  error TEXT,
  result JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Index for job processing
CREATE INDEX idx_jobs_pending ON background_jobs (scheduled_at) 
  WHERE status = 'pending';

-- ============================================
-- FILE UPLOADS TRACKING
-- ============================================
CREATE TABLE file_uploads (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  company_id UUID NOT NULL REFERENCES companies(id) ON DELETE CASCADE,
  uploaded_by UUID NOT NULL REFERENCES auth.users(id),
  bucket TEXT NOT NULL,
  path TEXT NOT NULL,
  filename TEXT NOT NULL,
  content_type TEXT,
  size_bytes BIGINT NOT NULL,
  entity_type TEXT, -- 'task', 'profile', 'company', etc.
  entity_id UUID,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE file_uploads ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Members can view company files"
  ON file_uploads FOR SELECT
  USING (is_company_member(auth.uid(), company_id));

CREATE POLICY "Members can upload files"
  ON file_uploads FOR INSERT
  WITH CHECK (is_company_member(auth.uid(), company_id));

-- ============================================
-- HELPER FUNCTIONS
-- ============================================

-- Get company plan limits
CREATE OR REPLACE FUNCTION get_company_limits(p_company_id UUID)
RETURNS TABLE(
  max_users INT,
  max_storage_gb INT,
  max_departments INT,
  max_teams INT,
  features JSONB
)
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
DECLARE
  v_plan TEXT;
BEGIN
  SELECT COALESCE(s.plan, 'free') INTO v_plan
  FROM companies c
  LEFT JOIN subscriptions s ON s.company_id = c.id
  WHERE c.id = p_company_id;

  RETURN QUERY
  SELECT pl.max_users, pl.max_storage_gb, pl.max_departments, pl.max_teams, pl.features
  FROM plan_limits pl
  WHERE pl.plan = v_plan;
END;
$$;

-- Check if company is within limits
CREATE OR REPLACE FUNCTION check_company_limit(
  p_company_id UUID,
  p_limit_type TEXT
)
RETURNS BOOLEAN
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
DECLARE
  v_limits RECORD;
  v_current INT;
BEGIN
  SELECT * INTO v_limits FROM get_company_limits(p_company_id);

  CASE p_limit_type
    WHEN 'users' THEN
      SELECT COUNT(*) INTO v_current FROM company_members WHERE company_id = p_company_id AND is_active = true;
      RETURN v_limits.max_users = -1 OR v_current < v_limits.max_users;
    WHEN 'departments' THEN
      SELECT COUNT(*) INTO v_current FROM departments WHERE company_id = p_company_id;
      RETURN v_limits.max_departments = -1 OR v_current < v_limits.max_departments;
    WHEN 'teams' THEN
      SELECT COUNT(*) INTO v_current FROM teams WHERE company_id = p_company_id;
      RETURN v_limits.max_teams = -1 OR v_current < v_limits.max_teams;
    ELSE
      RETURN true;
  END CASE;
END;
$$;

-- Timestamps
CREATE TRIGGER update_subscriptions_updated_at
  BEFORE UPDATE ON subscriptions
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

#### 2. Stripe Integration

```typescript
// src/lib/stripe.ts
import { loadStripe } from '@stripe/stripe-js'

export const stripePromise = loadStripe(import.meta.env.VITE_STRIPE_PUBLISHABLE_KEY)

export const PLANS = {
  free: {
    name: 'Free',
    price: 0,
    priceId: null,
    features: ['Up to 5 users', '1 GB storage', '2 departments', 'Basic support'],
  },
  starter: {
    name: 'Starter',
    price: 29,
    priceId: import.meta.env.VITE_STRIPE_STARTER_PRICE_ID,
    features: ['Up to 15 users', '10 GB storage', '5 departments', 'API access', 'Email support'],
  },
  pro: {
    name: 'Pro',
    price: 79,
    priceId: import.meta.env.VITE_STRIPE_PRO_PRICE_ID,
    features: ['Up to 50 users', '50 GB storage', '20 departments', 'Analytics', 'Priority support'],
  },
  enterprise: {
    name: 'Enterprise',
    price: null, // Contact sales
    priceId: null,
    features: ['Unlimited users', 'Unlimited storage', 'SSO', 'Dedicated support', 'Custom integrations'],
  },
}
```

#### 3. Edge Function - Create Checkout Session

```typescript
// supabase/functions/create-checkout/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'
import Stripe from 'https://esm.sh/stripe@13?target=deno'

const stripe = new Stripe(Deno.env.get('STRIPE_SECRET_KEY')!, {
  apiVersion: '2023-10-16',
})

const corsHeaders = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
}

serve(async (req) => {
  if (req.method === 'OPTIONS') {
    return new Response('ok', { headers: corsHeaders })
  }

  try {
    const supabase = createClient(
      Deno.env.get('SUPABASE_URL')!,
      Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
    )

    // Get user from auth header
    const authHeader = req.headers.get('Authorization')!
    const { data: { user }, error: authError } = await supabase.auth.getUser(
      authHeader.replace('Bearer ', '')
    )

    if (authError || !user) {
      return new Response(JSON.stringify({ error: 'Unauthorized' }), {
        status: 401,
        headers: { ...corsHeaders, 'Content-Type': 'application/json' },
      })
    }

    const { companyId, priceId, successUrl, cancelUrl } = await req.json()

    // Verify user is admin of company
    const { data: membership } = await supabase
      .from('company_members')
      .select('role')
      .eq('company_id', companyId)
      .eq('user_id', user.id)
      .single()

    if (membership?.role !== 'admin') {
      return new Response(JSON.stringify({ error: 'Only admins can manage billing' }), {
        status: 403,
        headers: { ...corsHeaders, 'Content-Type': 'application/json' },
      })
    }

    // Get or create Stripe customer
    let { data: subscription } = await supabase
      .from('subscriptions')
      .select('stripe_customer_id')
      .eq('company_id', companyId)
      .single()

    let customerId = subscription?.stripe_customer_id

    if (!customerId) {
      // Get company info
      const { data: company } = await supabase
        .from('companies')
        .select('name')
        .eq('id', companyId)
        .single()

      // Create Stripe customer
      const customer = await stripe.customers.create({
        email: user.email,
        name: company?.name,
        metadata: {
          company_id: companyId,
          user_id: user.id,
        },
      })

      customerId = customer.id

      // Save customer ID
      await supabase.from('subscriptions').upsert({
        company_id: companyId,
        stripe_customer_id: customerId,
      })
    }

    // Create checkout session
    const session = await stripe.checkout.sessions.create({
      customer: customerId,
      mode: 'subscription',
      line_items: [{ price: priceId, quantity: 1 }],
      success_url: successUrl,
      cancel_url: cancelUrl,
      metadata: {
        company_id: companyId,
      },
    })

    return new Response(JSON.stringify({ sessionId: session.id, url: session.url }), {
      headers: { ...corsHeaders, 'Content-Type': 'application/json' },
    })
  } catch (error) {
    console.error('Error:', error)
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { ...corsHeaders, 'Content-Type': 'application/json' },
    })
  }
})
```

#### 4. Edge Function - Stripe Webhook

```typescript
// supabase/functions/stripe-webhook/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'
import Stripe from 'https://esm.sh/stripe@13?target=deno'

const stripe = new Stripe(Deno.env.get('STRIPE_SECRET_KEY')!, {
  apiVersion: '2023-10-16',
})

const webhookSecret = Deno.env.get('STRIPE_WEBHOOK_SECRET')!

serve(async (req) => {
  const signature = req.headers.get('stripe-signature')!
  const body = await req.text()

  let event: Stripe.Event

  try {
    event = stripe.webhooks.constructEvent(body, signature, webhookSecret)
  } catch (err) {
    console.error('Webhook signature verification failed:', err)
    return new Response('Webhook Error', { status: 400 })
  }

  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  )

  try {
    switch (event.type) {
      case 'checkout.session.completed': {
        const session = event.data.object as Stripe.Checkout.Session
        const companyId = session.metadata?.company_id

        if (companyId && session.subscription) {
          // Get subscription details
          const subscription = await stripe.subscriptions.retrieve(
            session.subscription as string
          )

          const plan = getPlanFromPriceId(subscription.items.data[0].price.id)

          // Update database
          await supabase.from('subscriptions').upsert({
            company_id: companyId,
            stripe_customer_id: session.customer as string,
            stripe_subscription_id: subscription.id,
            plan,
            status: subscription.status === 'active' ? 'active' : 'trialing',
            current_period_start: new Date(subscription.current_period_start * 1000).toISOString(),
            current_period_end: new Date(subscription.current_period_end * 1000).toISOString(),
          })

          // Update company subscription_tier
          await supabase
            .from('companies')
            .update({ subscription_tier: plan })
            .eq('id', companyId)
        }
        break
      }

      case 'customer.subscription.updated': {
        const subscription = event.data.object as Stripe.Subscription
        
        const { data: sub } = await supabase
          .from('subscriptions')
          .select('company_id')
          .eq('stripe_subscription_id', subscription.id)
          .single()

        if (sub) {
          const plan = getPlanFromPriceId(subscription.items.data[0].price.id)

          await supabase
            .from('subscriptions')
            .update({
              plan,
              status: mapStripeStatus(subscription.status),
              current_period_start: new Date(subscription.current_period_start * 1000).toISOString(),
              current_period_end: new Date(subscription.current_period_end * 1000).toISOString(),
              cancel_at_period_end: subscription.cancel_at_period_end,
            })
            .eq('stripe_subscription_id', subscription.id)

          await supabase
            .from('companies')
            .update({ subscription_tier: plan })
            .eq('id', sub.company_id)
        }
        break
      }

      case 'customer.subscription.deleted': {
        const subscription = event.data.object as Stripe.Subscription

        const { data: sub } = await supabase
          .from('subscriptions')
          .select('company_id')
          .eq('stripe_subscription_id', subscription.id)
          .single()

        if (sub) {
          await supabase
            .from('subscriptions')
            .update({ status: 'cancelled', plan: 'free' })
            .eq('stripe_subscription_id', subscription.id)

          await supabase
            .from('companies')
            .update({ subscription_tier: 'free' })
            .eq('id', sub.company_id)
        }
        break
      }

      case 'invoice.payment_failed': {
        const invoice = event.data.object as Stripe.Invoice
        
        if (invoice.subscription) {
          await supabase
            .from('subscriptions')
            .update({ status: 'past_due' })
            .eq('stripe_subscription_id', invoice.subscription)
        }
        break
      }
    }

    return new Response(JSON.stringify({ received: true }), { status: 200 })
  } catch (error) {
    console.error('Error processing webhook:', error)
    return new Response('Webhook Error', { status: 500 })
  }
})

function getPlanFromPriceId(priceId: string): string {
  const priceMap: Record<string, string> = {
    [Deno.env.get('STRIPE_STARTER_PRICE_ID')!]: 'starter',
    [Deno.env.get('STRIPE_PRO_PRICE_ID')!]: 'pro',
  }
  return priceMap[priceId] || 'free'
}

function mapStripeStatus(status: Stripe.Subscription.Status): string {
  const statusMap: Record<string, string> = {
    active: 'active',
    past_due: 'past_due',
    canceled: 'cancelled',
    trialing: 'trialing',
  }
  return statusMap[status] || 'active'
}
```

#### 5. Billing Page

```typescript
// src/pages/dashboard/BillingPage.tsx
import { useQuery, useMutation } from '@tanstack/react-query'
import { useCompany } from '@/contexts/CompanyContext'
import { supabase } from '@/lib/supabase'
import { PLANS } from '@/lib/stripe'
import { Button } from '@/components/ui/button'
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { Progress } from '@/components/ui/progress'
import { Check, AlertCircle } from 'lucide-react'
import { toast } from 'sonner'
import { cn } from '@/lib/utils'

interface Subscription {
  plan: string
  status: string
  current_period_end: string
  cancel_at_period_end: boolean
}

interface Usage {
  users: { current: number; max: number }
  storage: { current: number; max: number }
  departments: { current: number; max: number }
}

export function BillingPage() {
  const { currentCompany } = useCompany()

  const { data: subscription } = useQuery({
    queryKey: ['subscription', currentCompany?.id],
    queryFn: async () => {
      const { data, error } = await supabase
        .from('subscriptions')
        .select('*')
        .eq('company_id', currentCompany!.id)
        .single()
      
      if (error && error.code !== 'PGRST116') throw error
      return data as Subscription | null
    },
    enabled: !!currentCompany,
  })

  const { data: usage } = useQuery({
    queryKey: ['usage', currentCompany?.id],
    queryFn: async () => {
      // Get current usage
      const [users, departments, storage] = await Promise.all([
        supabase.from('company_members').select('id', { count: 'exact' }).eq('company_id', currentCompany!.id).eq('is_active', true),
        supabase.from('departments').select('id', { count: 'exact' }).eq('company_id', currentCompany!.id),
        supabase.from('file_uploads').select('size_bytes').eq('company_id', currentCompany!.id),
      ])

      // Get limits
      const { data: limits } = await supabase.rpc('get_company_limits', { p_company_id: currentCompany!.id })
      const limit = limits?.[0]

      const totalStorage = storage.data?.reduce((acc, f) => acc + (f.size_bytes || 0), 0) || 0

      return {
        users: { current: users.count || 0, max: limit?.max_users || 5 },
        storage: { current: Math.round(totalStorage / (1024 * 1024 * 1024) * 100) / 100, max: limit?.max_storage_gb || 1 },
        departments: { current: departments.count || 0, max: limit?.max_departments || 2 },
      } as Usage
    },
    enabled: !!currentCompany,
  })

  const checkoutMutation = useMutation({
    mutationFn: async (priceId: string) => {
      const { data: { session } } = await supabase.auth.getSession()
      
      const response = await fetch(`${import.meta.env.VITE_SUPABASE_URL}/functions/v1/create-checkout`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${session?.access_token}`,
        },
        body: JSON.stringify({
          companyId: currentCompany!.id,
          priceId,
          successUrl: `${window.location.origin}/dashboard/billing?success=true`,
          cancelUrl: `${window.location.origin}/dashboard/billing?cancelled=true`,
        }),
      })

      const data = await response.json()
      if (data.error) throw new Error(data.error)
      
      // Redirect to Stripe Checkout
      window.location.href = data.url
    },
    onError: (error: Error) => {
      toast.error(error.message)
    },
  })

  const currentPlan = subscription?.plan || 'free'

  return (
    <div className="space-y-6">
      <div>
        <h2 className="text-2xl font-bold">Billing</h2>
        <p className="text-muted-foreground">
          Manage your subscription and billing
        </p>
      </div>

      {/* Current Plan */}
      <Card>
        <CardHeader>
          <CardTitle>Current Plan</CardTitle>
          <CardDescription>
            You are currently on the <strong className="capitalize">{currentPlan}</strong> plan
          </CardDescription>
        </CardHeader>
        <CardContent className="space-y-4">
          {subscription?.status === 'past_due' && (
            <div className="flex items-center gap-2 p-3 bg-destructive/10 text-destructive rounded-lg">
              <AlertCircle className="h-5 w-5" />
              <span>Your payment is past due. Please update your payment method.</span>
            </div>
          )}

          {subscription?.cancel_at_period_end && (
            <div className="flex items-center gap-2 p-3 bg-yellow-100 text-yellow-800 rounded-lg">
              <AlertCircle className="h-5 w-5" />
              <span>
                Your subscription will be cancelled at the end of the billing period (
                {new Date(subscription.current_period_end).toLocaleDateString()})
              </span>
            </div>
          )}

          {/* Usage */}
          {usage && (
            <div className="space-y-4">
              <h4 className="font-medium">Usage</h4>
              
              <div className="space-y-3">
                <div>
                  <div className="flex justify-between text-sm mb-1">
                    <span>Users</span>
                    <span>{usage.users.current} / {usage.users.max === -1 ? '∞' : usage.users.max}</span>
                  </div>
                  <Progress value={usage.users.max === -1 ? 0 : (usage.users.current / usage.users.max) * 100} />
                </div>

                <div>
                  <div className="flex justify-between text-sm mb-1">
                    <span>Storage</span>
                    <span>{usage.storage.current} GB / {usage.storage.max === -1 ? '∞' : `${usage.storage.max} GB`}</span>
                  </div>
                  <Progress value={usage.storage.max === -1 ? 0 : (usage.storage.current / usage.storage.max) * 100} />
                </div>

                <div>
                  <div className="flex justify-between text-sm mb-1">
                    <span>Departments</span>
                    <span>{usage.departments.current} / {usage.departments.max === -1 ? '∞' : usage.departments.max}</span>
                  </div>
                  <Progress value={usage.departments.max === -1 ? 0 : (usage.departments.current / usage.departments.max) * 100} />
                </div>
              </div>
            </div>
          )}
        </CardContent>
      </Card>

      {/* Plans */}
      <div>
        <h3 className="text-lg font-semibold mb-4">Available Plans</h3>
        <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
          {Object.entries(PLANS).map(([key, plan]) => (
            <Card key={key} className={cn(currentPlan === key && 'border-primary')}>
              <CardHeader>
                <CardTitle className="flex items-center justify-between">
                  {plan.name}
                  {currentPlan === key && <Badge>Current</Badge>}
                </CardTitle>
                <CardDescription>
                  {plan.price === null ? (
                    'Contact sales'
                  ) : plan.price === 0 ? (
                    'Free forever'
                  ) : (
                    <span>
                      <span className="text-2xl font-bold text-foreground">${plan.price}</span>
                      /month
                    </span>
                  )}
                </CardDescription>
              </CardHeader>
              <CardContent>
                <ul className="space-y-2">
                  {plan.features.map((feature) => (
                    <li key={feature} className="flex items-center gap-2 text-sm">
                      <Check className="h-4 w-4 text-green-500" />
                      {feature}
                    </li>
                  ))}
                </ul>
              </CardContent>
              <CardFooter>
                {key === 'enterprise' ? (
                  <Button variant="outline" className="w-full">
                    Contact Sales
                  </Button>
                ) : currentPlan === key ? (
                  <Button variant="outline" className="w-full" disabled>
                    Current Plan
                  </Button>
                ) : plan.priceId ? (
                  <Button
                    className="w-full"
                    onClick={() => checkoutMutation.mutate(plan.priceId!)}
                    disabled={checkoutMutation.isPending}
                  >
                    {checkoutMutation.isPending ? 'Loading...' : 'Upgrade'}
                  </Button>
                ) : (
                  <Button variant="outline" className="w-full" disabled>
                    Free Plan
                  </Button>
                )}
              </CardFooter>
            </Card>
          ))}
        </div>
      </div>
    </div>
  )
}
```

#### 6. File Upload Component

```typescript
// src/components/FileUpload.tsx
import { useState, useCallback } from 'react'
import { useDropzone } from 'react-dropzone'
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { supabase } from '@/lib/supabase'
import { useCompany } from '@/contexts/CompanyContext'
import { Button } from '@/components/ui/button'
import { Progress } from '@/components/ui/progress'
import { toast } from 'sonner'
import { Upload, X, File } from 'lucide-react'
import { cn } from '@/lib/utils'

interface FileUploadProps {
  bucket: string
  entityType?: string
  entityId?: string
  maxSize?: number // in bytes
  accept?: Record<string, string[]>
  onUploadComplete?: (file: { path: string; url: string }) => void
}

export function FileUpload({
  bucket,
  entityType,
  entityId,
  maxSize = 10 * 1024 * 1024, // 10MB default
  accept = { 'image/*': [], 'application/pdf': [] },
  onUploadComplete,
}: FileUploadProps) {
  const { currentCompany } = useCompany()
  const queryClient = useQueryClient()
  const [progress, setProgress] = useState(0)

  const uploadMutation = useMutation({
    mutationFn: async (file: File) => {
      if (!currentCompany) throw new Error('No company selected')

      const fileExt = file.name.split('.').pop()
      const fileName = `${crypto.randomUUID()}.${fileExt}`
      const filePath = `${currentCompany.id}/${entityType || 'general'}/${fileName}`

      // Upload to Supabase Storage
      const { error: uploadError } = await supabase.storage
        .from(bucket)
        .upload(filePath, file, {
          cacheControl: '3600',
          upsert: false,
        })

      if (uploadError) throw uploadError

      // Get public URL
      const { data: { publicUrl } } = supabase.storage
        .from(bucket)
        .getPublicUrl(filePath)

      // Track in database
      const { data: { user } } = await supabase.auth.getUser()
      
      await supabase.from('file_uploads').insert({
        company_id: currentCompany.id,
        uploaded_by: user!.id,
        bucket,
        path: filePath,
        filename: file.name,
        content_type: file.type,
        size_bytes: file.size,
        entity_type: entityType,
        entity_id: entityId,
      })

      return { path: filePath, url: publicUrl }
    },
    onSuccess: (data) => {
      toast.success('File uploaded successfully')
      queryClient.invalidateQueries({ queryKey: ['files'] })
      onUploadComplete?.(data)
      setProgress(0)
    },
    onError: (error: Error) => {
      toast.error(error.message || 'Failed to upload file')
      setProgress(0)
    },
  })

  const onDrop = useCallback((acceptedFiles: File[]) => {
    const file = acceptedFiles[0]
    if (file) {
      uploadMutation.mutate(file)
    }
  }, [uploadMutation])

  const { getRootProps, getInputProps, isDragActive, fileRejections } = useDropzone({
    onDrop,
    accept,
    maxSize,
    multiple: false,
  })

  return (
    <div className="space-y-4">
      <div
        {...getRootProps()}
        className={cn(
          'border-2 border-dashed rounded-lg p-8 text-center cursor-pointer transition-colors',
          isDragActive ? 'border-primary bg-primary/5' : 'border-muted-foreground/25 hover:border-primary/50',
          uploadMutation.isPending && 'pointer-events-none opacity-50'
        )}
      >
        <input {...getInputProps()} />
        <Upload className="h-10 w-10 mx-auto mb-4 text-muted-foreground" />
        {isDragActive ? (
          <p>Drop the file here...</p>
        ) : (
          <div>
            <p className="font-medium">Drag & drop a file here, or click to select</p>
            <p className="text-sm text-muted-foreground mt-1">
              Max size: {Math.round(maxSize / (1024 * 1024))}MB
            </p>
          </div>
        )}
      </div>

      {fileRejections.length > 0 && (
        <p className="text-sm text-destructive">
          {fileRejections[0].errors[0].message}
        </p>
      )}

      {uploadMutation.isPending && (
        <div className="space-y-2">
          <Progress value={progress} />
          <p className="text-sm text-muted-foreground text-center">Uploading...</p>
        </div>
      )}
    </div>
  )
}
```

#### 7. Background Jobs (Edge Function)

```typescript
// supabase/functions/process-jobs/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  )

  // Get pending jobs
  const { data: jobs, error } = await supabase
    .from('background_jobs')
    .select('*')
    .eq('status', 'pending')
    .lte('scheduled_at', new Date().toISOString())
    .lt('attempts', supabase.sql`max_attempts`)
    .order('scheduled_at', { ascending: true })
    .limit(10)

  if (error) {
    console.error('Error fetching jobs:', error)
    return new Response(JSON.stringify({ error: error.message }), { status: 500 })
  }

  const results = []

  for (const job of jobs || []) {
    // Mark as processing
    await supabase
      .from('background_jobs')
      .update({ status: 'processing', started_at: new Date().toISOString(), attempts: job.attempts + 1 })
      .eq('id', job.id)

    try {
      let result: any

      switch (job.type) {
        case 'send_email':
          result = await handleSendEmail(job.payload)
          break
        case 'generate_report':
          result = await handleGenerateReport(job.payload, supabase)
          break
        case 'cleanup_expired_invitations':
          result = await handleCleanupInvitations(supabase)
          break
        default:
          throw new Error(`Unknown job type: ${job.type}`)
      }

      // Mark as completed
      await supabase
        .from('background_jobs')
        .update({
          status: 'completed',
          completed_at: new Date().toISOString(),
          result,
        })
        .eq('id', job.id)

      results.push({ id: job.id, status: 'completed' })
    } catch (error) {
      console.error(`Job ${job.id} failed:`, error)

      const status = job.attempts + 1 >= job.max_attempts ? 'failed' : 'pending'
      
      await supabase
        .from('background_jobs')
        .update({
          status,
          error: error.message,
          // Retry with exponential backoff
          scheduled_at: status === 'pending' 
            ? new Date(Date.now() + Math.pow(2, job.attempts) * 60000).toISOString()
            : undefined,
        })
        .eq('id', job.id)

      results.push({ id: job.id, status: 'failed', error: error.message })
    }
  }

  return new Response(JSON.stringify({ processed: results.length, results }), {
    headers: { 'Content-Type': 'application/json' },
  })
})

async function handleSendEmail(payload: { to: string; subject: string; body: string }) {
  // Integrate with Resend, SendGrid, etc.
  console.log('Would send email:', payload)
  return { sent: true }
}

async function handleGenerateReport(payload: { companyId: string; type: string }, supabase: any) {
  // Generate report logic
  return { generated: true }
}

async function handleCleanupInvitations(supabase: any) {
  const { data, error } = await supabase
    .from('invitations')
    .update({ status: 'expired' })
    .eq('status', 'pending')
    .lt('expires_at', new Date().toISOString())
    .select('id')

  return { expired: data?.length || 0 }
}
```

### Done When

- [ ] Subscriptions table created
- [ ] Plan limits enforced (users, storage, departments)
- [ ] Stripe checkout working
- [ ] Webhooks handling subscription events
- [ ] Billing page shows current plan & usage
- [ ] Can upgrade/downgrade plans
- [ ] File uploads to Supabase Storage
- [ ] File tracking in database
- [ ] Background jobs table created
- [ ] Job processor edge function working

### Commit

```bash
git add .
git commit -m "feat(billing): add Stripe billing and background jobs"
git checkout develop
git merge phase-4/billing
```

---

## Phase 5: Performance & Stability

**Branch:** `develop → phase-5/performance`

### Learn First (1-2 hours)

| Topic | Resource |
|-------|----------|
| TanStack Query Caching | https://tanstack.com/query/latest/docs/react/guides/caching |
| React.lazy | https://react.dev/reference/react/lazy |
| PostgreSQL Indexes | https://www.postgresql.org/docs/current/indexes.html |

### Do

#### 1. Database Indexes

```sql
-- ============================================
-- PHASE 5: PERFORMANCE INDEXES
-- ============================================

-- Company members - frequently queried
CREATE INDEX IF NOT EXISTS idx_company_members_company_user 
  ON company_members (company_id, user_id);

CREATE INDEX IF NOT EXISTS idx_company_members_active 
  ON company_members (company_id) 
  WHERE is_active = true;

-- Tasks - common queries
CREATE INDEX IF NOT EXISTS idx_tasks_company_status 
  ON tasks (company_id, status);

CREATE INDEX IF NOT EXISTS idx_tasks_assigned 
  ON tasks (assigned_to, status) 
  WHERE status != 'done';

CREATE INDEX IF NOT EXISTS idx_tasks_due_date 
  ON tasks (company_id, due_date) 
  WHERE status NOT IN ('done', 'cancelled');

-- Activities - timeline queries
CREATE INDEX IF NOT EXISTS idx_activities_company_time 
  ON activities (company_id, created_at DESC);

-- Audit logs
CREATE INDEX IF NOT EXISTS idx_audit_logs_user_time 
  ON audit_logs (user_id, created_at DESC);

CREATE INDEX IF NOT EXISTS idx_audit_logs_entity 
  ON audit_logs (entity_type, entity_id);

-- Invitations
CREATE INDEX IF NOT EXISTS idx_invitations_token 
  ON invitations (token) 
  WHERE status = 'pending';

CREATE INDEX IF NOT EXISTS idx_invitations_company_status 
  ON invitations (company_id, status);

-- Background jobs
CREATE INDEX IF NOT EXISTS idx_jobs_status_scheduled 
  ON background_jobs (status, scheduled_at) 
  WHERE status = 'pending';

-- File uploads
CREATE INDEX IF NOT EXISTS idx_files_company_entity 
  ON file_uploads (company_id, entity_type, entity_id);
```

#### 2. Query Optimization - Use Pagination

```typescript
// src/hooks/usePaginatedQuery.ts
import { useQuery, UseQueryOptions } from '@tanstack/react-query'
import { useState } from 'react'

interface PaginationParams {
  page: number
  pageSize: number
}

interface PaginatedResult<T> {
  data: T[]
  total: number
  page: number
  pageSize: number
  totalPages: number
}

export function usePaginatedQuery<T>(
  queryKey: string[],
  fetcher: (params: PaginationParams) => Promise<PaginatedResult<T>>,
  options?: Omit<UseQueryOptions<PaginatedResult<T>>, 'queryKey' | 'queryFn'>
) {
  const [page, setPage] = useState(1)
  const [pageSize, setPageSize] = useState(20)

  const query = useQuery({
    queryKey: [...queryKey, page, pageSize],
    queryFn: () => fetcher({ page, pageSize }),
    ...options,
  })

  return {
    ...query,
    page,
    pageSize,
    setPage,
    setPageSize,
    nextPage: () => setPage((p) => Math.min(p + 1, query.data?.totalPages || p)),
    prevPage: () => setPage((p) => Math.max(p - 1, 1)),
  }
}

// Example usage
export function useCompanyMembers(companyId: string) {
  return usePaginatedQuery(
    ['company-members', companyId],
    async ({ page, pageSize }) => {
      const from = (page - 1) * pageSize
      const to = from + pageSize - 1

      const { data, error, count } = await supabase
        .from('company_members')
        .select('*, profiles!user_id(*), departments(*)', { count: 'exact' })
        .eq('company_id', companyId)
        .range(from, to)
        .order('joined_at', { ascending: false })

      if (error) throw error

      return {
        data: data || [],
        total: count || 0,
        page,
        pageSize,
        totalPages: Math.ceil((count || 0) / pageSize),
      }
    }
  )
}
```

#### 3. Lazy Loading Routes

```typescript
// src/App.tsx
import { lazy, Suspense } from 'react'

// Lazy load heavy pages
const AnalyticsPage = lazy(() => import('@/pages/dashboard/AnalyticsPage'))
const BillingPage = lazy(() => import('@/pages/dashboard/BillingPage'))
const SettingsPage = lazy(() => import('@/pages/dashboard/SettingsPage'))

// Loading fallback
function PageLoader() {
  return (
    <div className="flex items-center justify-center h-64">
      <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-primary" />
    </div>
  )
}

// In routes:
<Route path="analytics" element={
  <Suspense fallback={<PageLoader />}>
    <AnalyticsPage />
  </Suspense>
} />
```

#### 4. Optimistic Updates

```typescript
// src/hooks/useTasks.ts
import { useMutation, useQueryClient } from '@tanstack/react-query'

export function useUpdateTaskStatus() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async ({ taskId, status }: { taskId: string; status: string }) => {
      const { error } = await supabase
        .from('tasks')
        .update({ status })
        .eq('id', taskId)

      if (error) throw error
    },
    // Optimistic update
    onMutate: async ({ taskId, status }) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['tasks'] })

      // Snapshot previous value
      const previousTasks = queryClient.getQueryData(['tasks'])

      // Optimistically update
      queryClient.setQueryData(['tasks'], (old: any) => {
        if (!old) return old
        return old.map((task: any) =>
          task.id === taskId ? { ...task, status } : task
        )
      })

      return { previousTasks }
    },
    onError: (err, variables, context) => {
      // Rollback on error
      queryClient.setQueryData(['tasks'], context?.previousTasks)
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['tasks'] })
    },
  })
}
```

#### 5. Query Client Configuration

```typescript
// src/lib/queryClient.ts
import { QueryClient } from '@tanstack/react-query'

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // Data is fresh for 1 minute
      staleTime: 60 * 1000,
      // Cache for 5 minutes
      gcTime: 5 * 60 * 1000,
      // Retry failed requests twice
      retry: 2,
      // Don't refetch on window focus in development
      refetchOnWindowFocus: import.meta.env.PROD,
    },
    mutations: {
      retry: 1,
    },
  },
})
```

### Done When

- [ ] Database indexes created
- [ ] Pagination implemented for lists
- [ ] Heavy routes lazy loaded
- [ ] Optimistic updates for common actions
- [ ] Query caching configured
- [ ] Bundle size reduced (check with `npm run build`)

### Commit

```bash
git add .
git commit -m "perf: add indexes, pagination, and lazy loading"
git checkout develop
git merge phase-5/performance
```

---

## Phase 6: Security Hardening

**Branch:** `develop → phase-6/security`

### Learn First (2-3 hours)

| Topic | Resource |
|-------|----------|
| OWASP Top 10 | https://owasp.org/www-project-top-ten/ |
| Supabase Security | https://supabase.com/docs/guides/auth/row-level-security |
| Content Security Policy | https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP |

### Do

#### 1. Input Validation with Zod

```typescript
// src/lib/validations.ts
import { z } from 'zod'

// Reusable schemas
export const emailSchema = z.string().email().toLowerCase().trim()

export const passwordSchema = z
  .string()
  .min(8, 'Password must be at least 8 characters')
  .regex(/[A-Z]/, 'Must contain uppercase')
  .regex(/[a-z]/, 'Must contain lowercase')
  .regex(/[0-9]/, 'Must contain number')
  .regex(/[^A-Za-z0-9]/, 'Must contain special character')

export const slugSchema = z
  .string()
  .min(3)
  .max(50)
  .regex(/^[a-z0-9-]+$/, 'Only lowercase letters, numbers, and hyphens')

export const uuidSchema = z.string().uuid()

// Entity schemas
export const createCompanySchema = z.object({
  name: z.string().min(2).max(100).trim(),
  slug: slugSchema,
})

export const createTaskSchema = z.object({
  title: z.string().min(1).max(200).trim(),
  description: z.string().max(5000).optional(),
  status: z.enum(['todo', 'in_progress', 'review', 'done']).default('todo'),
  priority: z.enum(['low', 'medium', 'high', 'urgent']).default('medium'),
  due_date: z.string().datetime().optional().nullable(),
  assigned_to: uuidSchema.optional().nullable(),
  team_id: uuidSchema.optional().nullable(),
})

export const inviteUserSchema = z.object({
  email: emailSchema,
  role: z.enum(['admin', 'manager', 'user']),
  department_id: uuidSchema.optional(),
})

// Sanitization
export function sanitizeHtml(input: string): string {
  return input
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;')
}
```

#### 2. Rate Limiting (Edge Function Middleware)

```typescript
// supabase/functions/_shared/rate-limit.ts
const rateLimitMap = new Map<string, { count: number; resetTime: number }>()

export function checkRateLimit(
  identifier: string,
  maxRequests: number = 100,
  windowMs: number = 60000
): { allowed: boolean; remaining: number; resetIn: number } {
  const now = Date.now()
  const record = rateLimitMap.get(identifier)

  if (!record || now > record.resetTime) {
    rateLimitMap.set(identifier, { count: 1, resetTime: now + windowMs })
    return { allowed: true, remaining: maxRequests - 1, resetIn: windowMs }
  }

  if (record.count >= maxRequests) {
    return { allowed: false, remaining: 0, resetIn: record.resetTime - now }
  }

  record.count++
  return { allowed: true, remaining: maxRequests - record.count, resetIn: record.resetTime - now }
}

// Usage in Edge Function
export function withRateLimit(handler: (req: Request) => Promise<Response>) {
  return async (req: Request): Promise<Response> => {
    const ip = req.headers.get('x-forwarded-for') || 'unknown'
    const { allowed, remaining, resetIn } = checkRateLimit(ip, 100, 60000)

    if (!allowed) {
      return new Response(JSON.stringify({ error: 'Too many requests' }), {
        status: 429,
        headers: {
          'Content-Type': 'application/json',
          'Retry-After': Math.ceil(resetIn / 1000).toString(),
          'X-RateLimit-Remaining': '0',
        },
      })
    }

    const response = await handler(req)
    response.headers.set('X-RateLimit-Remaining', remaining.toString())
    return response
  }
}
```

#### 3. Security Headers

```typescript
// src/lib/security.ts

// CSP Header for production
export const CSP_HEADER = [
  "default-src 'self'",
  "script-src 'self' 'unsafe-inline' https://js.stripe.com",
  "style-src 'self' 'unsafe-inline'",
  "img-src 'self' data: https: blob:",
  "font-src 'self'",
  "connect-src 'self' https://*.supabase.co wss://*.supabase.co https://api.stripe.com",
  "frame-src https://js.stripe.com https://hooks.stripe.com",
  "object-src 'none'",
  "base-uri 'self'",
].join('; ')

// Add to vite.config.ts for dev server
export default defineConfig({
  server: {
    headers: {
      'X-Content-Type-Options': 'nosniff',
      'X-Frame-Options': 'DENY',
      'X-XSS-Protection': '1; mode=block',
      'Referrer-Policy': 'strict-origin-when-cross-origin',
    },
  },
})
```

#### 4. Webhook Signature Verification

```typescript
// supabase/functions/_shared/verify-webhook.ts
import { hmac } from 'https://deno.land/x/hmac@v2.0.1/mod.ts'

export async function verifyStripeWebhook(
  payload: string,
  signature: string,
  secret: string
): Promise<boolean> {
  const elements = signature.split(',')
  const timestamp = elements.find((e) => e.startsWith('t='))?.split('=')[1]
  const v1Signature = elements.find((e) => e.startsWith('v1='))?.split('=')[1]

  if (!timestamp || !v1Signature) return false

  // Check timestamp is within 5 minutes
  const age = Date.now() / 1000 - parseInt(timestamp)
  if (age > 300) return false

  // Verify signature
  const signedPayload = `${timestamp}.${payload}`
  const expectedSignature = hmac('sha256', secret, signedPayload, 'utf8', 'hex')

  return v1Signature === expectedSignature
}
```

#### 5. Audit Log Everything

```typescript
// src/lib/audit.ts
import { supabase } from './supabase'

type AuditAction =
  | 'user_login'
  | 'user_logout'
  | 'user_created'
  | 'user_updated'
  | 'user_deleted'
  | 'role_changed'
  | 'invitation_sent'
  | 'invitation_accepted'
  | 'company_created'
  | 'company_updated'
  | 'subscription_changed'
  | 'task_created'
  | 'task_updated'
  | 'task_deleted'
  | 'file_uploaded'
  | 'file_deleted'
  | 'settings_changed'

export async function logAudit(
  action: AuditAction,
  entityType: string,
  entityId?: string,
  oldData?: Record<string, any>,
  newData?: Record<string, any>
) {
  try {
    await supabase.rpc('log_audit', {
      p_action: action,
      p_entity_type: entityType,
      p_entity_id: entityId,
      p_old_data: oldData,
      p_new_data: newData,
    })
  } catch (error) {
    console.error('Failed to log audit:', error)
  }
}

// Wrapper for mutations
export function withAudit<T extends (...args: any[]) => Promise<any>>(
  fn: T,
  getAuditParams: (...args: Parameters<T>) => {
    action: AuditAction
    entityType: string
    entityId?: string
    getData?: () => Promise<Record<string, any>>
  }
): T {
  return (async (...args: Parameters<T>) => {
    const params = getAuditParams(...args)
    const oldData = params.getData ? await params.getData() : undefined

    const result = await fn(...args)

    await logAudit(params.action, params.entityType, params.entityId, oldData, result)

    return result
  }) as T
}
```

### Done When

- [ ] All inputs validated with Zod
- [ ] Rate limiting on Edge Functions
- [ ] Security headers configured
- [ ] Webhook signatures verified
- [ ] Audit logs comprehensive
- [ ] No sensitive data in console.log
- [ ] Environment variables not exposed

### Commit

```bash
git add .
git commit -m "security: add validation, rate limiting, and audit logging"
git checkout develop
git merge phase-6/security
```

---

## Phase 7: Observability

**Branch:** `develop → phase-7/observability`

### Learn First (1-2 hours)

| Topic | Resource |
|-------|----------|
| Error Tracking | https://docs.sentry.io/platforms/javascript/guides/react/ |
| Logging Best Practices | https://www.loggly.com/ultimate-guide/node-logging-basics/ |

### Do

#### 1. Error Boundary

```typescript
// src/components/ErrorBoundary.tsx
import { Component, ReactNode } from 'react'
import { Button } from '@/components/ui/button'
import { AlertTriangle } from 'lucide-react'

interface Props {
  children: ReactNode
  fallback?: ReactNode
}

interface State {
  hasError: boolean
  error?: Error
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props)
    this.state = { hasError: false }
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error }
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log to error tracking service
    console.error('Error caught by boundary:', error, errorInfo)
    
    // If using Sentry:
    // Sentry.captureException(error, { extra: errorInfo })
  }

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback
      }

      return (
        <div className="min-h-[400px] flex items-center justify-center">
          <div className="text-center">
            <AlertTriangle className="h-12 w-12 mx-auto text-destructive mb-4" />
            <h2 className="text-xl font-semibold mb-2">Something went wrong</h2>
            <p className="text-muted-foreground mb-4">
              {this.state.error?.message || 'An unexpected error occurred'}
            </p>
            <Button onClick={() => window.location.reload()}>
              Reload Page
            </Button>
          </div>
        </div>
      )
    }

    return this.props.children
  }
}
```

#### 2. Error Tracking Hook

```typescript
// src/hooks/useErrorTracking.ts
import { useEffect } from 'react'
import { useAuth } from '@/contexts/AuthContext'
import { useCompany } from '@/contexts/CompanyContext'

export function useErrorTracking() {
  const { user } = useAuth()
  const { currentCompany } = useCompany()

  useEffect(() => {
    // Set user context for error tracking
    if (user) {
      // Sentry.setUser({ id: user.id, email: user.email })
      console.log('[ErrorTracking] User set:', user.id)
    }
  }, [user])

  useEffect(() => {
    // Set company context
    if (currentCompany) {
      // Sentry.setTag('company_id', currentCompany.id)
      console.log('[ErrorTracking] Company set:', currentCompany.id)
    }
  }, [currentCompany])

  const captureError = (error: Error, context?: Record<string, any>) => {
    console.error('[Error]', error, context)
    // Sentry.captureException(error, { extra: context })
  }

  const captureMessage = (message: string, level: 'info' | 'warning' | 'error' = 'info') => {
    console.log(`[${level.toUpperCase()}]`, message)
    // Sentry.captureMessage(message, level)
  }

  return { captureError, captureMessage }
}
```

#### 3. Audit Logs Dashboard

```typescript
// src/pages/dashboard/AuditLogsPage.tsx
import { useState } from 'react'
import { useQuery } from '@tanstack/react-query'
import { useCompany } from '@/contexts/CompanyContext'
import { supabase } from '@/lib/supabase'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Input } from '@/components/ui/input'
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table'
import { Badge } from '@/components/ui/badge'
import { formatDistanceToNow } from 'date-fns'
import { Search } from 'lucide-react'

const ACTION_LABELS: Record<string, { label: string; color: string }> = {
  user_created: { label: 'User Created', color: 'bg-green-100 text-green-800' },
  user_updated: { label: 'User Updated', color: 'bg-blue-100 text-blue-800' },
  user_deleted: { label: 'User Deleted', color: 'bg-red-100 text-red-800' },
  role_changed: { label: 'Role Changed', color: 'bg-purple-100 text-purple-800' },
  invitation_sent: { label: 'Invitation Sent', color: 'bg-blue-100 text-blue-800' },
  invitation_accepted: { label: 'Invitation Accepted', color: 'bg-green-100 text-green-800' },
  task_created: { label: 'Task Created', color: 'bg-blue-100 text-blue-800' },
  task_updated: { label: 'Task Updated', color: 'bg-yellow-100 text-yellow-800' },
  settings_changed: { label: 'Settings Changed', color: 'bg-orange-100 text-orange-800' },
}

export function AuditLogsPage() {
  const { currentCompany } = useCompany()
  const [search, setSearch] = useState('')
  const [actionFilter, setActionFilter] = useState<string>('all')

  const { data: logs, isLoading } = useQuery({
    queryKey: ['audit-logs', currentCompany?.id, actionFilter],
    queryFn: async () => {
      let query = supabase
        .from('audit_logs')
        .select(`
          *,
          profiles:user_id (full_name, email)
        `)
        .order('created_at', { ascending: false })
        .limit(100)

      // Filter by company-related entities
      // This requires the audit_logs to have company context

      if (actionFilter !== 'all') {
        query = query.eq('action', actionFilter)
      }

      const { data, error } = await query
      if (error) throw error
      return data
    },
    enabled: !!currentCompany,
  })

  const filteredLogs = logs?.filter((log) => {
    if (!search) return true
    const searchLower = search.toLowerCase()
    return (
      log.action.toLowerCase().includes(searchLower) ||
      log.entity_type.toLowerCase().includes(searchLower) ||
      log.profiles?.email?.toLowerCase().includes(searchLower) ||
      log.profiles?.full_name?.toLowerCase().includes(searchLower)
    )
  })

  return (
    <div className="space-y-6">
      <div>
        <h2 className="text-2xl font-bold">Audit Logs</h2>
        <p className="text-muted-foreground">
          Track all actions and changes in your company
        </p>
      </div>

      {/* Filters */}
      <div className="flex gap-4">
        <div className="relative flex-1 max-w-sm">
          <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
          <Input
            placeholder="Search logs..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="pl-9"
          />
        </div>
        <Select value={actionFilter} onValueChange={setActionFilter}>
          <SelectTrigger className="w-[200px]">
            <SelectValue placeholder="Filter by action" />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="all">All Actions</SelectItem>
            {Object.entries(ACTION_LABELS).map(([key, { label }]) => (
              <SelectItem key={key} value={key}>{label}</SelectItem>
            ))}
          </SelectContent>
        </Select>
      </div>

      {/* Logs Table */}
      <Card>
        <CardContent className="p-0">
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead>Action</TableHead>
                <TableHead>User</TableHead>
                <TableHead>Entity</TableHead>
                <TableHead>Details</TableHead>
                <TableHead>Time</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              {isLoading ? (
                <TableRow>
                  <TableCell colSpan={5} className="text-center py-8">
                    Loading...
                  </TableCell>
                </TableRow>
              ) : filteredLogs?.length === 0 ? (
                <TableRow>
                  <TableCell colSpan={5} className="text-center py-8">
                    No audit logs found
                  </TableCell>
                </TableRow>
              ) : (
                filteredLogs?.map((log) => {
                  const actionInfo = ACTION_LABELS[log.action] || {
                    label: log.action,
                    color: 'bg-gray-100 text-gray-800',
                  }

                  return (
                    <TableRow key={log.id}>
                      <TableCell>
                        <Badge className={actionInfo.color}>{actionInfo.label}</Badge>
                      </TableCell>
                      <TableCell>
                        <div>
                          <p className="font-medium">{log.profiles?.full_name || 'System'}</p>
                          <p className="text-sm text-muted-foreground">{log.profiles?.email}</p>
                        </div>
                      </TableCell>
                      <TableCell>
                        <span className="capitalize">{log.entity_type}</span>
                        {log.entity_id && (
                          <p className="text-xs text-muted-foreground font-mono">
                            {log.entity_id.slice(0, 8)}...
                          </p>
                        )}
                      </TableCell>
                      <TableCell className="max-w-xs">
                        {log.new_data && (
                          <pre className="text-xs text-muted-foreground truncate">
                            {JSON.stringify(log.new_data, null, 0).slice(0, 50)}...
                          </pre>
                        )}
                      </TableCell>
                      <TableCell className="text-muted-foreground whitespace-nowrap">
                        {formatDistanceToNow(new Date(log.created_at), { addSuffix: true })}
                      </TableCell>
                    </TableRow>
                  )
                })
              )}
            </TableBody>
          </Table>
        </CardContent>
      </Card>
    </div>
  )
}
```

#### 4. Job Status Dashboard

```typescript
// src/pages/dashboard/JobsPage.tsx (Admin only)
import { useQuery } from '@tanstack/react-query'
import { supabase } from '@/lib/supabase'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table'
import { Badge } from '@/components/ui/badge'
import { formatDistanceToNow } from 'date-fns'

const STATUS_COLORS: Record<string, string> = {
  pending: 'bg-yellow-100 text-yellow-800',
  processing: 'bg-blue-100 text-blue-800',
  completed: 'bg-green-100 text-green-800',
  failed: 'bg-red-100 text-red-800',
}

export function JobsPage() {
  const { data: jobs, isLoading } = useQuery({
    queryKey: ['background-jobs'],
    queryFn: async () => {
      const { data, error } = await supabase
        .from('background_jobs')
        .select('*')
        .order('created_at', { ascending: false })
        .limit(100)

      if (error) throw error
      return data
    },
    refetchInterval: 5000, // Refresh every 5 seconds
  })

  const stats = {
    pending: jobs?.filter((j) => j.status === 'pending').length || 0,
    processing: jobs?.filter((j) => j.status === 'processing').length || 0,
    completed: jobs?.filter((j) => j.status === 'completed').length || 0,
    failed: jobs?.filter((j) => j.status === 'failed').length || 0,
  }

  return (
    <div className="space-y-6">
      <h2 className="text-2xl font-bold">Background Jobs</h2>

      {/* Stats */}
      <div className="grid gap-4 md:grid-cols-4">
        {Object.entries(stats).map(([status, count]) => (
          <Card key={status}>
            <CardHeader className="pb-2">
              <CardTitle className="text-sm font-medium capitalize">{status}</CardTitle>
            </CardHeader>
            <CardContent>
              <div className="text-2xl font-bold">{count}</div>
            </CardContent>
          </Card>
        ))}
      </div>

      {/* Jobs Table */}
      <Card>
        <CardContent className="p-0">
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead>Type</TableHead>
                <TableHead>Status</TableHead>
                <TableHead>Attempts</TableHead>
                <TableHead>Created</TableHead>
                <TableHead>Error</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              {jobs?.map((job) => (
                <TableRow key={job.id}>
                  <TableCell className="font-medium">{job.type}</TableCell>
                  <TableCell>
                    <Badge className={STATUS_COLORS[job.status]}>{job.status}</Badge>
                  </TableCell>
                  <TableCell>{job.attempts} / {job.max_attempts}</TableCell>
                  <TableCell className="text-muted-foreground">
                    {formatDistanceToNow(new Date(job.created_at), { addSuffix: true })}
                  </TableCell>
                  <TableCell className="max-w-xs truncate text-destructive">
                    {job.error}
                  </TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </CardContent>
      </Card>
    </div>
  )
}
```

### Done When

- [ ] Error boundary wraps app
- [ ] Error tracking captures context
- [ ] Audit logs dashboard working
- [ ] Job status dashboard working
- [ ] Failed jobs visible and retry-able
- [ ] Alerts for critical failures (optional)

### Commit

```bash
git add .
git commit -m "feat(observability): add error tracking and audit dashboard"
git checkout develop
git merge phase-7/observability
```

---

## Phase 8: Customization & UX Polish

**Branch:** `develop → phase-8/ux-polish`

### Learn First (2-3 hours)

| Topic | Resource |
|-------|----------|
| Framer Motion | https://www.framer.com/motion/animation/ |
| Accessibility | https://www.w3.org/WAI/ARIA/apg/ |
| Dark Mode | https://tailwindcss.com/docs/dark-mode |

### Do

#### 1. Theme Provider with Dark Mode

```typescript
// src/contexts/ThemeContext.tsx
import { createContext, useContext, useEffect, useState, ReactNode } from 'react'

type Theme = 'light' | 'dark' | 'system'

interface ThemeContextType {
  theme: Theme
  setTheme: (theme: Theme) => void
  resolvedTheme: 'light' | 'dark'
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined)

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<Theme>(() => {
    if (typeof window !== 'undefined') {
      return (localStorage.getItem('theme') as Theme) || 'system'
    }
    return 'system'
  })

  const [resolvedTheme, setResolvedTheme] = useState<'light' | 'dark'>('light')

  useEffect(() => {
    const root = window.document.documentElement

    const applyTheme = (newTheme: 'light' | 'dark') => {
      root.classList.remove('light', 'dark')
      root.classList.add(newTheme)
      setResolvedTheme(newTheme)
    }

    if (theme === 'system') {
      const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)')
      applyTheme(mediaQuery.matches ? 'dark' : 'light')

      const listener = (e: MediaQueryListEvent) => {
        applyTheme(e.matches ? 'dark' : 'light')
      }

      mediaQuery.addEventListener('change', listener)
      return () => mediaQuery.removeEventListener('change', listener)
    } else {
      applyTheme(theme)
    }
  }, [theme])

  const handleSetTheme = (newTheme: Theme) => {
    localStorage.setItem('theme', newTheme)
    setTheme(newTheme)
  }

  return (
    <ThemeContext.Provider value={{ theme, setTheme: handleSetTheme, resolvedTheme }}>
      {children}
    </ThemeContext.Provider>
  )
}

export function useTheme() {
  const context = useContext(ThemeContext)
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider')
  }
  return context
}
```

#### 2. Theme Switcher Component

```typescript
// src/components/ThemeSwitcher.tsx
import { Moon, Sun, Monitor } from 'lucide-react'
import { useTheme } from '@/contexts/ThemeContext'
import { Button } from '@/components/ui/button'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu'

export function ThemeSwitcher() {
  const { theme, setTheme, resolvedTheme } = useTheme()

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="ghost" size="icon">
          {resolvedTheme === 'dark' ? (
            <Moon className="h-5 w-5" />
          ) : (
            <Sun className="h-5 w-5" />
          )}
          <span className="sr-only">Toggle theme</span>
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end">
        <DropdownMenuItem onClick={() => setTheme('light')}>
          <Sun className="mr-2 h-4 w-4" />
          Light
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme('dark')}>
          <Moon className="mr-2 h-4 w-4" />
          Dark
        </DropdownMenuItem>
        <DropdownMenuItem onClick={() => setTheme('system')}>
          <Monitor className="mr-2 h-4 w-4" />
          System
        </DropdownMenuItem>
      </DropdownMenuContent>
    </DropdownMenu>
  )
}
```

#### 3. Page Transitions with Framer Motion

```typescript
// src/components/PageTransition.tsx
import { motion, AnimatePresence } from 'framer-motion'
import { useLocation } from 'react-router-dom'
import { ReactNode } from 'react'

const pageVariants = {
  initial: {
    opacity: 0,
    y: 10,
  },
  enter: {
    opacity: 1,
    y: 0,
    transition: {
      duration: 0.2,
      ease: 'easeOut',
    },
  },
  exit: {
    opacity: 0,
    y: -10,
    transition: {
      duration: 0.15,
      ease: 'easeIn',
    },
  },
}

export function PageTransition({ children }: { children: ReactNode }) {
  const location = useLocation()

  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={location.pathname}
        initial="initial"
        animate="enter"
        exit="exit"
        variants={pageVariants}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  )
}
```

#### 4. Loading Skeleton Components

```typescript
// src/components/ui/skeleton.tsx
import { cn } from '@/lib/utils'

interface SkeletonProps extends React.HTMLAttributes<HTMLDivElement> {}

export function Skeleton({ className, ...props }: SkeletonProps) {
  return (
    <div
      className={cn('animate-pulse rounded-md bg-muted', className)}
      {...props}
    />
  )
}

// Usage examples
export function TableSkeleton({ rows = 5 }: { rows?: number }) {
  return (
    <div className="space-y-3">
      {Array.from({ length: rows }).map((_, i) => (
        <div key={i} className="flex gap-4">
          <Skeleton className="h-10 w-10 rounded-full" />
          <div className="space-y-2 flex-1">
            <Skeleton className="h-4 w-[200px]" />
            <Skeleton className="h-3 w-[150px]" />
          </div>
          <Skeleton className="h-8 w-[100px]" />
        </div>
      ))}
    </div>
  )
}

export function CardSkeleton() {
  return (
    <div className="p-6 border rounded-lg space-y-4">
      <Skeleton className="h-5 w-[120px]" />
      <Skeleton className="h-8 w-[80px]" />
      <Skeleton className="h-3 w-[200px]" />
    </div>
  )
}
```

#### 5. Accessibility Improvements

```typescript
// src/components/ui/accessible-dialog.tsx
import { useEffect, useRef } from 'react'
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '@/components/ui/dialog'

interface AccessibleDialogProps {
  open: boolean
  onOpenChange: (open: boolean) => void
  title: string
  description?: string
  children: React.ReactNode
}

export function AccessibleDialog({
  open,
  onOpenChange,
  title,
  description,
  children,
}: AccessibleDialogProps) {
  const closeButtonRef = useRef<HTMLButtonElement>(null)

  // Focus trap and keyboard navigation
  useEffect(() => {
    if (open) {
      const handleKeyDown = (e: KeyboardEvent) => {
        if (e.key === 'Escape') {
          onOpenChange(false)
        }
      }

      document.addEventListener('keydown', handleKeyDown)
      return () => document.removeEventListener('keydown', handleKeyDown)
    }
  }, [open, onOpenChange])

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent
        aria-labelledby="dialog-title"
        aria-describedby={description ? 'dialog-description' : undefined}
      >
        <DialogHeader>
          <DialogTitle id="dialog-title">{title}</DialogTitle>
          {description && (
            <p id="dialog-description" className="text-sm text-muted-foreground">
              {description}
            </p>
          )}
        </DialogHeader>
        {children}
      </DialogContent>
    </Dialog>
  )
}
```

#### 6. Mobile Responsive Sidebar

```typescript
// src/components/layout/MobileSidebar.tsx
import { Sheet, SheetContent, SheetTrigger } from '@/components/ui/sheet'
import { Button } from '@/components/ui/button'
import { Menu } from 'lucide-react'
import { useState } from 'react'
import { Link, useLocation } from 'react-router-dom'
import { cn } from '@/lib/utils'

// Same navItems as before...

export function MobileSidebar({ navItems }: { navItems: NavItem[] }) {
  const [open, setOpen] = useState(false)
  const { pathname } = useLocation()

  return (
    <Sheet open={open} onOpenChange={setOpen}>
      <SheetTrigger asChild>
        <Button variant="ghost" size="icon" className="md:hidden">
          <Menu className="h-5 w-5" />
          <span className="sr-only">Open menu</span>
        </Button>
      </SheetTrigger>
      <SheetContent side="left" className="w-[280px] p-0">
        <div className="p-4 border-b">
          <h2 className="font-bold text-lg">NovaPulse</h2>
        </div>
        <nav className="p-4 space-y-1">
          {navItems.map((item) => {
            const Icon = item.icon
            const isActive = pathname === item.href

            return (
              <Link
                key={item.href}
                to={item.href}
                onClick={() => setOpen(false)}
                className={cn(
                  'flex items-center gap-3 px-3 py-2 rounded-md transition-colors',
                  isActive
                    ? 'bg-primary text-primary-foreground'
                    : 'text-muted-foreground hover:bg-muted'
                )}
              >
                <Icon className="h-5 w-5" />
                {item.label}
              </Link>
            )
          })}
        </nav>
      </SheetContent>
    </Sheet>
  )
}
```

### Done When

- [ ] Dark mode working
- [ ] Theme persists in localStorage
- [ ] Page transitions smooth
- [ ] Loading skeletons for all data
- [ ] Mobile responsive
- [ ] Keyboard navigation works
- [ ] Screen reader friendly (test with VoiceOver/NVDA)

### Commit

```bash
git add .
git commit -m "feat(ux): add dark mode, animations, and accessibility"
git checkout develop
git merge phase-8/ux-polish
```

---

## Phase 9: Testing & QA

**Branch:** `develop → phase-9/testing`

### Learn First (2-3 hours)

| Topic | Resource |
|-------|----------|
| Vitest | https://vitest.dev/guide/ |
| React Testing Library | https://testing-library.com/docs/react-testing-library/intro/ |
| Playwright E2E | https://playwright.dev/docs/intro |

### Do

#### 1. Setup Vitest

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
```

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./src/test/setup.ts'],
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})
```

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom'
import { vi } from 'vitest'

// Mock Supabase
vi.mock('@/lib/supabase', () => ({
  supabase: {
    auth: {
      getSession: vi.fn().mockResolvedValue({ data: { session: null } }),
      getUser: vi.fn().mockResolvedValue({ data: { user: null } }),
      onAuthStateChange: vi.fn().mockReturnValue({ data: { subscription: { unsubscribe: vi.fn() } } }),
    },
    from: vi.fn().mockReturnValue({
      select: vi.fn().mockReturnThis(),
      insert: vi.fn().mockReturnThis(),
      update: vi.fn().mockReturnThis(),
      delete: vi.fn().mockReturnThis(),
      eq: vi.fn().mockReturnThis(),
      single: vi.fn().mockResolvedValue({ data: null, error: null }),
    }),
  },
}))
```

#### 2. Unit Tests

```typescript
// src/lib/__tests__/validations.test.ts
import { describe, it, expect } from 'vitest'
import { emailSchema, passwordSchema, slugSchema } from '../validations'

describe('Validation Schemas', () => {
  describe('emailSchema', () => {
    it('accepts valid emails', () => {
      expect(emailSchema.safeParse('test@example.com').success).toBe(true)
      expect(emailSchema.safeParse('USER@DOMAIN.COM').data).toBe('user@domain.com')
    })

    it('rejects invalid emails', () => {
      expect(emailSchema.safeParse('invalid').success).toBe(false)
      expect(emailSchema.safeParse('no@domain').success).toBe(false)
    })
  })

  describe('passwordSchema', () => {
    it('accepts strong passwords', () => {
      expect(passwordSchema.safeParse('Password1!').success).toBe(true)
      expect(passwordSchema.safeParse('MySecure123@').success).toBe(true)
    })

    it('rejects weak passwords', () => {
      expect(passwordSchema.safeParse('short').success).toBe(false)
      expect(passwordSchema.safeParse('nouppercase1!').success).toBe(false)
      expect(passwordSchema.safeParse('NOLOWERCASE1!').success).toBe(false)
      expect(passwordSchema.safeParse('NoNumbers!').success).toBe(false)
      expect(passwordSchema.safeParse('NoSpecial123').success).toBe(false)
    })
  })

  describe('slugSchema', () => {
    it('accepts valid slugs', () => {
      expect(slugSchema.safeParse('my-company').success).toBe(true)
      expect(slugSchema.safeParse('company123').success).toBe(true)
    })

    it('rejects invalid slugs', () => {
      expect(slugSchema.safeParse('ab').success).toBe(false) // too short
      expect(slugSchema.safeParse('My Company').success).toBe(false) // spaces
      expect(slugSchema.safeParse('UPPERCASE').success).toBe(false)
    })
  })
})
```

#### 3. Component Tests

```typescript
// src/components/__tests__/RoleGuard.test.tsx
import { describe, it, expect, vi } from 'vitest'
import { render, screen } from '@testing-library/react'
import { RoleGuard } from '../RoleGuard'

// Mock the useCompany hook
vi.mock('@/contexts/CompanyContext', () => ({
  useCompany: vi.fn(),
}))

import { useCompany } from '@/contexts/CompanyContext'

describe('RoleGuard', () => {
  it('renders children when role matches', () => {
    vi.mocked(useCompany).mockReturnValue({
      currentRole: 'admin',
      companies: [],
      currentCompany: null,
      setCurrentCompany: vi.fn(),
      loading: false,
      refetch: vi.fn(),
    })

    render(
      <RoleGuard allowedRoles={['admin']}>
        <div>Admin Content</div>
      </RoleGuard>
    )

    expect(screen.getByText('Admin Content')).toBeInTheDocument()
  })

  it('renders fallback when role does not match', () => {
    vi.mocked(useCompany).mockReturnValue({
      currentRole: 'user',
      companies: [],
      currentCompany: null,
      setCurrentCompany: vi.fn(),
      loading: false,
      refetch: vi.fn(),
    })

    render(
      <RoleGuard allowedRoles={['admin']} fallback={<div>Access Denied</div>}>
        <div>Admin Content</div>
      </RoleGuard>
    )

    expect(screen.queryByText('Admin Content')).not.toBeInTheDocument()
    expect(screen.getByText('Access Denied')).toBeInTheDocument()
  })
})
```

#### 4. E2E Tests with Playwright

```bash
npm install -D @playwright/test
npx playwright install
```

```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Authentication', () => {
  test('should redirect unauthenticated users to login', async ({ page }) => {
    await page.goto('/dashboard')
    await expect(page).toHaveURL(/.*login/)
  })

  test('should show error for invalid credentials', async ({ page }) => {
    await page.goto('/login')
    await page.fill('input[type="email"]', 'invalid@test.com')
    await page.fill('input[type="password"]', 'wrongpassword')
    await page.click('button[type="submit"]')
    await expect(page.locator('.text-destructive')).toBeVisible()
  })

  test('should login with valid credentials', async ({ page }) => {
    await page.goto('/login')
    await page.fill('input[type="email"]', 'test@example.com')
    await page.fill('input[type="password"]', 'TestPassword123!')
    await page.click('button[type="submit"]')
    await expect(page).toHaveURL(/.*dashboard/)
  })
})
```

#### 5. Test Checklist

```markdown
## Manual Test Checklist

### Authentication
- [ ] Register with valid email/password
- [ ] Register with weak password (should fail)
- [ ] Login with correct credentials
- [ ] Login with wrong password (should fail, max 5 attempts)
- [ ] Logout
- [ ] Session persists on refresh
- [ ] Protected routes redirect when logged out

### Multi-Tenancy
- [ ] Create a company
- [ ] Switch between companies
- [ ] Data is isolated between companies
- [ ] User A cannot see User B's company

### RBAC
- [ ] Admin sees all menu items
- [ ] Manager sees limited menu items
- [ ] User sees minimal menu items
- [ ] Role changes take effect immediately

### Invitations
- [ ] Send invitation
- [ ] Copy invite link
- [ ] Accept invitation (correct email)
- [ ] Reject invitation (wrong email)
- [ ] Expired invitation fails

### Tasks
- [ ] Create task
- [ ] Assign task to user
- [ ] Change task status
- [ ] Filter/search tasks
- [ ] Task shows in assigned user's dashboard

### Billing
- [ ] View current plan
- [ ] Start Stripe checkout
- [ ] Webhook updates subscription
- [ ] Plan limits enforced

### File Uploads
- [ ] Upload file
- [ ] File appears in storage
- [ ] File tracked in database
- [ ] Download file

### Cross-Browser
- [ ] Chrome
- [ ] Firefox
- [ ] Safari
- [ ] Edge
- [ ] Mobile Chrome
- [ ] Mobile Safari
```

### Done When

- [ ] Unit tests passing
- [ ] Component tests passing
- [ ] E2E tests passing
- [ ] Manual test checklist complete
- [ ] No console errors
- [ ] Lighthouse score > 80

### Commit

```bash
git add .
git commit -m "test: add unit, component, and e2e tests"
git checkout develop
git merge phase-9/testing
```

---

## Phase 10: Deployment & Release

**Branch:** `develop → phase-10/deployment`

### Learn First (1-2 hours)

| Topic | Resource |
|-------|----------|
| Vercel Deployment | https://vercel.com/docs |
| Supabase Production | https://supabase.com/docs/guides/platform |
| Environment Variables | https://vitejs.dev/guide/env-and-mode.html |

### Do

#### 1. Environment Configuration

```bash
# .env.local (Development)
VITE_SUPABASE_URL=https://xxx.supabase.co
VITE_SUPABASE_ANON_KEY=eyJ...
VITE_STRIPE_PUBLISHABLE_KEY=pk_test_...
VITE_STRIPE_STARTER_PRICE_ID=price_...
VITE_STRIPE_PRO_PRICE_ID=price_...

# .env.production (Production - set in Vercel)
VITE_SUPABASE_URL=https://yyy.supabase.co
VITE_SUPABASE_ANON_KEY=eyJ...
VITE_STRIPE_PUBLISHABLE_KEY=pk_live_...
VITE_STRIPE_STARTER_PRICE_ID=price_...
VITE_STRIPE_PRO_PRICE_ID=price_...
```

#### 2. Production Supabase Setup

```sql
-- Run in production Supabase SQL Editor

-- 1. Verify all RLS policies are enabled
SELECT tablename, rowsecurity 
FROM pg_tables 
WHERE schemaname = 'public';

-- 2. Create production indexes (if not exists)
-- (Run all indexes from Phase 5)

-- 3. Set up database backups
-- Done via Supabase Dashboard > Database > Backups

-- 4. Enable Point-in-Time Recovery (Pro plan)
-- Done via Supabase Dashboard

-- 5. Review and restrict service role key access
```

#### 3. Vercel Deployment

```bash
# Install Vercel CLI
npm install -g vercel

# Login
vercel login

# Deploy preview
vercel

# Deploy production
vercel --prod
```

```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "framework": "vite",
  "rewrites": [
    { "source": "/(.*)", "destination": "/" }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-XSS-Protection", "value": "1; mode=block" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" }
      ]
    }
  ]
}
```

#### 4. Deploy Edge Functions

```bash
# Install Supabase CLI
npm install -g supabase

# Login
supabase login

# Link to project
supabase link --project-ref your-project-ref

# Deploy all functions
supabase functions deploy

# Set secrets
supabase secrets set STRIPE_SECRET_KEY=sk_live_...
supabase secrets set STRIPE_WEBHOOK_SECRET=whsec_...
```

#### 5. Stripe Production Setup

```markdown
## Stripe Production Checklist

1. [ ] Create production Stripe account
2. [ ] Create products and prices
3. [ ] Update price IDs in environment
4. [ ] Set up webhook endpoint: `https://xxx.supabase.co/functions/v1/stripe-webhook`
5. [ ] Add webhook events:
   - checkout.session.completed
   - customer.subscription.updated
   - customer.subscription.deleted
   - invoice.payment_failed
6. [ ] Test with Stripe test card
7. [ ] Switch to live mode
```

#### 6. Domain & SSL

```markdown
## Custom Domain Setup

### Vercel (Frontend)
1. Add domain in Vercel Dashboard
2. Update DNS records
3. SSL auto-provisioned

### Supabase (API)
1. Go to Settings > Custom Domains
2. Add your domain
3. Update DNS CNAME record
4. Update frontend env vars
```

#### 7. Deployment Checklist

```markdown
## Pre-Deployment Checklist

### Code
- [ ] All tests passing
- [ ] No console.log statements
- [ ] No hardcoded secrets
- [ ] Build succeeds locally

### Supabase
- [ ] All migrations applied
- [ ] RLS enabled on all tables
- [ ] Indexes created
- [ ] Backups configured
- [ ] Edge functions deployed

### Stripe
- [ ] Products created
- [ ] Webhook endpoint configured
- [ ] Test payment works

### Vercel
- [ ] Environment variables set
- [ ] Custom domain configured
- [ ] SSL active

### Monitoring
- [ ] Error tracking configured
- [ ] Uptime monitoring set up
- [ ] Alerts configured

## Post-Deployment Checklist

- [ ] Verify login works
- [ ] Verify protected routes
- [ ] Test Stripe checkout
- [ ] Test file uploads
- [ ] Check audit logs
- [ ] Monitor error rates
- [ ] Check performance metrics
```

#### 8. Monitoring & Alerts

```typescript
// Set up basic uptime monitoring

// Using Supabase Edge Function for health check
// supabase/functions/health/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async () => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  )

  try {
    // Check database
    const { error: dbError } = await supabase.from('profiles').select('id').limit(1)
    if (dbError) throw new Error('Database unhealthy')

    // Check auth
    const { error: authError } = await supabase.auth.getSession()
    if (authError) throw new Error('Auth unhealthy')

    return new Response(JSON.stringify({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      services: {
        database: 'ok',
        auth: 'ok',
      }
    }), {
      headers: { 'Content-Type': 'application/json' },
    })
  } catch (error) {
    return new Response(JSON.stringify({
      status: 'unhealthy',
      error: error.message,
    }), {
      status: 500,
      headers: { 'Content-Type': 'application/json' },
    })
  }
})
```

### Done When

- [ ] Production Supabase configured
- [ ] Edge functions deployed
- [ ] Stripe webhooks working
- [ ] Vercel deployment successful
- [ ] Custom domain active
- [ ] SSL working
- [ ] Health check endpoint responding
- [ ] Monitoring configured

### Commit

```bash
git add .
git commit -m "chore: production deployment configuration"
git checkout develop
git merge phase-10/deployment
git checkout main
git merge develop
git tag v1.0.0
git push origin main --tags
```

---

## 🎉 Phase A Complete!

You've built a working SaaS Admin Platform:

| Component | What It Does |
|-----------|--------------|
| **Auth** | Register, login, sessions, password security |
| **Multi-Tenancy** | Company isolation, RLS, data security |
| **RBAC** | Admin, Manager, User roles with permissions |
| **Invitations** | Token-based team onboarding |
| **Dashboard** | Overview, stats, activity feed |
| **User Management** | CRUD, role assignment, deactivation |
| **Billing** | Stripe subscriptions, plan limits |
| **File Storage** | Secure uploads with tracking |
| **Background Jobs** | DB-backed async processing |
| **Audit Logs** | Complete activity tracking |
| **Dark Mode** | Theme switching with persistence |
| **Testing** | Unit, component, and E2E tests |
| **Deployment** | Vercel + Supabase production setup |

---

## Future Phases (11+)

These are **optional** and should only be started after Phase 10 is stable:

| Phase | What You Add | When to Add |
|-------|--------------|-------------|
| 11 | Redis Caching | High traffic |
| 12 | Read Replicas | Database bottleneck |
| 13 | Multi-Region | Global users |
| 14 | Microservices | Team scaling |
| 15 | SSO/SAML | Enterprise customers |
| 16 | API Rate Limiting | Public API |
| 17 | Webhooks System | Integrations |
| 18 | Mobile App | User demand |

---

## Summary

### One Document, Ten Phases

| Phase | What | Duration |
|-------|------|----------|
| 1 | Foundation & Auth | Days 1-3 |
| 2 | Multi-Tenancy & RBAC | Days 4-6 |
| 2.5 | Invitations | Days 7-8 |
| 3 | Core Product Features | Days 9-14 |
| 4 | Billing & Background | Days 15-19 |
| 5 | Performance | Days 20-22 |
| 6 | Security Hardening | Days 23-25 |
| 7 | Observability | Days 26-28 |
| 8 | Customization & UX | Days 29-32 |
| 9 | Testing & QA | Days 33-36 |
| 10 | Deployment & Release | Days 37-40 |

**Total: ~40 days / 6 weeks**

### Technologies You'll Learn

**Frontend:**
- React 18 + TypeScript
- Vite
- React Router v6
- Tailwind CSS + shadcn/ui
- TanStack Query v5
- React Hook Form + Zod
- Framer Motion
- Recharts

**Backend:**
- Supabase (PostgreSQL)
- Row Level Security
- Edge Functions (Deno)
- Supabase Auth
- Supabase Storage

**Tools:**
- Stripe (Billing)
- Vitest + Playwright (Testing)
- Vercel (Deployment)

### Start Now

```bash
mkdir novapulse && cd novapulse
npm create vite@latest web -- --template react-ts
# Follow Phase 1...
```

---

## Appendix: Quick Reference

### Database Tables

| Table | Purpose |
|-------|---------|
| `profiles` | User profiles (linked to auth.users) |
| `user_roles` | Role assignments (separate from profile!) |
| `companies` | Multi-tenant companies |
| `company_members` | User ↔ Company with role |
| `departments` | Organization units |
| `teams` | Working teams |
| `team_members` | Team membership |
| `tasks` | Work items |
| `activities` | Activity feed |
| `audit_logs` | Security audit trail |
| `invitations` | Team invites |
| `subscriptions` | Stripe subscriptions |
| `plan_limits` | Plan configuration |
| `background_jobs` | Async job queue |
| `file_uploads` | File tracking |

### Key Security Functions

| Function | Purpose |
|----------|---------|
| `get_user_role()` | Get user's role in company |
| `is_company_member()` | Check company membership |
| `is_company_admin()` | Check admin role |
| `is_company_admin_or_manager()` | Check admin or manager |
| `check_company_limit()` | Enforce plan limits |
| `log_audit()` | Record audit event |
| `accept_invitation()` | Process invite acceptance |

### Common Commands

```bash
# Development
npm run dev

# Build
npm run build

# Test
npm run test
npm run test:e2e

# Deploy
vercel --prod
supabase functions deploy
```

---

**One document. Top to bottom. That's it.**


