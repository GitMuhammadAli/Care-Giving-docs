# 🔄 Complete Guide to Offline Sync in Web & Mobile Applications

> A comprehensive guide covering everything you need to know about building offline-first applications with robust synchronization strategies.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Offline sync lets users work without internet by storing data locally and automatically syncing when connectivity returns."

### The 6 Key Concepts (Remember These!)
```
1. OPTIMISTIC UI      → Show success immediately, sync in background
2. ACTION QUEUE       → Store pending actions in IndexedDB
3. CONFLICT RESOLUTION → Last-write-wins, three-way merge, or CRDTs
4. SERVICE WORKER     → Background sync even when app is closed
5. CACHE FIRST        → Local data is primary, server is secondary
6. BACKPRESSURE       → Sync slowly to not overwhelm server
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Optimistic UI"** | "We use optimistic UI for instant feedback" |
| **"Eventual consistency"** | "The system is eventually consistent" |
| **"IndexedDB"** | "We store offline data in IndexedDB, not localStorage" |
| **"Exponential backoff"** | "Retries use exponential backoff" |
| **"Idempotent"** | "Operations must be idempotent for safe retries" |
| **"CRDT"** | "For real-time collab, CRDTs eliminate conflicts" |
| **"Backpressure"** | "We apply backpressure to not overwhelm server during sync" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| localStorage limit | **5-10 MB** | Too small for real apps |
| IndexedDB limit | **50MB+** | Dynamic, browser manages |
| Retry attempts | **3-5** | More = annoying, less = data loss |
| Backoff max | **30 seconds** | Don't wait forever |

### The "Wow" Statement (Memorize This!)
> "The hardest part of offline sync isn't storing data - it's conflict resolution. When the same record is edited on multiple devices offline, you need a strategy. For simple data, last-write-wins works. For collaborative editing, CRDTs like Automerge mathematically guarantee conflict-free merging."

### Quick Architecture Drawing (Draw This!)
```
User Action
    ↓
┌─────────────┐
│ Save Local  │ ← IndexedDB
│ (Optimistic)│
└─────────────┘
    ↓
┌─────────────┐     ┌─────────────┐
│ Add to Queue│ ──▶ │ Sync When   │ ──▶ Server
│             │     │ Online      │
└─────────────┘     └─────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is offline sync?"**
> "Storing data locally, queueing actions, syncing when online, handling conflicts."

**Q: "Where do you store offline data?"**
> "IndexedDB via localforage - localStorage is only 5MB."

**Q: "How do you handle conflicts?"**
> "Depends on use case: last-write-wins for simple data, CRDTs for collaboration."

**Q: "What if user was offline for a month with 1000 pending actions?"**
> "Backpressure - sync in batches with delays. Don't flood the server, process 10-50 at a time."

**Q: "What's the hardest part?"**
> "Cache invalidation and conflict resolution."

---

## 🎯 How to Explain Like a Senior Developer

### The Perfect Answer: "What is Offline Sync?"

**❌ Junior Answer:**
> "Offline sync is when you save data locally and then sync it to the server when you're back online."

**✅ Senior Answer:**
> "Offline sync is an architectural pattern that allows applications to remain fully functional without an internet connection. The core idea is to treat the local device as the source of truth for user interactions, queue any mutations, and reconcile with the server when connectivity returns.
>
> The key challenges are: deciding what to cache, handling conflicts when the same data is modified in multiple places, and ensuring data consistency. We typically use IndexedDB for storage, implement optimistic UI for instant feedback, and use strategies like 'last write wins' or 'three-way merge' for conflict resolution.
>
> It's essential for apps where users might have unreliable connectivity - like field service apps, healthcare apps, or anything used in areas with poor network coverage."

---

### Structure Your Answer (STAR-like for Technical Questions)

```
┌─────────────────────────────────────────────────────────────┐
│           HOW TO STRUCTURE TECHNICAL ANSWERS                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. WHAT (Definition) - 1-2 sentences                       │
│     "Offline sync is an architectural pattern that..."      │
│                                                              │
│  2. WHY (Problem it solves) - 1 sentence                    │
│     "It solves the problem of..."                           │
│                                                              │
│  3. HOW (Key mechanisms) - 2-3 points                       │
│     "It works by: (1) storing locally, (2) queueing,        │
│      (3) syncing when online"                               │
│                                                              │
│  4. CHALLENGES (Show depth) - 1-2 points                    │
│     "The tricky parts are conflict resolution and..."       │
│                                                              │
│  5. EXAMPLE (Real-world context) - 1 sentence               │
│     "For example, in a healthcare app..."                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

### Follow-up Questions They WILL Ask (Be Ready!)

| They Ask | You Should Cover |
|----------|------------------|
| "How do you handle conflicts?" | Last-write-wins, versioning, three-way merge, CRDT |
| "What if user is offline for a month?" | Graduated sync, full resync, data backup |
| "Where do you store the data?" | IndexedDB (via localforage), not localStorage (size limits) |
| "How do you know you're offline?" | navigator.onLine + actual fetch health check |
| "What about large files?" | Chunked uploads, resumable uploads |
| "How do you test offline?" | DevTools offline mode, Playwright setOffline() |
| "What's the difference from caching?" | Caching = read optimization, Offline sync = write + read |

---

### Explain to Different Audiences

**To a Non-Technical Person (PM, CEO):**
> "It's like writing notes in a notebook when you don't have WiFi. When you get WiFi back, those notes automatically get saved to the cloud. The user never loses their work."

**To a Junior Developer:**
> "Instead of failing when there's no internet, we save everything locally first. We keep a queue of things to sync. When internet comes back, we process that queue and send everything to the server."

**To a Senior Developer / Architect:**
> "It's an offline-first architecture using IndexedDB for persistence, an action queue with exponential backoff for failed syncs, and conflict resolution via vector clocks or CRDTs depending on the consistency requirements. We use service workers for background sync when the app isn't active."

**To an Interviewer (Impress them):**
> Start with the senior answer, then say: "Would you like me to go deeper into any specific aspect - like conflict resolution strategies, or the technical implementation with service workers?"

---

### Real Conversation Example

**Interviewer:** "What is offline sync?"

**You:** "Offline sync is an architectural pattern that allows applications to function without internet by treating the local device as the primary data store. 

When a user performs an action offline - say, creating a note - we immediately save it locally to IndexedDB and show success. Behind the scenes, we queue that action. When connectivity returns, we process the queue, send changes to the server, and handle any conflicts.

The main challenges are: what to cache proactively, how to resolve conflicts when the same record was edited on multiple devices, and maintaining a good user experience during sync.

In my experience building a caregiving app, we made sure emergency contact info was always cached and medication logs could be recorded offline - because caregivers often work in hospitals with poor WiFi."

**Interviewer:** "How did you handle conflicts?"

**You:** "We used a combination of strategies. For simple data like settings, last-write-wins with timestamps was sufficient. For more critical data like medication logs, we used server-side reconciliation - the server would flag conflicts and we'd show the user both versions to choose from.

For real-time collaborative scenarios - which we didn't need but I've researched - CRDTs like Automerge are the gold standard because they mathematically guarantee conflict-free merging."

**Interviewer:** "What technologies did you use?"

**You:** "LocalForage for IndexedDB abstraction - it's simpler than raw IndexedDB and falls back to localStorage on older browsers. Service workers for background sync when the app is closed. React Query on the frontend for cache management with stale-while-revalidate patterns. The queue was persisted to IndexedDB so pending actions survive page refreshes."

---

### Power Phrases That Show Expertise

Use these naturally in your answers:

| Phrase | Shows You Know |
|--------|---------------|
| "Optimistic UI" | UX patterns |
| "Eventual consistency" | Distributed systems |
| "Idempotent operations" | Reliability |
| "Exponential backoff" | Retry strategies |
| "CRDT" | Advanced conflict resolution |
| "Service worker background sync" | PWA/browser APIs |
| "Stale-while-revalidate" | Caching strategies |
| "Queue with dead letter" | Production reliability |
| "Three-way merge" | Git-like conflict resolution |

---

### What NOT to Say

| ❌ Don't Say | ✅ Say Instead |
|-------------|----------------|
| "Just save to localStorage" | "Use IndexedDB via localforage for larger capacity" |
| "Check navigator.onLine" | "navigator.onLine plus actual health check fetch" |
| "Sync everything" | "Prioritize what to cache based on criticality" |
| "It's complicated" | "The key challenges are X, Y, Z" |
| "I haven't done that" | "I haven't implemented that, but the approach would be..." |

---

1. [Introduction](#introduction)
2. [Core Concepts](#core-concepts)
3. [Technology Stack & Packages](#technology-stack--packages)
4. [Architecture Patterns](#architecture-patterns)
5. [Implementation Guide](#implementation-guide)
6. [Sync Strategies](#sync-strategies)
7. [Conflict Resolution](#conflict-resolution)
8. [Scenario-Based Solutions](#scenario-based-solutions)
9. [Edge Cases & Solutions](#edge-cases--solutions)
10. [Testing Offline Features](#testing-offline-features)
11. [Performance Optimization](#performance-optimization)
12. [Security Considerations](#security-considerations)
13. [Best Practices](#best-practices)
14. [Real-World Examples](#real-world-examples)

---

## Introduction

### What is Offline Sync?

Offline sync enables applications to function without an internet connection by:
- Storing data locally on the device
- Queuing user actions for later synchronization
- Automatically syncing when connectivity returns
- Handling conflicts between local and server data

### Why Offline-First Matters

| Benefit | Description |
|---------|-------------|
| **Better UX** | Users can work uninterrupted regardless of connectivity |
| **Performance** | Local data access is instant (no network latency) |
| **Reliability** | App works in elevators, tunnels, airplanes, poor signal areas |
| **Data Integrity** | No lost work due to connection drops |
| **Global Reach** | Works in regions with unreliable internet |

### The Offline Spectrum

```
┌─────────────────────────────────────────────────────────────────┐
│                    OFFLINE CAPABILITY LEVELS                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Level 0: Online Only                                           │
│  └── App completely fails without internet                      │
│                                                                  │
│  Level 1: Graceful Degradation                                  │
│  └── Shows cached data, blocks writes                           │
│                                                                  │
│  Level 2: Read-Only Offline                                     │
│  └── Full read access, writes queued                            │
│                                                                  │
│  Level 3: Full Offline                                          │
│  └── Complete functionality, background sync                    │
│                                                                  │
│  Level 4: Offline-First                                         │
│  └── Local-first with eventual consistency                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Core Concepts

### 1. Optimistic UI

Show immediate feedback to users as if the action succeeded, then sync in background.

```javascript
// ❌ Pessimistic (bad UX)
async function saveNote(note) {
  setLoading(true);
  await api.save(note);        // User waits...
  setLoading(false);
  showSuccess();
}

// ✅ Optimistic (good UX)
async function saveNote(note) {
  addToUI(note);               // Instant feedback
  showSuccess();
  try {
    await api.save(note);      // Background sync
  } catch {
    removeFromUI(note);
    showError("Failed to save");
  }
}
```

### 2. Action Queue

Store pending operations in a queue for later execution:

```javascript
interface PendingAction {
  id: string;                  // Unique identifier
  type: string;                // Action type (CREATE, UPDATE, DELETE)
  entity: string;              // What entity (note, task, message)
  payload: any;                // The data
  timestamp: number;           // When created
  retryCount: number;          // Failed attempts
  priority: number;            // Sync order priority
  dependencies: string[];      // Other actions this depends on
}
```

### 3. Sync States

```
┌──────────────────────────────────────────────────────────────┐
│                       SYNC STATES                             │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│   SYNCED ──────▶ PENDING ──────▶ SYNCING ──────▶ SYNCED     │
│      │              │               │               │         │
│      │              │               │               │         │
│      │              ▼               ▼               │         │
│      │          CONFLICT ◀──── FAILED              │         │
│      │              │               │               │         │
│      │              ▼               ▼               │         │
│      │         RESOLVED ────▶ RETRY ───────────────┘         │
│      │                                                        │
│      └────────────────────────────────────────────────────────│
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### 4. Data Flow Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT                                   │
│                                                                  │
│   ┌──────────┐    ┌──────────────┐    ┌──────────────────┐     │
│   │    UI    │◀──▶│  Sync Layer  │◀──▶│   Local Store    │     │
│   │ (React)  │    │  (Manager)   │    │   (IndexedDB)    │     │
│   └──────────┘    └──────┬───────┘    └──────────────────┘     │
│                          │                                      │
│                          ▼                                      │
│                   ┌──────────────┐                              │
│                   │   Network    │                              │
│                   │   Monitor    │                              │
│                   └──────┬───────┘                              │
│                          │                                      │
│         ┌────────────────┼────────────────┐                    │
│         │                │                │                     │
│         ▼                ▼                ▼                     │
│   ┌──────────┐    ┌──────────┐    ┌──────────────┐            │
│   │ REST API │    │WebSocket │    │Service Worker│            │
│   └──────────┘    └──────────┘    │(Background)  │            │
│                                   └──────────────┘            │
│                                                                │
└────────────────────────────┬───────────────────────────────────┘
                             │
                             ▼
                      ┌──────────────┐
                      │    SERVER    │
                      │   DATABASE   │
                      └──────────────┘
```

---

## Technology Stack & Packages

### Storage Solutions

#### 1. IndexedDB (Browser Native)

The most powerful browser storage - async, large capacity, structured data.

```javascript
// Raw IndexedDB (complex)
const request = indexedDB.open('myDB', 1);
request.onupgradeneeded = (e) => {
  const db = e.target.result;
  db.createObjectStore('notes', { keyPath: 'id' });
};
```

#### 2. LocalForage (Recommended)

Simplified IndexedDB with localStorage fallback.

```bash
npm install localforage
```

```javascript
import localforage from 'localforage';

// Configure
localforage.config({
  name: 'MyApp',
  storeName: 'app_store',
  description: 'App offline storage'
});

// Usage (simple async/await)
await localforage.setItem('user', { name: 'John' });
const user = await localforage.getItem('user');
await localforage.removeItem('user');
await localforage.clear();
```

#### 3. Dexie.js (Advanced IndexedDB)

Type-safe IndexedDB wrapper with powerful querying.

```bash
npm install dexie
```

```javascript
import Dexie from 'dexie';

// Define schema
const db = new Dexie('MyApp');
db.version(1).stores({
  notes: '++id, title, createdAt, syncStatus',
  pendingActions: '++id, type, timestamp'
});

// Typed queries
const unsynced = await db.notes
  .where('syncStatus')
  .equals('pending')
  .toArray();
```

#### 4. RxDB (Reactive Database)

Realtime database with sync built-in.

```bash
npm install rxdb rxjs
```

```javascript
import { createRxDatabase } from 'rxdb';
import { getRxStorageDexie } from 'rxdb/plugins/storage-dexie';

const db = await createRxDatabase({
  name: 'mydb',
  storage: getRxStorageDexie()
});

// Reactive queries
db.notes.find().$.subscribe(notes => {
  console.log('Notes updated:', notes);
});
```

#### 5. PouchDB + CouchDB (Full Sync Solution)

Automatic bi-directional sync with CouchDB.

```bash
npm install pouchdb pouchdb-find
```

```javascript
import PouchDB from 'pouchdb';

const localDB = new PouchDB('myapp');
const remoteDB = new PouchDB('https://server.com/myapp');

// Automatic sync!
localDB.sync(remoteDB, {
  live: true,
  retry: true
}).on('change', (info) => {
  console.log('Sync change:', info);
}).on('error', (err) => {
  console.log('Sync error:', err);
});
```

### Comparison Table

| Package | Size | Complexity | Best For | Sync Built-in |
|---------|------|------------|----------|---------------|
| **localStorage** | 0kb | Very Low | Small data (<5MB) | No |
| **localForage** | 10kb | Low | General offline storage | No |
| **Dexie.js** | 25kb | Medium | Complex queries | No |
| **RxDB** | 45kb | High | Reactive apps | Optional |
| **PouchDB** | 46kb | Medium | CouchDB sync | Yes |
| **WatermelonDB** | 30kb | High | React Native | Optional |

### Service Worker Libraries

#### 1. Workbox (Google)

```bash
npm install workbox-webpack-plugin
```

```javascript
// sw.js with Workbox
import { precacheAndRoute } from 'workbox-precaching';
import { registerRoute } from 'workbox-routing';
import { StaleWhileRevalidate, CacheFirst } from 'workbox-strategies';

// Precache static assets
precacheAndRoute(self.__WB_MANIFEST);

// API caching strategy
registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  new StaleWhileRevalidate({
    cacheName: 'api-cache'
  })
);

// Image caching
registerRoute(
  ({ request }) => request.destination === 'image',
  new CacheFirst({
    cacheName: 'images',
    plugins: [
      new ExpirationPlugin({
        maxEntries: 50,
        maxAgeSeconds: 30 * 24 * 60 * 60 // 30 days
      })
    ]
  })
);
```

#### 2. Background Sync API

```javascript
// Register sync
async function queueSync(tag) {
  const registration = await navigator.serviceWorker.ready;
  await registration.sync.register(tag);
}

// In service worker
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-messages') {
    event.waitUntil(syncMessages());
  }
});
```

### State Management with Offline Support

#### TanStack Query (React Query)

```bash
npm install @tanstack/react-query
```

```javascript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Offline-aware query
function useNotes() {
  return useQuery({
    queryKey: ['notes'],
    queryFn: fetchNotes,
    staleTime: 5 * 60 * 1000,        // Fresh for 5 min
    gcTime: 24 * 60 * 60 * 1000,     // Keep 24 hours
    networkMode: 'offlineFirst',      // Use cache first
    retry: 3,
    retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 30000)
  });
}

// Optimistic mutation
function useCreateNote() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: createNote,
    onMutate: async (newNote) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['notes'] });
      
      // Snapshot previous value
      const previous = queryClient.getQueryData(['notes']);
      
      // Optimistically update
      queryClient.setQueryData(['notes'], (old) => [...old, newNote]);
      
      return { previous };
    },
    onError: (err, newNote, context) => {
      // Rollback on error
      queryClient.setQueryData(['notes'], context.previous);
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['notes'] });
    }
  });
}
```

#### Zustand with Persist

```bash
npm install zustand
```

```javascript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

const useStore = create(
  persist(
    (set, get) => ({
      notes: [],
      pendingActions: [],
      
      addNote: (note) => {
        set((state) => ({
          notes: [...state.notes, note],
          pendingActions: [...state.pendingActions, {
            type: 'CREATE_NOTE',
            payload: note,
            timestamp: Date.now()
          }]
        }));
      },
      
      syncComplete: (actionId) => {
        set((state) => ({
          pendingActions: state.pendingActions.filter(a => a.id !== actionId)
        }));
      }
    }),
    {
      name: 'app-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({
        notes: state.notes,
        pendingActions: state.pendingActions
      })
    }
  )
);
```

### Mobile (React Native)

#### MMKV (Fast Storage)

```bash
npm install react-native-mmkv
```

```javascript
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();

storage.set('user', JSON.stringify({ name: 'John' }));
const user = JSON.parse(storage.getString('user'));
```

#### WatermelonDB (SQLite)

```bash
npm install @nozbe/watermelondb
```

```javascript
import { Database } from '@nozbe/watermelondb';
import SQLiteAdapter from '@nozbe/watermelondb/adapters/sqlite';

const adapter = new SQLiteAdapter({
  schema,
  migrations,
  jsi: true
});

const database = new Database({
  adapter,
  modelClasses: [Note, Task]
});

// Sync with server
await database.sync({
  pullChanges: async ({ lastPulledAt }) => {
    const response = await api.pull(lastPulledAt);
    return response;
  },
  pushChanges: async ({ changes }) => {
    await api.push(changes);
  }
});
```

---

## Architecture Patterns

### Pattern 1: Simple Queue Pattern

Best for: Forms, CRUD apps, simple data entry.

```
┌──────────────┐
│   User       │
│   Action     │
└──────┬───────┘
       │
       ▼
┌──────────────┐     ┌──────────────┐
│  Is Online?  │────▶│ Send to API  │
└──────┬───────┘ YES └──────────────┘
       │ NO
       ▼
┌──────────────┐
│ Add to Queue │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Save Locally │
└──────┬───────┘
       │
       ▼ (when online)
┌──────────────┐
│Process Queue │
│  One by One  │
└──────────────┘
```

### Pattern 2: Event Sourcing

Best for: Complex apps, audit trails, undo/redo.

```
┌────────────────────────────────────────────────────────────────┐
│                      EVENT SOURCING                             │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Events (Immutable Log):                                      │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │ [1] NoteCreated { id: 1, text: "Hello" }                │  │
│   │ [2] NoteUpdated { id: 1, text: "Hello World" }          │  │
│   │ [3] NoteDeleted { id: 1 }                               │  │
│   └─────────────────────────────────────────────────────────┘  │
│                           │                                     │
│                           ▼                                     │
│   Current State = replay(events)                               │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │ Notes: []  (after applying all events)                  │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

```javascript
// Event store
class EventStore {
  events = [];
  
  append(event) {
    this.events.push({
      ...event,
      id: uuid(),
      timestamp: Date.now(),
      synced: false
    });
    this.saveToStorage();
  }
  
  getState() {
    return this.events.reduce(reducer, initialState);
  }
  
  getUnsyncedEvents() {
    return this.events.filter(e => !e.synced);
  }
  
  markSynced(eventIds) {
    this.events = this.events.map(e => 
      eventIds.includes(e.id) ? { ...e, synced: true } : e
    );
  }
}
```

### Pattern 3: CRDT (Conflict-free Replicated Data Types)

Best for: Real-time collaboration, no conflicts ever.

```javascript
// G-Counter CRDT (Grow-only counter)
class GCounter {
  constructor(nodeId) {
    this.nodeId = nodeId;
    this.counts = {};
  }
  
  increment() {
    this.counts[this.nodeId] = (this.counts[this.nodeId] || 0) + 1;
  }
  
  value() {
    return Object.values(this.counts).reduce((a, b) => a + b, 0);
  }
  
  merge(other) {
    for (const [node, count] of Object.entries(other.counts)) {
      this.counts[node] = Math.max(this.counts[node] || 0, count);
    }
  }
}

// Usage
const counter1 = new GCounter('node1');
const counter2 = new GCounter('node2');

counter1.increment(); // node1: 1
counter2.increment(); // node2: 1
counter2.increment(); // node2: 2

// Merge - no conflicts!
counter1.merge(counter2);
console.log(counter1.value()); // 3
```

### Pattern 4: Hybrid (Online-First with Offline Fallback)

Best for: Apps where fresh data is preferred but offline is supported.

```
┌──────────────────────────────────────────────────────────────┐
│                    HYBRID STRATEGY                            │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│   READ:                                                       │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐                 │
│   │ Network │───▶│ Success │───▶│ Update  │                 │
│   │ Request │    │         │    │ Cache   │                 │
│   └────┬────┘    └─────────┘    └─────────┘                 │
│        │                                                      │
│        │ Fail                                                 │
│        ▼                                                      │
│   ┌─────────┐                                                │
│   │  Use    │                                                │
│   │ Cache   │                                                │
│   └─────────┘                                                │
│                                                               │
│   WRITE:                                                      │
│   ┌─────────┐    ┌─────────┐    ┌─────────┐                 │
│   │ Save    │───▶│ Network │───▶│ Success │                 │
│   │ Local   │    │ Request │    │         │                 │
│   └─────────┘    └────┬────┘    └─────────┘                 │
│                       │                                       │
│                       │ Fail                                  │
│                       ▼                                       │
│                  ┌─────────┐                                 │
│                  │ Queue   │                                 │
│                  │ Action  │                                 │
│                  └─────────┘                                 │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Implementation Guide

### Step 1: Setup Storage Layer

```javascript
// lib/storage.ts
import localforage from 'localforage';

// Configure stores
export const dataStore = localforage.createInstance({
  name: 'MyApp',
  storeName: 'data'
});

export const queueStore = localforage.createInstance({
  name: 'MyApp',
  storeName: 'sync_queue'
});

export const metaStore = localforage.createInstance({
  name: 'MyApp',
  storeName: 'metadata'
});

// Storage helpers
export const storage = {
  // Data operations
  async get<T>(key: string): Promise<T | null> {
    return dataStore.getItem(key);
  },
  
  async set<T>(key: string, value: T): Promise<void> {
    await dataStore.setItem(key, {
      data: value,
      updatedAt: Date.now()
    });
  },
  
  async remove(key: string): Promise<void> {
    await dataStore.removeItem(key);
  },
  
  // Metadata
  async getLastSync(): Promise<number> {
    return (await metaStore.getItem('lastSync')) || 0;
  },
  
  async setLastSync(timestamp: number): Promise<void> {
    await metaStore.setItem('lastSync', timestamp);
  }
};
```

### Step 2: Create Action Queue

```javascript
// lib/sync-queue.ts
import { queueStore } from './storage';

export interface QueuedAction {
  id: string;
  type: string;
  entity: string;
  payload: any;
  timestamp: number;
  retryCount: number;
  maxRetries: number;
  priority: number;
}

export const syncQueue = {
  async add(action: Omit<QueuedAction, 'id' | 'timestamp' | 'retryCount'>): Promise<string> {
    const id = `${Date.now()}-${Math.random().toString(36).slice(2)}`;
    
    const queuedAction: QueuedAction = {
      ...action,
      id,
      timestamp: Date.now(),
      retryCount: 0,
      maxRetries: action.maxRetries ?? 5
    };
    
    const queue = await this.getAll();
    queue.push(queuedAction);
    
    // Sort by priority (higher first) then timestamp (older first)
    queue.sort((a, b) => {
      if (a.priority !== b.priority) return b.priority - a.priority;
      return a.timestamp - b.timestamp;
    });
    
    await queueStore.setItem('queue', queue);
    return id;
  },
  
  async getAll(): Promise<QueuedAction[]> {
    return (await queueStore.getItem('queue')) || [];
  },
  
  async remove(id: string): Promise<void> {
    const queue = await this.getAll();
    await queueStore.setItem('queue', queue.filter(a => a.id !== id));
  },
  
  async incrementRetry(id: string): Promise<boolean> {
    const queue = await this.getAll();
    const index = queue.findIndex(a => a.id === id);
    
    if (index === -1) return false;
    
    queue[index].retryCount++;
    
    if (queue[index].retryCount >= queue[index].maxRetries) {
      // Move to dead letter queue
      await this.moveToDead(queue[index]);
      queue.splice(index, 1);
    }
    
    await queueStore.setItem('queue', queue);
    return queue[index]?.retryCount < queue[index]?.maxRetries;
  },
  
  async moveToDead(action: QueuedAction): Promise<void> {
    const dead = (await queueStore.getItem('dead')) || [];
    dead.push({ ...action, failedAt: Date.now() });
    await queueStore.setItem('dead', dead);
  },
  
  async getDeadLetters(): Promise<QueuedAction[]> {
    return (await queueStore.getItem('dead')) || [];
  },
  
  async retryDeadLetter(id: string): Promise<void> {
    const dead = await this.getDeadLetters();
    const action = dead.find(a => a.id === id);
    
    if (action) {
      action.retryCount = 0;
      await this.add(action);
      await queueStore.setItem('dead', dead.filter(a => a.id !== id));
    }
  },
  
  async clear(): Promise<void> {
    await queueStore.setItem('queue', []);
  },
  
  async count(): Promise<number> {
    const queue = await this.getAll();
    return queue.length;
  }
};
```

### Step 3: Network Monitor

```javascript
// lib/network.ts
type NetworkCallback = (online: boolean) => void;

class NetworkMonitor {
  private listeners: Set<NetworkCallback> = new Set();
  private _online: boolean = navigator.onLine;
  
  constructor() {
    window.addEventListener('online', () => this.setOnline(true));
    window.addEventListener('offline', () => this.setOnline(false));
    
    // Periodic check (navigator.onLine can be unreliable)
    setInterval(() => this.checkConnection(), 30000);
  }
  
  private setOnline(online: boolean) {
    if (this._online !== online) {
      this._online = online;
      this.listeners.forEach(cb => cb(online));
    }
  }
  
  private async checkConnection() {
    try {
      const response = await fetch('/api/health', { 
        method: 'HEAD',
        cache: 'no-store'
      });
      this.setOnline(response.ok);
    } catch {
      this.setOnline(false);
    }
  }
  
  get online() {
    return this._online;
  }
  
  subscribe(callback: NetworkCallback): () => void {
    this.listeners.add(callback);
    return () => this.listeners.delete(callback);
  }
}

export const network = new NetworkMonitor();
```

### Step 4: Sync Engine

```javascript
// lib/sync-engine.ts
import { syncQueue, QueuedAction } from './sync-queue';
import { network } from './network';
import { storage } from './storage';
import { api } from './api';

interface SyncResult {
  success: boolean;
  synced: number;
  failed: number;
  errors: Array<{ id: string; error: string }>;
}

class SyncEngine {
  private syncing = false;
  private syncPromise: Promise<SyncResult> | null = null;
  
  constructor() {
    // Auto-sync when coming online
    network.subscribe((online) => {
      if (online) {
        this.sync();
      }
    });
    
    // Periodic sync when online
    setInterval(() => {
      if (network.online && !this.syncing) {
        this.sync();
      }
    }, 60000); // Every minute
  }
  
  async sync(): Promise<SyncResult> {
    // Prevent concurrent syncs
    if (this.syncPromise) {
      return this.syncPromise;
    }
    
    if (!network.online) {
      return { success: false, synced: 0, failed: 0, errors: [] };
    }
    
    this.syncing = true;
    this.syncPromise = this.performSync();
    
    try {
      return await this.syncPromise;
    } finally {
      this.syncing = false;
      this.syncPromise = null;
    }
  }
  
  private async performSync(): Promise<SyncResult> {
    const result: SyncResult = {
      success: true,
      synced: 0,
      failed: 0,
      errors: []
    };
    
    const queue = await syncQueue.getAll();
    
    for (const action of queue) {
      try {
        await this.processAction(action);
        await syncQueue.remove(action.id);
        result.synced++;
      } catch (error) {
        const shouldRetry = await syncQueue.incrementRetry(action.id);
        
        if (!shouldRetry) {
          result.failed++;
          result.errors.push({
            id: action.id,
            error: error.message
          });
        }
        
        // If network error, stop processing
        if (!network.online) {
          break;
        }
        
        // Exponential backoff for retries
        await this.delay(Math.min(1000 * Math.pow(2, action.retryCount), 30000));
      }
    }
    
    if (result.synced > 0) {
      await storage.setLastSync(Date.now());
    }
    
    result.success = result.failed === 0;
    return result;
  }
  
  private async processAction(action: QueuedAction): Promise<void> {
    const { type, entity, payload } = action;
    
    switch (type) {
      case 'CREATE':
        await api.post(`/${entity}`, payload);
        break;
      case 'UPDATE':
        await api.patch(`/${entity}/${payload.id}`, payload);
        break;
      case 'DELETE':
        await api.delete(`/${entity}/${payload.id}`);
        break;
      default:
        throw new Error(`Unknown action type: ${type}`);
    }
  }
  
  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  // Status
  get isSyncing() {
    return this.syncing;
  }
  
  async getPendingCount(): Promise<number> {
    return syncQueue.count();
  }
}

export const syncEngine = new SyncEngine();
```

### Step 5: React Hook

```javascript
// hooks/use-offline-sync.ts
import { useState, useEffect, useCallback } from 'react';
import { syncEngine } from '@/lib/sync-engine';
import { syncQueue } from '@/lib/sync-queue';
import { network } from '@/lib/network';
import { storage } from '@/lib/storage';
import { useQueryClient } from '@tanstack/react-query';
import toast from 'react-hot-toast';

export function useOfflineSync() {
  const [online, setOnline] = useState(network.online);
  const [syncing, setSyncing] = useState(false);
  const [pendingCount, setPendingCount] = useState(0);
  const [lastSync, setLastSync] = useState<number | null>(null);
  const queryClient = useQueryClient();
  
  // Monitor online status
  useEffect(() => {
    const unsubscribe = network.subscribe((isOnline) => {
      setOnline(isOnline);
      
      if (isOnline) {
        toast.success('Back online! Syncing...', { icon: '🌐' });
        sync();
      } else {
        toast('You are offline', { icon: '📴' });
      }
    });
    
    return unsubscribe;
  }, []);
  
  // Update pending count
  useEffect(() => {
    const updateCount = async () => {
      const count = await syncQueue.count();
      setPendingCount(count);
    };
    
    updateCount();
    const interval = setInterval(updateCount, 10000);
    
    return () => clearInterval(interval);
  }, []);
  
  // Load last sync time
  useEffect(() => {
    storage.getLastSync().then(setLastSync);
  }, []);
  
  // Manual sync
  const sync = useCallback(async () => {
    if (!online || syncing) return;
    
    setSyncing(true);
    
    try {
      const result = await syncEngine.sync();
      
      if (result.synced > 0) {
        toast.success(`Synced ${result.synced} changes`);
        setLastSync(Date.now());
        
        // Invalidate queries to refresh data
        queryClient.invalidateQueries();
      }
      
      if (result.failed > 0) {
        toast.error(`${result.failed} changes failed to sync`);
      }
    } finally {
      setSyncing(false);
      setPendingCount(await syncQueue.count());
    }
  }, [online, syncing, queryClient]);
  
  return {
    online,
    syncing,
    pendingCount,
    lastSync,
    sync
  };
}
```

### Step 6: Offline-Aware Mutations

```javascript
// hooks/use-offline-mutation.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { syncQueue } from '@/lib/sync-queue';
import { network } from '@/lib/network';
import { storage } from '@/lib/storage';
import toast from 'react-hot-toast';

interface OfflineMutationOptions<T, V> {
  entity: string;
  type: 'CREATE' | 'UPDATE' | 'DELETE';
  onlineFn: (variables: V) => Promise<T>;
  queryKey: string[];
  optimisticUpdate?: (old: T[], variables: V) => T[];
}

export function useOfflineMutation<T, V>(options: OfflineMutationOptions<T, V>) {
  const queryClient = useQueryClient();
  const { entity, type, onlineFn, queryKey, optimisticUpdate } = options;
  
  return useMutation({
    mutationFn: async (variables: V) => {
      if (network.online) {
        try {
          return await onlineFn(variables);
        } catch (error) {
          // Network error - queue it
          await syncQueue.add({
            type,
            entity,
            payload: variables,
            priority: 1,
            maxRetries: 5
          });
          
          toast('Saved offline', { icon: '📴' });
          return variables as unknown as T;
        }
      } else {
        // Offline - queue it
        await syncQueue.add({
          type,
          entity,
          payload: variables,
          priority: 1,
          maxRetries: 5
        });
        
        toast('Saved offline. Will sync when online.', { icon: '📴' });
        return variables as unknown as T;
      }
    },
    
    onMutate: async (variables) => {
      if (!optimisticUpdate) return;
      
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey });
      
      // Snapshot
      const previous = queryClient.getQueryData(queryKey);
      
      // Optimistic update
      queryClient.setQueryData(queryKey, (old: T[]) => 
        optimisticUpdate(old || [], variables)
      );
      
      return { previous };
    },
    
    onError: (error, variables, context) => {
      // Rollback on error
      if (context?.previous) {
        queryClient.setQueryData(queryKey, context.previous);
      }
    },
    
    onSettled: () => {
      // Refetch after mutation
      if (network.online) {
        queryClient.invalidateQueries({ queryKey });
      }
    }
  });
}

// Usage example
function useCreateNote() {
  return useOfflineMutation({
    entity: 'notes',
    type: 'CREATE',
    onlineFn: (note) => api.post('/notes', note),
    queryKey: ['notes'],
    optimisticUpdate: (old, newNote) => [...old, { ...newNote, id: `temp-${Date.now()}` }]
  });
}
```

---

## Sync Strategies

### 1. Network First (Online Preferred)

Best for: Data that changes frequently, real-time apps.

```javascript
async function fetchNetworkFirst(key, fetcher) {
  if (network.online) {
    try {
      const data = await fetcher();
      await storage.set(key, data);
      return data;
    } catch (error) {
      // Network error - fallback to cache
      const cached = await storage.get(key);
      if (cached) return cached;
      throw error;
    }
  } else {
    // Offline - use cache
    const cached = await storage.get(key);
    if (cached) return cached;
    throw new Error('No cached data available');
  }
}
```

### 2. Cache First (Offline Preferred)

Best for: Static content, rarely changing data.

```javascript
async function fetchCacheFirst(key, fetcher, maxAge = 3600000) {
  const cached = await storage.get(key);
  
  if (cached) {
    const age = Date.now() - cached.updatedAt;
    
    if (age < maxAge) {
      // Cache is fresh
      return cached.data;
    }
    
    // Cache is stale - refresh in background
    if (network.online) {
      refreshInBackground(key, fetcher);
    }
    
    return cached.data;
  }
  
  // No cache - must fetch
  if (!network.online) {
    throw new Error('No cached data available');
  }
  
  const data = await fetcher();
  await storage.set(key, data);
  return data;
}
```

### 3. Stale While Revalidate

Best for: Balance between freshness and performance.

```javascript
async function fetchSWR(key, fetcher) {
  // Return stale data immediately
  const cached = await storage.get(key);
  
  // Revalidate in background
  const revalidate = async () => {
    if (network.online) {
      try {
        const data = await fetcher();
        await storage.set(key, data);
        return data;
      } catch {
        return null;
      }
    }
  };
  
  if (cached) {
    // Return cached, revalidate in background
    revalidate().then(fresh => {
      if (fresh) {
        notifyUpdate(key, fresh);
      }
    });
    return cached.data;
  }
  
  // No cache - wait for fetch
  const data = await revalidate();
  if (data) return data;
  
  throw new Error('Unable to fetch data');
}
```

### 4. Background Sync

Best for: Non-urgent data, large uploads.

```javascript
// Register for background sync
async function scheduleBackgroundSync(tag) {
  if ('serviceWorker' in navigator && 'sync' in ServiceWorkerRegistration.prototype) {
    const registration = await navigator.serviceWorker.ready;
    await registration.sync.register(tag);
  } else {
    // Fallback - sync immediately
    await syncEngine.sync();
  }
}

// Service worker handler
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-pending') {
    event.waitUntil(syncPendingActions());
  }
});
```

### 5. Periodic Background Sync

Best for: News feeds, weather updates.

```javascript
// Register periodic sync (requires permission)
async function registerPeriodicSync() {
  const registration = await navigator.serviceWorker.ready;
  
  try {
    await registration.periodicSync.register('content-sync', {
      minInterval: 24 * 60 * 60 * 1000 // Daily
    });
  } catch {
    console.log('Periodic sync not supported');
  }
}

// Service worker handler
self.addEventListener('periodicsync', (event) => {
  if (event.tag === 'content-sync') {
    event.waitUntil(refreshContent());
  }
});
```

### 6. Backpressure (Don't Overwhelm the Server)

> **"It's better to sync slowly than crash the server."**

When a user has been offline for a long time (days/weeks), they might have hundreds or thousands of pending actions. Syncing all at once will:
- Overwhelm the server
- Cause timeouts
- Get rate-limited
- Potentially lose data

**Solution: Apply backpressure - sync in controlled batches with delays.**

```javascript
class BackpressureSyncManager {
  constructor() {
    this.batchSize = 10;        // Process 10 actions at a time
    this.delayBetweenBatches = 500;  // 500ms pause between batches
    this.maxConcurrent = 3;     // Max 3 requests in parallel
    this.isSyncing = false;
  }

  async syncAllPending() {
    if (this.isSyncing) return;
    this.isSyncing = true;

    const pending = await this.getPendingActions();
    const total = pending.length;
    let synced = 0;
    let failed = [];

    console.log(`Starting sync of ${total} pending actions with backpressure`);

    // Process in batches
    for (let i = 0; i < pending.length; i += this.batchSize) {
      const batch = pending.slice(i, i + this.batchSize);
      
      // Process batch with limited concurrency
      const results = await this.processBatchWithConcurrency(batch);
      
      // Track results
      results.forEach((result, idx) => {
        if (result.success) {
          synced++;
        } else {
          failed.push({ action: batch[idx], error: result.error });
        }
      });

      // Update progress
      this.onProgress?.({
        total,
        synced,
        failed: failed.length,
        remaining: total - synced - failed.length
      });

      // BACKPRESSURE: Pause between batches
      if (i + this.batchSize < pending.length) {
        console.log(`Batch complete. Pausing ${this.delayBetweenBatches}ms...`);
        await this.sleep(this.delayBetweenBatches);
      }

      // Check if we should stop (e.g., went offline)
      if (!navigator.onLine) {
        console.log('Went offline, pausing sync');
        break;
      }
    }

    this.isSyncing = false;
    return { synced, failed };
  }

  // Process batch with concurrency limit
  async processBatchWithConcurrency(batch) {
    const results = [];
    const executing = [];

    for (const action of batch) {
      const promise = this.processAction(action)
        .then(result => ({ success: true, result }))
        .catch(error => ({ success: false, error }));

      results.push(promise);
      executing.push(promise);

      // Limit concurrent requests
      if (executing.length >= this.maxConcurrent) {
        await Promise.race(executing);
        executing.splice(executing.findIndex(p => p === promise), 1);
      }
    }

    return Promise.all(results);
  }

  async processAction(action) {
    // Add exponential backoff for retries
    let attempt = 0;
    const maxAttempts = 3;

    while (attempt < maxAttempts) {
      try {
        const response = await fetch('/api/sync', {
          method: 'POST',
          body: JSON.stringify(action),
          headers: { 'Content-Type': 'application/json' }
        });

        if (response.status === 429) {
          // RATE LIMITED: Increase delay, wait longer
          console.log('Rate limited! Slowing down...');
          this.delayBetweenBatches = Math.min(
            this.delayBetweenBatches * 2, 
            5000  // Max 5 seconds
          );
          await this.sleep(this.delayBetweenBatches);
          attempt++;
          continue;
        }

        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        
        // Success - remove from queue
        await this.removeFromQueue(action.id);
        return response.json();

      } catch (error) {
        attempt++;
        if (attempt >= maxAttempts) throw error;
        
        // Exponential backoff
        await this.sleep(Math.pow(2, attempt) * 1000);
      }
    }
  }

  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Usage with progress UI
const syncManager = new BackpressureSyncManager();
syncManager.onProgress = (progress) => {
  updateUI(`Syncing: ${progress.synced}/${progress.total} (${progress.failed} failed)`);
};

await syncManager.syncAllPending();
```

### Backpressure Quick Reference

| Strategy | What It Does | When To Use |
|----------|--------------|-------------|
| **Batch Processing** | Sync N actions at a time | Always |
| **Delay Between Batches** | Pause 500ms-2s between batches | High volume |
| **Concurrency Limit** | Max 3 parallel requests | Prevent server overload |
| **Rate Limit Detection** | Check for 429, slow down | API has rate limits |
| **Progress UI** | Show "Syncing 50/200..." | Long syncs |
| **Pausable Sync** | Stop if offline | Network changes |

### Key Insight (Memorize This!)
> "A user offline for 1 month with 500 pending actions: Don't sync all at once! Process 10 at a time with 500ms delays. Show progress. Handle 429 rate limits by slowing down. Better to take 2 minutes than to crash or lose data."

---

## Conflict Resolution

### Understanding Conflicts

```
Timeline:
─────────────────────────────────────────────────────────────────

T1: User A reads data   { name: "Original" }
    User B reads data   { name: "Original" }

T2: User A goes offline

T3: User A edits        { name: "Alice's Version" }  (offline)
    User B edits        { name: "Bob's Version" }    (online)
    Server saves        { name: "Bob's Version" }

T4: User A comes online
    User A tries to sync { name: "Alice's Version" }
    
    CONFLICT! 💥
    - Server has: "Bob's Version"
    - Client has: "Alice's Version"
    
─────────────────────────────────────────────────────────────────
```

### Strategy 1: Last Write Wins (LWW)

Simple but may lose data.

```javascript
// Include timestamp with every update
interface VersionedData {
  id: string;
  data: any;
  updatedAt: number;
  updatedBy: string;
}

// Server-side
async function saveWithLWW(incoming: VersionedData) {
  const existing = await db.get(incoming.id);
  
  if (!existing || incoming.updatedAt > existing.updatedAt) {
    await db.save(incoming);
    return { saved: true, data: incoming };
  }
  
  return { saved: false, data: existing, conflict: true };
}

// Client-side handling
async function handleLWWResponse(response, localData) {
  if (response.conflict) {
    // Server has newer data - accept it
    await storage.set(localData.id, response.data);
    notifyUser('Your changes were overwritten by a newer version');
  }
}
```

### Strategy 2: Version Vectors

Track changes per device/user.

```javascript
interface VersionVector {
  [nodeId: string]: number;
}

interface VersionedData {
  id: string;
  data: any;
  version: VersionVector;
}

function compareVersions(a: VersionVector, b: VersionVector): 'before' | 'after' | 'concurrent' {
  let aBefore = false;
  let aAfter = false;
  
  const allNodes = new Set([...Object.keys(a), ...Object.keys(b)]);
  
  for (const node of allNodes) {
    const va = a[node] || 0;
    const vb = b[node] || 0;
    
    if (va < vb) aBefore = true;
    if (va > vb) aAfter = true;
  }
  
  if (aBefore && !aAfter) return 'before';
  if (aAfter && !aBefore) return 'after';
  return 'concurrent'; // Conflict!
}

function mergeVersions(a: VersionVector, b: VersionVector): VersionVector {
  const merged: VersionVector = {};
  const allNodes = new Set([...Object.keys(a), ...Object.keys(b)]);
  
  for (const node of allNodes) {
    merged[node] = Math.max(a[node] || 0, b[node] || 0);
  }
  
  return merged;
}
```

### Strategy 3: Operational Transform (OT)

Best for: Text editing, real-time collaboration.

```javascript
// Simple OT for text
interface TextOperation {
  type: 'insert' | 'delete';
  position: number;
  text?: string;      // for insert
  length?: number;    // for delete
}

function transformOperation(op: TextOperation, against: TextOperation): TextOperation {
  const transformed = { ...op };
  
  if (against.type === 'insert') {
    if (op.position >= against.position) {
      transformed.position += against.text!.length;
    }
  } else if (against.type === 'delete') {
    if (op.position > against.position) {
      transformed.position -= Math.min(
        against.length!,
        op.position - against.position
      );
    }
  }
  
  return transformed;
}
```

### Strategy 4: Three-Way Merge

Best for: Structured data, form fields.

```javascript
interface MergeResult<T> {
  merged: T;
  conflicts: string[];
  autoResolved: string[];
}

function threeWayMerge<T extends object>(
  base: T,      // Original version
  local: T,     // Local changes
  remote: T     // Server changes
): MergeResult<T> {
  const merged: any = { ...base };
  const conflicts: string[] = [];
  const autoResolved: string[] = [];
  
  const allKeys = new Set([
    ...Object.keys(base),
    ...Object.keys(local),
    ...Object.keys(remote)
  ]);
  
  for (const key of allKeys) {
    const baseVal = (base as any)[key];
    const localVal = (local as any)[key];
    const remoteVal = (remote as any)[key];
    
    const localChanged = !deepEqual(baseVal, localVal);
    const remoteChanged = !deepEqual(baseVal, remoteVal);
    
    if (!localChanged && !remoteChanged) {
      merged[key] = baseVal;
    } else if (localChanged && !remoteChanged) {
      merged[key] = localVal;
      autoResolved.push(key);
    } else if (!localChanged && remoteChanged) {
      merged[key] = remoteVal;
      autoResolved.push(key);
    } else if (deepEqual(localVal, remoteVal)) {
      merged[key] = localVal;
      autoResolved.push(key);
    } else {
      // True conflict
      conflicts.push(key);
      merged[key] = remoteVal; // Default to remote
    }
  }
  
  return { merged, conflicts, autoResolved };
}
```

### Strategy 5: User-Assisted Resolution

Best for: Important data where loss is unacceptable.

```javascript
// Conflict resolution UI
interface ConflictResolution {
  showConflictDialog: (local: any, remote: any) => Promise<'local' | 'remote' | 'merge'>;
  showMergeEditor: (local: any, remote: any) => Promise<any>;
}

async function resolveConflict(local: any, remote: any, resolver: ConflictResolution) {
  const choice = await resolver.showConflictDialog(local, remote);
  
  switch (choice) {
    case 'local':
      return { data: local, forceOverwrite: true };
    case 'remote':
      return { data: remote, discardLocal: true };
    case 'merge':
      const merged = await resolver.showMergeEditor(local, remote);
      return { data: merged };
  }
}

// React component example
function ConflictDialog({ local, remote, onResolve }) {
  return (
    <Dialog>
      <h2>Conflict Detected</h2>
      <p>This item was modified on another device.</p>
      
      <div className="comparison">
        <div className="local">
          <h3>Your Version</h3>
          <pre>{JSON.stringify(local, null, 2)}</pre>
        </div>
        <div className="remote">
          <h3>Server Version</h3>
          <pre>{JSON.stringify(remote, null, 2)}</pre>
        </div>
      </div>
      
      <div className="actions">
        <Button onClick={() => onResolve('local')}>
          Keep My Version
        </Button>
        <Button onClick={() => onResolve('remote')}>
          Use Server Version
        </Button>
        <Button onClick={() => onResolve('merge')}>
          Merge Changes
        </Button>
      </div>
    </Dialog>
  );
}
```

---

## Scenario-Based Solutions

### Scenario 1: CareGiving App (Your Project)

**Challenge:** Caregivers may have poor connectivity at hospitals, care homes.

**Critical Requirements:**
- Emergency info ALWAYS accessible
- Medication logs can't be lost
- Timeline entries should sync eventually

```javascript
// Priority-based offline handling
const PRIORITIES = {
  EMERGENCY_INFO: 100,    // Cache aggressively, never expire
  MEDICATIONS: 90,        // Critical, short expiry
  TIMELINE_ENTRY: 50,     // Queue for sync
  SETTINGS: 10            // Low priority
};

// Emergency info - cache on every load
async function cacheEmergencyInfo(careRecipientId: string) {
  const info = await api.get(`/emergency/${careRecipientId}`);
  
  await storage.set(`emergency:${careRecipientId}`, {
    data: info,
    cachedAt: Date.now(),
    neverExpire: true  // Always keep
  });
}

// Medication logging - optimistic with queue
async function logMedication(medicationId: string, status: 'given' | 'skipped') {
  const log = {
    medicationId,
    status,
    timestamp: Date.now(),
    localId: uuid()
  };
  
  // Save locally immediately
  await storage.append('medication_logs', log);
  
  // Queue for sync
  await syncQueue.add({
    type: 'CREATE',
    entity: 'medication_logs',
    payload: log,
    priority: PRIORITIES.MEDICATIONS,
    maxRetries: 10  // More retries for critical data
  });
  
  return log;
}
```

### Scenario 2: E-Commerce App

**Challenge:** Cart and orders must never be lost, inventory needs real-time accuracy.

**Critical Requirements:**
- Cart persists across sessions
- Orders queue when offline
- Inventory checks before checkout
- Payment requires online

```javascript
// Cart - fully offline capable
class OfflineCart {
  private cartKey = 'shopping_cart';
  
  async getCart(): Promise<CartItem[]> {
    return (await storage.get(this.cartKey)) || [];
  }
  
  async addItem(item: CartItem): Promise<void> {
    const cart = await this.getCart();
    
    const existing = cart.find(i => i.productId === item.productId);
    if (existing) {
      existing.quantity += item.quantity;
    } else {
      cart.push(item);
    }
    
    await storage.set(this.cartKey, cart);
    
    // Sync to server if online (for cross-device access)
    if (network.online) {
      this.syncCartToServer(cart);
    }
  }
  
  async checkout(): Promise<CheckoutResult> {
    if (!network.online) {
      throw new Error('Internet connection required for checkout');
    }
    
    const cart = await this.getCart();
    
    // Verify inventory before proceeding
    const inventoryCheck = await api.post('/inventory/check', {
      items: cart.map(i => ({ productId: i.productId, quantity: i.quantity }))
    });
    
    if (!inventoryCheck.allAvailable) {
      return {
        success: false,
        unavailable: inventoryCheck.unavailableItems
      };
    }
    
    // Create order
    const order = await api.post('/orders', { items: cart });
    
    // Clear cart on success
    await storage.set(this.cartKey, []);
    
    return { success: true, order };
  }
}

// Wishlist - fully offline
async function addToWishlist(productId: string) {
  await storage.append('wishlist', {
    productId,
    addedAt: Date.now()
  });
  
  await syncQueue.add({
    type: 'CREATE',
    entity: 'wishlist',
    payload: { productId },
    priority: 10  // Low priority
  });
}

// Product browsing - cache heavily
async function getProduct(productId: string) {
  return fetchCacheFirst(
    `product:${productId}`,
    () => api.get(`/products/${productId}`),
    24 * 60 * 60 * 1000  // 24 hour cache
  );
}
```

### Scenario 3: Chat Application

**Challenge:** Messages must be delivered reliably, order matters.

**Critical Requirements:**
- Messages never lost
- Correct ordering
- Read receipts
- Presence awareness

```javascript
// Message with local state
interface Message {
  id: string;
  localId: string;          // Client-generated ID
  chatId: string;
  content: string;
  timestamp: number;
  status: 'sending' | 'sent' | 'delivered' | 'read' | 'failed';
}

class OfflineChat {
  private messageQueue: Message[] = [];
  
  async sendMessage(chatId: string, content: string): Promise<Message> {
    const message: Message = {
      id: '',                  // Server will assign
      localId: uuid(),
      chatId,
      content,
      timestamp: Date.now(),
      status: 'sending'
    };
    
    // Save locally immediately
    await this.saveMessageLocally(message);
    
    // Add to queue
    await syncQueue.add({
      type: 'CREATE',
      entity: 'messages',
      payload: message,
      priority: 80,
      maxRetries: 20  // Persistent retry
    });
    
    // Try to send immediately
    if (network.online) {
      this.trySendMessage(message);
    }
    
    return message;
  }
  
  private async trySendMessage(message: Message): Promise<void> {
    try {
      const sent = await api.post('/messages', message);
      
      // Update with server ID and status
      await this.updateMessageStatus(message.localId, {
        id: sent.id,
        status: 'sent'
      });
      
      // Remove from queue
      await syncQueue.remove(message.localId);
    } catch (error) {
      await this.updateMessageStatus(message.localId, {
        status: 'failed'
      });
    }
  }
  
  // Fetch messages with offline support
  async getMessages(chatId: string, options: { limit: number; before?: string }) {
    // Get local messages first
    const local = await storage.get(`messages:${chatId}`) || [];
    
    if (network.online) {
      try {
        // Fetch from server
        const remote = await api.get(`/chats/${chatId}/messages`, options);
        
        // Merge local pending messages with server messages
        const merged = this.mergeMessages(local, remote);
        await storage.set(`messages:${chatId}`, merged);
        
        return merged;
      } catch {
        return local;
      }
    }
    
    return local;
  }
  
  private mergeMessages(local: Message[], remote: Message[]): Message[] {
    const merged = [...remote];
    
    // Add local-only messages (pending/failed)
    for (const localMsg of local) {
      if (localMsg.status === 'sending' || localMsg.status === 'failed') {
        // Check if not already in remote (matched by localId)
        const inRemote = remote.some(r => r.localId === localMsg.localId);
        if (!inRemote) {
          merged.push(localMsg);
        }
      }
    }
    
    // Sort by timestamp
    merged.sort((a, b) => a.timestamp - b.timestamp);
    
    return merged;
  }
}
```

### Scenario 4: Note-Taking / Document App

**Challenge:** Long-form content, potential for complex conflicts.

**Critical Requirements:**
- Never lose user's work
- Handle concurrent edits
- Version history
- Large content support

```javascript
// Auto-save with debounce
class OfflineDocument {
  private saveTimeout: NodeJS.Timeout | null = null;
  private pendingChanges: Map<string, any> = new Map();
  
  async saveDocument(docId: string, content: string, cursor?: number) {
    // Save to local immediately
    const doc = {
      id: docId,
      content,
      cursor,
      localVersion: Date.now(),
      syncedVersion: null
    };
    
    await storage.set(`doc:${docId}`, doc);
    
    // Debounce sync
    this.pendingChanges.set(docId, doc);
    
    if (this.saveTimeout) {
      clearTimeout(this.saveTimeout);
    }
    
    this.saveTimeout = setTimeout(() => {
      this.syncPendingDocuments();
    }, 2000);  // 2 second debounce
  }
  
  private async syncPendingDocuments() {
    if (!network.online) return;
    
    for (const [docId, doc] of this.pendingChanges) {
      try {
        const response = await api.patch(`/documents/${docId}`, {
          content: doc.content,
          baseVersion: doc.syncedVersion
        });
        
        if (response.conflict) {
          await this.handleConflict(docId, doc, response.serverVersion);
        } else {
          await storage.set(`doc:${docId}`, {
            ...doc,
            syncedVersion: response.version
          });
        }
        
        this.pendingChanges.delete(docId);
      } catch (error) {
        // Will retry on next save or online event
        console.error('Failed to sync document:', docId);
      }
    }
  }
  
  private async handleConflict(docId: string, local: any, server: any) {
    // Use diff-match-patch for text merge
    const dmp = new diff_match_patch();
    
    const patches = dmp.patch_make(server.content, local.content);
    const [merged, results] = dmp.patch_apply(patches, server.content);
    
    const hasConflicts = results.some(r => !r);
    
    if (hasConflicts) {
      // Save both versions, let user resolve
      await storage.set(`doc:${docId}:conflict`, {
        local: local.content,
        server: server.content,
        attempted: merged
      });
      
      notifyUser('Document conflict detected. Please review.');
    } else {
      // Auto-merged successfully
      await this.saveDocument(docId, merged);
    }
  }
}
```

### Scenario 5: Task Management / To-Do App

**Challenge:** Simple data but important ordering and completion tracking.

```javascript
// Task with offline support
interface Task {
  id: string;
  localId: string;
  title: string;
  completed: boolean;
  order: number;
  createdAt: number;
  updatedAt: number;
}

class OfflineTasks {
  async createTask(title: string): Promise<Task> {
    const tasks = await this.getTasks();
    
    const task: Task = {
      id: '',
      localId: uuid(),
      title,
      completed: false,
      order: tasks.length,
      createdAt: Date.now(),
      updatedAt: Date.now()
    };
    
    tasks.push(task);
    await storage.set('tasks', tasks);
    
    await syncQueue.add({
      type: 'CREATE',
      entity: 'tasks',
      payload: task,
      priority: 50
    });
    
    return task;
  }
  
  async toggleComplete(taskId: string): Promise<void> {
    const tasks = await this.getTasks();
    const task = tasks.find(t => t.id === taskId || t.localId === taskId);
    
    if (!task) return;
    
    task.completed = !task.completed;
    task.updatedAt = Date.now();
    
    await storage.set('tasks', tasks);
    
    await syncQueue.add({
      type: 'UPDATE',
      entity: 'tasks',
      payload: { id: task.id || task.localId, completed: task.completed },
      priority: 50
    });
  }
  
  async reorderTasks(taskIds: string[]): Promise<void> {
    const tasks = await this.getTasks();
    
    // Update order based on new array order
    taskIds.forEach((id, index) => {
      const task = tasks.find(t => t.id === id || t.localId === id);
      if (task) task.order = index;
    });
    
    // Sort by order
    tasks.sort((a, b) => a.order - b.order);
    
    await storage.set('tasks', tasks);
    
    // Batch update for order
    await syncQueue.add({
      type: 'UPDATE',
      entity: 'tasks/reorder',
      payload: { order: taskIds },
      priority: 30  // Lower priority for reorder
    });
  }
  
  async getTasks(): Promise<Task[]> {
    return (await storage.get('tasks')) || [];
  }
}
```

### Scenario 6: Form / Survey App

**Challenge:** Long forms, partial saves, validation.

```javascript
// Form with auto-save
class OfflineForm {
  private formId: string;
  private autoSaveInterval: NodeJS.Timeout | null = null;
  
  constructor(formId: string) {
    this.formId = formId;
    this.startAutoSave();
  }
  
  private startAutoSave() {
    this.autoSaveInterval = setInterval(() => {
      this.saveDraft();
    }, 30000);  // Every 30 seconds
  }
  
  async saveDraft(): Promise<void> {
    const formData = this.collectFormData();
    
    await storage.set(`form_draft:${this.formId}`, {
      data: formData,
      savedAt: Date.now(),
      submitted: false
    });
  }
  
  async loadDraft(): Promise<any | null> {
    const draft = await storage.get(`form_draft:${this.formId}`);
    return draft?.submitted ? null : draft?.data;
  }
  
  async submit(): Promise<SubmitResult> {
    const formData = this.collectFormData();
    
    // Validate
    const validation = this.validate(formData);
    if (!validation.valid) {
      return { success: false, errors: validation.errors };
    }
    
    // Save complete draft
    await storage.set(`form_draft:${this.formId}`, {
      data: formData,
      savedAt: Date.now(),
      submitted: false
    });
    
    if (network.online) {
      try {
        const result = await api.post('/forms/submit', {
          formId: this.formId,
          data: formData
        });
        
        // Mark as submitted
        await storage.set(`form_draft:${this.formId}`, {
          data: formData,
          savedAt: Date.now(),
          submitted: true,
          submissionId: result.id
        });
        
        return { success: true, submissionId: result.id };
      } catch {
        // Queue for later
        await this.queueSubmission(formData);
        return { success: true, queued: true };
      }
    } else {
      await this.queueSubmission(formData);
      return { success: true, queued: true };
    }
  }
  
  private async queueSubmission(formData: any) {
    await syncQueue.add({
      type: 'CREATE',
      entity: 'form_submissions',
      payload: {
        formId: this.formId,
        data: formData,
        submittedAt: Date.now()
      },
      priority: 70,
      maxRetries: 10
    });
  }
  
  destroy() {
    if (this.autoSaveInterval) {
      clearInterval(this.autoSaveInterval);
    }
  }
}
```

---

## Edge Cases & Solutions

### Edge Case 1: Offline for Extended Periods (Days/Weeks/Months)

**Problem:** User offline for a month, lots of local changes, server data has changed significantly.

**Solutions:**

```javascript
// 1. Track offline duration
async function checkOfflineDuration() {
  const lastOnline = await storage.get('lastOnline');
  const duration = Date.now() - lastOnline;
  
  const ONE_DAY = 24 * 60 * 60 * 1000;
  const ONE_WEEK = 7 * ONE_DAY;
  const ONE_MONTH = 30 * ONE_DAY;
  
  if (duration > ONE_MONTH) {
    return 'extended';  // Full resync needed
  } else if (duration > ONE_WEEK) {
    return 'long';      // Careful sync needed
  } else if (duration > ONE_DAY) {
    return 'medium';    // Normal sync
  }
  return 'short';       // Quick sync
}

// 2. Graduated sync strategy
async function syncAfterOffline() {
  const duration = await checkOfflineDuration();
  
  switch (duration) {
    case 'extended':
      // Show warning to user
      const proceed = await showExtendedOfflineWarning();
      if (!proceed) return;
      
      // Full resync - may lose some local changes
      await fullResync();
      break;
      
    case 'long':
      // Sync carefully, check each item
      await carefulSync();
      break;
      
    default:
      // Normal sync
      await syncEngine.sync();
  }
}

// 3. Full resync with user confirmation
async function fullResync() {
  // Get all local pending changes
  const pending = await syncQueue.getAll();
  
  if (pending.length > 0) {
    // Export local changes for safety
    const backup = await exportLocalChanges(pending);
    await downloadBackup(backup);
    
    // Notify user
    notifyUser(`${pending.length} local changes backed up. Starting fresh sync.`);
  }
  
  // Clear local data
  await storage.clear();
  await syncQueue.clear();
  
  // Fetch fresh data from server
  await fetchAllData();
}

// 4. Careful sync with conflict review
async function carefulSync() {
  const pending = await syncQueue.getAll();
  const conflicts: Array<{ local: any; server: any }> = [];
  
  for (const action of pending) {
    const serverVersion = await fetchServerVersion(action.entity, action.payload.id);
    
    if (serverVersion && serverVersion.updatedAt > action.timestamp) {
      // Potential conflict
      conflicts.push({
        local: action,
        server: serverVersion
      });
    } else {
      // Safe to sync
      await processAction(action);
    }
  }
  
  if (conflicts.length > 0) {
    await showConflictReviewUI(conflicts);
  }
}
```

### Edge Case 2: Storage Quota Exceeded

**Problem:** IndexedDB has limits, especially on mobile.

```javascript
// 1. Monitor storage usage
async function checkStorageQuota() {
  if ('storage' in navigator && 'estimate' in navigator.storage) {
    const estimate = await navigator.storage.estimate();
    const percentUsed = (estimate.usage! / estimate.quota!) * 100;
    
    return {
      used: estimate.usage,
      quota: estimate.quota,
      percentUsed,
      isLow: percentUsed > 80
    };
  }
  return null;
}

// 2. Cleanup old data when needed
async function cleanupStorage() {
  const quota = await checkStorageQuota();
  
  if (!quota || !quota.isLow) return;
  
  console.log(`Storage ${quota.percentUsed.toFixed(1)}% full, cleaning up...`);
  
  // Priority-based cleanup
  const cleanupOrder = [
    { store: 'cache', maxAge: 7 * 24 * 60 * 60 * 1000 },      // 7 day cache
    { store: 'images', maxAge: 30 * 24 * 60 * 60 * 1000 },    // 30 day images
    { store: 'old_logs', maxAge: 24 * 60 * 60 * 1000 },       // 1 day logs
  ];
  
  for (const { store, maxAge } of cleanupOrder) {
    await cleanupOldItems(store, maxAge);
    
    const newQuota = await checkStorageQuota();
    if (newQuota && !newQuota.isLow) {
      break;  // Enough space now
    }
  }
}

// 3. Persist important data
async function requestPersistentStorage() {
  if ('storage' in navigator && 'persist' in navigator.storage) {
    const isPersisted = await navigator.storage.persisted();
    
    if (!isPersisted) {
      const granted = await navigator.storage.persist();
      console.log('Persistent storage:', granted ? 'granted' : 'denied');
    }
  }
}
```

### Edge Case 3: Multiple Tabs/Windows

**Problem:** Multiple tabs can cause sync conflicts and data races.

```javascript
// 1. Use BroadcastChannel for tab coordination
const channel = new BroadcastChannel('app_sync');

// Leader election - only one tab syncs
class TabCoordinator {
  private isLeader = false;
  private leaderId: string | null = null;
  private myId = uuid();
  
  constructor() {
    channel.addEventListener('message', this.handleMessage.bind(this));
    this.electLeader();
  }
  
  private electLeader() {
    channel.postMessage({ type: 'LEADER_PING', id: this.myId });
    
    setTimeout(() => {
      if (!this.leaderId) {
        // No leader responded, become leader
        this.isLeader = true;
        channel.postMessage({ type: 'LEADER_ANNOUNCE', id: this.myId });
      }
    }, 100);
  }
  
  private handleMessage(event: MessageEvent) {
    const { type, id } = event.data;
    
    switch (type) {
      case 'LEADER_PING':
        if (this.isLeader) {
          channel.postMessage({ type: 'LEADER_ANNOUNCE', id: this.myId });
        }
        break;
        
      case 'LEADER_ANNOUNCE':
        this.leaderId = id;
        if (id !== this.myId) {
          this.isLeader = false;
        }
        break;
        
      case 'DATA_UPDATED':
        // Refresh local cache from storage
        this.refreshCache();
        break;
    }
  }
  
  canSync(): boolean {
    return this.isLeader;
  }
  
  notifyDataUpdate() {
    channel.postMessage({ type: 'DATA_UPDATED' });
  }
}

// 2. Lock mechanism for critical operations
async function withLock<T>(key: string, fn: () => Promise<T>): Promise<T> {
  const lockKey = `lock:${key}`;
  
  // Try to acquire lock
  const existing = await storage.get(lockKey);
  if (existing && Date.now() - existing.timestamp < 30000) {
    throw new Error('Operation in progress in another tab');
  }
  
  // Acquire lock
  await storage.set(lockKey, { timestamp: Date.now(), tab: tabId });
  
  try {
    return await fn();
  } finally {
    // Release lock
    await storage.remove(lockKey);
  }
}
```

### Edge Case 4: Large File Uploads

**Problem:** Uploading large files can fail, need resumable uploads.

```javascript
// Chunked upload with resume capability
class ResumableUpload {
  private chunkSize = 1024 * 1024;  // 1MB chunks
  
  async upload(file: File, endpoint: string): Promise<string> {
    const uploadId = uuid();
    const totalChunks = Math.ceil(file.size / this.chunkSize);
    
    // Save upload state
    await storage.set(`upload:${uploadId}`, {
      fileName: file.name,
      fileSize: file.size,
      totalChunks,
      uploadedChunks: [],
      endpoint,
      startedAt: Date.now()
    });
    
    // Upload chunks
    for (let i = 0; i < totalChunks; i++) {
      const state = await storage.get(`upload:${uploadId}`);
      
      if (state.uploadedChunks.includes(i)) {
        continue;  // Already uploaded
      }
      
      const start = i * this.chunkSize;
      const end = Math.min(start + this.chunkSize, file.size);
      const chunk = file.slice(start, end);
      
      await this.uploadChunk(uploadId, i, chunk, totalChunks, endpoint);
      
      // Update state
      state.uploadedChunks.push(i);
      await storage.set(`upload:${uploadId}`, state);
    }
    
    // Complete upload
    const result = await api.post(`${endpoint}/complete`, { uploadId });
    
    // Cleanup
    await storage.remove(`upload:${uploadId}`);
    
    return result.url;
  }
  
  private async uploadChunk(
    uploadId: string,
    index: number,
    chunk: Blob,
    total: number,
    endpoint: string
  ): Promise<void> {
    const formData = new FormData();
    formData.append('chunk', chunk);
    formData.append('uploadId', uploadId);
    formData.append('index', index.toString());
    formData.append('total', total.toString());
    
    let retries = 0;
    const maxRetries = 3;
    
    while (retries < maxRetries) {
      try {
        await fetch(`${endpoint}/chunk`, {
          method: 'POST',
          body: formData
        });
        return;
      } catch (error) {
        retries++;
        if (retries === maxRetries) throw error;
        await new Promise(r => setTimeout(r, 1000 * retries));
      }
    }
  }
  
  // Resume interrupted upload
  async resumePendingUploads(): Promise<void> {
    const keys = await storage.keys();
    const uploadKeys = keys.filter(k => k.startsWith('upload:'));
    
    for (const key of uploadKeys) {
      const state = await storage.get(key);
      // Would need to reconstruct file from user or skip
      console.log('Pending upload:', state.fileName);
    }
  }
}
```

### Edge Case 5: Authentication Expiry While Offline

**Problem:** Token expires during offline period.

```javascript
// Handle auth expiry gracefully
class OfflineAuth {
  async checkAuthOnline(): Promise<boolean> {
    try {
      await api.get('/auth/verify');
      return true;
    } catch (error) {
      if (error.status === 401) {
        return false;
      }
      throw error;
    }
  }
  
  async handleSyncWithExpiredAuth(): Promise<void> {
    const isValid = await this.checkAuthOnline();
    
    if (!isValid) {
      // Save pending actions securely
      const pending = await syncQueue.getAll();
      
      if (pending.length > 0) {
        // Store with marker that auth is needed
        await storage.set('pending_after_reauth', pending);
        
        notifyUser(
          'Your session expired. Please log in again to sync your offline changes.'
        );
      }
      
      // Redirect to login
      redirectToLogin();
    }
  }
  
  async onReauthenticated(): Promise<void> {
    const pending = await storage.get('pending_after_reauth');
    
    if (pending && pending.length > 0) {
      // Restore pending actions
      for (const action of pending) {
        await syncQueue.add(action);
      }
      
      await storage.remove('pending_after_reauth');
      
      // Sync
      await syncEngine.sync();
    }
  }
}
```

### Edge Case 6: Data Corruption

**Problem:** Local storage can get corrupted.

```javascript
// Data integrity checks
class DataIntegrity {
  // Add checksums to stored data
  async setWithChecksum<T>(key: string, data: T): Promise<void> {
    const json = JSON.stringify(data);
    const checksum = await this.hash(json);
    
    await storage.set(key, {
      data,
      checksum,
      savedAt: Date.now()
    });
  }
  
  async getWithVerify<T>(key: string): Promise<T | null> {
    const stored = await storage.get(key);
    
    if (!stored) return null;
    
    const json = JSON.stringify(stored.data);
    const checksum = await this.hash(json);
    
    if (checksum !== stored.checksum) {
      console.error('Data corruption detected for key:', key);
      await this.handleCorruption(key, stored);
      return null;
    }
    
    return stored.data;
  }
  
  private async hash(data: string): Promise<string> {
    const encoder = new TextEncoder();
    const buffer = await crypto.subtle.digest('SHA-256', encoder.encode(data));
    return Array.from(new Uint8Array(buffer))
      .map(b => b.toString(16).padStart(2, '0'))
      .join('');
  }
  
  private async handleCorruption(key: string, corrupted: any): Promise<void> {
    // Log corruption
    await storage.append('corruption_log', {
      key,
      timestamp: Date.now(),
      data: corrupted
    });
    
    // Remove corrupted data
    await storage.remove(key);
    
    // Notify user
    notifyUser('Some data was corrupted and has been cleared. It will be re-downloaded.');
    
    // Try to recover from server
    if (network.online) {
      await this.recoverFromServer(key);
    }
  }
  
  // Periodic integrity check
  async runIntegrityCheck(): Promise<void> {
    const keys = await storage.keys();
    const issues: string[] = [];
    
    for (const key of keys) {
      try {
        const data = await this.getWithVerify(key);
        if (data === null && await storage.get(key)) {
          issues.push(key);
        }
      } catch {
        issues.push(key);
      }
    }
    
    if (issues.length > 0) {
      console.warn('Integrity issues found:', issues);
    }
  }
}
```

---

## Testing Offline Features

### 1. Manual Testing Checklist

```markdown
## Offline Testing Checklist

### Network Conditions
- [ ] Toggle airplane mode
- [ ] Disconnect WiFi
- [ ] Use Chrome DevTools Network throttling
- [ ] Use "Offline" checkbox in DevTools

### Scenarios to Test
- [ ] Load app while offline (with cached data)
- [ ] Load app while offline (no cache)
- [ ] Go offline while using app
- [ ] Come back online after being offline
- [ ] Perform CRUD operations while offline
- [ ] Sync after coming online
- [ ] Handle sync conflicts

### Edge Cases
- [ ] Very slow connection (2G)
- [ ] Intermittent connection
- [ ] Long offline period (hours)
- [ ] Multiple tabs open
- [ ] Close and reopen app while offline
- [ ] Storage quota exceeded
```

### 2. Automated Testing

```javascript
// Jest tests for offline functionality
describe('Offline Sync', () => {
  beforeEach(async () => {
    await storage.clear();
    await syncQueue.clear();
  });
  
  describe('Queue Management', () => {
    it('should queue actions when offline', async () => {
      // Simulate offline
      jest.spyOn(network, 'online', 'get').mockReturnValue(false);
      
      await createNote({ title: 'Test' });
      
      const queue = await syncQueue.getAll();
      expect(queue).toHaveLength(1);
      expect(queue[0].type).toBe('CREATE');
    });
    
    it('should process queue when coming online', async () => {
      // Queue some actions
      jest.spyOn(network, 'online', 'get').mockReturnValue(false);
      await createNote({ title: 'Test 1' });
      await createNote({ title: 'Test 2' });
      
      // Verify queued
      expect(await syncQueue.count()).toBe(2);
      
      // Come online
      jest.spyOn(network, 'online', 'get').mockReturnValue(true);
      const mockApi = jest.spyOn(api, 'post').mockResolvedValue({ id: '123' });
      
      await syncEngine.sync();
      
      expect(mockApi).toHaveBeenCalledTimes(2);
      expect(await syncQueue.count()).toBe(0);
    });
    
    it('should retry failed actions', async () => {
      jest.spyOn(network, 'online', 'get').mockReturnValue(true);
      
      let attempts = 0;
      jest.spyOn(api, 'post').mockImplementation(() => {
        attempts++;
        if (attempts < 3) {
          return Promise.reject(new Error('Server error'));
        }
        return Promise.resolve({ id: '123' });
      });
      
      await syncQueue.add({
        type: 'CREATE',
        entity: 'notes',
        payload: { title: 'Test' },
        priority: 50,
        maxRetries: 5
      });
      
      await syncEngine.sync();
      
      expect(attempts).toBe(3);
      expect(await syncQueue.count()).toBe(0);
    });
  });
  
  describe('Conflict Resolution', () => {
    it('should detect conflicts', async () => {
      const base = { id: '1', title: 'Original', version: 1 };
      const local = { id: '1', title: 'Local Edit', version: 1 };
      const server = { id: '1', title: 'Server Edit', version: 2 };
      
      const result = threeWayMerge(base, local, server);
      
      expect(result.conflicts).toContain('title');
    });
    
    it('should auto-merge non-conflicting changes', async () => {
      const base = { id: '1', title: 'Original', desc: 'Desc' };
      const local = { id: '1', title: 'New Title', desc: 'Desc' };
      const server = { id: '1', title: 'Original', desc: 'New Desc' };
      
      const result = threeWayMerge(base, local, server);
      
      expect(result.merged.title).toBe('New Title');
      expect(result.merged.desc).toBe('New Desc');
      expect(result.conflicts).toHaveLength(0);
    });
  });
});
```

### 3. E2E Testing with Playwright

```javascript
// Playwright test
import { test, expect } from '@playwright/test';

test.describe('Offline Mode', () => {
  test('should work offline after initial load', async ({ page, context }) => {
    // Load app online
    await page.goto('/');
    await expect(page.locator('.dashboard')).toBeVisible();
    
    // Go offline
    await context.setOffline(true);
    
    // Reload - should work from cache
    await page.reload();
    await expect(page.locator('.dashboard')).toBeVisible();
    
    // Create item offline
    await page.click('[data-testid="add-note"]');
    await page.fill('[data-testid="note-title"]', 'Offline Note');
    await page.click('[data-testid="save-note"]');
    
    // Verify saved locally
    await expect(page.locator('.toast')).toContainText('Saved offline');
    await expect(page.locator('[data-testid="note-list"]')).toContainText('Offline Note');
    
    // Come online
    await context.setOffline(false);
    
    // Wait for sync
    await expect(page.locator('.sync-indicator')).toHaveAttribute('data-status', 'synced');
    
    // Verify synced
    await expect(page.locator('.toast')).toContainText('Synced');
  });
  
  test('should show offline indicator', async ({ page, context }) => {
    await page.goto('/');
    
    // Initially online
    await expect(page.locator('[data-testid="connection-status"]')).toHaveText('Online');
    
    // Go offline
    await context.setOffline(true);
    await expect(page.locator('[data-testid="connection-status"]')).toHaveText('Offline');
    
    // Come online
    await context.setOffline(false);
    await expect(page.locator('[data-testid="connection-status"]')).toHaveText('Online');
  });
});
```

---

## Performance Optimization

### 1. Lazy Loading Data

```javascript
// Only cache what's needed
async function cacheForOffline() {
  // Essential data - cache immediately
  await cacheEssentialData();
  
  // Secondary data - cache in background
  requestIdleCallback(() => {
    cacheSecondaryData();
  });
}

// Use virtual lists for large datasets
function VirtualList({ items, renderItem }) {
  return (
    <AutoSizer>
      {({ height, width }) => (
        <FixedSizeList
          height={height}
          width={width}
          itemCount={items.length}
          itemSize={50}
        >
          {({ index, style }) => (
            <div style={style}>
              {renderItem(items[index])}
            </div>
          )}
        </FixedSizeList>
      )}
    </AutoSizer>
  );
}
```

### 2. Batch Operations

```javascript
// Batch multiple actions into one sync
async function batchSync(actions: Action[]) {
  if (!network.online) {
    for (const action of actions) {
      await syncQueue.add(action);
    }
    return;
  }
  
  // Send as batch
  try {
    await api.post('/batch', { actions });
  } catch {
    // Queue individually for retry
    for (const action of actions) {
      await syncQueue.add(action);
    }
  }
}
```

### 3. Delta Sync

```javascript
// Only sync changes, not full data
async function deltaSync(entity: string) {
  const lastSync = await storage.getLastSync(entity);
  
  // Fetch only changed items
  const changes = await api.get(`/${entity}/changes`, {
    since: lastSync
  });
  
  for (const change of changes) {
    switch (change.type) {
      case 'created':
      case 'updated':
        await storage.set(`${entity}:${change.id}`, change.data);
        break;
      case 'deleted':
        await storage.remove(`${entity}:${change.id}`);
        break;
    }
  }
  
  await storage.setLastSync(entity, Date.now());
}
```

---

## Security Considerations

### 1. Encrypt Sensitive Data

```javascript
// Encrypt before storing
class SecureStorage {
  private key: CryptoKey | null = null;
  
  async init(password: string) {
    const encoder = new TextEncoder();
    const keyMaterial = await crypto.subtle.importKey(
      'raw',
      encoder.encode(password),
      'PBKDF2',
      false,
      ['deriveKey']
    );
    
    this.key = await crypto.subtle.deriveKey(
      {
        name: 'PBKDF2',
        salt: encoder.encode('your-salt'),
        iterations: 100000,
        hash: 'SHA-256'
      },
      keyMaterial,
      { name: 'AES-GCM', length: 256 },
      false,
      ['encrypt', 'decrypt']
    );
  }
  
  async setSecure(key: string, data: any): Promise<void> {
    const encoder = new TextEncoder();
    const iv = crypto.getRandomValues(new Uint8Array(12));
    
    const encrypted = await crypto.subtle.encrypt(
      { name: 'AES-GCM', iv },
      this.key!,
      encoder.encode(JSON.stringify(data))
    );
    
    await storage.set(key, {
      iv: Array.from(iv),
      data: Array.from(new Uint8Array(encrypted))
    });
  }
  
  async getSecure<T>(key: string): Promise<T | null> {
    const stored = await storage.get(key);
    if (!stored) return null;
    
    const decrypted = await crypto.subtle.decrypt(
      { name: 'AES-GCM', iv: new Uint8Array(stored.iv) },
      this.key!,
      new Uint8Array(stored.data)
    );
    
    const decoder = new TextDecoder();
    return JSON.parse(decoder.decode(decrypted));
  }
}
```

### 2. Clear Sensitive Data on Logout

```javascript
async function logout() {
  // Clear all sensitive offline data
  await storage.remove('user');
  await storage.remove('token');
  await secureStorage.clear();
  
  // Keep non-sensitive cache
  // await storage.remove('cache:*');
  
  // Clear pending actions that may contain sensitive data
  await syncQueue.clear();
}
```

### 3. Token Storage Best Practices

```javascript
// Don't store tokens in localStorage
// Use httpOnly cookies when possible
// For SPAs, store in memory + refresh token rotation

class TokenManager {
  private accessToken: string | null = null;
  
  setToken(token: string) {
    // Memory only, not persisted
    this.accessToken = token;
  }
  
  getToken(): string | null {
    return this.accessToken;
  }
  
  async refreshToken(): Promise<void> {
    // Refresh token stored in httpOnly cookie
    const response = await fetch('/auth/refresh', {
      method: 'POST',
      credentials: 'include'  // Sends httpOnly cookie
    });
    
    const { accessToken } = await response.json();
    this.setToken(accessToken);
  }
}
```

---

## Best Practices Summary

### DO ✅

1. **Always save locally first** - Users get instant feedback
2. **Use optimistic UI** - Show success immediately, rollback on failure
3. **Queue all mutations** - Nothing should be lost
4. **Handle conflicts gracefully** - User should never lose work
5. **Show sync status** - Users should know what's happening
6. **Test offline scenarios** - Both automated and manual
7. **Set reasonable expiry** - Cached data shouldn't be stale forever
8. **Encrypt sensitive data** - LocalStorage/IndexedDB isn't secure
9. **Implement retry logic** - With exponential backoff
10. **Monitor storage usage** - Cleanup when needed

### DON'T ❌

1. **Don't assume online** - Always check network status
2. **Don't block on network** - Use timeouts and fallbacks
3. **Don't store tokens in localStorage** - Use memory or httpOnly cookies
4. **Don't ignore conflicts** - Have a resolution strategy
5. **Don't sync everything** - Be selective about what to cache
6. **Don't forget edge cases** - Multiple tabs, expired auth, etc.
7. **Don't skip validation** - Validate before queueing
8. **Don't ignore storage limits** - Have cleanup strategies
9. **Don't assume order** - Use timestamps and priorities
10. **Don't forget to cleanup** - Remove synced actions from queue

---

## Resources & Further Reading

### Libraries & Tools

| Tool | Purpose | Link |
|------|---------|------|
| **localForage** | IndexedDB wrapper | https://localforage.github.io/localForage/ |
| **Dexie.js** | IndexedDB wrapper | https://dexie.org/ |
| **PouchDB** | Offline-first database | https://pouchdb.com/ |
| **RxDB** | Reactive database | https://rxdb.info/ |
| **Workbox** | Service worker toolkit | https://developers.google.com/web/tools/workbox |
| **TanStack Query** | Data fetching | https://tanstack.com/query |
| **WatermelonDB** | React Native database | https://nozbe.github.io/WatermelonDB/ |

### Articles & Documentation

- [Offline First - A Better UX](https://offlinefirst.org/)
- [Service Workers API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [IndexedDB API - MDN](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [Background Sync API](https://developers.google.com/web/updates/2015/12/background-sync)
- [CRDTs Explained](https://crdt.tech/)

### Design Patterns

- [Offline First Design Patterns](https://alistapart.com/article/offline-first/)
- [Eventual Consistency](https://www.allthingsdistributed.com/2008/12/eventually_consistent.html)
- [Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
- [CQRS Pattern](https://martinfowler.com/bliki/CQRS.html)

---

## Conclusion

Building offline-capable applications requires careful planning and implementation. The key principles are:

1. **Local First** - Always save to local storage before network
2. **Optimistic UI** - Show immediate feedback
3. **Queue & Sync** - Queue operations for reliable delivery
4. **Conflict Resolution** - Have strategies for handling conflicts
5. **User Communication** - Keep users informed of sync status

By following the patterns and practices outlined in this guide, you can build applications that provide excellent user experience regardless of network conditions.

---

## Interview Questions & Answers

### Basic Level Questions

#### Q1: What is offline-first architecture?

**Answer:**
Offline-first is a design approach where the application is built to work without an internet connection as the default state, rather than treating offline as an error condition.

**Key principles:**
- Local storage is the primary data source
- Network is used for synchronization, not as a requirement
- User actions are never blocked by network status
- Data syncs automatically when connectivity returns

```javascript
// Offline-first approach
async function saveData(data) {
  // 1. Always save locally FIRST
  await localStorage.save(data);
  
  // 2. Then try to sync (non-blocking)
  if (navigator.onLine) {
    syncToServer(data).catch(console.error);
  }
}
```

---

#### Q2: What is the difference between localStorage, sessionStorage, and IndexedDB?

**Answer:**

| Feature | localStorage | sessionStorage | IndexedDB |
|---------|--------------|----------------|-----------|
| **Capacity** | ~5-10 MB | ~5-10 MB | 50MB+ (dynamic) |
| **Data Type** | Strings only | Strings only | Any (including blobs) |
| **API** | Synchronous | Synchronous | Asynchronous |
| **Persistence** | Until cleared | Tab session only | Until cleared |
| **Indexed** | No (key-value) | No (key-value) | Yes (queryable) |
| **Transactions** | No | No | Yes (ACID) |
| **Web Workers** | No | No | Yes |

**When to use:**
- **localStorage**: Small settings, tokens, simple key-value
- **sessionStorage**: Temporary data for single tab
- **IndexedDB**: Large datasets, complex queries, offline apps

---

#### Q3: How do you detect if a user is online or offline?

**Answer:**

```javascript
// 1. Check current status
const isOnline = navigator.onLine;

// 2. Listen for changes
window.addEventListener('online', () => {
  console.log('Back online!');
});

window.addEventListener('offline', () => {
  console.log('Gone offline!');
});

// 3. More reliable: actual network check
async function checkConnection() {
  try {
    const response = await fetch('/api/health', { 
      method: 'HEAD',
      cache: 'no-store'
    });
    return response.ok;
  } catch {
    return false;
  }
}
```

**Important:** `navigator.onLine` can be unreliable - it only checks if there's a network interface, not actual internet connectivity. Always combine with actual network requests for critical operations.

---

#### Q4: What is a Service Worker?

**Answer:**
A Service Worker is a JavaScript file that runs in the background, separate from the web page. It acts as a proxy between the browser and the network.

**Key capabilities:**
- Intercept and cache network requests
- Enable offline functionality
- Handle push notifications
- Perform background sync

**Lifecycle:**
```
Install → Waiting → Activate → Running
   ↓                              ↓
 Cache                        Fetch events
 assets                       handled
```

**Basic example:**
```javascript
// Register
navigator.serviceWorker.register('/sw.js');

// sw.js
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((cached) => {
      return cached || fetch(event.request);
    })
  );
});
```

---

#### Q5: What is optimistic UI?

**Answer:**
Optimistic UI is a pattern where the interface immediately reflects the expected result of an action before the server confirms it.

**Without optimistic UI (slow):**
```javascript
async function likePost(postId) {
  setLoading(true);
  await api.like(postId);      // User waits...
  setLoading(false);
  setLiked(true);              // Finally shows
}
```

**With optimistic UI (instant):**
```javascript
async function likePost(postId) {
  setLiked(true);              // Instant feedback!
  try {
    await api.like(postId);
  } catch (error) {
    setLiked(false);           // Rollback on failure
    showError('Failed to like');
  }
}
```

**Benefits:**
- Instant user feedback
- App feels faster
- Works naturally with offline mode

---

### Intermediate Level Questions

#### Q6: Explain different caching strategies for offline support.

**Answer:**

**1. Cache First (Offline First)**
```javascript
// Return cache, fallback to network
async function cacheFirst(request) {
  const cached = await caches.match(request);
  return cached || fetch(request);
}
```
Best for: Static assets, fonts, images

**2. Network First (Online First)**
```javascript
// Try network, fallback to cache
async function networkFirst(request) {
  try {
    const response = await fetch(request);
    cache.put(request, response.clone());
    return response;
  } catch {
    return caches.match(request);
  }
}
```
Best for: API data that changes frequently

**3. Stale While Revalidate**
```javascript
// Return cache immediately, update in background
async function staleWhileRevalidate(request) {
  const cached = await caches.match(request);
  
  const fetchPromise = fetch(request).then((response) => {
    cache.put(request, response.clone());
    return response;
  });
  
  return cached || fetchPromise;
}
```
Best for: Balance between freshness and speed

**4. Network Only**
```javascript
// Always fetch, no cache
async function networkOnly(request) {
  return fetch(request);
}
```
Best for: Real-time data, analytics

**5. Cache Only**
```javascript
// Only use cache
async function cacheOnly(request) {
  return caches.match(request);
}
```
Best for: Precached static assets

---

#### Q7: How do you handle conflicts when syncing offline data?

**Answer:**

**Common strategies:**

**1. Last Write Wins (LWW)**
```javascript
// Compare timestamps, newest wins
if (localData.updatedAt > serverData.updatedAt) {
  await saveToServer(localData);
} else {
  await saveToLocal(serverData);
}
```
Pros: Simple | Cons: May lose data

**2. Server Wins**
```javascript
// Server is always authoritative
await saveToLocal(serverData);
```
Pros: Consistent | Cons: Loses local changes

**3. Client Wins**
```javascript
// Local changes always win
await saveToServer(localData);
```
Pros: Preserves user work | Cons: May overwrite others' changes

**4. Three-Way Merge**
```javascript
function merge(base, local, server) {
  const result = {};
  for (const key of allKeys) {
    const localChanged = base[key] !== local[key];
    const serverChanged = base[key] !== server[key];
    
    if (localChanged && !serverChanged) result[key] = local[key];
    else if (!localChanged && serverChanged) result[key] = server[key];
    else if (localChanged && serverChanged) {
      // Conflict - need resolution
      result[key] = resolveConflict(local[key], server[key]);
    }
  }
  return result;
}
```
Pros: Smart merging | Cons: Complex

**5. User Resolution**
```javascript
// Ask user to choose
const choice = await showConflictDialog(local, server);
```
Pros: User control | Cons: Interrupts workflow

---

#### Q8: What is the Background Sync API?

**Answer:**
Background Sync allows you to defer actions until the user has stable connectivity, even if the user leaves the page.

**How it works:**
```javascript
// 1. Register sync in your app
async function queueSync() {
  const registration = await navigator.serviceWorker.ready;
  await registration.sync.register('sync-messages');
}

// 2. Handle in Service Worker
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-messages') {
    event.waitUntil(syncMessages());
  }
});

async function syncMessages() {
  const pending = await getFromIndexedDB('pending_messages');
  for (const message of pending) {
    await fetch('/api/messages', {
      method: 'POST',
      body: JSON.stringify(message)
    });
    await removeFromIndexedDB(message.id);
  }
}
```

**Key features:**
- Syncs even after tab is closed
- Retries automatically on failure
- Battery and network aware

**Browser support:** Chrome, Edge (not Safari/Firefox)

---

#### Q9: How would you implement a retry mechanism with exponential backoff?

**Answer:**

```javascript
async function fetchWithRetry(url, options = {}, maxRetries = 5) {
  let lastError;
  
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);
      
      if (!response.ok && response.status >= 500) {
        throw new Error(`Server error: ${response.status}`);
      }
      
      return response;
    } catch (error) {
      lastError = error;
      
      // Don't retry on client errors (4xx)
      if (error.status >= 400 && error.status < 500) {
        throw error;
      }
      
      // Calculate delay: 1s, 2s, 4s, 8s, 16s (with jitter)
      const baseDelay = Math.pow(2, attempt) * 1000;
      const jitter = Math.random() * 1000;
      const delay = Math.min(baseDelay + jitter, 30000); // Max 30s
      
      console.log(`Retry ${attempt + 1}/${maxRetries} in ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  
  throw lastError;
}

// Usage
try {
  const response = await fetchWithRetry('/api/data', { method: 'POST' });
} catch (error) {
  console.log('All retries failed:', error);
}
```

**Why exponential backoff?**
- Prevents overwhelming the server
- Gives server time to recover
- Reduces network congestion
- Jitter prevents thundering herd

---

#### Q10: How do you handle large file uploads offline?

**Answer:**

**Chunked resumable upload:**

```javascript
class ResumableUploader {
  constructor(file, chunkSize = 1024 * 1024) { // 1MB chunks
    this.file = file;
    this.chunkSize = chunkSize;
    this.uploadId = null;
  }
  
  async upload() {
    // Check for existing upload
    this.uploadId = await this.getOrCreateUpload();
    const uploadedChunks = await this.getUploadedChunks();
    
    const totalChunks = Math.ceil(this.file.size / this.chunkSize);
    
    for (let i = 0; i < totalChunks; i++) {
      if (uploadedChunks.includes(i)) continue; // Skip uploaded
      
      // Wait for online
      await this.waitForOnline();
      
      const chunk = this.file.slice(
        i * this.chunkSize,
        (i + 1) * this.chunkSize
      );
      
      await this.uploadChunk(chunk, i, totalChunks);
      await this.saveProgress(i);
    }
    
    return this.completeUpload();
  }
  
  async waitForOnline() {
    if (navigator.onLine) return;
    
    return new Promise(resolve => {
      window.addEventListener('online', resolve, { once: true });
    });
  }
  
  async saveProgress(chunkIndex) {
    await localforage.setItem(`upload:${this.uploadId}`, {
      fileId: this.uploadId,
      fileName: this.file.name,
      lastChunk: chunkIndex,
      timestamp: Date.now()
    });
  }
}
```

**Key features:**
- Resumable after connection loss
- Progress saved to IndexedDB
- Waits for connectivity
- Can resume after app restart

---

### Advanced Level Questions

#### Q11: Explain CRDTs and when you would use them.

**Answer:**

**CRDT = Conflict-free Replicated Data Type**

CRDTs are data structures that can be replicated across multiple nodes, updated independently, and merged without conflicts.

**Types of CRDTs:**

**1. G-Counter (Grow-only Counter)**
```javascript
class GCounter {
  constructor(nodeId) {
    this.nodeId = nodeId;
    this.counts = {}; // { nodeId: count }
  }
  
  increment() {
    this.counts[this.nodeId] = (this.counts[this.nodeId] || 0) + 1;
  }
  
  value() {
    return Object.values(this.counts).reduce((a, b) => a + b, 0);
  }
  
  merge(other) {
    for (const [node, count] of Object.entries(other.counts)) {
      this.counts[node] = Math.max(this.counts[node] || 0, count);
    }
  }
}
```

**2. LWW-Register (Last-Writer-Wins)**
```javascript
class LWWRegister {
  constructor() {
    this.value = null;
    this.timestamp = 0;
  }
  
  set(value) {
    this.value = value;
    this.timestamp = Date.now();
  }
  
  merge(other) {
    if (other.timestamp > this.timestamp) {
      this.value = other.value;
      this.timestamp = other.timestamp;
    }
  }
}
```

**3. G-Set (Grow-only Set)**
```javascript
class GSet {
  constructor() {
    this.items = new Set();
  }
  
  add(item) {
    this.items.add(item);
  }
  
  merge(other) {
    for (const item of other.items) {
      this.items.add(item);
    }
  }
}
```

**When to use CRDTs:**
- Real-time collaboration (Google Docs-like)
- Distributed systems with no central server
- Peer-to-peer applications
- Apps requiring eventual consistency without conflicts

**Libraries:** Automerge, Yjs, Y-CRDT

---

#### Q12: How would you design an offline-first architecture for a collaborative document editor?

**Answer:**

**Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENT                                │
│                                                              │
│  ┌──────────┐   ┌──────────────┐   ┌──────────────────┐    │
│  │  Editor  │◀─▶│  CRDT State  │◀─▶│   IndexedDB      │    │
│  │  (UI)    │   │  (Automerge) │   │   (Persistence)  │    │
│  └──────────┘   └──────┬───────┘   └──────────────────┘    │
│                        │                                     │
│                        ▼                                     │
│               ┌──────────────────┐                          │
│               │   Sync Manager   │                          │
│               └────────┬─────────┘                          │
│                        │                                     │
│         ┌──────────────┼──────────────┐                     │
│         ▼              ▼              ▼                     │
│   ┌──────────┐  ┌──────────┐  ┌──────────────┐            │
│   │WebSocket │  │  HTTP    │  │  Background  │            │
│   │(realtime)│  │ (batch)  │  │    Sync      │            │
│   └──────────┘  └──────────┘  └──────────────┘            │
│                                                             │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
                   ┌──────────────┐
                   │    SERVER    │
                   │  (Changes)   │
                   └──────────────┘
```

**Implementation:**

```javascript
class CollaborativeDocument {
  constructor(docId) {
    this.docId = docId;
    this.doc = Automerge.init();
    this.peers = new Map();
  }
  
  // Local edit
  applyLocalChange(changeFn) {
    this.doc = Automerge.change(this.doc, changeFn);
    this.persistLocally();
    this.broadcastChange();
  }
  
  // Receive remote change
  applyRemoteChange(changes) {
    this.doc = Automerge.applyChanges(this.doc, changes);
    this.persistLocally();
    this.notifyUI();
  }
  
  // Persist to IndexedDB
  async persistLocally() {
    const binary = Automerge.save(this.doc);
    await localforage.setItem(`doc:${this.docId}`, binary);
  }
  
  // Load from IndexedDB
  async loadLocally() {
    const binary = await localforage.getItem(`doc:${this.docId}`);
    if (binary) {
      this.doc = Automerge.load(binary);
    }
  }
  
  // Sync with server
  async syncWithServer() {
    const localChanges = Automerge.getChanges(
      Automerge.init(),
      this.doc
    );
    
    const response = await api.post(`/docs/${this.docId}/sync`, {
      changes: localChanges,
      lastSyncVersion: this.lastSyncVersion
    });
    
    if (response.changes.length > 0) {
      this.applyRemoteChange(response.changes);
    }
    
    this.lastSyncVersion = response.version;
  }
}
```

**Key considerations:**
1. **CRDT for conflict-free merging** - Automerge/Yjs
2. **Local persistence** - IndexedDB for offline access
3. **Multi-channel sync** - WebSocket for real-time, HTTP for batch
4. **Background sync** - Service Worker for tab-closed sync
5. **Cursor/selection sync** - Separate ephemeral state

---

#### Q13: How do you handle database schema migrations in an offline-first app?

**Answer:**

**The challenge:** Users might be offline when you deploy a schema change.

**Solution: Versioned migrations with IndexedDB**

```javascript
// Define migrations
const MIGRATIONS = {
  1: (db) => {
    // Initial schema
    db.createObjectStore('notes', { keyPath: 'id' });
  },
  
  2: (db, transaction) => {
    // Add index
    const store = transaction.objectStore('notes');
    store.createIndex('createdAt', 'createdAt');
  },
  
  3: async (db, transaction) => {
    // Add new field to existing data
    const store = transaction.objectStore('notes');
    const notes = await store.getAll();
    
    for (const note of notes) {
      note.category = note.category || 'uncategorized';
      store.put(note);
    }
  },
  
  4: (db) => {
    // Add new store
    db.createObjectStore('tags', { keyPath: 'id' });
  }
};

// Run migrations
function openDatabase() {
  const CURRENT_VERSION = 4;
  
  return new Promise((resolve, reject) => {
    const request = indexedDB.open('myapp', CURRENT_VERSION);
    
    request.onupgradeneeded = (event) => {
      const db = event.target.result;
      const transaction = event.target.transaction;
      const oldVersion = event.oldVersion;
      
      // Run each migration in order
      for (let v = oldVersion + 1; v <= CURRENT_VERSION; v++) {
        console.log(`Running migration ${v}`);
        MIGRATIONS[v](db, transaction);
      }
    };
    
    request.onsuccess = () => resolve(request.result);
    request.onerror = () => reject(request.error);
  });
}
```

**With Dexie.js:**

```javascript
const db = new Dexie('myapp');

db.version(1).stores({
  notes: '++id, title'
});

db.version(2).stores({
  notes: '++id, title, createdAt'
}).upgrade(tx => {
  return tx.notes.toCollection().modify(note => {
    note.createdAt = note.createdAt || Date.now();
  });
});

db.version(3).stores({
  notes: '++id, title, createdAt',
  tags: '++id, name'
});
```

**Best practices:**
1. Never delete migrations
2. Make migrations idempotent
3. Handle missing fields gracefully
4. Test migrations with real data
5. Keep old version compatibility in API

---

#### Q14: How would you implement offline analytics/event tracking?

**Answer:**

```javascript
class OfflineAnalytics {
  constructor() {
    this.queue = [];
    this.maxQueueSize = 1000;
    this.flushInterval = 30000; // 30 seconds
    
    this.init();
  }
  
  async init() {
    // Load persisted events
    this.queue = await localforage.getItem('analytics_queue') || [];
    
    // Periodic flush
    setInterval(() => this.flush(), this.flushInterval);
    
    // Flush on online
    window.addEventListener('online', () => this.flush());
    
    // Flush before page unload
    window.addEventListener('beforeunload', () => this.persistQueue());
    
    // Flush on visibility change (mobile background)
    document.addEventListener('visibilitychange', () => {
      if (document.hidden) this.persistQueue();
    });
  }
  
  track(eventName, properties = {}) {
    const event = {
      id: uuid(),
      event: eventName,
      properties,
      timestamp: Date.now(),
      sessionId: this.sessionId,
      deviceId: this.deviceId,
      offline: !navigator.onLine
    };
    
    this.queue.push(event);
    
    // Prevent memory issues
    if (this.queue.length > this.maxQueueSize) {
      this.queue = this.queue.slice(-this.maxQueueSize);
    }
    
    // Try to send immediately if online
    if (navigator.onLine && this.queue.length >= 10) {
      this.flush();
    }
  }
  
  async flush() {
    if (!navigator.onLine || this.queue.length === 0) return;
    
    const toSend = [...this.queue];
    this.queue = [];
    
    try {
      await fetch('/api/analytics/batch', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ events: toSend }),
        // Use keepalive for beforeunload
        keepalive: true
      });
      
      await localforage.setItem('analytics_queue', this.queue);
    } catch (error) {
      // Put events back in queue
      this.queue = [...toSend, ...this.queue];
      await this.persistQueue();
    }
  }
  
  async persistQueue() {
    await localforage.setItem('analytics_queue', this.queue);
  }
}

// Usage
const analytics = new OfflineAnalytics();
analytics.track('page_view', { page: '/home' });
analytics.track('button_click', { button: 'signup' });
```

**Key features:**
- Events queued when offline
- Batch sending for efficiency
- Persisted to survive page refresh
- `keepalive` for beforeunload
- Deduplication via event IDs

---

#### Q15: What are the security considerations for offline storage?

**Answer:**

**1. Data Encryption**

```javascript
class SecureStorage {
  async init(password) {
    // Derive key from password
    const keyMaterial = await crypto.subtle.importKey(
      'raw',
      new TextEncoder().encode(password),
      'PBKDF2',
      false,
      ['deriveKey']
    );
    
    this.key = await crypto.subtle.deriveKey(
      {
        name: 'PBKDF2',
        salt: crypto.getRandomValues(new Uint8Array(16)),
        iterations: 100000,
        hash: 'SHA-256'
      },
      keyMaterial,
      { name: 'AES-GCM', length: 256 },
      false,
      ['encrypt', 'decrypt']
    );
  }
  
  async encrypt(data) {
    const iv = crypto.getRandomValues(new Uint8Array(12));
    const encrypted = await crypto.subtle.encrypt(
      { name: 'AES-GCM', iv },
      this.key,
      new TextEncoder().encode(JSON.stringify(data))
    );
    return { iv: Array.from(iv), data: Array.from(new Uint8Array(encrypted)) };
  }
  
  async decrypt(encrypted) {
    const decrypted = await crypto.subtle.decrypt(
      { name: 'AES-GCM', iv: new Uint8Array(encrypted.iv) },
      this.key,
      new Uint8Array(encrypted.data)
    );
    return JSON.parse(new TextDecoder().decode(decrypted));
  }
}
```

**2. Token Storage**

```javascript
// ❌ BAD - Tokens in localStorage
localStorage.setItem('token', accessToken);

// ✅ GOOD - Tokens in memory only
class TokenManager {
  #accessToken = null; // Private, not persisted
  
  setToken(token) {
    this.#accessToken = token;
  }
  
  getToken() {
    return this.#accessToken;
  }
}

// ✅ BETTER - Use httpOnly cookies for refresh tokens
// Server sets: Set-Cookie: refresh_token=xxx; HttpOnly; Secure; SameSite=Strict
```

**3. Clear Data on Logout**

```javascript
async function secureLogout() {
  // Clear all sensitive data
  await localforage.clear();
  localStorage.clear();
  sessionStorage.clear();
  
  // Clear caches
  const cacheNames = await caches.keys();
  await Promise.all(cacheNames.map(name => caches.delete(name)));
  
  // Unregister service workers
  const registrations = await navigator.serviceWorker.getRegistrations();
  await Promise.all(registrations.map(reg => reg.unregister()));
  
  // Clear memory
  tokenManager.clear();
  
  // Redirect to login
  window.location.href = '/login';
}
```

**4. Data Classification**

```javascript
const STORAGE_POLICY = {
  // Public - no encryption needed
  PUBLIC: {
    encrypt: false,
    persist: true,
    clearOnLogout: false
  },
  // User data - encrypt
  PRIVATE: {
    encrypt: true,
    persist: true,
    clearOnLogout: true
  },
  // Sensitive - encrypt + session only
  SENSITIVE: {
    encrypt: true,
    persist: false, // Memory only
    clearOnLogout: true
  },
  // Credentials - never store
  CREDENTIAL: {
    encrypt: false,
    persist: false,
    clearOnLogout: true
  }
};
```

**Security checklist:**
- ✅ Encrypt sensitive data at rest
- ✅ Use httpOnly cookies for auth tokens
- ✅ Clear all data on logout
- ✅ Don't store passwords/secrets
- ✅ Validate data integrity (checksums)
- ✅ Use HTTPS only
- ✅ Implement session timeout
- ✅ Consider data classification

---

### Scenario-Based Questions

#### Q16: A user makes 100 changes offline over a week. When they come online, the sync takes too long and fails. How do you handle this?

**Answer:**

**Problem analysis:**
- Large batch might timeout
- Single failure shouldn't lose all data
- User should see progress

**Solution:**

```javascript
class BatchSyncManager {
  constructor() {
    this.batchSize = 10;
    this.maxRetries = 3;
  }
  
  async syncAll() {
    const pending = await getPendingActions();
    const total = pending.length;
    let synced = 0;
    let failed = [];
    
    // Process in batches
    for (let i = 0; i < pending.length; i += this.batchSize) {
      const batch = pending.slice(i, i + this.batchSize);
      
      try {
        // Send batch with timeout
        await this.syncBatch(batch);
        synced += batch.length;
        
        // Update progress
        this.reportProgress(synced, total);
        
        // Remove synced from queue
        await this.removeSynced(batch);
        
      } catch (error) {
        // Partial batch failure - retry individually
        for (const action of batch) {
          try {
            await this.syncSingle(action);
            synced++;
            await this.removeAction(action.id);
          } catch {
            failed.push(action);
          }
        }
      }
      
      // Brief pause between batches
      await sleep(100);
    }
    
    // Handle failures
    if (failed.length > 0) {
      await this.handleFailures(failed);
    }
    
    return { synced, failed: failed.length };
  }
  
  async syncBatch(batch) {
    const controller = new AbortController();
    const timeout = setTimeout(() => controller.abort(), 30000);
    
    try {
      await fetch('/api/sync/batch', {
        method: 'POST',
        body: JSON.stringify({ actions: batch }),
        signal: controller.signal
      });
    } finally {
      clearTimeout(timeout);
    }
  }
  
  async handleFailures(failed) {
    // Option 1: Retry later
    for (const action of failed) {
      action.retryCount = (action.retryCount || 0) + 1;
      if (action.retryCount < this.maxRetries) {
        await this.requeueAction(action);
      } else {
        await this.moveToDeadLetter(action);
        this.notifyUser(`Failed to sync: ${action.description}`);
      }
    }
  }
  
  reportProgress(synced, total) {
    const percent = Math.round((synced / total) * 100);
    showToast(`Syncing... ${percent}% (${synced}/${total})`);
  }
}
```

**Key strategies:**
1. **Batch processing** - Send 10 at a time
2. **Progress feedback** - Show sync progress
3. **Individual retry** - If batch fails, try one by one
4. **Timeouts** - Prevent hanging
5. **Dead letter queue** - Don't lose permanently failed items
6. **User notification** - Tell user about failures

---

#### Q17: How would you handle a scenario where the same item is edited on 3 different devices, all offline?

**Answer:**

```javascript
// Scenario: Note edited offline on Phone, Tablet, and Desktop
// Phone:   "Hello" → "Hello World"
// Tablet:  "Hello" → "Hello There"
// Desktop: "Hello" → "Hi"

// Solution 1: Timestamps with User Choice
class MultiDeviceResolver {
  async resolveConflict(versions) {
    // Sort by timestamp (newest first)
    versions.sort((a, b) => b.updatedAt - a.updatedAt);
    
    // If timestamps are close (within 1 hour), ask user
    const newest = versions[0];
    const closeVersions = versions.filter(
      v => newest.updatedAt - v.updatedAt < 3600000
    );
    
    if (closeVersions.length > 1) {
      // Multiple recent edits - show conflict UI
      return this.showConflictUI(closeVersions);
    }
    
    // Clear winner by time
    return newest;
  }
  
  async showConflictUI(versions) {
    return new Promise(resolve => {
      showModal({
        title: 'Sync Conflict',
        message: 'This item was edited on multiple devices:',
        versions: versions.map(v => ({
          device: v.deviceName,
          preview: v.content.substring(0, 100),
          time: formatTime(v.updatedAt)
        })),
        actions: [
          ...versions.map((v, i) => ({
            label: `Use ${v.deviceName} version`,
            action: () => resolve(v)
          })),
          {
            label: 'Merge all changes',
            action: () => resolve(this.mergeVersions(versions))
          }
        ]
      });
    });
  }
}

// Solution 2: CRDT (No conflicts ever)
class CRDTDocument {
  constructor() {
    this.doc = Automerge.init();
  }
  
  // Each device makes changes independently
  edit(text, deviceId) {
    this.doc = Automerge.change(this.doc, `Edit from ${deviceId}`, doc => {
      doc.text = new Automerge.Text(text);
    });
  }
  
  // Merge all versions - no conflicts!
  merge(otherDoc) {
    this.doc = Automerge.merge(this.doc, otherDoc);
    // Automerge handles concurrent edits automatically
  }
}

// Solution 3: Operational Transform
class OTDocument {
  constructor() {
    this.operations = [];
    this.baseVersion = 0;
  }
  
  applyOperation(op, fromVersion) {
    // Transform operation against concurrent ops
    let transformed = op;
    
    for (let i = fromVersion; i < this.operations.length; i++) {
      transformed = transform(transformed, this.operations[i]);
    }
    
    this.operations.push(transformed);
    return this.computeContent();
  }
}
```

**Best approach depends on data type:**
- **Simple values:** Last Write Wins or User Choice
- **Text content:** CRDT (Automerge/Yjs) or OT
- **Structured data:** Three-way merge with field-level resolution

---

#### Q18: Design an offline sync system for a banking app. What special considerations apply?

**Answer:**

**Special considerations for financial apps:**

```javascript
class SecureBankingSync {
  constructor() {
    this.pendingTransactions = [];
  }
  
  // 1. NEVER allow offline money transfers
  async transfer(fromAccount, toAccount, amount) {
    if (!navigator.onLine) {
      throw new Error('Internet connection required for transfers');
    }
    
    // Real-time balance check required
    const balance = await this.fetchBalance(fromAccount);
    if (balance < amount) {
      throw new Error('Insufficient funds');
    }
    
    return this.executeTransfer(fromAccount, toAccount, amount);
  }
  
  // 2. Read-only data CAN be cached
  async getTransactionHistory(accountId) {
    const cached = await secureStorage.get(`history:${accountId}`);
    
    if (navigator.onLine) {
      try {
        const fresh = await api.get(`/accounts/${accountId}/transactions`);
        await secureStorage.set(`history:${accountId}`, fresh);
        return fresh;
      } catch {
        return cached; // Fallback to cache
      }
    }
    
    return cached;
  }
  
  // 3. Balance is always server-authoritative
  async getBalance(accountId) {
    if (!navigator.onLine) {
      // Show cached with warning
      const cached = await secureStorage.get(`balance:${accountId}`);
      return {
        ...cached,
        isStale: true,
        warning: 'Showing last known balance. Connect to see current balance.'
      };
    }
    
    const balance = await api.get(`/accounts/${accountId}/balance`);
    await secureStorage.set(`balance:${accountId}`, balance);
    return balance;
  }
  
  // 4. Bill payments CAN be scheduled offline
  async scheduleBillPayment(payment) {
    const scheduled = {
      ...payment,
      id: uuid(),
      status: 'pending_sync',
      createdAt: Date.now(),
      // Must be future dated
      executeDate: payment.date // Must be > now + 24h
    };
    
    // Validate future date
    if (scheduled.executeDate < Date.now() + 24 * 60 * 60 * 1000) {
      throw new Error('Offline payments must be scheduled 24+ hours ahead');
    }
    
    await secureStorage.append('scheduled_payments', scheduled);
    
    if (navigator.onLine) {
      await this.syncScheduledPayments();
    }
    
    return scheduled;
  }
  
  // 5. Strong encryption for all local data
  async initSecureStorage(pin) {
    const key = await deriveKey(pin, 'banking-salt', 200000); // High iterations
    this.encryption = new AESEncryption(key);
  }
  
  // 6. Session timeout
  startSessionTimer() {
    this.sessionTimeout = setTimeout(() => {
      this.logout();
      showAlert('Session expired for security');
    }, 5 * 60 * 1000); // 5 minutes
  }
  
  // 7. Clear everything on logout
  async logout() {
    await secureStorage.clear();
    await caches.delete('banking-cache');
    this.encryption = null;
    window.location.href = '/login';
  }
}
```

**Banking-specific rules:**
| Feature | Offline Allowed? | Reason |
|---------|-----------------|--------|
| View balance | ✅ (with warning) | Convenience |
| View history | ✅ (cached) | Convenience |
| Transfer money | ❌ Never | Balance verification required |
| Schedule future payment | ✅ (24h+ ahead) | Can verify before execution |
| Pay bills | ❌ (unless scheduled) | Real-time verification |
| View statements | ✅ (cached) | Read-only |
| Change settings | ❌ | Security |

**Security requirements:**
- ✅ PIN/biometric to access offline data
- ✅ AES-256 encryption at rest
- ✅ Short session timeouts
- ✅ Full wipe on logout
- ✅ No sensitive data in logs
- ✅ Certificate pinning

---

#### Q19: How do you test offline functionality in CI/CD pipelines?

**Answer:**

```javascript
// 1. Unit tests for sync logic
describe('SyncQueue', () => {
  beforeEach(async () => {
    await localforage.clear();
  });
  
  it('should queue actions when offline', async () => {
    // Mock offline
    jest.spyOn(navigator, 'onLine', 'get').mockReturnValue(false);
    
    await syncQueue.add({
      type: 'CREATE',
      entity: 'notes',
      payload: { title: 'Test' }
    });
    
    expect(await syncQueue.count()).toBe(1);
  });
  
  it('should process queue in order', async () => {
    const processed = [];
    const mockProcess = jest.fn(action => {
      processed.push(action.payload.order);
    });
    
    await syncQueue.add({ type: 'CREATE', payload: { order: 1 } });
    await syncQueue.add({ type: 'CREATE', payload: { order: 2 } });
    await syncQueue.add({ type: 'CREATE', payload: { order: 3 } });
    
    await processQueue(mockProcess);
    
    expect(processed).toEqual([1, 2, 3]);
  });
});

// 2. Integration tests with Playwright
import { test, expect } from '@playwright/test';

test.describe('Offline Mode', () => {
  test('should queue actions when offline', async ({ page, context }) => {
    await page.goto('/notes');
    
    // Go offline
    await context.setOffline(true);
    
    // Create note
    await page.click('[data-testid="new-note"]');
    await page.fill('[data-testid="note-content"]', 'Offline note');
    await page.click('[data-testid="save"]');
    
    // Verify queued
    await expect(page.locator('[data-testid="pending-badge"]'))
      .toHaveText('1');
    
    // Come online
    await context.setOffline(false);
    
    // Wait for sync
    await expect(page.locator('[data-testid="pending-badge"]'))
      .toHaveText('0');
  });
  
  test('should work with service worker', async ({ page }) => {
    await page.goto('/');
    
    // Wait for SW registration
    await page.waitForFunction(() => 
      navigator.serviceWorker.controller !== null
    );
    
    // Go offline
    await page.context().setOffline(true);
    
    // Navigate - should load from cache
    await page.goto('/dashboard');
    await expect(page.locator('h1')).toContainText('Dashboard');
  });
});

// 3. E2E test with real network simulation
test('should handle network flakiness', async ({ page, context }) => {
  await page.goto('/');
  
  // Simulate flaky network
  for (let i = 0; i < 5; i++) {
    await context.setOffline(true);
    await page.waitForTimeout(1000);
    await context.setOffline(false);
    await page.waitForTimeout(1000);
  }
  
  // App should still be functional
  await expect(page.locator('[data-testid="app"]')).toBeVisible();
  
  // No errors in console
  const errors = [];
  page.on('console', msg => {
    if (msg.type() === 'error') errors.push(msg.text());
  });
  expect(errors).toHaveLength(0);
});

// 4. GitHub Actions workflow
/*
name: Test Offline Features

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run unit tests
        run: npm test -- --coverage
        
      - name: Install Playwright
        run: npx playwright install --with-deps
        
      - name: Run E2E tests
        run: npx playwright test
        
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: test-results
          path: test-results/
*/
```

**CI/CD checklist:**
- ✅ Unit tests for sync logic
- ✅ Integration tests with mocked offline
- ✅ E2E tests with Playwright offline mode
- ✅ Service Worker tests
- ✅ Network flakiness simulation
- ✅ Test data persistence across restarts

---

#### Q20: What metrics would you track to monitor offline sync health in production?

**Answer:**

```javascript
// Analytics for offline sync monitoring
class SyncAnalytics {
  // Track sync operations
  async trackSync(result) {
    await analytics.track('sync_completed', {
      duration_ms: result.duration,
      actions_synced: result.synced,
      actions_failed: result.failed,
      queue_size_before: result.queueSizeBefore,
      queue_size_after: result.queueSizeAfter,
      offline_duration_ms: result.offlineDuration,
      retry_count: result.retryCount,
      conflict_count: result.conflicts
    });
  }
  
  // Track offline sessions
  async trackOfflineSession(session) {
    await analytics.track('offline_session', {
      duration_ms: session.duration,
      actions_queued: session.actionsQueued,
      pages_visited: session.pagesVisited,
      cache_hits: session.cacheHits,
      cache_misses: session.cacheMisses
    });
  }
  
  // Track conflicts
  async trackConflict(conflict) {
    await analytics.track('sync_conflict', {
      entity_type: conflict.entityType,
      resolution: conflict.resolution, // 'auto', 'user_local', 'user_server'
      time_since_edit_ms: conflict.timeSinceEdit,
      offline_duration_ms: conflict.offlineDuration
    });
  }
  
  // Track failures
  async trackSyncFailure(failure) {
    await analytics.track('sync_failure', {
      action_type: failure.actionType,
      error_type: failure.errorType,
      retry_count: failure.retryCount,
      will_retry: failure.willRetry,
      error_message: failure.message
    });
  }
}

// Metrics dashboard queries (example for Datadog/Grafana)
const METRICS = {
  // Sync success rate
  syncSuccessRate: `
    sum(sync_completed{actions_failed=0}) / 
    sum(sync_completed) * 100
  `,
  
  // Average sync duration
  avgSyncDuration: `
    avg(sync_completed.duration_ms)
  `,
  
  // 95th percentile queue size
  p95QueueSize: `
    percentile(sync_completed.queue_size_before, 95)
  `,
  
  // Conflict rate
  conflictRate: `
    sum(sync_conflict) / sum(sync_completed) * 100
  `,
  
  // Average offline duration
  avgOfflineDuration: `
    avg(offline_session.duration_ms)
  `,
  
  // Failed actions (needs attention)
  failedActions: `
    sum(sync_failure{will_retry=false})
  `
};

// Alerts
const ALERTS = [
  {
    name: 'High sync failure rate',
    condition: 'syncSuccessRate < 95%',
    severity: 'warning'
  },
  {
    name: 'Sync queue backing up',
    condition: 'p95QueueSize > 100',
    severity: 'warning'
  },
  {
    name: 'Many unresolved conflicts',
    condition: 'conflictRate > 5%',
    severity: 'info'
  },
  {
    name: 'Critical sync failures',
    condition: 'failedActions > 0 AND action_type IN (payment, order)',
    severity: 'critical'
  }
];
```

**Key metrics to track:**

| Metric | Why It Matters |
|--------|---------------|
| **Sync success rate** | Overall health indicator |
| **Sync duration** | Performance monitoring |
| **Queue size** | Backlog indicator |
| **Conflict rate** | UX issue indicator |
| **Offline duration** | Usage patterns |
| **Failed actions** | Data loss risk |
| **Retry count** | Network reliability |
| **Cache hit rate** | Offline readiness |

**Dashboard example:**
```
┌─────────────────────────────────────────────────────────────┐
│                    OFFLINE SYNC DASHBOARD                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Sync Success Rate    Queue Size (p95)    Avg Sync Time     │
│  ┌───────────────┐   ┌───────────────┐   ┌───────────────┐ │
│  │     98.5%     │   │      12       │   │    1.2s       │ │
│  │    ▲ 0.3%     │   │    ▼ 5        │   │   ▼ 0.3s      │ │
│  └───────────────┘   └───────────────┘   └───────────────┘ │
│                                                              │
│  Offline Sessions     Conflicts Today    Failed Actions     │
│  ┌───────────────┐   ┌───────────────┐   ┌───────────────┐ │
│  │     1,234     │   │      23       │   │      0        │ │
│  │    ▲ 12%      │   │    ▼ 15%      │   │    ✓ Good     │ │
│  └───────────────┘   └───────────────┘   └───────────────┘ │
│                                                              │
│  [Chart: Sync operations over time]                         │
│  [Chart: Offline duration distribution]                     │
│  [Chart: Error breakdown by type]                           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

### Quick Reference Cheat Sheet

```
┌─────────────────────────────────────────────────────────────┐
│              OFFLINE SYNC CHEAT SHEET                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  STORAGE OPTIONS:                                           │
│  • localStorage     → Small, sync, 5MB                      │
│  • IndexedDB        → Large, async, 50MB+                   │
│  • Cache API        → HTTP responses                        │
│                                                              │
│  PACKAGES:                                                   │
│  • localforage      → Simple IndexedDB wrapper              │
│  • Dexie.js         → Advanced IndexedDB                    │
│  • PouchDB          → Full sync with CouchDB                │
│  • RxDB             → Reactive + sync                       │
│                                                              │
│  CACHE STRATEGIES:                                          │
│  • Cache First      → Fast, may be stale                    │
│  • Network First    → Fresh, needs connection               │
│  • Stale While Rev  → Fast + fresh                          │
│                                                              │
│  CONFLICT RESOLUTION:                                       │
│  • Last Write Wins  → Simple, may lose data                 │
│  • Three-Way Merge  → Smart, complex                        │
│  • CRDT             → No conflicts, complex                 │
│  • User Choice      → Safe, interrupts UX                   │
│                                                              │
│  KEY EVENTS:                                                │
│  • window.ononline/onoffline                                │
│  • navigator.onLine                                         │
│  • serviceWorker.sync                                       │
│                                                              │
│  MUST DO:                                                   │
│  ✓ Save locally first                                       │
│  ✓ Queue all mutations                                      │
│  ✓ Show sync status                                         │
│  ✓ Handle conflicts                                         │
│  ✓ Retry with backoff                                       │
│  ✓ Encrypt sensitive data                                   │
│                                                              │
│  NEVER DO:                                                  │
│  ✗ Block on network                                         │
│  ✗ Store tokens in localStorage                             │
│  ✗ Ignore conflicts                                         │
│  ✗ Skip offline testing                                     │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Conclusion

Building offline-capable applications requires careful planning and implementation. The key principles are:

1. **Local First** - Always save to local storage before network
2. **Optimistic UI** - Show immediate feedback
3. **Queue & Sync** - Queue operations for reliable delivery
4. **Conflict Resolution** - Have strategies for handling conflicts
5. **User Communication** - Keep users informed of sync status

By following the patterns and practices outlined in this guide, you can build applications that provide excellent user experience regardless of network conditions.

---

*Last updated: January 2026*

