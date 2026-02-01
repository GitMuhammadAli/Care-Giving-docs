# Swagger / OpenAPI

> Interactive API documentation with automatic generation.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | OpenAPI documentation generator |
| **Why** | API documentation, testing, client generation |
| **Package** | `@nestjs/swagger` |
| **Location** | `apps/api/src/main.ts` |

## Setup

```typescript
// main.ts
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Swagger configuration
  const config = new DocumentBuilder()
    .setTitle('CareCircle API')
    .setDescription(`
## Overview
CareCircle is a family caregiving coordination platform.

## Authentication
Most endpoints require JWT authentication. To authenticate:
1. Login via \`POST /auth/login\`
2. Copy the \`accessToken\` from the response
3. Click **Authorize** button above
4. Paste token and click **Authorize**

## Rate Limiting
- 100 requests per minute per IP (general)
- 10 login attempts per minute
- 5 registration attempts per minute
    `)
    .setVersion('1.0')
    .addBearerAuth(
      {
        type: 'http',
        scheme: 'bearer',
        bearerFormat: 'JWT',
        description: 'Enter your JWT access token',
      },
      'JWT-auth',
    )
    .addTag('Auth', 'Authentication & user management')
    .addTag('Families', 'Family management & invitations')
    .addTag('Care Recipients', 'Care recipient profiles')
    .addTag('Medications', 'Medication tracking')
    .addTag('Appointments', 'Appointment scheduling')
    .addTag('Caregiver Shifts', 'Shift management')
    .addTag('Documents', 'Document storage')
    .addTag('Emergency', 'Emergency alerts')
    .addTag('Timeline', 'Health timeline entries')
    .addTag('Notifications', 'Push notifications')
    .addTag('Chat', 'Stream Chat integration')
    .addTag('Health', 'System health checks')
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document, {
    swaggerOptions: {
      persistAuthorization: true,
      docExpansion: 'none',
      tagsSorter: 'alpha',
      operationsSorter: 'alpha',
    },
  });

  await app.listen(4000);
}
```

## Controller Decorators

```typescript
@ApiTags('Medications')
@ApiBearerAuth('JWT-auth')
@Controller('care-recipients/:careRecipientId/medications')
export class MedicationsController {
  
  @Post()
  @ApiOperation({ 
    summary: 'Create a new medication',
    description: 'Creates a medication for the specified care recipient. Requires ADMIN or CAREGIVER role.',
  })
  @ApiParam({ name: 'careRecipientId', type: 'string', format: 'uuid' })
  @ApiResponse({ 
    status: 201, 
    description: 'Medication created successfully',
    type: MedicationResponseDto,
  })
  @ApiResponse({ 
    status: 400, 
    description: 'Validation error',
    type: ErrorResponseDto,
  })
  @ApiResponse({ 
    status: 403, 
    description: 'Insufficient permissions',
    type: ErrorResponseDto,
  })
  @ApiResponse({ 
    status: 404, 
    description: 'Care recipient not found',
    type: ErrorResponseDto,
  })
  create(
    @Param('careRecipientId', ParseUUIDPipe) careRecipientId: string,
    @CurrentUser() user: CurrentUserPayload,
    @Body() dto: CreateMedicationDto,
  ) {
    return this.medicationsService.create(careRecipientId, user.id, dto);
  }
}
```

## DTO Decorators

```typescript
// dto/create-medication.dto.ts
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';

export class CreateMedicationDto {
  @ApiProperty({
    description: 'Name of the medication',
    example: 'Metformin',
    minLength: 1,
    maxLength: 100,
  })
  @IsString()
  name: string;

  @ApiProperty({
    description: 'Dosage amount and unit',
    example: '500mg',
  })
  @IsString()
  dosage: string;

  @ApiProperty({
    enum: MedicationForm,
    description: 'Physical form of the medication',
    example: 'TABLET',
  })
  @IsEnum(MedicationForm)
  form: MedicationForm;

  @ApiProperty({
    type: [String],
    description: 'Scheduled times in HH:mm format',
    example: ['08:00', '20:00'],
  })
  @IsArray()
  @IsString({ each: true })
  scheduledTimes: string[];

  @ApiPropertyOptional({
    description: 'Special instructions for taking this medication',
    example: 'Take with food',
  })
  @IsOptional()
  @IsString()
  instructions?: string;

  @ApiPropertyOptional({
    description: 'Current supply count',
    example: 30,
    minimum: 0,
  })
  @IsOptional()
  @IsNumber()
  @Min(0)
  currentSupply?: number;
}
```

## Response DTOs

```typescript
// dto/responses/medication-response.dto.ts
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';

export class MedicationResponseDto {
  @ApiProperty({ format: 'uuid' })
  id: string;

  @ApiProperty({ example: 'Metformin' })
  name: string;

  @ApiProperty({ example: '500mg' })
  dosage: string;

  @ApiProperty({ enum: MedicationForm })
  form: MedicationForm;

  @ApiProperty({ enum: MedicationFrequency })
  frequency: MedicationFrequency;

  @ApiProperty({ type: [String], example: ['08:00', '20:00'] })
  scheduledTimes: string[];

  @ApiPropertyOptional({ example: 'Take with food' })
  instructions?: string;

  @ApiProperty({ example: true })
  isActive: boolean;

  @ApiProperty({ format: 'date-time' })
  createdAt: Date;

  @ApiProperty({ format: 'date-time' })
  updatedAt: Date;
}

// Generic error response
export class ErrorResponseDto {
  @ApiProperty({ example: false })
  success: boolean;

  @ApiProperty({ example: 400 })
  statusCode: number;

  @ApiProperty({ example: 'Validation failed' })
  message: string;

  @ApiPropertyOptional({
    type: [String],
    example: ['name must be a string'],
  })
  errors?: string[];
}
```

## Enum Documentation

```typescript
// Enums are automatically documented when used in DTOs
export enum MedicationForm {
  TABLET = 'TABLET',
  CAPSULE = 'CAPSULE',
  LIQUID = 'LIQUID',
  INJECTION = 'INJECTION',
  PATCH = 'PATCH',
  CREAM = 'CREAM',
  DROPS = 'DROPS',
  INHALER = 'INHALER',
  OTHER = 'OTHER',
}

// Add descriptions with ApiSchema
@ApiSchema({ description: 'Physical form of medication' })
export enum MedicationForm { ... }
```

## Query Parameters

```typescript
@Get()
@ApiOperation({ summary: 'Get all medications' })
@ApiQuery({ 
  name: 'activeOnly', 
  required: false, 
  type: Boolean,
  description: 'Filter to active medications only',
})
@ApiQuery({
  name: 'search',
  required: false,
  type: String,
  description: 'Search by medication name',
})
findAll(
  @Query('activeOnly', new DefaultValuePipe(true), ParseBoolPipe) activeOnly: boolean,
  @Query('search') search?: string,
) { ... }
```

## File Upload

```typescript
@Post('upload')
@ApiOperation({ summary: 'Upload a document' })
@ApiConsumes('multipart/form-data')
@ApiBody({
  schema: {
    type: 'object',
    properties: {
      file: {
        type: 'string',
        format: 'binary',
      },
      name: {
        type: 'string',
      },
      type: {
        type: 'string',
        enum: ['INSURANCE_CARD', 'MEDICAL_RECORD', 'OTHER'],
      },
    },
    required: ['file'],
  },
})
@UseInterceptors(FileInterceptor('file'))
upload(
  @UploadedFile() file: Express.Multer.File,
  @Body() dto: CreateDocumentDto,
) { ... }
```

## Accessing Documentation

| Environment | URL |
|-------------|-----|
| Development | `http://localhost:4000/api` |
| Production | `https://your-domain.com/api` |

## Features

- **Try It Out**: Execute API calls directly from docs
- **Persist Authorization**: Token survives page refresh
- **Model Schemas**: View request/response structures
- **Enum Values**: See all valid values for enums
- **Code Samples**: Generate cURL commands

## Generate OpenAPI JSON

```typescript
// After SwaggerModule.createDocument
import * as fs from 'fs';

const document = SwaggerModule.createDocument(app, config);
fs.writeFileSync('./openapi.json', JSON.stringify(document, null, 2));
```

## Troubleshooting

### DTOs Not Showing
- Add `@ApiProperty()` to all fields
- Ensure class is exported
- Run with `--watch` for hot reload

### Enums Not Working
- Import from same location consistently
- Use `@ApiProperty({ enum: MyEnum })`

### Auth Not Persisting
- Set `persistAuthorization: true`
- Clear browser storage if issues

---

*See also: [NestJS](nestjs.md), [class-validator](class-validator.md)*


