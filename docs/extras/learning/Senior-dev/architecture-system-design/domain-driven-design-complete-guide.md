# 🏛️ Domain-Driven Design (DDD) Complete Guide

> A comprehensive guide to designing complex software by focusing on the core business domain, ubiquitous language, and tactical patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "DDD is a software design approach that focuses on modeling the core business domain with a shared language between developers and domain experts, using patterns like bounded contexts, aggregates, and entities."

### The 7 Key Concepts (Remember These!)
```
1. UBIQUITOUS LANGUAGE  → Same terms used by devs AND business (not "user" vs "customer")
2. BOUNDED CONTEXT      → Clear boundary where a model applies (Sales vs Shipping)
3. AGGREGATE            → Cluster of objects treated as a unit (Order + OrderLines)
4. AGGREGATE ROOT       → Single entry point to the aggregate (Order, not OrderLine)
5. ENTITY               → Object with identity that persists (User id=123)
6. VALUE OBJECT         → Object defined by attributes, no identity (Money, Address)
7. DOMAIN EVENT         → Something that happened in the domain (OrderPlaced)
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Ubiquitous language"** | "We use ubiquitous language so devs and business speak the same terms" |
| **"Bounded context"** | "Each microservice aligns with a bounded context" |
| **"Aggregate root"** | "Order is the aggregate root - you can't modify OrderLine directly" |
| **"Value object"** | "Money is a value object - $10 equals any other $10" |
| **"Anemic domain model"** | "We avoid anemic models - logic lives in the domain, not services" |
| **"Context mapping"** | "Context mapping shows how bounded contexts communicate" |
| **"Anti-corruption layer"** | "We use an ACL to translate between legacy and new systems" |
| **"Invariant"** | "The aggregate protects domain invariants" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Aggregate size | **1-3 entities** | Smaller = easier to maintain consistency |
| Bounded context team | **1 team** | Conway's Law - team owns context |
| Transaction boundary | **1 aggregate** | Don't span transactions across aggregates |
| Entity vs Value Object | **80% VO** | Most objects should be value objects |

### Entity vs Value Object (Memorize This!)
| | Entity | Value Object |
|--|--------|--------------|
| **Identity** | Has unique ID | No ID, defined by attributes |
| **Equality** | Same ID = same entity | Same attributes = equal |
| **Mutability** | Can change over time | Immutable (create new instead) |
| **Example** | User, Order, Product | Money, Address, DateRange |
| **Question** | "Is this THE specific one?" | "Is this equivalent to another?" |

### The "Wow" Statement (Memorize This!)
> "Most codebases fail because developers and business people speak different languages. DDD solves this by creating a ubiquitous language shared by everyone. When a developer says 'Order' and a business person says 'Order', they mean exactly the same thing. The bounded context ensures this language is consistent within a boundary. The aggregate pattern then protects business rules - you can't create an Order without at least one OrderLine because the aggregate enforces that invariant."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                         DOMAIN                                   │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐      ┌─────────────────┐                  │
│  │ BOUNDED CONTEXT │      │ BOUNDED CONTEXT │                  │
│  │     "Sales"     │      │   "Shipping"    │                  │
│  │                 │      │                 │                  │
│  │ ┌─────────────┐ │      │ ┌─────────────┐ │                  │
│  │ │  Aggregate  │ │      │ │  Aggregate  │ │                  │
│  │ │   (Order)   │ │      │ │ (Shipment)  │ │                  │
│  │ │  ┌───────┐  │ │      │ │  ┌───────┐  │ │                  │
│  │ │  │Entity │  │ │      │ │  │Entity │  │ │                  │
│  │ │  │Order  │◄─┼─┼──────┼─┼──│Shipment│ │ │                  │
│  │ │  └───────┘  │ │ Event│ │  └───────┘  │ │                  │
│  │ │  ┌───────┐  │ │      │ │  ┌───────┐  │ │                  │
│  │ │  │Value  │  │ │      │ │  │Value  │  │ │                  │
│  │ │  │Object │  │ │      │ │  │Object │  │ │                  │
│  │ │  │Money  │  │ │      │ │  │Address│  │ │                  │
│  │ │  └───────┘  │ │      │ │  └───────┘  │ │                  │
│  │ └─────────────┘ │      │ └─────────────┘ │                  │
│  └─────────────────┘      └─────────────────┘                  │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is DDD?"**
> "Design approach that models the business domain with ubiquitous language, bounded contexts, and tactical patterns like aggregates and entities."

**Q: "What is a bounded context?"**
> "A boundary where a domain model and its language apply. 'Customer' in Sales means something different than in Support."

**Q: "Entity vs Value Object?"**
> "Entity has identity (User id=123). Value object has no identity, defined by attributes ($10 = any $10). Value objects are immutable."

**Q: "What is an aggregate?"**
> "Cluster of objects treated as a unit with one root. You can't modify children directly - go through the root. It protects invariants."

**Q: "When NOT to use DDD?"**
> "Simple CRUD apps, small teams, well-understood domains. DDD adds complexity - only worth it for complex business logic."

**Q: "What is anemic domain model?"**
> "Anti-pattern where entities are just data bags and all logic is in services. Domain should contain behavior, not just data."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "What is Domain-Driven Design?"

**Junior Answer:**
> "DDD is a way to organize code using patterns like entities and repositories."

**Senior Answer:**
> "DDD is a software design philosophy that tackles complexity by focusing on the core business domain. It has two parts:

**Strategic Design** - The big picture:
- **Ubiquitous Language**: Developers and domain experts use the same terminology. If business says 'Policy', code has a `Policy` class, not `Contract`.
- **Bounded Contexts**: Clear boundaries where a model is valid. 'Customer' in Sales (buyer info) differs from 'Customer' in Shipping (delivery address).
- **Context Mapping**: How contexts relate - shared kernel, customer-supplier, anti-corruption layer.

**Tactical Design** - Implementation patterns:
- **Entities**: Objects with identity that persist over time
- **Value Objects**: Immutable objects defined by attributes
- **Aggregates**: Transaction boundaries that protect business rules
- **Domain Events**: Capture what happened for other contexts to react

The key insight is that software should model how the business actually thinks, not how the database is structured."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "How do you identify bounded contexts?" | "Event Storming workshops with domain experts. Look for where language changes - same word, different meaning = different context." |
| "How big should an aggregate be?" | "As small as possible while protecting invariants. Large aggregates cause concurrency issues. Prefer eventual consistency between aggregates." |
| "How do aggregates communicate?" | "Through domain events. Order aggregate publishes OrderPlaced, Shipping aggregate reacts. Never reference another aggregate directly." |
| "What's the difference from Clean Architecture?" | "DDD is about modeling the domain. Clean Architecture is about dependency direction. They complement each other - DDD inside Clean Architecture's domain layer." |

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Strategic Design](#2-strategic-design)
3. [Tactical Design](#3-tactical-design)
4. [Domain Events & Services](#4-domain-events--services)
5. [Layers & Architecture](#5-layers--architecture)
6. [Implementation Examples](#6-implementation-examples)
7. [Common Mistakes](#7-common-mistakes)
8. [When to Use / Not Use DDD](#8-when-to-use--not-use-ddd)
9. [Interview Questions](#9-interview-questions)

---

## 1. Core Concepts

### What Problem Does DDD Solve?

```
THE PROBLEM:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Business Expert says:     Developer understands:              │
│   "Create a Policy"    →    "INSERT INTO contracts..."         │
│   "Apply discount"     →    "UPDATE price SET..."              │
│   "Customer churned"   →    "user.status = 'inactive'"         │
│                                                                  │
│   Result: Miscommunication, bugs, wrong features                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

THE SOLUTION (DDD):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Business Expert says:     Developer writes:                   │
│   "Create a Policy"    →    policy = Policy.create(...)        │
│   "Apply discount"     →    order.applyDiscount(discount)      │
│   "Customer churned"   →    customer.markAsChurned()           │
│                                                                  │
│   Code mirrors business language exactly                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The Two Parts of DDD

```
DOMAIN-DRIVEN DESIGN
         │
         ├── STRATEGIC DESIGN (Big Picture)
         │   ├── Ubiquitous Language
         │   ├── Bounded Contexts
         │   ├── Context Mapping
         │   └── Subdomains (Core, Supporting, Generic)
         │
         └── TACTICAL DESIGN (Implementation)
             ├── Entities
             ├── Value Objects
             ├── Aggregates
             ├── Domain Events
             ├── Repositories
             ├── Domain Services
             ├── Application Services
             └── Factories
```

### Ubiquitous Language

The **single most important concept** in DDD.

```typescript
// ❌ BAD: Developer terminology
class User {
  data: UserData;
  updateData(data: UserData) { this.data = data; }
}

// ✅ GOOD: Business terminology (Ubiquitous Language)
class Customer {
  private shippingAddress: Address;
  private billingAddress: Address;
  
  relocate(newAddress: Address): void {
    // "Relocate" is what business calls it, not "updateAddress"
    this.shippingAddress = newAddress;
    this.addDomainEvent(new CustomerRelocated(this.id, newAddress));
  }
  
  markAsChurned(reason: ChurnReason): void {
    // "Churned" is business term, not "deactivate" or "set inactive"
    this.status = CustomerStatus.Churned;
    this.churnedAt = new Date();
    this.addDomainEvent(new CustomerChurned(this.id, reason));
  }
}
```

### Building the Ubiquitous Language

```
HOW TO BUILD IT:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. EVENT STORMING                                              │
│     └── Workshop with domain experts                            │
│     └── Identify domain events on sticky notes                  │
│     └── "What happens when...?"                                 │
│                                                                  │
│  2. GLOSSARY                                                    │
│     └── Document every term                                     │
│     └── "Policy" means X, not Y                                 │
│     └── Update when language evolves                            │
│                                                                  │
│  3. CODE = LANGUAGE                                             │
│     └── Class names match business terms                        │
│     └── Method names match business actions                     │
│     └── No translation layer in developer's head               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Subdomains

Not all parts of your domain are equally important.

```
┌─────────────────────────────────────────────────────────────────┐
│                    SUBDOMAIN TYPES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CORE DOMAIN (Invest heavily!)                                  │
│  └── Your competitive advantage                                 │
│  └── What makes you unique                                      │
│  └── Example: Pricing algorithm for insurance company          │
│  └── Build in-house with best developers                        │
│                                                                  │
│  SUPPORTING SUBDOMAIN (Build or buy)                           │
│  └── Necessary but not competitive advantage                    │
│  └── Example: Customer management                               │
│  └── Can be custom but doesn't need A-team                     │
│                                                                  │
│  GENERIC SUBDOMAIN (Buy off-the-shelf)                         │
│  └── Solved problems with standard solutions                    │
│  └── Example: Authentication, email, payments                   │
│  └── Use existing solutions (Auth0, Stripe, SendGrid)          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

Example for E-commerce:
┌──────────────────┬────────────────────┬──────────────────────┐
│   CORE DOMAIN    │    SUPPORTING      │      GENERIC         │
├──────────────────┼────────────────────┼──────────────────────┤
│ Recommendation   │ Order Management   │ Authentication       │
│ Engine           │ Inventory          │ Payments (Stripe)    │
│ Pricing Strategy │ Customer Support   │ Email (SendGrid)     │
│                  │ Shipping           │ Analytics (Mixpanel) │
└──────────────────┴────────────────────┴──────────────────────┘
```

---

## 2. Strategic Design

### Bounded Contexts

A **bounded context** is a boundary within which a particular model is defined and applicable.

```
THE KEY INSIGHT:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   The word "Customer" means different things:                   │
│                                                                  │
│   SALES CONTEXT:           SHIPPING CONTEXT:                    │
│   ┌─────────────────┐     ┌─────────────────┐                  │
│   │ Customer        │     │ Customer        │                  │
│   │ - name          │     │ - deliveryAddr  │                  │
│   │ - email         │     │ - contactPhone  │                  │
│   │ - creditLimit   │     │ - deliveryNotes │                  │
│   │ - purchaseHist  │     │                 │                  │
│   └─────────────────┘     └─────────────────┘                  │
│                                                                  │
│   Same word, DIFFERENT models!                                  │
│   This is why we need BOUNDARIES.                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Identifying Bounded Contexts

```
SIGNALS THAT YOU NEED SEPARATE CONTEXTS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. LANGUAGE CHANGES                                            │
│     └── Same term means different things                        │
│     └── Different terms for same concept                        │
│     └── "We call it X, but they call it Y"                     │
│                                                                  │
│  2. DIFFERENT EXPERTS                                           │
│     └── Sales team vs Support team                             │
│     └── Different people own different parts                   │
│                                                                  │
│  3. DIFFERENT RULES                                             │
│     └── Business rules differ by area                          │
│     └── Validation differs                                     │
│                                                                  │
│  4. DIFFERENT LIFECYCLE                                         │
│     └── Data changes at different rates                        │
│     └── Different retention requirements                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Real Example: E-commerce Bounded Contexts

```typescript
// ════════════════════════════════════════════════════════════════
// SALES CONTEXT - Focused on purchasing
// ════════════════════════════════════════════════════════════════
namespace Sales {
  class Customer {
    id: CustomerId;
    email: Email;
    creditLimit: Money;
    loyaltyTier: LoyaltyTier;
    
    canPurchase(amount: Money): boolean {
      return amount.isLessThan(this.creditLimit);
    }
  }

  class Order {
    id: OrderId;
    customerId: CustomerId;  // Reference by ID, not object!
    lines: OrderLine[];
    
    calculateTotal(): Money { /* ... */ }
    applyDiscount(discount: Discount): void { /* ... */ }
  }

  class Product {
    id: ProductId;
    name: string;
    price: Money;
    discountable: boolean;
  }
}

// ════════════════════════════════════════════════════════════════
// SHIPPING CONTEXT - Focused on delivery
// ════════════════════════════════════════════════════════════════
namespace Shipping {
  class Customer {
    // Different model! Only shipping-relevant data
    id: CustomerId;
    deliveryAddress: Address;
    contactPhone: Phone;
    deliveryInstructions: string;
    preferredTimeSlot: TimeSlot;
  }

  class Shipment {
    id: ShipmentId;
    orderId: OrderId;  // Reference to Sales context
    destination: Address;
    packages: Package[];
    carrier: Carrier;
    
    dispatch(): void { /* ... */ }
    markDelivered(signature: Signature): void { /* ... */ }
  }

  class Product {
    // Completely different! Shipping cares about physical properties
    id: ProductId;
    weight: Weight;
    dimensions: Dimensions;
    isFragile: boolean;
    requiresRefrigeration: boolean;
  }
}

// ════════════════════════════════════════════════════════════════
// INVENTORY CONTEXT - Focused on stock management
// ════════════════════════════════════════════════════════════════
namespace Inventory {
  class Product {
    // Yet another view of Product!
    sku: SKU;
    warehouseLocation: Location;
    quantityOnHand: number;
    reorderPoint: number;
    
    reserve(quantity: number): Reservation { /* ... */ }
    restock(quantity: number): void { /* ... */ }
  }
}
```

### Context Mapping

How bounded contexts relate to each other.

```
CONTEXT MAP PATTERNS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. SHARED KERNEL                                               │
│     └── Two contexts share a subset of the model               │
│     └── Both teams must agree on changes                       │
│     └── Tight coupling - use sparingly!                        │
│                                                                  │
│     ┌──────────┐  shared  ┌──────────┐                        │
│     │ Context A │◄───────►│ Context B │                        │
│     └──────────┘  model   └──────────┘                        │
│                                                                  │
│  2. CUSTOMER-SUPPLIER                                           │
│     └── Upstream (supplier) provides what downstream needs     │
│     └── Downstream (customer) can request features             │
│     └── Clear dependency direction                             │
│                                                                  │
│     ┌──────────┐          ┌──────────┐                        │
│     │ Supplier │─────────►│ Customer │                        │
│     │(upstream)│  feeds   │(downstream)                        │
│     └──────────┘          └──────────┘                        │
│                                                                  │
│  3. CONFORMIST                                                  │
│     └── Downstream conforms to upstream's model                │
│     └── No negotiation - take it or leave it                   │
│     └── Common with external APIs                              │
│                                                                  │
│  4. ANTI-CORRUPTION LAYER (ACL)                                │
│     └── Translation layer between contexts                     │
│     └── Protects your model from external influence           │
│     └── Essential for legacy integration!                      │
│                                                                  │
│     ┌──────────┐   ┌─────┐   ┌──────────┐                    │
│     │ Legacy   │──►│ ACL │──►│ Your     │                    │
│     │ System   │   └─────┘   │ Context  │                    │
│     └──────────┘  translates └──────────┘                    │
│                                                                  │
│  5. OPEN HOST SERVICE                                          │
│     └── Well-defined API for multiple consumers               │
│     └── Published language others can use                      │
│                                                                  │
│  6. PUBLISHED LANGUAGE                                         │
│     └── Shared language (often JSON/XML schema)               │
│     └── Standard format for integration                        │
│                                                                  │
│  7. SEPARATE WAYS                                              │
│     └── No integration - completely independent               │
│     └── Sometimes the best choice!                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Anti-Corruption Layer (ACL) Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// PROBLEM: Legacy system has different model
// ════════════════════════════════════════════════════════════════

// Legacy CRM system returns this mess
interface LegacyCRMCustomer {
  CUST_ID: string;
  CUST_NM: string;
  CUST_EMAIL_ADDR: string;
  CUST_STAT_CD: 'A' | 'I' | 'D';
  CUST_CR_LMT: number;
  LST_UPDT_TS: string;
}

// ════════════════════════════════════════════════════════════════
// SOLUTION: Anti-Corruption Layer translates
// ════════════════════════════════════════════════════════════════

// Your clean domain model
class Customer {
  constructor(
    public readonly id: CustomerId,
    public readonly name: CustomerName,
    public readonly email: Email,
    public readonly status: CustomerStatus,
    public readonly creditLimit: Money
  ) {}
}

enum CustomerStatus {
  Active = 'active',
  Inactive = 'inactive',
  Deleted = 'deleted'
}

// ACL: Translator
class CustomerTranslator {
  fromLegacy(legacy: LegacyCRMCustomer): Customer {
    return new Customer(
      new CustomerId(legacy.CUST_ID),
      new CustomerName(legacy.CUST_NM),
      new Email(legacy.CUST_EMAIL_ADDR),
      this.translateStatus(legacy.CUST_STAT_CD),
      Money.fromCents(legacy.CUST_CR_LMT)
    );
  }

  toLegacy(customer: Customer): LegacyCRMCustomer {
    return {
      CUST_ID: customer.id.value,
      CUST_NM: customer.name.value,
      CUST_EMAIL_ADDR: customer.email.value,
      CUST_STAT_CD: this.translateStatusToLegacy(customer.status),
      CUST_CR_LMT: customer.creditLimit.cents,
      LST_UPDT_TS: new Date().toISOString()
    };
  }

  private translateStatus(code: string): CustomerStatus {
    const mapping: Record<string, CustomerStatus> = {
      'A': CustomerStatus.Active,
      'I': CustomerStatus.Inactive,
      'D': CustomerStatus.Deleted
    };
    return mapping[code] ?? CustomerStatus.Inactive;
  }
}

// ACL: Adapter wraps legacy service
class LegacyCustomerAdapter implements CustomerRepository {
  constructor(
    private legacyClient: LegacyCRMClient,
    private translator: CustomerTranslator
  ) {}

  async findById(id: CustomerId): Promise<Customer | null> {
    const legacy = await this.legacyClient.getCustomer(id.value);
    if (!legacy) return null;
    
    // Translate from legacy to our clean model
    return this.translator.fromLegacy(legacy);
  }

  async save(customer: Customer): Promise<void> {
    // Translate from our model to legacy format
    const legacy = this.translator.toLegacy(customer);
    await this.legacyClient.updateCustomer(legacy);
  }
}

// Your domain code stays CLEAN - no legacy pollution
class OrderService {
  constructor(private customers: CustomerRepository) {}

  async canCustomerOrder(customerId: CustomerId, amount: Money): Promise<boolean> {
    const customer = await this.customers.findById(customerId);
    if (!customer) return false;
    
    // Works with clean domain model
    // Doesn't know about legacy CUST_STAT_CD nonsense
    return customer.status === CustomerStatus.Active 
        && customer.creditLimit.isGreaterThan(amount);
  }
}
```

### Context Map Diagram Example

```
┌─────────────────────────────────────────────────────────────────┐
│                    E-COMMERCE CONTEXT MAP                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                        ┌─────────────┐                          │
│                        │   SALES     │                          │
│                        │  (Core)     │                          │
│                        └──────┬──────┘                          │
│                               │                                  │
│          ┌────────────────────┼────────────────────┐            │
│          │                    │                    │            │
│          ▼                    ▼                    ▼            │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐      │
│   │  SHIPPING   │     │  INVENTORY  │     │   BILLING   │      │
│   │ (Supporting)│     │ (Supporting)│     │ (Supporting)│      │
│   └──────┬──────┘     └─────────────┘     └──────┬──────┘      │
│          │                                        │             │
│          │ Conformist                    ACL      │             │
│          ▼                                        ▼             │
│   ┌─────────────┐                         ┌─────────────┐      │
│   │  CARRIER    │                         │  PAYMENT    │      │
│   │   (3rd      │                         │  GATEWAY    │      │
│   │   party)    │                         │  (Stripe)   │      │
│   └─────────────┘                         └─────────────┘      │
│                                                                  │
│  LEGEND:                                                        │
│  ───────► Customer-Supplier                                     │
│  ═══════► Conformist                                            │
│  ──ACL──► Anti-Corruption Layer                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Bounded Context vs Microservice

```
IMPORTANT DISTINCTION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Bounded Context ≠ Microservice (but often 1:1)                │
│                                                                  │
│  BOUNDED CONTEXT                    MICROSERVICE                │
│  └── Logical boundary               └── Deployment unit        │
│  └── Language/model scope           └── Physical boundary      │
│  └── Design concept                 └── Architecture pattern   │
│                                                                  │
│  RELATIONSHIP:                                                  │
│  └── 1 Bounded Context = 1 Microservice (ideal)               │
│  └── 1 Bounded Context = N Microservices (if too big)         │
│  └── 1 Microservice = N Bounded Contexts (AVOID! 🚫)          │
│                                                                  │
│  A microservice that spans multiple bounded contexts           │
│  = DISTRIBUTED MONOLITH = worst of both worlds                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Tactical Design

### Entities

An **Entity** is an object defined by its identity, not its attributes.

```typescript
// ════════════════════════════════════════════════════════════════
// ENTITY: Defined by IDENTITY
// ════════════════════════════════════════════════════════════════

// Two users with same name are DIFFERENT people
// Identity matters, not attributes

class User {
  private constructor(
    private readonly _id: UserId,
    private _name: UserName,
    private _email: Email,
    private _status: UserStatus
  ) {}

  // Identity
  get id(): UserId {
    return this._id;
  }

  // Can change over time
  changeName(newName: UserName): void {
    this._name = newName;
    this.addDomainEvent(new UserNameChanged(this._id, newName));
  }

  changeEmail(newEmail: Email): void {
    this._email = newEmail;
    this.addDomainEvent(new UserEmailChanged(this._id, newEmail));
  }

  // Equality based on ID, not attributes
  equals(other: User): boolean {
    return this._id.equals(other._id);
  }

  // Factory method ensures valid creation
  static create(name: UserName, email: Email): User {
    const id = UserId.generate();
    return new User(id, name, email, UserStatus.Active);
  }

  // Reconstitute from persistence (no validation - already validated)
  static reconstitute(id: UserId, name: UserName, email: Email, status: UserStatus): User {
    return new User(id, name, email, status);
  }
}

// Entity ID as Value Object (best practice)
class UserId {
  private constructor(private readonly value: string) {
    if (!value || value.trim().length === 0) {
      throw new Error('UserId cannot be empty');
    }
  }

  static generate(): UserId {
    return new UserId(crypto.randomUUID());
  }

  static fromString(value: string): UserId {
    return new UserId(value);
  }

  equals(other: UserId): boolean {
    return this.value === other.value;
  }

  toString(): string {
    return this.value;
  }
}
```

### Value Objects

A **Value Object** is defined by its attributes, has no identity, and is immutable.

```typescript
// ════════════════════════════════════════════════════════════════
// VALUE OBJECT: Defined by ATTRIBUTES, no identity, immutable
// ════════════════════════════════════════════════════════════════

// $10 bill is same as any other $10 bill - no unique identity
// Two Addresses with same street, city, zip are EQUAL

class Money {
  private constructor(
    private readonly amount: number,
    private readonly currency: Currency
  ) {
    if (amount < 0) {
      throw new Error('Money cannot be negative');
    }
  }

  // Named constructors
  static dollars(amount: number): Money {
    return new Money(amount, Currency.USD);
  }

  static euros(amount: number): Money {
    return new Money(amount, Currency.EUR);
  }

  static zero(currency: Currency = Currency.USD): Money {
    return new Money(0, currency);
  }

  // IMMUTABLE: Operations return NEW instances
  add(other: Money): Money {
    this.ensureSameCurrency(other);
    return new Money(this.amount + other.amount, this.currency);
  }

  subtract(other: Money): Money {
    this.ensureSameCurrency(other);
    const result = this.amount - other.amount;
    if (result < 0) {
      throw new Error('Insufficient funds');
    }
    return new Money(result, this.currency);
  }

  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }

  // Equality based on ATTRIBUTES, not identity
  equals(other: Money): boolean {
    return this.amount === other.amount 
        && this.currency === other.currency;
  }

  isGreaterThan(other: Money): boolean {
    this.ensureSameCurrency(other);
    return this.amount > other.amount;
  }

  private ensureSameCurrency(other: Money): void {
    if (this.currency !== other.currency) {
      throw new Error('Cannot compare different currencies');
    }
  }
}

// Another Value Object: Address
class Address {
  private constructor(
    public readonly street: string,
    public readonly city: string,
    public readonly state: string,
    public readonly zipCode: string,
    public readonly country: string
  ) {
    // Validation in constructor
    if (!street || !city || !zipCode) {
      throw new Error('Invalid address');
    }
  }

  static create(street: string, city: string, state: string, zipCode: string, country: string): Address {
    return new Address(street, city, state, zipCode, country);
  }

  // Change = create new instance
  withStreet(newStreet: string): Address {
    return new Address(newStreet, this.city, this.state, this.zipCode, this.country);
  }

  equals(other: Address): boolean {
    return this.street === other.street
        && this.city === other.city
        && this.state === other.state
        && this.zipCode === other.zipCode
        && this.country === other.country;
  }

  format(): string {
    return `${this.street}, ${this.city}, ${this.state} ${this.zipCode}, ${this.country}`;
  }
}

// Value Object: DateRange
class DateRange {
  private constructor(
    public readonly start: Date,
    public readonly end: Date
  ) {
    if (start > end) {
      throw new Error('Start date must be before end date');
    }
  }

  static create(start: Date, end: Date): DateRange {
    return new DateRange(start, end);
  }

  contains(date: Date): boolean {
    return date >= this.start && date <= this.end;
  }

  overlaps(other: DateRange): boolean {
    return this.start <= other.end && this.end >= other.start;
  }

  durationInDays(): number {
    const diff = this.end.getTime() - this.start.getTime();
    return Math.ceil(diff / (1000 * 60 * 60 * 24));
  }
}
```

### Entity vs Value Object Decision Guide

```
HOW TO DECIDE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Ask: "Do I care about WHICH specific one?"                    │
│                                                                  │
│  YES → ENTITY                    NO → VALUE OBJECT              │
│  └── Which user?                 └── Which $10 bill?           │
│  └── Which order?                └── Which address format?     │
│  └── Which product?              └── Which color red?          │
│                                                                  │
│  Ask: "Can I replace it with another equivalent one?"          │
│                                                                  │
│  NO → ENTITY                     YES → VALUE OBJECT             │
│  └── Can't swap users            └── Any $10 works             │
│  └── This specific order         └── Same address is same     │
│                                                                  │
│  Ask: "Does it change over time while remaining 'the same'?"   │
│                                                                  │
│  YES → ENTITY                    NO → VALUE OBJECT              │
│  └── User changes name,          └── $10 is always $10        │
│      still same user             └── Address changes =        │
│                                       new address              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

RULE OF THUMB:
┌─────────────────────────────────────────────────────────────────┐
│  80% Value Objects, 20% Entities                                │
│                                                                  │
│  Most things should be Value Objects!                          │
│  Only use Entity when identity truly matters.                  │
└─────────────────────────────────────────────────────────────────┘
```

### Aggregates

An **Aggregate** is a cluster of domain objects treated as a single unit for data changes.

```
AGGREGATE RULES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. ONE AGGREGATE ROOT                                          │
│     └── Single entry point to the aggregate                    │
│     └── Only root has global identity                          │
│     └── External objects can ONLY reference the root           │
│                                                                  │
│  2. TRANSACTIONAL CONSISTENCY                                   │
│     └── Changes within aggregate = single transaction          │
│     └── Changes across aggregates = eventual consistency       │
│                                                                  │
│  3. PROTECT INVARIANTS                                          │
│     └── Aggregate enforces business rules                      │
│     └── Can't create invalid state                             │
│                                                                  │
│  4. REFERENCE BY ID ONLY                                        │
│     └── Don't hold references to other aggregates             │
│     └── Store their ID, load when needed                       │
│                                                                  │
│  5. SMALL AGGREGATES                                            │
│     └── Prefer smaller aggregates                              │
│     └── Large aggregates = concurrency problems               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Aggregate Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// ORDER AGGREGATE
// Order is the root, OrderLine is a child entity
// ════════════════════════════════════════════════════════════════

class Order {
  private _id: OrderId;
  private _customerId: CustomerId;  // Reference by ID!
  private _lines: OrderLine[] = [];
  private _status: OrderStatus;
  private _createdAt: Date;
  private _domainEvents: DomainEvent[] = [];

  private constructor(
    id: OrderId,
    customerId: CustomerId
  ) {
    this._id = id;
    this._customerId = customerId;
    this._status = OrderStatus.Draft;
    this._createdAt = new Date();
  }

  // ══════════════════════════════════════════════════════════════
  // INVARIANT: Order must have at least one line to be placed
  // INVARIANT: Cannot modify placed order
  // INVARIANT: Total must be positive
  // ══════════════════════════════════════════════════════════════

  // Factory method
  static create(customerId: CustomerId): Order {
    const order = new Order(OrderId.generate(), customerId);
    order.addDomainEvent(new OrderCreated(order._id, customerId));
    return order;
  }

  // ══════════════════════════════════════════════════════════════
  // BEHAVIOR: All modifications go through aggregate root
  // ══════════════════════════════════════════════════════════════

  addLine(productId: ProductId, quantity: number, unitPrice: Money): void {
    this.ensureCanModify();
    
    // Check if product already in order
    const existingLine = this._lines.find(l => l.productId.equals(productId));
    if (existingLine) {
      existingLine.increaseQuantity(quantity);
    } else {
      const line = new OrderLine(
        OrderLineId.generate(),
        productId,
        quantity,
        unitPrice
      );
      this._lines.push(line);
    }

    this.addDomainEvent(new OrderLineAdded(this._id, productId, quantity));
  }

  removeLine(productId: ProductId): void {
    this.ensureCanModify();
    
    const index = this._lines.findIndex(l => l.productId.equals(productId));
    if (index === -1) {
      throw new Error('Product not in order');
    }
    
    this._lines.splice(index, 1);
    this.addDomainEvent(new OrderLineRemoved(this._id, productId));
  }

  place(): void {
    // INVARIANT: Must have at least one line
    if (this._lines.length === 0) {
      throw new Error('Cannot place empty order');
    }

    // INVARIANT: Total must be positive
    if (this.total().equals(Money.zero())) {
      throw new Error('Order total must be positive');
    }

    this._status = OrderStatus.Placed;
    this.addDomainEvent(new OrderPlaced(this._id, this.total()));
  }

  cancel(reason: string): void {
    if (this._status === OrderStatus.Shipped) {
      throw new Error('Cannot cancel shipped order');
    }

    this._status = OrderStatus.Cancelled;
    this.addDomainEvent(new OrderCancelled(this._id, reason));
  }

  // ══════════════════════════════════════════════════════════════
  // QUERIES: Expose data safely
  // ══════════════════════════════════════════════════════════════

  get id(): OrderId { return this._id; }
  get customerId(): CustomerId { return this._customerId; }
  get status(): OrderStatus { return this._status; }
  
  // Return copy to prevent external modification
  get lines(): ReadonlyArray<OrderLine> { 
    return [...this._lines]; 
  }

  total(): Money {
    return this._lines.reduce(
      (sum, line) => sum.add(line.subtotal()),
      Money.zero()
    );
  }

  lineCount(): number {
    return this._lines.length;
  }

  // ══════════════════════════════════════════════════════════════
  // DOMAIN EVENTS
  // ══════════════════════════════════════════════════════════════

  pullDomainEvents(): DomainEvent[] {
    const events = [...this._domainEvents];
    this._domainEvents = [];
    return events;
  }

  private addDomainEvent(event: DomainEvent): void {
    this._domainEvents.push(event);
  }

  private ensureCanModify(): void {
    if (this._status !== OrderStatus.Draft) {
      throw new Error('Cannot modify order after it is placed');
    }
  }
}

// ════════════════════════════════════════════════════════════════
// ORDER LINE: Child entity (only accessible through Order)
// ════════════════════════════════════════════════════════════════

class OrderLine {
  constructor(
    private readonly _id: OrderLineId,
    private readonly _productId: ProductId,
    private _quantity: number,
    private readonly _unitPrice: Money
  ) {
    if (quantity <= 0) {
      throw new Error('Quantity must be positive');
    }
  }

  get id(): OrderLineId { return this._id; }
  get productId(): ProductId { return this._productId; }
  get quantity(): number { return this._quantity; }
  get unitPrice(): Money { return this._unitPrice; }

  subtotal(): Money {
    return this._unitPrice.multiply(this._quantity);
  }

  increaseQuantity(amount: number): void {
    if (amount <= 0) throw new Error('Amount must be positive');
    this._quantity += amount;
  }

  decreaseQuantity(amount: number): void {
    if (amount <= 0) throw new Error('Amount must be positive');
    if (this._quantity - amount <= 0) {
      throw new Error('Quantity would become zero or negative');
    }
    this._quantity -= amount;
  }
}
```

### Aggregate Boundaries - Getting it Right

```
WRONG: Too Large Aggregate
┌─────────────────────────────────────────────────────────────────┐
│                        Customer Aggregate                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Customer                                                  │   │
│  │ ├── Address[]                                            │   │
│  │ ├── PaymentMethod[]                                      │   │
│  │ ├── Order[]           ← Every order in the aggregate??  │   │
│  │ │   ├── OrderLine[]                                      │   │
│  │ │   └── ...                                              │   │
│  │ ├── Wishlist                                             │   │
│  │ ├── Reviews[]                                            │   │
│  │ └── ...                                                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  PROBLEMS:                                                      │
│  └── Locks entire customer on every order change               │
│  └── Loads megabytes of data to add one order line            │
│  └── Concurrency nightmare                                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

RIGHT: Small, Focused Aggregates
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐          │
│  │  Customer   │   │    Order    │   │   Review    │          │
│  │  Aggregate  │   │  Aggregate  │   │  Aggregate  │          │
│  ├─────────────┤   ├─────────────┤   ├─────────────┤          │
│  │ CustomerId  │◄──│ customerId  │   │ customerId  │──►       │
│  │ Name        │   │ OrderLine[] │   │ productId   │          │
│  │ Email       │   │ Status      │   │ Rating      │          │
│  │ Address[]   │   │ Total       │   │ Text        │          │
│  └─────────────┘   └─────────────┘   └─────────────┘          │
│                           │                                     │
│                           │ References by ID                   │
│                           ▼                                     │
│                    ┌─────────────┐                              │
│                    │   Product   │                              │
│                    │  Aggregate  │                              │
│                    └─────────────┘                              │
│                                                                  │
│  BENEFITS:                                                      │
│  └── Independent transactions                                  │
│  └── Better concurrency                                        │
│  └── Smaller, focused models                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Communication Between Aggregates

```typescript
// ════════════════════════════════════════════════════════════════
// WRONG: Direct reference between aggregates
// ════════════════════════════════════════════════════════════════

class Order {
  private customer: Customer;  // ❌ Direct reference
  
  place(): void {
    if (!this.customer.isActive()) {  // ❌ Reaches into another aggregate
      throw new Error('Customer inactive');
    }
  }
}

// ════════════════════════════════════════════════════════════════
// RIGHT: Reference by ID, communicate via events
// ════════════════════════════════════════════════════════════════

class Order {
  private customerId: CustomerId;  // ✅ Reference by ID
  
  // Validation done in Application Service before calling this
  place(): void {
    this._status = OrderStatus.Placed;
    this.addDomainEvent(new OrderPlaced(this._id, this.total()));
  }
}

// Application Service coordinates
class PlaceOrderService {
  constructor(
    private orders: OrderRepository,
    private customers: CustomerRepository
  ) {}

  async execute(orderId: OrderId): Promise<void> {
    const order = await this.orders.findById(orderId);
    const customer = await this.customers.findById(order.customerId);

    // Validation at application layer
    if (!customer.isActive()) {
      throw new Error('Customer inactive');
    }

    // Domain logic in aggregate
    order.place();

    await this.orders.save(order);
    
    // Events handled by separate handlers
    // Shipping context reacts to OrderPlaced
  }
}
```

---

## 4. Domain Events & Services

### Domain Events

A **Domain Event** captures something that happened in the domain that domain experts care about.

```typescript
// ════════════════════════════════════════════════════════════════
// DOMAIN EVENT STRUCTURE
// ════════════════════════════════════════════════════════════════

interface DomainEvent {
  eventId: string;
  occurredOn: Date;
  aggregateId: string;
  eventType: string;
}

// Naming: Past tense, business language
// ✅ OrderPlaced, PaymentReceived, CustomerRelocated
// ❌ OrderCreatedEvent, UpdateOrder, ProcessPayment

class OrderPlaced implements DomainEvent {
  readonly eventId: string;
  readonly occurredOn: Date;
  readonly eventType = 'OrderPlaced';

  constructor(
    public readonly aggregateId: string,  // OrderId
    public readonly customerId: string,
    public readonly total: Money,
    public readonly lineItems: Array<{
      productId: string;
      quantity: number;
      unitPrice: Money;
    }>
  ) {
    this.eventId = crypto.randomUUID();
    this.occurredOn = new Date();
  }
}

class CustomerRelocated implements DomainEvent {
  readonly eventId: string;
  readonly occurredOn: Date;
  readonly eventType = 'CustomerRelocated';

  constructor(
    public readonly aggregateId: string,  // CustomerId
    public readonly newAddress: Address,
    public readonly previousAddress: Address
  ) {
    this.eventId = crypto.randomUUID();
    this.occurredOn = new Date();
  }
}
```

### Raising Events from Aggregates

```typescript
// Base class for aggregates that raise events
abstract class AggregateRoot {
  private _domainEvents: DomainEvent[] = [];

  protected addDomainEvent(event: DomainEvent): void {
    this._domainEvents.push(event);
  }

  pullDomainEvents(): DomainEvent[] {
    const events = [...this._domainEvents];
    this._domainEvents = [];
    return events;
  }
}

// Order aggregate raises events
class Order extends AggregateRoot {
  place(): void {
    if (this._lines.length === 0) {
      throw new Error('Cannot place empty order');
    }

    this._status = OrderStatus.Placed;

    // Raise domain event - something happened!
    this.addDomainEvent(new OrderPlaced(
      this._id.value,
      this._customerId.value,
      this.total(),
      this._lines.map(l => ({
        productId: l.productId.value,
        quantity: l.quantity,
        unitPrice: l.unitPrice
      }))
    ));
  }

  cancel(reason: CancellationReason): void {
    if (this._status === OrderStatus.Shipped) {
      throw new Error('Cannot cancel shipped order');
    }

    this._status = OrderStatus.Cancelled;
    
    this.addDomainEvent(new OrderCancelled(
      this._id.value,
      reason,
      this.total()
    ));
  }
}
```

### Publishing Events

```typescript
// ════════════════════════════════════════════════════════════════
// EVENT DISPATCHER
// ════════════════════════════════════════════════════════════════

interface EventHandler<T extends DomainEvent> {
  handle(event: T): Promise<void>;
}

class DomainEventDispatcher {
  private handlers: Map<string, EventHandler<any>[]> = new Map();

  register<T extends DomainEvent>(
    eventType: string, 
    handler: EventHandler<T>
  ): void {
    const existing = this.handlers.get(eventType) || [];
    this.handlers.set(eventType, [...existing, handler]);
  }

  async dispatch(events: DomainEvent[]): Promise<void> {
    for (const event of events) {
      const handlers = this.handlers.get(event.eventType) || [];
      await Promise.all(
        handlers.map(h => h.handle(event))
      );
    }
  }
}

// Repository publishes events after save
class OrderRepository {
  constructor(
    private db: Database,
    private eventDispatcher: DomainEventDispatcher
  ) {}

  async save(order: Order): Promise<void> {
    // Get events before clearing
    const events = order.pullDomainEvents();

    // Save aggregate
    await this.db.orders.save(order.toSnapshot());

    // Dispatch events (after successful save)
    await this.eventDispatcher.dispatch(events);
  }
}
```

### Event Handlers (Reacting to Events)

```typescript
// ════════════════════════════════════════════════════════════════
// EVENT HANDLERS - React to domain events
// ════════════════════════════════════════════════════════════════

// Same context: Update read model
class OrderPlacedReadModelHandler implements EventHandler<OrderPlaced> {
  constructor(private readDb: ReadDatabase) {}

  async handle(event: OrderPlaced): Promise<void> {
    await this.readDb.orderSummaries.insert({
      orderId: event.aggregateId,
      customerId: event.customerId,
      total: event.total.amount,
      itemCount: event.lineItems.length,
      placedAt: event.occurredOn
    });
  }
}

// Same context: Send notification
class OrderPlacedNotificationHandler implements EventHandler<OrderPlaced> {
  constructor(private notificationService: NotificationService) {}

  async handle(event: OrderPlaced): Promise<void> {
    await this.notificationService.sendOrderConfirmation(
      event.customerId,
      event.aggregateId
    );
  }
}

// CROSS-CONTEXT: Publish to message broker for other contexts
class OrderPlacedIntegrationHandler implements EventHandler<OrderPlaced> {
  constructor(private messageBroker: MessageBroker) {}

  async handle(event: OrderPlaced): Promise<void> {
    // Convert domain event to integration event
    await this.messageBroker.publish('orders', {
      type: 'OrderPlaced',
      orderId: event.aggregateId,
      customerId: event.customerId,
      total: event.total.amount,
      occurredOn: event.occurredOn.toISOString()
    });
  }
}

// SHIPPING CONTEXT: Reacts to OrderPlaced
class ShippingOrderPlacedHandler {
  constructor(private shipmentRepository: ShipmentRepository) {}

  async handle(message: OrderPlacedMessage): Promise<void> {
    // Create shipment in shipping context
    const shipment = Shipment.createFor(
      ShipmentId.generate(),
      new OrderId(message.orderId)
    );
    
    await this.shipmentRepository.save(shipment);
  }
}
```

### Domain Services

A **Domain Service** contains domain logic that doesn't naturally fit in any entity or value object.

```
WHEN TO USE DOMAIN SERVICE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Use Domain Service when:                                       │
│  └── Operation involves multiple aggregates                    │
│  └── Logic doesn't belong to any single entity                 │
│  └── Represents a domain concept that's a verb, not a noun    │
│                                                                  │
│  Examples:                                                      │
│  └── TransferMoney (involves 2 accounts)                       │
│  └── CalculateShipping (involves order + address + carrier)   │
│  └── CheckInventoryAvailability (cross-aggregate check)       │
│                                                                  │
│  Domain Service is:                                             │
│  └── Stateless                                                 │
│  └── Named after domain action (not technical action)         │
│  └── Part of domain layer (not application layer)             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```typescript
// ════════════════════════════════════════════════════════════════
// DOMAIN SERVICE: Logic spanning multiple aggregates
// ════════════════════════════════════════════════════════════════

// Money transfer involves TWO account aggregates
class MoneyTransferService {
  transfer(
    from: BankAccount,
    to: BankAccount,
    amount: Money
  ): TransferResult {
    // Domain logic that doesn't belong to either account
    if (from.currency !== to.currency) {
      throw new Error('Cross-currency transfer not supported');
    }

    if (from.balance.isLessThan(amount)) {
      return TransferResult.insufficientFunds();
    }

    // Both aggregates are modified
    from.withdraw(amount);
    to.deposit(amount);

    return TransferResult.success(from.id, to.id, amount);
  }
}

// Shipping cost calculation
class ShippingCostCalculator {
  constructor(
    private carrierRates: CarrierRateProvider
  ) {}

  calculate(
    order: Order,
    destination: Address,
    carrier: Carrier
  ): Money {
    const weight = this.calculateTotalWeight(order);
    const zone = this.determineShippingZone(destination);
    const baseRate = this.carrierRates.getRate(carrier, zone, weight);
    
    // Domain logic: Express shipping, fragile items, etc.
    let finalCost = baseRate;
    
    if (order.hasFragileItems()) {
      finalCost = finalCost.add(Money.dollars(5));
    }
    
    if (order.requestsExpressShipping()) {
      finalCost = finalCost.multiply(1.5);
    }
    
    return finalCost;
  }
}

// Inventory availability (cross-aggregate check)
class InventoryAvailabilityService {
  constructor(private inventoryRepository: InventoryRepository) {}

  async checkAvailability(
    items: Array<{ productId: ProductId; quantity: number }>
  ): Promise<AvailabilityResult> {
    const unavailable: ProductId[] = [];

    for (const item of items) {
      const inventory = await this.inventoryRepository.findByProductId(item.productId);
      
      if (!inventory || !inventory.hasStock(item.quantity)) {
        unavailable.push(item.productId);
      }
    }

    return unavailable.length === 0
      ? AvailabilityResult.allAvailable()
      : AvailabilityResult.someUnavailable(unavailable);
  }
}
```

### Domain Service vs Application Service

```
CRITICAL DISTINCTION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  DOMAIN SERVICE                    APPLICATION SERVICE          │
│  ═══════════════                   ════════════════════          │
│  └── Domain logic                  └── Orchestration            │
│  └── Business rules                └── Transaction management  │
│  └── Pure domain concepts          └── Security                 │
│  └── No infrastructure             └── Calls repositories       │
│  └── Could be tested with          └── Calls external services │
│      just domain objects               like email, payment     │
│                                                                  │
│  EXAMPLE - Domain Service:                                      │
│  └── MoneyTransferService.transfer(from, to, amount)           │
│  └── Pure business logic: can you transfer?                    │
│                                                                  │
│  EXAMPLE - Application Service:                                 │
│  └── TransferMoneyUseCase.execute(fromId, toId, amount)        │
│  └── Load accounts, call domain service, save, send email     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```typescript
// APPLICATION SERVICE (Orchestration)
class TransferMoneyUseCase {
  constructor(
    private accountRepository: AccountRepository,
    private transferService: MoneyTransferService,  // Domain service
    private notificationService: NotificationService,
    private unitOfWork: UnitOfWork
  ) {}

  async execute(command: TransferMoneyCommand): Promise<TransferResult> {
    return this.unitOfWork.execute(async () => {
      // Load aggregates
      const fromAccount = await this.accountRepository.findById(command.fromAccountId);
      const toAccount = await this.accountRepository.findById(command.toAccountId);

      // Call DOMAIN SERVICE for business logic
      const result = this.transferService.transfer(
        fromAccount,
        toAccount,
        command.amount
      );

      if (result.isSuccess()) {
        // Save aggregates
        await this.accountRepository.save(fromAccount);
        await this.accountRepository.save(toAccount);

        // Infrastructure concern: notification
        await this.notificationService.sendTransferConfirmation(
          fromAccount.ownerId,
          result
        );
      }

      return result;
    });
  }
}
```

### Repositories

A **Repository** mediates between domain and data mapping, providing a collection-like interface for accessing aggregates.

```
REPOSITORY RULES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. ONE REPOSITORY PER AGGREGATE ROOT                          │
│     └── OrderRepository, CustomerRepository                    │
│     └── NOT OrderLineRepository (it's not an aggregate root)  │
│                                                                  │
│  2. INTERFACE IN DOMAIN LAYER                                  │
│     └── Domain defines what it needs                           │
│     └── Implementation in infrastructure layer                 │
│                                                                  │
│  3. COLLECTION-LIKE INTERFACE                                  │
│     └── Think of it as a collection of aggregates             │
│     └── add(), findById(), remove()                            │
│                                                                  │
│  4. RETURN COMPLETE AGGREGATES                                 │
│     └── Always return fully hydrated aggregates               │
│     └── No lazy loading surprises                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```typescript
// ════════════════════════════════════════════════════════════════
// REPOSITORY INTERFACE (Domain Layer)
// ════════════════════════════════════════════════════════════════

// In domain layer - no implementation details
interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>;
  findByCustomerId(customerId: CustomerId): Promise<Order[]>;
  save(order: Order): Promise<void>;
  remove(order: Order): Promise<void>;
  nextId(): OrderId;
}

// ════════════════════════════════════════════════════════════════
// REPOSITORY IMPLEMENTATION (Infrastructure Layer)
// ════════════════════════════════════════════════════════════════

class PostgresOrderRepository implements OrderRepository {
  constructor(
    private db: Database,
    private eventDispatcher: DomainEventDispatcher
  ) {}

  async findById(id: OrderId): Promise<Order | null> {
    const row = await this.db.query(
      'SELECT * FROM orders WHERE id = $1',
      [id.value]
    );

    if (!row) return null;

    const lines = await this.db.query(
      'SELECT * FROM order_lines WHERE order_id = $1',
      [id.value]
    );

    // Reconstitute aggregate from data
    return this.toDomain(row, lines);
  }

  async save(order: Order): Promise<void> {
    const events = order.pullDomainEvents();
    const snapshot = order.toSnapshot();

    await this.db.transaction(async (tx) => {
      // Upsert order
      await tx.query(`
        INSERT INTO orders (id, customer_id, status, created_at)
        VALUES ($1, $2, $3, $4)
        ON CONFLICT (id) DO UPDATE SET status = $3
      `, [snapshot.id, snapshot.customerId, snapshot.status, snapshot.createdAt]);

      // Sync order lines
      await tx.query('DELETE FROM order_lines WHERE order_id = $1', [snapshot.id]);
      
      for (const line of snapshot.lines) {
        await tx.query(`
          INSERT INTO order_lines (id, order_id, product_id, quantity, unit_price)
          VALUES ($1, $2, $3, $4, $5)
        `, [line.id, snapshot.id, line.productId, line.quantity, line.unitPrice]);
      }
    });

    // Dispatch events after successful save
    await this.eventDispatcher.dispatch(events);
  }

  private toDomain(row: OrderRow, lines: OrderLineRow[]): Order {
    return Order.reconstitute(
      new OrderId(row.id),
      new CustomerId(row.customer_id),
      row.status as OrderStatus,
      new Date(row.created_at),
      lines.map(l => OrderLine.reconstitute(
        new OrderLineId(l.id),
        new ProductId(l.product_id),
        l.quantity,
        Money.fromCents(l.unit_price)
      ))
    );
  }

  nextId(): OrderId {
    return OrderId.generate();
  }
}

// ════════════════════════════════════════════════════════════════
// SPECIFICATION PATTERN (Complex queries)
// ════════════════════════════════════════════════════════════════

interface Specification<T> {
  isSatisfiedBy(entity: T): boolean;
  toSql(): string;
}

class HighValueOrderSpecification implements Specification<Order> {
  constructor(private threshold: Money) {}

  isSatisfiedBy(order: Order): boolean {
    return order.total().isGreaterThan(this.threshold);
  }

  toSql(): string {
    return `total > ${this.threshold.cents}`;
  }
}

class OrdersByStatusSpecification implements Specification<Order> {
  constructor(private status: OrderStatus) {}

  isSatisfiedBy(order: Order): boolean {
    return order.status === this.status;
  }

  toSql(): string {
    return `status = '${this.status}'`;
  }
}

// Composite specifications
class AndSpecification<T> implements Specification<T> {
  constructor(
    private left: Specification<T>,
    private right: Specification<T>
  ) {}

  isSatisfiedBy(entity: T): boolean {
    return this.left.isSatisfiedBy(entity) && this.right.isSatisfiedBy(entity);
  }

  toSql(): string {
    return `(${this.left.toSql()}) AND (${this.right.toSql()})`;
  }
}

// Usage
const highValuePlacedOrders = new AndSpecification(
  new HighValueOrderSpecification(Money.dollars(1000)),
  new OrdersByStatusSpecification(OrderStatus.Placed)
);

const orders = await orderRepository.findBySpecification(highValuePlacedOrders);
```

### Factories

A **Factory** encapsulates complex object creation logic.

```typescript
// ════════════════════════════════════════════════════════════════
// FACTORY: Complex creation logic
// ════════════════════════════════════════════════════════════════

class OrderFactory {
  constructor(
    private inventoryService: InventoryAvailabilityService,
    private pricingService: PricingService
  ) {}

  async createOrder(
    customerId: CustomerId,
    items: Array<{ productId: ProductId; quantity: number }>
  ): Promise<Order> {
    // Complex validation: check inventory
    const availability = await this.inventoryService.checkAvailability(items);
    if (!availability.allAvailable) {
      throw new Error(`Products unavailable: ${availability.unavailable.join(', ')}`);
    }

    // Get current prices
    const order = Order.create(customerId);
    
    for (const item of items) {
      const price = await this.pricingService.getCurrentPrice(
        item.productId,
        customerId
      );
      
      order.addLine(item.productId, item.quantity, price);
    }

    return order;
  }
}

// Reconstitution factory (from persistence)
class OrderReconstitutionFactory {
  reconstitute(data: OrderSnapshot): Order {
    return Order.reconstitute(
      new OrderId(data.id),
      new CustomerId(data.customerId),
      data.status,
      data.createdAt,
      data.lines.map(l => OrderLine.reconstitute(
        new OrderLineId(l.id),
        new ProductId(l.productId),
        l.quantity,
        Money.fromCents(l.unitPriceCents)
      ))
    );
  }
}
```

---

## 5. Layers & Architecture

### DDD Layer Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      DDD LAYERED ARCHITECTURE                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              PRESENTATION / UI LAYER                      │   │
│  │  └── REST Controllers                                    │   │
│  │  └── GraphQL Resolvers                                   │   │
│  │  └── CLI Commands                                        │   │
│  │  └── Converts HTTP ↔ Application DTOs                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              APPLICATION LAYER                            │   │
│  │  └── Use Cases / Application Services                   │   │
│  │  └── Command/Query Handlers                             │   │
│  │  └── Orchestrates domain objects                        │   │
│  │  └── Transaction management                              │   │
│  │  └── NO business logic here!                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              DOMAIN LAYER (The Heart!)                    │   │
│  │  └── Entities                                            │   │
│  │  └── Value Objects                                       │   │
│  │  └── Aggregates                                          │   │
│  │  └── Domain Events                                       │   │
│  │  └── Domain Services                                     │   │
│  │  └── Repository Interfaces (not implementations!)       │   │
│  │  └── ALL business logic lives here                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              INFRASTRUCTURE LAYER                         │   │
│  │  └── Repository Implementations                          │   │
│  │  └── Database access (Prisma, TypeORM)                  │   │
│  │  └── External service integrations                       │   │
│  │  └── Message broker implementations                      │   │
│  │  └── Email, Payment gateways                            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  DEPENDENCY RULE:                                               │
│  └── Outer layers depend on inner layers                       │
│  └── Domain layer has NO dependencies on other layers         │
│  └── Infrastructure implements domain interfaces              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Folder Structure

```
src/
├── presentation/          # UI/API Layer
│   ├── rest/
│   │   ├── controllers/
│   │   │   ├── order.controller.ts
│   │   │   └── customer.controller.ts
│   │   └── dto/
│   │       ├── create-order.dto.ts
│   │       └── order-response.dto.ts
│   └── graphql/
│       └── resolvers/
│
├── application/           # Application Layer
│   ├── commands/
│   │   ├── place-order.command.ts
│   │   └── place-order.handler.ts
│   ├── queries/
│   │   ├── get-order.query.ts
│   │   └── get-order.handler.ts
│   └── services/
│       └── order.application-service.ts
│
├── domain/                # Domain Layer (The Core!)
│   ├── order/
│   │   ├── order.aggregate.ts
│   │   ├── order-line.entity.ts
│   │   ├── order-id.value-object.ts
│   │   ├── order-status.enum.ts
│   │   ├── order.repository.ts    # Interface only!
│   │   └── events/
│   │       ├── order-placed.event.ts
│   │       └── order-cancelled.event.ts
│   ├── customer/
│   │   ├── customer.aggregate.ts
│   │   └── ...
│   ├── shared/
│   │   ├── money.value-object.ts
│   │   ├── address.value-object.ts
│   │   └── domain-event.ts
│   └── services/
│       └── shipping-cost.domain-service.ts
│
└── infrastructure/        # Infrastructure Layer
    ├── persistence/
    │   ├── repositories/
    │   │   ├── postgres-order.repository.ts
    │   │   └── postgres-customer.repository.ts
    │   ├── entities/         # ORM entities (not domain entities!)
    │   │   ├── order.orm-entity.ts
    │   │   └── order-line.orm-entity.ts
    │   └── mappers/
    │       └── order.mapper.ts
    ├── messaging/
    │   └── rabbitmq-event-publisher.ts
    └── external/
        ├── stripe-payment.service.ts
        └── sendgrid-email.service.ts
```

### Complete Example: Place Order Use Case

```typescript
// ════════════════════════════════════════════════════════════════
// PRESENTATION LAYER: REST Controller
// ════════════════════════════════════════════════════════════════

// src/presentation/rest/controllers/order.controller.ts
@Controller('/orders')
class OrderController {
  constructor(private placeOrderHandler: PlaceOrderHandler) {}

  @Post('/')
  async placeOrder(@Body() dto: PlaceOrderDto): Promise<OrderResponseDto> {
    // Convert DTO to Command
    const command = new PlaceOrderCommand(
      dto.customerId,
      dto.items.map(i => ({
        productId: i.productId,
        quantity: i.quantity
      }))
    );

    // Delegate to application layer
    const result = await this.placeOrderHandler.execute(command);

    // Convert to response DTO
    return OrderResponseDto.fromDomain(result);
  }
}

// ════════════════════════════════════════════════════════════════
// APPLICATION LAYER: Command Handler
// ════════════════════════════════════════════════════════════════

// src/application/commands/place-order.handler.ts
class PlaceOrderHandler {
  constructor(
    private orderRepository: OrderRepository,
    private customerRepository: CustomerRepository,
    private inventoryService: InventoryAvailabilityService,
    private pricingService: PricingService,
    private unitOfWork: UnitOfWork
  ) {}

  async execute(command: PlaceOrderCommand): Promise<Order> {
    return this.unitOfWork.execute(async () => {
      // Load customer
      const customer = await this.customerRepository.findById(
        new CustomerId(command.customerId)
      );
      if (!customer) {
        throw new CustomerNotFoundError(command.customerId);
      }

      // Check inventory (domain service)
      const availability = await this.inventoryService.checkAvailability(
        command.items.map(i => ({
          productId: new ProductId(i.productId),
          quantity: i.quantity
        }))
      );
      if (!availability.allAvailable) {
        throw new ProductsUnavailableError(availability.unavailable);
      }

      // Create order aggregate
      const order = Order.create(customer.id);

      // Add lines with current prices
      for (const item of command.items) {
        const productId = new ProductId(item.productId);
        const price = await this.pricingService.getCurrentPrice(
          productId,
          customer.id
        );
        order.addLine(productId, item.quantity, price);
      }

      // Place the order (domain logic)
      order.place();

      // Persist
      await this.orderRepository.save(order);

      return order;
    });
  }
}

// ════════════════════════════════════════════════════════════════
// DOMAIN LAYER: Aggregate (already shown above)
// ════════════════════════════════════════════════════════════════

// src/domain/order/order.aggregate.ts
// (See Order aggregate implementation above)

// ════════════════════════════════════════════════════════════════
// INFRASTRUCTURE LAYER: Repository Implementation
// ════════════════════════════════════════════════════════════════

// src/infrastructure/persistence/repositories/postgres-order.repository.ts
class PostgresOrderRepository implements OrderRepository {
  constructor(
    private prisma: PrismaClient,
    private mapper: OrderMapper,
    private eventPublisher: EventPublisher
  ) {}

  async findById(id: OrderId): Promise<Order | null> {
    const data = await this.prisma.order.findUnique({
      where: { id: id.value },
      include: { lines: true }
    });

    if (!data) return null;

    return this.mapper.toDomain(data);
  }

  async save(order: Order): Promise<void> {
    const events = order.pullDomainEvents();
    const data = this.mapper.toPersistence(order);

    await this.prisma.$transaction(async (tx) => {
      await tx.order.upsert({
        where: { id: data.id },
        create: data,
        update: data
      });

      // Handle order lines
      await tx.orderLine.deleteMany({
        where: { orderId: data.id }
      });

      await tx.orderLine.createMany({
        data: data.lines
      });
    });

    // Publish events after successful transaction
    for (const event of events) {
      await this.eventPublisher.publish(event);
    }
  }
}
```

### Real-World Example: E-Commerce Domain

```typescript
// ════════════════════════════════════════════════════════════════
// COMPLETE E-COMMERCE BOUNDED CONTEXTS
// ════════════════════════════════════════════════════════════════

// SALES CONTEXT
namespace Sales {
  // Aggregates
  class Customer extends AggregateRoot { /* ... */ }
  class Order extends AggregateRoot { /* ... */ }
  class ShoppingCart extends AggregateRoot { /* ... */ }

  // Value Objects
  class Money { /* ... */ }
  class Address { /* ... */ }
  class Discount { /* ... */ }

  // Domain Events
  class OrderPlaced implements DomainEvent { /* ... */ }
  class CartAbandoned implements DomainEvent { /* ... */ }

  // Domain Services
  class PricingService { /* ... */ }
  class DiscountCalculator { /* ... */ }
}

// INVENTORY CONTEXT
namespace Inventory {
  class Product extends AggregateRoot {
    private sku: SKU;
    private quantityOnHand: number;
    private reorderPoint: number;
    private reservations: Reservation[] = [];

    reserve(orderId: OrderId, quantity: number): Reservation {
      if (this.availableQuantity() < quantity) {
        throw new InsufficientStockError(this.sku, quantity);
      }

      const reservation = new Reservation(
        ReservationId.generate(),
        orderId,
        quantity,
        new Date()
      );

      this.reservations.push(reservation);
      this.addDomainEvent(new StockReserved(this.sku, orderId, quantity));
      
      return reservation;
    }

    releaseReservation(orderId: OrderId): void {
      const index = this.reservations.findIndex(r => r.orderId.equals(orderId));
      if (index === -1) return;

      this.reservations.splice(index, 1);
      this.addDomainEvent(new ReservationReleased(this.sku, orderId));
    }

    private availableQuantity(): number {
      const reserved = this.reservations.reduce((sum, r) => sum + r.quantity, 0);
      return this.quantityOnHand - reserved;
    }
  }

  // React to Sales events
  class OrderPlacedHandler {
    async handle(event: Sales.OrderPlaced): Promise<void> {
      for (const line of event.lineItems) {
        const product = await this.productRepo.findBySku(line.sku);
        product.reserve(event.orderId, line.quantity);
        await this.productRepo.save(product);
      }
    }
  }
}

// SHIPPING CONTEXT
namespace Shipping {
  class Shipment extends AggregateRoot {
    private id: ShipmentId;
    private orderId: OrderId;  // Reference to Sales context
    private destination: Address;
    private carrier: Carrier;
    private packages: Package[] = [];
    private status: ShipmentStatus;
    private trackingNumber?: string;

    static createFor(orderId: OrderId, destination: Address): Shipment {
      const shipment = new Shipment();
      shipment.id = ShipmentId.generate();
      shipment.orderId = orderId;
      shipment.destination = destination;
      shipment.status = ShipmentStatus.Pending;
      
      shipment.addDomainEvent(new ShipmentCreated(shipment.id, orderId));
      return shipment;
    }

    assignCarrier(carrier: Carrier, trackingNumber: string): void {
      this.carrier = carrier;
      this.trackingNumber = trackingNumber;
      this.addDomainEvent(new CarrierAssigned(this.id, carrier, trackingNumber));
    }

    dispatch(): void {
      if (this.packages.length === 0) {
        throw new Error('Cannot dispatch shipment without packages');
      }

      this.status = ShipmentStatus.InTransit;
      this.addDomainEvent(new ShipmentDispatched(this.id, this.trackingNumber));
    }

    markDelivered(signature: string, deliveredAt: Date): void {
      this.status = ShipmentStatus.Delivered;
      this.addDomainEvent(new ShipmentDelivered(
        this.id,
        this.orderId,
        signature,
        deliveredAt
      ));
    }
  }
}

// BILLING CONTEXT
namespace Billing {
  class Invoice extends AggregateRoot {
    private id: InvoiceId;
    private orderId: OrderId;
    private customerId: CustomerId;
    private lineItems: InvoiceLineItem[] = [];
    private status: InvoiceStatus;
    private dueDate: Date;
    private payments: Payment[] = [];

    get totalDue(): Money {
      const invoiced = this.lineItems.reduce(
        (sum, line) => sum.add(line.amount),
        Money.zero()
      );
      const paid = this.payments.reduce(
        (sum, payment) => sum.add(payment.amount),
        Money.zero()
      );
      return invoiced.subtract(paid);
    }

    recordPayment(payment: Payment): void {
      this.payments.push(payment);

      if (this.totalDue.isZero()) {
        this.status = InvoiceStatus.Paid;
        this.addDomainEvent(new InvoicePaid(this.id, this.orderId));
      } else {
        this.addDomainEvent(new PartialPaymentReceived(
          this.id,
          payment.amount,
          this.totalDue
        ));
      }
    }
  }
}
```

### Integration Between Contexts

```typescript
// ════════════════════════════════════════════════════════════════
// INTEGRATION: How contexts communicate
// ════════════════════════════════════════════════════════════════

// 1. INTEGRATION EVENTS (Published to message broker)
class OrderPlacedIntegrationEvent {
  constructor(
    public readonly orderId: string,
    public readonly customerId: string,
    public readonly shippingAddress: {
      street: string;
      city: string;
      zipCode: string;
      country: string;
    },
    public readonly lineItems: Array<{
      sku: string;
      quantity: number;
      unitPrice: number;
    }>,
    public readonly totalAmount: number,
    public readonly occurredOn: string
  ) {}
}

// 2. SALES CONTEXT: Publishes when order is placed
class SalesOrderPlacedHandler {
  constructor(private messageBroker: MessageBroker) {}

  async handle(domainEvent: Sales.OrderPlaced): Promise<void> {
    // Transform domain event to integration event
    const integrationEvent = new OrderPlacedIntegrationEvent(
      domainEvent.aggregateId,
      domainEvent.customerId,
      domainEvent.shippingAddress,
      domainEvent.lineItems,
      domainEvent.total.amount,
      domainEvent.occurredOn.toISOString()
    );

    await this.messageBroker.publish('orders.placed', integrationEvent);
  }
}

// 3. SHIPPING CONTEXT: Subscribes and reacts
class ShippingOrderPlacedSubscriber {
  constructor(private shipmentFactory: ShipmentFactory) {}

  async handle(event: OrderPlacedIntegrationEvent): Promise<void> {
    // Create shipment in shipping context
    const shipment = await this.shipmentFactory.createFromOrder(
      new OrderId(event.orderId),
      Address.create(
        event.shippingAddress.street,
        event.shippingAddress.city,
        event.shippingAddress.zipCode,
        event.shippingAddress.country
      )
    );

    await this.shipmentRepository.save(shipment);
  }
}

// 4. INVENTORY CONTEXT: Subscribes and reacts
class InventoryOrderPlacedSubscriber {
  async handle(event: OrderPlacedIntegrationEvent): Promise<void> {
    for (const item of event.lineItems) {
      const product = await this.productRepo.findBySku(new SKU(item.sku));
      product.reserve(new OrderId(event.orderId), item.quantity);
      await this.productRepo.save(product);
    }
  }
}

// 5. BILLING CONTEXT: Subscribes and reacts
class BillingOrderPlacedSubscriber {
  async handle(event: OrderPlacedIntegrationEvent): Promise<void> {
    const invoice = Invoice.createForOrder(
      new OrderId(event.orderId),
      new CustomerId(event.customerId),
      event.lineItems.map(item => new InvoiceLineItem(
        item.sku,
        item.quantity,
        Money.fromCents(item.unitPrice)
      ))
    );

    await this.invoiceRepository.save(invoice);
  }
}
```

---

## 7. Common Mistakes

### Mistake 1: Anemic Domain Model

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ ANEMIC DOMAIN MODEL (Anti-pattern!)
// ════════════════════════════════════════════════════════════════

// Entity is just a data bag - no behavior
class Order {
  id: string;
  customerId: string;
  status: string;
  lines: OrderLine[];
  total: number;
}

// All logic in service - domain is anemic
class OrderService {
  placeOrder(order: Order): void {
    if (order.lines.length === 0) {
      throw new Error('Empty order');
    }
    order.status = 'placed';
    order.total = this.calculateTotal(order.lines);
  }

  cancelOrder(order: Order): void {
    if (order.status === 'shipped') {
      throw new Error('Cannot cancel');
    }
    order.status = 'cancelled';
  }

  calculateTotal(lines: OrderLine[]): number {
    return lines.reduce((sum, l) => sum + l.price * l.quantity, 0);
  }
}

// ════════════════════════════════════════════════════════════════
// ✅ RICH DOMAIN MODEL (Correct!)
// ════════════════════════════════════════════════════════════════

class Order extends AggregateRoot {
  private _status: OrderStatus;
  private _lines: OrderLine[] = [];

  // Behavior lives IN the entity
  place(): void {
    if (this._lines.length === 0) {
      throw new DomainError('Cannot place empty order');
    }
    this._status = OrderStatus.Placed;
    this.addDomainEvent(new OrderPlaced(this.id, this.total()));
  }

  cancel(reason: CancellationReason): void {
    if (this._status === OrderStatus.Shipped) {
      throw new DomainError('Cannot cancel shipped order');
    }
    this._status = OrderStatus.Cancelled;
    this.addDomainEvent(new OrderCancelled(this.id, reason));
  }

  total(): Money {
    return this._lines.reduce(
      (sum, line) => sum.add(line.subtotal()),
      Money.zero()
    );
  }
}
```

### Mistake 2: Too Large Aggregates

```
❌ WRONG: Everything in one aggregate
┌─────────────────────────────────────────────────────────────────┐
│                     Customer Aggregate                           │
│  ├── Orders[] (thousands!)                                      │
│  │   └── OrderLines[]                                           │
│  ├── Addresses[]                                                │
│  ├── PaymentMethods[]                                           │
│  ├── Reviews[]                                                  │
│  └── Wishlist                                                   │
│                                                                  │
│  PROBLEMS:                                                      │
│  └── Loading customer loads ALL orders (megabytes)             │
│  └── Modifying one order locks entire customer                 │
│  └── Concurrency nightmare                                     │
└─────────────────────────────────────────────────────────────────┘

✅ RIGHT: Small, focused aggregates
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Customer   │  │    Order     │  │   Review     │
│  Aggregate   │  │  Aggregate   │  │  Aggregate   │
├──────────────┤  ├──────────────┤  ├──────────────┤
│ - Name       │  │ - customerId │  │ - customerId │
│ - Email      │  │ - Lines[]    │  │ - productId  │
│ - Addresses[]│  │ - Status     │  │ - Rating     │
└──────────────┘  └──────────────┘  └──────────────┘
       │                 │                  │
       └────── Reference by ID only ───────┘
```

### Mistake 3: Transactions Across Aggregates

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ WRONG: One transaction modifies multiple aggregates
// ════════════════════════════════════════════════════════════════

class PlaceOrderService {
  async execute(command: PlaceOrderCommand): Promise<void> {
    await this.unitOfWork.transaction(async () => {
      // Loading and modifying MULTIPLE aggregates in one transaction
      const customer = await this.customerRepo.findById(command.customerId);
      const inventory = await this.inventoryRepo.findById(command.productId);
      const order = Order.create(customer.id);

      // ❌ All in same transaction - tight coupling!
      customer.addOrder(order);
      inventory.reserve(order.id, command.quantity);
      await this.customerRepo.save(customer);
      await this.inventoryRepo.save(inventory);
      await this.orderRepo.save(order);
    });
  }
}

// ════════════════════════════════════════════════════════════════
// ✅ RIGHT: One transaction per aggregate, eventual consistency
// ════════════════════════════════════════════════════════════════

class PlaceOrderService {
  async execute(command: PlaceOrderCommand): Promise<void> {
    // Transaction 1: Create and place order
    const order = Order.create(new CustomerId(command.customerId));
    order.addLine(new ProductId(command.productId), command.quantity, command.price);
    order.place();
    await this.orderRepo.save(order);
    // OrderPlaced event is published

    // Other aggregates react to event (separate transactions)
  }
}

// Inventory reacts to OrderPlaced event
class InventoryOrderPlacedHandler {
  async handle(event: OrderPlaced): Promise<void> {
    // Transaction 2: Reserve inventory
    const product = await this.productRepo.findById(event.productId);
    product.reserve(event.orderId, event.quantity);
    await this.productRepo.save(product);
  }
}
```

### Mistake 4: Exposing Domain Internals

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ WRONG: Exposing mutable collections
// ════════════════════════════════════════════════════════════════

class Order {
  lines: OrderLine[] = [];  // ❌ Public mutable collection

  get total(): number {
    return this.lines.reduce((sum, l) => sum + l.subtotal, 0);
  }
}

// External code can break invariants
order.lines.push(new OrderLine(...));  // ❌ Bypasses validation
order.lines = [];  // ❌ Can empty the order

// ════════════════════════════════════════════════════════════════
// ✅ RIGHT: Encapsulate and protect
// ════════════════════════════════════════════════════════════════

class Order {
  private _lines: OrderLine[] = [];

  // Return copy or readonly
  get lines(): ReadonlyArray<OrderLine> {
    return [...this._lines];
  }

  // Only way to modify is through methods that enforce invariants
  addLine(productId: ProductId, quantity: number, price: Money): void {
    this.ensureCanModify();
    if (quantity <= 0) {
      throw new DomainError('Quantity must be positive');
    }
    this._lines.push(new OrderLine(productId, quantity, price));
  }
}
```

### Mistake 5: Missing Ubiquitous Language

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ WRONG: Developer terminology, not business language
// ════════════════════════════════════════════════════════════════

class User {
  setStatus(status: number): void { /* ... */ }
  updateAddress(addr: AddressDTO): void { /* ... */ }
}

userService.setUserInactive(userId);
userService.processUserUpdate(userId, data);

// ════════════════════════════════════════════════════════════════
// ✅ RIGHT: Business language (what domain experts say)
// ════════════════════════════════════════════════════════════════

class Customer {
  markAsChurned(reason: ChurnReason): void { /* ... */ }
  relocate(newAddress: Address): void { /* ... */ }
  suspend(until: Date): void { /* ... */ }
  reinstate(): void { /* ... */ }
}

// Code reads like business language
customer.markAsChurned(ChurnReason.CompetitorSwitch);
customer.relocate(newAddress);
```

### Mistake 6: Repository Returns Primitives/DTOs

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ WRONG: Repository returns raw data
// ════════════════════════════════════════════════════════════════

interface OrderRepository {
  findById(id: string): Promise<OrderDTO | null>;  // ❌ Returns DTO
  getOrderTotal(id: string): Promise<number>;       // ❌ Returns primitive
}

// ════════════════════════════════════════════════════════════════
// ✅ RIGHT: Repository returns complete aggregates
// ════════════════════════════════════════════════════════════════

interface OrderRepository {
  findById(id: OrderId): Promise<Order | null>;  // ✅ Returns aggregate
  save(order: Order): Promise<void>;
}

// If you need read-optimized queries, use separate Query Service (CQRS)
interface OrderQueryService {
  getOrderSummary(id: string): Promise<OrderSummaryDTO>;
  getOrdersForDashboard(): Promise<DashboardDTO>;
}
```

---

## 8. When to Use / Not Use DDD

### When TO Use DDD

```
✅ USE DDD WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. COMPLEX BUSINESS LOGIC                                      │
│     └── Many business rules that change over time              │
│     └── Logic that's hard to understand                        │
│     └── Domain experts needed to explain requirements          │
│                                                                  │
│  2. LONG-LIVED PROJECT                                          │
│     └── Will be maintained for years                           │
│     └── Requirements will evolve                               │
│     └── Multiple teams will work on it                         │
│                                                                  │
│  3. CORE COMPETITIVE ADVANTAGE                                  │
│     └── The domain IS your business                            │
│     └── Getting it right matters enormously                    │
│                                                                  │
│  4. DOMAIN EXPERTS AVAILABLE                                    │
│     └── You have access to people who know the business       │
│     └── They can help define ubiquitous language              │
│                                                                  │
│  EXAMPLES:                                                      │
│  └── Banking / Financial systems                               │
│  └── Insurance policy management                               │
│  └── Healthcare / Medical records                              │
│  └── E-commerce with complex pricing/inventory                │
│  └── Logistics / Supply chain                                  │
│  └── Trading platforms                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When NOT to Use DDD

```
❌ DON'T USE DDD WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. SIMPLE CRUD APPLICATION                                     │
│     └── Mostly data in, data out                               │
│     └── Little business logic                                  │
│     └── Admin panels, content management                       │
│                                                                  │
│  2. SMALL TEAM / SHORT PROJECT                                  │
│     └── 1-3 developers                                         │
│     └── MVP or prototype                                       │
│     └── Throwaway code                                         │
│                                                                  │
│  3. WELL-UNDERSTOOD DOMAIN                                      │
│     └── Standard problem with standard solution                │
│     └── No competitive advantage in the domain                 │
│     └── Generic subdomain (auth, email, payments)             │
│                                                                  │
│  4. NO DOMAIN EXPERTS                                           │
│     └── Can't get access to business people                   │
│     └── Requirements unclear                                   │
│                                                                  │
│  5. DATA-CENTRIC APPLICATION                                    │
│     └── Reporting / Analytics                                  │
│     └── ETL pipelines                                          │
│     └── Data warehousing                                       │
│                                                                  │
│  IN THESE CASES:                                               │
│  └── Use simple layered architecture                          │
│  └── Active Record pattern                                     │
│  └── Transaction Script                                        │
│  └── Just use your ORM directly                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### DDD Complexity vs Value

```
                    HIGH
                      │
                      │    ┌─────────────────────────┐
        Value of DDD  │    │  COMPLEX DOMAIN         │
                      │    │  Use full DDD           │
                      │    │  (Banking, Insurance)   │
                      │    └─────────────────────────┘
                      │
                      │    ┌─────────────────────────┐
                      │    │  MEDIUM COMPLEXITY      │
                      │    │  Use DDD-Lite           │
                      │    │  (E-commerce, SaaS)     │
                      │    └─────────────────────────┘
                      │
                      │    ┌─────────────────────────┐
                      │    │  SIMPLE CRUD            │
                      │    │  Skip DDD               │
                      │    │  (Blogs, Admin panels)  │
                      │    └─────────────────────────┘
                    LOW
                      └───────────────────────────────►
                        LOW                        HIGH
                              Domain Complexity
```

---

## 9. Interview Questions & Answers

### Basic Questions

**Q1: What is Domain-Driven Design?**
> **A:** DDD is a software design approach that focuses on modeling the core business domain with a shared language between developers and domain experts. It has two parts:
> - **Strategic Design**: Ubiquitous language, bounded contexts, context mapping
> - **Tactical Design**: Entities, value objects, aggregates, domain events, repositories
>
> The key insight is that software should mirror how the business thinks, not how the database is structured.

**Q2: What is ubiquitous language?**
> **A:** It's a shared vocabulary used by both developers and domain experts. When a developer says "Order" and a business person says "Order," they mean exactly the same thing. The code uses business terminology - we have `customer.markAsChurned()` instead of `user.setStatus('inactive')`. This eliminates translation errors and makes the code a direct expression of business knowledge.

**Q3: What is a bounded context?**
> **A:** A boundary within which a domain model and its language apply. The same word can mean different things in different contexts - "Customer" in Sales (buyer info, credit limit) is different from "Customer" in Shipping (delivery address, contact phone). Each bounded context has its own model, and they communicate through well-defined interfaces.

**Q4: What's the difference between an Entity and a Value Object?**
> **A:** 
> - **Entity**: Has identity. Two users with the same name are different people (different IDs). Equality is based on identity. Can change over time while remaining the same entity.
> - **Value Object**: Defined by attributes, no identity. $10 equals any other $10. Immutable - you create a new one instead of modifying. Same attributes = equal objects.
>
> Rule of thumb: 80% should be value objects.

### Intermediate Questions

**Q5: What is an Aggregate? Why do we need aggregate boundaries?**
> **A:** An aggregate is a cluster of domain objects treated as a single unit for data changes. It has one root (aggregate root) that's the only entry point.
>
> We need boundaries for:
> - **Consistency**: Changes within an aggregate are one transaction
> - **Concurrency**: Smaller aggregates = less locking
> - **Encapsulation**: Invariants are protected by the root
>
> Example: Order (root) + OrderLines (children). You can't modify OrderLine directly - you go through Order, which enforces rules like "order must have at least one line."

**Q6: How do aggregates communicate with each other?**
> **A:** Through domain events and ID references, never direct object references.
>
> - Store other aggregate's ID, not the object itself
> - Publish domain events when something important happens
> - Other aggregates subscribe and react
> - This gives eventual consistency, not immediate consistency
>
> Example: Order publishes `OrderPlaced`, Shipping context reacts by creating a Shipment.

**Q7: What's the difference between Domain Service and Application Service?**
> **A:**
> - **Domain Service**: Contains domain logic that doesn't fit in any entity. Stateless. No infrastructure. Example: `MoneyTransferService.transfer(from, to, amount)` - logic involving multiple aggregates.
> - **Application Service**: Orchestration layer. Loads aggregates, calls domain logic, saves, sends emails. Example: `TransferMoneyUseCase.execute(command)` - coordinates the whole flow, manages transactions.

**Q8: What is an Anti-Corruption Layer?**
> **A:** A translation layer that protects your domain model from external systems. When integrating with a legacy system that has a messy model, the ACL translates between their model and your clean domain model. This prevents legacy "pollution" from spreading into your code. It's like an adapter with translation logic.

### Advanced Questions

**Q9: How do you identify bounded contexts?**
> **A:** Through **Event Storming** workshops with domain experts:
> 1. Map out domain events on sticky notes ("OrderPlaced", "PaymentReceived")
> 2. Look for where language changes - same word, different meaning
> 3. Identify different experts for different areas
> 4. Look for different business rules per area
>
> Signals for separate contexts: different vocabulary, different experts, different lifecycles, different deployment needs.

**Q10: How do you handle transactions across aggregates?**
> **A:** You generally don't - that's the point of aggregate boundaries.
>
> Instead:
> - One aggregate per transaction
> - Use eventual consistency via domain events
> - For complex multi-step processes, use the **Saga pattern**
>
> If you find yourself needing transactions across aggregates, your boundaries might be wrong - consider if they should be one aggregate.

**Q11: What is the Saga pattern?**
> **A:** A way to handle distributed transactions across multiple aggregates/services using compensating actions.
>
> Example: Place Order saga:
> 1. Create Order → success
> 2. Reserve Inventory → success
> 3. Charge Payment → **fails**
> 4. Compensate: Release Inventory, Cancel Order
>
> Two styles: **Choreography** (events, no central coordinator) or **Orchestration** (saga manager coordinates).

**Q12: How do you deal with large aggregates?**
> **A:** Split them! Large aggregates cause:
> - Performance issues (loading too much data)
> - Concurrency problems (locking)
>
> Solutions:
> - Prefer eventual consistency between aggregates
> - Reference by ID, not by object
> - Question whether children really need to be in the same aggregate
> - Ask: "Does this child NEED immediate consistency with the root?"

**Q13: When would you NOT use DDD?**
> **A:** 
> - Simple CRUD apps with little business logic
> - Small teams / short projects / MVPs
> - Well-understood generic domains (auth, email)
> - Data-centric apps (reporting, ETL)
> - When you don't have access to domain experts
>
> DDD has overhead. Only worth it when domain complexity justifies it.

### Scenario Questions

**Q14: Design an order system with DDD. What aggregates would you have?**
> **A:** I would identify these aggregates:
>
> **Order Aggregate** (root: Order)
> - OrderId, CustomerId (reference), OrderLines[], Status
> - Behaviors: addLine(), removeLine(), place(), cancel()
> - Invariants: Must have lines to place, can't modify after placed
>
> **Customer Aggregate** (separate - not inside Order!)
> - CustomerId, Name, Email, Addresses[]
> - Why separate: Customer exists independently, orders just reference
>
> **Product Aggregate** (in Catalog context)
> - ProductId, Name, Price, Description
>
> **Inventory** (in Inventory context)
> - Different bounded context, reacts to OrderPlaced events
>
> Communication: Order publishes events, other contexts react.

**Q15: The business says "Customer" but Sales and Support mean different things. How do you handle this?**
> **A:** This is a classic bounded context situation.
>
> Create two bounded contexts:
> - **Sales Context**: Customer has creditLimit, purchaseHistory, loyaltyTier
> - **Support Context**: Customer has tickets[], supportLevel, preferredAgent
>
> They share customerId but have different models. If they need each other's data, use:
> - Integration events to sync what's needed
> - API calls through ACL if needed
>
> This is why DDD exists - to handle exactly this scenario where the same term means different things.

---

## 🎓 Key Takeaways

1. **Ubiquitous Language** is the most important concept - code should speak business language
2. **Bounded Contexts** define where models apply - same word can mean different things
3. **Aggregates** are transaction boundaries - keep them small
4. **Entities** have identity, **Value Objects** don't - prefer value objects
5. **Domain Events** enable loose coupling between aggregates
6. **Repositories** return aggregates, not DTOs
7. **Avoid anemic domain models** - behavior belongs in entities
8. **Don't use DDD for simple CRUD** - it's for complex domains only
9. **One transaction = one aggregate** - use eventual consistency across aggregates
10. **Reference by ID** between aggregates, never by object

---

## 📚 Resources

### Books
- **"Domain-Driven Design" by Eric Evans** (The Blue Book) - The original
- **"Implementing Domain-Driven Design" by Vaughn Vernon** (The Red Book) - Practical
- **"Domain-Driven Design Distilled" by Vaughn Vernon** - Quick overview

### Online
- [DDD Community](https://dddcommunity.org/)
- [Event Storming](https://www.eventstorming.com/)
- [Martin Fowler's DDD Articles](https://martinfowler.com/tags/domain%20driven%20design.html)
```


