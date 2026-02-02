# 🔗 Hypermedia APIs (HATEOAS) - Complete Guide

> A comprehensive guide to hypermedia APIs - HATEOAS, discoverability, self-documenting APIs, and building truly RESTful interfaces.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "HATEOAS (Hypermedia As The Engine Of Application State) is a REST constraint where API responses include links to related resources and available actions, enabling clients to navigate the API dynamically without hardcoding URLs."

### The 7 Key Concepts (Remember These!)
```
1. HATEOAS           → Links in responses guide client navigation
2. SELF LINK         → URL to the current resource
3. RELATED LINKS     → URLs to associated resources
4. ACTION LINKS      → Available operations on the resource
5. HAL               → Hypertext Application Language format
6. JSON:API          → Another hypermedia specification
7. AFFORDANCES       → What actions the client can take
```

### Richardson Maturity Model
```
┌─────────────────────────────────────────────────────────────────┐
│              RICHARDSON MATURITY MODEL                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LEVEL 3: Hypermedia Controls (HATEOAS) ← TRUE REST            │
│  ─────────────────────────────────────────                      │
│  • Links tell client what's possible                           │
│  • No hardcoded URLs in client                                 │
│  • API is self-documenting                                     │
│  • State transitions via links                                 │
│                                                                 │
│  Example Response:                                             │
│  {                                                             │
│    "id": "order_123",                                          │
│    "status": "pending",                                        │
│    "_links": {                                                 │
│      "self": { "href": "/orders/order_123" },                  │
│      "cancel": { "href": "/orders/order_123/cancel" },         │
│      "pay": { "href": "/orders/order_123/pay" }                │
│    }                                                           │
│  }                                                             │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  LEVEL 2: HTTP Verbs (Most APIs stop here)                     │
│  ─────────────────────────────────────────                      │
│  • Proper GET, POST, PUT, DELETE                               │
│  • Meaningful status codes                                     │
│                                                                 │
│  LEVEL 1: Resources                                            │
│  ──────────────────                                             │
│  • Individual URIs for resources                               │
│                                                                 │
│  LEVEL 0: The Swamp of POX                                     │
│  ─────────────────────────                                      │
│  • Single endpoint, RPC style                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### HATEOAS Benefits
```
┌─────────────────────────────────────────────────────────────────┐
│                    HATEOAS BENEFITS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DISCOVERABILITY                                               │
│  ───────────────                                                │
│  • Client doesn't need to know URLs                            │
│  • Start from root, follow links                               │
│  • New endpoints automatically available                       │
│                                                                 │
│  LOOSE COUPLING                                                │
│  ──────────────                                                 │
│  • Server can change URLs                                      │
│  • Client follows links, doesn't hardcode                      │
│  • Easier API evolution                                        │
│                                                                 │
│  SELF-DOCUMENTING                                              │
│  ────────────────                                               │
│  • Links show what's possible                                  │
│  • Response tells available actions                            │
│  • Reduced need for documentation                              │
│                                                                 │
│  STATE-DRIVEN                                                  │
│  ────────────                                                   │
│  • Links change based on resource state                        │
│  • "cancel" only appears if cancellable                        │
│  • Client UI can enable/disable based on links                 │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  TRADE-OFFS                                                    │
│  ──────────                                                     │
│  ❌ Larger response size                                       │
│  ❌ More complex responses                                     │
│  ❌ Overhead for simple APIs                                   │
│  ❌ Clients may ignore links anyway                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"HATEOAS"** | "Our API follows HATEOAS - Richardson Level 3" |
| **"Affordances"** | "Links represent available affordances" |
| **"HAL format"** | "We use HAL for hypermedia responses" |
| **"Discoverable"** | "The API is fully discoverable from root" |
| **"Self-describing"** | "Responses are self-describing with links" |
| **"State machine"** | "Resources are state machines, links are transitions" |

### Key Numbers to Remember
| Metric | Note | Why |
|--------|------|-----|
| Richardson Level 3 | True REST | HATEOAS is highest maturity |
| Link overhead | ~10-20% | Added response size |
| Adoption | ~5% of APIs | Most stop at Level 2 |

### The "Wow" Statement (Memorize This!)
> "We implement HATEOAS at Richardson Level 3 using HAL format. Every response includes _links with self reference and available actions. An order response shows 'cancel' link only when order is cancellable, 'pay' only when awaiting payment. Clients don't hardcode URLs - they follow links from the root endpoint. This enables API evolution without breaking clients. We also include _embedded for related resources to reduce round trips. It's more effort to implement, but the discoverability and decoupling benefits are significant for public APIs. For internal APIs, Level 2 is often sufficient."

---

## 📚 Table of Contents

1. [HATEOAS Fundamentals](#1-hateoas-fundamentals)
2. [HAL Format](#2-hal-format)
3. [Implementation](#3-implementation)
4. [State-Driven Links](#4-state-driven-links)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. HATEOAS Fundamentals

```typescript
// ════════════════════════════════════════════════════════════════
// HATEOAS RESPONSE STRUCTURE
// ════════════════════════════════════════════════════════════════

// Basic link structure
interface Link {
  href: string;           // URL to resource
  method?: string;        // HTTP method (GET if not specified)
  title?: string;         // Human-readable description
  type?: string;          // Media type
  templated?: boolean;    // URL is a template
}

interface Links {
  [rel: string]: Link | Link[];
}

// Resource with HATEOAS links
interface HypermediaResource<T> {
  // Resource data
  [key: string]: any;
  
  // Links to related resources and actions
  _links: Links;
  
  // Embedded related resources (optional)
  _embedded?: {
    [rel: string]: HypermediaResource<any> | HypermediaResource<any>[];
  };
}

// ════════════════════════════════════════════════════════════════
// EXAMPLE: USER RESOURCE
// ════════════════════════════════════════════════════════════════

// Response: GET /users/user_123
const userResponse: HypermediaResource<User> = {
  id: 'user_123',
  name: 'John Doe',
  email: 'john@example.com',
  status: 'active',
  createdAt: '2024-01-15T10:30:00Z',
  
  _links: {
    // Self reference
    self: {
      href: '/users/user_123',
    },
    
    // Related resources
    orders: {
      href: '/users/user_123/orders',
      title: 'User orders',
    },
    profile: {
      href: '/users/user_123/profile',
      title: 'User profile',
    },
    
    // Actions
    update: {
      href: '/users/user_123',
      method: 'PATCH',
      title: 'Update user',
    },
    deactivate: {
      href: '/users/user_123/deactivate',
      method: 'POST',
      title: 'Deactivate user',
    },
    
    // Collection links
    collection: {
      href: '/users',
      title: 'All users',
    },
  },
  
  // Embedded resources (optional - reduce round trips)
  _embedded: {
    latestOrders: [
      {
        id: 'order_456',
        total: 99.99,
        status: 'delivered',
        _links: {
          self: { href: '/orders/order_456' },
        },
      },
    ],
  },
};

// ════════════════════════════════════════════════════════════════
// ROOT ENDPOINT (Entry Point)
// ════════════════════════════════════════════════════════════════

// Response: GET /api
const apiRoot = {
  name: 'Example API',
  version: '1.0.0',
  
  _links: {
    self: { href: '/api' },
    
    // All available resources
    users: {
      href: '/api/users{?page,limit}',
      templated: true,
      title: 'User collection',
    },
    orders: {
      href: '/api/orders{?status,page,limit}',
      templated: true,
      title: 'Order collection',
    },
    products: {
      href: '/api/products{?category,search}',
      templated: true,
      title: 'Product collection',
    },
    
    // Documentation
    documentation: {
      href: 'https://docs.example.com/api',
      type: 'text/html',
      title: 'API documentation',
    },
  },
};

// ════════════════════════════════════════════════════════════════
// COLLECTION RESPONSE WITH PAGINATION
// ════════════════════════════════════════════════════════════════

// Response: GET /users?page=2&limit=20
const usersCollection = {
  _links: {
    self: { href: '/users?page=2&limit=20' },
    first: { href: '/users?page=1&limit=20' },
    prev: { href: '/users?page=1&limit=20' },
    next: { href: '/users?page=3&limit=20' },
    last: { href: '/users?page=5&limit=20' },
    
    // Templated link for search
    find: {
      href: '/users{?email,name}',
      templated: true,
    },
    
    // Create action
    create: {
      href: '/users',
      method: 'POST',
    },
  },
  
  _embedded: {
    users: [
      {
        id: 'user_21',
        name: 'User 21',
        _links: {
          self: { href: '/users/user_21' },
        },
      },
      // ... more users
    ],
  },
  
  // Pagination metadata
  page: 2,
  limit: 20,
  total: 100,
  totalPages: 5,
};
```

---

## 2. HAL Format

```typescript
// ════════════════════════════════════════════════════════════════
// HAL (Hypertext Application Language)
// ════════════════════════════════════════════════════════════════

/*
 * HAL is a simple format for hypermedia APIs
 * Media type: application/hal+json
 * 
 * Reserved properties:
 * - _links: Related resources and actions
 * - _embedded: Embedded resources
 */

// HAL Link Interface
interface HalLink {
  href: string;           // Required: URI or URI template
  templated?: boolean;    // Is href a URI template?
  type?: string;          // Media type hint
  deprecation?: string;   // URL to deprecation info
  name?: string;          // Secondary key for links
  profile?: string;       // URI for additional semantics
  title?: string;         // Human-readable identifier
  hreflang?: string;      // Language hint
}

interface HalResource {
  _links?: {
    self?: HalLink;
    [rel: string]: HalLink | HalLink[] | undefined;
  };
  _embedded?: {
    [rel: string]: HalResource | HalResource[];
  };
  [property: string]: any;
}

// ════════════════════════════════════════════════════════════════
// HAL BUILDER
// ════════════════════════════════════════════════════════════════

class HalBuilder<T extends object> {
  private resource: T;
  private links: { [rel: string]: HalLink | HalLink[] } = {};
  private embedded: { [rel: string]: HalResource | HalResource[] } = {};

  constructor(resource: T) {
    this.resource = resource;
  }

  addLink(rel: string, link: HalLink | HalLink[]): this {
    this.links[rel] = link;
    return this;
  }

  addSelfLink(href: string): this {
    return this.addLink('self', { href });
  }

  addEmbedded(rel: string, resource: HalResource | HalResource[]): this {
    this.embedded[rel] = resource;
    return this;
  }

  build(): HalResource {
    const result: HalResource = { ...this.resource };
    
    if (Object.keys(this.links).length > 0) {
      result._links = this.links;
    }
    
    if (Object.keys(this.embedded).length > 0) {
      result._embedded = this.embedded;
    }
    
    return result;
  }
}

// Usage
function toHalUser(user: User, baseUrl: string): HalResource {
  return new HalBuilder(user)
    .addSelfLink(`${baseUrl}/users/${user.id}`)
    .addLink('orders', { href: `${baseUrl}/users/${user.id}/orders` })
    .addLink('profile', { href: `${baseUrl}/users/${user.id}/profile` })
    .addLink('collection', { href: `${baseUrl}/users` })
    .build();
}

// ════════════════════════════════════════════════════════════════
// EXPRESS MIDDLEWARE FOR HAL
// ════════════════════════════════════════════════════════════════

// Set HAL content type
app.use((req, res, next) => {
  const originalJson = res.json.bind(res);
  
  res.hal = function(resource: HalResource) {
    res.setHeader('Content-Type', 'application/hal+json');
    return originalJson(resource);
  };
  
  next();
});

// Controller using HAL
app.get('/users/:id', async (req, res) => {
  const user = await userService.findById(req.params.id);
  
  if (!user) {
    return res.status(404).json({ error: 'Not found' });
  }
  
  const baseUrl = `${req.protocol}://${req.get('host')}/api`;
  res.hal(toHalUser(user, baseUrl));
});

// ════════════════════════════════════════════════════════════════
// CURIES (Compact URIs)
// ════════════════════════════════════════════════════════════════

// Curies allow using shorthand relation names
const responseWithCuries = {
  _links: {
    // Define curies for documentation
    curies: [
      {
        name: 'ex',
        href: 'https://docs.example.com/rels/{rel}',
        templated: true,
      },
    ],
    
    self: { href: '/orders/123' },
    
    // Custom relations with documentation
    'ex:customer': { href: '/users/456' },
    'ex:payment': { href: '/orders/123/payment' },
    'ex:items': { href: '/orders/123/items' },
  },
  
  id: 'order_123',
  total: 99.99,
};

// Client can look up "ex:customer" at:
// https://docs.example.com/rels/customer
```

---

## 3. Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// HYPERMEDIA RESPONSE BUILDER
// ════════════════════════════════════════════════════════════════

interface LinkDefinition {
  rel: string;
  href: string | ((resource: any) => string);
  method?: string;
  title?: string;
  templated?: boolean;
  condition?: (resource: any) => boolean;
}

class HypermediaBuilder<T> {
  private linkDefinitions: LinkDefinition[] = [];
  private embeddedDefinitions: Array<{
    rel: string;
    loader: (resource: T) => Promise<any>;
  }> = [];

  constructor(private resourceType: string) {}

  addLink(definition: LinkDefinition): this {
    this.linkDefinitions.push(definition);
    return this;
  }

  addEmbed(rel: string, loader: (resource: T) => Promise<any>): this {
    this.embeddedDefinitions.push({ rel, loader });
    return this;
  }

  async build(resource: T, baseUrl: string): Promise<HalResource> {
    const links: Record<string, HalLink> = {};
    
    for (const def of this.linkDefinitions) {
      // Check condition
      if (def.condition && !def.condition(resource)) {
        continue;
      }
      
      // Build href
      const href = typeof def.href === 'function' 
        ? def.href(resource) 
        : def.href;
      
      links[def.rel] = {
        href: baseUrl + href,
        ...(def.method && { method: def.method }),
        ...(def.title && { title: def.title }),
        ...(def.templated && { templated: def.templated }),
      };
    }

    // Load embedded resources
    const embedded: Record<string, any> = {};
    for (const def of this.embeddedDefinitions) {
      embedded[def.rel] = await def.loader(resource);
    }

    return {
      ...resource,
      _links: links,
      ...(Object.keys(embedded).length > 0 && { _embedded: embedded }),
    };
  }
}

// ════════════════════════════════════════════════════════════════
// ORDER RESOURCE BUILDER
// ════════════════════════════════════════════════════════════════

const orderBuilder = new HypermediaBuilder<Order>('order')
  // Self link (always)
  .addLink({
    rel: 'self',
    href: (order) => `/orders/${order.id}`,
  })
  
  // Collection link
  .addLink({
    rel: 'collection',
    href: '/orders',
  })
  
  // Related customer
  .addLink({
    rel: 'customer',
    href: (order) => `/users/${order.customerId}`,
    title: 'Order customer',
  })
  
  // Items
  .addLink({
    rel: 'items',
    href: (order) => `/orders/${order.id}/items`,
    title: 'Order items',
  })
  
  // Pay action - only for pending orders
  .addLink({
    rel: 'pay',
    href: (order) => `/orders/${order.id}/pay`,
    method: 'POST',
    title: 'Pay for order',
    condition: (order) => order.status === 'pending',
  })
  
  // Cancel action - only for pending or processing
  .addLink({
    rel: 'cancel',
    href: (order) => `/orders/${order.id}/cancel`,
    method: 'POST',
    title: 'Cancel order',
    condition: (order) => ['pending', 'processing'].includes(order.status),
  })
  
  // Ship action - only for paid orders
  .addLink({
    rel: 'ship',
    href: (order) => `/orders/${order.id}/ship`,
    method: 'POST',
    title: 'Ship order',
    condition: (order) => order.status === 'paid',
  })
  
  // Invoice - only for paid/shipped/delivered
  .addLink({
    rel: 'invoice',
    href: (order) => `/orders/${order.id}/invoice`,
    title: 'Download invoice',
    condition: (order) => ['paid', 'shipped', 'delivered'].includes(order.status),
  });

// ════════════════════════════════════════════════════════════════
// CONTROLLER WITH HYPERMEDIA
// ════════════════════════════════════════════════════════════════

app.get('/orders/:id', async (req, res) => {
  const order = await orderService.findById(req.params.id);
  
  if (!order) {
    return res.status(404).json({
      _links: {
        collection: { href: '/orders' },
      },
      error: 'Order not found',
    });
  }
  
  const baseUrl = `${req.protocol}://${req.get('host')}/api`;
  const response = await orderBuilder.build(order, baseUrl);
  
  res
    .setHeader('Content-Type', 'application/hal+json')
    .json(response);
});

// Collection endpoint
app.get('/orders', async (req, res) => {
  const { page = 1, limit = 20, status } = req.query;
  const baseUrl = `${req.protocol}://${req.get('host')}/api`;
  
  const { orders, total } = await orderService.findAll({
    page: Number(page),
    limit: Number(limit),
    status: status as string,
  });
  
  const totalPages = Math.ceil(total / Number(limit));
  
  // Build collection response
  const response = {
    _links: {
      self: { href: `/orders?page=${page}&limit=${limit}` },
      first: { href: `/orders?page=1&limit=${limit}` },
      last: { href: `/orders?page=${totalPages}&limit=${limit}` },
      ...(Number(page) > 1 && {
        prev: { href: `/orders?page=${Number(page) - 1}&limit=${limit}` },
      }),
      ...(Number(page) < totalPages && {
        next: { href: `/orders?page=${Number(page) + 1}&limit=${limit}` },
      }),
      create: {
        href: '/orders',
        method: 'POST',
        title: 'Create new order',
      },
      find: {
        href: '/orders{?status,customerId}',
        templated: true,
      },
    },
    _embedded: {
      orders: await Promise.all(
        orders.map(order => orderBuilder.build(order, baseUrl))
      ),
    },
    page: Number(page),
    limit: Number(limit),
    total,
    totalPages,
  };
  
  res
    .setHeader('Content-Type', 'application/hal+json')
    .json(response);
});
```

---

## 4. State-Driven Links

```typescript
// ════════════════════════════════════════════════════════════════
// STATE MACHINE WITH HYPERMEDIA
// ════════════════════════════════════════════════════════════════

/*
 * Order State Machine:
 * 
 *   ┌─────────┐     ┌───────────┐     ┌──────┐
 *   │ pending │ ──> │ processing│ ──> │ paid │
 *   └────┬────┘     └─────┬─────┘     └──┬───┘
 *        │                │              │
 *        └───────┬────────┘              │
 *                ▼                       ▼
 *          ┌───────────┐           ┌─────────┐
 *          │ cancelled │           │ shipped │
 *          └───────────┘           └────┬────┘
 *                                       │
 *                                       ▼
 *                                 ┌───────────┐
 *                                 │ delivered │
 *                                 └───────────┘
 */

type OrderStatus = 'pending' | 'processing' | 'paid' | 'shipped' | 'delivered' | 'cancelled';

interface OrderTransition {
  action: string;
  toStatus: OrderStatus;
  href: (order: Order) => string;
  method: string;
  title: string;
}

const orderTransitions: Record<OrderStatus, OrderTransition[]> = {
  pending: [
    {
      action: 'process',
      toStatus: 'processing',
      href: (order) => `/orders/${order.id}/process`,
      method: 'POST',
      title: 'Start processing',
    },
    {
      action: 'cancel',
      toStatus: 'cancelled',
      href: (order) => `/orders/${order.id}/cancel`,
      method: 'POST',
      title: 'Cancel order',
    },
  ],
  processing: [
    {
      action: 'pay',
      toStatus: 'paid',
      href: (order) => `/orders/${order.id}/pay`,
      method: 'POST',
      title: 'Mark as paid',
    },
    {
      action: 'cancel',
      toStatus: 'cancelled',
      href: (order) => `/orders/${order.id}/cancel`,
      method: 'POST',
      title: 'Cancel order',
    },
  ],
  paid: [
    {
      action: 'ship',
      toStatus: 'shipped',
      href: (order) => `/orders/${order.id}/ship`,
      method: 'POST',
      title: 'Ship order',
    },
  ],
  shipped: [
    {
      action: 'deliver',
      toStatus: 'delivered',
      href: (order) => `/orders/${order.id}/deliver`,
      method: 'POST',
      title: 'Mark as delivered',
    },
  ],
  delivered: [],
  cancelled: [],
};

// Generate links based on current state
function getOrderLinks(order: Order, baseUrl: string): Record<string, HalLink> {
  const links: Record<string, HalLink> = {
    self: { href: `${baseUrl}/orders/${order.id}` },
    collection: { href: `${baseUrl}/orders` },
    customer: { href: `${baseUrl}/users/${order.customerId}` },
    items: { href: `${baseUrl}/orders/${order.id}/items` },
  };

  // Add state-specific action links
  const transitions = orderTransitions[order.status] || [];
  for (const transition of transitions) {
    links[transition.action] = {
      href: baseUrl + transition.href(order),
      method: transition.method,
      title: transition.title,
    };
  }

  // Add invoice link for paid+ orders
  if (['paid', 'shipped', 'delivered'].includes(order.status)) {
    links.invoice = {
      href: `${baseUrl}/orders/${order.id}/invoice`,
      title: 'Download invoice',
      type: 'application/pdf',
    };
  }

  // Add tracking link for shipped orders
  if (['shipped', 'delivered'].includes(order.status) && order.trackingNumber) {
    links.tracking = {
      href: `https://tracking.example.com/${order.trackingNumber}`,
      title: 'Track shipment',
    };
  }

  return links;
}

// ════════════════════════════════════════════════════════════════
// CLIENT-SIDE USAGE
// ════════════════════════════════════════════════════════════════

// Client discovers what actions are available
async function renderOrderActions(orderId: string) {
  const response = await fetch(`/api/orders/${orderId}`);
  const order = await response.json();
  
  const actions = [];
  
  // Check what links are available
  if (order._links.pay) {
    actions.push({
      label: order._links.pay.title || 'Pay',
      onClick: () => fetch(order._links.pay.href, { method: 'POST' }),
    });
  }
  
  if (order._links.cancel) {
    actions.push({
      label: order._links.cancel.title || 'Cancel',
      onClick: () => fetch(order._links.cancel.href, { method: 'POST' }),
    });
  }
  
  if (order._links.ship) {
    actions.push({
      label: order._links.ship.title || 'Ship',
      onClick: () => fetch(order._links.ship.href, { method: 'POST' }),
    });
  }
  
  if (order._links.invoice) {
    actions.push({
      label: order._links.invoice.title || 'Invoice',
      onClick: () => window.open(order._links.invoice.href),
    });
  }
  
  return actions;
}

// Client navigates via links, not hardcoded URLs
async function navigateToOrders(apiRoot: any) {
  // Follow link from root
  const ordersLink = apiRoot._links.orders;
  
  // Handle templated link
  let ordersUrl = ordersLink.href;
  if (ordersLink.templated) {
    ordersUrl = ordersUrl.replace('{?status,page,limit}', '?status=pending&page=1&limit=20');
  }
  
  const response = await fetch(ordersUrl);
  return response.json();
}
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# HYPERMEDIA API BEST PRACTICES
# ════════════════════════════════════════════════════════════════

link_design:
  always_include:
    - self: Link to current resource
    - collection: Link to parent collection
    
  standard_relations:
    - Use IANA link relations when possible
    - https://www.iana.org/assignments/link-relations
    - Common: self, collection, next, prev, first, last
    
  custom_relations:
    - Use curies for documentation
    - Or full URIs: https://example.com/rels/custom
    - Consistent naming (kebab-case or camelCase)

state_driven:
  - Only include links for valid transitions
  - Order has "cancel" only when cancellable
  - Admin actions only for admin users
  - Links reflect what user CAN do

embedded_resources:
  - Embed to reduce round trips
  - Don't over-embed (response size)
  - Consider: first page of related collection
  - Client can follow link for more

response_format:
  content_type: application/hal+json
  # Or: application/vnd.api+json (JSON:API)
  
  structure:
    - Resource properties at top level
    - _links for navigation
    - _embedded for related resources
    - Consistent across all endpoints

root_endpoint:
  - Always provide GET /api entry point
  - List all available resources
  - Use templated links for search
  - Include API version and documentation link
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# HYPERMEDIA API PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Hardcoded URLs in client
# Bad - Client code
const ordersUrl = "/api/orders";
const userOrdersUrl = `/api/users/${userId}/orders`;

# Good - Client follows links
const ordersUrl = apiRoot._links.orders.href;
const userOrdersUrl = user._links.orders.href;

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Missing self link
# Bad
{
  "id": "123",
  "name": "Product"
  // No way to reference this resource
}

# Good
{
  "id": "123",
  "name": "Product",
  "_links": {
    "self": { "href": "/products/123" }
  }
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Always including all links
# Bad - Cancel link even for delivered order
"_links": {
  "cancel": { "href": "/orders/123/cancel" }  // Order is delivered!
}

# Good - State-driven links
"_links": {
  // No cancel for delivered order
  "invoice": { "href": "/orders/123/invoice" }
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Over-embedding
# Bad
{
  "_embedded": {
    "all_orders": [...thousands of orders...],
    "all_products": [...],
  }
}

# Good
{
  "_embedded": {
    "recent_orders": [...first 5...],
  },
  "_links": {
    "all_orders": { "href": "/users/123/orders" }
  }
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: No root endpoint
# Bad - Client must know about /users, /orders, etc.

# Good - Discoverable from root
GET /api
{
  "_links": {
    "users": { "href": "/api/users" },
    "orders": { "href": "/api/orders" },
    "products": { "href": "/api/products" }
  }
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Inconsistent link structure
# Bad
{
  "_links": {
    "self": "/products/123",           // Just string
    "category": { "url": "/categories/456" },  // Wrong property
    "reviews": { "href": "/products/123/reviews", "rel": "reviews" }  // Redundant
  }
}

# Good
{
  "_links": {
    "self": { "href": "/products/123" },
    "category": { "href": "/categories/456" },
    "reviews": { "href": "/products/123/reviews" }
  }
}
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is HATEOAS?"**
> "Hypermedia As The Engine Of Application State. API responses include links to related resources and available actions. Client navigates by following links, not hardcoding URLs. It's Richardson Maturity Level 3 - true REST. Makes API discoverable and self-documenting."

**Q: "What is HAL?"**
> "Hypertext Application Language - a format for hypermedia APIs. Uses _links for navigation and _embedded for related resources. Media type is application/hal+json. Simple, widely adopted. Alternative: JSON:API which is more opinionated."

**Q: "Why use hypermedia?"**
> "Discoverability: clients find resources via links. Loose coupling: server can change URLs. Self-documenting: links show what's possible. State-driven: links change based on resource state. Tradeoff: larger responses, more complex."

### Intermediate Questions

**Q: "How do state-driven links work?"**
> "Links in response depend on resource state. Order with status 'pending' has 'cancel' link. Order with status 'delivered' has no 'cancel', but has 'invoice'. Client UI enables buttons based on link presence. Business rules are in API, not client."

**Q: "What is the root endpoint?"**
> "Entry point for API discovery at GET /api. Lists all available resources with links. Client starts here, follows links to navigate. Like a homepage. Includes templated links for search/filter. Should include API version and documentation link."

**Q: "What are curies?"**
> "Compact URIs - shorthand for custom link relations with documentation. Define curie like 'ex' pointing to docs template. Use 'ex:customer' as relation name. Client can look up https://docs.example.com/rels/customer. Documents custom relations."

### Advanced Questions

**Q: "What are the tradeoffs of HATEOAS?"**
> "Pros: Discoverability, loose coupling, self-documenting, evolvable API. Cons: Larger responses (~10-20%), more complex to build, many clients ignore links anyway, overhead for simple internal APIs. Worth it for public APIs, maybe overkill internally."

**Q: "How do you implement HATEOAS with GraphQL?"**
> "GraphQL is different paradigm - client specifies shape. No URLs to follow. Can include links in fields. Or use GraphQL connections (edges/pageInfo). HATEOAS is more REST-specific. GraphQL has its own discoverability via introspection."

**Q: "When would you NOT use HATEOAS?"**
> "Simple internal APIs where coupling is acceptable. Performance-critical endpoints where payload size matters. APIs with fixed, well-known structure. When clients are tightly controlled. Level 2 REST is often sufficient for internal microservices."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  HYPERMEDIA API CHECKLIST                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LINKS:                                                         │
│  □ self link on every resource                                 │
│  □ collection link to parent                                   │
│  □ related resource links                                      │
│  □ action links (POST/PUT/DELETE)                              │
│  □ State-driven (only valid actions)                           │
│                                                                 │
│  COLLECTION:                                                    │
│  □ Pagination links (first, prev, next, last)                  │
│  □ Create action link                                          │
│  □ Search/filter templated link                                │
│                                                                 │
│  ROOT ENDPOINT:                                                 │
│  □ GET /api entry point                                        │
│  □ Links to all resources                                      │
│  □ API version and documentation                               │
│                                                                 │
│  FORMAT:                                                        │
│  □ application/hal+json content type                           │
│  □ _links for navigation                                       │
│  □ _embedded for related resources                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

HAL STRUCTURE:
┌────────────────────────────────────────────────────────────────┐
│ {                                                              │
│   "id": "123",                                                 │
│   "name": "Resource",                                          │
│   "_links": {                                                  │
│     "self": { "href": "/resources/123" },                     │
│     "action": { "href": "/resources/123/action", "method": "POST" }│
│   },                                                           │
│   "_embedded": {                                               │
│     "related": [{ ... }]                                       │
│   }                                                            │
│ }                                                              │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

