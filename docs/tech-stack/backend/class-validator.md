# class-validator & class-transformer

> Decorator-based validation for NestJS DTOs.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | Validation library using decorators |
| **Why** | Type-safe request validation, automatic transformation |
| **Packages** | `class-validator`, `class-transformer` |
| **Location** | `apps/api/src/**/dto/` |

## Setup

```typescript
// main.ts
import { ValidationPipe } from '@nestjs/common';

app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,           // Strip non-decorated properties
    forbidNonWhitelisted: true, // Throw on extra properties
    transform: true,           // Auto-transform to DTO types
    transformOptions: {
      enableImplicitConversion: true, // Convert query strings to types
    },
  }),
);
```

## Basic Validators

```typescript
import {
  IsString,
  IsEmail,
  IsNumber,
  IsBoolean,
  IsDate,
  IsUUID,
  IsArray,
  IsEnum,
  IsOptional,
  IsNotEmpty,
  MinLength,
  MaxLength,
  Min,
  Max,
  Matches,
  ArrayMinSize,
  ArrayMaxSize,
} from 'class-validator';

export class CreateUserDto {
  @IsEmail({}, { message: 'Please provide a valid email address' })
  email: string;

  @IsString()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  @MaxLength(128)
  @Matches(/[A-Z]/, { message: 'Password must contain uppercase letter' })
  @Matches(/[a-z]/, { message: 'Password must contain lowercase letter' })
  @Matches(/[0-9]/, { message: 'Password must contain a number' })
  password: string;

  @IsString()
  @MinLength(2)
  @MaxLength(100)
  fullName: string;

  @IsOptional()
  @IsString()
  phone?: string;
}
```

## Common Patterns

### UUID Validation
```typescript
export class GetMedicationDto {
  @IsUUID('4', { message: 'Invalid medication ID' })
  id: string;
}
```

### Enum Validation
```typescript
export class CreateMedicationDto {
  @IsEnum(MedicationForm, { message: 'Invalid medication form' })
  form: MedicationForm;

  @IsEnum(MedicationFrequency)
  frequency: MedicationFrequency;
}
```

### Array Validation
```typescript
export class CreateMedicationDto {
  @IsArray()
  @ArrayMinSize(1, { message: 'At least one scheduled time required' })
  @ArrayMaxSize(10)
  @IsString({ each: true })
  @Matches(/^([01]\d|2[0-3]):([0-5]\d)$/, {
    each: true,
    message: 'Times must be in HH:mm format',
  })
  scheduledTimes: string[];
}
```

### Nested Object Validation
```typescript
import { Type, ValidateNested } from 'class-transformer';

class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;

  @IsString()
  @Matches(/^\d{5}(-\d{4})?$/)
  zipCode: string;
}

export class CreateCareRecipientDto {
  @IsString()
  name: string;

  @ValidateNested()
  @Type(() => AddressDto)
  address: AddressDto;
}
```

### Optional Fields
```typescript
export class UpdateMedicationDto {
  @IsOptional()
  @IsString()
  name?: string;

  @IsOptional()
  @IsNumber()
  @Min(0)
  currentSupply?: number;
}
```

### Date Validation
```typescript
import { IsDateString, IsISO8601 } from 'class-validator';
import { Type } from 'class-transformer';

export class CreateAppointmentDto {
  @IsISO8601()
  dateTime: string;

  // Or transform to Date object
  @Type(() => Date)
  @IsDate()
  scheduledDate: Date;
}
```

## Custom Validators

### Create Custom Decorator
```typescript
// validators/is-future-date.validator.ts
import {
  registerDecorator,
  ValidationOptions,
  ValidationArguments,
} from 'class-validator';

export function IsFutureDate(validationOptions?: ValidationOptions) {
  return function (object: Object, propertyName: string) {
    registerDecorator({
      name: 'isFutureDate',
      target: object.constructor,
      propertyName: propertyName,
      options: validationOptions,
      validator: {
        validate(value: any) {
          return value instanceof Date && value > new Date();
        },
        defaultMessage(args: ValidationArguments) {
          return `${args.property} must be a future date`;
        },
      },
    });
  };
}

// Usage
export class CreateAppointmentDto {
  @Type(() => Date)
  @IsDate()
  @IsFutureDate({ message: 'Appointment must be scheduled in the future' })
  dateTime: Date;
}
```

### Conditional Validation
```typescript
import { ValidateIf } from 'class-validator';

export class CreateMedicationDto {
  @IsEnum(MedicationFrequency)
  frequency: MedicationFrequency;

  // Only required if frequency is CUSTOM
  @ValidateIf(o => o.frequency === MedicationFrequency.CUSTOM)
  @IsString()
  customSchedule?: string;
}
```

### Cross-Field Validation
```typescript
import { Validate, ValidatorConstraint, ValidatorConstraintInterface } from 'class-validator';

@ValidatorConstraint({ name: 'passwordsMatch', async: false })
class PasswordsMatchConstraint implements ValidatorConstraintInterface {
  validate(confirmPassword: string, args: ValidationArguments) {
    const [relatedPropertyName] = args.constraints;
    const relatedValue = (args.object as any)[relatedPropertyName];
    return confirmPassword === relatedValue;
  }

  defaultMessage() {
    return 'Passwords do not match';
  }
}

export class RegisterDto {
  @IsString()
  @MinLength(8)
  password: string;

  @Validate(PasswordsMatchConstraint, ['password'])
  confirmPassword: string;
}
```

## Transformation

```typescript
import { Transform, Type, Expose, Exclude } from 'class-transformer';

export class CreateMedicationDto {
  // Trim whitespace
  @Transform(({ value }) => value?.trim())
  @IsString()
  name: string;

  // Convert string to lowercase
  @Transform(({ value }) => value?.toLowerCase())
  @IsEmail()
  email: string;

  // Parse boolean from query string
  @Transform(({ value }) => value === 'true')
  @IsBoolean()
  isActive: boolean;

  // Parse number from query string
  @Type(() => Number)
  @IsNumber()
  @Min(0)
  supply: number;
}
```

## Response Transformation

```typescript
import { Exclude, Expose, Transform } from 'class-transformer';

export class UserResponseDto {
  @Expose()
  id: string;

  @Expose()
  email: string;

  @Expose()
  fullName: string;

  // Never include in response
  @Exclude()
  password: string;

  @Exclude()
  passwordResetToken: string;

  // Transform date to ISO string
  @Expose()
  @Transform(({ value }) => value?.toISOString())
  createdAt: Date;
}

// Use with plainToInstance
const response = plainToInstance(UserResponseDto, user, {
  excludeExtraneousValues: true,
});
```

## Error Messages

```typescript
export class CreateMedicationDto {
  @IsString({ message: 'Medication name must be a string' })
  @IsNotEmpty({ message: 'Medication name is required' })
  @MinLength(1, { message: 'Medication name cannot be empty' })
  @MaxLength(100, { message: 'Medication name cannot exceed 100 characters' })
  name: string;
}

// Custom message with value
@Min(0, { message: 'Supply ($value) cannot be negative' })
currentSupply: number;
```

## Validation Groups

```typescript
export class UserDto {
  @IsString({ groups: ['create', 'update'] })
  name: string;

  @IsEmail({}, { groups: ['create'] })
  email: string; // Only validated on create
}

// Usage in controller
@Post()
create(@Body(new ValidationPipe({ groups: ['create'] })) dto: UserDto) { ... }

@Patch(':id')
update(@Body(new ValidationPipe({ groups: ['update'] })) dto: UserDto) { ... }
```

## Troubleshooting

### Validation Not Working
- Ensure `ValidationPipe` is registered
- Check decorator is from `class-validator`
- Verify property has decorator

### Transform Not Working
- Enable `transform: true` in ValidationPipe
- Use `@Type()` for nested objects
- Check `enableImplicitConversion`

### Extra Properties Not Stripped
- Set `whitelist: true`
- Add `@ApiProperty()` for Swagger DTOs

---

*See also: [NestJS](nestjs.md), [Swagger](swagger.md), [Zod](../frontend/zod.md)*


