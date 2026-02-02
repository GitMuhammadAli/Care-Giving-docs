# ORM Patterns - Complete Guide

> **MUST REMEMBER**: ORMs abstract database interactions into objects. Two main patterns: Active Record (model contains data AND database operations - Rails, Eloquent) vs Data Mapper (entities are plain objects, separate repository handles persistence - TypeORM, Prisma). Choose based on complexity: Active Record for simple CRUD, Data Mapper for complex domain logic.

---

## How to Explain Like a Senior Developer

"ORMs solve the object-relational impedance mismatch - databases think in tables and rows, your code thinks in objects. Active Record is simpler - your User model has save(), find(), delete() methods. Data Mapper separates concerns - User is just data, UserRepository handles database operations. Prisma is my go-to because it generates type-safe clients and handles migrations well. But remember: ORMs aren't magic. You still need to understand N+1 queries, eager vs lazy loading, and when to drop down to raw SQL for complex operations."

---

## Core Implementation

### Active Record Pattern

```typescript
// active-record/base.ts

// Base Active Record class
abstract class ActiveRecord {
  id?: number;
  createdAt?: Date;
  updatedAt?: Date;
  
  protected static tableName: string;
  protected static db: any; // Database client
  
  // Save (insert or update)
  async save(): Promise<this> {
    const constructor = this.constructor as typeof ActiveRecord;
    const data = this.toJSON();
    
    if (this.id) {
      // Update
      delete data.id;
      data.updatedAt = new Date();
      
      const result = await constructor.db.query(
        `UPDATE ${constructor.tableName} SET ${
          Object.keys(data).map((k, i) => `${k} = $${i + 1}`).join(', ')
        } WHERE id = $${Object.keys(data).length + 1} RETURNING *`,
        [...Object.values(data), this.id]
      );
      
      return Object.assign(this, result.rows[0]);
    } else {
      // Insert
      data.createdAt = new Date();
      data.updatedAt = new Date();
      
      const keys = Object.keys(data);
      const result = await constructor.db.query(
        `INSERT INTO ${constructor.tableName} (${keys.join(', ')}) 
         VALUES (${keys.map((_, i) => `$${i + 1}`).join(', ')}) 
         RETURNING *`,
        Object.values(data)
      );
      
      return Object.assign(this, result.rows[0]);
    }
  }
  
  // Delete
  async delete(): Promise<void> {
    if (!this.id) throw new Error('Cannot delete unsaved record');
    
    const constructor = this.constructor as typeof ActiveRecord;
    await constructor.db.query(
      `DELETE FROM ${constructor.tableName} WHERE id = $1`,
      [this.id]
    );
  }
  
  // Find by ID
  static async find<T extends ActiveRecord>(
    this: new () => T,
    id: number
  ): Promise<T | null> {
    const constructor = this as unknown as typeof ActiveRecord;
    const result = await constructor.db.query(
      `SELECT * FROM ${constructor.tableName} WHERE id = $1`,
      [id]
    );
    
    if (result.rows.length === 0) return null;
    
    const instance = new this();
    return Object.assign(instance, result.rows[0]);
  }
  
  // Find all
  static async all<T extends ActiveRecord>(
    this: new () => T
  ): Promise<T[]> {
    const constructor = this as unknown as typeof ActiveRecord;
    const result = await constructor.db.query(
      `SELECT * FROM ${constructor.tableName}`
    );
    
    return result.rows.map((row: any) => {
      const instance = new this();
      return Object.assign(instance, row);
    });
  }
  
  // Find by conditions
  static async where<T extends ActiveRecord>(
    this: new () => T,
    conditions: Partial<T>
  ): Promise<T[]> {
    const constructor = this as unknown as typeof ActiveRecord;
    const keys = Object.keys(conditions);
    const values = Object.values(conditions);
    
    const whereClause = keys
      .map((k, i) => `${k} = $${i + 1}`)
      .join(' AND ');
    
    const result = await constructor.db.query(
      `SELECT * FROM ${constructor.tableName} WHERE ${whereClause}`,
      values
    );
    
    return result.rows.map((row: any) => {
      const instance = new this();
      return Object.assign(instance, row);
    });
  }
  
  protected toJSON(): Record<string, unknown> {
    const data: Record<string, unknown> = {};
    for (const [key, value] of Object.entries(this)) {
      if (value !== undefined) {
        data[key] = value;
      }
    }
    return data;
  }
}

// User model using Active Record
class User extends ActiveRecord {
  protected static tableName = 'users';
  
  id?: number;
  email!: string;
  name!: string;
  passwordHash?: string;
  role: 'user' | 'admin' = 'user';
  
  // Business logic in the model
  isAdmin(): boolean {
    return this.role === 'admin';
  }
  
  async setPassword(password: string): Promise<void> {
    this.passwordHash = await hashPassword(password);
  }
  
  async verifyPassword(password: string): Promise<boolean> {
    if (!this.passwordHash) return false;
    return verifyPassword(password, this.passwordHash);
  }
  
  // Custom query methods
  static async findByEmail(email: string): Promise<User | null> {
    const users = await User.where<User>({ email } as Partial<User>);
    return users[0] || null;
  }
}

// Usage
async function activeRecordExample() {
  // Create
  const user = new User();
  user.email = 'john@example.com';
  user.name = 'John Doe';
  await user.setPassword('password123');
  await user.save();
  
  console.log('Created user:', user.id);
  
  // Find
  const found = await User.find(user.id!);
  console.log('Found:', found?.name);
  
  // Update
  found!.name = 'John Smith';
  await found!.save();
  
  // Query
  const admins = await User.where<User>({ role: 'admin' } as Partial<User>);
  console.log('Admins:', admins.length);
  
  // Delete
  await found!.delete();
}

async function hashPassword(password: string): Promise<string> {
  return password;
}

async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return password === hash;
}
```

### Data Mapper Pattern

```typescript
// data-mapper/entities.ts

// Pure domain entities - no database logic
class User {
  constructor(
    public id: string,
    public email: string,
    public name: string,
    public role: 'user' | 'admin',
    public createdAt: Date,
    public updatedAt: Date
  ) {}
  
  // Only business logic, no persistence
  isAdmin(): boolean {
    return this.role === 'admin';
  }
  
  canAccessResource(resource: Resource): boolean {
    if (this.isAdmin()) return true;
    return resource.ownerId === this.id;
  }
}

class Resource {
  constructor(
    public id: string,
    public ownerId: string,
    public name: string
  ) {}
}

// data-mapper/repositories.ts

// Repository interface
interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  findAll(options?: { limit?: number; offset?: number }): Promise<User[]>;
  save(user: User): Promise<User>;
  delete(id: string): Promise<void>;
}

// PostgreSQL implementation
class PostgresUserRepository implements IUserRepository {
  constructor(private db: any) {}
  
  async findById(id: string): Promise<User | null> {
    const result = await this.db.query(
      'SELECT * FROM users WHERE id = $1',
      [id]
    );
    
    if (result.rows.length === 0) return null;
    return this.mapToEntity(result.rows[0]);
  }
  
  async findByEmail(email: string): Promise<User | null> {
    const result = await this.db.query(
      'SELECT * FROM users WHERE email = $1',
      [email]
    );
    
    if (result.rows.length === 0) return null;
    return this.mapToEntity(result.rows[0]);
  }
  
  async findAll(options: { limit?: number; offset?: number } = {}): Promise<User[]> {
    const { limit = 100, offset = 0 } = options;
    
    const result = await this.db.query(
      'SELECT * FROM users ORDER BY created_at DESC LIMIT $1 OFFSET $2',
      [limit, offset]
    );
    
    return result.rows.map(this.mapToEntity);
  }
  
  async save(user: User): Promise<User> {
    const existing = await this.findById(user.id);
    
    if (existing) {
      // Update
      const result = await this.db.query(
        `UPDATE users SET 
         email = $1, name = $2, role = $3, updated_at = NOW()
         WHERE id = $4 RETURNING *`,
        [user.email, user.name, user.role, user.id]
      );
      return this.mapToEntity(result.rows[0]);
    } else {
      // Insert
      const result = await this.db.query(
        `INSERT INTO users (id, email, name, role, created_at, updated_at)
         VALUES ($1, $2, $3, $4, NOW(), NOW()) RETURNING *`,
        [user.id, user.email, user.name, user.role]
      );
      return this.mapToEntity(result.rows[0]);
    }
  }
  
  async delete(id: string): Promise<void> {
    await this.db.query('DELETE FROM users WHERE id = $1', [id]);
  }
  
  // Map database row to domain entity
  private mapToEntity(row: any): User {
    return new User(
      row.id,
      row.email,
      row.name,
      row.role,
      row.created_at,
      row.updated_at
    );
  }
}

// Service layer using repository
class UserService {
  constructor(private userRepository: IUserRepository) {}
  
  async getUser(id: string): Promise<User | null> {
    return this.userRepository.findById(id);
  }
  
  async createUser(data: {
    email: string;
    name: string;
    role?: 'user' | 'admin';
  }): Promise<User> {
    // Check for existing
    const existing = await this.userRepository.findByEmail(data.email);
    if (existing) {
      throw new Error('User already exists');
    }
    
    // Create domain entity
    const user = new User(
      generateId(),
      data.email,
      data.name,
      data.role || 'user',
      new Date(),
      new Date()
    );
    
    // Persist
    return this.userRepository.save(user);
  }
}

function generateId(): string {
  return Math.random().toString(36).substring(2);
}
```

### Prisma (Modern Data Mapper)

```typescript
// prisma/schema.prisma
/*
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  role      Role     @default(USER)
  posts     Post[]
  profile   Profile?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Profile {
  id     String  @id @default(cuid())
  bio    String?
  avatar String?
  user   User    @relation(fields: [userId], references: [id])
  userId String  @unique
}

model Post {
  id        String     @id @default(cuid())
  title     String
  content   String?
  published Boolean    @default(false)
  author    User       @relation(fields: [authorId], references: [id])
  authorId  String
  categories Category[]
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt
}

model Category {
  id    String @id @default(cuid())
  name  String @unique
  posts Post[]
}

enum Role {
  USER
  ADMIN
}
*/

// prisma/usage.ts
import { PrismaClient, Prisma } from '@prisma/client';

const prisma = new PrismaClient({
  log: ['query', 'info', 'warn', 'error'],
});

// Type-safe queries
async function prismaExamples() {
  // Create with relations
  const user = await prisma.user.create({
    data: {
      email: 'alice@example.com',
      name: 'Alice',
      role: 'ADMIN',
      profile: {
        create: {
          bio: 'Software developer',
          avatar: 'https://example.com/avatar.jpg',
        },
      },
      posts: {
        create: [
          {
            title: 'First Post',
            content: 'Hello world!',
            published: true,
            categories: {
              connectOrCreate: {
                where: { name: 'Technology' },
                create: { name: 'Technology' },
              },
            },
          },
        ],
      },
    },
    include: {
      profile: true,
      posts: true,
    },
  });
  
  // Type-safe select (only returns selected fields)
  const userEmail = await prisma.user.findUnique({
    where: { id: user.id },
    select: {
      email: true,
      name: true,
      _count: {
        select: { posts: true },
      },
    },
  });
  // userEmail is typed as { email: string; name: string; _count: { posts: number } }
  
  // Complex filtering
  const publishedPosts = await prisma.post.findMany({
    where: {
      published: true,
      author: {
        role: 'ADMIN',
      },
      categories: {
        some: {
          name: 'Technology',
        },
      },
    },
    orderBy: {
      createdAt: 'desc',
    },
    take: 10,
    include: {
      author: {
        select: { name: true, email: true },
      },
      categories: true,
    },
  });
  
  // Transactions
  const [newUser, newPost] = await prisma.$transaction([
    prisma.user.create({
      data: { email: 'bob@example.com', name: 'Bob' },
    }),
    prisma.post.create({
      data: {
        title: 'Transaction Post',
        authorId: user.id,
      },
    }),
  ]);
  
  // Interactive transaction
  await prisma.$transaction(async (tx) => {
    const author = await tx.user.findUnique({
      where: { email: 'alice@example.com' },
    });
    
    if (!author) throw new Error('Author not found');
    
    await tx.post.updateMany({
      where: { authorId: author.id },
      data: { published: true },
    });
  });
  
  // Aggregations
  const stats = await prisma.post.aggregate({
    _count: true,
    _avg: {
      // If you had a numeric field
    },
    where: {
      published: true,
    },
  });
  
  // Group by
  const postsByAuthor = await prisma.post.groupBy({
    by: ['authorId'],
    _count: {
      id: true,
    },
    having: {
      id: {
        _count: {
          gt: 5,
        },
      },
    },
  });
  
  // Raw queries when needed
  const result = await prisma.$queryRaw<{ count: bigint }[]>`
    SELECT COUNT(*) as count FROM "Post" WHERE published = true
  `;
  
  return { user, publishedPosts, stats };
}

// Repository pattern with Prisma
class PrismaUserRepository {
  constructor(private prisma: PrismaClient) {}
  
  async findById(id: string) {
    return this.prisma.user.findUnique({
      where: { id },
      include: { profile: true },
    });
  }
  
  async findByEmail(email: string) {
    return this.prisma.user.findUnique({
      where: { email },
    });
  }
  
  async create(data: Prisma.UserCreateInput) {
    return this.prisma.user.create({
      data,
      include: { profile: true },
    });
  }
  
  async update(id: string, data: Prisma.UserUpdateInput) {
    return this.prisma.user.update({
      where: { id },
      data,
    });
  }
  
  async delete(id: string) {
    return this.prisma.user.delete({
      where: { id },
    });
  }
  
  async findWithPosts(id: string) {
    return this.prisma.user.findUnique({
      where: { id },
      include: {
        posts: {
          where: { published: true },
          orderBy: { createdAt: 'desc' },
          take: 10,
        },
      },
    });
  }
}
```

### TypeORM (Active Record + Data Mapper)

```typescript
// typeorm/entities.ts
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  OneToMany,
  ManyToOne,
  ManyToMany,
  JoinTable,
  BaseEntity,
  Repository,
} from 'typeorm';

// Active Record style
@Entity('users')
export class User extends BaseEntity {
  @PrimaryGeneratedColumn('uuid')
  id!: string;
  
  @Column({ unique: true })
  email!: string;
  
  @Column()
  name!: string;
  
  @Column({ type: 'enum', enum: ['user', 'admin'], default: 'user' })
  role!: 'user' | 'admin';
  
  @OneToMany(() => Post, (post) => post.author)
  posts!: Post[];
  
  @CreateDateColumn()
  createdAt!: Date;
  
  @UpdateDateColumn()
  updatedAt!: Date;
  
  // Instance methods
  isAdmin(): boolean {
    return this.role === 'admin';
  }
  
  // Active Record methods (inherited from BaseEntity)
  // user.save(), User.find(), User.findOne(), etc.
}

@Entity('posts')
export class Post extends BaseEntity {
  @PrimaryGeneratedColumn('uuid')
  id!: string;
  
  @Column()
  title!: string;
  
  @Column({ type: 'text', nullable: true })
  content?: string;
  
  @Column({ default: false })
  published!: boolean;
  
  @ManyToOne(() => User, (user) => user.posts)
  author!: User;
  
  @Column()
  authorId!: string;
  
  @ManyToMany(() => Category)
  @JoinTable()
  categories!: Category[];
  
  @CreateDateColumn()
  createdAt!: Date;
  
  @UpdateDateColumn()
  updatedAt!: Date;
}

@Entity('categories')
export class Category extends BaseEntity {
  @PrimaryGeneratedColumn('uuid')
  id!: string;
  
  @Column({ unique: true })
  name!: string;
}

// Active Record usage
async function typeormActiveRecordExample() {
  // Create
  const user = new User();
  user.email = 'john@example.com';
  user.name = 'John';
  await user.save();
  
  // Find
  const found = await User.findOne({
    where: { email: 'john@example.com' },
    relations: ['posts'],
  });
  
  // Query builder
  const admins = await User.createQueryBuilder('user')
    .where('user.role = :role', { role: 'admin' })
    .leftJoinAndSelect('user.posts', 'post')
    .getMany();
  
  return { user, found, admins };
}

// Data Mapper style (using repositories)
import { DataSource } from 'typeorm';

async function typeormDataMapperExample(dataSource: DataSource) {
  const userRepository = dataSource.getRepository(User);
  const postRepository = dataSource.getRepository(Post);
  
  // Create
  const user = userRepository.create({
    email: 'jane@example.com',
    name: 'Jane',
    role: 'admin',
  });
  await userRepository.save(user);
  
  // Complex queries
  const usersWithPostCount = await userRepository
    .createQueryBuilder('user')
    .loadRelationCountAndMap('user.postCount', 'user.posts')
    .getMany();
  
  // Custom repository
  const customUserRepo = dataSource.getRepository(User).extend({
    findByEmailDomain(domain: string) {
      return this.createQueryBuilder('user')
        .where('user.email LIKE :domain', { domain: `%@${domain}` })
        .getMany();
    },
    
    async findActiveWithPosts() {
      return this.find({
        where: {
          posts: {
            published: true,
          },
        },
        relations: ['posts'],
      });
    },
  });
  
  const gmailUsers = await customUserRepo.findByEmailDomain('gmail.com');
  
  return { user, usersWithPostCount, gmailUsers };
}
```

---

## Real-World Scenarios

### Scenario 1: Soft Deletes and Audit Logging

```typescript
// soft-delete/model.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient().$extends({
  query: {
    $allModels: {
      // Override delete to soft delete
      async delete({ model, operation, args, query }) {
        return (prisma as any)[model].update({
          ...args,
          data: { deletedAt: new Date() },
        });
      },
      
      // Filter out soft-deleted by default
      async findMany({ model, operation, args, query }) {
        args.where = {
          ...args.where,
          deletedAt: null,
        };
        return query(args);
      },
      
      async findFirst({ model, operation, args, query }) {
        args.where = {
          ...args.where,
          deletedAt: null,
        };
        return query(args);
      },
    },
  },
});

// Audit logging middleware
prisma.$use(async (params, next) => {
  const before = Date.now();
  const result = await next(params);
  const after = Date.now();
  
  if (['create', 'update', 'delete'].includes(params.action)) {
    await prisma.auditLog.create({
      data: {
        model: params.model!,
        action: params.action,
        args: JSON.stringify(params.args),
        duration: after - before,
        userId: getCurrentUserId(), // From context
      },
    });
  }
  
  return result;
});

function getCurrentUserId(): string {
  return 'system';
}
```

### Scenario 2: N+1 Query Prevention

```typescript
// n-plus-one/prevention.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// ❌ BAD: N+1 query problem
async function badExample() {
  const posts = await prisma.post.findMany();
  
  // N additional queries!
  for (const post of posts) {
    const author = await prisma.user.findUnique({
      where: { id: post.authorId },
    });
    console.log(`${post.title} by ${author?.name}`);
  }
}

// ✅ GOOD: Eager loading with include
async function goodExample() {
  const posts = await prisma.post.findMany({
    include: {
      author: true, // Single JOIN query
    },
  });
  
  for (const post of posts) {
    console.log(`${post.title} by ${post.author.name}`);
  }
}

// ✅ GOOD: Select only needed fields
async function optimizedExample() {
  const posts = await prisma.post.findMany({
    select: {
      title: true,
      author: {
        select: {
          name: true,
        },
      },
    },
  });
  
  // Minimal data transfer
  for (const post of posts) {
    console.log(`${post.title} by ${post.author.name}`);
  }
}

// DataLoader pattern for GraphQL
import DataLoader from 'dataloader';

function createUserLoader() {
  return new DataLoader<string, any>(async (userIds) => {
    const users = await prisma.user.findMany({
      where: { id: { in: [...userIds] } },
    });
    
    const userMap = new Map(users.map(u => [u.id, u]));
    return userIds.map(id => userMap.get(id));
  });
}

// Usage in resolvers
const userLoader = createUserLoader();

async function resolvePostAuthor(post: { authorId: string }) {
  return userLoader.load(post.authorId); // Batched!
}
```

---

## Common Pitfalls

### 1. Over-fetching with Eager Loading

```typescript
// ❌ BAD: Loading everything
const user = await prisma.user.findUnique({
  where: { id },
  include: {
    posts: true,      // 1000 posts!
    comments: true,   // 5000 comments!
    followers: true,  // 10000 followers!
  },
});

// ✅ GOOD: Select what you need
const user = await prisma.user.findUnique({
  where: { id },
  select: {
    id: true,
    name: true,
    _count: {
      select: {
        posts: true,
        followers: true,
      },
    },
    posts: {
      take: 10,
      orderBy: { createdAt: 'desc' },
      select: { id: true, title: true },
    },
  },
});
```

### 2. Missing Indexes

```typescript
// ❌ BAD: Querying without index
const users = await prisma.user.findMany({
  where: {
    lastLoginAt: {
      gte: new Date('2024-01-01'),
    },
  },
});

// ✅ GOOD: Add index in schema
/*
model User {
  lastLoginAt DateTime?
  
  @@index([lastLoginAt])
}
*/
```

### 3. Leaking Database Concerns to Domain

```typescript
// ❌ BAD: Prisma types in domain layer
class UserService {
  async getUser(id: string): Promise<Prisma.User> {
    return this.prisma.user.findUnique({ where: { id } });
  }
}

// ✅ GOOD: Map to domain types
interface DomainUser {
  id: string;
  email: string;
  name: string;
}

class UserService {
  async getUser(id: string): Promise<DomainUser | null> {
    const dbUser = await this.prisma.user.findUnique({
      where: { id },
      select: { id: true, email: true, name: true },
    });
    
    return dbUser ? this.mapToDomain(dbUser) : null;
  }
  
  private mapToDomain(dbUser: { id: string; email: string; name: string }): DomainUser {
    return {
      id: dbUser.id,
      email: dbUser.email,
      name: dbUser.name,
    };
  }
}

import { Prisma, PrismaClient } from '@prisma/client';
```

---

## Interview Questions

### Q1: What's the difference between Active Record and Data Mapper?

**A:** Active Record combines data and database operations in the same class (User.save(), User.find()). Simple but couples domain to persistence. Data Mapper separates them - User is just data, UserRepository handles database operations. More complex but better separation of concerns, easier testing, and domain logic isn't tied to database structure.

### Q2: How do you prevent N+1 queries?

**A:** 
1. **Eager loading**: Include related data in initial query (Prisma's include)
2. **DataLoader**: Batch multiple lookups into single query (for GraphQL)
3. **Select specific fields**: Don't fetch more than needed
4. **Query analysis**: Use database query logs to identify N+1

### Q3: When would you use raw SQL over an ORM?

**A:** 
1. Complex queries ORMs struggle with (recursive CTEs, window functions)
2. Performance-critical queries that need optimization
3. Database-specific features not supported by ORM
4. Bulk operations where ORM overhead matters
5. Complex reporting/analytics queries

### Q4: How do you handle migrations in production?

**A:** 
1. Generate migrations during development (prisma migrate dev)
2. Test migrations on staging with production-like data
3. Apply migrations in CI/CD before deployment
4. For large tables, consider online schema changes (pt-online-schema-change)
5. Always have a rollback plan

---

## Quick Reference Checklist

### ORM Selection
- [ ] Active Record: Simple CRUD, rapid development
- [ ] Data Mapper: Complex domains, testability focus
- [ ] Prisma: Type safety priority, modern Node.js
- [ ] TypeORM: Flexibility, both patterns supported

### Query Optimization
- [ ] Use eager loading for known relations
- [ ] Select only needed fields
- [ ] Add indexes for frequently queried columns
- [ ] Monitor and analyze slow queries

### Best Practices
- [ ] Keep domain logic separate from persistence
- [ ] Use transactions for related operations
- [ ] Implement soft deletes where appropriate
- [ ] Version your migrations

### Testing
- [ ] Use in-memory database for unit tests
- [ ] Test with real database for integration
- [ ] Reset database state between tests
- [ ] Mock repositories when testing services

---

*Last updated: February 2026*

