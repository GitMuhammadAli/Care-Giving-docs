# TanStack Query (React Query)

> Powerful server state management for React applications.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | Async state management for React |
| **Why** | Caching, background updates, optimistic UI |
| **Version** | 5.x |
| **Location** | `apps/web/src/` |

## Setup

```tsx
// app/providers.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useState } from 'react';

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 60 * 1000, // 1 minute
        gcTime: 5 * 60 * 1000, // 5 minutes (formerly cacheTime)
        retry: 1,
        refetchOnWindowFocus: false,
      },
    },
  }));

  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

## Basic Usage

### useQuery - Fetching Data
```tsx
import { useQuery } from '@tanstack/react-query';

function MedicationsList({ careRecipientId }: { careRecipientId: string }) {
  const { data, isLoading, error, refetch } = useQuery({
    queryKey: ['medications', careRecipientId],
    queryFn: () => api.getMedications(careRecipientId),
    enabled: !!careRecipientId, // Only fetch if ID exists
  });

  if (isLoading) return <Spinner />;
  if (error) return <Error message={error.message} />;

  return (
    <ul>
      {data?.map(med => (
        <MedicationCard key={med.id} medication={med} />
      ))}
    </ul>
  );
}
```

### useMutation - Modifying Data
```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function CreateMedicationForm() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (data: CreateMedicationDto) => api.createMedication(data),
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['medications'] });
    },
    onError: (error) => {
      toast.error(`Failed: ${error.message}`);
    },
  });

  const handleSubmit = (data: CreateMedicationDto) => {
    mutation.mutate(data);
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
      <button disabled={mutation.isPending}>
        {mutation.isPending ? 'Creating...' : 'Create'}
      </button>
    </form>
  );
}
```

## Query Keys Convention

```typescript
// Hierarchical query keys for targeted invalidation
const queryKeys = {
  // Base
  all: ['medications'] as const,
  
  // Lists
  lists: () => [...queryKeys.all, 'list'] as const,
  list: (filters: Filters) => [...queryKeys.lists(), filters] as const,
  
  // Details
  details: () => [...queryKeys.all, 'detail'] as const,
  detail: (id: string) => [...queryKeys.details(), id] as const,
};

// Usage
useQuery({
  queryKey: queryKeys.detail(medicationId),
  queryFn: () => api.getMedication(medicationId),
});

// Invalidate all medication queries
queryClient.invalidateQueries({ queryKey: queryKeys.all });

// Invalidate only lists
queryClient.invalidateQueries({ queryKey: queryKeys.lists() });
```

## Optimistic Updates

```tsx
const updateMedication = useMutation({
  mutationFn: api.updateMedication,
  // Optimistically update before server response
  onMutate: async (newData) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['medications', newData.id] });

    // Snapshot previous value
    const previousData = queryClient.getQueryData(['medications', newData.id]);

    // Optimistically update
    queryClient.setQueryData(['medications', newData.id], newData);

    // Return context for rollback
    return { previousData };
  },
  // Rollback on error
  onError: (err, newData, context) => {
    queryClient.setQueryData(
      ['medications', newData.id],
      context?.previousData
    );
  },
  // Always refetch after success or error
  onSettled: (data, error, variables) => {
    queryClient.invalidateQueries({ queryKey: ['medications', variables.id] });
  },
});
```

## Common Patterns

### Dependent Queries
```tsx
// Query 1: Get user
const { data: user } = useQuery({
  queryKey: ['user'],
  queryFn: fetchUser,
});

// Query 2: Get user's families (depends on user)
const { data: families } = useQuery({
  queryKey: ['families', user?.id],
  queryFn: () => fetchFamilies(user!.id),
  enabled: !!user, // Only runs when user exists
});
```

### Parallel Queries
```tsx
function Dashboard() {
  const medicationsQuery = useQuery({
    queryKey: ['medications'],
    queryFn: fetchMedications,
  });

  const appointmentsQuery = useQuery({
    queryKey: ['appointments'],
    queryFn: fetchAppointments,
  });

  // Both fetch in parallel
  const isLoading = medicationsQuery.isLoading || appointmentsQuery.isLoading;
}
```

### Infinite Queries
```tsx
const {
  data,
  fetchNextPage,
  hasNextPage,
  isFetchingNextPage,
} = useInfiniteQuery({
  queryKey: ['timeline', careRecipientId],
  queryFn: ({ pageParam = 0 }) => api.getTimeline(careRecipientId, pageParam),
  getNextPageParam: (lastPage) => lastPage.nextCursor,
  initialPageParam: 0,
});

// Render all pages
{data?.pages.map(page =>
  page.items.map(item => <TimelineItem key={item.id} item={item} />)
)}

// Load more button
<button
  onClick={() => fetchNextPage()}
  disabled={!hasNextPage || isFetchingNextPage}
>
  {isFetchingNextPage ? 'Loading...' : 'Load More'}
</button>
```

### Prefetching
```tsx
// In a list, prefetch details on hover
function MedicationCard({ id }: { id: string }) {
  const queryClient = useQueryClient();

  const prefetchDetails = () => {
    queryClient.prefetchQuery({
      queryKey: ['medications', id],
      queryFn: () => api.getMedication(id),
      staleTime: 60 * 1000,
    });
  };

  return (
    <div onMouseEnter={prefetchDetails}>
      {/* card content */}
    </div>
  );
}
```

## Custom Hooks

```typescript
// hooks/useMedications.ts
export function useMedications(careRecipientId: string) {
  return useQuery({
    queryKey: ['medications', careRecipientId],
    queryFn: () => api.getMedications(careRecipientId),
    enabled: !!careRecipientId,
  });
}

export function useCreateMedication() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: api.createMedication,
    onSuccess: (data) => {
      queryClient.invalidateQueries({
        queryKey: ['medications', data.careRecipientId],
      });
      toast.success('Medication created');
    },
  });
}

// Usage
const { data: medications, isLoading } = useMedications(careRecipientId);
const createMutation = useCreateMedication();
```

## Troubleshooting

### Stale Data
- Check `staleTime` configuration
- Use `queryClient.invalidateQueries()` after mutations
- Consider `refetchOnWindowFocus: true`

### Over-fetching
- Use appropriate `staleTime` and `gcTime`
- Enable `refetchOnWindowFocus: false` if not needed
- Use `enabled` option for conditional fetching

### Memory Issues
- Reduce `gcTime` for large datasets
- Use `select` to minimize stored data
- Consider pagination for large lists

---

*See also: [Zustand](zustand.md), [React](react.md)*


