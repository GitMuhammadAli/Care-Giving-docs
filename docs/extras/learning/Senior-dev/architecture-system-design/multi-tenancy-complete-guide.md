# 🏢 Multi-Tenancy Complete Guide

> A comprehensive guide to Multi-tenancy - data isolation strategies, tenant management, scaling patterns, and building SaaS applications that serve multiple customers from a single codebase.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Multi-tenancy is an architecture where a single instance of software serves multiple customers (tenants), with each tenant's data isolated and invisible to others, while sharing compute resources for cost efficiency."

### The 3 Isolation Models (Memorize!)
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. DATABASE PER TENANT (Siloed)                                │
│     └── Each tenant gets own database                          │
│     └── Strongest isolation, highest cost                      │
│     └── Best for: Compliance, enterprise, regulated industries │
│                                                                  │
│  2. SCHEMA PER TENANT (Bridge)                                  │
│     └── Shared database, separate schemas                      │
│     └── Good isolation, moderate cost                          │
│     └── Best for: Mid-market, moderate data volumes            │
│                                                                  │
│  3. SHARED SCHEMA (Pool)                                        │
│     └── All tenants in same tables with tenant_id column       │
│     └── Weakest isolation, lowest cost                         │
│     └── Best for: High volume, small tenants, startups         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Tenant isolation"** | "We ensure tenant isolation at the database level with RLS" |
| **"Noisy neighbor"** | "Rate limiting prevents noisy neighbor problems" |
| **"Row-Level Security"** | "PostgreSQL RLS automatically filters queries by tenant" |
| **"Tenant context"** | "Middleware extracts tenant context from JWT or subdomain" |
| **"Pool vs Silo"** | "We use pool model for small tenants, silo for enterprise" |
| **"Tenant sharding"** | "Large tenants are sharded to dedicated infrastructure" |
| **"Data residency"** | "EU tenants' data stays in EU region for GDPR compliance" |

### The Trade-offs Triangle
```
                    ISOLATION
                       /\
                      /  \
                     /    \
                    /      \
                   /   ✓    \
                  /          \
                 /            \
                /______________\
            COST            COMPLEXITY

Pool (Shared):     Low cost, Low isolation, Low complexity
Bridge (Schema):   Medium all
Silo (Database):   High cost, High isolation, High complexity
```

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Tenant ID column | **indexed** | Every query filters by tenant |
| Connection pool | **per tenant** | Prevent cross-tenant resource contention |
| Rate limit | **per tenant** | Prevent noisy neighbor |
| Backup | **per tenant** | Enterprise needs individual restore |
| Schema migrations | **all at once** | Can't have version drift |

### The "Wow" Statement (Memorize This!)
> "Our SaaS uses a hybrid multi-tenancy model: small tenants share a database with Row-Level Security for isolation, while enterprise tenants get dedicated databases for compliance. Tenant context is resolved from the JWT at the middleware level and flows through the request. Every query automatically filters by tenant_id using Prisma middleware - developers can't accidentally leak data. For noisy neighbor protection, we rate limit per tenant and use separate connection pools. Enterprise tenants can also choose data residency - their data stays in their preferred region. We can onboard thousands of small tenants without infrastructure changes, while giving enterprise customers the isolation they need."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                   MULTI-TENANT ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   TENANT RESOLUTION                                             │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │  Request → [Subdomain/JWT/Header] → Tenant Context      │  │
│   └─────────────────────────────────────────────────────────┘  │
│                          │                                       │
│                          ▼                                       │
│   APPLICATION LAYER                                             │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐    │  │
│   │  │ Auth    │  │ Rate    │  │ Feature │  │ Billing │    │  │
│   │  │(tenant) │  │ Limit   │  │ Flags   │  │ (tenant)│    │  │
│   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘    │  │
│   └─────────────────────────────────────────────────────────┘  │
│                          │                                       │
│                          ▼                                       │
│   DATA LAYER (Hybrid Model)                                     │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │                                                          │  │
│   │  POOL (Small Tenants)        SILO (Enterprise)          │  │
│   │  ┌──────────────────┐       ┌──────────────────┐       │  │
│   │  │ Shared Database  │       │ Tenant A DB      │       │  │
│   │  │ ┌──────────────┐ │       │ (Dedicated)      │       │  │
│   │  │ │tenant_id: 1  │ │       └──────────────────┘       │  │
│   │  │ │tenant_id: 2  │ │       ┌──────────────────┐       │  │
│   │  │ │tenant_id: 3  │ │       │ Tenant B DB      │       │  │
│   │  │ └──────────────┘ │       │ (Dedicated)      │       │  │
│   │  └──────────────────┘       └──────────────────┘       │  │
│   │                                                          │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is multi-tenancy?"**
> "Single software instance serving multiple customers. Each tenant's data is isolated. Shared infrastructure reduces cost. Core SaaS architecture pattern."

**Q: "Pool vs Silo?"**
> "Pool: shared database, tenant_id column, lowest cost, risk of data leaks. Silo: database per tenant, strong isolation, higher cost. Choose based on compliance needs and customer size."

**Q: "How do you prevent data leaks?"**
> "Row-Level Security (RLS), middleware that injects tenant filter, ORM hooks that auto-add WHERE tenant_id=X, code reviews, and automated tests that verify isolation."

**Q: "What's the noisy neighbor problem?"**
> "One tenant consuming excessive resources degrades performance for others. Solved with: rate limiting per tenant, separate connection pools, resource quotas, and tenant-based throttling."

**Q: "How do you handle enterprise vs small tenants?"**
> "Hybrid model: small tenants in shared pool (cost-efficient), enterprise tenants get dedicated database/schema (isolation, compliance, custom SLAs)."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How do you design a multi-tenant system?"

**Junior Answer:**
> "Add a tenant_id column to every table."

**Senior Answer:**
> "Multi-tenant design involves several key decisions:

**1. Isolation Model Selection**
- Start with shared schema (pool) for simplicity
- Offer dedicated database for enterprise/compliance
- Consider hybrid: pool for SMB, silo for enterprise

**2. Tenant Context Flow**
- Resolve tenant early (subdomain, JWT claim, header)
- Pass through request context
- Never trust client-provided tenant ID for data access

**3. Data Access Layer**
- ORM middleware auto-adds tenant filter
- RLS as database-level safety net
- Audit logging for compliance

**4. Resource Isolation**
- Rate limiting per tenant
- Connection pool isolation
- Background job queues per tenant (optional)

**5. Operational Concerns**
- Schema migrations must work for all tenants
- Backups: individual restore for enterprise
- Monitoring: metrics per tenant
- Data residency for compliance

Trade-offs:
- Pool: simple, cheap, but cross-tenant bugs are catastrophic
- Silo: isolated, expensive, complex migrations
- Start pool, offer silo upgrade path"

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "What about migrations?" | "Shared schema: one migration, all tenants. Dedicated DB: must migrate each, use orchestration. Test migrations with tenant data representative of all sizes." |
| "How do you test isolation?" | "Unit tests with multiple tenants, integration tests that verify cross-tenant queries return empty, chaos testing that simulates tenant context loss." |
| "Schema customization?" | "Avoid per-tenant schema changes. Use JSON columns or feature flags. If truly needed, schema-per-tenant model required." |
| "What about caching?" | "Tenant-prefixed cache keys. Redis: `tenant:{id}:user:{userId}`. Never share cache entries across tenants." |

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Data Isolation Strategies](#2-data-isolation-strategies)
3. [Tenant Management](#3-tenant-management)
4. [Scaling Strategies](#4-scaling-strategies)
5. [Implementation Patterns](#5-implementation-patterns)
6. [When to Use / Not Use](#6-when-to-use--not-use)
7. [Interview Questions](#7-interview-questions)

---

## 1. Core Concepts

### What is a Tenant?

```
TENANT DEFINITION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  TENANT = A customer organization using your SaaS               │
│                                                                  │
│  Examples:                                                      │
│  ├── Slack: Each company (Acme Inc, BigCorp) is a tenant       │
│  ├── Shopify: Each store is a tenant                           │
│  ├── GitHub: Each organization is a tenant                     │
│  └── Salesforce: Each company account is a tenant              │
│                                                                  │
│  TENANT vs USER:                                                │
│  ├── Tenant: The organization/company                          │
│  ├── User: Individual person within a tenant                   │
│  └── One tenant has many users                                 │
│                                                                  │
│  TENANT HIERARCHY:                                              │
│  ┌──────────────────────────────────────────────────┐          │
│  │ Tenant (Acme Inc)                                │          │
│  │  ├── Organization Settings                       │          │
│  │  ├── Billing & Subscription                      │          │
│  │  ├── Users                                       │          │
│  │  │    ├── Admin users                           │          │
│  │  │    └── Regular users                         │          │
│  │  ├── Resources (projects, files, data)         │          │
│  │  └── Integrations                               │          │
│  └──────────────────────────────────────────────────┘          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Multi-Tenancy vs Single-Tenancy

```
DEPLOYMENT MODELS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SINGLE-TENANT (Dedicated)                                      │
│  ─────────────────────────                                      │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                        │
│  │ App +   │  │ App +   │  │ App +   │                        │
│  │ DB for  │  │ DB for  │  │ DB for  │                        │
│  │Tenant A │  │Tenant B │  │Tenant C │                        │
│  └─────────┘  └─────────┘  └─────────┘                        │
│                                                                  │
│  ✓ Complete isolation                                          │
│  ✓ Custom configurations                                       │
│  ✗ High cost (3x infrastructure)                              │
│  ✗ Complex updates (deploy 3 times)                           │
│                                                                  │
│  ═══════════════════════════════════════════════════════════   │
│                                                                  │
│  MULTI-TENANT (Shared)                                          │
│  ─────────────────────                                          │
│  ┌─────────────────────────────────────────┐                   │
│  │        Single Application               │                   │
│  │  ┌─────────┬─────────┬─────────┐       │                   │
│  │  │Tenant A │Tenant B │Tenant C │       │                   │
│  │  │  Data   │  Data   │  Data   │       │                   │
│  │  └─────────┴─────────┴─────────┘       │                   │
│  └─────────────────────────────────────────┘                   │
│                                                                  │
│  ✓ Cost efficient (shared resources)                           │
│  ✓ Simple updates (deploy once)                                │
│  ✗ Isolation complexity                                        │
│  ✗ Noisy neighbor risk                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Tenant Resolution Strategies

```
HOW TO IDENTIFY THE TENANT:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. SUBDOMAIN                                                   │
│     └── acme.myapp.com → tenant: "acme"                        │
│     └── bigcorp.myapp.com → tenant: "bigcorp"                  │
│     ✓ Clean URLs, easy to remember                             │
│     ✗ DNS/SSL complexity, can't change easily                  │
│                                                                  │
│  2. PATH PREFIX                                                 │
│     └── myapp.com/acme/dashboard → tenant: "acme"              │
│     └── myapp.com/bigcorp/dashboard → tenant: "bigcorp"        │
│     ✓ Simple, single domain                                    │
│     ✗ URLs less clean, routing complexity                      │
│                                                                  │
│  3. CUSTOM DOMAIN                                               │
│     └── app.acme.com → tenant: "acme"                          │
│     └── portal.bigcorp.io → tenant: "bigcorp"                  │
│     ✓ White-label, professional                                │
│     ✗ Complex DNS, SSL certificate management                  │
│                                                                  │
│  4. JWT CLAIM                                                   │
│     └── JWT payload: { "tenant_id": "acme" }                   │
│     ✓ Works for APIs, mobile apps                              │
│     ✗ Need token for every request                             │
│                                                                  │
│  5. HEADER                                                      │
│     └── X-Tenant-ID: acme                                      │
│     ✓ Flexible, works with any client                          │
│     ✗ Easy to forget, must validate                            │
│                                                                  │
│  RECOMMENDATION:                                                │
│  └── Web: Subdomain (best UX)                                  │
│  └── API: JWT claim (secure, stateless)                        │
│  └── Enterprise: Custom domain option                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Data Isolation Strategies

### Strategy 1: Shared Schema (Pool Model)

```typescript
// ════════════════════════════════════════════════════════════════
// SHARED SCHEMA: All tenants in same tables
// ════════════════════════════════════════════════════════════════

// Database schema - every table has tenant_id
/*
CREATE TABLE users (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  email VARCHAR(255) NOT NULL,
  name VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW(),
  
  -- Composite unique constraint (email unique per tenant)
  UNIQUE(tenant_id, email)
);

CREATE INDEX idx_users_tenant ON users(tenant_id);
*/

// Prisma schema
model User {
  id        String   @id @default(uuid())
  tenantId  String   @map("tenant_id")
  email     String
  name      String?
  createdAt DateTime @default(now()) @map("created_at")
  
  tenant    Tenant   @relation(fields: [tenantId], references: [id])
  
  @@unique([tenantId, email])
  @@index([tenantId])
  @@map("users")
}

// Prisma middleware to auto-inject tenant filter
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// Middleware that adds tenant filter to all queries
prisma.$use(async (params, next) => {
  const tenantId = getTenantFromContext();  // From AsyncLocalStorage
  
  if (!tenantId) {
    throw new Error('Tenant context required');
  }
  
  // Auto-add tenant filter to queries
  if (params.action === 'findMany' || params.action === 'findFirst') {
    params.args.where = {
      ...params.args.where,
      tenantId,
    };
  }
  
  // Auto-add tenant to creates
  if (params.action === 'create') {
    params.args.data = {
      ...params.args.data,
      tenantId,
    };
  }
  
  // Prevent updates/deletes across tenants
  if (params.action === 'update' || params.action === 'delete') {
    params.args.where = {
      ...params.args.where,
      tenantId,
    };
  }
  
  return next(params);
});
```

### PostgreSQL Row-Level Security (RLS)

```sql
-- ════════════════════════════════════════════════════════════════
-- ROW-LEVEL SECURITY: Database enforces tenant isolation
-- ════════════════════════════════════════════════════════════════

-- Enable RLS on table
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their tenant's data
CREATE POLICY tenant_isolation ON users
  USING (tenant_id = current_setting('app.tenant_id')::uuid);

-- Policy for INSERT: Can only insert for own tenant
CREATE POLICY tenant_insert ON users
  FOR INSERT
  WITH CHECK (tenant_id = current_setting('app.tenant_id')::uuid);

-- Policy for UPDATE: Can only update own tenant's data
CREATE POLICY tenant_update ON users
  FOR UPDATE
  USING (tenant_id = current_setting('app.tenant_id')::uuid)
  WITH CHECK (tenant_id = current_setting('app.tenant_id')::uuid);

-- Policy for DELETE: Can only delete own tenant's data
CREATE POLICY tenant_delete ON users
  FOR DELETE
  USING (tenant_id = current_setting('app.tenant_id')::uuid);

-- Usage: Set tenant context before queries
-- SET app.tenant_id = 'tenant-uuid-here';
-- SELECT * FROM users;  -- Only returns current tenant's users
```

```typescript
// ════════════════════════════════════════════════════════════════
// Using RLS with Node.js
// ════════════════════════════════════════════════════════════════

import { Pool } from 'pg';

const pool = new Pool({ connectionString: process.env.DATABASE_URL });

async function queryWithTenant<T>(
  tenantId: string,
  query: string,
  params: any[] = []
): Promise<T[]> {
  const client = await pool.connect();
  
  try {
    // Set tenant context for RLS
    await client.query(`SET app.tenant_id = $1`, [tenantId]);
    
    // Execute query - RLS automatically filters
    const result = await client.query(query, params);
    
    return result.rows;
  } finally {
    // Reset and release connection
    await client.query(`RESET app.tenant_id`);
    client.release();
  }
}

// Usage
const users = await queryWithTenant(
  'tenant-123',
  'SELECT * FROM users WHERE active = true'
);
// RLS ensures only tenant-123's users are returned
```

### Strategy 2: Schema Per Tenant

```typescript
// ════════════════════════════════════════════════════════════════
// SCHEMA PER TENANT: Separate schema for each tenant
// ════════════════════════════════════════════════════════════════

/*
Database structure:
├── public (shared)
│   └── tenants table
├── tenant_acme
│   ├── users
│   ├── projects
│   └── ...
├── tenant_bigcorp
│   ├── users
│   ├── projects
│   └── ...
*/

// Create tenant schema
async function createTenantSchema(tenantSlug: string): Promise<void> {
  const schemaName = `tenant_${tenantSlug}`;
  
  await pool.query(`CREATE SCHEMA IF NOT EXISTS ${schemaName}`);
  
  // Create tables in tenant schema
  await pool.query(`
    CREATE TABLE ${schemaName}.users (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      email VARCHAR(255) NOT NULL UNIQUE,
      name VARCHAR(255),
      created_at TIMESTAMP DEFAULT NOW()
    )
  `);
  
  // ... create other tables
}

// Query with schema
async function queryTenantSchema<T>(
  tenantSlug: string,
  query: string,
  params: any[] = []
): Promise<T[]> {
  const client = await pool.connect();
  
  try {
    // Set search path to tenant schema
    await client.query(`SET search_path TO tenant_${tenantSlug}, public`);
    
    const result = await client.query(query, params);
    return result.rows;
  } finally {
    await client.query(`RESET search_path`);
    client.release();
  }
}

// Usage
const users = await queryTenantSchema('acme', 'SELECT * FROM users');
// Queries tenant_acme.users
```

### Strategy 3: Database Per Tenant

```typescript
// ════════════════════════════════════════════════════════════════
// DATABASE PER TENANT: Strongest isolation
// ════════════════════════════════════════════════════════════════

import { Pool, PoolConfig } from 'pg';

// Tenant database registry
interface TenantDatabase {
  tenantId: string;
  connectionString: string;
  pool?: Pool;
}

class TenantDatabaseManager {
  private tenantDatabases: Map<string, Pool> = new Map();
  private masterPool: Pool;
  
  constructor(masterConnectionString: string) {
    this.masterPool = new Pool({ connectionString: masterConnectionString });
  }
  
  // Get or create connection pool for tenant
  async getPool(tenantId: string): Promise<Pool> {
    if (this.tenantDatabases.has(tenantId)) {
      return this.tenantDatabases.get(tenantId)!;
    }
    
    // Lookup tenant database connection from master
    const result = await this.masterPool.query(
      'SELECT connection_string FROM tenant_databases WHERE tenant_id = $1',
      [tenantId]
    );
    
    if (result.rows.length === 0) {
      throw new Error(`Tenant ${tenantId} not found`);
    }
    
    const pool = new Pool({
      connectionString: result.rows[0].connection_string,
      max: 10,  // Smaller pool per tenant
    });
    
    this.tenantDatabases.set(tenantId, pool);
    return pool;
  }
  
  // Provision new tenant database
  async provisionTenantDatabase(tenantId: string): Promise<string> {
    const dbName = `tenant_${tenantId.replace(/-/g, '_')}`;
    
    // Create database
    await this.masterPool.query(`CREATE DATABASE ${dbName}`);
    
    // Run migrations on new database
    const connectionString = `postgresql://user:pass@host/${dbName}`;
    await this.runMigrations(connectionString);
    
    // Store connection info
    await this.masterPool.query(
      'INSERT INTO tenant_databases (tenant_id, connection_string) VALUES ($1, $2)',
      [tenantId, connectionString]
    );
    
    return connectionString;
  }
  
  private async runMigrations(connectionString: string): Promise<void> {
    // Run your migration tool against the new database
    // e.g., Prisma, Knex, TypeORM migrations
  }
}

// Usage
const dbManager = new TenantDatabaseManager(process.env.MASTER_DB_URL);

async function getUsersForTenant(tenantId: string) {
  const pool = await dbManager.getPool(tenantId);
  const result = await pool.query('SELECT * FROM users');
  return result.rows;
}
```

### Isolation Comparison

```
DATA ISOLATION COMPARISON:
┌────────────────┬───────────────┬───────────────┬───────────────┐
│ Factor         │ Shared Schema │ Schema/Tenant │ DB/Tenant     │
├────────────────┼───────────────┼───────────────┼───────────────┤
│ Isolation      │ Low (RLS)     │ Medium        │ High          │
│ Cost           │ Lowest        │ Medium        │ Highest       │
│ Complexity     │ Low           │ Medium        │ High          │
│ Migration      │ Easy (once)   │ Medium (each) │ Hard (each)   │
│ Customization  │ None          │ Possible      │ Full          │
│ Backup/Restore │ Complex       │ Per schema    │ Per database  │
│ Performance    │ Shared        │ Better        │ Dedicated     │
│ Compliance     │ Harder        │ Easier        │ Easiest       │
│ Scaling        │ Vertical      │ Mixed         │ Horizontal    │
├────────────────┼───────────────┼───────────────┼───────────────┤
│ Best For       │ Startups,     │ Mid-market,   │ Enterprise,   │
│                │ many small    │ moderate      │ regulated     │
│                │ tenants       │ tenants       │ industries    │
└────────────────┴───────────────┴───────────────┴───────────────┘
```

---

## 3. Tenant Management

### Tenant Resolution Middleware

```typescript
// ════════════════════════════════════════════════════════════════
// TENANT CONTEXT with AsyncLocalStorage
// ════════════════════════════════════════════════════════════════

import { AsyncLocalStorage } from 'async_hooks';
import { Request, Response, NextFunction } from 'express';

interface TenantContext {
  tenantId: string;
  tenantSlug: string;
  tier: 'free' | 'pro' | 'enterprise';
  features: string[];
}

const tenantStorage = new AsyncLocalStorage<TenantContext>();

// Get current tenant (from anywhere in code)
export function getTenant(): TenantContext {
  const tenant = tenantStorage.getStore();
  if (!tenant) {
    throw new Error('No tenant context available');
  }
  return tenant;
}

// Get tenant or null (for routes that don't require tenant)
export function getTenantOrNull(): TenantContext | null {
  return tenantStorage.getStore() || null;
}

// ════════════════════════════════════════════════════════════════
// SUBDOMAIN RESOLUTION
// ════════════════════════════════════════════════════════════════

async function resolveTenantFromSubdomain(req: Request): Promise<TenantContext | null> {
  const host = req.hostname;  // e.g., "acme.myapp.com"
  
  // Skip for main domain
  if (host === 'myapp.com' || host === 'www.myapp.com') {
    return null;
  }
  
  const subdomain = host.split('.')[0];  // "acme"
  
  // Lookup tenant by subdomain
  const tenant = await prisma.tenant.findUnique({
    where: { slug: subdomain },
    include: { subscription: true },
  });
  
  if (!tenant) return null;
  
  return {
    tenantId: tenant.id,
    tenantSlug: tenant.slug,
    tier: tenant.subscription?.tier || 'free',
    features: tenant.subscription?.features || [],
  };
}

// ════════════════════════════════════════════════════════════════
// JWT CLAIM RESOLUTION
// ════════════════════════════════════════════════════════════════

async function resolveTenantFromJWT(req: Request): Promise<TenantContext | null> {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) return null;
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET) as {
      tenantId: string;
      userId: string;
    };
    
    const tenant = await prisma.tenant.findUnique({
      where: { id: decoded.tenantId },
      include: { subscription: true },
    });
    
    if (!tenant) return null;
    
    return {
      tenantId: tenant.id,
      tenantSlug: tenant.slug,
      tier: tenant.subscription?.tier || 'free',
      features: tenant.subscription?.features || [],
    };
  } catch {
    return null;
  }
}

// ════════════════════════════════════════════════════════════════
// TENANT MIDDLEWARE
// ════════════════════════════════════════════════════════════════

export function tenantMiddleware(options: { required: boolean } = { required: true }) {
  return async (req: Request, res: Response, next: NextFunction) => {
    // Try multiple resolution strategies
    let tenant = await resolveTenantFromSubdomain(req);
    
    if (!tenant) {
      tenant = await resolveTenantFromJWT(req);
    }
    
    if (!tenant && options.required) {
      return res.status(400).json({ error: 'Tenant context required' });
    }
    
    if (tenant) {
      // Run request in tenant context
      tenantStorage.run(tenant, () => next());
    } else {
      next();
    }
  };
}

// Usage
app.use('/api', tenantMiddleware({ required: true }));

// In any service/controller
async function getUsers() {
  const { tenantId } = getTenant();
  return prisma.user.findMany({ where: { tenantId } });
}
```

### Tenant Onboarding

```typescript
// ════════════════════════════════════════════════════════════════
// TENANT PROVISIONING
// ════════════════════════════════════════════════════════════════

interface CreateTenantInput {
  name: string;
  slug: string;
  adminEmail: string;
  tier: 'free' | 'pro' | 'enterprise';
}

async function provisionTenant(input: CreateTenantInput): Promise<Tenant> {
  return prisma.$transaction(async (tx) => {
    // 1. Create tenant record
    const tenant = await tx.tenant.create({
      data: {
        name: input.name,
        slug: input.slug,
        status: 'provisioning',
      },
    });
    
    // 2. Create subscription
    await tx.subscription.create({
      data: {
        tenantId: tenant.id,
        tier: input.tier,
        features: getTierFeatures(input.tier),
        startDate: new Date(),
      },
    });
    
    // 3. Create admin user
    const adminUser = await tx.user.create({
      data: {
        tenantId: tenant.id,
        email: input.adminEmail,
        role: 'admin',
      },
    });
    
    // 4. Create default resources
    await tx.workspace.create({
      data: {
        tenantId: tenant.id,
        name: 'Default Workspace',
      },
    });
    
    // 5. Set up integrations (if enterprise)
    if (input.tier === 'enterprise') {
      await provisionDedicatedResources(tenant.id);
    }
    
    // 6. Mark as active
    await tx.tenant.update({
      where: { id: tenant.id },
      data: { status: 'active' },
    });
    
    // 7. Send welcome email
    await sendWelcomeEmail(input.adminEmail, tenant.slug);
    
    return tenant;
  });
}

// Enterprise: provision dedicated database
async function provisionDedicatedResources(tenantId: string): Promise<void> {
  // Create dedicated database
  const connectionString = await dbManager.provisionTenantDatabase(tenantId);
  
  // Store connection info
  await prisma.tenantConfig.create({
    data: {
      tenantId,
      key: 'database_connection',
      value: encrypt(connectionString),
    },
  });
  
  // Provision dedicated Redis namespace
  // Provision dedicated S3 bucket
  // Set up dedicated monitoring
}
```

### Multi-Tenant Authentication

```typescript
// ════════════════════════════════════════════════════════════════
// AUTHENTICATION WITH TENANT CONTEXT
// ════════════════════════════════════════════════════════════════

// JWT payload includes tenant
interface JWTPayload {
  userId: string;
  tenantId: string;
  email: string;
  role: string;
}

// Login: validate user belongs to tenant
async function login(
  email: string,
  password: string,
  tenantSlug: string
): Promise<{ token: string; user: User }> {
  // 1. Find tenant
  const tenant = await prisma.tenant.findUnique({
    where: { slug: tenantSlug },
  });
  
  if (!tenant) {
    throw new Error('Invalid credentials');  // Don't reveal tenant doesn't exist
  }
  
  // 2. Find user within tenant
  const user = await prisma.user.findFirst({
    where: {
      email,
      tenantId: tenant.id,
    },
  });
  
  if (!user || !await bcrypt.compare(password, user.passwordHash)) {
    throw new Error('Invalid credentials');
  }
  
  // 3. Generate JWT with tenant context
  const token = jwt.sign(
    {
      userId: user.id,
      tenantId: tenant.id,
      email: user.email,
      role: user.role,
    },
    process.env.JWT_SECRET,
    { expiresIn: '24h' }
  );
  
  return { token, user };
}

// ════════════════════════════════════════════════════════════════
// SSO: User belongs to multiple tenants
// ════════════════════════════════════════════════════════════════

interface TenantMembership {
  tenantId: string;
  tenantName: string;
  tenantSlug: string;
  role: string;
}

async function getUserTenants(email: string): Promise<TenantMembership[]> {
  const memberships = await prisma.user.findMany({
    where: { email },
    include: { tenant: true },
  });
  
  return memberships.map(m => ({
    tenantId: m.tenantId,
    tenantName: m.tenant.name,
    tenantSlug: m.tenant.slug,
    role: m.role,
  }));
}

// After SSO login, user selects which tenant to access
async function switchTenant(userId: string, newTenantId: string): Promise<string> {
  // Verify user has access to this tenant
  const membership = await prisma.user.findFirst({
    where: {
      id: userId,
      tenantId: newTenantId,
    },
  });
  
  if (!membership) {
    throw new Error('Access denied');
  }
  
  // Issue new JWT for the selected tenant
  return jwt.sign(
    {
      userId,
      tenantId: newTenantId,
      role: membership.role,
    },
    process.env.JWT_SECRET,
    { expiresIn: '24h' }
  );
}
```

### Tenant-Aware Authorization

```typescript
// ════════════════════════════════════════════════════════════════
// ROLE-BASED ACCESS WITHIN TENANT
// ════════════════════════════════════════════════════════════════

type TenantRole = 'owner' | 'admin' | 'member' | 'viewer';

const ROLE_PERMISSIONS: Record<TenantRole, string[]> = {
  owner: ['*'],  // Everything
  admin: ['users:manage', 'settings:manage', 'billing:view', 'data:*'],
  member: ['data:read', 'data:write', 'data:delete:own'],
  viewer: ['data:read'],
};

function hasPermission(role: TenantRole, permission: string): boolean {
  const permissions = ROLE_PERMISSIONS[role];
  
  if (permissions.includes('*')) return true;
  if (permissions.includes(permission)) return true;
  
  // Check wildcard permissions (e.g., data:*)
  const [resource, action] = permission.split(':');
  if (permissions.includes(`${resource}:*`)) return true;
  
  return false;
}

// Middleware to check permission
function requirePermission(permission: string) {
  return (req: Request, res: Response, next: NextFunction) => {
    const user = req.user;
    
    if (!hasPermission(user.role, permission)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    
    next();
  };
}

// Usage
app.delete('/api/users/:id', 
  requirePermission('users:manage'),
  deleteUser
);

// ════════════════════════════════════════════════════════════════
// FEATURE FLAGS PER TENANT
// ════════════════════════════════════════════════════════════════

function requireFeature(feature: string) {
  return (req: Request, res: Response, next: NextFunction) => {
    const tenant = getTenant();
    
    if (!tenant.features.includes(feature)) {
      return res.status(402).json({
        error: 'Feature not available',
        message: `Upgrade to access ${feature}`,
        upgradeUrl: `/billing/upgrade?feature=${feature}`,
      });
    }
    
    next();
  };
}

// Usage
app.post('/api/reports/advanced',
  requireFeature('advanced-analytics'),
  generateAdvancedReport
);
```

---

## 4. Scaling Strategies

### Noisy Neighbor Prevention

```typescript
// ════════════════════════════════════════════════════════════════
// RATE LIMITING PER TENANT
// ════════════════════════════════════════════════════════════════

import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

interface TenantRateLimits {
  free: { requests: number; window: number };
  pro: { requests: number; window: number };
  enterprise: { requests: number; window: number };
}

const RATE_LIMITS: TenantRateLimits = {
  free: { requests: 100, window: 60 },      // 100 req/min
  pro: { requests: 1000, window: 60 },      // 1000 req/min
  enterprise: { requests: 10000, window: 60 }, // 10000 req/min
};

async function checkRateLimit(tenantId: string, tier: string): Promise<boolean> {
  const limits = RATE_LIMITS[tier as keyof TenantRateLimits];
  const key = `ratelimit:${tenantId}`;
  
  const current = await redis.incr(key);
  
  if (current === 1) {
    await redis.expire(key, limits.window);
  }
  
  return current <= limits.requests;
}

// Middleware
function tenantRateLimiter() {
  return async (req: Request, res: Response, next: NextFunction) => {
    const tenant = getTenant();
    
    const allowed = await checkRateLimit(tenant.tenantId, tenant.tier);
    
    if (!allowed) {
      const limits = RATE_LIMITS[tenant.tier as keyof TenantRateLimits];
      
      res.setHeader('X-RateLimit-Limit', limits.requests);
      res.setHeader('Retry-After', limits.window);
      
      return res.status(429).json({
        error: 'Rate limit exceeded',
        upgrade: tenant.tier !== 'enterprise' 
          ? 'Upgrade for higher limits' 
          : undefined,
      });
    }
    
    next();
  };
}

// ════════════════════════════════════════════════════════════════
// RESOURCE QUOTAS PER TENANT
// ════════════════════════════════════════════════════════════════

interface TenantQuotas {
  maxUsers: number;
  maxProjects: number;
  maxStorageGB: number;
  maxApiKeys: number;
}

const TIER_QUOTAS: Record<string, TenantQuotas> = {
  free: { maxUsers: 5, maxProjects: 3, maxStorageGB: 1, maxApiKeys: 2 },
  pro: { maxUsers: 50, maxProjects: 25, maxStorageGB: 50, maxApiKeys: 10 },
  enterprise: { maxUsers: -1, maxProjects: -1, maxStorageGB: -1, maxApiKeys: -1 }, // Unlimited
};

async function checkQuota(
  tenantId: string,
  tier: string,
  resource: keyof TenantQuotas
): Promise<{ allowed: boolean; current: number; limit: number }> {
  const quotas = TIER_QUOTAS[tier];
  const limit = quotas[resource];
  
  // Unlimited
  if (limit === -1) {
    return { allowed: true, current: 0, limit: -1 };
  }
  
  // Count current usage
  let current: number;
  switch (resource) {
    case 'maxUsers':
      current = await prisma.user.count({ where: { tenantId } });
      break;
    case 'maxProjects':
      current = await prisma.project.count({ where: { tenantId } });
      break;
    case 'maxStorageGB':
      current = await getStorageUsageGB(tenantId);
      break;
    case 'maxApiKeys':
      current = await prisma.apiKey.count({ where: { tenantId } });
      break;
    default:
      current = 0;
  }
  
  return { allowed: current < limit, current, limit };
}
```

### Tenant Sharding

```typescript
// ════════════════════════════════════════════════════════════════
// SHARDING LARGE TENANTS
// ════════════════════════════════════════════════════════════════

interface ShardConfig {
  shardId: string;
  connectionString: string;
  region: string;
  tenants: string[];
}

class TenantShardManager {
  private shards: Map<string, Pool> = new Map();
  private tenantShardMap: Map<string, string> = new Map();
  
  async initialize(): Promise<void> {
    // Load shard configuration
    const shardConfigs = await this.loadShardConfigs();
    
    for (const config of shardConfigs) {
      this.shards.set(config.shardId, new Pool({
        connectionString: config.connectionString,
      }));
      
      for (const tenantId of config.tenants) {
        this.tenantShardMap.set(tenantId, config.shardId);
      }
    }
  }
  
  getShardForTenant(tenantId: string): Pool {
    const shardId = this.tenantShardMap.get(tenantId);
    
    if (!shardId) {
      // Default shard for new/small tenants
      return this.shards.get('default')!;
    }
    
    return this.shards.get(shardId)!;
  }
  
  // Move tenant to dedicated shard (when they grow)
  async migrateToShard(tenantId: string, targetShardId: string): Promise<void> {
    const sourcePool = this.getShardForTenant(tenantId);
    const targetPool = this.shards.get(targetShardId)!;
    
    // 1. Copy data to target shard
    await this.copyTenantData(tenantId, sourcePool, targetPool);
    
    // 2. Update shard mapping (atomic)
    await this.updateShardMapping(tenantId, targetShardId);
    
    // 3. Delete from source (after grace period)
    setTimeout(async () => {
      await this.deleteTenantData(tenantId, sourcePool);
    }, 24 * 60 * 60 * 1000); // 24 hours
  }
}

// ════════════════════════════════════════════════════════════════
// CONNECTION POOL PER TENANT
// ════════════════════════════════════════════════════════════════

class TenantConnectionManager {
  private pools: Map<string, Pool> = new Map();
  private readonly maxPoolSize = 10;
  private readonly maxTenantPools = 100;
  
  getPool(tenantId: string): Pool {
    if (this.pools.has(tenantId)) {
      return this.pools.get(tenantId)!;
    }
    
    // Evict LRU pool if at capacity
    if (this.pools.size >= this.maxTenantPools) {
      this.evictLRUPool();
    }
    
    // Create tenant-specific pool
    const pool = new Pool({
      connectionString: process.env.DATABASE_URL,
      max: this.maxPoolSize,
      application_name: `tenant_${tenantId}`,  // For monitoring
    });
    
    this.pools.set(tenantId, pool);
    return pool;
  }
  
  private evictLRUPool(): void {
    // In production, use proper LRU cache
    const firstKey = this.pools.keys().next().value;
    const pool = this.pools.get(firstKey);
    pool?.end();
    this.pools.delete(firstKey);
  }
}
```

### Tenant-Aware Caching

```typescript
// ════════════════════════════════════════════════════════════════
// CACHE ISOLATION PER TENANT
// ════════════════════════════════════════════════════════════════

class TenantCache {
  private redis: Redis;
  
  constructor(redisUrl: string) {
    this.redis = new Redis(redisUrl);
  }
  
  // Always prefix keys with tenant ID
  private getKey(tenantId: string, key: string): string {
    return `tenant:${tenantId}:${key}`;
  }
  
  async get<T>(tenantId: string, key: string): Promise<T | null> {
    const data = await this.redis.get(this.getKey(tenantId, key));
    return data ? JSON.parse(data) : null;
  }
  
  async set<T>(
    tenantId: string,
    key: string,
    value: T,
    ttlSeconds?: number
  ): Promise<void> {
    const fullKey = this.getKey(tenantId, key);
    const data = JSON.stringify(value);
    
    if (ttlSeconds) {
      await this.redis.setex(fullKey, ttlSeconds, data);
    } else {
      await this.redis.set(fullKey, data);
    }
  }
  
  async delete(tenantId: string, key: string): Promise<void> {
    await this.redis.del(this.getKey(tenantId, key));
  }
  
  // Clear all cache for a tenant
  async clearTenant(tenantId: string): Promise<void> {
    const pattern = `tenant:${tenantId}:*`;
    const keys = await this.redis.keys(pattern);
    
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
  
  // Cache with tenant quotas
  async setWithQuota(
    tenantId: string,
    tier: string,
    key: string,
    value: any
  ): Promise<boolean> {
    const cacheQuotas = { free: 100, pro: 1000, enterprise: 10000 };
    const maxKeys = cacheQuotas[tier as keyof typeof cacheQuotas];
    
    // Check current cache usage
    const pattern = `tenant:${tenantId}:*`;
    const currentKeys = await this.redis.keys(pattern);
    
    if (currentKeys.length >= maxKeys) {
      // Evict oldest keys or reject
      return false;
    }
    
    await this.set(tenantId, key, value);
    return true;
  }
}

// Usage
const cache = new TenantCache(process.env.REDIS_URL);

async function getUser(userId: string): Promise<User> {
  const tenant = getTenant();
  
  // Check cache (tenant-isolated)
  const cached = await cache.get<User>(tenant.tenantId, `user:${userId}`);
  if (cached) return cached;
  
  // Fetch from DB
  const user = await prisma.user.findFirst({
    where: { id: userId, tenantId: tenant.tenantId },
  });
  
  if (user) {
    await cache.set(tenant.tenantId, `user:${userId}`, user, 300);
  }
  
  return user!;
}
```

### Data Residency

```typescript
// ════════════════════════════════════════════════════════════════
// DATA RESIDENCY: Keep data in specific regions
// ════════════════════════════════════════════════════════════════

interface RegionConfig {
  region: string;
  databaseUrl: string;
  redisUrl: string;
  s3Bucket: string;
}

const REGION_CONFIGS: Record<string, RegionConfig> = {
  'us-east': {
    region: 'us-east-1',
    databaseUrl: process.env.US_EAST_DB_URL!,
    redisUrl: process.env.US_EAST_REDIS_URL!,
    s3Bucket: 'myapp-us-east',
  },
  'eu-west': {
    region: 'eu-west-1',
    databaseUrl: process.env.EU_WEST_DB_URL!,
    redisUrl: process.env.EU_WEST_REDIS_URL!,
    s3Bucket: 'myapp-eu-west',
  },
  'ap-southeast': {
    region: 'ap-southeast-1',
    databaseUrl: process.env.AP_SOUTHEAST_DB_URL!,
    redisUrl: process.env.AP_SOUTHEAST_REDIS_URL!,
    s3Bucket: 'myapp-ap-southeast',
  },
};

class RegionalDataManager {
  private regionPools: Map<string, Pool> = new Map();
  
  constructor() {
    for (const [region, config] of Object.entries(REGION_CONFIGS)) {
      this.regionPools.set(region, new Pool({
        connectionString: config.databaseUrl,
      }));
    }
  }
  
  async getPoolForTenant(tenantId: string): Promise<Pool> {
    // Lookup tenant's data residency preference
    const tenant = await this.getTenantConfig(tenantId);
    const region = tenant.dataResidency || 'us-east';
    
    return this.regionPools.get(region)!;
  }
  
  async setDataResidency(tenantId: string, region: string): Promise<void> {
    if (!REGION_CONFIGS[region]) {
      throw new Error(`Invalid region: ${region}`);
    }
    
    // This would trigger a data migration job
    await prisma.tenant.update({
      where: { id: tenantId },
      data: { dataResidency: region },
    });
    
    // Queue data migration
    await queue.add('migrate-tenant-data', {
      tenantId,
      targetRegion: region,
    });
  }
}
```

---

## 5. Implementation Patterns

### Next.js Multi-Tenant Setup

```typescript
// ════════════════════════════════════════════════════════════════
// NEXT.JS: Subdomain-based multi-tenancy
// ════════════════════════════════════════════════════════════════

// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const hostname = request.headers.get('host') || '';
  const currentHost = hostname.split(':')[0];  // Remove port
  
  // Get subdomain
  const baseDomain = process.env.NEXT_PUBLIC_ROOT_DOMAIN || 'myapp.com';
  const subdomain = currentHost.replace(`.${baseDomain}`, '');
  
  // Skip for main domain
  if (currentHost === baseDomain || currentHost === `www.${baseDomain}`) {
    return NextResponse.next();
  }
  
  // Skip for localhost/dev
  if (currentHost === 'localhost') {
    return NextResponse.next();
  }
  
  // Rewrite to tenant-specific route
  const url = request.nextUrl.clone();
  url.pathname = `/tenant/${subdomain}${url.pathname}`;
  
  return NextResponse.rewrite(url);
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};

// ════════════════════════════════════════════════════════════════
// app/tenant/[tenant]/layout.tsx
// ════════════════════════════════════════════════════════════════

import { notFound } from 'next/navigation';
import { getTenantBySlug } from '@/lib/tenants';
import { TenantProvider } from '@/components/TenantProvider';

export default async function TenantLayout({
  children,
  params,
}: {
  children: React.ReactNode;
  params: { tenant: string };
}) {
  const tenant = await getTenantBySlug(params.tenant);
  
  if (!tenant) {
    notFound();
  }
  
  return (
    <TenantProvider tenant={tenant}>
      {children}
    </TenantProvider>
  );
}

// ════════════════════════════════════════════════════════════════
// TenantProvider.tsx
// ════════════════════════════════════════════════════════════════

'use client';

import { createContext, useContext } from 'react';

interface Tenant {
  id: string;
  slug: string;
  name: string;
  tier: string;
  features: string[];
  branding?: {
    logo?: string;
    primaryColor?: string;
  };
}

const TenantContext = createContext<Tenant | null>(null);

export function useTenant(): Tenant {
  const tenant = useContext(TenantContext);
  if (!tenant) throw new Error('useTenant must be used within TenantProvider');
  return tenant;
}

export function TenantProvider({
  tenant,
  children,
}: {
  tenant: Tenant;
  children: React.ReactNode;
}) {
  return (
    <TenantContext.Provider value={tenant}>
      <style>{`
        :root {
          --primary-color: ${tenant.branding?.primaryColor || '#3B82F6'};
        }
      `}</style>
      {children}
    </TenantContext.Provider>
  );
}
```

### Background Jobs Per Tenant

```typescript
// ════════════════════════════════════════════════════════════════
// BULL QUEUE: Tenant-aware job processing
// ════════════════════════════════════════════════════════════════

import { Queue, Worker, Job } from 'bullmq';

// Create queue per job type (not per tenant)
const emailQueue = new Queue('emails', {
  connection: { host: 'redis', port: 6379 },
});

// Job payload includes tenant context
interface TenantJob<T> {
  tenantId: string;
  data: T;
}

// Queue job with tenant context
async function queueEmail(tenantId: string, emailData: EmailData) {
  await emailQueue.add('send-email', {
    tenantId,
    data: emailData,
  } as TenantJob<EmailData>, {
    // Priority based on tenant tier
    priority: await getTenantPriority(tenantId),
    attempts: 3,
    backoff: {
      type: 'exponential',
      delay: 1000,
    },
  });
}

// Worker processes jobs with tenant context
const emailWorker = new Worker('emails', async (job: Job<TenantJob<EmailData>>) => {
  const { tenantId, data } = job.data;
  
  // Run in tenant context
  await runWithTenant(tenantId, async () => {
    // Tenant-specific email settings
    const tenant = getTenant();
    const emailConfig = await getTenantEmailConfig(tenant.tenantId);
    
    // Check tenant's email quota
    const quotaOk = await checkQuota(tenantId, tenant.tier, 'emails');
    if (!quotaOk) {
      throw new Error('Email quota exceeded');
    }
    
    await sendEmail(data, emailConfig);
  });
}, {
  connection: { host: 'redis', port: 6379 },
  concurrency: 10,
  limiter: {
    max: 100,
    duration: 60000, // 100 jobs per minute max
  },
});

// ════════════════════════════════════════════════════════════════
// SEPARATE QUEUES FOR ENTERPRISE TENANTS
// ════════════════════════════════════════════════════════════════

class TenantQueueManager {
  private queues: Map<string, Queue> = new Map();
  private defaultQueue: Queue;
  
  constructor() {
    this.defaultQueue = new Queue('jobs-default');
  }
  
  async getQueue(tenantId: string): Promise<Queue> {
    const tenant = await getTenantById(tenantId);
    
    // Enterprise tenants get dedicated queues
    if (tenant.tier === 'enterprise') {
      if (!this.queues.has(tenantId)) {
        this.queues.set(tenantId, new Queue(`jobs-${tenantId}`));
      }
      return this.queues.get(tenantId)!;
    }
    
    return this.defaultQueue;
  }
  
  async addJob(tenantId: string, jobName: string, data: any) {
    const queue = await this.getQueue(tenantId);
    await queue.add(jobName, { tenantId, ...data });
  }
}
```

### Testing Multi-Tenant Code

```typescript
// ════════════════════════════════════════════════════════════════
// TESTING: Verify tenant isolation
// ════════════════════════════════════════════════════════════════

import { describe, it, expect, beforeEach } from 'vitest';

describe('Multi-tenant isolation', () => {
  let tenantA: string;
  let tenantB: string;
  
  beforeEach(async () => {
    // Create test tenants
    tenantA = await createTestTenant('tenant-a');
    tenantB = await createTestTenant('tenant-b');
    
    // Create data for each tenant
    await runWithTenant(tenantA, async () => {
      await prisma.user.create({ data: { email: 'user@tenant-a.com' } });
    });
    
    await runWithTenant(tenantB, async () => {
      await prisma.user.create({ data: { email: 'user@tenant-b.com' } });
    });
  });
  
  it('should only return data for current tenant', async () => {
    await runWithTenant(tenantA, async () => {
      const users = await prisma.user.findMany();
      
      // Should only see tenant A's users
      expect(users).toHaveLength(1);
      expect(users[0].email).toBe('user@tenant-a.com');
      
      // Should NOT see tenant B's users
      const tenantBUsers = users.filter(u => u.email.includes('tenant-b'));
      expect(tenantBUsers).toHaveLength(0);
    });
  });
  
  it('should prevent cross-tenant updates', async () => {
    await runWithTenant(tenantA, async () => {
      // Get tenant B's user ID (simulating a bug/attack)
      const tenantBUser = await prisma.user.findFirst({
        where: { tenantId: tenantB },
      });
      
      // Attempt to update should fail or affect nothing
      const result = await prisma.user.updateMany({
        where: { id: tenantBUser?.id },
        data: { email: 'hacked@evil.com' },
      });
      
      expect(result.count).toBe(0); // No rows updated
    });
  });
  
  it('should require tenant context', async () => {
    // Without tenant context, should throw
    await expect(
      prisma.user.findMany()
    ).rejects.toThrow('Tenant context required');
  });
});

// ════════════════════════════════════════════════════════════════
// CHAOS TESTING: Simulate tenant context loss
// ════════════════════════════════════════════════════════════════

describe('Chaos: Missing tenant context', () => {
  it('should handle missing tenant gracefully', async () => {
    // Simulate middleware failure
    const req = mockRequest({ headers: {} });
    const res = mockResponse();
    
    await tenantMiddleware({ required: true })(req, res, () => {});
    
    expect(res.status).toHaveBeenCalledWith(400);
    expect(res.json).toHaveBeenCalledWith({ error: 'Tenant context required' });
  });
  
  it('should not leak data on tenant context loss', async () => {
    // Start with tenant context
    await runWithTenant('tenant-a', async () => {
      // Simulate async operation that loses context
      await new Promise(resolve => setTimeout(resolve, 100));
      
      // Context should still be available
      expect(getTenant().tenantId).toBe('tenant-a');
    });
  });
});
```

### Migrations for Multi-Tenant

```typescript
// ════════════════════════════════════════════════════════════════
// SCHEMA MIGRATIONS
// ════════════════════════════════════════════════════════════════

// Shared schema: Normal migration, runs once
// prisma/migrations/20240101_add_user_role/migration.sql
/*
ALTER TABLE users ADD COLUMN role VARCHAR(50) DEFAULT 'member';
*/

// Schema-per-tenant: Must run for each tenant
async function runMigrationForAllTenants(migrationFn: (schema: string) => Promise<void>) {
  const tenants = await prisma.tenant.findMany();
  
  for (const tenant of tenants) {
    const schema = `tenant_${tenant.slug}`;
    
    try {
      await migrationFn(schema);
      console.log(`✓ Migrated ${schema}`);
    } catch (error) {
      console.error(`✗ Failed ${schema}:`, error);
      // Decide: continue or abort all?
    }
  }
}

// Example migration
await runMigrationForAllTenants(async (schema) => {
  await pool.query(`ALTER TABLE ${schema}.users ADD COLUMN role VARCHAR(50) DEFAULT 'member'`);
});

// ════════════════════════════════════════════════════════════════
// DATABASE-PER-TENANT: Coordinate migrations
// ════════════════════════════════════════════════════════════════

async function migrateTenantDatabases() {
  const tenantConfigs = await prisma.tenantDatabase.findMany();
  
  const results = await Promise.allSettled(
    tenantConfigs.map(async (config) => {
      const pool = new Pool({ connectionString: config.connectionString });
      
      try {
        // Run Prisma migrations
        await execAsync(`DATABASE_URL=${config.connectionString} npx prisma migrate deploy`);
        return { tenantId: config.tenantId, status: 'success' };
      } catch (error) {
        return { tenantId: config.tenantId, status: 'failed', error };
      } finally {
        await pool.end();
      }
    })
  );
  
  // Report results
  const failed = results.filter(r => r.status === 'rejected');
  if (failed.length > 0) {
    throw new Error(`${failed.length} tenant migrations failed`);
  }
}
```

---

## 6. When to Use / Not Use

### When TO Use Multi-Tenancy

```
✅ USE MULTI-TENANCY WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. BUILDING A SAAS PRODUCT                                     │
│     └── Multiple customers using same application              │
│     └── Subscription-based business model                      │
│     └── Need to scale to many customers efficiently            │
│                                                                  │
│  2. COST EFFICIENCY MATTERS                                     │
│     └── Single deployment, single codebase                     │
│     └── Shared infrastructure reduces costs                    │
│     └── Operational simplicity (deploy once)                   │
│                                                                  │
│  3. RAPID CUSTOMER ONBOARDING                                   │
│     └── No need to provision new infrastructure per customer   │
│     └── Self-service signup                                    │
│     └── Tenant ready in seconds, not days                      │
│                                                                  │
│  4. FEATURE PARITY                                              │
│     └── All tenants get same features (tier-based gating)     │
│     └── Consistent UX across customers                         │
│     └── Centralized updates benefit everyone                   │
│                                                                  │
│  5. ANALYTICS & LEARNING                                        │
│     └── Aggregate insights across tenants                      │
│     └── ML models benefit from more data                       │
│     └── Benchmark tenants against each other                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When NOT to Use Multi-Tenancy

```
❌ DON'T USE MULTI-TENANCY WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. HIGH CUSTOMIZATION REQUIREMENTS                             │
│     └── Each customer needs different features                 │
│     └── Schema changes per customer                            │
│     └── Different deployment configurations                    │
│     → Use: Single-tenant with white-labeling                   │
│                                                                  │
│  2. EXTREME ISOLATION REQUIREMENTS                              │
│     └── Regulatory: data cannot share infrastructure           │
│     └── Government, healthcare with strict compliance          │
│     └── Customers won't accept shared resources               │
│     → Use: Single-tenant or dedicated DB per tenant            │
│                                                                  │
│  3. FEW LARGE CUSTOMERS                                         │
│     └── Only 5-10 enterprise customers                        │
│     └── Each pays enough to justify dedicated infra           │
│     └── Custom SLAs per customer                               │
│     → Use: Single-tenant deployments                           │
│                                                                  │
│  4. UNPREDICTABLE RESOURCE USAGE                               │
│     └── Tenants have wildly different usage patterns           │
│     └── One tenant could consume 90% of resources             │
│     └── Noisy neighbor is high risk                           │
│     → Use: Hybrid with enterprise on dedicated infra           │
│                                                                  │
│  5. SIMPLE B2C APPLICATION                                      │
│     └── Users, not organizations                               │
│     └── No concept of "tenant" or "organization"              │
│     → Use: Standard single-user architecture                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Common Pitfalls

```
⚠️ MULTI-TENANCY PITFALLS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. FORGETTING tenant_id FILTER                                │
│     └── One missing WHERE clause leaks all data               │
│     └── Fix: ORM middleware, RLS, code review                 │
│                                                                  │
│  2. TRUSTING CLIENT-PROVIDED TENANT ID                         │
│     └── User could change tenant_id in request                │
│     └── Fix: Always derive from JWT/session, never from body  │
│                                                                  │
│  3. SHARED CACHE WITHOUT TENANT PREFIX                         │
│     └── Cache key collision returns wrong data                │
│     └── Fix: Always prefix: `tenant:{id}:key`                 │
│                                                                  │
│  4. BACKGROUND JOBS WITHOUT TENANT CONTEXT                     │
│     └── Job runs without knowing which tenant                  │
│     └── Fix: Always include tenantId in job payload           │
│                                                                  │
│  5. LOGS WITHOUT TENANT IDENTIFIER                             │
│     └── Can't debug issues for specific tenant                │
│     └── Fix: Include tenantId in all log entries              │
│                                                                  │
│  6. MIGRATIONS BREAKING ONE TENANT                             │
│     └── Migration works for most, fails for edge case         │
│     └── Fix: Test migrations with representative data sets    │
│                                                                  │
│  7. NO PER-TENANT RATE LIMITING                                │
│     └── One tenant can DoS everyone                           │
│     └── Fix: Rate limit per tenant, not just global           │
│                                                                  │
│  8. UNIQUE CONSTRAINTS WITHOUT TENANT                          │
│     └── Email unique globally, not per-tenant                 │
│     └── Fix: UNIQUE(tenant_id, email) not just UNIQUE(email) │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Interview Questions & Answers

### Basic Questions

**Q1: What is multi-tenancy?**
> **A:** Architecture where single software instance serves multiple customers (tenants). Each tenant's data is isolated while sharing compute resources. Core pattern for SaaS - reduces cost, simplifies operations. Example: Slack, Shopify, Salesforce.

**Q2: What are the isolation models?**
> **A:** Three main models:
> - **Shared Schema (Pool)**: All tenants in same tables, `tenant_id` column. Lowest cost, weakest isolation.
> - **Schema Per Tenant (Bridge)**: Shared database, separate schemas. Medium cost/isolation.
> - **Database Per Tenant (Silo)**: Each tenant has own database. Highest cost, strongest isolation.
>
> Choose based on compliance needs, customer size, and cost constraints.

**Q3: How do you identify which tenant a request is for?**
> **A:** Tenant resolution strategies:
> - **Subdomain**: `acme.myapp.com` → tenant "acme"
> - **JWT claim**: `{ "tenant_id": "123" }`
> - **Path prefix**: `/acme/dashboard`
> - **Custom domain**: `app.customer.com`
>
> Web apps typically use subdomain, APIs use JWT claims.

**Q4: What's the noisy neighbor problem?**
> **A:** One tenant consuming excessive resources degrades performance for others. Solutions:
> - Per-tenant rate limiting
> - Resource quotas (storage, API calls)
> - Separate connection pools
> - Dedicated infrastructure for large tenants

### Intermediate Questions

**Q5: How do you prevent data leaks between tenants?**
> **A:** Multiple layers of defense:
> 1. **ORM middleware**: Auto-inject `tenant_id` filter on all queries
> 2. **Row-Level Security (RLS)**: Database enforces isolation
> 3. **Never trust client**: Derive tenant from JWT, not request body
> 4. **Composite unique constraints**: `UNIQUE(tenant_id, email)`
> 5. **Code reviews**: Check every query has tenant filter
> 6. **Automated tests**: Verify cross-tenant queries return empty

**Q6: How do you handle schema migrations?**
> **A:** Depends on isolation model:
> - **Shared schema**: One migration, affects all tenants. Must be backward-compatible.
> - **Schema per tenant**: Must run migration for each schema. Use orchestration.
> - **Database per tenant**: Must coordinate across all databases. Most complex.
>
> Key: Test migrations with representative tenant data. Have rollback plan.

**Q7: How do you handle caching?**
> **A:** Always prefix cache keys with tenant ID: `tenant:{id}:user:{userId}`. Never share cache entries across tenants. Clear tenant's entire cache on sensitive operations. Consider per-tenant cache quotas to prevent one tenant filling cache.

**Q8: What about background jobs?**
> **A:**
> - Always include `tenantId` in job payload
> - Restore tenant context when processing
> - Consider separate queues for enterprise tenants
> - Priority based on tenant tier
> - Don't let one tenant's jobs block others

### Advanced Questions

**Q9: Design a hybrid multi-tenant system**
> **A:** 
> 1. **Small tenants**: Shared database with RLS for isolation
> 2. **Enterprise tenants**: Dedicated database, auto-provisioned on upgrade
> 3. **Tenant router**: Middleware that checks tenant tier, routes to appropriate DB
> 4. **Feature flags**: Gate features by tier
> 5. **Migration path**: Data export/import when tenant upgrades
> 6. **Monitoring**: Per-tenant metrics, alerts for enterprise SLAs

**Q10: How do you handle data residency requirements?**
> **A:**
> - Deploy infrastructure in multiple regions (US, EU, APAC)
> - Store tenant's region preference
> - Route requests to correct regional database
> - Ensure backups stay in region
> - Consider: latency for global teams (read replicas vs data locality)

**Q11: What about tenant customization (white-labeling)?**
> **A:**
> - **Branding**: Store logo, colors in tenant config, apply via CSS variables
> - **Custom domains**: Wildcard SSL + DNS mapping table
> - **Feature toggles**: Tenant-specific feature flags
> - **Schema customization**: Avoid if possible. Use JSON columns or metadata tables. If truly needed, requires schema-per-tenant model.

**Q12: How do you migrate from single-tenant to multi-tenant?**
> **A:**
> 1. Add `tenant_id` column to all tables
> 2. Create default tenant for existing data
> 3. Update all queries to filter by tenant
> 4. Add middleware for tenant resolution
> 5. Implement RLS as safety net
> 6. Update unique constraints to be tenant-scoped
> 7. Prefix all cache keys
> 8. Add tenant context to background jobs
> 9. Update logging to include tenant
> 10. Extensive testing for data isolation

### Scenario Questions

**Q13: A customer reports seeing another tenant's data. How do you investigate?**
> **A:**
> 1. **Immediate**: Disable affected account, notify customers
> 2. **Investigate**: Check logs for requests without tenant filter
> 3. **Identify**: Find the query missing `tenant_id` WHERE clause
> 4. **Root cause**: Why did code review/tests miss this?
> 5. **Fix**: Add filter, add RLS if not present
> 6. **Prevent**: Add linter rule, mandatory test coverage for isolation
> 7. **Communicate**: Transparency with affected customers
> 8. **Post-mortem**: Document and share learnings

**Q14: Design tenant onboarding that can handle 1000 signups/day**
> **A:**
> 1. **Async provisioning**: API returns immediately, provision in background
> 2. **Tenant status**: `provisioning` → `active`
> 3. **Optimistic UI**: Show welcome while provisioning completes
> 4. **Shared resources**: Small tenants share infrastructure immediately
> 5. **Queued setup**: Background jobs for email setup, webhooks, integrations
> 6. **Rate limiting**: Prevent abuse of signup endpoint
> 7. **Monitoring**: Alert if provisioning queue backs up

---

## 🎓 Key Takeaways

1. **Multi-tenancy = single app serving multiple customers** with isolated data
2. **3 isolation models**: Pool (shared tables), Bridge (schema/tenant), Silo (DB/tenant)
3. **Tenant resolution**: subdomain for web, JWT for APIs
4. **Always filter by tenant_id** - ORM middleware + RLS as safety net
5. **Noisy neighbor prevention**: rate limiting, quotas, connection pools per tenant
6. **Cache keys must include tenant**: `tenant:{id}:key`
7. **Background jobs need tenant context** in payload
8. **Hybrid model**: shared for SMB, dedicated for enterprise
9. **Test isolation**: verify cross-tenant queries return empty
10. **Unique constraints**: must be scoped to tenant `UNIQUE(tenant_id, email)`

---

## 📚 Resources

### Documentation
- [PostgreSQL Row-Level Security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [Prisma Multi-Tenancy Guide](https://www.prisma.io/docs/guides/other/multi-tenancy)
- [AWS SaaS Lens](https://docs.aws.amazon.com/wellarchitected/latest/saas-lens/saas-lens.html)

### Tools & Libraries
- [Prisma](https://www.prisma.io/) - ORM with middleware support
- [TypeORM](https://typeorm.io/) - Supports multiple connections
- [Bull](https://github.com/OptimalBits/bull) - Background jobs with priorities

### Patterns
- [Microsoft SaaS Tenancy Patterns](https://docs.microsoft.com/en-us/azure/azure-sql/database/saas-tenancy-app-design-patterns)
- [Multi-Tenant Data Architecture](https://www.citusdata.com/blog/2016/10/03/designing-your-saas-database-for-scale-with-postgres/)


