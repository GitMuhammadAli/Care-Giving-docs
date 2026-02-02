# 🔄 Server State vs Client State - Complete Guide

> A comprehensive guide to managing server state with React Query, SWR, and understanding the distinction between server and client state in modern React applications.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Server state is data that lives on the server and is cached/synchronized on the client (fetched via API), while client state is data that exists only in the browser (UI state, form inputs) - each requires different management strategies and tools."

### The 7 Key Concepts (Remember These!)
```
1. SERVER STATE     → API data, remote, async, shared ownership
2. CLIENT STATE     → UI state, local, synchronous, single owner
3. CACHING          → Store fetched data to avoid refetching
4. STALE-WHILE-REVALIDATE → Show cached data, refetch in background
5. DEDUPLICATION    → Multiple components, single request
6. OPTIMISTIC UPDATES → Update UI before server confirms
7. BACKGROUND REFETCHING → Keep data fresh automatically
```

### Server State vs Client State
```
┌─────────────────────────────────────────────────────────────────┐
│              SERVER STATE vs CLIENT STATE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SERVER STATE                   │ CLIENT STATE                  │
│  ─────────────────────────────  │ ─────────────────────────────│
│  Lives on server                │ Lives in browser only         │
│  Async (network latency)        │ Synchronous                   │
│  Shared ownership               │ Single owner                  │
│  Can become stale               │ Always current                │
│  Requires caching               │ No caching needed             │
│  Examples:                      │ Examples:                     │
│  • User profile                 │ • Modal open/closed           │
│  • Product list                 │ • Selected tab                │
│  • Comments                     │ • Form input values           │
│  • Order history                │ • Sidebar collapsed           │
│  • Search results               │ • Theme preference            │
│                                                                 │
│  TOOLS                          │ TOOLS                         │
│  ─────                          │ ─────                         │
│  • React Query                  │ • useState                    │
│  • SWR                          │ • useReducer                  │
│  • Apollo Client                │ • Zustand                     │
│  • RTK Query                    │ • Jotai                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### React Query vs SWR
```
┌─────────────────────────────────────────────────────────────────┐
│                REACT QUERY vs SWR                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  REACT QUERY (TanStack Query)                                  │
│  ────────────────────────────                                   │
│  • More features out of the box                                │
│  • Mutations with optimistic updates                           │
│  • Pagination helpers                                          │
│  • DevTools included                                           │
│  • Parallel/dependent queries                                  │
│  • Bundle: ~12KB                                               │
│  Use: Complex data requirements                                │
│                                                                 │
│  SWR (Stale-While-Revalidate)                                  │
│  ────────────────────────────                                   │
│  • Simpler API                                                 │
│  • Lighter weight                                              │
│  • Focus on data fetching                                      │
│  • Less opinionated                                            │
│  • Bundle: ~4KB                                                │
│  Use: Simpler requirements, Next.js                            │
│                                                                 │
│  BOTH PROVIDE:                                                 │
│  • Automatic caching                                           │
│  • Request deduplication                                       │
│  • Background refetching                                       │
│  • Stale-while-revalidate                                      │
│  • Error handling                                              │
│  • Loading states                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Stale-while-revalidate"** | "We use stale-while-revalidate for instant UI while refetching" |
| **"Request deduplication"** | "React Query deduplicates parallel requests automatically" |
| **"Optimistic update"** | "Mutations use optimistic updates for instant feedback" |
| **"Cache invalidation"** | "We invalidate related queries after mutations" |
| **"Background refetch"** | "Queries refetch in background on window focus" |
| **"Query key"** | "Query keys determine cache identity and dependencies" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Default stale time | **0ms** | Immediately considered stale |
| Default cache time | **5 minutes** | Inactive queries cached |
| Refetch on focus | **Enabled** | Fresh data on tab switch |
| Retry count | **3** | Auto-retry failed requests |

### The "Wow" Statement (Memorize This!)
> "We separate server state from client state - React Query handles API data with automatic caching, deduplication, and background refetching. Multiple components can useQuery with the same key, and only one request fires. We set staleTime based on data volatility - user profile is 5 minutes, stock prices are 0. Mutations invalidate related queries and use optimistic updates for instant UI feedback. If mutation fails, we rollback. Client state stays in Zustand for UI-only concerns. This separation eliminates manual loading/error states, reduces duplicate fetches by 80%, and keeps data fresh automatically. DevTools show cache status, helping debug stale data issues."

---

## 📚 Table of Contents

1. [React Query Basics](#1-react-query-basics)
2. [Advanced Queries](#2-advanced-queries)
3. [Mutations](#3-mutations)
4. [SWR](#4-swr)
5. [Caching Strategies](#5-caching-strategies)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. React Query Basics

```tsx
// ════════════════════════════════════════════════════════════════
// REACT QUERY SETUP
// ════════════════════════════════════════════════════════════════

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

// Create client with default options
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60, // 1 minute
      gcTime: 1000 * 60 * 5, // 5 minutes (was cacheTime)
      retry: 3,
      refetchOnWindowFocus: true,
      refetchOnReconnect: true,
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}

// ════════════════════════════════════════════════════════════════
// BASIC QUERY
// ════════════════════════════════════════════════════════════════

import { useQuery } from '@tanstack/react-query';

// API function (separate from hook)
async function fetchUsers(): Promise<User[]> {
  const response = await fetch('/api/users');
  if (!response.ok) throw new Error('Failed to fetch users');
  return response.json();
}

// Query hook
function useUsers() {
  return useQuery({
    queryKey: ['users'],           // Cache key
    queryFn: fetchUsers,           // Fetch function
    staleTime: 1000 * 60 * 5,     // Fresh for 5 minutes
  });
}

// Component usage
function UserList() {
  const { data, isLoading, isError, error, refetch } = useUsers();

  if (isLoading) return <Spinner />;
  if (isError) return <Error message={error.message} />;

  return (
    <div>
      <button onClick={() => refetch()}>Refresh</button>
      <ul>
        {data?.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// QUERY WITH PARAMETERS
// ════════════════════════════════════════════════════════════════

async function fetchUser(userId: string): Promise<User> {
  const response = await fetch(`/api/users/${userId}`);
  if (!response.ok) throw new Error('User not found');
  return response.json();
}

function useUser(userId: string) {
  return useQuery({
    queryKey: ['users', userId],   // Key includes parameter
    queryFn: () => fetchUser(userId),
    enabled: !!userId,             // Only run if userId exists
  });
}

// Shared query key constants
const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: UserFilters) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: string) => [...userKeys.details(), id] as const,
};

// Usage
function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading } = useQuery({
    queryKey: userKeys.detail(userId),
    queryFn: () => fetchUser(userId),
  });

  if (isLoading) return <Skeleton />;
  return <div>{user?.name}</div>;
}

// ════════════════════════════════════════════════════════════════
// QUERY STATUS STATES
// ════════════════════════════════════════════════════════════════

function UserListWithStates() {
  const {
    data,
    // Loading states
    isLoading,         // First load, no data
    isFetching,        // Any fetch (including background)
    isRefetching,      // Background refetch (has data)
    isPending,         // No data yet (new in v5)
    
    // Result states
    isSuccess,         // Has data
    isError,           // Has error
    
    // Data/error
    error,
    
    // Status (union type)
    status,            // 'pending' | 'error' | 'success'
    fetchStatus,       // 'fetching' | 'paused' | 'idle'
  } = useUsers();

  // Common patterns
  if (isLoading) {
    // First load - show skeleton
    return <Skeleton />;
  }

  if (isError) {
    return <Error message={error.message} />;
  }

  return (
    <div>
      {/* Show indicator for background refetch */}
      {isFetching && <RefetchingIndicator />}
      
      <ul>
        {data?.map(user => <UserItem key={user.id} user={user} />)}
      </ul>
    </div>
  );
}
```

---

## 2. Advanced Queries

```tsx
// ════════════════════════════════════════════════════════════════
// DEPENDENT QUERIES
// ════════════════════════════════════════════════════════════════

function UserPosts({ userId }: { userId: string }) {
  // First query: Get user
  const { data: user } = useQuery({
    queryKey: ['users', userId],
    queryFn: () => fetchUser(userId),
  });

  // Second query: Get posts (depends on user)
  const { data: posts, isLoading } = useQuery({
    queryKey: ['posts', user?.id],
    queryFn: () => fetchPostsByUser(user!.id),
    enabled: !!user, // Only run when user is available
  });

  if (isLoading) return <Spinner />;
  return <PostList posts={posts} />;
}

// ════════════════════════════════════════════════════════════════
// PARALLEL QUERIES
// ════════════════════════════════════════════════════════════════

import { useQueries } from '@tanstack/react-query';

function Dashboard() {
  // Multiple independent queries in parallel
  const results = useQueries({
    queries: [
      {
        queryKey: ['users'],
        queryFn: fetchUsers,
      },
      {
        queryKey: ['products'],
        queryFn: fetchProducts,
      },
      {
        queryKey: ['orders'],
        queryFn: fetchOrders,
      },
    ],
  });

  const isLoading = results.some(r => r.isLoading);
  const [usersQuery, productsQuery, ordersQuery] = results;

  if (isLoading) return <DashboardSkeleton />;

  return (
    <div>
      <UserStats users={usersQuery.data} />
      <ProductList products={productsQuery.data} />
      <RecentOrders orders={ordersQuery.data} />
    </div>
  );
}

// Dynamic parallel queries
function UserDetails({ userIds }: { userIds: string[] }) {
  const userQueries = useQueries({
    queries: userIds.map(id => ({
      queryKey: ['users', id],
      queryFn: () => fetchUser(id),
    })),
  });

  return (
    <ul>
      {userQueries.map((query, index) => (
        <li key={userIds[index]}>
          {query.isLoading ? 'Loading...' : query.data?.name}
        </li>
      ))}
    </ul>
  );
}

// ════════════════════════════════════════════════════════════════
// INFINITE QUERIES (Pagination)
// ════════════════════════════════════════════════════════════════

import { useInfiniteQuery } from '@tanstack/react-query';

interface PageData {
  items: User[];
  nextCursor: string | null;
}

async function fetchUsersPage({ pageParam }: { pageParam: string | null }): Promise<PageData> {
  const url = pageParam 
    ? `/api/users?cursor=${pageParam}` 
    : '/api/users';
  const response = await fetch(url);
  return response.json();
}

function InfiniteUserList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
    isError,
  } = useInfiniteQuery({
    queryKey: ['users', 'infinite'],
    queryFn: fetchUsersPage,
    initialPageParam: null,
    getNextPageParam: (lastPage) => lastPage.nextCursor,
  });

  if (isLoading) return <Spinner />;
  if (isError) return <Error />;

  // Flatten all pages into single array
  const allUsers = data?.pages.flatMap(page => page.items) ?? [];

  return (
    <div>
      <ul>
        {allUsers.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
      
      {hasNextPage && (
        <button
          onClick={() => fetchNextPage()}
          disabled={isFetchingNextPage}
        >
          {isFetchingNextPage ? 'Loading...' : 'Load More'}
        </button>
      )}
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// PREFETCHING
// ════════════════════════════════════════════════════════════════

import { useQueryClient } from '@tanstack/react-query';

function UserListWithPrefetch() {
  const queryClient = useQueryClient();
  const { data: users } = useUsers();

  // Prefetch user details on hover
  const prefetchUser = (userId: string) => {
    queryClient.prefetchQuery({
      queryKey: ['users', userId],
      queryFn: () => fetchUser(userId),
      staleTime: 1000 * 60 * 5, // 5 minutes
    });
  };

  return (
    <ul>
      {users?.map(user => (
        <li
          key={user.id}
          onMouseEnter={() => prefetchUser(user.id)}
        >
          <Link to={`/users/${user.id}`}>{user.name}</Link>
        </li>
      ))}
    </ul>
  );
}

// Prefetch in loader (React Router)
export async function userLoader({ params }: LoaderFunctionArgs) {
  const queryClient = getQueryClient();
  
  await queryClient.prefetchQuery({
    queryKey: ['users', params.userId],
    queryFn: () => fetchUser(params.userId!),
  });
  
  return null;
}
```

---

## 3. Mutations

```tsx
// ════════════════════════════════════════════════════════════════
// BASIC MUTATION
// ════════════════════════════════════════════════════════════════

import { useMutation, useQueryClient } from '@tanstack/react-query';

async function createUser(userData: CreateUserInput): Promise<User> {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(userData),
  });
  if (!response.ok) throw new Error('Failed to create user');
  return response.json();
}

function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createUser,
    onSuccess: (newUser) => {
      // Invalidate and refetch users list
      queryClient.invalidateQueries({ queryKey: ['users'] });
      
      // Or update cache directly
      queryClient.setQueryData(['users'], (old: User[] | undefined) => 
        old ? [...old, newUser] : [newUser]
      );
    },
    onError: (error) => {
      console.error('Failed to create user:', error);
      toast.error('Failed to create user');
    },
  });
}

function CreateUserForm() {
  const { mutate, isPending, isError, error } = useCreateUser();

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    
    mutate({
      name: formData.get('name') as string,
      email: formData.get('email') as string,
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create User'}
      </button>
      {isError && <p className="error">{error.message}</p>}
    </form>
  );
}

// ════════════════════════════════════════════════════════════════
// OPTIMISTIC UPDATES
// ════════════════════════════════════════════════════════════════

async function updateUser(userId: string, data: Partial<User>): Promise<User> {
  const response = await fetch(`/api/users/${userId}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  });
  return response.json();
}

function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ userId, data }: { userId: string; data: Partial<User> }) =>
      updateUser(userId, data),
    
    // Optimistic update
    onMutate: async ({ userId, data }) => {
      // Cancel any outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['users', userId] });

      // Snapshot previous value
      const previousUser = queryClient.getQueryData<User>(['users', userId]);

      // Optimistically update
      queryClient.setQueryData(['users', userId], (old: User | undefined) =>
        old ? { ...old, ...data } : old
      );

      // Return context with snapshot
      return { previousUser };
    },
    
    // Rollback on error
    onError: (err, { userId }, context) => {
      if (context?.previousUser) {
        queryClient.setQueryData(['users', userId], context.previousUser);
      }
      toast.error('Failed to update user');
    },
    
    // Always refetch after error or success
    onSettled: (data, error, { userId }) => {
      queryClient.invalidateQueries({ queryKey: ['users', userId] });
    },
  });
}

// Usage
function UserEditForm({ user }: { user: User }) {
  const { mutate, isPending } = useUpdateUser();
  const [name, setName] = useState(user.name);

  const handleSave = () => {
    mutate({ userId: user.id, data: { name } });
  };

  return (
    <div>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
      />
      <button onClick={handleSave} disabled={isPending}>
        {isPending ? 'Saving...' : 'Save'}
      </button>
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// DELETE WITH OPTIMISTIC UPDATE
// ════════════════════════════════════════════════════════════════

function useDeleteUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (userId: string) =>
      fetch(`/api/users/${userId}`, { method: 'DELETE' }),
    
    onMutate: async (userId) => {
      await queryClient.cancelQueries({ queryKey: ['users'] });
      
      const previousUsers = queryClient.getQueryData<User[]>(['users']);
      
      // Optimistically remove from list
      queryClient.setQueryData(['users'], (old: User[] | undefined) =>
        old?.filter(u => u.id !== userId)
      );
      
      return { previousUsers };
    },
    
    onError: (err, userId, context) => {
      queryClient.setQueryData(['users'], context?.previousUsers);
      toast.error('Failed to delete user');
    },
    
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

function UserItem({ user }: { user: User }) {
  const { mutate: deleteUser, isPending } = useDeleteUser();

  return (
    <li className={isPending ? 'opacity-50' : ''}>
      {user.name}
      <button 
        onClick={() => deleteUser(user.id)} 
        disabled={isPending}
      >
        Delete
      </button>
    </li>
  );
}
```

---

## 4. SWR

```tsx
// ════════════════════════════════════════════════════════════════
// SWR BASICS
// ════════════════════════════════════════════════════════════════

import useSWR, { SWRConfig } from 'swr';

// Global fetcher
const fetcher = async (url: string) => {
  const response = await fetch(url);
  if (!response.ok) throw new Error('Failed to fetch');
  return response.json();
};

// Provider with defaults
function App() {
  return (
    <SWRConfig
      value={{
        fetcher,
        revalidateOnFocus: true,
        revalidateOnReconnect: true,
        dedupingInterval: 2000,
      }}
    >
      <YourApp />
    </SWRConfig>
  );
}

// Basic usage
function UserList() {
  const { data, error, isLoading, isValidating, mutate } = useSWR<User[]>('/api/users');

  if (isLoading) return <Spinner />;
  if (error) return <Error message={error.message} />;

  return (
    <div>
      {isValidating && <RefreshIndicator />}
      <button onClick={() => mutate()}>Refresh</button>
      <ul>
        {data?.map(user => <li key={user.id}>{user.name}</li>)}
      </ul>
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// SWR WITH PARAMETERS
// ════════════════════════════════════════════════════════════════

function UserProfile({ userId }: { userId: string }) {
  // Key can be array or null (to disable)
  const { data: user, error } = useSWR<User>(
    userId ? `/api/users/${userId}` : null
  );

  if (!user) return <Spinner />;
  return <div>{user.name}</div>;
}

// Custom hook pattern
function useUser(userId: string | null) {
  return useSWR<User>(
    userId ? `/api/users/${userId}` : null,
    fetcher,
    {
      revalidateOnFocus: false,
      dedupingInterval: 60000, // 1 minute
    }
  );
}

// ════════════════════════════════════════════════════════════════
// SWR MUTATIONS
// ════════════════════════════════════════════════════════════════

import useSWRMutation from 'swr/mutation';

// Mutation fetcher
async function updateUser(url: string, { arg }: { arg: Partial<User> }) {
  const response = await fetch(url, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(arg),
  });
  return response.json();
}

function UserEditor({ userId }: { userId: string }) {
  const { data: user, mutate } = useSWR<User>(`/api/users/${userId}`);
  
  const { trigger, isMutating } = useSWRMutation(
    `/api/users/${userId}`,
    updateUser
  );

  const handleUpdate = async (newName: string) => {
    // Optimistic update
    await trigger(
      { name: newName },
      {
        optimisticData: user ? { ...user, name: newName } : undefined,
        rollbackOnError: true,
        populateCache: true,
        revalidate: false,
      }
    );
  };

  return (
    <div>
      <input 
        defaultValue={user?.name}
        onBlur={(e) => handleUpdate(e.target.value)}
        disabled={isMutating}
      />
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// SWR INFINITE (Pagination)
// ════════════════════════════════════════════════════════════════

import useSWRInfinite from 'swr/infinite';

function InfiniteUserList() {
  const getKey = (pageIndex: number, previousPageData: User[] | null) => {
    // Reached the end
    if (previousPageData && previousPageData.length === 0) return null;
    
    // First page
    if (pageIndex === 0) return '/api/users?limit=10';
    
    // Next page
    return `/api/users?limit=10&offset=${pageIndex * 10}`;
  };

  const {
    data,
    size,
    setSize,
    isLoading,
    isValidating,
  } = useSWRInfinite<User[]>(getKey, fetcher);

  const users = data ? data.flat() : [];
  const isLoadingMore = isLoading || (size > 0 && data && typeof data[size - 1] === 'undefined');
  const isEmpty = data?.[0]?.length === 0;
  const isReachingEnd = isEmpty || (data && data[data.length - 1]?.length < 10);

  return (
    <div>
      {users.map(user => <div key={user.id}>{user.name}</div>)}
      
      <button
        onClick={() => setSize(size + 1)}
        disabled={isLoadingMore || isReachingEnd}
      >
        {isLoadingMore ? 'Loading...' : isReachingEnd ? 'No more' : 'Load more'}
      </button>
    </div>
  );
}
```

---

## 5. Caching Strategies

```tsx
// ════════════════════════════════════════════════════════════════
// CACHE CONFIGURATION
// ════════════════════════════════════════════════════════════════

// Different stale times based on data type
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60, // Default: 1 minute
    },
  },
});

// User profile - changes infrequently
const useUserProfile = (userId: string) =>
  useQuery({
    queryKey: ['users', userId, 'profile'],
    queryFn: () => fetchUserProfile(userId),
    staleTime: 1000 * 60 * 10, // 10 minutes
  });

// Stock prices - always fresh
const useStockPrice = (symbol: string) =>
  useQuery({
    queryKey: ['stocks', symbol],
    queryFn: () => fetchStockPrice(symbol),
    staleTime: 0, // Always stale
    refetchInterval: 5000, // Poll every 5 seconds
  });

// Static data - cache forever
const useCountries = () =>
  useQuery({
    queryKey: ['countries'],
    queryFn: fetchCountries,
    staleTime: Infinity, // Never stale
    gcTime: Infinity, // Never garbage collect
  });

// ════════════════════════════════════════════════════════════════
// CACHE INVALIDATION PATTERNS
// ════════════════════════════════════════════════════════════════

function useOrderMutation() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createOrder,
    onSuccess: () => {
      // Invalidate specific query
      queryClient.invalidateQueries({ queryKey: ['orders'] });
      
      // Invalidate all queries starting with 'orders'
      queryClient.invalidateQueries({ queryKey: ['orders'], exact: false });
      
      // Invalidate multiple related queries
      queryClient.invalidateQueries({ queryKey: ['orders'] });
      queryClient.invalidateQueries({ queryKey: ['user', 'stats'] });
      queryClient.invalidateQueries({ queryKey: ['inventory'] });
    },
  });
}

// ════════════════════════════════════════════════════════════════
// CACHE PERSISTENCE
// ════════════════════════════════════════════════════════════════

import { persistQueryClient } from '@tanstack/react-query-persist-client';
import { createSyncStoragePersister } from '@tanstack/query-sync-storage-persister';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      gcTime: 1000 * 60 * 60 * 24, // 24 hours
    },
  },
});

const persister = createSyncStoragePersister({
  storage: window.localStorage,
});

persistQueryClient({
  queryClient,
  persister,
  maxAge: 1000 * 60 * 60 * 24, // 24 hours
});

// ════════════════════════════════════════════════════════════════
// INITIAL DATA & PLACEHOLDER
// ════════════════════════════════════════════════════════════════

function UserProfile({ userId }: { userId: string }) {
  const queryClient = useQueryClient();

  const { data } = useQuery({
    queryKey: ['users', userId],
    queryFn: () => fetchUser(userId),
    
    // Use cached data from list as initial
    initialData: () => {
      const users = queryClient.getQueryData<User[]>(['users']);
      return users?.find(u => u.id === userId);
    },
    initialDataUpdatedAt: () => {
      return queryClient.getQueryState(['users'])?.dataUpdatedAt;
    },
    
    // Or use placeholder while loading
    placeholderData: {
      id: userId,
      name: 'Loading...',
      email: '',
    },
  });

  return <div>{data?.name}</div>;
}

// ════════════════════════════════════════════════════════════════
// HYDRATION (SSR)
// ════════════════════════════════════════════════════════════════

import { HydrationBoundary, dehydrate } from '@tanstack/react-query';

// Server-side (Next.js)
export async function getServerSideProps() {
  const queryClient = new QueryClient();

  await queryClient.prefetchQuery({
    queryKey: ['users'],
    queryFn: fetchUsers,
  });

  return {
    props: {
      dehydratedState: dehydrate(queryClient),
    },
  };
}

// Client-side
function App({ dehydratedState }) {
  const [queryClient] = useState(() => new QueryClient());

  return (
    <QueryClientProvider client={queryClient}>
      <HydrationBoundary state={dehydratedState}>
        <UserList />
      </HydrationBoundary>
    </QueryClientProvider>
  );
}
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# SERVER STATE MANAGEMENT PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Using Redux for server state
# Bad
dispatch(fetchUsersStart());
const users = await api.getUsers();
dispatch(fetchUsersSuccess(users));
# Manual loading, error, caching logic

# Good
const { data, isLoading, error } = useQuery(['users'], fetchUsers);
# Automatic caching, deduplication, refetching

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Fetching on every render
# Bad
useEffect(() => {
  fetch('/api/users').then(setUsers);
}, []); // No caching, duplicate requests

# Good
const { data } = useQuery(['users'], fetchUsers);
# Cached, deduplicated, smart refetching

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Not setting appropriate staleTime
# Bad
# Default staleTime is 0 - always refetches
useQuery(['users'], fetchUsers);

# Good - Match to data volatility
useQuery(['users'], fetchUsers, { staleTime: 5 * 60 * 1000 });
# User list: 5 minutes
# Stock price: 0 (always fresh)
# Countries: Infinity (static)

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Forgetting to invalidate after mutation
# Bad
const mutation = useMutation(createUser);
# List still shows old data

# Good
const mutation = useMutation(createUser, {
  onSuccess: () => {
    queryClient.invalidateQueries(['users']);
  },
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Inconsistent query keys
# Bad
useQuery(['user', id])
useQuery(['users', id])
useQuery([id, 'user'])
# Different cache entries!

# Good - Use key factory
const userKeys = {
  detail: (id: string) => ['users', id] as const,
};
useQuery(userKeys.detail(id));

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Not handling loading/error states
# Bad
const { data } = useQuery(['users'], fetchUsers);
return <ul>{data.map(...)}</ul>; // Crashes if data undefined

# Good
const { data, isLoading, error } = useQuery(['users'], fetchUsers);
if (isLoading) return <Spinner />;
if (error) return <Error />;
return <ul>{data?.map(...)}</ul>;
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is server state vs client state?"**
> "Server state is data from the server that we cache on the client - it can become stale, is async, and has shared ownership. Client state exists only in the browser (modals, forms, theme). They need different tools: React Query/SWR for server state, useState/Zustand for client state."

**Q: "What does 'stale-while-revalidate' mean?"**
> "Show cached (stale) data immediately for instant UI, while refetching fresh data in the background. When new data arrives, update the UI. Users see content instantly but always get fresh data. It's the core pattern of React Query and SWR."

**Q: "Why use React Query instead of useEffect + useState?"**
> "React Query provides: automatic caching, request deduplication (multiple components, one request), background refetching, error/retry handling, optimistic updates, and devtools. With useEffect, you manually implement all of this - more code, more bugs."

### Intermediate Questions

**Q: "How does query deduplication work?"**
> "If multiple components use the same query key, only one request fires. React Query tracks in-flight requests by key. Components mount, see same key, share the same promise. Result is shared, all components update together. Huge performance benefit."

**Q: "What is optimistic update and when would you use it?"**
> "Update the UI immediately before server confirms, assuming success. If server fails, rollback. Use for: likes, toggles, edits - actions that usually succeed. onMutate: save previous state, update cache. onError: rollback. onSettled: refetch to ensure sync."

**Q: "Explain staleTime vs gcTime (cacheTime)."**
> "staleTime: How long data is 'fresh' (won't refetch). Default 0. gcTime: How long inactive query stays in cache. Default 5 minutes. After staleTime, data is stale but still shown while refetching. After gcTime, data is garbage collected completely."

### Advanced Questions

**Q: "How would you handle real-time data with React Query?"**
> "Options: 1) refetchInterval for polling. 2) WebSocket updates + queryClient.setQueryData for real-time push. 3) queryClient.invalidateQueries when WS message arrives. For chat/feeds: combine WS for instant updates with React Query for initial load and caching."

**Q: "How do you test components using React Query?"**
> "Wrap in QueryClientProvider with test-specific client (retry: false, gcTime: 0). Use msw to mock API responses. For mutations, mock and assert onSuccess called. Test loading/error states by controlling mock timing. React Query provides testing utilities."

**Q: "How do you handle offline support with React Query?"**
> "Enable persistence (react-query-persist-client) to localStorage/IndexedDB. Mutations queue when offline, execute when online (onlineManager). Set networkMode: 'offlineFirst' to serve cache when offline. Combine with service worker for full offline experience."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│            SERVER STATE MANAGEMENT CHECKLIST                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SETUP:                                                         │
│  □ QueryClientProvider at app root                             │
│  □ Default staleTime based on data type                        │
│  □ DevTools in development                                     │
│                                                                 │
│  QUERIES:                                                       │
│  □ Consistent query key patterns                               │
│  □ Handle loading, error, success states                       │
│  □ Appropriate staleTime per query                             │
│  □ Enable/disable with enabled option                          │
│                                                                 │
│  MUTATIONS:                                                     │
│  □ Invalidate related queries onSuccess                        │
│  □ Optimistic updates for instant feedback                     │
│  □ Rollback on error                                           │
│  □ Show loading state during mutation                          │
│                                                                 │
│  PERFORMANCE:                                                   │
│  □ Prefetch on hover for navigation                            │
│  □ Parallel queries when independent                           │
│  □ Infinite queries for pagination                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

REACT QUERY vs SWR:
┌────────────────────────────────────────────────────────────────┐
│ React Query: More features, ~12KB, complex requirements        │
│ SWR:         Simpler API, ~4KB, Next.js integration           │
│ Both:        Caching, deduplication, background refetch        │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

