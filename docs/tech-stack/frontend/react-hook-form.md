# React Hook Form

> Performant, flexible form handling for React.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | Form state management library |
| **Why** | Performance, minimal re-renders, validation |
| **Version** | 7.x |
| **Location** | `apps/web/src/components/forms/` |

## Basic Usage

### Simple Form
```tsx
import { useForm } from 'react-hook-form';

interface LoginForm {
  email: string;
  password: string;
}

function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<LoginForm>();

  const onSubmit = async (data: LoginForm) => {
    await api.login(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input
          {...register('email', { required: 'Email is required' })}
          type="email"
          placeholder="Email"
        />
        {errors.email && <span className="text-red-500">{errors.email.message}</span>}
      </div>

      <div>
        <input
          {...register('password', { required: 'Password is required' })}
          type="password"
          placeholder="Password"
        />
        {errors.password && <span className="text-red-500">{errors.password.message}</span>}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

## With Zod Validation

### Schema Definition
```typescript
// schemas/medication.ts
import { z } from 'zod';

export const createMedicationSchema = z.object({
  name: z.string().min(1, 'Name is required'),
  dosage: z.string().min(1, 'Dosage is required'),
  form: z.enum(['TABLET', 'CAPSULE', 'LIQUID', 'INJECTION', 'PATCH', 'OTHER']),
  frequency: z.enum(['ONCE_DAILY', 'TWICE_DAILY', 'THREE_TIMES_DAILY', 'AS_NEEDED']),
  scheduledTimes: z.array(z.string()).min(1, 'At least one time required'),
  instructions: z.string().optional(),
  currentSupply: z.number().min(0).optional(),
  refillAt: z.number().min(0).optional(),
});

export type CreateMedicationInput = z.infer<typeof createMedicationSchema>;
```

### Form with Zod Resolver
```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { createMedicationSchema, CreateMedicationInput } from '@/schemas/medication';

function CreateMedicationForm() {
  const {
    register,
    handleSubmit,
    control,
    formState: { errors, isSubmitting },
  } = useForm<CreateMedicationInput>({
    resolver: zodResolver(createMedicationSchema),
    defaultValues: {
      form: 'TABLET',
      frequency: 'ONCE_DAILY',
      scheduledTimes: ['08:00'],
    },
  });

  const onSubmit = async (data: CreateMedicationInput) => {
    await api.createMedication(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Form fields */}
    </form>
  );
}
```

## Form Components

### Reusable Input
```tsx
// components/forms/Input.tsx
import { UseFormRegister, FieldError } from 'react-hook-form';

interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  name: string;
  register: UseFormRegister<any>;
  error?: FieldError;
  required?: boolean;
}

export function Input({ label, name, register, error, required, ...props }: InputProps) {
  return (
    <div className="space-y-1">
      <label htmlFor={name} className="block text-sm font-medium text-gray-700">
        {label} {required && <span className="text-red-500">*</span>}
      </label>
      <input
        id={name}
        {...register(name)}
        {...props}
        className={`w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-primary-500 ${
          error ? 'border-red-500' : 'border-gray-300'
        }`}
      />
      {error && <p className="text-sm text-red-500">{error.message}</p>}
    </div>
  );
}
```

### Reusable Select
```tsx
// components/forms/Select.tsx
import { UseFormRegister, FieldError } from 'react-hook-form';

interface Option {
  value: string;
  label: string;
}

interface SelectProps {
  label: string;
  name: string;
  options: Option[];
  register: UseFormRegister<any>;
  error?: FieldError;
}

export function Select({ label, name, options, register, error }: SelectProps) {
  return (
    <div className="space-y-1">
      <label className="block text-sm font-medium text-gray-700">{label}</label>
      <select
        {...register(name)}
        className={`w-full px-4 py-2 border rounded-lg ${
          error ? 'border-red-500' : 'border-gray-300'
        }`}
      >
        {options.map((opt) => (
          <option key={opt.value} value={opt.value}>
            {opt.label}
          </option>
        ))}
      </select>
      {error && <p className="text-sm text-red-500">{error.message}</p>}
    </div>
  );
}
```

## Controlled Components

### With Controller
```tsx
import { useForm, Controller } from 'react-hook-form';
import { DatePicker } from '@/components/ui/date-picker';

function AppointmentForm() {
  const { control, handleSubmit } = useForm();

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="appointmentDate"
        control={control}
        rules={{ required: 'Date is required' }}
        render={({ field, fieldState: { error } }) => (
          <div>
            <DatePicker
              value={field.value}
              onChange={field.onChange}
            />
            {error && <span className="text-red-500">{error.message}</span>}
          </div>
        )}
      />
    </form>
  );
}
```

### Radix UI Select
```tsx
import { Controller } from 'react-hook-form';
import * as Select from '@radix-ui/react-select';

<Controller
  name="role"
  control={control}
  render={({ field }) => (
    <Select.Root value={field.value} onValueChange={field.onChange}>
      <Select.Trigger>
        <Select.Value placeholder="Select role" />
      </Select.Trigger>
      <Select.Portal>
        <Select.Content>
          <Select.Item value="ADMIN">Admin</Select.Item>
          <Select.Item value="CAREGIVER">Caregiver</Select.Item>
          <Select.Item value="VIEWER">Viewer</Select.Item>
        </Select.Content>
      </Select.Portal>
    </Select.Root>
  )}
/>
```

## Array Fields

### Dynamic Fields with useFieldArray
```tsx
import { useForm, useFieldArray } from 'react-hook-form';

interface MedicationForm {
  name: string;
  scheduledTimes: { time: string }[];
}

function MedicationTimeForm() {
  const { register, control, handleSubmit } = useForm<MedicationForm>({
    defaultValues: {
      scheduledTimes: [{ time: '08:00' }],
    },
  });

  const { fields, append, remove } = useFieldArray({
    control,
    name: 'scheduledTimes',
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {fields.map((field, index) => (
        <div key={field.id} className="flex gap-2">
          <input
            {...register(`scheduledTimes.${index}.time`)}
            type="time"
          />
          <button type="button" onClick={() => remove(index)}>
            Remove
          </button>
        </div>
      ))}
      <button type="button" onClick={() => append({ time: '' })}>
        Add Time
      </button>
    </form>
  );
}
```

## Form State

### Watch Values
```tsx
const { watch, register } = useForm();

// Watch single field
const frequency = watch('frequency');

// Watch multiple fields
const [name, dosage] = watch(['name', 'dosage']);

// Watch all fields
const allValues = watch();

// Conditional rendering based on value
{frequency === 'AS_NEEDED' && (
  <input {...register('maxDailyDoses')} placeholder="Max daily doses" />
)}
```

### Form State Values
```tsx
const {
  formState: {
    errors,        // Validation errors
    isSubmitting,  // Submit in progress
    isValid,       // All validations pass
    isDirty,       // Form has been modified
    dirtyFields,   // Which fields were modified
    touchedFields, // Which fields were touched
    submitCount,   // Number of submit attempts
  }
} = useForm();
```

## Error Handling

### Set Errors Manually
```tsx
const { setError, clearErrors } = useForm();

// After API error
try {
  await api.createMedication(data);
} catch (error) {
  if (error.response?.data?.field) {
    setError(error.response.data.field, {
      type: 'server',
      message: error.response.data.message,
    });
  } else {
    setError('root', {
      type: 'server',
      message: 'An error occurred',
    });
  }
}

// Display root error
{errors.root && (
  <div className="bg-red-50 text-red-500 p-4 rounded">
    {errors.root.message}
  </div>
)}
```

## Reset & Default Values

```tsx
const { reset, setValue, getValues } = useForm({
  defaultValues: {
    name: '',
    dosage: '',
  },
});

// Reset to defaults
reset();

// Reset to specific values
reset({ name: 'New Name', dosage: '10mg' });

// Set single value
setValue('name', 'Updated Name');

// Get current values
const currentValues = getValues();
```

## Troubleshooting

### Form Not Submitting
- Check `handleSubmit` is properly called
- Verify validation is passing
- Check for `event.preventDefault()` issues

### Validation Not Working
- Ensure `resolver` is configured correctly
- Check field names match schema
- Verify Zod schema is valid

### Re-render Issues
- Use `watch` sparingly
- Consider `useWatch` for specific fields
- Check if controlled components are needed

---

*See also: [Zod](zod.md), [React](react.md)*


