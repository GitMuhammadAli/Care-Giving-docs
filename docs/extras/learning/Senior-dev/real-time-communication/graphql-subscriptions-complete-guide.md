# 🔔 GraphQL Subscriptions - Complete Guide

> A comprehensive guide to GraphQL Subscriptions - WebSocket transport, Apollo Server, real-time data, scaling considerations, and production patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "GraphQL Subscriptions provide real-time, event-driven data delivery using WebSockets, allowing clients to subscribe to specific events and receive updates automatically when the subscribed data changes on the server."

### The 6 Key Concepts (Remember These!)
```
1. SUBSCRIPTION   → Third operation type (after Query, Mutation)
2. WEBSOCKET      → Underlying transport for persistent connection
3. PUB/SUB        → Server-side event distribution mechanism
4. RESOLVER       → Subscribe function returns AsyncIterator
5. FILTER         → Client-side selection of specific events
6. GRAPHQL-WS     → Modern protocol (replaces subscriptions-transport-ws)
```

### GraphQL Operations Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│              GRAPHQL OPERATIONS                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  QUERY (Read)                                                   │
│  ─────────────                                                  │
│  Client ──GET──▶ Server                                        │
│  Client ◀──Response── Server                                   │
│  • Request-response                                            │
│  • Client initiates                                            │
│  • One-time fetch                                              │
│                                                                 │
│  MUTATION (Write)                                               │
│  ────────────────                                               │
│  Client ──POST──▶ Server                                       │
│  Client ◀──Response── Server                                   │
│  • Request-response                                            │
│  • Client initiates                                            │
│  • Modifies data                                               │
│                                                                 │
│  SUBSCRIPTION (Real-time)                                       │
│  ────────────────────────                                       │
│  Client ══WebSocket══ Server                                   │
│  Client ◀══Event 1════ Server                                  │
│  Client ◀══Event 2════ Server                                  │
│  Client ◀══Event 3════ Server                                  │
│  • Persistent connection                                       │
│  • Server pushes updates                                       │
│  • Real-time stream                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Subscription Flow
```
┌─────────────────────────────────────────────────────────────────┐
│                 SUBSCRIPTION FLOW                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. CLIENT SUBSCRIBES                                           │
│     subscription { messageAdded(roomId: "123") { text } }      │
│                        │                                        │
│                        ▼                                        │
│  2. SERVER CREATES ASYNCITERATOR                               │
│     subscribe: () => pubsub.asyncIterator('MESSAGE_ADDED')     │
│                        │                                        │
│                        ▼                                        │
│  3. EVENT OCCURS (e.g., Mutation)                              │
│     pubsub.publish('MESSAGE_ADDED', { messageAdded: msg })     │
│                        │                                        │
│                        ▼                                        │
│  4. SERVER SENDS TO SUBSCRIBER                                  │
│     { data: { messageAdded: { text: "Hello" } } }              │
│                        │                                        │
│                        ▼                                        │
│  5. CLIENT RECEIVES UPDATE                                      │
│     onData: (data) => updateUI(data)                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"AsyncIterator"** | "Subscription resolvers return an AsyncIterator from the PubSub" |
| **"graphql-ws"** | "We use the graphql-ws protocol, not the legacy subscriptions-transport-ws" |
| **"PubSub"** | "Events are distributed via PubSub - we use Redis PubSub for scaling" |
| **"Subscription filter"** | "We filter subscriptions server-side to reduce unnecessary traffic" |
| **"Connection params"** | "Authentication is passed via connectionParams during WebSocket handshake" |
| **"Horizontal scaling"** | "Redis PubSub enables subscription scaling across multiple server instances" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| WS connection overhead | **~2KB** | Per connection memory |
| Heartbeat interval | **30 seconds** | Keep connection alive |
| Connection timeout | **5 seconds** | Initial connection limit |
| Max subscriptions per client | **~100** | Practical limit |

### The "Wow" Statement (Memorize This!)
> "We implemented GraphQL Subscriptions using Apollo Server with the graphql-ws protocol over WebSockets. Authentication happens during connection initialization via connectionParams - we verify the JWT and attach user context. For scaling, we use Redis PubSub so events published on one server reach subscribers on all servers. We implement server-side filtering with `withFilter` to ensure users only receive events they're authorized to see. Each subscription type has a dedicated topic with the entity ID for efficient routing. We monitor active subscriptions per user to prevent abuse, and implement graceful degradation - if WebSocket fails, clients fall back to polling."

### Quick Architecture Drawing (Draw This!)
```
CLIENT                         SERVER
┌──────────────┐              ┌──────────────────────────────┐
│              │   WebSocket  │                              │
│  useSubscription ═══════════│  GraphQL Server              │
│              │              │  ┌────────────────────────┐  │
│  subscription {             │  │  Subscription Resolver │  │
│    messageAdded {           │  │  ───────────────────── │  │
│      text                   │  │  subscribe: () =>      │  │
│    }                        │  │    pubsub.asyncIter()  │  │
│  }                          │  └──────────┬─────────────┘  │
│              │              │             │                │
│              │              │             ▼                │
│              │              │  ┌────────────────────────┐  │
│              │              │  │     Redis PubSub       │  │
│              │◀═════════════│  │  (for multi-server)    │  │
│              │   events     │  └────────────────────────┘  │
└──────────────┘              └──────────────────────────────┘
                                           ▲
                                           │ publish()
                               ┌───────────┴───────────┐
                               │   Mutation Resolver   │
                               │   (or external event) │
                               └───────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What are GraphQL Subscriptions?"**
> "Third operation type for real-time data. Client subscribes, server pushes updates over WebSocket when relevant events occur."

**Q: "How do Subscriptions differ from Queries?"**
> "Queries are request-response, one-time fetch. Subscriptions are persistent - server pushes updates whenever data changes."

**Q: "What protocol do Subscriptions use?"**
> "WebSocket transport with graphql-ws protocol. Legacy apps used subscriptions-transport-ws but it's deprecated."

**Q: "How do you handle authentication?"**
> "Pass token via connectionParams during WebSocket handshake. Verify in onConnect callback, attach to context."

**Q: "How do you scale Subscriptions?"**
> "Use Redis PubSub instead of in-memory. Events published on any server reach all servers. Each server manages its own WebSocket connections."

**Q: "What is withFilter?"**
> "Server-side subscription filtering. Checks if published event should be sent to specific subscriber. Reduces unnecessary traffic."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "What are GraphQL Subscriptions?"

**Junior Answer:**
> "They let you get real-time updates from the server."

**Senior Answer:**
> "GraphQL Subscriptions are the third operation type alongside Query and Mutation. While queries and mutations follow request-response, subscriptions establish a persistent WebSocket connection where the server pushes updates to the client.

The flow:
1. Client sends subscription operation
2. Server creates an AsyncIterator that listens for events
3. When events occur (usually from mutations), server publishes to PubSub
4. AsyncIterator yields the event
5. Server sends GraphQL response over WebSocket
6. Client receives typed, validated data

Key benefits over raw WebSockets:
- Type safety from schema
- Only requested fields sent
- Built-in filtering
- Integrates with existing GraphQL tooling"

### When Asked: "How do you scale GraphQL Subscriptions?"

**Junior Answer:**
> "Add more servers."

**Senior Answer:**
> "Subscriptions are stateful - each client has a WebSocket to a specific server. Scaling requires coordination:

1. **PubSub Backend**: Replace in-memory PubSub with Redis. When any server publishes an event, all servers receive it.

2. **Connection Distribution**: Load balancer with sticky sessions (for HTTP upgrade), or use a dedicated WebSocket gateway.

3. **Subscription State**: Each server tracks its own subscribers. Event published → Redis → all servers → each checks if any local subscriber wants it.

4. **Resource Limits**: Monitor connections per server, implement per-user subscription limits.

5. **Graceful Degradation**: If WebSocket fails, clients can fall back to polling queries.

For very large scale, consider dedicated subscription servers separate from query/mutation servers."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "How do you authenticate subscriptions?" | "During WebSocket connection via connectionParams. Verify JWT in onConnect, reject invalid tokens. Token refresh requires reconnection." |
| "What about authorization per subscription?" | "Two levels: connection-level auth in onConnect, subscription-level in withFilter. Check if user can see specific resource." |
| "How do you handle reconnection?" | "graphql-ws handles it automatically with exponential backoff. Client resubscribes after reconnect. Track last event ID for gap recovery if needed." |
| "Subscriptions vs SSE?" | "Subscriptions are bidirectional (client can unsubscribe, send more subscriptions). SSE is simpler but one-way. Subscriptions integrate with GraphQL ecosystem." |

---

## 📚 Table of Contents

1. [Server Setup (Apollo Server)](#1-server-setup-apollo-server)
2. [Client Setup (Apollo Client)](#2-client-setup-apollo-client)
3. [Authentication & Authorization](#3-authentication--authorization)
4. [PubSub Patterns](#4-pubsub-patterns)
5. [Filtering Subscriptions](#5-filtering-subscriptions)
6. [Scaling with Redis](#6-scaling-with-redis)
7. [Error Handling](#7-error-handling)
8. [Real-World Examples](#8-real-world-examples)
9. [Common Pitfalls](#9-common-pitfalls)
10. [Interview Questions](#10-interview-questions)

---

## 1. Server Setup (Apollo Server)

```typescript
// ════════════════════════════════════════════════════════════════
// APOLLO SERVER 4 WITH GRAPHQL-WS
// ════════════════════════════════════════════════════════════════

import { ApolloServer } from '@apollo/server';
import { expressMiddleware } from '@apollo/server/express4';
import { createServer } from 'http';
import express from 'express';
import { WebSocketServer } from 'ws';
import { useServer } from 'graphql-ws/lib/use/ws';
import { makeExecutableSchema } from '@graphql-tools/schema';
import { PubSub } from 'graphql-subscriptions';

// ════════════════════════════════════════════════════════════════
// SCHEMA
// ════════════════════════════════════════════════════════════════

const typeDefs = `#graphql
  type Message {
    id: ID!
    text: String!
    userId: String!
    roomId: String!
    createdAt: String!
  }

  type User {
    id: ID!
    name: String!
    status: String!
  }

  type Query {
    messages(roomId: String!): [Message!]!
    user(id: ID!): User
  }

  type Mutation {
    sendMessage(roomId: String!, text: String!): Message!
    updateUserStatus(status: String!): User!
  }

  type Subscription {
    # New message in a room
    messageAdded(roomId: String!): Message!
    
    # User status changes
    userStatusChanged(userId: ID!): User!
    
    # Any message in any room (admin)
    allMessages: Message!
  }
`;

// ════════════════════════════════════════════════════════════════
// PUBSUB
// ════════════════════════════════════════════════════════════════

const pubsub = new PubSub();

// Event names
const EVENTS = {
  MESSAGE_ADDED: 'MESSAGE_ADDED',
  USER_STATUS_CHANGED: 'USER_STATUS_CHANGED'
};

// ════════════════════════════════════════════════════════════════
// RESOLVERS
// ════════════════════════════════════════════════════════════════

const resolvers = {
  Query: {
    messages: async (_: any, { roomId }: { roomId: string }, context: Context) => {
      return context.dataSources.messages.getByRoom(roomId);
    },
    user: async (_: any, { id }: { id: string }, context: Context) => {
      return context.dataSources.users.getById(id);
    }
  },

  Mutation: {
    sendMessage: async (
      _: any,
      { roomId, text }: { roomId: string; text: string },
      context: Context
    ) => {
      // Create message
      const message = await context.dataSources.messages.create({
        roomId,
        text,
        userId: context.user.id
      });

      // Publish to subscribers
      pubsub.publish(EVENTS.MESSAGE_ADDED, {
        messageAdded: message,
        roomId  // Include for filtering
      });

      return message;
    },

    updateUserStatus: async (
      _: any,
      { status }: { status: string },
      context: Context
    ) => {
      const user = await context.dataSources.users.updateStatus(
        context.user.id,
        status
      );

      // Publish status change
      pubsub.publish(EVENTS.USER_STATUS_CHANGED, {
        userStatusChanged: user
      });

      return user;
    }
  },

  Subscription: {
    messageAdded: {
      // Subscribe function returns AsyncIterator
      subscribe: (_: any, { roomId }: { roomId: string }) => {
        return pubsub.asyncIterator([EVENTS.MESSAGE_ADDED]);
      },
      // Filter: only send if roomId matches
      resolve: (payload: any) => payload.messageAdded
    },

    userStatusChanged: {
      subscribe: (_: any, { userId }: { userId: string }) => {
        return pubsub.asyncIterator([EVENTS.USER_STATUS_CHANGED]);
      },
      resolve: (payload: any) => payload.userStatusChanged
    },

    allMessages: {
      subscribe: (_: any, __: any, context: Context) => {
        // Check admin permission
        if (!context.user?.isAdmin) {
          throw new Error('Not authorized');
        }
        return pubsub.asyncIterator([EVENTS.MESSAGE_ADDED]);
      },
      resolve: (payload: any) => payload.messageAdded
    }
  }
};

// ════════════════════════════════════════════════════════════════
// CONTEXT TYPE
// ════════════════════════════════════════════════════════════════

interface Context {
  user: {
    id: string;
    isAdmin: boolean;
  } | null;
  dataSources: {
    messages: MessageDataSource;
    users: UserDataSource;
  };
}

// ════════════════════════════════════════════════════════════════
// SERVER SETUP
// ════════════════════════════════════════════════════════════════

async function startServer() {
  const app = express();
  const httpServer = createServer(app);

  // Create schema
  const schema = makeExecutableSchema({ typeDefs, resolvers });

  // Create WebSocket server
  const wsServer = new WebSocketServer({
    server: httpServer,
    path: '/graphql'
  });

  // Setup graphql-ws
  const serverCleanup = useServer(
    {
      schema,
      
      // Authentication during connection
      onConnect: async (ctx) => {
        const token = ctx.connectionParams?.authToken as string;
        
        if (!token) {
          throw new Error('Missing auth token');
        }

        try {
          const user = await verifyToken(token);
          return { user };  // Available in context
        } catch {
          throw new Error('Invalid auth token');
        }
      },

      // Build context for each subscription
      context: async (ctx) => {
        return {
          user: (ctx as any).user,
          dataSources: {
            messages: new MessageDataSource(),
            users: new UserDataSource()
          }
        };
      },

      onDisconnect: (ctx) => {
        console.log('Client disconnected');
      }
    },
    wsServer
  );

  // Create Apollo Server
  const server = new ApolloServer({
    schema,
    plugins: [
      // Proper shutdown
      {
        async serverWillStart() {
          return {
            async drainServer() {
              await serverCleanup.dispose();
            }
          };
        }
      }
    ]
  });

  await server.start();

  // Express middleware for queries/mutations
  app.use(
    '/graphql',
    express.json(),
    expressMiddleware(server, {
      context: async ({ req }) => ({
        user: await getUserFromRequest(req),
        dataSources: {
          messages: new MessageDataSource(),
          users: new UserDataSource()
        }
      })
    })
  );

  httpServer.listen(4000, () => {
    console.log('Server running at http://localhost:4000/graphql');
    console.log('Subscriptions at ws://localhost:4000/graphql');
  });
}

startServer();
```

---

## 2. Client Setup (Apollo Client)

```typescript
// ════════════════════════════════════════════════════════════════
// APOLLO CLIENT WITH SUBSCRIPTIONS
// ════════════════════════════════════════════════════════════════

import {
  ApolloClient,
  InMemoryCache,
  HttpLink,
  split
} from '@apollo/client';
import { GraphQLWsLink } from '@apollo/client/link/subscriptions';
import { getMainDefinition } from '@apollo/client/utilities';
import { createClient } from 'graphql-ws';

// ════════════════════════════════════════════════════════════════
// SETUP LINKS
// ════════════════════════════════════════════════════════════════

// HTTP link for queries and mutations
const httpLink = new HttpLink({
  uri: 'http://localhost:4000/graphql',
  headers: {
    authorization: `Bearer ${getToken()}`
  }
});

// WebSocket link for subscriptions
const wsLink = new GraphQLWsLink(
  createClient({
    url: 'ws://localhost:4000/graphql',
    
    // Authentication
    connectionParams: () => ({
      authToken: getToken()
    }),
    
    // Reconnection
    retryAttempts: Infinity,
    shouldRetry: () => true,
    
    // Connection lifecycle
    on: {
      connected: () => console.log('WebSocket connected'),
      closed: () => console.log('WebSocket closed'),
      error: (error) => console.error('WebSocket error:', error)
    }
  })
);

// Split based on operation type
const splitLink = split(
  ({ query }) => {
    const definition = getMainDefinition(query);
    return (
      definition.kind === 'OperationDefinition' &&
      definition.operation === 'subscription'
    );
  },
  wsLink,    // Subscriptions → WebSocket
  httpLink   // Queries/Mutations → HTTP
);

// ════════════════════════════════════════════════════════════════
// APOLLO CLIENT
// ════════════════════════════════════════════════════════════════

const client = new ApolloClient({
  link: splitLink,
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          messages: {
            // Merge new messages from subscriptions
            merge(existing = [], incoming) {
              return [...existing, ...incoming];
            }
          }
        }
      }
    }
  })
});

// ════════════════════════════════════════════════════════════════
// REACT HOOKS
// ════════════════════════════════════════════════════════════════

import { gql, useSubscription, useQuery } from '@apollo/client';

// Subscription definition
const MESSAGE_ADDED = gql`
  subscription MessageAdded($roomId: String!) {
    messageAdded(roomId: $roomId) {
      id
      text
      userId
      createdAt
    }
  }
`;

const MESSAGES_QUERY = gql`
  query Messages($roomId: String!) {
    messages(roomId: $roomId) {
      id
      text
      userId
      createdAt
    }
  }
`;

// ════════════════════════════════════════════════════════════════
// COMPONENT EXAMPLE
// ════════════════════════════════════════════════════════════════

function ChatRoom({ roomId }: { roomId: string }) {
  // Initial messages
  const { data, loading } = useQuery(MESSAGES_QUERY, {
    variables: { roomId }
  });

  // Subscribe to new messages
  const { data: subData, error: subError } = useSubscription(MESSAGE_ADDED, {
    variables: { roomId },
    onData: ({ data }) => {
      console.log('New message:', data.data?.messageAdded);
    },
    onError: (error) => {
      console.error('Subscription error:', error);
    }
  });

  // Combine query data with subscription updates
  const [messages, setMessages] = useState<Message[]>([]);

  useEffect(() => {
    if (data?.messages) {
      setMessages(data.messages);
    }
  }, [data]);

  useEffect(() => {
    if (subData?.messageAdded) {
      setMessages(prev => [...prev, subData.messageAdded]);
    }
  }, [subData]);

  if (loading) return <Loading />;

  return (
    <div className="chat-room">
      {messages.map(msg => (
        <Message key={msg.id} message={msg} />
      ))}
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// SUBSCRIPTION WITH CACHE UPDATE
// ════════════════════════════════════════════════════════════════

function ChatRoomWithCacheUpdate({ roomId }: { roomId: string }) {
  const { data, loading, subscribeToMore } = useQuery(MESSAGES_QUERY, {
    variables: { roomId }
  });

  useEffect(() => {
    const unsubscribe = subscribeToMore({
      document: MESSAGE_ADDED,
      variables: { roomId },
      updateQuery: (prev, { subscriptionData }) => {
        if (!subscriptionData.data) return prev;

        const newMessage = subscriptionData.data.messageAdded;

        // Check for duplicates
        if (prev.messages.some(m => m.id === newMessage.id)) {
          return prev;
        }

        return {
          ...prev,
          messages: [...prev.messages, newMessage]
        };
      }
    });

    return () => unsubscribe();
  }, [roomId, subscribeToMore]);

  if (loading) return <Loading />;

  return (
    <div className="chat-room">
      {data?.messages.map(msg => (
        <Message key={msg.id} message={msg} />
      ))}
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// MULTIPLE SUBSCRIPTIONS HOOK
// ════════════════════════════════════════════════════════════════

function useMultipleSubscriptions(roomIds: string[]) {
  const [messages, setMessages] = useState<Map<string, Message[]>>(new Map());

  useEffect(() => {
    const subscriptions: (() => void)[] = [];

    roomIds.forEach(roomId => {
      const observable = client.subscribe({
        query: MESSAGE_ADDED,
        variables: { roomId }
      });

      const subscription = observable.subscribe({
        next: ({ data }) => {
          if (data?.messageAdded) {
            setMessages(prev => {
              const roomMessages = prev.get(roomId) || [];
              const updated = new Map(prev);
              updated.set(roomId, [...roomMessages, data.messageAdded]);
              return updated;
            });
          }
        },
        error: (error) => {
          console.error(`Subscription error for room ${roomId}:`, error);
        }
      });

      subscriptions.push(() => subscription.unsubscribe());
    });

    return () => {
      subscriptions.forEach(unsub => unsub());
    };
  }, [roomIds.join(',')]);

  return messages;
}
```

---

## 3. Authentication & Authorization

```typescript
// ════════════════════════════════════════════════════════════════
// SUBSCRIPTION AUTHENTICATION
// ════════════════════════════════════════════════════════════════

import { useServer } from 'graphql-ws/lib/use/ws';
import jwt from 'jsonwebtoken';

// Server-side authentication
const serverCleanup = useServer(
  {
    schema,

    // ════════════════════════════════════════════════════════════
    // CONNECTION-LEVEL AUTHENTICATION
    // ════════════════════════════════════════════════════════════

    onConnect: async (ctx) => {
      console.log('Client connecting...');
      
      // Get token from connectionParams
      const token = ctx.connectionParams?.authToken as string;
      
      if (!token) {
        // Reject connection
        throw new Error('Authentication required');
      }

      try {
        // Verify JWT
        const decoded = jwt.verify(token, process.env.JWT_SECRET!) as {
          userId: string;
          roles: string[];
        };

        // Load user
        const user = await db.users.findUnique({
          where: { id: decoded.userId }
        });

        if (!user || user.disabled) {
          throw new Error('User not found or disabled');
        }

        // Return context additions
        return {
          user: {
            id: user.id,
            roles: decoded.roles,
            isAdmin: decoded.roles.includes('admin')
          }
        };

      } catch (error) {
        console.error('Auth failed:', error);
        throw new Error('Invalid authentication token');
      }
    },

    // ════════════════════════════════════════════════════════════
    // SUBSCRIPTION-LEVEL AUTHORIZATION
    // ════════════════════════════════════════════════════════════

    onSubscribe: async (ctx, msg) => {
      const user = (ctx as any).user;
      const { operationName, variables } = msg.payload;

      console.log(`User ${user?.id} subscribing to ${operationName}`);

      // Example: Check room membership
      if (operationName === 'MessageAdded') {
        const roomId = variables?.roomId;
        
        if (roomId) {
          const isMember = await checkRoomMembership(user.id, roomId);
          
          if (!isMember) {
            throw new Error('Not authorized for this room');
          }
        }
      }

      // Return void to allow, throw to reject
    },

    onOperation: (ctx, msg, args, result) => {
      // Can modify execution args
      return result;
    },

    onNext: (ctx, msg, args, result) => {
      // Called for each subscription event
      // Can filter or transform results
    },

    onError: (ctx, msg, errors) => {
      console.error('Subscription error:', errors);
    },

    onComplete: (ctx, msg) => {
      console.log('Subscription completed');
    },

    context: async (ctx) => ({
      user: (ctx as any).user,
      dataSources: createDataSources()
    })
  },
  wsServer
);

// ════════════════════════════════════════════════════════════════
// RESOLVER-LEVEL AUTHORIZATION
// ════════════════════════════════════════════════════════════════

import { withFilter } from 'graphql-subscriptions';

const resolvers = {
  Subscription: {
    messageAdded: {
      subscribe: withFilter(
        // Base iterator
        () => pubsub.asyncIterator([EVENTS.MESSAGE_ADDED]),
        
        // Filter function - runs for EACH event
        async (payload, variables, context) => {
          const { roomId } = variables;
          const { user } = context;

          // Check if this event is for the subscribed room
          if (payload.roomId !== roomId) {
            return false;
          }

          // Check if user has access to this room
          const hasAccess = await checkRoomMembership(user.id, roomId);
          
          return hasAccess;
        }
      )
    },

    // Admin-only subscription
    allMessages: {
      subscribe: (_, __, context) => {
        if (!context.user?.isAdmin) {
          throw new Error('Admin access required');
        }
        return pubsub.asyncIterator([EVENTS.MESSAGE_ADDED]);
      },
      resolve: (payload) => payload.messageAdded
    }
  }
};

// ════════════════════════════════════════════════════════════════
// CLIENT-SIDE AUTH WITH TOKEN REFRESH
// ════════════════════════════════════════════════════════════════

import { createClient } from 'graphql-ws';

let wsClient: ReturnType<typeof createClient> | null = null;

function createWsClient(token: string) {
  // Close existing client
  if (wsClient) {
    wsClient.dispose();
  }

  wsClient = createClient({
    url: 'ws://localhost:4000/graphql',
    connectionParams: {
      authToken: token
    },
    retryAttempts: 5,
    shouldRetry: (errOrCloseEvent) => {
      // Don't retry on auth errors
      if (errOrCloseEvent instanceof CloseEvent) {
        return errOrCloseEvent.code !== 4401;  // Custom auth error code
      }
      return true;
    },
    on: {
      error: (error) => {
        if (error.message?.includes('auth')) {
          // Token expired, try refresh
          refreshTokenAndReconnect();
        }
      }
    }
  });

  return wsClient;
}

async function refreshTokenAndReconnect() {
  try {
    const newToken = await refreshAuthToken();
    createWsClient(newToken);
    // Re-subscribe to active subscriptions
    resubscribeAll();
  } catch {
    // Redirect to login
    window.location.href = '/login';
  }
}
```

---

## 4. PubSub Patterns

```typescript
// ════════════════════════════════════════════════════════════════
// PUBSUB PATTERNS
// ════════════════════════════════════════════════════════════════

import { PubSub } from 'graphql-subscriptions';

// ════════════════════════════════════════════════════════════════
// TOPIC NAMING CONVENTIONS
// ════════════════════════════════════════════════════════════════

const TOPICS = {
  // Entity-based topics
  MESSAGE: {
    ADDED: 'MESSAGE_ADDED',
    UPDATED: 'MESSAGE_UPDATED',
    DELETED: 'MESSAGE_DELETED'
  },
  
  // Scoped topics (include ID for filtering)
  roomMessage: (roomId: string) => `MESSAGE_ADDED_${roomId}`,
  userStatus: (userId: string) => `USER_STATUS_${userId}`,
  
  // Wildcard topics
  ALL_MESSAGES: 'ALL_MESSAGES'
};

// ════════════════════════════════════════════════════════════════
// TYPED PUBSUB WRAPPER
// ════════════════════════════════════════════════════════════════

interface PubSubEvents {
  MESSAGE_ADDED: { messageAdded: Message; roomId: string };
  MESSAGE_UPDATED: { messageUpdated: Message };
  MESSAGE_DELETED: { messageDeleted: { id: string; roomId: string } };
  USER_STATUS_CHANGED: { userStatusChanged: User };
  TYPING_STARTED: { typingStarted: { userId: string; roomId: string } };
  TYPING_STOPPED: { typingStopped: { userId: string; roomId: string } };
}

class TypedPubSub {
  private pubsub = new PubSub();

  publish<K extends keyof PubSubEvents>(
    event: K,
    payload: PubSubEvents[K]
  ): Promise<void> {
    return this.pubsub.publish(event, payload);
  }

  asyncIterator<K extends keyof PubSubEvents>(
    events: K | K[]
  ) {
    return this.pubsub.asyncIterator(Array.isArray(events) ? events : [events]);
  }
}

const pubsub = new TypedPubSub();

// ════════════════════════════════════════════════════════════════
// PUBLISH ON MUTATION
// ════════════════════════════════════════════════════════════════

const resolvers = {
  Mutation: {
    sendMessage: async (_, { roomId, text }, context) => {
      const message = await context.dataSources.messages.create({
        roomId,
        text,
        userId: context.user.id
      });

      // Publish to general topic
      await pubsub.publish('MESSAGE_ADDED', {
        messageAdded: message,
        roomId
      });

      return message;
    },

    updateMessage: async (_, { id, text }, context) => {
      const message = await context.dataSources.messages.update(id, { text });

      await pubsub.publish('MESSAGE_UPDATED', {
        messageUpdated: message
      });

      return message;
    },

    deleteMessage: async (_, { id }, context) => {
      const message = await context.dataSources.messages.findById(id);
      await context.dataSources.messages.delete(id);

      await pubsub.publish('MESSAGE_DELETED', {
        messageDeleted: { id, roomId: message.roomId }
      });

      return true;
    }
  }
};

// ════════════════════════════════════════════════════════════════
// PUBLISH FROM EXTERNAL EVENTS
// ════════════════════════════════════════════════════════════════

// Webhook handler
app.post('/webhooks/payment', async (req, res) => {
  const { orderId, status } = req.body;

  // Publish to GraphQL subscriptions
  await pubsub.publish('PAYMENT_STATUS', {
    paymentStatus: { orderId, status }
  });

  res.json({ received: true });
});

// Database trigger simulation
class MessageService {
  async onMessageCreated(message: Message) {
    // Called by database trigger or ORM hook
    await pubsub.publish('MESSAGE_ADDED', {
      messageAdded: message,
      roomId: message.roomId
    });
  }
}

// ════════════════════════════════════════════════════════════════
// BATCHED PUBLISHING
// ════════════════════════════════════════════════════════════════

class BatchedPubSub {
  private queue: Map<string, any[]> = new Map();
  private flushTimeout: NodeJS.Timeout | null = null;
  private readonly batchMs = 100;

  constructor(private pubsub: PubSub) {}

  queuePublish(topic: string, payload: any) {
    if (!this.queue.has(topic)) {
      this.queue.set(topic, []);
    }
    this.queue.get(topic)!.push(payload);

    if (!this.flushTimeout) {
      this.flushTimeout = setTimeout(() => this.flush(), this.batchMs);
    }
  }

  private async flush() {
    this.flushTimeout = null;

    for (const [topic, payloads] of this.queue) {
      // Publish batch as array
      await this.pubsub.publish(topic, {
        [`${topic.toLowerCase()}Batch`]: payloads
      });
    }

    this.queue.clear();
  }
}
```

---

## 5. Filtering Subscriptions

```typescript
// ════════════════════════════════════════════════════════════════
// SUBSCRIPTION FILTERING WITH withFilter
// ════════════════════════════════════════════════════════════════

import { withFilter } from 'graphql-subscriptions';

const resolvers = {
  Subscription: {
    // ══════════════════════════════════════════════════════════════
    // SIMPLE FILTER - By argument
    // ══════════════════════════════════════════════════════════════
    
    messageAdded: {
      subscribe: withFilter(
        () => pubsub.asyncIterator('MESSAGE_ADDED'),
        (payload, variables) => {
          // Only send if roomId matches
          return payload.roomId === variables.roomId;
        }
      ),
      resolve: (payload) => payload.messageAdded
    },

    // ══════════════════════════════════════════════════════════════
    // ASYNC FILTER - With database lookup
    // ══════════════════════════════════════════════════════════════
    
    messageAddedWithAuth: {
      subscribe: withFilter(
        () => pubsub.asyncIterator('MESSAGE_ADDED'),
        async (payload, variables, context) => {
          const { roomId } = variables;
          const { user } = context;

          // Check subscription roomId matches
          if (payload.roomId !== roomId) {
            return false;
          }

          // Check user has access to room
          const membership = await db.roomMembers.findFirst({
            where: {
              roomId,
              userId: user.id
            }
          });

          return !!membership;
        }
      )
    },

    // ══════════════════════════════════════════════════════════════
    // FILTER BY USER
    // ══════════════════════════════════════════════════════════════
    
    myNotifications: {
      subscribe: withFilter(
        () => pubsub.asyncIterator('NOTIFICATION'),
        (payload, _, context) => {
          // Only send notifications for this user
          return payload.notification.userId === context.user.id;
        }
      ),
      resolve: (payload) => payload.notification
    },

    // ══════════════════════════════════════════════════════════════
    // MULTIPLE CONDITIONS
    // ══════════════════════════════════════════════════════════════
    
    activityFeed: {
      subscribe: withFilter(
        () => pubsub.asyncIterator(['POST_CREATED', 'COMMENT_ADDED', 'LIKE_ADDED']),
        async (payload, variables, context) => {
          const { feedType, userId } = variables;

          // Filter by type
          if (feedType === 'personal' && payload.userId !== userId) {
            return false;
          }

          // Filter by following
          if (feedType === 'following') {
            const isFollowing = await context.dataSources.users.isFollowing(
              context.user.id,
              payload.userId
            );
            if (!isFollowing) return false;
          }

          // Filter by blocked users
          const isBlocked = await context.dataSources.users.isBlocked(
            context.user.id,
            payload.userId
          );
          if (isBlocked) return false;

          return true;
        }
      ),
      resolve: (payload) => {
        // Normalize different event types
        if (payload.postCreated) return { type: 'post', data: payload.postCreated };
        if (payload.commentAdded) return { type: 'comment', data: payload.commentAdded };
        if (payload.likeAdded) return { type: 'like', data: payload.likeAdded };
      }
    },

    // ══════════════════════════════════════════════════════════════
    // OPTIMIZED - Use topic per room instead of filtering
    // ══════════════════════════════════════════════════════════════
    
    messageAddedOptimized: {
      subscribe: (_, { roomId }) => {
        // Subscribe to room-specific topic
        // No filtering needed - topic is already scoped
        return pubsub.asyncIterator(`MESSAGE_ADDED_${roomId}`);
      },
      resolve: (payload) => payload.messageAdded
    }
  }
};

// When publishing with optimized approach:
async function publishMessage(message: Message) {
  // Publish to room-specific topic
  await pubsub.publish(`MESSAGE_ADDED_${message.roomId}`, {
    messageAdded: message
  });

  // Also publish to admin topic for monitoring
  await pubsub.publish('ALL_MESSAGES', {
    messageAdded: message
  });
}

// ════════════════════════════════════════════════════════════════
// DYNAMIC SUBSCRIPTIONS
// ════════════════════════════════════════════════════════════════

// Subscribe to multiple topics dynamically
const resolvers = {
  Subscription: {
    roomUpdates: {
      subscribe: async (_, { roomIds }, context) => {
        // Verify access to all rooms
        for (const roomId of roomIds) {
          const hasAccess = await checkRoomMembership(context.user.id, roomId);
          if (!hasAccess) {
            throw new Error(`No access to room ${roomId}`);
          }
        }

        // Subscribe to all room topics
        const topics = roomIds.map(id => `ROOM_UPDATE_${id}`);
        return pubsub.asyncIterator(topics);
      }
    }
  }
};
```

---

## 6. Scaling with Redis

```typescript
// ════════════════════════════════════════════════════════════════
// REDIS PUBSUB FOR SCALING SUBSCRIPTIONS
// ════════════════════════════════════════════════════════════════

import { RedisPubSub } from 'graphql-redis-subscriptions';
import Redis from 'ioredis';

// ════════════════════════════════════════════════════════════════
// REDIS PUBSUB SETUP
// ════════════════════════════════════════════════════════════════

const options: Redis.RedisOptions = {
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT || '6379'),
  password: process.env.REDIS_PASSWORD,
  retryStrategy: (times) => Math.min(times * 50, 2000)
};

const pubsub = new RedisPubSub({
  publisher: new Redis(options),
  subscriber: new Redis(options),
  
  // Optional: Custom serializer
  serializer: (source) => JSON.stringify(source),
  deserializer: (message) => JSON.parse(message)
});

// ════════════════════════════════════════════════════════════════
// HOW IT WORKS
// ════════════════════════════════════════════════════════════════

/*
SERVER A (has Client 1, Client 2)
┌──────────────────────────────────────┐
│  Mutation: sendMessage()             │
│         │                            │
│         ▼                            │
│  pubsub.publish('MESSAGE_ADDED')     │
│         │                            │
└─────────┼────────────────────────────┘
          │
          ▼
    ┌─────────────┐
    │    REDIS    │
    │   PUB/SUB   │
    └─────────────┘
          │
    ┌─────┴─────┐
    ▼           ▼
SERVER A    SERVER B (has Client 3, Client 4)
(notifies   (notifies
 1 & 2)      3 & 4)
*/

// ════════════════════════════════════════════════════════════════
// USAGE WITH RESOLVERS
// ════════════════════════════════════════════════════════════════

const resolvers = {
  Mutation: {
    sendMessage: async (_, { roomId, text }, context) => {
      const message = await db.messages.create({
        data: {
          roomId,
          text,
          userId: context.user.id
        }
      });

      // Publishes to Redis, all server instances receive
      await pubsub.publish('MESSAGE_ADDED', {
        messageAdded: message,
        roomId
      });

      return message;
    }
  },

  Subscription: {
    messageAdded: {
      subscribe: withFilter(
        () => pubsub.asyncIterator('MESSAGE_ADDED'),
        (payload, variables) => payload.roomId === variables.roomId
      ),
      resolve: (payload) => payload.messageAdded
    }
  }
};

// ════════════════════════════════════════════════════════════════
// ADVANCED: REDIS STREAMS FOR DURABILITY
// ════════════════════════════════════════════════════════════════

import Redis from 'ioredis';

class DurableSubscriptionPubSub {
  private redis: Redis;
  private subscriber: Redis;
  private handlers: Map<string, Set<(data: any) => void>> = new Map();

  constructor(redisUrl: string) {
    this.redis = new Redis(redisUrl);
    this.subscriber = new Redis(redisUrl);
  }

  async publish(topic: string, payload: any): Promise<string> {
    // Add to Redis Stream for durability
    const messageId = await this.redis.xadd(
      `subscription:${topic}`,
      'MAXLEN', '~', '10000',  // Keep ~10k messages
      '*',
      'payload', JSON.stringify(payload)
    );

    // Also publish to pub/sub for real-time
    await this.redis.publish(topic, JSON.stringify({ ...payload, messageId }));

    return messageId;
  }

  async subscribe(topic: string, handler: (data: any) => void, lastMessageId?: string) {
    // First, catch up from last known message
    if (lastMessageId) {
      const messages = await this.redis.xrange(
        `subscription:${topic}`,
        lastMessageId,
        '+'
      );

      for (const [id, fields] of messages) {
        if (id !== lastMessageId) {  // Skip the message we already have
          const payload = JSON.parse(fields[1]);
          handler({ ...payload, messageId: id });
        }
      }
    }

    // Then subscribe to real-time updates
    if (!this.handlers.has(topic)) {
      this.handlers.set(topic, new Set());
      await this.subscriber.subscribe(topic);
    }

    this.handlers.get(topic)!.add(handler);

    // Handle incoming messages
    this.subscriber.on('message', (channel, message) => {
      if (channel === topic) {
        const handlers = this.handlers.get(topic);
        if (handlers) {
          const data = JSON.parse(message);
          handlers.forEach(h => h(data));
        }
      }
    });

    // Return unsubscribe function
    return () => {
      this.handlers.get(topic)?.delete(handler);
    };
  }
}

// ════════════════════════════════════════════════════════════════
// CONNECTION MANAGEMENT FOR SCALE
// ════════════════════════════════════════════════════════════════

interface ConnectionInfo {
  userId: string;
  subscriptions: Set<string>;
  connectedAt: Date;
}

class SubscriptionConnectionManager {
  private connections: Map<string, ConnectionInfo> = new Map();
  private redis: Redis;

  constructor(redisUrl: string) {
    this.redis = new Redis(redisUrl);
  }

  async onConnect(connectionId: string, userId: string) {
    this.connections.set(connectionId, {
      userId,
      subscriptions: new Set(),
      connectedAt: new Date()
    });

    // Track in Redis for cross-server visibility
    await this.redis.hset(
      `subscriptions:user:${userId}`,
      connectionId,
      JSON.stringify({
        serverId: process.env.SERVER_ID,
        connectedAt: new Date().toISOString()
      })
    );

    // Update user presence
    await this.redis.set(`presence:${userId}`, 'online', 'EX', 60);
  }

  async onSubscribe(connectionId: string, topic: string) {
    const conn = this.connections.get(connectionId);
    if (conn) {
      conn.subscriptions.add(topic);

      // Track subscription in Redis
      await this.redis.sadd(`topic:${topic}:subscribers`, conn.userId);
    }
  }

  async onDisconnect(connectionId: string) {
    const conn = this.connections.get(connectionId);
    if (!conn) return;

    // Clean up subscriptions
    for (const topic of conn.subscriptions) {
      await this.redis.srem(`topic:${topic}:subscribers`, conn.userId);
    }

    // Remove connection tracking
    await this.redis.hdel(`subscriptions:user:${conn.userId}`, connectionId);

    // Check if user has other connections
    const otherConnections = await this.redis.hlen(`subscriptions:user:${conn.userId}`);
    if (otherConnections === 0) {
      await this.redis.del(`presence:${conn.userId}`);
    }

    this.connections.delete(connectionId);
  }

  async getOnlineUsers(): Promise<string[]> {
    const keys = await this.redis.keys('presence:*');
    return keys.map(k => k.replace('presence:', ''));
  }

  async getTopicSubscriberCount(topic: string): Promise<number> {
    return this.redis.scard(`topic:${topic}:subscribers`);
  }
}
```

---

## 7. Error Handling

```typescript
// ════════════════════════════════════════════════════════════════
// SUBSCRIPTION ERROR HANDLING
// ════════════════════════════════════════════════════════════════

// Server-side error handling
const serverCleanup = useServer(
  {
    schema,

    onConnect: async (ctx) => {
      try {
        const token = ctx.connectionParams?.authToken;
        if (!token) {
          // Return false to reject connection silently
          // Or throw to send error message
          throw new Error('Authentication required');
        }
        const user = await verifyToken(token);
        return { user };
      } catch (error) {
        console.error('Connection error:', error);
        throw error;  // Sends error to client
      }
    },

    onSubscribe: (ctx, msg) => {
      console.log('Subscribe:', msg.payload.operationName);
    },

    onError: (ctx, msg, errors) => {
      console.error('Subscription execution error:', errors);
      // errors is array of GraphQL errors
    },

    onComplete: (ctx, msg) => {
      console.log('Subscription completed');
    }
  },
  wsServer
);

// ════════════════════════════════════════════════════════════════
// RESOLVER ERROR HANDLING
// ════════════════════════════════════════════════════════════════

const resolvers = {
  Subscription: {
    messageAdded: {
      subscribe: async function* (_, { roomId }, context) {
        // Validate access
        const hasAccess = await checkRoomMembership(context.user.id, roomId);
        if (!hasAccess) {
          throw new GraphQLError('Not authorized for this room', {
            extensions: { code: 'FORBIDDEN' }
          });
        }

        // Wrap iterator with error handling
        const iterator = pubsub.asyncIterator('MESSAGE_ADDED');

        try {
          for await (const event of iterator) {
            if (event.roomId === roomId) {
              yield event;
            }
          }
        } catch (error) {
          console.error('Subscription error:', error);
          throw new GraphQLError('Subscription failed', {
            extensions: { code: 'INTERNAL_ERROR' }
          });
        }
      }
    }
  }
};

// ════════════════════════════════════════════════════════════════
// CLIENT-SIDE ERROR HANDLING
// ════════════════════════════════════════════════════════════════

import { useSubscription } from '@apollo/client';

function ChatRoom({ roomId }: { roomId: string }) {
  const { data, loading, error } = useSubscription(MESSAGE_ADDED, {
    variables: { roomId },
    
    onError: (error) => {
      console.error('Subscription error:', error);
      
      // Handle specific errors
      if (error.message.includes('FORBIDDEN')) {
        showToast('You do not have access to this room');
        redirectToHome();
      } else if (error.message.includes('auth')) {
        // Token expired
        refreshToken().then(() => {
          // Will auto-reconnect
        }).catch(() => {
          redirectToLogin();
        });
      }
    },
    
    onComplete: () => {
      console.log('Subscription completed');
    },
    
    shouldResubscribe: true  // Auto resubscribe on error
  });

  if (error) {
    return <ErrorMessage error={error} />;
  }

  // ...
}

// ════════════════════════════════════════════════════════════════
// GRAPHQL-WS CLIENT ERROR HANDLING
// ════════════════════════════════════════════════════════════════

import { createClient, CloseCode } from 'graphql-ws';

const client = createClient({
  url: 'ws://localhost:4000/graphql',
  connectionParams: () => ({
    authToken: getToken()
  }),
  
  on: {
    connected: () => {
      console.log('Connected');
      hideReconnectingUI();
    },
    
    closed: (event) => {
      console.log('Closed:', event.code, event.reason);
      
      if (event.code === CloseCode.Forbidden) {
        // Auth error
        showAuthError();
      }
    },
    
    error: (error) => {
      console.error('Connection error:', error);
    }
  },
  
  retryAttempts: 5,
  
  shouldRetry: (errOrCloseEvent) => {
    if (errOrCloseEvent instanceof CloseEvent) {
      // Don't retry on auth errors
      if (errOrCloseEvent.code === 4401 || 
          errOrCloseEvent.code === 4403) {
        return false;
      }
    }
    return true;
  },
  
  retryWait: async (retries) => {
    // Exponential backoff
    const delay = Math.min(1000 * Math.pow(2, retries), 30000);
    await new Promise(resolve => setTimeout(resolve, delay));
  }
});

// ════════════════════════════════════════════════════════════════
// GRACEFUL DEGRADATION
// ════════════════════════════════════════════════════════════════

function useMessageWithFallback(roomId: string) {
  const [messages, setMessages] = useState<Message[]>([]);
  const [usingPolling, setUsingPolling] = useState(false);

  // Try subscription first
  const { error: subError } = useSubscription(MESSAGE_ADDED, {
    variables: { roomId },
    skip: usingPolling,
    onData: ({ data }) => {
      if (data.data?.messageAdded) {
        setMessages(prev => [...prev, data.data.messageAdded]);
      }
    },
    onError: () => {
      console.log('Subscription failed, falling back to polling');
      setUsingPolling(true);
    }
  });

  // Fallback to polling
  const { data: pollData } = useQuery(MESSAGES_QUERY, {
    variables: { roomId },
    pollInterval: usingPolling ? 3000 : 0,  // Poll every 3s
    skip: !usingPolling
  });

  useEffect(() => {
    if (pollData?.messages) {
      setMessages(pollData.messages);
    }
  }, [pollData]);

  return { messages, isPolling: usingPolling };
}
```

---

## 8. Real-World Examples

### Real-Time Chat Application

```typescript
// ════════════════════════════════════════════════════════════════
// CHAT APPLICATION SCHEMA
// ════════════════════════════════════════════════════════════════

const typeDefs = gql`
  type Message {
    id: ID!
    text: String!
    user: User!
    room: Room!
    createdAt: DateTime!
    updatedAt: DateTime
    readBy: [User!]!
  }

  type Room {
    id: ID!
    name: String!
    members: [User!]!
    messages(limit: Int, before: ID): [Message!]!
    typingUsers: [User!]!
  }

  type User {
    id: ID!
    name: String!
    avatar: String
    status: UserStatus!
  }

  enum UserStatus {
    ONLINE
    AWAY
    OFFLINE
  }

  type Query {
    room(id: ID!): Room
    myRooms: [Room!]!
  }

  type Mutation {
    sendMessage(roomId: ID!, text: String!): Message!
    markMessageRead(messageId: ID!): Message!
    startTyping(roomId: ID!): Boolean!
    stopTyping(roomId: ID!): Boolean!
    updateStatus(status: UserStatus!): User!
  }

  type Subscription {
    messageAdded(roomId: ID!): Message!
    messageRead(roomId: ID!): MessageReadPayload!
    typingUpdated(roomId: ID!): TypingPayload!
    userStatusChanged(userId: ID!): User!
    roomUpdated(roomId: ID!): Room!
  }

  type MessageReadPayload {
    messageId: ID!
    user: User!
  }

  type TypingPayload {
    roomId: ID!
    typingUsers: [User!]!
  }
`;

// ════════════════════════════════════════════════════════════════
// CHAT RESOLVERS
// ════════════════════════════════════════════════════════════════

const resolvers = {
  Mutation: {
    sendMessage: async (_, { roomId, text }, { user, pubsub, dataSources }) => {
      // Verify membership
      const room = await dataSources.rooms.findById(roomId);
      if (!room.memberIds.includes(user.id)) {
        throw new ForbiddenError('Not a member of this room');
      }

      // Create message
      const message = await dataSources.messages.create({
        roomId,
        text,
        userId: user.id
      });

      // Load full message with relations
      const fullMessage = await dataSources.messages.findById(message.id);

      // Stop typing indicator
      await dataSources.rooms.removeTyping(roomId, user.id);

      // Publish events
      await pubsub.publish(`MESSAGE_ADDED_${roomId}`, {
        messageAdded: fullMessage
      });

      await pubsub.publish(`TYPING_${roomId}`, {
        typingUpdated: {
          roomId,
          typingUsers: await dataSources.rooms.getTypingUsers(roomId)
        }
      });

      return fullMessage;
    },

    startTyping: async (_, { roomId }, { user, pubsub, dataSources }) => {
      await dataSources.rooms.addTyping(roomId, user.id);

      await pubsub.publish(`TYPING_${roomId}`, {
        typingUpdated: {
          roomId,
          typingUsers: await dataSources.rooms.getTypingUsers(roomId)
        }
      });

      // Auto-stop after 5 seconds
      setTimeout(async () => {
        await dataSources.rooms.removeTyping(roomId, user.id);
        await pubsub.publish(`TYPING_${roomId}`, {
          typingUpdated: {
            roomId,
            typingUsers: await dataSources.rooms.getTypingUsers(roomId)
          }
        });
      }, 5000);

      return true;
    }
  },

  Subscription: {
    messageAdded: {
      subscribe: async (_, { roomId }, { user, pubsub, dataSources }) => {
        // Verify membership
        const room = await dataSources.rooms.findById(roomId);
        if (!room.memberIds.includes(user.id)) {
          throw new ForbiddenError('Not a member of this room');
        }

        return pubsub.asyncIterator(`MESSAGE_ADDED_${roomId}`);
      }
    },

    typingUpdated: {
      subscribe: async (_, { roomId }, { user, pubsub, dataSources }) => {
        const room = await dataSources.rooms.findById(roomId);
        if (!room.memberIds.includes(user.id)) {
          throw new ForbiddenError('Not a member of this room');
        }

        return pubsub.asyncIterator(`TYPING_${roomId}`);
      }
    },

    userStatusChanged: {
      subscribe: (_, { userId }, { pubsub }) => {
        return pubsub.asyncIterator(`USER_STATUS_${userId}`);
      }
    }
  }
};

// ════════════════════════════════════════════════════════════════
// CHAT CLIENT COMPONENT
// ════════════════════════════════════════════════════════════════

function ChatRoom({ roomId }: { roomId: string }) {
  const [messages, setMessages] = useState<Message[]>([]);
  const [typingUsers, setTypingUsers] = useState<User[]>([]);
  const typingTimeoutRef = useRef<NodeJS.Timeout>();

  // Load initial messages
  const { data } = useQuery(ROOM_QUERY, {
    variables: { roomId }
  });

  useEffect(() => {
    if (data?.room?.messages) {
      setMessages(data.room.messages);
    }
  }, [data]);

  // Subscribe to new messages
  useSubscription(MESSAGE_ADDED_SUBSCRIPTION, {
    variables: { roomId },
    onData: ({ data }) => {
      if (data.data?.messageAdded) {
        setMessages(prev => [...prev, data.data.messageAdded]);
      }
    }
  });

  // Subscribe to typing indicators
  useSubscription(TYPING_SUBSCRIPTION, {
    variables: { roomId },
    onData: ({ data }) => {
      if (data.data?.typingUpdated) {
        setTypingUsers(data.data.typingUpdated.typingUsers);
      }
    }
  });

  // Send message
  const [sendMessage] = useMutation(SEND_MESSAGE_MUTATION);
  
  const handleSend = async (text: string) => {
    await sendMessage({ variables: { roomId, text } });
  };

  // Typing indicator
  const [startTyping] = useMutation(START_TYPING_MUTATION);
  
  const handleInputChange = () => {
    // Debounce typing indicator
    if (typingTimeoutRef.current) {
      clearTimeout(typingTimeoutRef.current);
    } else {
      startTyping({ variables: { roomId } });
    }
    
    typingTimeoutRef.current = setTimeout(() => {
      typingTimeoutRef.current = undefined;
    }, 3000);
  };

  return (
    <div className="chat-room">
      <MessageList messages={messages} />
      
      {typingUsers.length > 0 && (
        <TypingIndicator users={typingUsers} />
      )}
      
      <MessageInput 
        onSend={handleSend}
        onChange={handleInputChange}
      />
    </div>
  );
}
```

---

## 9. Common Pitfalls

```typescript
// ════════════════════════════════════════════════════════════════
// GRAPHQL SUBSCRIPTION PITFALLS
// ════════════════════════════════════════════════════════════════

// ❌ PITFALL 1: Using in-memory PubSub in production
// Problem: Events don't reach subscribers on other servers

// Bad
const pubsub = new PubSub();  // In-memory only

// Good
import { RedisPubSub } from 'graphql-redis-subscriptions';
const pubsub = new RedisPubSub({ ... });

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 2: Not filtering subscriptions server-side
// Problem: All events sent to all subscribers, waste bandwidth

// Bad
subscribe: () => pubsub.asyncIterator('MESSAGE_ADDED')
// Every message sent to every subscriber, filtered client-side

// Good
subscribe: withFilter(
  () => pubsub.asyncIterator('MESSAGE_ADDED'),
  (payload, variables) => payload.roomId === variables.roomId
)
// Only matching messages sent

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 3: Heavy filtering functions
// Problem: Async filter called for every event × every subscriber

// Bad
subscribe: withFilter(
  () => pubsub.asyncIterator('MESSAGE'),
  async (payload, variables, context) => {
    // Database query for EVERY event
    const room = await db.rooms.findUnique({ where: { id: payload.roomId } });
    return room.memberIds.includes(context.user.id);
  }
)

// Good - Use room-specific topics
subscribe: async (_, { roomId }, context) => {
  // Verify once at subscription time
  const room = await db.rooms.findUnique({ where: { id: roomId } });
  if (!room.memberIds.includes(context.user.id)) {
    throw new Error('Not a member');
  }
  return pubsub.asyncIterator(`MESSAGE_${roomId}`);
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 4: Not handling disconnection
// Problem: Memory leaks, stale subscriptions

// Bad
useServer({
  // No onDisconnect handler
})

// Good
useServer({
  onDisconnect: async (ctx) => {
    const user = ctx.extra?.user;
    if (user) {
      await cleanupUserSubscriptions(user.id);
      await updateUserPresence(user.id, 'offline');
    }
  }
})

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 5: Using legacy subscriptions-transport-ws
// Problem: Deprecated, has bugs, no maintenance

// Bad
import { SubscriptionServer } from 'subscriptions-transport-ws';

// Good
import { useServer } from 'graphql-ws/lib/use/ws';

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 6: Not implementing authorization per subscription
// Problem: Users can subscribe to data they shouldn't see

// Bad
subscribe: () => pubsub.asyncIterator('PRIVATE_DATA')
// Anyone can subscribe!

// Good
subscribe: async (_, { resourceId }, context) => {
  const hasAccess = await checkAccess(context.user.id, resourceId);
  if (!hasAccess) {
    throw new ForbiddenError('Access denied');
  }
  return pubsub.asyncIterator(`PRIVATE_DATA_${resourceId}`);
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 7: Not handling reconnection on client
// Problem: Lost updates after network issues

// Bad
const wsLink = new GraphQLWsLink(createClient({
  url: 'ws://...',
  // No reconnection config
}));

// Good
const wsLink = new GraphQLWsLink(createClient({
  url: 'ws://...',
  retryAttempts: Infinity,
  shouldRetry: () => true,
  retryWait: async (retries) => {
    await new Promise(r => setTimeout(r, Math.min(1000 * 2 ** retries, 30000)));
  }
}));

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 8: Memory leak from unbounded subscriptions
// Problem: Users creating unlimited subscriptions

// Bad - No limits
onSubscribe: () => {
  // Allow unlimited subscriptions
}

// Good - Track and limit
const userSubscriptions = new Map<string, number>();
const MAX_SUBSCRIPTIONS = 50;

onSubscribe: (ctx, msg) => {
  const userId = ctx.extra?.user?.id;
  const count = userSubscriptions.get(userId) || 0;
  
  if (count >= MAX_SUBSCRIPTIONS) {
    throw new Error('Too many subscriptions');
  }
  
  userSubscriptions.set(userId, count + 1);
}

onComplete: (ctx, msg) => {
  const userId = ctx.extra?.user?.id;
  const count = userSubscriptions.get(userId) || 0;
  userSubscriptions.set(userId, Math.max(0, count - 1));
}
```

---

## 10. Interview Questions

### Basic Questions

**Q: "What are GraphQL Subscriptions?"**
> "Subscriptions are the third GraphQL operation type for real-time data. Unlike queries (request-response), subscriptions establish a persistent WebSocket connection where the server pushes updates to the client when relevant events occur. They're defined in the schema like queries but use WebSocket transport."

**Q: "How do Subscriptions work under the hood?"**
> "Client sends subscription operation over WebSocket. Server creates an AsyncIterator (usually from PubSub) that yields events. When something publishes to PubSub (typically a mutation), the iterator yields, and the server sends a GraphQL response over WebSocket to the client."

**Q: "What protocol do Subscriptions use?"**
> "Modern implementations use the graphql-ws protocol over WebSocket. The legacy subscriptions-transport-ws is deprecated. graphql-ws handles connection initialization, subscription management, and proper cleanup."

### Intermediate Questions

**Q: "How do you handle authentication in Subscriptions?"**
> "Authentication happens during WebSocket connection via `connectionParams`. Client passes token, server verifies in `onConnect` callback. If valid, user info is stored in context for all subscriptions on that connection. Subscription-level authorization uses `withFilter` or checks in the subscribe function."

**Q: "How do you scale GraphQL Subscriptions?"**
> "Replace in-memory PubSub with Redis PubSub. Events published on any server are received by all servers via Redis. Each server manages its own WebSocket connections and filters events for its subscribers. Use sticky sessions or a WebSocket gateway for load balancing."

**Q: "What is withFilter and when do you use it?"**
> "`withFilter` is a higher-order function that wraps an AsyncIterator and adds filtering. It receives each published event and determines if it should be sent to this specific subscriber. Use it when you have a general topic but subscribers want filtered subsets - like all messages but only for a specific room."

### Advanced Questions

**Q: "How would you implement a real-time notifications system with Subscriptions?"**
> "Schema: Subscription type with `notificationAdded(userId: ID!)`. Server: Use user-specific topics like `NOTIFICATION_${userId}`. Authentication: Verify user in connection, ensure they can only subscribe to their own notifications. Publish: From mutations or external webhooks. Scaling: Redis PubSub. Client: Subscribe on login, update cache or state on each notification."

**Q: "How do you handle subscription reconnection and missed events?"**
> "graphql-ws handles reconnection automatically. For missed events: option 1 is client refetches query data on reconnect. Option 2 is track last event ID, server sends missed events from Redis Stream on reconnect. Option 3 is hybrid - short-lived events are okay to miss (like typing), important ones trigger query refetch."

**Q: "What are the trade-offs of Subscriptions vs polling?"**
> "**Subscriptions**: Real-time (<100ms), efficient for frequent updates, requires WebSocket infrastructure, connection state to manage. **Polling**: Simpler infrastructure, works through all proxies, higher latency (polling interval), more server load for high-frequency polling. Use subscriptions for true real-time needs (chat, collaboration), polling for occasional updates or fallback."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│           GRAPHQL SUBSCRIPTIONS CHECKLIST                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SERVER SETUP:                                                  │
│  □ Use graphql-ws (not subscriptions-transport-ws)             │
│  □ Setup WebSocketServer on /graphql path                      │
│  □ Use Redis PubSub for production                             │
│  □ Implement proper shutdown (drainServer)                     │
│                                                                 │
│  AUTHENTICATION:                                                │
│  □ Verify token in onConnect                                   │
│  □ Attach user to connection context                           │
│  □ Implement subscription-level auth in withFilter             │
│  □ Handle token refresh (requires reconnection)                │
│                                                                 │
│  RESOLVERS:                                                     │
│  □ Use withFilter for event filtering                          │
│  □ Consider topic-per-resource for efficiency                  │
│  □ Publish from mutations (or external sources)                │
│  □ Include necessary data for filtering in payload             │
│                                                                 │
│  CLIENT:                                                        │
│  □ Split link: WebSocket for subscriptions, HTTP for rest      │
│  □ Pass auth via connectionParams                              │
│  □ Handle reconnection (retryAttempts: Infinity)               │
│  □ Update cache on subscription data                           │
│                                                                 │
│  SCALING:                                                       │
│  □ Redis PubSub backend                                        │
│  □ Sticky sessions or WS gateway for load balancing            │
│  □ Monitor subscription counts per user                        │
│  □ Implement graceful degradation to polling                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

SUBSCRIPTION FLOW:
┌────────────────────────────────────────────────────────────────┐
│ 1. Client → subscription { messageAdded { ... } }              │
│ 2. Server → creates AsyncIterator from PubSub                  │
│ 3. Mutation → pubsub.publish('MESSAGE_ADDED', data)            │
│ 4. AsyncIterator yields → withFilter checks                    │
│ 5. If passes filter → Server sends data over WebSocket         │
│ 6. Client receives → updates UI                                │
└────────────────────────────────────────────────────────────────┘

WHEN TO USE SUBSCRIPTIONS:
✅ Real-time chat
✅ Live notifications
✅ Collaborative editing
✅ Live dashboards
✅ Gaming state updates

WHEN TO USE POLLING:
✅ Occasional updates (every few minutes)
✅ Fallback when WebSocket unavailable
✅ Simple infrastructure requirements
✅ Behind strict firewalls/proxies
```

---

*Last updated: February 2026*

