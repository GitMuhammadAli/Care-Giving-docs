# 🔮 GraphQL Deep Dive - Complete Guide

> A comprehensive guide to GraphQL - schema design, resolvers, DataLoader, N+1 prevention, and building efficient GraphQL APIs.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "GraphQL is a query language for APIs that lets clients request exactly the data they need in a single request, with a strongly-typed schema that serves as a contract between client and server."

### The 7 Key Concepts (Remember These!)
```
1. SCHEMA           → Type definitions (SDL)
2. QUERIES          → Read data
3. MUTATIONS        → Write data
4. SUBSCRIPTIONS    → Real-time data
5. RESOLVERS        → Functions that fetch data
6. DATALOADER       → Batching & caching for N+1
7. INTROSPECTION    → Self-documenting API
```

### GraphQL vs REST
```
┌─────────────────────────────────────────────────────────────────┐
│                   GraphQL vs REST                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  REST                          │  GraphQL                       │
│  ───────────────────────────── │ ─────────────────────────────  │
│  Multiple endpoints            │  Single endpoint (/graphql)    │
│  Server defines response       │  Client defines response       │
│  Over/under-fetching           │  Exact data requested          │
│  Multiple round trips          │  Single request                │
│  Versioning (v1, v2)           │  Schema evolution              │
│  HTTP caching easy             │  Caching more complex          │
│  Simple to understand          │  Learning curve                │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  CHOOSE REST WHEN:                                             │
│  • Simple CRUD operations                                      │
│  • HTTP caching is critical                                    │
│  • Public API with many consumers                              │
│                                                                 │
│  CHOOSE GraphQL WHEN:                                          │
│  • Complex, nested data requirements                           │
│  • Multiple clients with different needs                       │
│  • Rapid frontend iteration                                    │
│  • Mobile apps (bandwidth matters)                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### GraphQL Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                  GraphQL REQUEST FLOW                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CLIENT                                                        │
│    │                                                           │
│    │  query {                                                  │
│    │    user(id: "123") {                                      │
│    │      name                                                 │
│    │      orders { id, total }                                 │
│    │    }                                                      │
│    │  }                                                        │
│    │                                                           │
│    ▼                                                           │
│  ┌─────────────────────────────────────────────┐               │
│  │           GraphQL Server                     │               │
│  │  ┌─────────────────────────────────────┐    │               │
│  │  │  1. Parse → AST                      │    │               │
│  │  │  2. Validate against Schema          │    │               │
│  │  │  3. Execute Resolvers                │    │               │
│  │  │     ├── User resolver                │    │               │
│  │  │     └── Orders resolver (DataLoader) │    │               │
│  │  │  4. Format Response                  │    │               │
│  │  └─────────────────────────────────────┘    │               │
│  └─────────────────────────────────────────────┘               │
│    │                                                           │
│    ▼                                                           │
│  {                                                             │
│    "data": {                                                   │
│      "user": {                                                 │
│        "name": "John",                                         │
│        "orders": [{ "id": "1", "total": 99.99 }]              │
│      }                                                         │
│    }                                                           │
│  }                                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Schema-first"** | "We use schema-first development with SDL" |
| **"DataLoader"** | "DataLoader batches database calls to solve N+1" |
| **"Resolver chain"** | "The resolver chain handles nested field resolution" |
| **"Introspection"** | "Introspection enables auto-generated documentation" |
| **"Fragment"** | "We use fragments for reusable field selections" |
| **"Persisted queries"** | "Persisted queries improve performance and security" |

### Key Numbers to Remember
| Metric | Target | Why |
|--------|-------|-----|
| Query depth | **< 7 levels** | Prevent abuse |
| Query complexity | **Limit per query** | Resource protection |
| Batch size | **100-1000** | DataLoader batching |
| Response time | **< 200ms** | User experience |

### The "Wow" Statement (Memorize This!)
> "We use GraphQL with Apollo Server, schema-first approach. The schema serves as our API contract, auto-generating TypeScript types. We solved N+1 problems with DataLoader - it batches and caches database calls per request. For security: query depth limiting (max 7), complexity analysis, persisted queries in production. We use field-level resolvers for flexibility - the user resolver doesn't fetch orders unless requested. Subscriptions over WebSocket for real-time features. We have federation for our microservices - each service owns its schema portion. Performance monitoring with Apollo Studio shows resolver timing, helping us optimize hot paths."

---

## 📚 Table of Contents

1. [Schema Design](#1-schema-design)
2. [Resolvers](#2-resolvers)
3. [DataLoader](#3-dataloader)
4. [Queries & Mutations](#4-queries--mutations)
5. [Subscriptions](#5-subscriptions)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Schema Design

```graphql
# ════════════════════════════════════════════════════════════════
# GRAPHQL SCHEMA (SDL - Schema Definition Language)
# ════════════════════════════════════════════════════════════════

# Scalar types: ID, String, Int, Float, Boolean
# Custom scalars for special types
scalar DateTime
scalar JSON
scalar Email

# Enums
enum OrderStatus {
  PENDING
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
}

enum Role {
  USER
  ADMIN
  MODERATOR
}

# Input types (for mutations)
input CreateUserInput {
  name: String!
  email: Email!
  password: String!
  role: Role = USER
}

input UpdateUserInput {
  name: String
  email: Email
}

input OrderFilterInput {
  status: OrderStatus
  minTotal: Float
  maxTotal: Float
  dateFrom: DateTime
  dateTo: DateTime
}

# Object types
type User {
  id: ID!
  name: String!
  email: String!
  role: Role!
  createdAt: DateTime!
  updatedAt: DateTime!
  
  # Relations (resolved separately)
  orders(first: Int, after: String, filter: OrderFilterInput): OrderConnection!
  profile: Profile
}

type Profile {
  id: ID!
  bio: String
  avatar: String
  user: User!
}

type Order {
  id: ID!
  status: OrderStatus!
  total: Float!
  createdAt: DateTime!
  user: User!
  items: [OrderItem!]!
}

type OrderItem {
  id: ID!
  product: Product!
  quantity: Int!
  price: Float!
}

type Product {
  id: ID!
  name: String!
  price: Float!
  description: String
  inStock: Boolean!
}

# Pagination (Relay-style connections)
type OrderConnection {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type OrderEdge {
  node: Order!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

# Query root
type Query {
  # Single resources
  user(id: ID!): User
  order(id: ID!): Order
  product(id: ID!): Product
  
  # Collections
  users(first: Int, after: String, role: Role): UserConnection!
  orders(first: Int, after: String, filter: OrderFilterInput): OrderConnection!
  products(first: Int, after: String, search: String): ProductConnection!
  
  # Current user
  me: User
}

# Mutation root
type Mutation {
  # User mutations
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
  
  # Order mutations
  createOrder(items: [OrderItemInput!]!): Order!
  updateOrderStatus(id: ID!, status: OrderStatus!): Order!
  cancelOrder(id: ID!): Order!
  
  # Auth mutations
  login(email: String!, password: String!): AuthPayload!
  logout: Boolean!
}

# Subscription root
type Subscription {
  orderStatusChanged(orderId: ID!): Order!
  newOrder: Order!
}

# Auth payload
type AuthPayload {
  token: String!
  user: User!
}
```

---

## 2. Resolvers

```typescript
// ════════════════════════════════════════════════════════════════
// RESOLVER IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

import { Resolvers } from './generated/graphql';

const resolvers: Resolvers = {
  Query: {
    // Simple resolver
    user: async (_, { id }, context) => {
      return context.dataSources.users.findById(id);
    },

    // With authentication
    me: async (_, __, context) => {
      if (!context.user) {
        throw new AuthenticationError('Not authenticated');
      }
      return context.dataSources.users.findById(context.user.id);
    },

    // Paginated query
    users: async (_, { first = 10, after, role }, context) => {
      const { users, totalCount, hasMore } = await context.dataSources.users.findAll({
        first,
        after,
        role,
      });

      return {
        edges: users.map(user => ({
          node: user,
          cursor: Buffer.from(user.id).toString('base64'),
        })),
        pageInfo: {
          hasNextPage: hasMore,
          hasPreviousPage: !!after,
          startCursor: users[0] ? Buffer.from(users[0].id).toString('base64') : null,
          endCursor: users[users.length - 1] 
            ? Buffer.from(users[users.length - 1].id).toString('base64') 
            : null,
        },
        totalCount,
      };
    },
  },

  Mutation: {
    createUser: async (_, { input }, context) => {
      // Validation
      const existingUser = await context.dataSources.users.findByEmail(input.email);
      if (existingUser) {
        throw new UserInputError('Email already exists');
      }

      // Create user
      const user = await context.dataSources.users.create(input);
      
      // Publish event
      context.pubsub.publish('USER_CREATED', { userCreated: user });
      
      return user;
    },

    updateOrderStatus: async (_, { id, status }, context) => {
      const order = await context.dataSources.orders.updateStatus(id, status);
      
      // Publish for subscriptions
      context.pubsub.publish(`ORDER_STATUS_${id}`, { 
        orderStatusChanged: order 
      });
      
      return order;
    },
  },

  // Field resolvers (executed only when field is requested)
  User: {
    // Resolve orders relation
    orders: async (user, { first, after, filter }, context) => {
      const orders = await context.dataSources.orders.findByUserId(
        user.id,
        { first, after, filter }
      );
      
      return {
        edges: orders.map(order => ({
          node: order,
          cursor: Buffer.from(order.id).toString('base64'),
        })),
        pageInfo: { /* ... */ },
        totalCount: orders.length,
      };
    },

    // Resolve profile relation
    profile: async (user, _, context) => {
      return context.dataSources.profiles.findByUserId(user.id);
    },
  },

  Order: {
    // Resolve user relation
    user: async (order, _, context) => {
      // Use DataLoader to batch requests
      return context.loaders.userLoader.load(order.userId);
    },

    // Resolve items relation
    items: async (order, _, context) => {
      return context.dataSources.orderItems.findByOrderId(order.id);
    },
  },

  Subscription: {
    orderStatusChanged: {
      subscribe: (_, { orderId }, context) => {
        return context.pubsub.asyncIterator(`ORDER_STATUS_${orderId}`);
      },
    },

    newOrder: {
      subscribe: (_, __, context) => {
        // Check authorization
        if (!context.user?.isAdmin) {
          throw new ForbiddenError('Admin only');
        }
        return context.pubsub.asyncIterator('NEW_ORDER');
      },
    },
  },
};

// ════════════════════════════════════════════════════════════════
// CONTEXT SETUP
// ════════════════════════════════════════════════════════════════

import { ApolloServer } from '@apollo/server';

const server = new ApolloServer({
  typeDefs,
  resolvers,
});

// Context function - runs for each request
const context = async ({ req }) => {
  // Get user from token
  const token = req.headers.authorization?.replace('Bearer ', '');
  const user = token ? await verifyToken(token) : null;

  return {
    user,
    dataSources: {
      users: new UserDataSource(),
      orders: new OrderDataSource(),
      profiles: new ProfileDataSource(),
    },
    loaders: {
      userLoader: createUserLoader(),
      productLoader: createProductLoader(),
    },
    pubsub: new PubSub(),
  };
};
```

---

## 3. DataLoader

```typescript
// ════════════════════════════════════════════════════════════════
// DATALOADER - SOLVING N+1 PROBLEM
// ════════════════════════════════════════════════════════════════

import DataLoader from 'dataloader';

// THE N+1 PROBLEM
// ─────────────────
// Query: Get all orders with their users
// Without DataLoader:
//   1 query: SELECT * FROM orders
//   N queries: SELECT * FROM users WHERE id = ? (for each order)
//   Total: N+1 queries!

// With DataLoader:
//   1 query: SELECT * FROM orders
//   1 query: SELECT * FROM users WHERE id IN (?, ?, ?)
//   Total: 2 queries!

// ════════════════════════════════════════════════════════════════
// DATALOADER IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

// User loader
function createUserLoader() {
  return new DataLoader<string, User>(async (userIds) => {
    console.log(`Batching ${userIds.length} user requests`);
    
    // Single query for all users
    const users = await prisma.user.findMany({
      where: { id: { in: [...userIds] } },
    });
    
    // Map results back to input order
    const userMap = new Map(users.map(user => [user.id, user]));
    return userIds.map(id => userMap.get(id) || null);
  });
}

// Product loader
function createProductLoader() {
  return new DataLoader<string, Product>(async (productIds) => {
    const products = await prisma.product.findMany({
      where: { id: { in: [...productIds] } },
    });
    
    const productMap = new Map(products.map(p => [p.id, p]));
    return productIds.map(id => productMap.get(id) || null);
  });
}

// Orders by user loader (one-to-many)
function createOrdersByUserLoader() {
  return new DataLoader<string, Order[]>(async (userIds) => {
    const orders = await prisma.order.findMany({
      where: { userId: { in: [...userIds] } },
    });
    
    // Group orders by userId
    const ordersByUser = new Map<string, Order[]>();
    for (const order of orders) {
      const existing = ordersByUser.get(order.userId) || [];
      ordersByUser.set(order.userId, [...existing, order]);
    }
    
    return userIds.map(id => ordersByUser.get(id) || []);
  });
}

// ════════════════════════════════════════════════════════════════
// USING DATALOADER IN RESOLVERS
// ════════════════════════════════════════════════════════════════

const resolvers = {
  Order: {
    // Without DataLoader - N+1 problem!
    // user: async (order, _, context) => {
    //   return context.dataSources.users.findById(order.userId);
    // },

    // With DataLoader - batched!
    user: async (order, _, context) => {
      return context.loaders.userLoader.load(order.userId);
    },
  },

  OrderItem: {
    product: async (item, _, context) => {
      return context.loaders.productLoader.load(item.productId);
    },
  },

  User: {
    orders: async (user, _, context) => {
      return context.loaders.ordersByUserLoader.load(user.id);
    },
  },
};

// ════════════════════════════════════════════════════════════════
// DATALOADER WITH CACHING
// ════════════════════════════════════════════════════════════════

// DataLoader caches within a single request by default
// For cross-request caching, combine with Redis

function createCachedUserLoader(redis: Redis) {
  return new DataLoader<string, User>(
    async (userIds) => {
      // Check cache first
      const cached = await Promise.all(
        userIds.map(id => redis.get(`user:${id}`))
      );
      
      // Find missing
      const missingIds = userIds.filter((_, i) => !cached[i]);
      
      // Fetch missing from DB
      let dbUsers: User[] = [];
      if (missingIds.length > 0) {
        dbUsers = await prisma.user.findMany({
          where: { id: { in: missingIds } },
        });
        
        // Cache fetched users
        await Promise.all(
          dbUsers.map(user => 
            redis.setex(`user:${user.id}`, 3600, JSON.stringify(user))
          )
        );
      }
      
      // Merge cached and fetched
      const userMap = new Map(dbUsers.map(u => [u.id, u]));
      return userIds.map((id, i) => {
        if (cached[i]) return JSON.parse(cached[i]);
        return userMap.get(id) || null;
      });
    },
    {
      cache: false, // Disable DataLoader cache, use Redis instead
    }
  );
}
```

---

## 4. Queries & Mutations

```typescript
// ════════════════════════════════════════════════════════════════
// CLIENT-SIDE QUERIES
// ════════════════════════════════════════════════════════════════

// Simple query
const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
    }
  }
`;

// Query with nested fields
const GET_USER_WITH_ORDERS = gql`
  query GetUserWithOrders($id: ID!, $first: Int) {
    user(id: $id) {
      id
      name
      email
      orders(first: $first) {
        edges {
          node {
            id
            total
            status
            items {
              product {
                name
                price
              }
              quantity
            }
          }
        }
        pageInfo {
          hasNextPage
          endCursor
        }
      }
    }
  }
`;

// Using fragments for reusability
const USER_FIELDS = gql`
  fragment UserFields on User {
    id
    name
    email
    role
    createdAt
  }
`;

const ORDER_FIELDS = gql`
  fragment OrderFields on Order {
    id
    total
    status
    createdAt
  }
`;

const GET_USER_COMPLETE = gql`
  ${USER_FIELDS}
  ${ORDER_FIELDS}
  
  query GetUserComplete($id: ID!) {
    user(id: $id) {
      ...UserFields
      orders(first: 5) {
        edges {
          node {
            ...OrderFields
          }
        }
      }
    }
  }
`;

// ════════════════════════════════════════════════════════════════
// MUTATIONS
// ════════════════════════════════════════════════════════════════

const CREATE_USER = gql`
  mutation CreateUser($input: CreateUserInput!) {
    createUser(input: $input) {
      id
      name
      email
    }
  }
`;

const UPDATE_ORDER_STATUS = gql`
  mutation UpdateOrderStatus($id: ID!, $status: OrderStatus!) {
    updateOrderStatus(id: $id, status: $status) {
      id
      status
      updatedAt
    }
  }
`;

// Mutation with optimistic response
const ADD_TO_CART = gql`
  mutation AddToCart($productId: ID!, $quantity: Int!) {
    addToCart(productId: $productId, quantity: $quantity) {
      id
      items {
        product {
          id
          name
        }
        quantity
      }
      totalItems
    }
  }
`;

// ════════════════════════════════════════════════════════════════
// REACT APOLLO CLIENT USAGE
// ════════════════════════════════════════════════════════════════

import { useQuery, useMutation } from '@apollo/client';

function UserProfile({ userId }: { userId: string }) {
  // Query
  const { data, loading, error } = useQuery(GET_USER_WITH_ORDERS, {
    variables: { id: userId, first: 5 },
  });

  // Mutation
  const [updateStatus, { loading: updating }] = useMutation(UPDATE_ORDER_STATUS, {
    // Update cache after mutation
    update(cache, { data: { updateOrderStatus } }) {
      cache.modify({
        id: cache.identify(updateOrderStatus),
        fields: {
          status: () => updateOrderStatus.status,
        },
      });
    },
    // Optimistic response
    optimisticResponse: {
      updateOrderStatus: {
        __typename: 'Order',
        id: orderId,
        status: 'PROCESSING',
        updatedAt: new Date().toISOString(),
      },
    },
  });

  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} />;

  return (
    <div>
      <h1>{data.user.name}</h1>
      <OrderList 
        orders={data.user.orders.edges.map(e => e.node)}
        onStatusChange={(id, status) => updateStatus({ variables: { id, status } })}
      />
    </div>
  );
}
```

---

## 5. Subscriptions

```typescript
// ════════════════════════════════════════════════════════════════
// GRAPHQL SUBSCRIPTIONS
// ════════════════════════════════════════════════════════════════

// Schema
const typeDefs = gql`
  type Subscription {
    orderStatusChanged(orderId: ID!): Order!
    newMessage(roomId: ID!): Message!
    onlineUsers: [User!]!
  }
`;

// Server-side resolver with PubSub
import { PubSub } from 'graphql-subscriptions';
const pubsub = new PubSub();

const resolvers = {
  Subscription: {
    orderStatusChanged: {
      subscribe: (_, { orderId }) => {
        return pubsub.asyncIterator(`ORDER_${orderId}`);
      },
    },

    newMessage: {
      subscribe: withFilter(
        () => pubsub.asyncIterator('NEW_MESSAGE'),
        (payload, variables) => {
          // Filter: only send to subscribers of this room
          return payload.newMessage.roomId === variables.roomId;
        }
      ),
    },

    onlineUsers: {
      subscribe: () => pubsub.asyncIterator('ONLINE_USERS'),
    },
  },

  Mutation: {
    updateOrderStatus: async (_, { id, status }) => {
      const order = await orderService.updateStatus(id, status);
      
      // Publish to subscribers
      pubsub.publish(`ORDER_${id}`, { orderStatusChanged: order });
      
      return order;
    },

    sendMessage: async (_, { input }, context) => {
      const message = await messageService.create({
        ...input,
        userId: context.user.id,
      });
      
      pubsub.publish('NEW_MESSAGE', { newMessage: message });
      
      return message;
    },
  },
};

// ════════════════════════════════════════════════════════════════
// APOLLO SERVER WITH SUBSCRIPTIONS
// ════════════════════════════════════════════════════════════════

import { ApolloServer } from '@apollo/server';
import { expressMiddleware } from '@apollo/server/express4';
import { createServer } from 'http';
import { WebSocketServer } from 'ws';
import { useServer } from 'graphql-ws/lib/use/ws';

// Create HTTP server
const app = express();
const httpServer = createServer(app);

// Create WebSocket server
const wsServer = new WebSocketServer({
  server: httpServer,
  path: '/graphql',
});

// Set up WebSocket server for subscriptions
const serverCleanup = useServer(
  {
    schema,
    context: async (ctx) => {
      // Authenticate WebSocket connection
      const token = ctx.connectionParams?.authToken;
      const user = token ? await verifyToken(token) : null;
      return { user, pubsub };
    },
    onConnect: async (ctx) => {
      console.log('Client connected');
    },
    onDisconnect: async (ctx) => {
      console.log('Client disconnected');
    },
  },
  wsServer
);

// Create Apollo Server
const server = new ApolloServer({
  schema,
  plugins: [
    {
      async serverWillStart() {
        return {
          async drainServer() {
            await serverCleanup.dispose();
          },
        };
      },
    },
  ],
});

// ════════════════════════════════════════════════════════════════
// CLIENT-SIDE SUBSCRIPTION
// ════════════════════════════════════════════════════════════════

import { useSubscription } from '@apollo/client';

const ORDER_STATUS_SUBSCRIPTION = gql`
  subscription OnOrderStatusChanged($orderId: ID!) {
    orderStatusChanged(orderId: $orderId) {
      id
      status
      updatedAt
    }
  }
`;

function OrderTracker({ orderId }: { orderId: string }) {
  const { data, loading } = useSubscription(ORDER_STATUS_SUBSCRIPTION, {
    variables: { orderId },
  });

  if (loading) return <p>Waiting for updates...</p>;

  return (
    <div>
      <p>Status: {data?.orderStatusChanged.status}</p>
    </div>
  );
}
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# GRAPHQL PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: N+1 Query Problem
# Bad - Each order fetches user separately
Order:
  user: (order) => db.users.findById(order.userId)
# Results in N+1 queries

# Good - Use DataLoader
Order:
  user: (order, _, ctx) => ctx.loaders.userLoader.load(order.userId)
# Results in batched query

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: No query depth/complexity limits
# Bad - Client can request infinitely nested queries
query {
  user {
    orders {
      user {
        orders {
          user { ... }  # Infinite!
        }
      }
    }
  }
}

# Good - Implement limits
const server = new ApolloServer({
  validationRules: [
    depthLimit(7),
    queryComplexity({ maximumComplexity: 1000 }),
  ],
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Exposing sensitive data
# Bad - No field-level authorization
type User {
  id: ID!
  email: String!
  password: String!  # Never expose!
  ssn: String!       # Sensitive!
}

# Good - Remove sensitive fields, add authorization
type User {
  id: ID!
  email: String! @auth(requires: OWNER)
  # No password field
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Fat resolvers
# Bad - All logic in resolver
Mutation:
  createOrder: async (_, { input }) => {
    // Validation
    // Business logic
    // Database operations
    // Email sending
    // 200 lines of code...
  }

# Good - Thin resolver, service layer
Mutation:
  createOrder: async (_, { input }, ctx) => {
    return ctx.services.orders.create(input);
  }

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Overfetching in resolvers
# Bad - Always fetch all relations
User:
  orders: async (user) => {
    return db.orders.findAll({
      where: { userId: user.id },
      include: [items, products, user]  # Always fetches everything
    });
  }

# Good - Only fetch what's needed
User:
  orders: async (user, _, ctx, info) => {
    const requestedFields = getRequestedFields(info);
    return db.orders.findAll({
      where: { userId: user.id },
      include: buildIncludes(requestedFields),
    });
  }
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is GraphQL?"**
> "GraphQL is a query language for APIs and runtime for executing queries. Clients request exactly the data they need in a single request. It has a strongly-typed schema that serves as a contract. Unlike REST's multiple endpoints, GraphQL uses a single endpoint where the query determines the response shape."

**Q: "What are the main operation types?"**
> "Three types: Queries for reading data (like GET), Mutations for writing data (like POST/PUT/DELETE), and Subscriptions for real-time updates via WebSocket. The schema defines what operations are available and their types."

**Q: "How does GraphQL prevent over-fetching?"**
> "Clients specify exactly which fields they need in the query. The server returns only those fields. In REST, endpoints return fixed responses. In GraphQL, `{ user { name } }` returns only the name, not the entire user object."

### Intermediate Questions

**Q: "What is the N+1 problem and how do you solve it?"**
> "N+1: When fetching a list of N items, each item triggers a separate query for related data. Example: 1 query for 100 orders, then 100 queries for users. Solution: DataLoader batches and caches requests. Collects all user IDs, makes one query: `WHERE id IN (...)`. Reduces N+1 to 2 queries."

**Q: "How do you handle authentication/authorization?"**
> "Authentication: Verify token in context function, attach user to context. Authorization: Check permissions in resolvers or use directives. Field-level auth for sensitive data. Example: `@auth(requires: ADMIN)` directive. Always validate in resolver, never trust client."

**Q: "What are fragments and why use them?"**
> "Fragments are reusable field selections. `fragment UserFields on User { id, name, email }`. Benefits: DRY code, consistent field selection, easier maintenance. Use when same fields needed in multiple queries. Spread with `...UserFields`."

### Advanced Questions

**Q: "How do you optimize GraphQL performance?"**
> "1) DataLoader for batching. 2) Query complexity limits. 3) Persisted queries (hash instead of full query). 4) Caching with CDN or Redis. 5) Field-level resolvers (don't fetch unless requested). 6) Pagination (connections pattern). 7) APM monitoring to find slow resolvers."

**Q: "What is GraphQL Federation?"**
> "Federation enables building a distributed GraphQL architecture. Each service owns part of the schema. Gateway composes schemas into unified API. Services can extend types from other services. Example: Users service defines User, Orders service extends User with orders field. Enables microservices with unified GraphQL API."

**Q: "GraphQL vs REST - how do you choose?"**
> "GraphQL: Complex data requirements, multiple clients, rapid iteration, mobile (bandwidth). REST: Simple CRUD, HTTP caching critical, public APIs, team familiarity. GraphQL has learning curve, caching complexity. REST is simpler but may need multiple endpoints. Choose based on specific needs."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                   GRAPHQL CHECKLIST                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SCHEMA DESIGN:                                                 │
│  □ Use SDL (Schema Definition Language)                        │
│  □ Define custom scalars (DateTime, Email)                     │
│  □ Use input types for mutations                               │
│  □ Implement connections for pagination                        │
│                                                                 │
│  PERFORMANCE:                                                   │
│  □ DataLoader for N+1 prevention                               │
│  □ Query depth/complexity limits                               │
│  □ Field-level resolvers                                       │
│  □ Persisted queries in production                             │
│                                                                 │
│  SECURITY:                                                      │
│  □ Authentication in context                                   │
│  □ Authorization per field/resolver                            │
│  □ Input validation                                            │
│  □ Rate limiting                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

OPERATION TYPES:
┌────────────────────────────────────────────────────────────────┐
│ query { ... }         - Read data (GET)                        │
│ mutation { ... }      - Write data (POST/PUT/DELETE)           │
│ subscription { ... }  - Real-time updates (WebSocket)          │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

