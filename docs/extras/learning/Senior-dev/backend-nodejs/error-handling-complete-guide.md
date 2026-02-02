# Error Handling Patterns - Complete Guide

> **MUST REMEMBER**: Effective error handling is about being explicit, consistent, and informative. Catch errors at the right level, transform them into appropriate types for each layer, and never swallow errors silently. In Node.js, unhandled promise rejections and uncaught exceptions should trigger graceful shutdown, not silent failure.

---

## How to Explain Like a Senior Developer

"Error handling isn't just try-catch - it's a system design decision. You need to think about error categories (operational vs programmer errors), how errors propagate through layers, what information to expose, and how to recover or fail gracefully. The key insight is that different layers need different error types: database layers throw connection errors, business logic throws domain errors, and APIs return HTTP status codes. A senior approach maps these appropriately, logs with context, and ensures nothing falls through the cracks."

---

## Core Implementation

### Error Type Hierarchy

```typescript
// errors/base.ts

/**
 * Base error class with proper stack trace support
 */
export class AppError extends Error {
  public readonly isOperational: boolean;
  public readonly statusCode: number;
  public readonly code: string;
  public readonly context?: Record<string, unknown>;
  public readonly timestamp: Date;
  
  constructor(
    message: string,
    options: {
      statusCode?: number;
      code?: string;
      isOperational?: boolean;
      context?: Record<string, unknown>;
      cause?: Error;
    } = {}
  ) {
    super(message, { cause: options.cause });
    
    this.name = this.constructor.name;
    this.statusCode = options.statusCode ?? 500;
    this.code = options.code ?? 'INTERNAL_ERROR';
    this.isOperational = options.isOperational ?? true;
    this.context = options.context;
    this.timestamp = new Date();
    
    // Maintains proper stack trace
    Error.captureStackTrace(this, this.constructor);
  }
  
  toJSON(): Record<string, unknown> {
    return {
      name: this.name,
      message: this.message,
      code: this.code,
      statusCode: this.statusCode,
      context: this.context,
      timestamp: this.timestamp.toISOString(),
      stack: this.stack,
    };
  }
}

// Specific error types
export class ValidationError extends AppError {
  public readonly errors: Array<{ field: string; message: string }>;
  
  constructor(
    errors: Array<{ field: string; message: string }>,
    context?: Record<string, unknown>
  ) {
    super('Validation failed', {
      statusCode: 400,
      code: 'VALIDATION_ERROR',
      context,
    });
    this.errors = errors;
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string, id?: string | number) {
    super(`${resource} not found${id ? `: ${id}` : ''}`, {
      statusCode: 404,
      code: 'NOT_FOUND',
      context: { resource, id },
    });
  }
}

export class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized', reason?: string) {
    super(message, {
      statusCode: 401,
      code: 'UNAUTHORIZED',
      context: { reason },
    });
  }
}

export class ForbiddenError extends AppError {
  constructor(message = 'Forbidden', action?: string) {
    super(message, {
      statusCode: 403,
      code: 'FORBIDDEN',
      context: { action },
    });
  }
}

export class ConflictError extends AppError {
  constructor(message: string, conflictingResource?: string) {
    super(message, {
      statusCode: 409,
      code: 'CONFLICT',
      context: { conflictingResource },
    });
  }
}

export class RateLimitError extends AppError {
  public readonly retryAfter: number;
  
  constructor(retryAfter: number) {
    super('Rate limit exceeded', {
      statusCode: 429,
      code: 'RATE_LIMIT_EXCEEDED',
      context: { retryAfter },
    });
    this.retryAfter = retryAfter;
  }
}

export class ExternalServiceError extends AppError {
  constructor(service: string, cause?: Error) {
    super(`External service error: ${service}`, {
      statusCode: 502,
      code: 'EXTERNAL_SERVICE_ERROR',
      context: { service },
      cause,
    });
  }
}

// Programmer errors (bugs - should not be caught normally)
export class ProgrammerError extends AppError {
  constructor(message: string, cause?: Error) {
    super(message, {
      statusCode: 500,
      code: 'PROGRAMMER_ERROR',
      isOperational: false, // Should trigger shutdown
      cause,
    });
  }
}
```

### Result Pattern (Alternative to Exceptions)

```typescript
// result.ts

/**
 * Result type for explicit error handling without exceptions
 * Useful for operations where failure is expected
 */
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

// Helper functions
function ok<T>(data: T): Result<T, never> {
  return { success: true, data };
}

function err<E>(error: E): Result<never, E> {
  return { success: false, error };
}

// Usage example
interface User {
  id: string;
  email: string;
  password: string;
}

interface CreateUserError {
  type: 'EMAIL_EXISTS' | 'INVALID_PASSWORD' | 'DATABASE_ERROR';
  message: string;
}

async function createUser(
  email: string,
  password: string
): Promise<Result<User, CreateUserError>> {
  // Check if email exists
  const existingUser = await db.users.findByEmail(email);
  if (existingUser) {
    return err({
      type: 'EMAIL_EXISTS',
      message: 'A user with this email already exists',
    });
  }
  
  // Validate password
  if (password.length < 8) {
    return err({
      type: 'INVALID_PASSWORD',
      message: 'Password must be at least 8 characters',
    });
  }
  
  try {
    const user = await db.users.create({ email, password });
    return ok(user);
  } catch (error) {
    return err({
      type: 'DATABASE_ERROR',
      message: 'Failed to create user',
    });
  }
}

// Using the result
async function handleCreateUser(req: Request, res: Response): Promise<void> {
  const result = await createUser(req.body.email, req.body.password);
  
  if (!result.success) {
    const statusMap = {
      EMAIL_EXISTS: 409,
      INVALID_PASSWORD: 400,
      DATABASE_ERROR: 500,
    };
    
    res.status(statusMap[result.error.type]).json({
      error: result.error.message,
    });
    return;
  }
  
  res.status(201).json(result.data);
}

// Mock db for type checking
const db = {
  users: {
    findByEmail: async (email: string): Promise<User | null> => null,
    create: async (data: { email: string; password: string }): Promise<User> => ({
      id: '1',
      email: data.email,
      password: data.password,
    }),
  },
};
```

### Express Error Handling Middleware

```typescript
// middleware/error-handler.ts
import { Request, Response, NextFunction, ErrorRequestHandler } from 'express';
import { AppError, ValidationError } from '../errors/base';
import { logger } from '../utils/logger';

interface ErrorResponse {
  status: 'error';
  code: string;
  message: string;
  errors?: Array<{ field: string; message: string }>;
  requestId?: string;
  timestamp: string;
}

export const errorHandler: ErrorRequestHandler = (
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  // Get request ID for tracing
  const requestId = req.headers['x-request-id'] as string;
  
  // Default error response
  let statusCode = 500;
  let response: ErrorResponse = {
    status: 'error',
    code: 'INTERNAL_ERROR',
    message: 'An unexpected error occurred',
    requestId,
    timestamp: new Date().toISOString(),
  };
  
  // Handle known errors
  if (error instanceof AppError) {
    statusCode = error.statusCode;
    response.code = error.code;
    response.message = error.message;
    
    if (error instanceof ValidationError) {
      response.errors = error.errors;
    }
    
    // Log operational errors as warnings
    if (error.isOperational) {
      logger.warn('Operational error', {
        error: error.toJSON(),
        requestId,
        path: req.path,
        method: req.method,
      });
    } else {
      // Non-operational errors are bugs - log as error
      logger.error('Programmer error', {
        error: error.toJSON(),
        requestId,
        path: req.path,
        method: req.method,
        stack: error.stack,
      });
    }
  } else if (error.name === 'JsonWebTokenError') {
    statusCode = 401;
    response.code = 'INVALID_TOKEN';
    response.message = 'Invalid authentication token';
  } else if (error.name === 'TokenExpiredError') {
    statusCode = 401;
    response.code = 'TOKEN_EXPIRED';
    response.message = 'Authentication token has expired';
  } else {
    // Unknown error - log full details
    logger.error('Unhandled error', {
      error: {
        name: error.name,
        message: error.message,
        stack: error.stack,
      },
      requestId,
      path: req.path,
      method: req.method,
    });
  }
  
  // Don't expose internal details in production
  if (process.env.NODE_ENV === 'production' && statusCode === 500) {
    response.message = 'An unexpected error occurred';
  }
  
  res.status(statusCode).json(response);
};

// Async handler wrapper
export function asyncHandler(
  fn: (req: Request, res: Response, next: NextFunction) => Promise<void>
): (req: Request, res: Response, next: NextFunction) => void {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

// 404 handler
export function notFoundHandler(req: Request, res: Response): void {
  res.status(404).json({
    status: 'error',
    code: 'NOT_FOUND',
    message: `Cannot ${req.method} ${req.path}`,
    timestamp: new Date().toISOString(),
  });
}

// Simple logger implementation
const logger = {
  warn: (msg: string, meta: object) => console.warn(msg, meta),
  error: (msg: string, meta: object) => console.error(msg, meta),
};
```

### Global Error Handlers

```typescript
// global-handlers.ts
import { logger } from './utils/logger';

/**
 * Setup global error handlers for unhandled errors
 * These should trigger graceful shutdown
 */
export function setupGlobalErrorHandlers(
  shutdownCallback: () => Promise<void>
): void {
  // Unhandled promise rejection
  process.on('unhandledRejection', (reason: unknown, promise: Promise<unknown>) => {
    logger.error('Unhandled Promise Rejection', {
      reason: reason instanceof Error ? {
        name: reason.name,
        message: reason.message,
        stack: reason.stack,
      } : reason,
    });
    
    // In production, trigger graceful shutdown
    if (process.env.NODE_ENV === 'production') {
      shutdownCallback().finally(() => process.exit(1));
    }
  });
  
  // Uncaught exception
  process.on('uncaughtException', (error: Error) => {
    logger.error('Uncaught Exception', {
      name: error.name,
      message: error.message,
      stack: error.stack,
    });
    
    // Always exit on uncaught exceptions - state may be corrupted
    shutdownCallback().finally(() => process.exit(1));
  });
  
  // Unhandled rejection in newer Node versions
  process.on('uncaughtExceptionMonitor', (error: Error, origin: string) => {
    logger.error('Uncaught Exception Monitor', {
      error: {
        name: error.name,
        message: error.message,
        stack: error.stack,
      },
      origin,
    });
  });
}

const logger = {
  error: (msg: string, meta: object) => console.error(msg, meta),
};
```

### Service Layer Error Handling

```typescript
// services/user-service.ts
import { AppError, NotFoundError, ConflictError, ExternalServiceError } from '../errors/base';
import { db } from '../db';
import { emailService } from './email-service';
import { logger } from '../utils/logger';

interface CreateUserDTO {
  email: string;
  name: string;
  password: string;
}

interface User {
  id: string;
  email: string;
  name: string;
}

export class UserService {
  async createUser(data: CreateUserDTO): Promise<User> {
    // Check for existing user
    const existing = await db.users.findByEmail(data.email);
    if (existing) {
      throw new ConflictError(
        'User with this email already exists',
        'user'
      );
    }
    
    // Create user in transaction
    const user = await db.transaction(async (tx) => {
      const newUser = await tx.users.create({
        email: data.email,
        name: data.name,
        passwordHash: await hashPassword(data.password),
      });
      
      await tx.userSettings.create({
        userId: newUser.id,
        theme: 'light',
        notifications: true,
      });
      
      return newUser;
    });
    
    // Send welcome email (non-critical)
    try {
      await emailService.sendWelcome(user.email, user.name);
    } catch (error) {
      // Log but don't fail user creation
      logger.warn('Failed to send welcome email', {
        userId: user.id,
        error: (error as Error).message,
      });
    }
    
    return user;
  }
  
  async getUserById(id: string): Promise<User> {
    const user = await db.users.findById(id);
    
    if (!user) {
      throw new NotFoundError('User', id);
    }
    
    return user;
  }
  
  async syncWithExternalCRM(userId: string): Promise<void> {
    const user = await this.getUserById(userId);
    
    try {
      await crmClient.syncUser(user);
    } catch (error) {
      // Wrap external errors
      throw new ExternalServiceError('CRM', error as Error);
    }
  }
}

// Mock implementations
async function hashPassword(password: string): Promise<string> {
  return password;
}

const db = {
  users: {
    findByEmail: async (email: string): Promise<User | null> => null,
    findById: async (id: string): Promise<User | null> => null,
  },
  transaction: async <T>(fn: (tx: any) => Promise<T>): Promise<T> => fn(db),
};

const emailService = {
  sendWelcome: async (email: string, name: string): Promise<void> => {},
};

const crmClient = {
  syncUser: async (user: User): Promise<void> => {},
};

const logger = {
  warn: (msg: string, meta: object) => console.warn(msg, meta),
};
```

### Error Boundary Pattern (Domain-Specific)

```typescript
// error-boundaries.ts

/**
 * Execute a function with error transformation
 * Useful for wrapping external library calls
 */
async function withErrorBoundary<T>(
  operation: () => Promise<T>,
  errorTransformer: (error: unknown) => Error
): Promise<T> {
  try {
    return await operation();
  } catch (error) {
    throw errorTransformer(error);
  }
}

// Database error boundary
import { AppError, NotFoundError } from '../errors/base';

class DatabaseError extends AppError {
  constructor(operation: string, cause: Error) {
    super(`Database error during ${operation}`, {
      statusCode: 500,
      code: 'DATABASE_ERROR',
      context: { operation },
      cause,
    });
  }
}

async function withDatabaseErrorBoundary<T>(
  operation: string,
  fn: () => Promise<T>
): Promise<T> {
  return withErrorBoundary(fn, (error) => {
    const err = error as Error & { code?: string };
    
    // Handle specific database errors
    if (err.code === 'P2025') { // Prisma not found
      return new NotFoundError('Record');
    }
    
    if (err.code === 'P2002') { // Unique constraint
      return new AppError('Duplicate entry', {
        statusCode: 409,
        code: 'DUPLICATE_ENTRY',
      });
    }
    
    return new DatabaseError(operation, err);
  });
}

// Usage
async function findUser(id: string): Promise<User> {
  return withDatabaseErrorBoundary('findUser', async () => {
    const user = await prisma.user.findUniqueOrThrow({
      where: { id },
    });
    return user;
  });
}

// HTTP client error boundary
class HttpClientError extends AppError {
  constructor(url: string, statusCode: number, cause?: Error) {
    super(`HTTP request failed: ${url}`, {
      statusCode: statusCode >= 500 ? 502 : statusCode,
      code: 'HTTP_CLIENT_ERROR',
      context: { url, originalStatus: statusCode },
      cause,
    });
  }
}

async function withHttpErrorBoundary<T>(
  url: string,
  fn: () => Promise<T>
): Promise<T> {
  return withErrorBoundary(fn, (error) => {
    const err = error as Error & { response?: { status: number } };
    
    if (err.response?.status) {
      return new HttpClientError(url, err.response.status, err);
    }
    
    // Network error
    return new ExternalServiceError(`HTTP: ${url}`, err);
  });
}

// Mock types
interface User {
  id: string;
  email: string;
}

const prisma = {
  user: {
    findUniqueOrThrow: async (params: { where: { id: string } }): Promise<User> => {
      throw new Error('Not found');
    },
  },
};
```

---

## Real-World Scenarios

### Scenario 1: API with Comprehensive Error Handling

```typescript
// app.ts
import express from 'express';
import { errorHandler, asyncHandler, notFoundHandler } from './middleware/error-handler';
import { ValidationError, UnauthorizedError, NotFoundError } from './errors/base';
import { setupGlobalErrorHandlers } from './global-handlers';

const app = express();
app.use(express.json());

// Request ID middleware
app.use((req, res, next) => {
  req.headers['x-request-id'] = req.headers['x-request-id'] || crypto.randomUUID();
  res.setHeader('x-request-id', req.headers['x-request-id']);
  next();
});

// Routes with proper error handling
app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await userService.getUserById(req.params.id);
  res.json(user);
}));

app.post('/users', asyncHandler(async (req, res) => {
  // Validate input
  const errors = validateCreateUserInput(req.body);
  if (errors.length > 0) {
    throw new ValidationError(errors);
  }
  
  const user = await userService.createUser(req.body);
  res.status(201).json(user);
}));

// Protected route
app.get('/profile', asyncHandler(async (req, res) => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) {
    throw new UnauthorizedError('No token provided');
  }
  
  const user = await authService.getUserFromToken(token);
  res.json(user);
}));

// 404 handler (must be before error handler)
app.use(notFoundHandler);

// Error handler (must be last)
app.use(errorHandler);

// Start server with global handlers
const server = app.listen(3000, () => {
  console.log('Server running on port 3000');
});

setupGlobalErrorHandlers(async () => {
  console.log('Shutting down gracefully...');
  await new Promise<void>((resolve) => server.close(() => resolve()));
  await db.disconnect();
});

// Mock implementations
function validateCreateUserInput(body: any): Array<{ field: string; message: string }> {
  const errors: Array<{ field: string; message: string }> = [];
  if (!body.email) errors.push({ field: 'email', message: 'Email is required' });
  return errors;
}

const userService = {
  getUserById: async (id: string) => ({ id, email: 'test@test.com' }),
  createUser: async (data: any) => ({ id: '1', ...data }),
};

const authService = {
  getUserFromToken: async (token: string) => ({ id: '1', email: 'test@test.com' }),
};

const db = {
  disconnect: async () => {},
};
```

### Scenario 2: Retry with Error Classification

```typescript
// retry-handler.ts
import { AppError, ExternalServiceError, RateLimitError } from '../errors/base';

interface RetryOptions {
  maxRetries: number;
  baseDelay: number;
  maxDelay: number;
  shouldRetry?: (error: Error, attempt: number) => boolean;
}

async function withRetry<T>(
  fn: () => Promise<T>,
  options: RetryOptions
): Promise<T> {
  const {
    maxRetries,
    baseDelay,
    maxDelay,
    shouldRetry = defaultShouldRetry,
  } = options;
  
  let lastError: Error;
  
  for (let attempt = 1; attempt <= maxRetries + 1; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      
      if (attempt > maxRetries || !shouldRetry(lastError, attempt)) {
        throw lastError;
      }
      
      // Calculate delay with exponential backoff
      let delay = Math.min(baseDelay * Math.pow(2, attempt - 1), maxDelay);
      
      // Respect Retry-After header
      if (lastError instanceof RateLimitError) {
        delay = Math.max(delay, lastError.retryAfter * 1000);
      }
      
      // Add jitter
      delay = delay * (0.5 + Math.random());
      
      console.log(`Retry ${attempt}/${maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  
  throw lastError!;
}

function defaultShouldRetry(error: Error, attempt: number): boolean {
  // Don't retry programmer errors
  if (error instanceof AppError && !error.isOperational) {
    return false;
  }
  
  // Don't retry client errors (4xx)
  if (error instanceof AppError && error.statusCode >= 400 && error.statusCode < 500) {
    // Except rate limiting
    if (!(error instanceof RateLimitError)) {
      return false;
    }
  }
  
  // Retry server errors and external service errors
  if (error instanceof ExternalServiceError) {
    return true;
  }
  
  // Retry network errors
  if ((error as any).code === 'ECONNRESET' || 
      (error as any).code === 'ETIMEDOUT') {
    return true;
  }
  
  return false;
}

// Usage
async function fetchDataWithRetry(): Promise<Data> {
  return withRetry(
    () => externalApiClient.fetchData(),
    {
      maxRetries: 3,
      baseDelay: 1000,
      maxDelay: 10000,
    }
  );
}

interface Data {
  id: string;
}

const externalApiClient = {
  fetchData: async (): Promise<Data> => ({ id: '1' }),
};
```

---

## Common Pitfalls

### 1. Swallowing Errors

```typescript
// ❌ BAD: Error silently swallowed
try {
  await riskyOperation();
} catch (error) {
  // Do nothing - error disappears!
}

// ✅ GOOD: At minimum, log the error
try {
  await riskyOperation();
} catch (error) {
  logger.error('Operation failed', { error });
  // Re-throw or handle appropriately
  throw error;
}

async function riskyOperation(): Promise<void> {}
const logger = { error: (msg: string, meta: object) => console.error(msg, meta) };
```

### 2. Exposing Internal Details

```typescript
// ❌ BAD: Exposes database details to client
app.get('/user/:id', async (req, res) => {
  try {
    const user = await db.query('SELECT * FROM users WHERE id = ?', [req.params.id]);
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: (error as Error).message }); // Exposes SQL!
  }
});

// ✅ GOOD: Transform errors appropriately
app.get('/user/:id', asyncHandler(async (req, res) => {
  const user = await userService.findById(req.params.id);
  res.json(user);
}));

// Service transforms DB errors
class UserService {
  async findById(id: string): Promise<User> {
    try {
      return await db.users.findById(id);
    } catch (error) {
      if (isNotFoundError(error)) {
        throw new NotFoundError('User', id);
      }
      throw new AppError('Failed to fetch user', { 
        cause: error as Error,
        statusCode: 500 
      });
    }
  }
}

function isNotFoundError(error: unknown): boolean {
  return false;
}
```

### 3. Not Differentiating Error Types

```typescript
// ❌ BAD: All errors treated the same
app.use((error: Error, req: Request, res: Response, next: NextFunction) => {
  res.status(500).json({ message: error.message });
});

// ✅ GOOD: Different handling for different errors
app.use((error: Error, req: Request, res: Response, next: NextFunction) => {
  if (error instanceof ValidationError) {
    return res.status(400).json({
      code: 'VALIDATION_ERROR',
      errors: error.errors,
    });
  }
  
  if (error instanceof NotFoundError) {
    return res.status(404).json({
      code: 'NOT_FOUND',
      message: error.message,
    });
  }
  
  // Unknown errors
  logger.error('Unhandled error', { error });
  res.status(500).json({
    code: 'INTERNAL_ERROR',
    message: 'An unexpected error occurred',
  });
});
```

---

## Interview Questions

### Q1: What's the difference between operational and programmer errors?

**A:** **Operational errors** are runtime problems that can happen in correctly-written programs (database down, network timeout, invalid user input). They should be handled gracefully. **Programmer errors** are bugs (null references, type errors, wrong API usage). They indicate broken code and should typically crash the process in production since the application state may be corrupted.

### Q2: Should you use exceptions or return values for errors?

**A:** Both have valid uses. Use **exceptions** for unexpected errors that require immediate handling up the call stack (I/O failures, invalid state). Use **Result types** for expected failures that callers should handle (validation, not-found scenarios). The key is consistency - don't mix approaches randomly.

### Q3: How should unhandled promise rejections be handled?

**A:** In production, unhandled rejections should trigger graceful shutdown - they indicate forgotten error handling, which could leave the app in an inconsistent state. Log the error with full context, close connections gracefully, then exit. In development, let them crash immediately to surface bugs.

### Q4: How do you handle errors across different layers (controller, service, repository)?

**A:** Each layer should catch errors from the layer below and transform them into appropriate abstractions. Repository throws database errors → Service catches and throws domain errors (NotFound, Conflict) → Controller catches and sends HTTP responses. Never let implementation details leak upward.

---

## Quick Reference Checklist

### Error Hierarchy
- [ ] Create base AppError with proper stack traces
- [ ] Define operational vs programmer errors
- [ ] Create specific error types (Validation, NotFound, etc.)
- [ ] Include error codes for client handling

### Express Middleware
- [ ] Use async wrapper for route handlers
- [ ] Create centralized error handler middleware
- [ ] Add 404 handler before error handler
- [ ] Include request ID in error responses

### Global Handlers
- [ ] Handle unhandledRejection
- [ ] Handle uncaughtException
- [ ] Trigger graceful shutdown on critical errors
- [ ] Log with full context

### Best Practices
- [ ] Never swallow errors silently
- [ ] Transform errors at layer boundaries
- [ ] Don't expose internal details in production
- [ ] Use Result types for expected failures

---

*Last updated: February 2026*

