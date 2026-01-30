# 🏛️ Clean Architecture Complete Guide

> A comprehensive guide to building maintainable, testable, and scalable software through proper layering, dependency inversion, and SOLID principles.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Clean Architecture is about organizing code so business logic is independent of frameworks, databases, and UI - dependencies point inward, and the domain is at the center."

### The Dependency Rule (MOST IMPORTANT!)
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│    THE DEPENDENCY RULE:                                         │
│    ════════════════════                                         │
│                                                                  │
│    Source code dependencies can only point INWARD.              │
│                                                                  │
│    Nothing in an inner circle can know anything                 │
│    about something in an outer circle.                          │
│                                                                  │
│    Inner layers define INTERFACES.                              │
│    Outer layers IMPLEMENT them.                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The 4 Layers (Memorize!)
```
OUTER (Details) ──────────────────────────────────────► INNER (Policy)

┌────────────────────────────────────────────────────────────────────┐
│ 4. FRAMEWORKS & DRIVERS (Outermost)                                 │
│    └── Web frameworks, databases, external APIs, UI                │
├────────────────────────────────────────────────────────────────────┤
│ 3. INTERFACE ADAPTERS                                              │
│    └── Controllers, gateways, presenters, repositories (impl)     │
├────────────────────────────────────────────────────────────────────┤
│ 2. APPLICATION (Use Cases)                                         │
│    └── Business rules specific to the application                  │
├────────────────────────────────────────────────────────────────────┤
│ 1. ENTITIES (Innermost - The Core!)                                │
│    └── Enterprise business rules, domain objects                   │
└────────────────────────────────────────────────────────────────────┘
```

### SOLID Principles (Know These Cold!)
| Principle | Meaning | 1-Liner |
|-----------|---------|---------|
| **S** - Single Responsibility | One reason to change | "A class should have only one job" |
| **O** - Open/Closed | Open for extension, closed for modification | "Add new behavior without changing existing code" |
| **L** - Liskov Substitution | Subtypes must be substitutable | "If it looks like a duck, it should quack like a duck" |
| **I** - Interface Segregation | Many specific interfaces > one general | "Don't force clients to depend on methods they don't use" |
| **D** - Dependency Inversion | Depend on abstractions, not concretions | "High-level shouldn't depend on low-level - both depend on abstractions" |

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Dependency Inversion"** | "We use dependency inversion so our use cases don't depend on the database" |
| **"Port and Adapter"** | "The repository is a port, PostgresRepository is the adapter" |
| **"Screaming Architecture"** | "Our folder structure screams 'Order System', not 'Express app'" |
| **"The Dependency Rule"** | "Dependencies point inward - entities know nothing about controllers" |
| **"Boundary"** | "We cross the boundary using DTOs, not domain objects" |
| **"Interactor"** | "The use case interactor contains the application-specific business rules" |
| **"Clean separation"** | "Business logic is cleanly separated from infrastructure concerns" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Layers | **4** | Entities → Use Cases → Adapters → Frameworks |
| SOLID | **5** | SRP, OCP, LSP, ISP, DIP |
| Dependencies direction | **Inward only** | Never outward |
| Test coverage focus | **Inner layers** | Entities & Use Cases = 100% |

### The "Wow" Statement (Memorize This!)
> "Most codebases are tightly coupled to their frameworks - swap Express for Fastify and you rewrite everything. Clean Architecture inverts this: the framework is a plugin. Our business logic doesn't know if it's being called from an HTTP controller, a CLI, or a test. We can swap PostgreSQL for MongoDB by implementing one interface - zero business logic changes. The app is testable without spinning up any infrastructure because the domain has no external dependencies."

### Quick Architecture Drawing (Draw This!)
```
                    ┌─────────────────────────────────────┐
                    │     FRAMEWORKS & DRIVERS            │
                    │  (Express, PostgreSQL, React)       │
                    │                                     │
                    │   ┌─────────────────────────────┐   │
                    │   │   INTERFACE ADAPTERS        │   │
                    │   │ (Controllers, Repositories) │   │
                    │   │                             │   │
                    │   │   ┌─────────────────────┐   │   │
                    │   │   │    USE CASES        │   │   │
                    │   │   │  (Interactors)      │   │   │
                    │   │   │                     │   │   │
                    │   │   │   ┌─────────────┐   │   │   │
                    │   │   │   │  ENTITIES   │   │   │   │
                    │   │   │   │  (Domain)   │   │   │   │
                    │   │   │   └─────────────┘   │   │   │
                    │   │   │         ▲           │   │   │
                    │   │   └─────────┼───────────┘   │   │
                    │   │             │               │   │
                    │   └─────────────┼───────────────┘   │
                    │                 │                   │
                    └─────────────────┼───────────────────┘
                                      │
                         Dependencies point INWARD
```

### Interview Rapid Fire (Practice These!)

**Q: "What is Clean Architecture?"**
> "An architecture where business logic is at the center, independent of frameworks and databases. Dependencies point inward - inner layers don't know about outer layers."

**Q: "What is the Dependency Rule?"**
> "Source code dependencies can only point inward. Nothing in an inner circle can know anything about something in an outer circle. Inner layers define interfaces, outer layers implement them."

**Q: "What is Dependency Inversion?"**
> "High-level modules shouldn't depend on low-level modules. Both should depend on abstractions. Example: Use case defines a repository interface, infrastructure implements it."

**Q: "Why use Clean Architecture?"**
> "Testability - test business logic without infrastructure. Flexibility - swap frameworks/databases easily. Maintainability - changes in one layer don't ripple everywhere."

**Q: "What are the layers?"**
> "From inside out: Entities (domain objects), Use Cases (application rules), Interface Adapters (controllers, repos), Frameworks & Drivers (Express, PostgreSQL)."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "What is Clean Architecture?"

**Junior Answer:**
> "It's organizing code into layers like controllers and services."

**Senior Answer:**
> "Clean Architecture is about **dependency management**. The key insight is the **Dependency Rule**: source code dependencies can only point inward.

At the center are **Entities** - pure business objects with no external dependencies. They represent enterprise-wide business rules.

Next layer is **Use Cases** - application-specific business rules. A use case orchestrates entities to accomplish a specific goal. It knows WHAT to do, not HOW to persist or display.

**Interface Adapters** convert data between use cases and external agents. Controllers receive HTTP requests and call use cases. Repositories implement data access interfaces defined by use cases.

**Frameworks & Drivers** are the outermost details - Express, PostgreSQL, React. They're plugins, not the architecture.

The result? I can test my entire business logic without a database. I can swap PostgreSQL for MongoDB by implementing one interface. The framework is a detail, not the foundation."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "How do you handle data crossing boundaries?" | "Using DTOs. The inner layer defines what data it needs. The outer layer converts its format to that DTO. Domain objects never cross boundaries." |
| "Where does validation go?" | "Two types: business validation in entities/use cases (order can't be empty), input validation in controllers (email format). Different concerns." |
| "How is this different from layered architecture?" | "Traditional layers allow upward calls. Clean Architecture inverts dependencies - use cases define interfaces, infrastructure implements. Framework is a plugin, not foundation." |
| "Isn't this over-engineering?" | "For simple CRUD, yes. For complex business logic, the testability and flexibility pay off. Start simple, add boundaries when complexity grows." |

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [SOLID Principles](#2-solid-principles)
3. [The Layers](#3-the-layers)
4. [Dependency Injection](#4-dependency-injection)
5. [Implementation Examples](#5-implementation-examples)
6. [Folder Structure](#6-folder-structure)
7. [Testing Strategy](#7-testing-strategy)
8. [Common Mistakes](#8-common-mistakes)
9. [When to Use / Not Use](#9-when-to-use--not-use)
10. [Interview Questions](#10-interview-questions)

---

## 1. Core Concepts

### The Problem Clean Architecture Solves

```
THE PROBLEM (Typical Codebase):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Business Logic is SCATTERED everywhere:                       │
│                                                                  │
│   ┌──────────────┐                                              │
│   │  Controller  │ ─── validates, calls DB, sends email        │
│   └──────────────┘                                              │
│          │                                                       │
│          ▼                                                       │
│   ┌──────────────┐                                              │
│   │   Service    │ ─── more business logic, more DB calls      │
│   └──────────────┘                                              │
│          │                                                       │
│          ▼                                                       │
│   ┌──────────────┐                                              │
│   │  Repository  │ ─── even more business logic mixed in       │
│   └──────────────┘                                              │
│                                                                  │
│   PROBLEMS:                                                     │
│   └── Can't test business logic without database               │
│   └── Changing ORM requires changing everywhere                │
│   └── Business rules duplicated or scattered                   │
│   └── Framework upgrade = massive rewrite                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

THE SOLUTION (Clean Architecture):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Business Logic is CENTRALIZED and PROTECTED:                  │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │                     DOMAIN CENTER                        │  │
│   │   ┌─────────────┐   ┌─────────────┐                     │  │
│   │   │  Entities   │   │  Use Cases  │                     │  │
│   │   │  (Rules)    │   │ (Workflows) │                     │  │
│   │   └─────────────┘   └─────────────┘                     │  │
│   │                                                          │  │
│   │   NO dependencies on frameworks, DB, or UI              │  │
│   │   100% testable with plain unit tests                   │  │
│   └─────────────────────────────────────────────────────────┘  │
│                           ▲                                     │
│   ┌───────────────────────┼─────────────────────────────────┐  │
│   │   Infrastructure IMPLEMENTS interfaces defined above     │  │
│   │   └── PostgresRepository implements OrderRepository     │  │
│   │   └── ExpressController calls use cases                 │  │
│   │   └── SendGridMailer implements MailerInterface         │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The Concentric Circles

```
┌──────────────────────────────────────────────────────────────────────┐
│                        FRAMEWORKS & DRIVERS                           │
│  Express, React, PostgreSQL, Redis, AWS SDK, SendGrid                │
│                                                                       │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                    INTERFACE ADAPTERS                           │  │
│  │  Controllers, Presenters, Gateways, Repository Implementations │  │
│  │                                                                 │  │
│  │  ┌──────────────────────────────────────────────────────────┐  │  │
│  │  │                    USE CASES                              │  │  │
│  │  │  Application-specific business rules                     │  │  │
│  │  │  Orchestrates flow of data to/from entities              │  │  │
│  │  │                                                          │  │  │
│  │  │  ┌────────────────────────────────────────────────────┐  │  │  │
│  │  │  │                   ENTITIES                          │  │  │  │
│  │  │  │  Enterprise-wide business rules                    │  │  │  │
│  │  │  │  Domain objects with behavior                      │  │  │  │
│  │  │  │  No dependencies on anything external             │  │  │  │
│  │  │  └────────────────────────────────────────────────────┘  │  │  │
│  │  │                           ▲                              │  │  │
│  │  └───────────────────────────┼──────────────────────────────┘  │  │
│  │                              │                                  │  │
│  └──────────────────────────────┼──────────────────────────────────┘  │
│                                 │                                     │
└─────────────────────────────────┼─────────────────────────────────────┘
                                  │
                    Dependencies point INWARD only
```

### What Each Layer Contains

```
LAYER 1: ENTITIES (Innermost)
┌─────────────────────────────────────────────────────────────────┐
│  PURPOSE: Enterprise-wide business rules                         │
│                                                                  │
│  CONTAINS:                                                      │
│  └── Domain entities (User, Order, Product)                    │
│  └── Value objects (Money, Email, Address)                     │
│  └── Domain services (pricing calculations)                    │
│  └── Business rule validations                                 │
│                                                                  │
│  KNOWS ABOUT: Nothing external (pure business logic)           │
│  EXAMPLE: Order.canBeCancelled() - pure business rule          │
└─────────────────────────────────────────────────────────────────┘

LAYER 2: USE CASES (Application Business Rules)
┌─────────────────────────────────────────────────────────────────┐
│  PURPOSE: Application-specific business rules                   │
│                                                                  │
│  CONTAINS:                                                      │
│  └── Use case interactors (PlaceOrderUseCase)                  │
│  └── Input/Output ports (interfaces)                           │
│  └── DTOs for boundary crossing                                │
│                                                                  │
│  KNOWS ABOUT: Entities (layer 1)                               │
│  EXAMPLE: PlaceOrderUseCase orchestrates order creation        │
└─────────────────────────────────────────────────────────────────┘

LAYER 3: INTERFACE ADAPTERS
┌─────────────────────────────────────────────────────────────────┐
│  PURPOSE: Convert data between use cases and external world    │
│                                                                  │
│  CONTAINS:                                                      │
│  └── Controllers (HTTP → Use Case)                             │
│  └── Presenters (Use Case → HTTP Response)                     │
│  └── Repository implementations                                │
│  └── Gateway implementations                                   │
│                                                                  │
│  KNOWS ABOUT: Use Cases (layer 2), Entities (layer 1)         │
│  EXAMPLE: OrderController converts HTTP request to command     │
└─────────────────────────────────────────────────────────────────┘

LAYER 4: FRAMEWORKS & DRIVERS (Outermost)
┌─────────────────────────────────────────────────────────────────┐
│  PURPOSE: Glue code that connects everything                   │
│                                                                  │
│  CONTAINS:                                                      │
│  └── Web framework (Express, Fastify, NestJS)                  │
│  └── Database drivers (pg, mongoose, prisma)                   │
│  └── External service SDKs (AWS, Stripe, SendGrid)            │
│  └── UI framework (React, Vue)                                 │
│                                                                  │
│  KNOWS ABOUT: Everything (wires it all together)               │
│  EXAMPLE: Express app setup, Prisma client configuration       │
└─────────────────────────────────────────────────────────────────┘
```

### Crossing Boundaries

```typescript
// ════════════════════════════════════════════════════════════════
// HOW DATA CROSSES BOUNDARIES
// ════════════════════════════════════════════════════════════════

// WRONG: Passing framework objects into use cases
class OrderController {
  async createOrder(req: Request, res: Response) {
    // ❌ Passing Express request to use case
    await this.createOrderUseCase.execute(req.body);
  }
}

// RIGHT: Convert to DTOs at boundaries
class OrderController {
  async createOrder(req: Request, res: Response) {
    // ✅ Convert to DTO that use case understands
    const input: CreateOrderInput = {
      customerId: req.body.customerId,
      items: req.body.items.map(i => ({
        productId: i.productId,
        quantity: i.quantity
      }))
    };

    const output = await this.createOrderUseCase.execute(input);

    // ✅ Convert output to HTTP response
    res.status(201).json({
      orderId: output.orderId,
      total: output.total,
      status: output.status
    });
  }
}
```

---

## 2. SOLID Principles

### S - Single Responsibility Principle (SRP)

> "A class should have only one reason to change."

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ VIOLATES SRP: Multiple reasons to change
// ════════════════════════════════════════════════════════════════

class UserService {
  // Reason 1: User management logic changes
  createUser(data: CreateUserDTO): User {
    const user = new User(data);
    
    // Reason 2: Validation rules change
    if (!this.isValidEmail(data.email)) {
      throw new Error('Invalid email');
    }
    
    // Reason 3: Database changes
    this.db.query('INSERT INTO users...');
    
    // Reason 4: Email template changes
    this.sendEmail(user.email, 'Welcome!', this.generateWelcomeHtml());
    
    // Reason 5: Logging format changes
    console.log(`[${new Date().toISOString()}] User created: ${user.id}`);
    
    return user;
  }
}

// ════════════════════════════════════════════════════════════════
// ✅ FOLLOWS SRP: Each class has ONE reason to change
// ════════════════════════════════════════════════════════════════

class UserFactory {
  // Only changes if user creation logic changes
  create(data: CreateUserDTO): User {
    return new User(
      UserId.generate(),
      new Email(data.email),  // Email validates itself
      new UserName(data.name)
    );
  }
}

class UserRepository {
  // Only changes if database access changes
  async save(user: User): Promise<void> {
    await this.db.query('INSERT INTO users...', user.toSnapshot());
  }
}

class WelcomeEmailSender {
  // Only changes if welcome email logic changes
  async send(user: User): Promise<void> {
    await this.mailer.send({
      to: user.email.value,
      subject: 'Welcome!',
      html: this.template.render(user)
    });
  }
}

class UserLogger {
  // Only changes if logging requirements change
  logCreation(user: User): void {
    this.logger.info('User created', { userId: user.id.value });
  }
}

// Use case orchestrates them all
class CreateUserUseCase {
  constructor(
    private userFactory: UserFactory,
    private userRepo: UserRepository,
    private welcomeEmail: WelcomeEmailSender,
    private logger: UserLogger
  ) {}

  async execute(input: CreateUserInput): Promise<CreateUserOutput> {
    const user = this.userFactory.create(input);
    await this.userRepo.save(user);
    await this.welcomeEmail.send(user);
    this.logger.logCreation(user);
    return { userId: user.id.value };
  }
}
```

### O - Open/Closed Principle (OCP)

> "Software entities should be open for extension, but closed for modification."

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ VIOLATES OCP: Adding new payment method requires modification
// ════════════════════════════════════════════════════════════════

class PaymentProcessor {
  process(payment: Payment): void {
    // Every new payment type = modify this class
    if (payment.type === 'credit_card') {
      this.processCreditCard(payment);
    } else if (payment.type === 'paypal') {
      this.processPayPal(payment);
    } else if (payment.type === 'crypto') {
      this.processCrypto(payment);  // Had to modify!
    } else if (payment.type === 'apple_pay') {
      this.processApplePay(payment);  // Had to modify again!
    }
  }
}

// ════════════════════════════════════════════════════════════════
// ✅ FOLLOWS OCP: Adding new payment method = add new class
// ════════════════════════════════════════════════════════════════

// Closed for modification - this interface doesn't change
interface PaymentMethod {
  process(amount: Money): PaymentResult;
}

// Open for extension - add new implementations
class CreditCardPayment implements PaymentMethod {
  constructor(private stripe: StripeClient) {}
  
  process(amount: Money): PaymentResult {
    return this.stripe.charge(amount);
  }
}

class PayPalPayment implements PaymentMethod {
  constructor(private paypal: PayPalClient) {}
  
  process(amount: Money): PaymentResult {
    return this.paypal.createPayment(amount);
  }
}

// Adding crypto? Just add a new class - no modification needed!
class CryptoPayment implements PaymentMethod {
  constructor(private coinbase: CoinbaseClient) {}
  
  process(amount: Money): PaymentResult {
    return this.coinbase.createCharge(amount);
  }
}

// PaymentProcessor is now closed for modification
class PaymentProcessor {
  process(method: PaymentMethod, amount: Money): PaymentResult {
    return method.process(amount);
  }
}
```

### L - Liskov Substitution Principle (LSP)

> "Subtypes must be substitutable for their base types."

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ VIOLATES LSP: Square is not substitutable for Rectangle
// ════════════════════════════════════════════════════════════════

class Rectangle {
  protected width: number;
  protected height: number;

  setWidth(width: number): void {
    this.width = width;
  }

  setHeight(height: number): void {
    this.height = height;
  }

  getArea(): number {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  // ❌ Violates LSP: Changes behavior of setWidth/setHeight
  setWidth(width: number): void {
    this.width = width;
    this.height = width;  // Square must keep sides equal
  }

  setHeight(height: number): void {
    this.width = height;
    this.height = height;
  }
}

// This test FAILS for Square!
function testRectangle(rect: Rectangle) {
  rect.setWidth(5);
  rect.setHeight(4);
  // Expected: 20, but Square gives 16!
  console.assert(rect.getArea() === 20);
}

// ════════════════════════════════════════════════════════════════
// ✅ FOLLOWS LSP: Use composition or separate hierarchies
// ════════════════════════════════════════════════════════════════

interface Shape {
  getArea(): number;
}

class Rectangle implements Shape {
  constructor(
    private readonly width: number,
    private readonly height: number
  ) {}

  getArea(): number {
    return this.width * this.height;
  }
}

class Square implements Shape {
  constructor(private readonly side: number) {}

  getArea(): number {
    return this.side * this.side;
  }
}

// Both are substitutable for Shape
function printArea(shape: Shape) {
  console.log(`Area: ${shape.getArea()}`);
}
```

```typescript
// ════════════════════════════════════════════════════════════════
// Real-World LSP Example: Repository Pattern
// ════════════════════════════════════════════════════════════════

interface UserRepository {
  findById(id: UserId): Promise<User | null>;
  save(user: User): Promise<void>;
}

// ✅ PostgresUserRepository is substitutable
class PostgresUserRepository implements UserRepository {
  async findById(id: UserId): Promise<User | null> {
    const row = await this.db.query('SELECT * FROM users WHERE id = $1', [id]);
    return row ? this.mapper.toDomain(row) : null;
  }

  async save(user: User): Promise<void> {
    await this.db.query('INSERT INTO users...');
  }
}

// ✅ InMemoryUserRepository is substitutable (for tests)
class InMemoryUserRepository implements UserRepository {
  private users: Map<string, User> = new Map();

  async findById(id: UserId): Promise<User | null> {
    return this.users.get(id.value) ?? null;
  }

  async save(user: User): Promise<void> {
    this.users.set(user.id.value, user);
  }
}

// Use case works with ANY implementation
class GetUserUseCase {
  constructor(private userRepo: UserRepository) {}

  async execute(userId: string): Promise<User | null> {
    // Doesn't know or care if it's Postgres or InMemory
    return this.userRepo.findById(new UserId(userId));
  }
}
```

### I - Interface Segregation Principle (ISP)

> "Clients should not be forced to depend on interfaces they don't use."

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ VIOLATES ISP: Fat interface forces unnecessary dependencies
// ════════════════════════════════════════════════════════════════

interface UserRepository {
  findById(id: string): Promise<User>;
  findByEmail(email: string): Promise<User>;
  findAll(): Promise<User[]>;
  save(user: User): Promise<void>;
  delete(id: string): Promise<void>;
  updatePassword(id: string, password: string): Promise<void>;
  updateProfile(id: string, profile: Profile): Promise<void>;
  findByDepartment(dept: string): Promise<User[]>;
  countByRole(role: string): Promise<number>;
  exportToCSV(): Promise<string>;
  importFromCSV(csv: string): Promise<void>;
}

// This use case only needs findById, but depends on 11 methods!
class GetUserUseCase {
  constructor(private repo: UserRepository) {}
  
  async execute(id: string): Promise<User> {
    return this.repo.findById(id);  // Only uses 1 of 11 methods
  }
}

// ════════════════════════════════════════════════════════════════
// ✅ FOLLOWS ISP: Small, focused interfaces
// ════════════════════════════════════════════════════════════════

// Each interface has a single purpose
interface UserReader {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
}

interface UserWriter {
  save(user: User): Promise<void>;
  delete(id: string): Promise<void>;
}

interface UserQueryService {
  findAll(): Promise<User[]>;
  findByDepartment(dept: string): Promise<User[]>;
  countByRole(role: string): Promise<number>;
}

interface UserImportExport {
  exportToCSV(): Promise<string>;
  importFromCSV(csv: string): Promise<void>;
}

// Use case only depends on what it needs
class GetUserUseCase {
  constructor(private userReader: UserReader) {}
  
  async execute(id: string): Promise<User | null> {
    return this.userReader.findById(id);
  }
}

class CreateUserUseCase {
  constructor(private userWriter: UserWriter) {}
  
  async execute(user: User): Promise<void> {
    await this.userWriter.save(user);
  }
}

// Implementation can implement multiple interfaces
class PostgresUserRepository implements UserReader, UserWriter {
  async findById(id: string): Promise<User | null> { /* ... */ }
  async findByEmail(email: string): Promise<User | null> { /* ... */ }
  async save(user: User): Promise<void> { /* ... */ }
  async delete(id: string): Promise<void> { /* ... */ }
}
```

### D - Dependency Inversion Principle (DIP)

> "High-level modules should not depend on low-level modules. Both should depend on abstractions."

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ VIOLATES DIP: High-level depends on low-level
// ════════════════════════════════════════════════════════════════

import { PrismaClient } from '@prisma/client';  // Low-level detail
import { SendGrid } from '@sendgrid/mail';       // Low-level detail

class CreateOrderUseCase {
  private prisma = new PrismaClient();  // ❌ Direct dependency
  private sendgrid = new SendGrid();     // ❌ Direct dependency

  async execute(data: CreateOrderData): Promise<void> {
    // ❌ Tightly coupled to Prisma
    await this.prisma.order.create({ data });
    
    // ❌ Tightly coupled to SendGrid
    await this.sendgrid.send({ to: data.email, subject: 'Order Placed' });
  }
}

// Problems:
// - Can't test without real database and email service
// - Changing to MongoDB requires rewriting use case
// - Changing email provider requires rewriting use case

// ════════════════════════════════════════════════════════════════
// ✅ FOLLOWS DIP: Both depend on abstractions
// ════════════════════════════════════════════════════════════════

// High-level module DEFINES the interfaces (in domain/use-case layer)
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: OrderId): Promise<Order | null>;
}

interface EmailService {
  send(to: string, subject: string, body: string): Promise<void>;
}

// High-level module depends on abstractions
class CreateOrderUseCase {
  constructor(
    private orderRepo: OrderRepository,    // ✅ Abstraction
    private emailService: EmailService     // ✅ Abstraction
  ) {}

  async execute(input: CreateOrderInput): Promise<CreateOrderOutput> {
    const order = Order.create(input);
    await this.orderRepo.save(order);
    await this.emailService.send(
      input.customerEmail,
      'Order Placed',
      `Your order ${order.id} has been placed!`
    );
    return { orderId: order.id.value };
  }
}

// Low-level modules IMPLEMENT the abstractions (in infrastructure layer)
class PrismaOrderRepository implements OrderRepository {
  constructor(private prisma: PrismaClient) {}

  async save(order: Order): Promise<void> {
    await this.prisma.order.create({ data: order.toSnapshot() });
  }

  async findById(id: OrderId): Promise<Order | null> {
    const data = await this.prisma.order.findUnique({ where: { id: id.value } });
    return data ? Order.reconstitute(data) : null;
  }
}

class SendGridEmailService implements EmailService {
  constructor(private sendgrid: SendGrid) {}

  async send(to: string, subject: string, body: string): Promise<void> {
    await this.sendgrid.send({ to, subject, html: body });
  }
}

// For tests - no external dependencies!
class InMemoryOrderRepository implements OrderRepository {
  private orders = new Map<string, Order>();

  async save(order: Order): Promise<void> {
    this.orders.set(order.id.value, order);
  }

  async findById(id: OrderId): Promise<Order | null> {
    return this.orders.get(id.value) ?? null;
  }
}

class FakeEmailService implements EmailService {
  public sentEmails: Array<{ to: string; subject: string; body: string }> = [];

  async send(to: string, subject: string, body: string): Promise<void> {
    this.sentEmails.push({ to, subject, body });
  }
}

// Now we can test without infrastructure!
describe('CreateOrderUseCase', () => {
  it('sends confirmation email', async () => {
    const orderRepo = new InMemoryOrderRepository();
    const emailService = new FakeEmailService();
    const useCase = new CreateOrderUseCase(orderRepo, emailService);

    await useCase.execute({ customerEmail: 'test@example.com', items: [] });

    expect(emailService.sentEmails).toHaveLength(1);
    expect(emailService.sentEmails[0].to).toBe('test@example.com');
  });
});
```

### SOLID Summary Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         SOLID PRINCIPLES                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  S - Single Responsibility                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                          │
│  │  Class A │    │  Class B │    │  Class C │                          │
│  │ (1 job)  │    │ (1 job)  │    │ (1 job)  │                          │
│  └──────────┘    └──────────┘    └──────────┘                          │
│                                                                          │
│  O - Open/Closed                                                        │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                          │
│  │Interface │◄───│  Impl A  │    │  Impl B  │  ← Add new, don't modify │
│  │ (stable) │◄───│  (new)   │    │  (new)   │                          │
│  └──────────┘    └──────────┘    └──────────┘                          │
│                                                                          │
│  L - Liskov Substitution                                                │
│  ┌──────────┐                                                           │
│  │  Base    │◄─── All subtypes work where Base expected                │
│  │ (Parent) │                                                           │
│  └──────────┘                                                           │
│       ▲  ▲                                                              │
│  ┌────┘  └────┐                                                         │
│  │SubA│  │SubB│  ← Both substitutable                                  │
│  └────┘  └────┘                                                         │
│                                                                          │
│  I - Interface Segregation                                              │
│  ┌────┐  ┌────┐  ┌────┐                                                │
│  │IA  │  │ IB │  │ IC │  ← Small, focused interfaces                   │
│  └────┘  └────┘  └────┘                                                │
│                                                                          │
│  D - Dependency Inversion                                               │
│  ┌───────────────┐     ┌───────────────┐                               │
│  │ High-Level    │────►│  Abstraction  │◄────┐                         │
│  │ (Use Case)    │     │  (Interface)  │     │                         │
│  └───────────────┘     └───────────────┘     │                         │
│                                              │                         │
│                        ┌───────────────┐     │                         │
│                        │  Low-Level    │─────┘                         │
│                        │ (Repository)  │                               │
│                        └───────────────┘                               │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. The Layers

### Layer 1: Entities (Domain Layer)

The innermost layer - pure business logic with NO external dependencies.

```typescript
// ════════════════════════════════════════════════════════════════
// ENTITIES: Pure business objects with behavior
// ════════════════════════════════════════════════════════════════

// src/domain/entities/order.ts

// Value Object: Immutable, equality by attributes
class Money {
  private constructor(
    private readonly amount: number,
    private readonly currency: string
  ) {
    if (amount < 0) throw new Error('Amount cannot be negative');
  }

  static create(amount: number, currency: string): Money {
    return new Money(amount, currency);
  }

  static zero(currency: string = 'USD'): Money {
    return new Money(0, currency);
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Currency mismatch');
    }
    return new Money(this.amount + other.amount, this.currency);
  }

  multiply(factor: number): Money {
    return new Money(this.amount * factor, this.currency);
  }

  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }

  get value(): number {
    return this.amount;
  }
}

// Entity: Identity, can change over time
class Order {
  private _id: OrderId;
  private _customerId: CustomerId;
  private _lines: OrderLine[] = [];
  private _status: OrderStatus;
  private _createdAt: Date;

  private constructor(id: OrderId, customerId: CustomerId) {
    this._id = id;
    this._customerId = customerId;
    this._status = OrderStatus.Draft;
    this._createdAt = new Date();
  }

  // Factory method - encapsulates creation logic
  static create(customerId: CustomerId): Order {
    return new Order(OrderId.generate(), customerId);
  }

  // Reconstitution from persistence (no validation, already valid)
  static reconstitute(data: OrderSnapshot): Order {
    const order = new Order(
      new OrderId(data.id),
      new CustomerId(data.customerId)
    );
    order._status = data.status;
    order._lines = data.lines.map(l => OrderLine.reconstitute(l));
    order._createdAt = data.createdAt;
    return order;
  }

  // ══════════════════════════════════════════════════════════════
  // BUSINESS RULES live here - not in services!
  // ══════════════════════════════════════════════════════════════

  addLine(productId: ProductId, quantity: number, unitPrice: Money): void {
    this.ensureCanModify();
    
    if (quantity <= 0) {
      throw new DomainError('Quantity must be positive');
    }

    // Business rule: Can't add same product twice
    const existing = this._lines.find(l => l.productId.equals(productId));
    if (existing) {
      existing.increaseQuantity(quantity);
    } else {
      this._lines.push(new OrderLine(productId, quantity, unitPrice));
    }
  }

  removeLine(productId: ProductId): void {
    this.ensureCanModify();
    
    const index = this._lines.findIndex(l => l.productId.equals(productId));
    if (index === -1) {
      throw new DomainError('Product not in order');
    }
    this._lines.splice(index, 1);
  }

  place(): void {
    // Business rule: Can't place empty order
    if (this._lines.length === 0) {
      throw new DomainError('Cannot place empty order');
    }

    // Business rule: Minimum order value
    if (this.total().value < 10) {
      throw new DomainError('Minimum order is $10');
    }

    this._status = OrderStatus.Placed;
  }

  cancel(): void {
    // Business rule: Can only cancel if not shipped
    if (this._status === OrderStatus.Shipped) {
      throw new DomainError('Cannot cancel shipped order');
    }

    if (this._status === OrderStatus.Cancelled) {
      throw new DomainError('Order already cancelled');
    }

    this._status = OrderStatus.Cancelled;
  }

  total(): Money {
    return this._lines.reduce(
      (sum, line) => sum.add(line.subtotal()),
      Money.zero()
    );
  }

  // Snapshot for persistence (Domain → Infrastructure)
  toSnapshot(): OrderSnapshot {
    return {
      id: this._id.value,
      customerId: this._customerId.value,
      status: this._status,
      lines: this._lines.map(l => l.toSnapshot()),
      createdAt: this._createdAt
    };
  }

  private ensureCanModify(): void {
    if (this._status !== OrderStatus.Draft) {
      throw new DomainError('Cannot modify order after placement');
    }
  }

  // Getters (expose what's needed)
  get id(): OrderId { return this._id; }
  get status(): OrderStatus { return this._status; }
  get lines(): ReadonlyArray<OrderLine> { return [...this._lines]; }
}

// Domain errors - business rule violations
class DomainError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'DomainError';
  }
}
```

### Layer 2: Use Cases (Application Layer)

Application-specific business rules. Orchestrates domain objects.

```typescript
// ════════════════════════════════════════════════════════════════
// USE CASES: Application-specific business rules
// ════════════════════════════════════════════════════════════════

// src/application/use-cases/place-order.use-case.ts

// Input DTO - what the use case needs
interface PlaceOrderInput {
  customerId: string;
  items: Array<{
    productId: string;
    quantity: number;
  }>;
}

// Output DTO - what the use case returns
interface PlaceOrderOutput {
  orderId: string;
  total: number;
  status: string;
}

// Port: Interface that infrastructure must implement
// Defined in use case layer, implemented in infrastructure
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: OrderId): Promise<Order | null>;
  nextId(): OrderId;
}

interface ProductCatalog {
  getPrice(productId: ProductId): Promise<Money>;
  exists(productId: ProductId): Promise<boolean>;
}

interface CustomerRepository {
  findById(id: CustomerId): Promise<Customer | null>;
  exists(id: CustomerId): Promise<boolean>;
}

// The Use Case (Interactor)
class PlaceOrderUseCase {
  constructor(
    private orderRepo: OrderRepository,
    private customerRepo: CustomerRepository,
    private productCatalog: ProductCatalog
  ) {}

  async execute(input: PlaceOrderInput): Promise<PlaceOrderOutput> {
    // 1. Validate customer exists
    const customerId = new CustomerId(input.customerId);
    const customerExists = await this.customerRepo.exists(customerId);
    if (!customerExists) {
      throw new ApplicationError('Customer not found');
    }

    // 2. Create order (domain factory)
    const order = Order.create(customerId);

    // 3. Add items with current prices
    for (const item of input.items) {
      const productId = new ProductId(item.productId);
      
      // Validate product exists
      const exists = await this.productCatalog.exists(productId);
      if (!exists) {
        throw new ApplicationError(`Product ${item.productId} not found`);
      }

      // Get current price (prices can change)
      const price = await this.productCatalog.getPrice(productId);
      
      // Domain logic in entity
      order.addLine(productId, item.quantity, price);
    }

    // 4. Place order (domain logic)
    order.place();

    // 5. Persist
    await this.orderRepo.save(order);

    // 6. Return output DTO
    return {
      orderId: order.id.value,
      total: order.total().value,
      status: order.status
    };
  }
}

// Application error - use case failures
class ApplicationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ApplicationError';
  }
}
```

### Use Case Patterns

```typescript
// ════════════════════════════════════════════════════════════════
// COMMAND USE CASE: Changes state
// ════════════════════════════════════════════════════════════════

interface CancelOrderInput {
  orderId: string;
  reason: string;
}

interface CancelOrderOutput {
  success: boolean;
  refundAmount?: number;
}

class CancelOrderUseCase {
  constructor(
    private orderRepo: OrderRepository,
    private paymentService: PaymentService
  ) {}

  async execute(input: CancelOrderInput): Promise<CancelOrderOutput> {
    const order = await this.orderRepo.findById(new OrderId(input.orderId));
    
    if (!order) {
      throw new ApplicationError('Order not found');
    }

    // Domain logic
    order.cancel();

    // Save state change
    await this.orderRepo.save(order);

    // Trigger refund if paid
    let refundAmount: number | undefined;
    if (order.isPaid()) {
      refundAmount = await this.paymentService.refund(order.id);
    }

    return { success: true, refundAmount };
  }
}

// ════════════════════════════════════════════════════════════════
// QUERY USE CASE: Reads state (no side effects)
// ════════════════════════════════════════════════════════════════

interface GetOrderInput {
  orderId: string;
}

interface GetOrderOutput {
  id: string;
  customerId: string;
  status: string;
  items: Array<{
    productId: string;
    productName: string;
    quantity: number;
    unitPrice: number;
    subtotal: number;
  }>;
  total: number;
  createdAt: string;
}

interface OrderReadModel {
  getOrderDetails(orderId: string): Promise<GetOrderOutput | null>;
}

class GetOrderUseCase {
  constructor(private orderReadModel: OrderReadModel) {}

  async execute(input: GetOrderInput): Promise<GetOrderOutput> {
    const order = await this.orderReadModel.getOrderDetails(input.orderId);
    
    if (!order) {
      throw new ApplicationError('Order not found');
    }

    return order;
  }
}
```

### Layer 3: Interface Adapters

Convert data between use cases and external formats.

```typescript
// ════════════════════════════════════════════════════════════════
// CONTROLLERS: HTTP → Use Case
// ════════════════════════════════════════════════════════════════

// src/adapters/controllers/order.controller.ts

class OrderController {
  constructor(
    private placeOrderUseCase: PlaceOrderUseCase,
    private cancelOrderUseCase: CancelOrderUseCase,
    private getOrderUseCase: GetOrderUseCase
  ) {}

  // Convert HTTP request → Use Case input
  async placeOrder(req: HttpRequest): Promise<HttpResponse> {
    try {
      // Validate HTTP input (not business validation!)
      const validation = this.validatePlaceOrderRequest(req.body);
      if (!validation.valid) {
        return HttpResponse.badRequest(validation.errors);
      }

      // Call use case with DTO
      const output = await this.placeOrderUseCase.execute({
        customerId: req.body.customerId,
        items: req.body.items
      });

      // Convert output → HTTP response
      return HttpResponse.created({
        orderId: output.orderId,
        total: output.total,
        status: output.status
      });

    } catch (error) {
      return this.handleError(error);
    }
  }

  async cancelOrder(req: HttpRequest): Promise<HttpResponse> {
    try {
      const output = await this.cancelOrderUseCase.execute({
        orderId: req.params.orderId,
        reason: req.body.reason
      });

      return HttpResponse.ok(output);

    } catch (error) {
      return this.handleError(error);
    }
  }

  async getOrder(req: HttpRequest): Promise<HttpResponse> {
    try {
      const output = await this.getOrderUseCase.execute({
        orderId: req.params.orderId
      });

      return HttpResponse.ok(output);

    } catch (error) {
      return this.handleError(error);
    }
  }

  private handleError(error: Error): HttpResponse {
    if (error instanceof ApplicationError) {
      return HttpResponse.badRequest({ message: error.message });
    }
    if (error instanceof DomainError) {
      return HttpResponse.unprocessableEntity({ message: error.message });
    }
    // Log and return generic error
    console.error(error);
    return HttpResponse.internalError({ message: 'Internal server error' });
  }

  private validatePlaceOrderRequest(body: any): ValidationResult {
    const errors: string[] = [];
    
    if (!body.customerId) errors.push('customerId is required');
    if (!Array.isArray(body.items)) errors.push('items must be an array');
    if (body.items?.length === 0) errors.push('items cannot be empty');

    return { valid: errors.length === 0, errors };
  }
}

// ════════════════════════════════════════════════════════════════
// REPOSITORY IMPLEMENTATION: Use Case Port → Database
// ════════════════════════════════════════════════════════════════

// src/adapters/repositories/postgres-order.repository.ts

class PostgresOrderRepository implements OrderRepository {
  constructor(private db: DatabaseConnection) {}

  async save(order: Order): Promise<void> {
    const snapshot = order.toSnapshot();

    await this.db.transaction(async (tx) => {
      await tx.query(`
        INSERT INTO orders (id, customer_id, status, created_at)
        VALUES ($1, $2, $3, $4)
        ON CONFLICT (id) DO UPDATE SET status = $3
      `, [snapshot.id, snapshot.customerId, snapshot.status, snapshot.createdAt]);

      // Delete and recreate lines (simple approach)
      await tx.query('DELETE FROM order_lines WHERE order_id = $1', [snapshot.id]);

      for (const line of snapshot.lines) {
        await tx.query(`
          INSERT INTO order_lines (order_id, product_id, quantity, unit_price)
          VALUES ($1, $2, $3, $4)
        `, [snapshot.id, line.productId, line.quantity, line.unitPrice]);
      }
    });
  }

  async findById(id: OrderId): Promise<Order | null> {
    const orderRow = await this.db.query(
      'SELECT * FROM orders WHERE id = $1',
      [id.value]
    );

    if (!orderRow) return null;

    const lineRows = await this.db.query(
      'SELECT * FROM order_lines WHERE order_id = $1',
      [id.value]
    );

    // Reconstitute domain object from database rows
    return Order.reconstitute({
      id: orderRow.id,
      customerId: orderRow.customer_id,
      status: orderRow.status as OrderStatus,
      lines: lineRows.map(row => ({
        productId: row.product_id,
        quantity: row.quantity,
        unitPrice: row.unit_price
      })),
      createdAt: orderRow.created_at
    });
  }

  nextId(): OrderId {
    return OrderId.generate();
  }
}

// ════════════════════════════════════════════════════════════════
// GATEWAY IMPLEMENTATION: Use Case Port → External Service
// ════════════════════════════════════════════════════════════════

// src/adapters/gateways/stripe-payment.service.ts

interface PaymentService {
  charge(amount: Money, customerId: CustomerId): Promise<PaymentResult>;
  refund(paymentId: PaymentId): Promise<number>;
}

class StripePaymentService implements PaymentService {
  constructor(private stripe: Stripe) {}

  async charge(amount: Money, customerId: CustomerId): Promise<PaymentResult> {
    try {
      const stripeCustomer = await this.getStripeCustomerId(customerId);
      
      const paymentIntent = await this.stripe.paymentIntents.create({
        amount: Math.round(amount.value * 100),  // Stripe uses cents
        currency: 'usd',
        customer: stripeCustomer
      });

      return {
        success: true,
        paymentId: new PaymentId(paymentIntent.id)
      };

    } catch (error) {
      return {
        success: false,
        error: error.message
      };
    }
  }

  async refund(paymentId: PaymentId): Promise<number> {
    const refund = await this.stripe.refunds.create({
      payment_intent: paymentId.value
    });
    
    return refund.amount / 100;  // Convert from cents
  }
}
```

### Layer 4: Frameworks & Drivers

The outermost layer - glue code that wires everything together.

```typescript
// ════════════════════════════════════════════════════════════════
// FRAMEWORK LAYER: Wiring and configuration
// ════════════════════════════════════════════════════════════════

// src/main/server.ts (Express setup)

import express from 'express';
import { Pool } from 'pg';
import Stripe from 'stripe';

// Create infrastructure instances
const db = new Pool({ connectionString: process.env.DATABASE_URL });
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

// Create repositories (adapters)
const orderRepository = new PostgresOrderRepository(db);
const customerRepository = new PostgresCustomerRepository(db);
const productCatalog = new PostgresProductCatalog(db);
const paymentService = new StripePaymentService(stripe);

// Create use cases
const placeOrderUseCase = new PlaceOrderUseCase(
  orderRepository,
  customerRepository,
  productCatalog
);
const cancelOrderUseCase = new CancelOrderUseCase(
  orderRepository,
  paymentService
);
const getOrderUseCase = new GetOrderUseCase(orderReadModel);

// Create controllers
const orderController = new OrderController(
  placeOrderUseCase,
  cancelOrderUseCase,
  getOrderUseCase
);

// Create Express app
const app = express();
app.use(express.json());

// Routes (thin layer - just wire HTTP to controller)
app.post('/orders', async (req, res) => {
  const httpRequest: HttpRequest = { body: req.body, params: {}, query: {} };
  const httpResponse = await orderController.placeOrder(httpRequest);
  res.status(httpResponse.statusCode).json(httpResponse.body);
});

app.delete('/orders/:orderId', async (req, res) => {
  const httpRequest: HttpRequest = { body: req.body, params: req.params, query: {} };
  const httpResponse = await orderController.cancelOrder(httpRequest);
  res.status(httpResponse.statusCode).json(httpResponse.body);
});

app.get('/orders/:orderId', async (req, res) => {
  const httpRequest: HttpRequest = { body: {}, params: req.params, query: {} };
  const httpResponse = await orderController.getOrder(httpRequest);
  res.status(httpResponse.statusCode).json(httpResponse.body);
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

---

## 4. Dependency Injection

### What is Dependency Injection?

```
DEPENDENCY INJECTION (DI):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Instead of creating dependencies inside a class,               │
│  INJECT them from outside.                                      │
│                                                                  │
│  ❌ WITHOUT DI:                                                 │
│  class UserService {                                            │
│    private db = new Database();  // Creates own dependency     │
│  }                                                              │
│                                                                  │
│  ✅ WITH DI:                                                    │
│  class UserService {                                            │
│    constructor(private db: Database) {}  // Injected           │
│  }                                                              │
│                                                                  │
│  BENEFITS:                                                      │
│  └── Testable: Inject mocks for testing                        │
│  └── Flexible: Swap implementations easily                     │
│  └── Decoupled: Class doesn't know how dependencies created   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Three Types of DI

```typescript
// ════════════════════════════════════════════════════════════════
// 1. CONSTRUCTOR INJECTION (Preferred!)
// ════════════════════════════════════════════════════════════════

class OrderService {
  constructor(
    private orderRepo: OrderRepository,
    private paymentService: PaymentService,
    private emailService: EmailService
  ) {}

  async placeOrder(order: Order): Promise<void> {
    await this.orderRepo.save(order);
    await this.paymentService.charge(order.total);
    await this.emailService.sendConfirmation(order);
  }
}

// ════════════════════════════════════════════════════════════════
// 2. SETTER INJECTION (Use sparingly)
// ════════════════════════════════════════════════════════════════

class OrderService {
  private logger?: Logger;

  setLogger(logger: Logger): void {
    this.logger = logger;
  }

  async placeOrder(order: Order): Promise<void> {
    this.logger?.info('Placing order');
    // ...
  }
}

// ════════════════════════════════════════════════════════════════
// 3. INTERFACE INJECTION (Rare)
// ════════════════════════════════════════════════════════════════

interface LoggerAware {
  setLogger(logger: Logger): void;
}

class OrderService implements LoggerAware {
  private logger!: Logger;

  setLogger(logger: Logger): void {
    this.logger = logger;
  }
}
```

### Manual DI (Composition Root)

```typescript
// ════════════════════════════════════════════════════════════════
// COMPOSITION ROOT: Where all dependencies are wired
// ════════════════════════════════════════════════════════════════

// src/main/composition-root.ts

class CompositionRoot {
  // Infrastructure
  private db: DatabaseConnection;
  private stripe: Stripe;
  private sendgrid: SendGrid;

  // Repositories
  private orderRepository: OrderRepository;
  private customerRepository: CustomerRepository;
  private productRepository: ProductRepository;

  // Services
  private paymentService: PaymentService;
  private emailService: EmailService;

  // Use Cases
  private placeOrderUseCase: PlaceOrderUseCase;
  private cancelOrderUseCase: CancelOrderUseCase;
  private getOrderUseCase: GetOrderUseCase;

  // Controllers
  private orderController: OrderController;

  constructor() {
    this.initializeInfrastructure();
    this.initializeRepositories();
    this.initializeServices();
    this.initializeUseCases();
    this.initializeControllers();
  }

  private initializeInfrastructure(): void {
    this.db = new PostgresConnection(process.env.DATABASE_URL!);
    this.stripe = new Stripe(process.env.STRIPE_KEY!);
    this.sendgrid = new SendGrid(process.env.SENDGRID_KEY!);
  }

  private initializeRepositories(): void {
    this.orderRepository = new PostgresOrderRepository(this.db);
    this.customerRepository = new PostgresCustomerRepository(this.db);
    this.productRepository = new PostgresProductRepository(this.db);
  }

  private initializeServices(): void {
    this.paymentService = new StripePaymentService(this.stripe);
    this.emailService = new SendGridEmailService(this.sendgrid);
  }

  private initializeUseCases(): void {
    this.placeOrderUseCase = new PlaceOrderUseCase(
      this.orderRepository,
      this.customerRepository,
      this.productRepository,
      this.paymentService,
      this.emailService
    );

    this.cancelOrderUseCase = new CancelOrderUseCase(
      this.orderRepository,
      this.paymentService
    );

    this.getOrderUseCase = new GetOrderUseCase(
      this.orderRepository
    );
  }

  private initializeControllers(): void {
    this.orderController = new OrderController(
      this.placeOrderUseCase,
      this.cancelOrderUseCase,
      this.getOrderUseCase
    );
  }

  getOrderController(): OrderController {
    return this.orderController;
  }
}

// Usage in main.ts
const root = new CompositionRoot();
const orderController = root.getOrderController();
```

### DI Container (Automated)

```typescript
// ════════════════════════════════════════════════════════════════
// USING A DI CONTAINER (e.g., tsyringe, inversify)
// ════════════════════════════════════════════════════════════════

// With tsyringe
import { container, injectable, inject } from 'tsyringe';

// Register implementations
container.register<OrderRepository>('OrderRepository', {
  useClass: PostgresOrderRepository
});

container.register<PaymentService>('PaymentService', {
  useClass: StripePaymentService
});

// Use case receives injected dependencies
@injectable()
class PlaceOrderUseCase {
  constructor(
    @inject('OrderRepository') private orderRepo: OrderRepository,
    @inject('PaymentService') private paymentService: PaymentService
  ) {}
}

// Resolve with all dependencies injected
const useCase = container.resolve(PlaceOrderUseCase);

// ════════════════════════════════════════════════════════════════
// USING INVERSIFY
// ════════════════════════════════════════════════════════════════

import { Container, injectable, inject } from 'inversify';

const TYPES = {
  OrderRepository: Symbol.for('OrderRepository'),
  PaymentService: Symbol.for('PaymentService'),
  PlaceOrderUseCase: Symbol.for('PlaceOrderUseCase')
};

@injectable()
class PlaceOrderUseCase {
  constructor(
    @inject(TYPES.OrderRepository) private orderRepo: OrderRepository,
    @inject(TYPES.PaymentService) private paymentService: PaymentService
  ) {}
}

// Configure container
const container = new Container();
container.bind<OrderRepository>(TYPES.OrderRepository)
  .to(PostgresOrderRepository);
container.bind<PaymentService>(TYPES.PaymentService)
  .to(StripePaymentService);
container.bind<PlaceOrderUseCase>(TYPES.PlaceOrderUseCase)
  .to(PlaceOrderUseCase);

// Resolve
const useCase = container.get<PlaceOrderUseCase>(TYPES.PlaceOrderUseCase);
```

### Testing with DI

```typescript
// ════════════════════════════════════════════════════════════════
// TESTING: Inject fakes/mocks instead of real implementations
// ════════════════════════════════════════════════════════════════

// Test doubles
class InMemoryOrderRepository implements OrderRepository {
  private orders = new Map<string, Order>();

  async save(order: Order): Promise<void> {
    this.orders.set(order.id.value, order);
  }

  async findById(id: OrderId): Promise<Order | null> {
    return this.orders.get(id.value) ?? null;
  }

  // Test helper
  getAll(): Order[] {
    return Array.from(this.orders.values());
  }
}

class FakePaymentService implements PaymentService {
  public chargeAttempts: Array<{ amount: Money }> = [];
  public shouldFail = false;

  async charge(amount: Money): Promise<PaymentResult> {
    this.chargeAttempts.push({ amount });
    
    if (this.shouldFail) {
      return { success: false, error: 'Payment declined' };
    }
    
    return { success: true, paymentId: new PaymentId('fake-payment-id') };
  }
}

class FakeEmailService implements EmailService {
  public sentEmails: Array<{ to: string; subject: string }> = [];

  async send(to: string, subject: string): Promise<void> {
    this.sentEmails.push({ to, subject });
  }
}

// Test
describe('PlaceOrderUseCase', () => {
  let orderRepo: InMemoryOrderRepository;
  let paymentService: FakePaymentService;
  let emailService: FakeEmailService;
  let useCase: PlaceOrderUseCase;

  beforeEach(() => {
    // Fresh fakes for each test
    orderRepo = new InMemoryOrderRepository();
    paymentService = new FakePaymentService();
    emailService = new FakeEmailService();
    
    // Inject fakes
    useCase = new PlaceOrderUseCase(
      orderRepo,
      paymentService,
      emailService
    );
  });

  it('should save order and charge payment', async () => {
    // Arrange
    const input = {
      customerId: 'customer-1',
      items: [{ productId: 'product-1', quantity: 2 }]
    };

    // Act
    const output = await useCase.execute(input);

    // Assert
    expect(output.orderId).toBeDefined();
    expect(orderRepo.getAll()).toHaveLength(1);
    expect(paymentService.chargeAttempts).toHaveLength(1);
    expect(emailService.sentEmails).toHaveLength(1);
  });

  it('should not save order if payment fails', async () => {
    // Arrange
    paymentService.shouldFail = true;
    const input = { customerId: 'customer-1', items: [] };

    // Act & Assert
    await expect(useCase.execute(input)).rejects.toThrow('Payment failed');
    expect(orderRepo.getAll()).toHaveLength(0);  // Not saved
    expect(emailService.sentEmails).toHaveLength(0);  // No email
  });
});
```

### Dependency Inversion vs Dependency Injection

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  DEPENDENCY INVERSION (Principle - the WHY)                    │
│  └── High-level modules shouldn't depend on low-level          │
│  └── Both should depend on abstractions                        │
│  └── DESIGN PRINCIPLE                                          │
│                                                                  │
│  DEPENDENCY INJECTION (Technique - the HOW)                    │
│  └── Pass dependencies from outside                            │
│  └── Constructor injection, setter injection                   │
│  └── IMPLEMENTATION TECHNIQUE                                  │
│                                                                  │
│  RELATIONSHIP:                                                  │
│  └── DI is ONE way to achieve Dependency Inversion            │
│  └── You can have DI without following DIP                    │
│  └── You can follow DIP with other techniques (like factories) │
│                                                                  │
│  EXAMPLE:                                                       │
│                                                                  │
│  // DI without DIP (still depends on concrete class)           │
│  class Service {                                                │
│    constructor(private repo: PostgresRepository) {}  // ❌      │
│  }                                                              │
│                                                                  │
│  // DI with DIP (depends on abstraction)                       │
│  class Service {                                                │
│    constructor(private repo: Repository) {}  // ✅ Interface   │
│  }                                                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Implementation Examples

### Complete Order Flow Example

```typescript
// ════════════════════════════════════════════════════════════════
// FULL FLOW: HTTP Request → Use Case → Domain → Database
// ════════════════════════════════════════════════════════════════

// 1️⃣ HTTP Request comes in
// POST /orders
// { "customerId": "cust-123", "items": [{ "productId": "prod-456", "quantity": 2 }] }

// 2️⃣ Express route receives request
app.post('/orders', async (req, res) => {
  const httpRequest = { body: req.body, params: {}, headers: req.headers };
  const response = await orderController.create(httpRequest);
  res.status(response.status).json(response.body);
});

// 3️⃣ Controller converts HTTP → Use Case Input
class OrderController {
  async create(req: HttpRequest): Promise<HttpResponse> {
    const input: CreateOrderInput = {
      customerId: req.body.customerId,
      items: req.body.items
    };

    try {
      const output = await this.createOrderUseCase.execute(input);
      return { status: 201, body: output };
    } catch (error) {
      // Error handling...
    }
  }
}

// 4️⃣ Use Case orchestrates domain objects
class CreateOrderUseCase {
  async execute(input: CreateOrderInput): Promise<CreateOrderOutput> {
    // Load customer
    const customer = await this.customerRepo.findById(input.customerId);
    if (!customer) throw new NotFoundError('Customer');

    // Create order (domain factory)
    const order = Order.create(customer.id);

    // Add items
    for (const item of input.items) {
      const product = await this.productRepo.findById(item.productId);
      order.addLine(product.id, item.quantity, product.price);
    }

    // Domain validation happens inside Order
    order.place();

    // Save
    await this.orderRepo.save(order);

    // Return output DTO
    return { orderId: order.id.value, total: order.total().value };
  }
}

// 5️⃣ Domain Entity contains business rules
class Order {
  place(): void {
    if (this._lines.length === 0) {
      throw new DomainError('Cannot place empty order');
    }
    if (this.total().value < 10) {
      throw new DomainError('Minimum order is $10');
    }
    this._status = OrderStatus.Placed;
  }
}

// 6️⃣ Repository persists to database
class PostgresOrderRepository {
  async save(order: Order): Promise<void> {
    const snapshot = order.toSnapshot();
    await this.db.query('INSERT INTO orders...', snapshot);
  }
}

// 7️⃣ Response flows back up
// { status: 201, body: { orderId: "ord-789", total: 59.99 } }
```

---

## 6. Folder Structure

### Option 1: Layer-Based (Common)

```
src/
├── domain/                        # Layer 1: Entities
│   ├── entities/
│   │   ├── order.ts
│   │   ├── customer.ts
│   │   └── product.ts
│   ├── value-objects/
│   │   ├── money.ts
│   │   ├── email.ts
│   │   └── order-id.ts
│   ├── services/                  # Domain services
│   │   └── pricing.service.ts
│   └── errors/
│       └── domain.error.ts
│
├── application/                   # Layer 2: Use Cases
│   ├── use-cases/
│   │   ├── orders/
│   │   │   ├── create-order.use-case.ts
│   │   │   ├── cancel-order.use-case.ts
│   │   │   └── get-order.use-case.ts
│   │   └── customers/
│   │       └── create-customer.use-case.ts
│   ├── ports/                     # Interfaces for infra
│   │   ├── repositories/
│   │   │   ├── order.repository.ts
│   │   │   └── customer.repository.ts
│   │   └── services/
│   │       ├── payment.service.ts
│   │       └── email.service.ts
│   └── dtos/
│       ├── create-order.dto.ts
│       └── order-response.dto.ts
│
├── adapters/                      # Layer 3: Interface Adapters
│   ├── controllers/
│   │   ├── order.controller.ts
│   │   └── customer.controller.ts
│   ├── repositories/
│   │   ├── postgres-order.repository.ts
│   │   └── postgres-customer.repository.ts
│   ├── gateways/
│   │   ├── stripe-payment.service.ts
│   │   └── sendgrid-email.service.ts
│   └── presenters/
│       └── order.presenter.ts
│
├── infrastructure/                # Layer 4: Frameworks & Drivers
│   ├── database/
│   │   ├── connection.ts
│   │   └── migrations/
│   ├── http/
│   │   ├── server.ts
│   │   └── routes.ts
│   └── config/
│       └── env.ts
│
└── main/                          # Composition Root
    ├── composition-root.ts
    └── index.ts
```

### Option 2: Feature-Based (Screaming Architecture)

```
src/
├── orders/                        # Feature: Orders
│   ├── domain/
│   │   ├── order.entity.ts
│   │   ├── order-line.entity.ts
│   │   └── order.repository.ts    # Interface
│   ├── application/
│   │   ├── create-order.use-case.ts
│   │   ├── cancel-order.use-case.ts
│   │   └── dtos/
│   ├── infrastructure/
│   │   ├── postgres-order.repository.ts
│   │   └── order.mapper.ts
│   └── presentation/
│       ├── order.controller.ts
│       └── order.routes.ts
│
├── customers/                     # Feature: Customers
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── presentation/
│
├── payments/                      # Feature: Payments
│   ├── domain/
│   ├── application/
│   ├── infrastructure/
│   └── presentation/
│
├── shared/                        # Shared kernel
│   ├── domain/
│   │   ├── value-objects/
│   │   │   ├── money.ts
│   │   │   └── email.ts
│   │   └── base/
│   │       ├── entity.ts
│   │       └── aggregate-root.ts
│   └── infrastructure/
│       ├── database/
│       └── http/
│
└── main/
    ├── composition-root.ts
    └── server.ts
```

### Screaming Architecture

```
"SCREAMING ARCHITECTURE":
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Your architecture should SCREAM its intent.                    │
│                                                                  │
│  ❌ BAD: Folder structure screams "Express app"                 │
│  src/                                                           │
│  ├── controllers/                                               │
│  ├── models/                                                    │
│  ├── routes/                                                    │
│  └── middleware/                                                │
│                                                                  │
│  What is this app about? No idea. Maybe a blog? E-commerce?    │
│                                                                  │
│  ✅ GOOD: Folder structure screams "Healthcare System"          │
│  src/                                                           │
│  ├── patients/                                                  │
│  ├── appointments/                                              │
│  ├── prescriptions/                                             │
│  ├── billing/                                                   │
│  └── medical-records/                                           │
│                                                                  │
│  Instantly know this is a healthcare system!                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Testing Strategy

### The Testing Pyramid in Clean Architecture

```
                    ┌───────────┐
                    │    E2E    │ ← Few: Test full system
                    │   Tests   │   HTTP → DB → HTTP
                    ├───────────┤
                    │Integration│ ← Some: Test adapters
                    │   Tests   │   Controller + Repository
                    ├───────────┤
                    │   Unit    │ ← Many: Test domain & use cases
                    │   Tests   │   Pure logic, no I/O
                    └───────────┘

WHERE TO FOCUS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  LAYER              │ TEST TYPE      │ COVERAGE                 │
│  ─────────────────────────────────────────────────────────────  │
│  Entities           │ Unit           │ 100% - Pure logic       │
│  Use Cases          │ Unit           │ 100% - Mock ports       │
│  Controllers        │ Integration    │ High - Test conversion  │
│  Repositories       │ Integration    │ High - Test with DB     │
│  Framework (routes) │ E2E            │ Smoke tests             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Unit Testing Domain (Entities)

```typescript
// ════════════════════════════════════════════════════════════════
// TESTING ENTITIES: No mocks needed - pure logic
// ════════════════════════════════════════════════════════════════

describe('Order', () => {
  describe('create', () => {
    it('should create order in draft status', () => {
      const order = Order.create(new CustomerId('cust-1'));
      
      expect(order.status).toBe(OrderStatus.Draft);
      expect(order.lines).toHaveLength(0);
    });
  });

  describe('addLine', () => {
    it('should add line to order', () => {
      const order = Order.create(new CustomerId('cust-1'));
      
      order.addLine(
        new ProductId('prod-1'),
        2,
        Money.create(10, 'USD')
      );

      expect(order.lines).toHaveLength(1);
      expect(order.total().value).toBe(20);
    });

    it('should increase quantity for existing product', () => {
      const order = Order.create(new CustomerId('cust-1'));
      const productId = new ProductId('prod-1');
      
      order.addLine(productId, 2, Money.create(10, 'USD'));
      order.addLine(productId, 3, Money.create(10, 'USD'));

      expect(order.lines).toHaveLength(1);
      expect(order.lines[0].quantity).toBe(5);
    });

    it('should throw when quantity is zero or negative', () => {
      const order = Order.create(new CustomerId('cust-1'));
      
      expect(() => {
        order.addLine(new ProductId('prod-1'), 0, Money.create(10, 'USD'));
      }).toThrow('Quantity must be positive');
    });
  });

  describe('place', () => {
    it('should place order with items', () => {
      const order = Order.create(new CustomerId('cust-1'));
      order.addLine(new ProductId('prod-1'), 2, Money.create(10, 'USD'));

      order.place();

      expect(order.status).toBe(OrderStatus.Placed);
    });

    it('should throw when order is empty', () => {
      const order = Order.create(new CustomerId('cust-1'));

      expect(() => order.place()).toThrow('Cannot place empty order');
    });

    it('should throw when total is below minimum', () => {
      const order = Order.create(new CustomerId('cust-1'));
      order.addLine(new ProductId('prod-1'), 1, Money.create(5, 'USD'));

      expect(() => order.place()).toThrow('Minimum order is $10');
    });
  });

  describe('cancel', () => {
    it('should cancel placed order', () => {
      const order = Order.create(new CustomerId('cust-1'));
      order.addLine(new ProductId('prod-1'), 2, Money.create(10, 'USD'));
      order.place();

      order.cancel();

      expect(order.status).toBe(OrderStatus.Cancelled);
    });

    it('should throw when cancelling shipped order', () => {
      const order = createShippedOrder();

      expect(() => order.cancel()).toThrow('Cannot cancel shipped order');
    });
  });
});
```

### Unit Testing Use Cases

```typescript
// ════════════════════════════════════════════════════════════════
// TESTING USE CASES: Mock/fake the ports
// ════════════════════════════════════════════════════════════════

describe('CreateOrderUseCase', () => {
  let useCase: CreateOrderUseCase;
  let orderRepo: InMemoryOrderRepository;
  let customerRepo: InMemoryCustomerRepository;
  let productRepo: InMemoryProductRepository;

  beforeEach(() => {
    orderRepo = new InMemoryOrderRepository();
    customerRepo = new InMemoryCustomerRepository();
    productRepo = new InMemoryProductRepository();

    // Add test data
    customerRepo.add(Customer.create(new CustomerId('cust-1'), 'John'));
    productRepo.add(Product.create(new ProductId('prod-1'), 'Widget', Money.create(25, 'USD')));

    useCase = new CreateOrderUseCase(orderRepo, customerRepo, productRepo);
  });

  it('should create and save order', async () => {
    const input: CreateOrderInput = {
      customerId: 'cust-1',
      items: [{ productId: 'prod-1', quantity: 2 }]
    };

    const output = await useCase.execute(input);

    expect(output.orderId).toBeDefined();
    expect(output.total).toBe(50);
    expect(orderRepo.count()).toBe(1);
  });

  it('should throw when customer not found', async () => {
    const input: CreateOrderInput = {
      customerId: 'non-existent',
      items: [{ productId: 'prod-1', quantity: 1 }]
    };

    await expect(useCase.execute(input))
      .rejects
      .toThrow('Customer not found');
  });

  it('should throw when product not found', async () => {
    const input: CreateOrderInput = {
      customerId: 'cust-1',
      items: [{ productId: 'non-existent', quantity: 1 }]
    };

    await expect(useCase.execute(input))
      .rejects
      .toThrow('Product not found');
  });
});
```

### Integration Testing Controllers

```typescript
// ════════════════════════════════════════════════════════════════
// TESTING CONTROLLERS: Test HTTP conversion
// ════════════════════════════════════════════════════════════════

describe('OrderController', () => {
  let controller: OrderController;
  let mockUseCase: jest.Mocked<CreateOrderUseCase>;

  beforeEach(() => {
    mockUseCase = {
      execute: jest.fn()
    } as any;

    controller = new OrderController(mockUseCase);
  });

  it('should return 201 with order data', async () => {
    mockUseCase.execute.mockResolvedValue({
      orderId: 'ord-123',
      total: 50
    });

    const request: HttpRequest = {
      body: { customerId: 'cust-1', items: [{ productId: 'prod-1', quantity: 2 }] },
      params: {},
      query: {}
    };

    const response = await controller.create(request);

    expect(response.status).toBe(201);
    expect(response.body.orderId).toBe('ord-123');
  });

  it('should return 400 for invalid input', async () => {
    const request: HttpRequest = {
      body: { items: [] },  // Missing customerId
      params: {},
      query: {}
    };

    const response = await controller.create(request);

    expect(response.status).toBe(400);
    expect(response.body.errors).toContain('customerId is required');
  });

  it('should return 422 for domain errors', async () => {
    mockUseCase.execute.mockRejectedValue(
      new DomainError('Minimum order is $10')
    );

    const request: HttpRequest = {
      body: { customerId: 'cust-1', items: [{ productId: 'prod-1', quantity: 1 }] },
      params: {},
      query: {}
    };

    const response = await controller.create(request);

    expect(response.status).toBe(422);
    expect(response.body.message).toBe('Minimum order is $10');
  });
});
```

### Integration Testing Repositories

```typescript
// ════════════════════════════════════════════════════════════════
// TESTING REPOSITORIES: Test with real DB (test container)
// ════════════════════════════════════════════════════════════════

describe('PostgresOrderRepository', () => {
  let db: DatabaseConnection;
  let repository: PostgresOrderRepository;

  beforeAll(async () => {
    // Use test database or test container
    db = await createTestDatabase();
  });

  beforeEach(async () => {
    await db.query('TRUNCATE orders, order_lines CASCADE');
    repository = new PostgresOrderRepository(db);
  });

  afterAll(async () => {
    await db.close();
  });

  it('should save and retrieve order', async () => {
    // Arrange
    const order = Order.create(new CustomerId('cust-1'));
    order.addLine(new ProductId('prod-1'), 2, Money.create(10, 'USD'));
    order.place();

    // Act
    await repository.save(order);
    const retrieved = await repository.findById(order.id);

    // Assert
    expect(retrieved).not.toBeNull();
    expect(retrieved!.id.equals(order.id)).toBe(true);
    expect(retrieved!.status).toBe(OrderStatus.Placed);
    expect(retrieved!.lines).toHaveLength(1);
    expect(retrieved!.total().value).toBe(20);
  });

  it('should return null for non-existent order', async () => {
    const result = await repository.findById(new OrderId('non-existent'));
    expect(result).toBeNull();
  });

  it('should update existing order', async () => {
    // Arrange
    const order = Order.create(new CustomerId('cust-1'));
    order.addLine(new ProductId('prod-1'), 1, Money.create(20, 'USD'));
    order.place();
    await repository.save(order);

    // Act
    order.cancel();
    await repository.save(order);
    const retrieved = await repository.findById(order.id);

    // Assert
    expect(retrieved!.status).toBe(OrderStatus.Cancelled);
  });
});
```

---

## 8. Common Mistakes

### Mistake 1: Leaking Framework Details

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ WRONG: Domain knows about Express
// ════════════════════════════════════════════════════════════════

// In domain layer
import { Request } from 'express';  // ❌ Framework dependency!

class CreateOrderUseCase {
  execute(req: Request): void {  // ❌ Express type in use case!
    const customerId = req.body.customerId;
    const order = new Order(customerId);
    // ...
  }
}

// ════════════════════════════════════════════════════════════════
// ✅ RIGHT: Domain knows nothing about frameworks
// ════════════════════════════════════════════════════════════════

// Use case defines its own input type
interface CreateOrderInput {
  customerId: string;
  items: Array<{ productId: string; quantity: number }>;
}

class CreateOrderUseCase {
  execute(input: CreateOrderInput): void {  // ✅ Framework-agnostic
    const order = new Order(input.customerId);
    // ...
  }
}

// Controller (adapter layer) does the conversion
class OrderController {
  async create(req: Request): Promise<Response> {
    // Convert Express request to use case input
    const input: CreateOrderInput = {
      customerId: req.body.customerId,
      items: req.body.items
    };
    
    await this.useCase.execute(input);
  }
}
```

### Mistake 2: Entities Know About Persistence

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ WRONG: Entity knows about database
// ════════════════════════════════════════════════════════════════

import { Column, Entity, PrimaryColumn } from 'typeorm';  // ❌

@Entity('orders')  // ❌ ORM decorator
class Order {
  @PrimaryColumn()  // ❌ Database detail
  id: string;

  @Column()  // ❌ Persistence in domain!
  customerId: string;

  @Column({ type: 'json' })  // ❌ Database type
  items: OrderItem[];
}

// ════════════════════════════════════════════════════════════════
// ✅ RIGHT: Entity is pure, separate ORM entity for persistence
// ════════════════════════════════════════════════════════════════

// Domain entity - pure business logic
class Order {
  private constructor(
    private readonly _id: OrderId,
    private readonly _customerId: CustomerId,
    private _items: OrderItem[]
  ) {}

  static create(customerId: CustomerId): Order {
    return new Order(OrderId.generate(), customerId, []);
  }

  // Business methods...
}

// Separate ORM entity (in infrastructure layer)
@Entity('orders')
class OrderOrmEntity {
  @PrimaryColumn()
  id: string;

  @Column()
  customerId: string;

  @Column({ type: 'json' })
  items: any[];
}

// Mapper converts between them
class OrderMapper {
  toDomain(orm: OrderOrmEntity): Order {
    return Order.reconstitute({
      id: orm.id,
      customerId: orm.customerId,
      items: orm.items
    });
  }

  toPersistence(domain: Order): OrderOrmEntity {
    const snapshot = domain.toSnapshot();
    const orm = new OrderOrmEntity();
    orm.id = snapshot.id;
    orm.customerId = snapshot.customerId;
    orm.items = snapshot.items;
    return orm;
  }
}
```

### Mistake 3: Use Cases Returning Domain Objects

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ WRONG: Returning domain object across boundary
// ════════════════════════════════════════════════════════════════

class GetOrderUseCase {
  async execute(orderId: string): Promise<Order> {  // ❌ Domain object!
    return this.orderRepo.findById(new OrderId(orderId));
  }
}

// Controller now has access to domain internals
class OrderController {
  async get(req: Request): Promise<Response> {
    const order = await this.useCase.execute(req.params.id);
    
    // ❌ Controller can call domain methods!
    order.cancel();  // Bypass use case!
    
    return { body: order };  // ❌ Exposes domain structure
  }
}

// ════════════════════════════════════════════════════════════════
// ✅ RIGHT: Return DTO at boundaries
// ════════════════════════════════════════════════════════════════

interface GetOrderOutput {
  id: string;
  customerId: string;
  status: string;
  items: Array<{
    productId: string;
    quantity: number;
    price: number;
  }>;
  total: number;
}

class GetOrderUseCase {
  async execute(orderId: string): Promise<GetOrderOutput> {  // ✅ DTO
    const order = await this.orderRepo.findById(new OrderId(orderId));
    
    if (!order) throw new NotFoundError('Order');

    // Map domain to output DTO
    return {
      id: order.id.value,
      customerId: order.customerId.value,
      status: order.status,
      items: order.items.map(item => ({
        productId: item.productId.value,
        quantity: item.quantity,
        price: item.price.value
      })),
      total: order.total().value
    };
  }
}
```

### Mistake 4: Anemic Domain Model

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ WRONG: Entity is just a data container
// ════════════════════════════════════════════════════════════════

class Order {
  id: string;
  customerId: string;
  status: string;
  items: OrderItem[];
  total: number;
}

// All logic in use case - entity is "anemic"
class PlaceOrderUseCase {
  execute(order: Order): void {
    // Business rules scattered in use case
    if (order.items.length === 0) {
      throw new Error('Cannot place empty order');
    }
    
    order.total = order.items.reduce((sum, item) => sum + item.price, 0);
    
    if (order.total < 10) {
      throw new Error('Minimum order is $10');
    }
    
    order.status = 'placed';
  }
}

// ════════════════════════════════════════════════════════════════
// ✅ RIGHT: Entity contains behavior
// ════════════════════════════════════════════════════════════════

class Order {
  private _status: OrderStatus;
  private _items: OrderItem[];

  place(): void {
    // Business rules in the entity
    if (this._items.length === 0) {
      throw new DomainError('Cannot place empty order');
    }
    
    if (this.total().value < 10) {
      throw new DomainError('Minimum order is $10');
    }
    
    this._status = OrderStatus.Placed;
  }

  total(): Money {
    return this._items.reduce(
      (sum, item) => sum.add(item.subtotal()),
      Money.zero()
    );
  }
}

// Use case is thin - just orchestration
class PlaceOrderUseCase {
  execute(orderId: string): void {
    const order = this.orderRepo.findById(orderId);
    order.place();  // Entity knows how to place itself
    this.orderRepo.save(order);
  }
}
```

### Mistake 5: Over-Engineering Simple Apps

```typescript
// ════════════════════════════════════════════════════════════════
// ❌ WRONG: Full Clean Architecture for a CRUD todo app
// ════════════════════════════════════════════════════════════════

// 50+ files for a todo list:
// - TodoEntity, TodoId, TodoValueObject
// - CreateTodoUseCase, UpdateTodoUseCase, DeleteTodoUseCase, GetTodoUseCase
// - TodoRepository interface, PostgresTodoRepository
// - TodoController, TodoPresenter, TodoMapper
// - CreateTodoInput, CreateTodoOutput, TodoDTO
// - ... etc

// ════════════════════════════════════════════════════════════════
// ✅ RIGHT: Simple architecture for simple apps
// ════════════════════════════════════════════════════════════════

// For a simple CRUD todo app, this is fine:
const app = express();

app.get('/todos', async (req, res) => {
  const todos = await db.query('SELECT * FROM todos');
  res.json(todos);
});

app.post('/todos', async (req, res) => {
  const { title } = req.body;
  const todo = await db.query('INSERT INTO todos (title) VALUES ($1) RETURNING *', [title]);
  res.status(201).json(todo);
});

// Clean Architecture adds value when:
// - Complex business logic
// - Multiple entry points (API, CLI, events)
// - Need to swap infrastructure
// - Long-lived project with evolving requirements
```

---

## 9. When to Use / Not Use

### When TO Use Clean Architecture

```
✅ USE CLEAN ARCHITECTURE WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. COMPLEX BUSINESS LOGIC                                      │
│     └── Many business rules                                    │
│     └── Rules change frequently                                │
│     └── Need to protect rules from infrastructure changes     │
│                                                                  │
│  2. LONG-LIVED PROJECT                                          │
│     └── Will be maintained for years                           │
│     └── Team members will change                               │
│     └── Need clear structure for onboarding                   │
│                                                                  │
│  3. MULTIPLE ENTRY POINTS                                       │
│     └── REST API + GraphQL                                     │
│     └── CLI + Web                                              │
│     └── Message consumers + HTTP                               │
│                                                                  │
│  4. INFRASTRUCTURE UNCERTAINTY                                  │
│     └── Might change databases                                 │
│     └── Might change cloud providers                           │
│     └── Evaluating frameworks                                  │
│                                                                  │
│  5. TESTABILITY IS CRITICAL                                     │
│     └── Need fast unit tests                                   │
│     └── Need to test business logic in isolation              │
│     └── CI/CD pipeline requires fast tests                    │
│                                                                  │
│  6. TEAM SEPARATION                                             │
│     └── Backend team vs infrastructure team                    │
│     └── Different teams own different layers                  │
│                                                                  │
│  EXAMPLES:                                                      │
│  └── Banking / Financial systems                               │
│  └── E-commerce platforms                                      │
│  └── Healthcare systems                                        │
│  └── Enterprise applications                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When NOT to Use Clean Architecture

```
❌ DON'T USE CLEAN ARCHITECTURE WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. SIMPLE CRUD APPLICATION                                     │
│     └── Mostly data in, data out                               │
│     └── Little to no business logic                            │
│     └── Admin panels, content management                       │
│                                                                  │
│  2. PROTOTYPE / MVP                                             │
│     └── Need to ship fast                                      │
│     └── Will likely throw away and rewrite                    │
│     └── Validating an idea                                     │
│                                                                  │
│  3. SMALL TEAM / SOLO PROJECT                                   │
│     └── 1-3 developers                                         │
│     └── Everyone knows the codebase                           │
│     └── Overhead exceeds benefits                             │
│                                                                  │
│  4. FRAMEWORK-SPECIFIC APP                                      │
│     └── Will never change framework                           │
│     └── Deep integration with framework features              │
│     └── E.g., WordPress plugin                                │
│                                                                  │
│  5. SCRIPT / AUTOMATION                                         │
│     └── One-off scripts                                        │
│     └── CI/CD automation                                       │
│     └── Data migration tools                                  │
│                                                                  │
│  BETTER ALTERNATIVES:                                          │
│  └── Simple 3-layer architecture                              │
│  └── Transaction script pattern                                │
│  └── Active Record pattern                                     │
│  └── Framework conventions (Rails, Laravel style)            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. Interview Questions & Answers

### Basic Questions

**Q1: What is Clean Architecture?**
> **A:** Clean Architecture is a software design approach where business logic is at the center, independent of frameworks, databases, and UI. It's organized in concentric layers with the **Dependency Rule**: dependencies can only point inward. Inner layers define interfaces, outer layers implement them. This makes the application testable without infrastructure and allows swapping frameworks/databases easily.

**Q2: What are the layers in Clean Architecture?**
> **A:** From inside out:
> 1. **Entities** - Enterprise-wide business rules, domain objects
> 2. **Use Cases** - Application-specific business rules, orchestration
> 3. **Interface Adapters** - Controllers, repositories, gateways, presenters
> 4. **Frameworks & Drivers** - Express, PostgreSQL, React, external APIs
>
> The key is dependencies only point inward - entities know nothing about databases.

**Q3: What is the Dependency Rule?**
> **A:** Source code dependencies can only point inward. Nothing in an inner circle can know anything about something in an outer circle. This means:
> - Entities don't know about use cases
> - Use cases don't know about controllers or databases
> - Inner layers define interfaces (ports)
> - Outer layers implement those interfaces (adapters)

**Q4: What is Dependency Inversion?**
> **A:** The D in SOLID. High-level modules (use cases) shouldn't depend on low-level modules (repositories). Both should depend on abstractions (interfaces). In Clean Architecture:
> - Use case defines `OrderRepository` interface
> - Infrastructure implements `PostgresOrderRepository`
> - Use case doesn't know or care if it's Postgres, MongoDB, or in-memory

### Intermediate Questions

**Q5: How do you test in Clean Architecture?**
> **A:** 
> - **Entities**: Pure unit tests - no mocks needed, just test business logic
> - **Use Cases**: Unit tests with fake implementations of ports (in-memory repositories)
> - **Controllers**: Integration tests - verify HTTP conversion
> - **Repositories**: Integration tests with real database (test containers)
>
> The power is testing domain logic without any infrastructure - no database, no HTTP, just pure logic.

**Q6: What's the difference between Clean Architecture and traditional layered architecture?**
> **A:** 
> - **Traditional**: Presentation → Business → Data. Dependencies flow downward, business layer depends on data layer.
> - **Clean Architecture**: Dependencies point inward. Business layer defines interfaces, data layer implements them. The framework is a plugin, not the foundation.
>
> Clean Architecture inverts the dependency between business and data layers.

**Q7: How do you handle data crossing boundaries?**
> **A:** Using DTOs (Data Transfer Objects):
> - Use cases define their own input/output DTOs
> - Controllers convert HTTP requests to input DTOs
> - Use cases return output DTOs, not domain objects
> - Domain objects never cross boundaries
>
> This prevents external code from calling domain methods and protects encapsulation.

**Q8: Where does validation go?**
> **A:** Two types:
> 1. **Input validation** - In controllers/adapters. Format checks (valid email, required fields). Returns 400 Bad Request.
> 2. **Business validation** - In entities/domain. Business rules (minimum order amount, can't cancel shipped order). Throws domain errors.
>
> Don't mix them - they change for different reasons.

### Advanced Questions

**Q9: What is Ports and Adapters architecture?**
> **A:** Another name for Hexagonal Architecture, closely related to Clean Architecture. 
> - **Ports**: Interfaces defined by the application (OrderRepository, PaymentGateway)
> - **Adapters**: Implementations that connect to external systems (PostgresOrderRepository, StripePaymentGateway)
>
> The application core is surrounded by adapters that can be swapped without changing business logic.

**Q10: How does Clean Architecture relate to DDD?**
> **A:** They complement each other:
> - **Clean Architecture**: Focuses on dependency direction and layer separation
> - **DDD**: Focuses on modeling the domain with bounded contexts, aggregates, entities
>
> Typically, DDD tactical patterns (entities, value objects, aggregates) live in Clean Architecture's entity layer. Clean Architecture provides the structure, DDD provides the domain modeling approach.

**Q11: How do you handle transactions across use cases?**
> **A:** 
> - Each use case should be a single unit of work
> - Use Unit of Work pattern if multiple repositories need transactional consistency
> - For cross-aggregate consistency, use domain events and eventual consistency
> - Don't let transactions span multiple use cases - that's a design smell

**Q12: When would you NOT use Clean Architecture?**
> **A:** 
> - Simple CRUD apps with little business logic
> - Prototypes/MVPs where speed matters
> - Small teams where the overhead exceeds benefits
> - Framework-specific apps that won't change infrastructure
> - Scripts and automation tools
>
> Clean Architecture adds complexity. Only worth it when that complexity pays off in testability and flexibility.

### Scenario Questions

**Q13: You need to add email notifications when an order is placed. Where does this go?**
> **A:** 
> 1. Define `EmailService` interface in the use case layer (port)
> 2. Implement `SendGridEmailService` in infrastructure (adapter)
> 3. Inject `EmailService` into `PlaceOrderUseCase`
> 4. Call `emailService.sendOrderConfirmation()` after saving the order
>
> The use case knows it needs to send email but doesn't know about SendGrid.

**Q14: How would you structure an app with both REST API and GraphQL?**
> **A:** Same use cases, different adapters:
> 
> ```
> Use Cases (shared)
> ├── PlaceOrderUseCase
> └── GetOrderUseCase
>
> Adapters
> ├── REST
> │   └── OrderController (Express)
> └── GraphQL
>     └── OrderResolver
> ```
>
> Both controllers/resolvers call the same use cases. Business logic is written once.

---

## 🎓 Key Takeaways

1. **Dependency Rule** is everything - dependencies point inward only
2. **SOLID principles** guide the design, especially Dependency Inversion
3. **Entities** contain business rules, know nothing about infrastructure
4. **Use Cases** orchestrate domain objects, define ports for infrastructure
5. **DTOs cross boundaries**, not domain objects
6. **Testability** is a major benefit - test business logic without infrastructure
7. **Framework is a plugin** - Express, Postgres are details, not foundations
8. **Don't over-engineer** - Clean Architecture adds complexity, use when justified
9. **Interface Segregation** - many small interfaces, not one big one
10. **Screaming Architecture** - folder structure should reveal intent

---

## 📚 Resources

### Books
- **"Clean Architecture" by Robert C. Martin** - The definitive book
- **"Clean Code" by Robert C. Martin** - Foundation principles
- **"Implementing Domain-Driven Design" by Vaughn Vernon** - DDD + Clean Architecture

### Online
- [The Clean Architecture (Uncle Bob's blog)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
```
```
```


