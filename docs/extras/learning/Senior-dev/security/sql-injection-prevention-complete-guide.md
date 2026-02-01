# 💉 SQL Injection Prevention - Complete Guide

> A comprehensive guide to SQL injection prevention - parameterized queries, ORMs, input validation, and protecting your database from the most dangerous web vulnerability.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "SQL injection happens when user input is concatenated into SQL queries, allowing attackers to manipulate the query - prevention is simple: NEVER concatenate user input, ALWAYS use parameterized queries or ORMs that handle escaping automatically."

### SQL Injection Attack Flow
```
SQL INJECTION ATTACK:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  VULNERABLE CODE:                                               │
│  query = "SELECT * FROM users WHERE email = '" + email + "'"   │
│                                                                  │
│  NORMAL INPUT:                                                  │
│  email = "user@example.com"                                    │
│  → SELECT * FROM users WHERE email = 'user@example.com'        │
│                                                                  │
│  MALICIOUS INPUT:                                               │
│  email = "' OR '1'='1"                                         │
│  → SELECT * FROM users WHERE email = '' OR '1'='1'             │
│  → Returns ALL users!                                          │
│                                                                  │
│  DESTRUCTIVE INPUT:                                             │
│  email = "'; DROP TABLE users; --"                             │
│  → SELECT * FROM users WHERE email = ''; DROP TABLE users; --' │
│  → Deletes the entire users table!                             │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  TYPES OF SQL INJECTION:                                        │
│                                                                  │
│  1. In-band (Classic)                                          │
│     • Error-based: Extract data from error messages            │
│     • Union-based: Use UNION to retrieve other tables          │
│                                                                  │
│  2. Blind                                                       │
│     • Boolean-based: True/false responses reveal data          │
│     • Time-based: Response delay reveals data                  │
│                                                                  │
│  3. Out-of-band                                                │
│     • DNS/HTTP requests to exfiltrate data                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement
> "When I audited our codebase, I found 3 SQL injection vulnerabilities in legacy code that used string concatenation. I fixed them by migrating to parameterized queries and added a custom ESLint rule that flags any string concatenation with SQL keywords. For our ORM (Prisma), we're protected by default, but I added monitoring for raw query usage. We also implemented defense in depth: input validation rejects unexpected characters, database user has minimal privileges, and our WAF blocks common SQLi patterns. The key insight: parameterized queries aren't just 'best practice' - they make SQL injection structurally impossible."

---

## 📚 Prevention Techniques

### Parameterized Queries (The Solution)

```typescript
// ════════════════════════════════════════════════════════════════
// PARAMETERIZED QUERIES - THE RIGHT WAY
// ════════════════════════════════════════════════════════════════

import { Pool } from 'pg';

const pool = new Pool();

// ❌ VULNERABLE: String concatenation
async function getUser_VULNERABLE(email: string) {
    const query = `SELECT * FROM users WHERE email = '${email}'`;
    // Attacker: email = "' OR '1'='1" → Returns all users!
    return pool.query(query);
}

// ✅ SECURE: Parameterized query
async function getUser_SECURE(email: string) {
    const query = 'SELECT * FROM users WHERE email = $1';
    // $1 is a placeholder, email is passed separately
    // The database treats it as DATA, never as SQL
    return pool.query(query, [email]);
}

// ✅ SECURE: Multiple parameters
async function searchUsers(name: string, role: string, limit: number) {
    const query = `
        SELECT * FROM users 
        WHERE name ILIKE $1 
        AND role = $2 
        LIMIT $3
    `;
    return pool.query(query, [`%${name}%`, role, limit]);
}

// ════════════════════════════════════════════════════════════════
// MYSQL WITH PLACEHOLDERS
// ════════════════════════════════════════════════════════════════

import mysql from 'mysql2/promise';

const connection = await mysql.createConnection({/* config */});

// ✅ SECURE: Using ? placeholders
async function getUser(email: string) {
    const [rows] = await connection.execute(
        'SELECT * FROM users WHERE email = ?',
        [email]
    );
    return rows;
}

// ✅ SECURE: Named parameters
async function searchUsers(params: { name: string; status: string }) {
    const [rows] = await connection.execute(
        'SELECT * FROM users WHERE name = :name AND status = :status',
        params
    );
    return rows;
}
```

### ORM Protection (Prisma, TypeORM)

```typescript
// ════════════════════════════════════════════════════════════════
// PRISMA - SAFE BY DEFAULT
// ════════════════════════════════════════════════════════════════

import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// ✅ Safe: Prisma handles parameterization
async function getUser(email: string) {
    return prisma.user.findUnique({
        where: { email }  // Automatically parameterized
    });
}

// ✅ Safe: Complex queries
async function searchUsers(name: string, roles: string[]) {
    return prisma.user.findMany({
        where: {
            name: { contains: name, mode: 'insensitive' },
            role: { in: roles }
        }
    });
}

// ⚠️ CAUTION: Raw queries need care
async function rawQuery(tableName: string) {
    // ❌ VULNERABLE: Don't interpolate table/column names
    // return prisma.$queryRaw`SELECT * FROM ${tableName}`;
    
    // ✅ SAFE: Validate table name against allowlist
    const ALLOWED_TABLES = ['users', 'posts', 'comments'];
    if (!ALLOWED_TABLES.includes(tableName)) {
        throw new Error('Invalid table name');
    }
    return prisma.$queryRawUnsafe(`SELECT * FROM ${tableName}`);
}

// ✅ SAFE: Using Prisma.sql for dynamic queries
async function dynamicSearch(searchTerm: string) {
    return prisma.$queryRaw`
        SELECT * FROM users 
        WHERE name ILIKE ${`%${searchTerm}%`}
    `;  // Tagged template literal parameterizes automatically
}

// ════════════════════════════════════════════════════════════════
// TYPEORM - SAFE PATTERNS
// ════════════════════════════════════════════════════════════════

import { getRepository } from 'typeorm';
import { User } from './entities/User';

// ✅ Safe: QueryBuilder with parameters
async function searchUsers(name: string, minAge: number) {
    return getRepository(User)
        .createQueryBuilder('user')
        .where('user.name ILIKE :name', { name: `%${name}%` })
        .andWhere('user.age >= :minAge', { minAge })
        .getMany();
}

// ❌ VULNERABLE: Don't use string interpolation in where clauses
// .where(`user.name = '${name}'`)  // DON'T DO THIS!
```

### Dynamic Queries (Safe Patterns)

```typescript
// ════════════════════════════════════════════════════════════════
// DYNAMIC QUERIES - HANDLING VARIABLE CONDITIONS
// ════════════════════════════════════════════════════════════════

// ✅ SAFE: Building queries dynamically
interface SearchParams {
    name?: string;
    email?: string;
    role?: string;
    minAge?: number;
    sortBy?: string;
    sortOrder?: 'asc' | 'desc';
}

async function searchUsers(params: SearchParams) {
    const conditions: string[] = [];
    const values: any[] = [];
    let paramIndex = 1;
    
    if (params.name) {
        conditions.push(`name ILIKE $${paramIndex++}`);
        values.push(`%${params.name}%`);
    }
    
    if (params.email) {
        conditions.push(`email = $${paramIndex++}`);
        values.push(params.email);
    }
    
    if (params.role) {
        conditions.push(`role = $${paramIndex++}`);
        values.push(params.role);
    }
    
    if (params.minAge) {
        conditions.push(`age >= $${paramIndex++}`);
        values.push(params.minAge);
    }
    
    // Build WHERE clause
    const whereClause = conditions.length > 0
        ? `WHERE ${conditions.join(' AND ')}`
        : '';
    
    // ⚠️ Sort columns must be validated (can't parameterize identifiers)
    const ALLOWED_SORT_COLUMNS = ['name', 'email', 'created_at'];
    const sortColumn = ALLOWED_SORT_COLUMNS.includes(params.sortBy || '')
        ? params.sortBy
        : 'created_at';
    const sortOrder = params.sortOrder === 'desc' ? 'DESC' : 'ASC';
    
    const query = `
        SELECT * FROM users 
        ${whereClause} 
        ORDER BY ${sortColumn} ${sortOrder}
    `;
    
    return pool.query(query, values);
}

// ════════════════════════════════════════════════════════════════
// HANDLING IDENTIFIERS (Table/Column names)
// ════════════════════════════════════════════════════════════════

// You CANNOT parameterize table or column names
// You MUST validate against an allowlist

const ALLOWED_COLUMNS = ['name', 'email', 'role', 'status'];

function validateColumn(column: string): string {
    if (!ALLOWED_COLUMNS.includes(column)) {
        throw new Error('Invalid column name');
    }
    return column;
}

async function sortBy(column: string) {
    const safeColumn = validateColumn(column);
    // Now safe to use in query
    return pool.query(`SELECT * FROM users ORDER BY ${safeColumn}`);
}
```

### Input Validation (Defense in Depth)

```typescript
// ════════════════════════════════════════════════════════════════
// INPUT VALIDATION - ADDITIONAL LAYER
// ════════════════════════════════════════════════════════════════

import { z } from 'zod';

// Validate input before it reaches the database
const userSearchSchema = z.object({
    email: z.string().email().max(255).optional(),
    name: z.string().max(100).regex(/^[a-zA-Z\s]+$/).optional(),
    role: z.enum(['user', 'admin', 'moderator']).optional(),
    age: z.number().int().min(0).max(150).optional()
});

async function searchUsers(input: unknown) {
    // Validate input
    const params = userSearchSchema.parse(input);
    
    // Now use params safely with parameterized query
    // Even if validation is bypassed, parameterization protects us
    return prisma.user.findMany({
        where: {
            email: params.email,
            name: params.name ? { contains: params.name } : undefined,
            role: params.role,
            age: params.age ? { gte: params.age } : undefined
        }
    });
}

// ════════════════════════════════════════════════════════════════
// DETECTING SQLi ATTEMPTS (Logging/Monitoring)
// ════════════════════════════════════════════════════════════════

const SQLI_PATTERNS = [
    /'\s*OR\s+/i,           // ' OR 
    /'\s*--/,               // '--
    /;\s*DROP/i,            // ; DROP
    /;\s*DELETE/i,          // ; DELETE
    /UNION\s+SELECT/i,      // UNION SELECT
    /'\s*;\s*--/,           // '; --
];

function detectSQLiAttempt(input: string): boolean {
    return SQLI_PATTERNS.some(pattern => pattern.test(input));
}

// Middleware to log suspicious inputs
app.use((req, res, next) => {
    const allInputs = [
        ...Object.values(req.query),
        ...Object.values(req.body),
        ...Object.values(req.params)
    ].filter(v => typeof v === 'string');
    
    for (const input of allInputs) {
        if (detectSQLiAttempt(input as string)) {
            console.warn('Potential SQLi attempt detected:', {
                input,
                ip: req.ip,
                path: req.path,
                timestamp: new Date()
            });
            // Don't block - parameterized queries protect us
            // But log for analysis
        }
    }
    
    next();
});
```

### Database Permissions (Least Privilege)

```sql
-- ════════════════════════════════════════════════════════════════
-- LEAST PRIVILEGE DATABASE USER
-- ════════════════════════════════════════════════════════════════

-- Create application user with minimal permissions
CREATE USER app_user WITH PASSWORD 'secure_password';

-- Grant only necessary permissions
GRANT SELECT, INSERT, UPDATE ON users TO app_user;
GRANT SELECT, INSERT ON posts TO app_user;
GRANT SELECT ON categories TO app_user;

-- DON'T grant dangerous permissions
-- REVOKE DELETE ON users FROM app_user;
-- REVOKE DROP ON ALL TABLES FROM app_user;
-- REVOKE CREATE ON SCHEMA public FROM app_user;

-- Separate admin user for migrations/admin tasks
CREATE USER admin_user WITH PASSWORD 'different_secure_password';
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO admin_user;
```

---

## Testing for SQL Injection

```typescript
// ════════════════════════════════════════════════════════════════
// AUTOMATED TESTING FOR SQLI
// ════════════════════════════════════════════════════════════════

const SQLI_PAYLOADS = [
    "' OR '1'='1",
    "'; DROP TABLE users; --",
    "' UNION SELECT * FROM users --",
    "1'; SELECT * FROM users WHERE 't' = 't",
    "admin'--",
    "1 OR 1=1",
    "' OR ''='",
];

async function testEndpointForSQLi(endpoint: string, paramName: string) {
    const results = [];
    
    for (const payload of SQLI_PAYLOADS) {
        const response = await fetch(`${endpoint}?${paramName}=${encodeURIComponent(payload)}`);
        const body = await response.text();
        
        // Look for signs of successful injection
        const suspicious = 
            response.status === 500 ||  // Error might indicate SQLi
            body.includes('syntax error') ||
            body.includes('mysql') ||
            body.includes('postgresql') ||
            body.includes('sqlite');
        
        results.push({
            payload,
            status: response.status,
            suspicious
        });
    }
    
    return results;
}
```

---

## Interview Questions

**Q: "How do you prevent SQL injection?"**
> "Never concatenate user input into SQL queries. Use parameterized queries or prepared statements where user input is passed as parameters, separate from the SQL string. The database treats parameters as data, never as executable SQL. ORMs like Prisma handle this automatically. For dynamic table/column names, use allowlists since you can't parameterize identifiers."

**Q: "Why do parameterized queries prevent SQL injection?"**
> "When you use parameterized queries, the SQL statement and the data are sent to the database separately. The database compiles the SQL first (query plan), then applies the data. The data can never be interpreted as SQL because the query structure is already fixed. It's like the difference between handing someone a signed contract vs. letting them write their own terms."

**Q: "Can ORMs be vulnerable to SQL injection?"**
> "ORMs like Prisma and TypeORM are safe by default because they parameterize automatically. But they can still be vulnerable if you use raw SQL features incorrectly - interpolating variables into raw queries, or building dynamic table/column names without validation. Always validate identifiers against allowlists, and use the ORM's safe tagging features for raw queries."

---

## Quick Reference

```
SQL INJECTION PREVENTION CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  PRIMARY DEFENSE:                                               │
│  ✓ Parameterized queries for ALL user input                    │
│  ✓ ORMs with automatic parameterization                        │
│  ✓ Allowlist validation for table/column names                 │
│                                                                  │
│  DEFENSE IN DEPTH:                                              │
│  ✓ Input validation (type, format, length)                     │
│  ✓ Least privilege database users                              │
│  ✓ WAF rules for common SQLi patterns                          │
│  ✓ Error messages don't expose SQL details                     │
│                                                                  │
│  TESTING:                                                       │
│  ✓ Automated scanning (sqlmap, ZAP)                            │
│  ✓ Code review for raw SQL usage                               │
│  ✓ ESLint rules for string concatenation in queries            │
│                                                                  │
│  NEVER:                                                         │
│  ✗ String concatenation in SQL queries                         │
│  ✗ Trust user input without parameterization                   │
│  ✗ Use admin database credentials in app                       │
│  ✗ Expose SQL errors to users                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


