# Zod

> TypeScript-first schema validation library.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | Schema validation library |
| **Why** | Type inference, composable schemas, great errors |
| **Version** | 3.x |
| **Location** | `apps/web/src/schemas/`, `packages/config/` |

## Basic Usage

### Primitive Types
```typescript
import { z } from 'zod';

// Strings
const nameSchema = z.string();
const emailSchema = z.string().email();
const uuidSchema = z.string().uuid();

// Numbers
const ageSchema = z.number().positive().int();
const priceSchema = z.number().min(0).max(1000000);

// Booleans
const isActiveSchema = z.boolean();

// Dates
const birthDateSchema = z.date();
const dateStringSchema = z.string().datetime();

// Enums
const roleSchema = z.enum(['ADMIN', 'CAREGIVER', 'VIEWER']);
```

### Objects
```typescript
const userSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  fullName: z.string().min(2).max(100),
  role: z.enum(['ADMIN', 'CAREGIVER', 'VIEWER']),
  createdAt: z.date(),
});

// Infer TypeScript type
type User = z.infer<typeof userSchema>;
// { id: string; email: string; fullName: string; role: 'ADMIN' | 'CAREGIVER' | 'VIEWER'; createdAt: Date; }
```

### Arrays
```typescript
const tagsSchema = z.array(z.string()).min(1).max(10);
const numbersSchema = z.array(z.number()).nonempty();
const medicationsSchema = z.array(medicationSchema);
```

## Real-World Schemas

### Authentication
```typescript
// schemas/auth.ts
export const loginSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  rememberMe: z.boolean().optional(),
});

export const registerSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Must contain uppercase letter')
    .regex(/[a-z]/, 'Must contain lowercase letter')
    .regex(/[0-9]/, 'Must contain number'),
  confirmPassword: z.string(),
  fullName: z.string().min(2, 'Name is required'),
}).refine((data) => data.password === data.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword'],
});

export type LoginInput = z.infer<typeof loginSchema>;
export type RegisterInput = z.infer<typeof registerSchema>;
```

### Medication
```typescript
// schemas/medication.ts
export const medicationFormSchema = z.enum([
  'TABLET',
  'CAPSULE',
  'LIQUID',
  'INJECTION',
  'PATCH',
  'CREAM',
  'DROPS',
  'INHALER',
  'OTHER',
]);

export const medicationFrequencySchema = z.enum([
  'ONCE_DAILY',
  'TWICE_DAILY',
  'THREE_TIMES_DAILY',
  'FOUR_TIMES_DAILY',
  'AS_NEEDED',
  'WEEKLY',
  'CUSTOM',
]);

export const createMedicationSchema = z.object({
  name: z.string().min(1, 'Medication name is required'),
  genericName: z.string().optional(),
  dosage: z.string().min(1, 'Dosage is required'),
  form: medicationFormSchema,
  frequency: medicationFrequencySchema,
  scheduledTimes: z.array(z.string().regex(/^([01]\d|2[0-3]):([0-5]\d)$/, 'Invalid time format')).min(1),
  instructions: z.string().optional(),
  prescribedBy: z.string().optional(),
  pharmacy: z.string().optional(),
  currentSupply: z.number().int().min(0).optional(),
  refillAt: z.number().int().min(0).optional(),
  startDate: z.string().datetime().optional(),
  endDate: z.string().datetime().optional().nullable(),
});

export type CreateMedicationInput = z.infer<typeof createMedicationSchema>;
```

### API Response
```typescript
// schemas/api.ts
const paginationSchema = z.object({
  page: z.number().int().positive(),
  limit: z.number().int().positive().max(100),
  total: z.number().int().nonnegative(),
  totalPages: z.number().int().nonnegative(),
});

const apiResponseSchema = <T extends z.ZodType>(dataSchema: T) =>
  z.object({
    success: z.boolean(),
    data: dataSchema,
    message: z.string().optional(),
    pagination: paginationSchema.optional(),
  });

// Usage
const medicationsResponseSchema = apiResponseSchema(z.array(medicationSchema));
```

## Validation

### Parse vs SafeParse
```typescript
// parse() - throws on error
try {
  const user = userSchema.parse(data);
  // user is typed correctly
} catch (error) {
  if (error instanceof z.ZodError) {
    console.log(error.errors);
  }
}

// safeParse() - returns result object
const result = userSchema.safeParse(data);
if (result.success) {
  console.log(result.data); // typed correctly
} else {
  console.log(result.error.errors);
}
```

### Error Formatting
```typescript
const result = schema.safeParse(data);

if (!result.success) {
  // Flat error format
  const flatErrors = result.error.flatten();
  // { formErrors: string[], fieldErrors: { [field]: string[] } }

  // Formatted errors
  const formatted = result.error.format();
  // { _errors: string[], fieldName: { _errors: string[] } }

  // Custom format
  const custom = result.error.errors.map(err => ({
    field: err.path.join('.'),
    message: err.message,
  }));
}
```

## Transformations

### Transform
```typescript
const dateSchema = z.string().transform((str) => new Date(str));

const lowercaseEmailSchema = z.string().email().transform((email) => email.toLowerCase());

const trimmedSchema = z.string().transform((str) => str.trim());
```

### Preprocess
```typescript
// Coerce string to number
const numberSchema = z.preprocess(
  (val) => (typeof val === 'string' ? parseInt(val, 10) : val),
  z.number()
);

// Handle empty strings
const optionalStringSchema = z.preprocess(
  (val) => (val === '' ? undefined : val),
  z.string().optional()
);
```

### Default & Catch
```typescript
// Default value
const roleSchema = z.enum(['ADMIN', 'CAREGIVER', 'VIEWER']).default('VIEWER');

// Catch (fallback on parse error)
const safeNumberSchema = z.number().catch(0);
```

## Advanced Patterns

### Discriminated Unions
```typescript
const notificationSchema = z.discriminatedUnion('type', [
  z.object({
    type: z.literal('MEDICATION_REMINDER'),
    medicationId: z.string().uuid(),
    scheduledTime: z.string(),
  }),
  z.object({
    type: z.literal('APPOINTMENT_REMINDER'),
    appointmentId: z.string().uuid(),
    appointmentDate: z.string().datetime(),
  }),
  z.object({
    type: z.literal('EMERGENCY_ALERT'),
    alertId: z.string().uuid(),
    severity: z.enum(['LOW', 'MEDIUM', 'HIGH', 'CRITICAL']),
  }),
]);
```

### Recursive Types
```typescript
interface Comment {
  id: string;
  text: string;
  replies: Comment[];
}

const commentSchema: z.ZodType<Comment> = z.lazy(() =>
  z.object({
    id: z.string().uuid(),
    text: z.string(),
    replies: z.array(commentSchema),
  })
);
```

### Extend & Merge
```typescript
const baseSchema = z.object({
  id: z.string().uuid(),
  createdAt: z.date(),
  updatedAt: z.date(),
});

const medicationSchema = baseSchema.extend({
  name: z.string(),
  dosage: z.string(),
});

// Or merge two schemas
const mergedSchema = schema1.merge(schema2);
```

### Partial & Required
```typescript
const fullSchema = z.object({
  name: z.string(),
  email: z.string().email(),
  phone: z.string(),
});

// All fields optional
const partialSchema = fullSchema.partial();

// Some fields optional
const createSchema = fullSchema.partial({ phone: true });

// Make optional fields required
const requiredSchema = partialSchema.required();
```

## Integration with React Hook Form

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

type FormData = z.infer<typeof schema>;

function Form() {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input {...register('password')} type="password" />
      {errors.password && <span>{errors.password.message}</span>}
    </form>
  );
}
```

## Environment Validation

```typescript
// packages/config/src/env.ts
const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.string().transform(Number).pipe(z.number().positive()),
  DATABASE_URL: z.string().url(),
  REDIS_HOST: z.string(),
  REDIS_PORT: z.string().transform(Number).pipe(z.number().positive()),
  JWT_SECRET: z.string().min(32),
});

export const env = envSchema.parse(process.env);
```

## Troubleshooting

### Type Inference Not Working
```typescript
// Ensure you're using z.infer
type MyType = z.infer<typeof mySchema>;

// For complex types, use explicit annotation
const schema: z.ZodType<MyInterface> = z.lazy(() => ...);
```

### Validation Too Strict
```typescript
// Use .passthrough() to allow extra fields
const flexibleSchema = schema.passthrough();

// Or .strip() to remove extra fields (default)
const strictSchema = schema.strict(); // Error on extra fields
```

---

*See also: [React Hook Form](react-hook-form.md), [class-validator](../backend/class-validator.md)*


