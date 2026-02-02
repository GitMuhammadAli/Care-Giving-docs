# Dependency Injection - Complete Guide

> **MUST REMEMBER**: Dependency Injection (DI) is about passing dependencies to a class rather than having it create them. This enables loose coupling, easier testing (mock dependencies), and configurability. In Node.js, use constructor injection for most cases. DI containers (like tsyringe, InversifyJS, or NestJS) automate dependency resolution.

---

## How to Explain Like a Senior Developer

"Dependency injection is about inverting control - instead of a class creating its own database connection, you give it one. This seems trivial but has huge implications: you can swap implementations (test database vs real), mock dependencies for unit tests, and configure differently per environment. In Node.js, you don't always need a DI container - manual constructor injection works great for smaller apps. But NestJS's DI container shines in larger apps by automatically resolving dependency graphs and managing singletons. The key insight is that DI is fundamentally about making your code more modular and testable."

---

## Core Implementation

### Manual Dependency Injection

```typescript
// manual-di/types.ts

// Define interfaces (contracts)
export interface ILogger {
  info(message: string, meta?: object): void;
  error(message: string, meta?: object): void;
  warn(message: string, meta?: object): void;
}

export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  create(data: CreateUserData): Promise<User>;
  update(id: string, data: Partial<User>): Promise<User>;
  delete(id: string): Promise<void>;
}

export interface IEmailService {
  send(to: string, subject: string, body: string): Promise<void>;
}

export interface ICacheService {
  get<T>(key: string): Promise<T | null>;
  set<T>(key: string, value: T, ttl?: number): Promise<void>;
  delete(key: string): Promise<void>;
}

export interface User {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
}

export interface CreateUserData {
  email: string;
  name: string;
  password: string;
}
```

```typescript
// manual-di/implementations.ts
import { ILogger, IUserRepository, IEmailService, ICacheService, User, CreateUserData } from './types';

// Logger implementation
export class ConsoleLogger implements ILogger {
  private context: string;
  
  constructor(context: string) {
    this.context = context;
  }
  
  info(message: string, meta?: object): void {
    console.log(`[INFO] [${this.context}] ${message}`, meta || '');
  }
  
  error(message: string, meta?: object): void {
    console.error(`[ERROR] [${this.context}] ${message}`, meta || '');
  }
  
  warn(message: string, meta?: object): void {
    console.warn(`[WARN] [${this.context}] ${message}`, meta || '');
  }
}

// User repository implementation
export class PostgresUserRepository implements IUserRepository {
  constructor(
    private db: any, // Database client
    private logger: ILogger
  ) {}
  
  async findById(id: string): Promise<User | null> {
    this.logger.info('Finding user by id', { id });
    return this.db.query('SELECT * FROM users WHERE id = $1', [id]);
  }
  
  async findByEmail(email: string): Promise<User | null> {
    return this.db.query('SELECT * FROM users WHERE email = $1', [email]);
  }
  
  async create(data: CreateUserData): Promise<User> {
    this.logger.info('Creating user', { email: data.email });
    return this.db.query(
      'INSERT INTO users (email, name, password_hash) VALUES ($1, $2, $3) RETURNING *',
      [data.email, data.name, data.password]
    );
  }
  
  async update(id: string, data: Partial<User>): Promise<User> {
    // Update implementation
    return {} as User;
  }
  
  async delete(id: string): Promise<void> {
    await this.db.query('DELETE FROM users WHERE id = $1', [id]);
  }
}

// Email service implementation
export class SendGridEmailService implements IEmailService {
  constructor(
    private apiKey: string,
    private logger: ILogger
  ) {}
  
  async send(to: string, subject: string, body: string): Promise<void> {
    this.logger.info('Sending email', { to, subject });
    // SendGrid API call
  }
}

// Cache implementation
export class RedisCacheService implements ICacheService {
  constructor(private redis: any) {}
  
  async get<T>(key: string): Promise<T | null> {
    const value = await this.redis.get(key);
    return value ? JSON.parse(value) : null;
  }
  
  async set<T>(key: string, value: T, ttl?: number): Promise<void> {
    const serialized = JSON.stringify(value);
    if (ttl) {
      await this.redis.setex(key, ttl, serialized);
    } else {
      await this.redis.set(key, serialized);
    }
  }
  
  async delete(key: string): Promise<void> {
    await this.redis.del(key);
  }
}
```

```typescript
// manual-di/services.ts
import { ILogger, IUserRepository, IEmailService, ICacheService, User, CreateUserData } from './types';

export class UserService {
  constructor(
    private userRepository: IUserRepository,
    private emailService: IEmailService,
    private cacheService: ICacheService,
    private logger: ILogger
  ) {}
  
  async getUser(id: string): Promise<User | null> {
    // Try cache first
    const cached = await this.cacheService.get<User>(`user:${id}`);
    if (cached) {
      this.logger.info('User found in cache', { id });
      return cached;
    }
    
    // Fetch from database
    const user = await this.userRepository.findById(id);
    if (user) {
      await this.cacheService.set(`user:${id}`, user, 3600);
    }
    
    return user;
  }
  
  async createUser(data: CreateUserData): Promise<User> {
    // Check for existing user
    const existing = await this.userRepository.findByEmail(data.email);
    if (existing) {
      throw new Error('User already exists');
    }
    
    // Create user
    const user = await this.userRepository.create(data);
    
    // Send welcome email
    try {
      await this.emailService.send(
        user.email,
        'Welcome!',
        `Hello ${user.name}, welcome to our platform!`
      );
    } catch (error) {
      this.logger.error('Failed to send welcome email', { userId: user.id });
    }
    
    return user;
  }
}
```

```typescript
// manual-di/container.ts
import { ConsoleLogger, PostgresUserRepository, SendGridEmailService, RedisCacheService } from './implementations';
import { UserService } from './services';

// Simple container / composition root
export function createContainer(config: {
  db: any;
  redis: any;
  sendgridApiKey: string;
}) {
  // Create loggers
  const userLogger = new ConsoleLogger('UserService');
  const repoLogger = new ConsoleLogger('UserRepository');
  const emailLogger = new ConsoleLogger('EmailService');
  
  // Create implementations
  const userRepository = new PostgresUserRepository(config.db, repoLogger);
  const emailService = new SendGridEmailService(config.sendgridApiKey, emailLogger);
  const cacheService = new RedisCacheService(config.redis);
  
  // Create services
  const userService = new UserService(
    userRepository,
    emailService,
    cacheService,
    userLogger
  );
  
  return {
    userService,
    // Add more services as needed
  };
}

// Usage in application
// const container = createContainer({
//   db: pgPool,
//   redis: redisClient,
//   sendgridApiKey: process.env.SENDGRID_API_KEY!,
// });
// 
// app.get('/users/:id', async (req, res) => {
//   const user = await container.userService.getUser(req.params.id);
//   res.json(user);
// });
```

### Using tsyringe (Lightweight DI Container)

```typescript
// tsyringe-di/container.ts
import 'reflect-metadata';
import { container, injectable, inject } from 'tsyringe';

// Tokens for interfaces
const TOKENS = {
  Logger: Symbol('Logger'),
  UserRepository: Symbol('UserRepository'),
  EmailService: Symbol('EmailService'),
  CacheService: Symbol('CacheService'),
  DatabaseClient: Symbol('DatabaseClient'),
  RedisClient: Symbol('RedisClient'),
};

// Implementations with decorators
@injectable()
class ConsoleLogger implements ILogger {
  info(message: string, meta?: object): void {
    console.log(`[INFO] ${message}`, meta || '');
  }
  error(message: string, meta?: object): void {
    console.error(`[ERROR] ${message}`, meta || '');
  }
  warn(message: string, meta?: object): void {
    console.warn(`[WARN] ${message}`, meta || '');
  }
}

@injectable()
class PostgresUserRepository implements IUserRepository {
  constructor(
    @inject(TOKENS.DatabaseClient) private db: any,
    @inject(TOKENS.Logger) private logger: ILogger
  ) {}
  
  async findById(id: string): Promise<User | null> {
    this.logger.info('Finding user', { id });
    return null;
  }
  
  async findByEmail(email: string): Promise<User | null> {
    return null;
  }
  
  async create(data: CreateUserData): Promise<User> {
    return {} as User;
  }
  
  async update(id: string, data: Partial<User>): Promise<User> {
    return {} as User;
  }
  
  async delete(id: string): Promise<void> {}
}

@injectable()
class UserService {
  constructor(
    @inject(TOKENS.UserRepository) private userRepository: IUserRepository,
    @inject(TOKENS.EmailService) private emailService: IEmailService,
    @inject(TOKENS.Logger) private logger: ILogger
  ) {}
  
  async getUser(id: string): Promise<User | null> {
    return this.userRepository.findById(id);
  }
}

// Register dependencies
function setupContainer() {
  // Register singletons
  container.registerSingleton(TOKENS.Logger, ConsoleLogger);
  container.registerSingleton(TOKENS.UserRepository, PostgresUserRepository);
  
  // Register with factory
  container.register(TOKENS.DatabaseClient, {
    useFactory: () => {
      // Create and return database client
      return {};
    },
  });
  
  return container;
}

// Resolve dependencies
const userService = container.resolve(UserService);

import { ILogger, IUserRepository, IEmailService, User, CreateUserData } from './types';
```

### NestJS Dependency Injection

```typescript
// nestjs-di/user.module.ts
import { Module } from '@nestjs/common';
import { UserController } from './user.controller';
import { UserService } from './user.service';
import { UserRepository } from './user.repository';
import { EmailModule } from '../email/email.module';
import { CacheModule } from '../cache/cache.module';

@Module({
  imports: [EmailModule, CacheModule],
  controllers: [UserController],
  providers: [
    UserService,
    UserRepository,
    // Custom provider with factory
    {
      provide: 'USER_CONFIG',
      useFactory: () => ({
        maxLoginAttempts: 5,
        sessionDuration: 3600,
      }),
    },
  ],
  exports: [UserService],
})
export class UserModule {}
```

```typescript
// nestjs-di/user.service.ts
import { Injectable, Inject } from '@nestjs/common';
import { UserRepository } from './user.repository';
import { EmailService } from '../email/email.service';
import { CacheService } from '../cache/cache.service';

interface UserConfig {
  maxLoginAttempts: number;
  sessionDuration: number;
}

@Injectable()
export class UserService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly emailService: EmailService,
    private readonly cacheService: CacheService,
    @Inject('USER_CONFIG') private readonly config: UserConfig
  ) {}
  
  async findById(id: string) {
    // Try cache
    const cached = await this.cacheService.get(`user:${id}`);
    if (cached) return cached;
    
    // Fetch from DB
    const user = await this.userRepository.findById(id);
    if (user) {
      await this.cacheService.set(
        `user:${id}`,
        user,
        this.config.sessionDuration
      );
    }
    
    return user;
  }
}
```

```typescript
// nestjs-di/user.controller.ts
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { UserService } from './user.service';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}
  
  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.userService.findById(id);
  }
  
  @Post()
  async create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto);
  }
}
```

### Testing with Mocked Dependencies

```typescript
// testing/user.service.spec.ts
import { UserService } from './user.service';
import { IUserRepository, IEmailService, ICacheService, ILogger, User } from './types';

// Mock implementations
class MockUserRepository implements IUserRepository {
  private users: User[] = [];
  
  async findById(id: string): Promise<User | null> {
    return this.users.find(u => u.id === id) || null;
  }
  
  async findByEmail(email: string): Promise<User | null> {
    return this.users.find(u => u.email === email) || null;
  }
  
  async create(data: any): Promise<User> {
    const user: User = {
      id: `user_${Date.now()}`,
      email: data.email,
      name: data.name,
      createdAt: new Date(),
    };
    this.users.push(user);
    return user;
  }
  
  async update(id: string, data: Partial<User>): Promise<User> {
    const user = await this.findById(id);
    if (!user) throw new Error('Not found');
    Object.assign(user, data);
    return user;
  }
  
  async delete(id: string): Promise<void> {
    const index = this.users.findIndex(u => u.id === id);
    if (index > -1) this.users.splice(index, 1);
  }
  
  // Helper for tests
  _addUser(user: User): void {
    this.users.push(user);
  }
}

class MockEmailService implements IEmailService {
  public sentEmails: Array<{ to: string; subject: string; body: string }> = [];
  public shouldFail = false;
  
  async send(to: string, subject: string, body: string): Promise<void> {
    if (this.shouldFail) {
      throw new Error('Email service failed');
    }
    this.sentEmails.push({ to, subject, body });
  }
}

class MockCacheService implements ICacheService {
  private cache = new Map<string, string>();
  
  async get<T>(key: string): Promise<T | null> {
    const value = this.cache.get(key);
    return value ? JSON.parse(value) : null;
  }
  
  async set<T>(key: string, value: T): Promise<void> {
    this.cache.set(key, JSON.stringify(value));
  }
  
  async delete(key: string): Promise<void> {
    this.cache.delete(key);
  }
}

class MockLogger implements ILogger {
  public logs: Array<{ level: string; message: string; meta?: object }> = [];
  
  info(message: string, meta?: object): void {
    this.logs.push({ level: 'info', message, meta });
  }
  
  error(message: string, meta?: object): void {
    this.logs.push({ level: 'error', message, meta });
  }
  
  warn(message: string, meta?: object): void {
    this.logs.push({ level: 'warn', message, meta });
  }
}

// Tests
describe('UserService', () => {
  let userService: UserService;
  let mockUserRepo: MockUserRepository;
  let mockEmailService: MockEmailService;
  let mockCacheService: MockCacheService;
  let mockLogger: MockLogger;
  
  beforeEach(() => {
    mockUserRepo = new MockUserRepository();
    mockEmailService = new MockEmailService();
    mockCacheService = new MockCacheService();
    mockLogger = new MockLogger();
    
    userService = new UserService(
      mockUserRepo,
      mockEmailService,
      mockCacheService,
      mockLogger
    );
  });
  
  describe('getUser', () => {
    it('should return cached user if available', async () => {
      const user: User = {
        id: '123',
        email: 'test@example.com',
        name: 'Test User',
        createdAt: new Date(),
      };
      
      await mockCacheService.set('user:123', user);
      
      const result = await userService.getUser('123');
      
      expect(result).toEqual(user);
      expect(mockLogger.logs).toContainEqual(
        expect.objectContaining({ message: 'User found in cache' })
      );
    });
    
    it('should fetch from repository and cache if not in cache', async () => {
      const user: User = {
        id: '123',
        email: 'test@example.com',
        name: 'Test User',
        createdAt: new Date(),
      };
      
      mockUserRepo._addUser(user);
      
      const result = await userService.getUser('123');
      
      expect(result).toEqual(user);
      
      // Verify cached
      const cached = await mockCacheService.get('user:123');
      expect(cached).toEqual(user);
    });
  });
  
  describe('createUser', () => {
    it('should create user and send welcome email', async () => {
      const result = await userService.createUser({
        email: 'new@example.com',
        name: 'New User',
        password: 'password123',
      });
      
      expect(result.email).toBe('new@example.com');
      expect(mockEmailService.sentEmails).toHaveLength(1);
      expect(mockEmailService.sentEmails[0].to).toBe('new@example.com');
    });
    
    it('should log error but not fail if email service fails', async () => {
      mockEmailService.shouldFail = true;
      
      const result = await userService.createUser({
        email: 'new@example.com',
        name: 'New User',
        password: 'password123',
      });
      
      expect(result.email).toBe('new@example.com');
      expect(mockLogger.logs).toContainEqual(
        expect.objectContaining({
          level: 'error',
          message: 'Failed to send welcome email',
        })
      );
    });
  });
});
```

---

## Real-World Scenarios

### Scenario 1: Feature Flags with DI

```typescript
// feature-flags/feature-flag.service.ts
export interface IFeatureFlagService {
  isEnabled(flag: string, userId?: string): Promise<boolean>;
  getVariant(flag: string, userId?: string): Promise<string | null>;
}

// Production implementation using LaunchDarkly
@injectable()
export class LaunchDarklyFeatureFlagService implements IFeatureFlagService {
  private client: any;
  
  constructor(@inject('LAUNCHDARKLY_SDK_KEY') sdkKey: string) {
    // Initialize LaunchDarkly client
  }
  
  async isEnabled(flag: string, userId?: string): Promise<boolean> {
    return this.client.variation(flag, { key: userId || 'anonymous' }, false);
  }
  
  async getVariant(flag: string, userId?: string): Promise<string | null> {
    return this.client.variation(flag, { key: userId || 'anonymous' }, null);
  }
}

// Test implementation
@injectable()
export class MockFeatureFlagService implements IFeatureFlagService {
  private flags = new Map<string, boolean>();
  private variants = new Map<string, string>();
  
  async isEnabled(flag: string): Promise<boolean> {
    return this.flags.get(flag) ?? false;
  }
  
  async getVariant(flag: string): Promise<string | null> {
    return this.variants.get(flag) ?? null;
  }
  
  // Test helpers
  setFlag(flag: string, enabled: boolean): void {
    this.flags.set(flag, enabled);
  }
  
  setVariant(flag: string, variant: string): void {
    this.variants.set(flag, variant);
  }
}

// Service using feature flags
@injectable()
export class PaymentService {
  constructor(
    @inject('FeatureFlagService') private featureFlags: IFeatureFlagService,
    @inject('StripeClient') private stripeClient: any,
    @inject('NewPaymentProcessor') private newProcessor: any
  ) {}
  
  async processPayment(userId: string, amount: number): Promise<void> {
    const useNewProcessor = await this.featureFlags.isEnabled(
      'new-payment-processor',
      userId
    );
    
    if (useNewProcessor) {
      await this.newProcessor.process(userId, amount);
    } else {
      await this.stripeClient.charge(userId, amount);
    }
  }
}

import { injectable, inject } from 'tsyringe';
```

---

## Common Pitfalls

### 1. Creating Dependencies Inside Classes

```typescript
// ❌ BAD: Hard-coded dependency
class UserService {
  private logger = new ConsoleLogger(); // Can't replace for testing!
  private repo = new PostgresUserRepository(); // Tight coupling!
}

// ✅ GOOD: Injected dependencies
class UserService {
  constructor(
    private logger: ILogger,
    private repo: IUserRepository
  ) {}
}
```

### 2. Service Locator Anti-Pattern

```typescript
// ❌ BAD: Service locator (hidden dependencies)
class UserService {
  async getUser(id: string) {
    const logger = container.resolve<ILogger>('Logger');
    const repo = container.resolve<IUserRepository>('UserRepo');
    // Dependencies are hidden, hard to test
  }
}

// ✅ GOOD: Constructor injection (explicit dependencies)
class UserService {
  constructor(
    private logger: ILogger,
    private repo: IUserRepository
  ) {}
  
  async getUser(id: string) {
    // Dependencies are clear
  }
}
```

### 3. Circular Dependencies

```typescript
// ❌ BAD: Circular dependency
class ServiceA {
  constructor(private serviceB: ServiceB) {}
}

class ServiceB {
  constructor(private serviceA: ServiceA) {} // Circular!
}

// ✅ GOOD: Break cycle with interface or events
class ServiceA {
  constructor(private serviceB: IServiceB) {}
}

class ServiceB {
  constructor(private eventEmitter: IEventEmitter) {}
  
  notifyA() {
    this.eventEmitter.emit('serviceB:event', data);
  }
}
```

---

## Interview Questions

### Q1: What's the difference between DI and IoC?

**A:** Inversion of Control (IoC) is the principle - instead of a class controlling its dependencies, control is inverted to an external entity. Dependency Injection is a pattern that implements IoC by providing dependencies from outside. Other IoC patterns include Service Locator and Factory Pattern.

### Q2: When would you NOT use a DI container?

**A:** For small applications, manual constructor injection is simpler and more explicit. DI containers add complexity and can make debugging harder (magic resolution). Use containers when you have many interdependent services, need automatic lifecycle management, or want to swap implementations easily.

### Q3: What are the three types of dependency injection?

**A:** 
1. **Constructor injection**: Dependencies passed via constructor (preferred)
2. **Setter injection**: Dependencies set via setter methods (optional deps)
3. **Interface injection**: Dependency provides injector method (rare in JS)
Constructor injection is best because dependencies are explicit and required.

### Q4: How does DI improve testability?

**A:** DI allows replacing real implementations with mocks/stubs. Instead of a class creating its own database connection, you inject it - in tests, inject a mock that returns test data. This isolates the unit being tested and makes tests fast (no real DB calls).

---

## Quick Reference Checklist

### DI Principles
- [ ] Depend on abstractions (interfaces), not implementations
- [ ] Pass dependencies via constructor
- [ ] Single responsibility - each class does one thing
- [ ] Composition root - wire dependencies in one place

### Container Usage
- [ ] Register interfaces with implementations
- [ ] Use appropriate scopes (singleton, transient, scoped)
- [ ] Avoid service locator pattern
- [ ] Handle circular dependencies

### Testing
- [ ] Create mock implementations for testing
- [ ] Inject mocks via constructor
- [ ] Make mocks configurable (fail modes, return values)
- [ ] Verify interactions on mocks

### Best Practices
- [ ] Keep dependency graphs shallow
- [ ] Avoid injecting the container itself
- [ ] Use factories for runtime-determined dependencies
- [ ] Document which implementation is used where

---

*Last updated: February 2026*

