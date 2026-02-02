# Request Validation - Complete Guide

> **MUST REMEMBER**: Never trust client input. Validate everything: body, query params, headers, and path params. Use schema validation libraries (Zod, Joi) for type-safe validation with clear error messages. Validate on entry but also sanitize - validation says "this is wrong", sanitization says "let me fix this". Different from authorization - validation checks format, authorization checks permission.

---

## How to Explain Like a Senior Developer

"Request validation is your first line of defense against bad data. I use Zod because it gives TypeScript types automatically - your validated data is properly typed without manual casting. The key is validating at the boundary: parse request data once, transform it to your internal types, then work with clean typed data. Structure your schemas to be reusable - share common patterns like email, UUID, pagination. Return clear error messages that help API consumers fix their requests without revealing internal details. And remember: validation prevents malformed data, it doesn't replace authorization."

---

## Core Implementation

### Zod Schema Validation

```typescript
// schemas/user.ts
import { z } from 'zod';

// Reusable primitives
export const emailSchema = z
  .string()
  .email('Invalid email format')
  .toLowerCase()
  .trim();

export const passwordSchema = z
  .string()
  .min(8, 'Password must be at least 8 characters')
  .max(100, 'Password too long')
  .regex(
    /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
    'Password must contain uppercase, lowercase, and number'
  );

export const uuidSchema = z.string().uuid('Invalid ID format');

// Create user schema
export const createUserSchema = z.object({
  email: emailSchema,
  password: passwordSchema,
  name: z.string().min(1).max(100).trim(),
  role: z.enum(['user', 'admin', 'moderator']).default('user'),
  metadata: z.record(z.string(), z.unknown()).optional(),
});

// Update user schema (partial, no password change here)
export const updateUserSchema = createUserSchema
  .omit({ password: true, email: true })
  .partial();

// Query params schema
export const listUsersQuerySchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  search: z.string().optional(),
  role: z.enum(['user', 'admin', 'moderator']).optional(),
  sortBy: z.enum(['name', 'createdAt', 'email']).default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).default('desc'),
});

// Type inference
export type CreateUserInput = z.infer<typeof createUserSchema>;
export type UpdateUserInput = z.infer<typeof updateUserSchema>;
export type ListUsersQuery = z.infer<typeof listUsersQuerySchema>;
```

### Express Validation Middleware

```typescript
// middleware/validate.ts
import { Request, Response, NextFunction } from 'express';
import { z, ZodError, ZodSchema } from 'zod';

interface ValidationSchemas {
  body?: ZodSchema;
  query?: ZodSchema;
  params?: ZodSchema;
  headers?: ZodSchema;
}

// Format Zod errors for API response
function formatZodError(error: ZodError): Array<{
  field: string;
  message: string;
  code: string;
}> {
  return error.errors.map((err) => ({
    field: err.path.join('.'),
    message: err.message,
    code: err.code,
  }));
}

// Validation middleware factory
export function validate(schemas: ValidationSchemas) {
  return async (req: Request, res: Response, next: NextFunction) => {
    try {
      // Validate each part of the request
      if (schemas.body) {
        req.body = schemas.body.parse(req.body);
      }
      
      if (schemas.query) {
        req.query = schemas.query.parse(req.query) as any;
      }
      
      if (schemas.params) {
        req.params = schemas.params.parse(req.params);
      }
      
      if (schemas.headers) {
        // Don't replace headers, just validate
        schemas.headers.parse(req.headers);
      }
      
      next();
    } catch (error) {
      if (error instanceof ZodError) {
        return res.status(400).json({
          error: 'Validation Error',
          message: 'Request validation failed',
          details: formatZodError(error),
        });
      }
      
      next(error);
    }
  };
}

// Typed request interface
export interface ValidatedRequest<
  TBody = unknown,
  TQuery = unknown,
  TParams = unknown
> extends Request {
  body: TBody;
  query: TQuery;
  params: TParams;
}

// Usage example
import { createUserSchema, listUsersQuerySchema } from '../schemas/user';

router.post(
  '/users',
  validate({ body: createUserSchema }),
  async (req: ValidatedRequest<CreateUserInput>, res) => {
    // req.body is fully typed
    const { email, password, name, role } = req.body;
    // ...
  }
);

router.get(
  '/users',
  validate({ query: listUsersQuerySchema }),
  async (req: ValidatedRequest<unknown, ListUsersQuery>, res) => {
    // req.query is fully typed
    const { page, limit, search, sortBy, sortOrder } = req.query;
    // ...
  }
);

import { Router } from 'express';
import { CreateUserInput, ListUsersQuery } from '../schemas/user';

const router = Router();
```

### Complex Schema Patterns

```typescript
// schemas/advanced.ts
import { z } from 'zod';

// Conditional validation
export const paymentSchema = z.discriminatedUnion('method', [
  z.object({
    method: z.literal('credit_card'),
    cardNumber: z.string().regex(/^\d{16}$/),
    expiryMonth: z.number().min(1).max(12),
    expiryYear: z.number().min(2024),
    cvv: z.string().regex(/^\d{3,4}$/),
  }),
  z.object({
    method: z.literal('bank_transfer'),
    accountNumber: z.string(),
    routingNumber: z.string(),
  }),
  z.object({
    method: z.literal('paypal'),
    email: z.string().email(),
  }),
]);

// Refinements for complex validation
export const dateRangeSchema = z.object({
  startDate: z.coerce.date(),
  endDate: z.coerce.date(),
}).refine(
  (data) => data.endDate > data.startDate,
  {
    message: 'End date must be after start date',
    path: ['endDate'],
  }
);

// Transform input
export const searchQuerySchema = z.object({
  q: z.string()
    .min(1)
    .transform((val) => val.trim().toLowerCase()),
  tags: z.string()
    .optional()
    .transform((val) => val?.split(',').map(t => t.trim()).filter(Boolean) || []),
  minPrice: z.coerce.number().optional(),
  maxPrice: z.coerce.number().optional(),
}).refine(
  (data) => {
    if (data.minPrice && data.maxPrice) {
      return data.maxPrice >= data.minPrice;
    }
    return true;
  },
  {
    message: 'maxPrice must be greater than minPrice',
    path: ['maxPrice'],
  }
);

// Array validation with items
export const bulkCreateSchema = z.object({
  items: z.array(
    z.object({
      name: z.string().min(1),
      quantity: z.number().int().positive(),
    })
  )
  .min(1, 'At least one item required')
  .max(100, 'Maximum 100 items per request'),
});

// Recursive schemas
interface Category {
  name: string;
  children?: Category[];
}

const categorySchema: z.ZodType<Category> = z.lazy(() =>
  z.object({
    name: z.string(),
    children: z.array(categorySchema).optional(),
  })
);

// Preprocess for cleaning input
export const sanitizedStringSchema = z.preprocess(
  (val) => {
    if (typeof val === 'string') {
      // Remove HTML tags, trim whitespace
      return val.replace(/<[^>]*>/g, '').trim();
    }
    return val;
  },
  z.string().min(1).max(1000)
);

// Custom error messages with context
export const passwordChangeSchema = z.object({
  currentPassword: z.string().min(1, 'Current password is required'),
  newPassword: passwordSchema,
  confirmPassword: z.string(),
}).refine(
  (data) => data.newPassword === data.confirmPassword,
  {
    message: 'Passwords do not match',
    path: ['confirmPassword'],
  }
).refine(
  (data) => data.newPassword !== data.currentPassword,
  {
    message: 'New password must be different from current password',
    path: ['newPassword'],
  }
);

const passwordSchema = z.string().min(8);
```

### Joi Validation (Alternative)

```typescript
// validation/joi-schemas.ts
import Joi from 'joi';

// Custom extensions
const customJoi = Joi.extend((joi) => ({
  type: 'string',
  base: joi.string(),
  messages: {
    'string.objectId': '{{#label}} must be a valid ObjectId',
  },
  rules: {
    objectId: {
      validate(value, helpers) {
        if (!/^[a-fA-F0-9]{24}$/.test(value)) {
          return helpers.error('string.objectId');
        }
        return value;
      },
    },
  },
}));

// User schemas
export const createUserJoiSchema = Joi.object({
  email: Joi.string().email().lowercase().trim().required(),
  password: Joi.string()
    .min(8)
    .max(100)
    .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
    .required()
    .messages({
      'string.pattern.base': 'Password must contain uppercase, lowercase, and number',
    }),
  name: Joi.string().min(1).max(100).trim().required(),
  role: Joi.string().valid('user', 'admin', 'moderator').default('user'),
  metadata: Joi.object().pattern(Joi.string(), Joi.any()).optional(),
});

// Query schema with defaults
export const paginationJoiSchema = Joi.object({
  page: Joi.number().integer().min(1).default(1),
  limit: Joi.number().integer().min(1).max(100).default(20),
  sortBy: Joi.string().valid('createdAt', 'updatedAt', 'name').default('createdAt'),
  sortOrder: Joi.string().valid('asc', 'desc').default('desc'),
});

// Joi middleware
export function validateJoi(schema: Joi.Schema, property: 'body' | 'query' | 'params' = 'body') {
  return (req: Request, res: Response, next: NextFunction) => {
    const { error, value } = schema.validate(req[property], {
      abortEarly: false,
      stripUnknown: true,
    });
    
    if (error) {
      const details = error.details.map((d) => ({
        field: d.path.join('.'),
        message: d.message,
      }));
      
      return res.status(400).json({
        error: 'Validation Error',
        details,
      });
    }
    
    req[property] = value;
    next();
  };
}

import { Request, Response, NextFunction } from 'express';
```

### File Validation

```typescript
// validation/file-schemas.ts
import { z } from 'zod';
import { Request } from 'express';

// File validation schema
export const fileSchema = z.object({
  fieldname: z.string(),
  originalname: z.string(),
  encoding: z.string(),
  mimetype: z.string(),
  size: z.number(),
  buffer: z.instanceof(Buffer).optional(),
  path: z.string().optional(),
});

// Image upload validation
export const imageUploadSchema = fileSchema.extend({
  mimetype: z.enum([
    'image/jpeg',
    'image/png',
    'image/webp',
    'image/gif',
  ]),
  size: z.number().max(5 * 1024 * 1024, 'Image must be less than 5MB'),
});

// Document upload validation
export const documentUploadSchema = fileSchema.extend({
  mimetype: z.enum([
    'application/pdf',
    'application/msword',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
  ]),
  size: z.number().max(10 * 1024 * 1024, 'Document must be less than 10MB'),
});

// Validate uploaded file
export function validateFile(
  file: Express.Multer.File | undefined,
  schema: z.ZodSchema
): { success: true; data: Express.Multer.File } | { success: false; error: string } {
  if (!file) {
    return { success: false, error: 'No file uploaded' };
  }
  
  const result = schema.safeParse(file);
  
  if (!result.success) {
    const message = result.error.errors[0]?.message || 'Invalid file';
    return { success: false, error: message };
  }
  
  return { success: true, data: file };
}

// Multiple files validation
export function validateFiles(
  files: Express.Multer.File[] | undefined,
  schema: z.ZodSchema,
  options: { min?: number; max?: number } = {}
): { success: true; data: Express.Multer.File[] } | { success: false; error: string } {
  if (!files || files.length === 0) {
    return { success: false, error: 'No files uploaded' };
  }
  
  if (options.min && files.length < options.min) {
    return { success: false, error: `At least ${options.min} files required` };
  }
  
  if (options.max && files.length > options.max) {
    return { success: false, error: `Maximum ${options.max} files allowed` };
  }
  
  for (const file of files) {
    const result = schema.safeParse(file);
    if (!result.success) {
      return { 
        success: false, 
        error: `Invalid file ${file.originalname}: ${result.error.errors[0]?.message}` 
      };
    }
  }
  
  return { success: true, data: files };
}
```

---

## Real-World Scenarios

### Scenario 1: API with Full Request Validation

```typescript
// routes/products.ts
import { Router } from 'express';
import { z } from 'zod';
import { validate, ValidatedRequest } from '../middleware/validate';

const router = Router();

// Schemas
const productIdSchema = z.object({
  id: z.string().uuid(),
});

const createProductSchema = z.object({
  name: z.string().min(1).max(200),
  description: z.string().max(5000).optional(),
  price: z.number().positive().multipleOf(0.01),
  currency: z.enum(['USD', 'EUR', 'GBP']).default('USD'),
  category: z.string(),
  tags: z.array(z.string()).max(10).default([]),
  inventory: z.object({
    quantity: z.number().int().min(0),
    sku: z.string().optional(),
  }),
  images: z.array(z.string().url()).max(10).default([]),
  active: z.boolean().default(true),
});

const updateProductSchema = createProductSchema.partial();

const listProductsSchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  category: z.string().optional(),
  minPrice: z.coerce.number().optional(),
  maxPrice: z.coerce.number().optional(),
  search: z.string().optional(),
  active: z.coerce.boolean().optional(),
  sortBy: z.enum(['name', 'price', 'createdAt']).default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).default('desc'),
});

type CreateProduct = z.infer<typeof createProductSchema>;
type UpdateProduct = z.infer<typeof updateProductSchema>;
type ListProducts = z.infer<typeof listProductsSchema>;
type ProductId = z.infer<typeof productIdSchema>;

// Routes
router.get(
  '/',
  validate({ query: listProductsSchema }),
  async (req: ValidatedRequest<unknown, ListProducts>, res) => {
    const { page, limit, category, minPrice, maxPrice, search, active, sortBy, sortOrder } = req.query;
    
    const products = await productService.list({
      page,
      limit,
      filters: { category, minPrice, maxPrice, search, active },
      sort: { field: sortBy, order: sortOrder },
    });
    
    res.json(products);
  }
);

router.post(
  '/',
  validate({ body: createProductSchema }),
  async (req: ValidatedRequest<CreateProduct>, res) => {
    const product = await productService.create(req.body);
    res.status(201).json(product);
  }
);

router.get(
  '/:id',
  validate({ params: productIdSchema }),
  async (req: ValidatedRequest<unknown, unknown, ProductId>, res) => {
    const product = await productService.findById(req.params.id);
    res.json(product);
  }
);

router.patch(
  '/:id',
  validate({ params: productIdSchema, body: updateProductSchema }),
  async (req: ValidatedRequest<UpdateProduct, unknown, ProductId>, res) => {
    const product = await productService.update(req.params.id, req.body);
    res.json(product);
  }
);

const productService = {
  list: async (params: any) => [],
  create: async (data: any) => data,
  findById: async (id: string) => null,
  update: async (id: string, data: any) => data,
};

export { router as productRouter };
```

### Scenario 2: Webhook Payload Validation

```typescript
// webhooks/stripe.ts
import { Router, Request, Response } from 'express';
import { z } from 'zod';
import Stripe from 'stripe';

const router = Router();

// Stripe webhook event schemas
const stripeCheckoutCompletedSchema = z.object({
  id: z.string(),
  object: z.literal('event'),
  type: z.literal('checkout.session.completed'),
  data: z.object({
    object: z.object({
      id: z.string(),
      customer: z.string().nullable(),
      customer_email: z.string().email().nullable(),
      payment_status: z.enum(['paid', 'unpaid', 'no_payment_required']),
      amount_total: z.number(),
      currency: z.string(),
      metadata: z.record(z.string()).optional(),
    }),
  }),
});

const stripeInvoicePaidSchema = z.object({
  id: z.string(),
  object: z.literal('event'),
  type: z.literal('invoice.paid'),
  data: z.object({
    object: z.object({
      id: z.string(),
      customer: z.string(),
      subscription: z.string().nullable(),
      amount_paid: z.number(),
      currency: z.string(),
    }),
  }),
});

// Union of all webhook types
const stripeWebhookSchema = z.discriminatedUnion('type', [
  stripeCheckoutCompletedSchema,
  stripeInvoicePaidSchema,
]);

router.post(
  '/stripe',
  // Raw body needed for signature verification
  async (req: Request, res: Response) => {
    const sig = req.headers['stripe-signature'];
    
    if (!sig) {
      return res.status(400).json({ error: 'Missing signature' });
    }
    
    // Verify webhook signature
    let event: Stripe.Event;
    try {
      event = stripe.webhooks.constructEvent(
        req.body,
        sig,
        process.env.STRIPE_WEBHOOK_SECRET!
      );
    } catch (err) {
      return res.status(400).json({ error: 'Invalid signature' });
    }
    
    // Validate event structure
    const result = stripeWebhookSchema.safeParse(event);
    if (!result.success) {
      console.warn('Unhandled webhook event type:', event.type);
      return res.json({ received: true }); // Acknowledge unknown events
    }
    
    // Handle validated event
    const validatedEvent = result.data;
    
    switch (validatedEvent.type) {
      case 'checkout.session.completed':
        await handleCheckoutCompleted(validatedEvent.data.object);
        break;
        
      case 'invoice.paid':
        await handleInvoicePaid(validatedEvent.data.object);
        break;
    }
    
    res.json({ received: true });
  }
);

async function handleCheckoutCompleted(session: any): Promise<void> {
  console.log('Checkout completed:', session.id);
}

async function handleInvoicePaid(invoice: any): Promise<void> {
  console.log('Invoice paid:', invoice.id);
}

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export { router as stripeWebhookRouter };
```

---

## Common Pitfalls

### 1. Not Validating All Input Sources

```typescript
// ❌ BAD: Only validating body
router.get('/users/:id', (req, res) => {
  const userId = req.params.id; // Could be anything!
  const page = req.query.page; // Could be "abc" or negative
});

// ✅ GOOD: Validate everything
router.get(
  '/users/:id',
  validate({
    params: z.object({ id: z.string().uuid() }),
    query: z.object({ page: z.coerce.number().int().min(1).default(1) }),
  }),
  (req, res) => {
    // Now typed and validated
  }
);
```

### 2. Trusting Coerced Values Without Limits

```typescript
// ❌ BAD: No limits on coerced number
const schema = z.object({
  limit: z.coerce.number(), // Could be Infinity or very large
});

// ✅ GOOD: Always set bounds
const schema = z.object({
  limit: z.coerce.number().int().min(1).max(100).default(20),
});
```

### 3. Exposing Validation Internals

```typescript
// ❌ BAD: Exposing schema details
res.status(400).json({
  error: zodError.errors, // Includes internal paths, codes
});

// ✅ GOOD: Clean error format
res.status(400).json({
  error: 'Validation Error',
  details: zodError.errors.map(e => ({
    field: e.path.join('.'),
    message: e.message,
  })),
});
```

---

## Interview Questions

### Q1: Why use Zod over manual validation?

**A:** Zod provides: 1) Type inference - validated data is automatically typed, 2) Composability - build complex schemas from simple ones, 3) Consistent error format, 4) Built-in transformations and refinements, 5) No runtime overhead for types. Manual validation requires maintaining types separately and is prone to drift.

### Q2: What's the difference between validation and sanitization?

**A:** Validation rejects invalid input ("this email is invalid"). Sanitization modifies input to be safe/normalized ("trim whitespace, lowercase email"). Use both: validate first to reject malformed data, then sanitize valid data to normalize it (Zod's `transform` does both).

### Q3: How do you validate nested objects with optional fields?

**A:** Use Zod's `.partial()` for making all fields optional, `.deepPartial()` for nested objects, or explicitly mark fields with `.optional()`. For conditional fields, use `.refine()` or discriminated unions.

### Q4: Should validation happen in middleware or controllers?

**A:** Middleware is better - it separates concerns, runs before business logic, and allows consistent error handling. Define schemas in a separate file, create validation middleware, then compose it with routes. The controller receives pre-validated, typed data.

---

## Quick Reference Checklist

### Validation Coverage
- [ ] Request body
- [ ] Query parameters
- [ ] Path parameters
- [ ] Headers (for auth tokens, API keys)
- [ ] File uploads

### Schema Best Practices
- [ ] Use TypeScript inference from schemas
- [ ] Create reusable primitive schemas
- [ ] Set sensible defaults
- [ ] Add bounds to numbers (min/max)
- [ ] Trim and normalize strings
- [ ] Validate enums strictly

### Error Handling
- [ ] Return consistent error format
- [ ] Include field names in errors
- [ ] Don't expose internal details
- [ ] Log validation failures for debugging

### Security
- [ ] Strip unknown fields
- [ ] Limit string lengths
- [ ] Limit array sizes
- [ ] Validate file types and sizes

---

*Last updated: February 2026*

