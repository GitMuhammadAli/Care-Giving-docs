# 🔧 CareCircle API Architecture - Complete Guide

> Deep dive into the NestJS backend architecture, module structure, and design patterns.

---

## Table of Contents

1. [Module Structure](#1-module-structure)
2. [Request Lifecycle](#2-request-lifecycle)
3. [Guards & Decorators](#3-guards--decorators)
4. [DTOs & Validation](#4-dtos--validation)
5. [Database Patterns](#5-database-patterns)
6. [Error Handling](#6-error-handling)
7. [Caching Strategy](#7-caching-strategy)
8. [Testing Patterns](#8-testing-patterns)

---

## 1. Module Structure

### NestJS Module Pattern

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      NESTJS MODULE ANATOMY                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Each feature module follows this structure:                               │
│                                                                             │
│   medications/                                                              │
│   ├── medications.module.ts      # Module definition                        │
│   ├── medications.controller.ts  # HTTP request handlers                    │
│   ├── medications.service.ts     # Business logic                           │
│   ├── dto/                       # Data Transfer Objects                    │
│   │   ├── create-medication.dto.ts                                          │
│   │   ├── update-medication.dto.ts                                          │
│   │   └── log-medication.dto.ts                                             │
│   ├── entities/                  # TypeORM entities                         │
│   │   ├── medication.entity.ts                                              │
│   │   └── medication-log.entity.ts                                          │
│   └── index.ts                   # Public exports                           │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ // medications.module.ts                                            │   │
│   │                                                                     │   │
│   │ @Module({                                                           │   │
│   │   imports: [                                                        │   │
│   │     TypeOrmModule.forFeature([Medication, MedicationLog]),          │   │
│   │     EventsModule,  // For publishing events                         │   │
│   │   ],                                                                │   │
│   │   controllers: [MedicationsController],                             │   │
│   │   providers: [MedicationsService],                                  │   │
│   │   exports: [MedicationsService],  // For use in other modules       │   │
│   │ })                                                                  │   │
│   │ export class MedicationsModule {}                                   │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Module Dependency Graph

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      MODULE DEPENDENCIES                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                          ┌─────────────────┐                                │
│                          │   AppModule     │                                │
│                          └────────┬────────┘                                │
│                                   │                                         │
│         ┌─────────────────────────┼─────────────────────────┐               │
│         │                         │                         │               │
│         ▼                         ▼                         ▼               │
│   ┌───────────┐           ┌───────────────┐           ┌───────────┐        │
│   │   Auth    │           │    System     │           │   Events  │        │
│   │  Module   │           │    Module     │           │   Module  │        │
│   └─────┬─────┘           └───────┬───────┘           └─────┬─────┘        │
│         │                         │                         │               │
│         │ ┌───────────────────────┤                         │               │
│         │ │                       │                         │               │
│         ▼ ▼                       ▼                         ▼               │
│   ┌───────────┐           ┌───────────────┐           ┌───────────┐        │
│   │   User    │           │     Mail      │           │  Gateway  │        │
│   │  Module   │◄──────────│    Module     │           │  Module   │        │
│   └─────┬─────┘           └───────────────┘           └───────────┘        │
│         │                         ▲                                         │
│         │                         │                                         │
│         ▼                         │                                         │
│   ┌───────────┐                   │                                         │
│   │ Families  │                   │                                         │
│   │  Module   │───────────────────┘                                         │
│   └─────┬─────┘                                                             │
│         │                                                                   │
│   ┌─────┴───────────────────────────────────────────────┐                   │
│   │                                                     │                   │
│   ▼                                                     ▼                   │
│ ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐              │
│ │ CareRecipients  │  │   Medications   │  │  Appointments   │              │
│ │     Module      │  │     Module      │  │     Module      │              │
│ └─────────────────┘  └─────────────────┘  └─────────────────┘              │
│                                                                             │
│ ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐              │
│ │    Timeline     │  │    Documents    │  │   Emergency     │              │
│ │     Module      │  │     Module      │  │     Module      │              │
│ └─────────────────┘  └─────────────────┘  └─────────────────┘              │
│                                                                             │
│ ┌─────────────────┐  ┌─────────────────┐                                   │
│ │   Caregivers    │  │  Notifications  │                                   │
│ │     Module      │  │     Module      │                                   │
│ └─────────────────┘  └─────────────────┘                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Request Lifecycle

### Complete Request Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      REQUEST LIFECYCLE                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   HTTP Request: POST /api/v1/medications/med_123/log                        │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ Cookie: accessToken=eyJhbGci...                                     │   │
│   │ Content-Type: application/json                                      │   │
│   │                                                                     │   │
│   │ { "status": "GIVEN", "notes": "Took with breakfast" }               │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                      │                                      │
│                                      ▼                                      │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ 1. GLOBAL MIDDLEWARE                                                │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │ • RequestLoggerMiddleware → Logs request details                    │   │
│   │ • CookieParserMiddleware → Parses cookies                           │   │
│   │ • HelmetMiddleware → Security headers                               │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                      │                                      │
│                                      ▼                                      │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ 2. GLOBAL GUARDS                                                    │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │ • ThrottlerGuard → Rate limiting (100 req/min)                      │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                      │                                      │
│                                      ▼                                      │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ 3. ROUTE GUARDS (Applied via decorators)                            │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │ @UseGuards(JwtAuthGuard)                                            │   │
│   │ • Extract JWT from cookie                                           │   │
│   │ • Verify signature                                                  │   │
│   │ • Attach user to request: req.user = { id, email, familyId, ... }   │   │
│   │                                                                     │   │
│   │ @UseGuards(RolesGuard) + @Roles('ADMIN', 'CAREGIVER')               │   │
│   │ • Check if user.role is in allowed roles                            │   │
│   │ • If not → 403 Forbidden                                            │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                      │                                      │
│                                      ▼                                      │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ 4. GLOBAL INTERCEPTORS                                              │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │ • TransformInterceptor → Wraps response: { data: ..., meta: ... }   │   │
│   │ • TimeoutInterceptor → 30s timeout                                  │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                      │                                      │
│                                      ▼                                      │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ 5. GLOBAL PIPES                                                     │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │ • ValidationPipe → Validates body against DTO                       │   │
│   │   - whitelist: true → Strips unknown properties                     │   │
│   │   - transform: true → Auto-transforms types                         │   │
│   │   - forbidNonWhitelisted: true → Error on unknown props             │   │
│   │                                                                     │   │
│   │ If validation fails → 400 Bad Request with error messages           │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                      │                                      │
│                                      ▼                                      │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ 6. CONTROLLER                                                       │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │ @Controller('medications')                                          │   │
│   │ export class MedicationsController {                                │   │
│   │                                                                     │   │
│   │   @Post(':id/log')                                                  │   │
│   │   @Roles('ADMIN', 'CAREGIVER')                                      │   │
│   │   @UseGuards(RolesGuard)                                            │   │
│   │   async logMedication(                                              │   │
│   │     @Param('id') id: string,                                        │   │
│   │     @Body() dto: LogMedicationDto,                                  │   │
│   │     @CurrentUser() user: User,                                      │   │
│   │   ) {                                                               │   │
│   │     return this.medicationsService.logMedication(id, dto, user);    │   │
│   │   }                                                                 │   │
│   │ }                                                                   │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                      │                                      │
│                                      ▼                                      │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ 7. SERVICE                                                          │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │ async logMedication(id: string, dto: LogMedicationDto, user: User) {│   │
│   │   // 1. Validate medication exists and belongs to user's family     │   │
│   │   const medication = await this.medicationRepo.findOne({            │   │
│   │     where: { id, careRecipient: { familyId: user.familyId } },       │   │
│   │   });                                                               │   │
│   │   if (!medication) throw new NotFoundException();                   │   │
│   │                                                                     │   │
│   │   // 2. Create log entry in transaction                             │   │
│   │   const log = await this.dataSource.transaction(async (em) => {     │   │
│   │     const log = em.create(MedicationLog, {                          │   │
│   │       medicationId: id,                                             │   │
│   │       status: dto.status,                                           │   │
│   │       loggedById: user.id,                                          │   │
│   │       notes: dto.notes,                                             │   │
│   │     });                                                             │   │
│   │     return em.save(log);                                            │   │
│   │   });                                                               │   │
│   │                                                                     │   │
│   │   // 3. Publish event                                               │   │
│   │   await this.eventPublisher.publish('medication.logged', {          │   │
│   │     medicationId: id,                                               │   │
│   │     status: dto.status,                                             │   │
│   │     loggedById: user.id,                                            │   │
│   │     familyId: user.familyId,                                        │   │
│   │   });                                                               │   │
│   │                                                                     │   │
│   │   return log;                                                       │   │
│   │ }                                                                   │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                      │                                      │
│                                      ▼                                      │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ 8. RESPONSE (via TransformInterceptor)                              │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │ {                                                                   │   │
│   │   "data": {                                                         │   │
│   │     "id": "log_abc123",                                             │   │
│   │     "medicationId": "med_123",                                      │   │
│   │     "status": "GIVEN",                                              │   │
│   │     "loggedAt": "2024-01-15T08:15:00Z",                             │   │
│   │     "notes": "Took with breakfast"                                  │   │
│   │   },                                                                │   │
│   │   "meta": {                                                         │   │
│   │     "timestamp": "2024-01-15T08:15:00.123Z",                         │   │
│   │     "path": "/api/v1/medications/med_123/log"                       │   │
│   │   }                                                                 │   │
│   │ }                                                                   │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Guards & Decorators

### Authentication Guard

```typescript
// auth/guards/jwt-auth.guard.ts

@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(
    private jwtService: JwtService,
    private reflector: Reflector,
    private configService: ConfigService
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    // Check if route is public
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (isPublic) return true;

    const request = context.switchToHttp().getRequest();

    // Extract token from cookie
    const token = request.cookies?.accessToken;

    if (!token) {
      throw new UnauthorizedException("No access token provided");
    }

    try {
      // Verify token
      const payload = await this.jwtService.verifyAsync(token, {
        secret: this.configService.get("jwt.secret"),
      });

      // Attach user to request
      request.user = payload;

      return true;
    } catch (error) {
      throw new UnauthorizedException("Invalid or expired token");
    }
  }
}
```

### Role-Based Access Guard

```typescript
// auth/guards/roles.guard.ts

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(
      ROLES_KEY,
      [context.getHandler(), context.getClass()]
    );

    if (!requiredRoles || requiredRoles.length === 0) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();

    if (!user || !user.role) {
      throw new ForbiddenException("No role assigned");
    }

    const hasRole = requiredRoles.includes(user.role);

    if (!hasRole) {
      throw new ForbiddenException(
        `Requires one of: ${requiredRoles.join(", ")}`
      );
    }

    return true;
  }
}
```

### Custom Decorators

```typescript
// auth/decorators/current-user.decorator.ts

/**
 * Extract current user from request
 * @example
 * @Get('profile')
 * getProfile(@CurrentUser() user: User) {
 *   return user;
 * }
 */
export const CurrentUser = createParamDecorator(
  (data: keyof User | undefined, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    return data ? user?.[data] : user;
  }
);

// auth/decorators/roles.decorator.ts

/**
 * Restrict access to specific roles
 * @example
 * @Post('invite')
 * @Roles('ADMIN')
 * inviteMember() { }
 */
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);

// auth/decorators/public.decorator.ts

/**
 * Mark endpoint as public (no auth required)
 * @example
 * @Get('health')
 * @Public()
 * healthCheck() { return 'OK'; }
 */
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

---

## 4. DTOs & Validation

### DTO Pattern

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      DTO VALIDATION PATTERN                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   // medications/dto/create-medication.dto.ts                               │
│                                                                             │
│   export class CreateMedicationDto {                                        │
│     @IsString()                                                             │
│     @IsNotEmpty({ message: 'Medication name is required' })                 │
│     @MinLength(2, { message: 'Name must be at least 2 characters' })        │
│     @MaxLength(100)                                                         │
│     name: string;                                                           │
│                                                                             │
│     @IsString()                                                             │
│     @IsNotEmpty({ message: 'Dosage is required' })                          │
│     @Matches(/^\d+(\.\d+)?\s?(mg|g|ml|mcg|iu|units?)$/i, {                   │
│       message: 'Invalid dosage format (e.g., "500mg", "10ml")'              │
│     })                                                                      │
│     dosage: string;                                                         │
│                                                                             │
│     @IsEnum(MedicationFrequency, {                                          │
│       message: 'Invalid frequency'                                          │
│     })                                                                      │
│     frequency: MedicationFrequency;                                         │
│                                                                             │
│     @IsArray()                                                              │
│     @ArrayMinSize(1, { message: 'At least one time required' })             │
│     @IsString({ each: true })                                               │
│     @Matches(/^([01]?[0-9]|2[0-3]):[0-5][0-9]$/, {                          │
│       each: true,                                                           │
│       message: 'Invalid time format (HH:mm)'                                │
│     })                                                                      │
│     times: string[];                                                        │
│                                                                             │
│     @IsString()                                                             │
│     @IsOptional()                                                           │
│     @MaxLength(500)                                                         │
│     instructions?: string;                                                  │
│                                                                             │
│     @IsInt()                                                                │
│     @IsOptional()                                                           │
│     @Min(0)                                                                 │
│     @Max(10000)                                                             │
│     currentSupply?: number;                                                 │
│                                                                             │
│     @IsInt()                                                                │
│     @IsOptional()                                                           │
│     @Min(1)                                                                 │
│     @Max(100)                                                               │
│     refillThreshold?: number;                                               │
│   }                                                                         │
│                                                                             │
│   // For updates, use PartialType to make all fields optional               │
│   export class UpdateMedicationDto extends PartialType(CreateMedicationDto) {}│
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Custom Validators

```typescript
// common/validators/is-family-member.validator.ts

@ValidatorConstraint({ async: true })
@Injectable()
export class IsFamilyMemberConstraint implements ValidatorConstraintInterface {
  constructor(private familyService: FamilyService) {}

  async validate(userId: string, args: ValidationArguments) {
    const object = args.object as any;
    const familyId = object.familyId;

    if (!familyId) return false;

    return this.familyService.isMember(userId, familyId);
  }

  defaultMessage() {
    return "User is not a member of this family";
  }
}

export function IsFamilyMember(validationOptions?: ValidationOptions) {
  return function (object: Object, propertyName: string) {
    registerDecorator({
      target: object.constructor,
      propertyName: propertyName,
      options: validationOptions,
      constraints: [],
      validator: IsFamilyMemberConstraint,
    });
  };
}

// Usage:
export class AssignCaregiverDto {
  @IsUUID()
  @IsFamilyMember({ message: "Caregiver must be a family member" })
  caregiverId: string;
}
```

---

## 5. Database Patterns

### Entity Design

```typescript
// common/entities/base.entity.ts

export abstract class BaseEntity {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @CreateDateColumn({ type: "timestamp with time zone" })
  createdAt: Date;

  @UpdateDateColumn({ type: "timestamp with time zone" })
  updatedAt: Date;

  @DeleteDateColumn({ type: "timestamp with time zone", nullable: true })
  deletedAt?: Date;
}

// medications/entities/medication.entity.ts

@Entity("medications")
export class Medication extends BaseEntity {
  @Column({ length: 100 })
  name: string;

  @Column({ length: 50 })
  dosage: string;

  @Column({ type: "enum", enum: MedicationFrequency })
  frequency: MedicationFrequency;

  @Column("simple-array")
  times: string[];

  @Column({ type: "text", nullable: true })
  instructions: string;

  @Column({ default: 0 })
  currentSupply: number;

  @Column({ default: 15 })
  refillThreshold: number;

  @Column({ default: true })
  isActive: boolean;

  // Relations
  @ManyToOne(() => CareRecipient, (cr) => cr.medications, {
    onDelete: "CASCADE",
  })
  @JoinColumn({ name: "care_recipient_id" })
  careRecipient: CareRecipient;

  @Column({ name: "care_recipient_id" })
  careRecipientId: string;

  @OneToMany(() => MedicationLog, (log) => log.medication)
  logs: MedicationLog[];

  // Computed properties
  @AfterLoad()
  computeNeedsRefill() {
    this.needsRefill = this.currentSupply <= this.refillThreshold;
  }

  needsRefill: boolean;
}
```

### Repository Pattern

```typescript
// medications/medications.service.ts

@Injectable()
export class MedicationsService {
  constructor(
    @InjectRepository(Medication)
    private medicationRepo: Repository<Medication>,
    @InjectRepository(MedicationLog)
    private logRepo: Repository<MedicationLog>,
    private dataSource: DataSource,
    private eventPublisher: EventPublisherService
  ) {}

  /**
   * Find medications for a care recipient
   * Always scoped to the user's family for security
   */
  async findAllForRecipient(
    recipientId: string,
    familyId: string,
    options?: { activeOnly?: boolean }
  ): Promise<Medication[]> {
    const query = this.medicationRepo
      .createQueryBuilder("medication")
      .leftJoinAndSelect("medication.careRecipient", "recipient")
      .where("medication.careRecipientId = :recipientId", { recipientId })
      .andWhere("recipient.familyId = :familyId", { familyId });

    if (options?.activeOnly) {
      query.andWhere("medication.isActive = :isActive", { isActive: true });
    }

    return query.orderBy("medication.name", "ASC").getMany();
  }

  /**
   * Get today's medication schedule
   */
  async getSchedule(
    recipientId: string,
    familyId: string,
    date: Date
  ): Promise<MedicationScheduleItem[]> {
    const medications = await this.findAllForRecipient(recipientId, familyId, {
      activeOnly: true,
    });

    const startOfDay = new Date(date.setHours(0, 0, 0, 0));
    const endOfDay = new Date(date.setHours(23, 59, 59, 999));

    // Get logs for the day
    const logs = await this.logRepo.find({
      where: {
        medication: { careRecipientId: recipientId },
        scheduledTime: Between(startOfDay, endOfDay),
      },
      relations: ["medication", "loggedBy"],
    });

    // Build schedule
    return medications.flatMap((med) =>
      med.times.map((time) => {
        const scheduledTime = this.parseTime(time, date);
        const log = logs.find(
          (l) =>
            l.medicationId === med.id &&
            l.scheduledTime.getTime() === scheduledTime.getTime()
        );

        return {
          medication: med,
          scheduledTime,
          status: log?.status || "PENDING",
          log: log || null,
        };
      })
    );
  }

  /**
   * Log a medication as given or skipped
   * Uses transaction for data integrity
   */
  async logMedication(
    medicationId: string,
    dto: LogMedicationDto,
    user: User
  ): Promise<MedicationLog> {
    // Verify medication exists and belongs to user's family
    const medication = await this.medicationRepo.findOne({
      where: { id: medicationId },
      relations: ["careRecipient"],
    });

    if (!medication) {
      throw new NotFoundException("Medication not found");
    }

    if (medication.careRecipient.familyId !== user.familyId) {
      throw new ForbiddenException("Not authorized");
    }

    // Create log in transaction
    return this.dataSource.transaction(async (em) => {
      const log = em.create(MedicationLog, {
        medicationId,
        status: dto.status,
        scheduledTime: dto.scheduledTime,
        loggedById: user.id,
        loggedAt: new Date(),
        notes: dto.notes,
        skipReason: dto.skipReason,
      });

      const savedLog = await em.save(log);

      // Update supply if given
      if (dto.status === "GIVEN") {
        await em.decrement(
          Medication,
          { id: medicationId },
          "currentSupply",
          1
        );
      }

      // Publish event (outside transaction for performance)
      setImmediate(async () => {
        await this.eventPublisher.publish(
          "medication.logged",
          {
            medicationId,
            careRecipientId: medication.careRecipientId,
            familyId: medication.careRecipient.familyId,
            status: dto.status,
            loggedById: user.id,
            loggedByName: user.fullName,
            medicationName: medication.name,
            scheduledTime: dto.scheduledTime.toISOString(),
          },
          `medication.${dto.status.toLowerCase()}.${
            medication.careRecipientId
          }`,
          `/api/v1/medications/${medicationId}/log`
        );
      });

      return savedLog;
    });
  }
}
```

---

## 6. Error Handling

### Global Exception Filter

```typescript
// common/filters/all-exceptions.filter.ts

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  private readonly logger = new Logger(AllExceptionsFilter.name);

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = "Internal server error";
    let errors: string[] | undefined;

    if (exception instanceof HttpException) {
      status = exception.getStatus();
      const exceptionResponse = exception.getResponse();

      if (typeof exceptionResponse === "object") {
        message = (exceptionResponse as any).message || message;
        errors = (exceptionResponse as any).errors;
      } else {
        message = exceptionResponse as string;
      }
    } else if (exception instanceof QueryFailedError) {
      // Database errors
      status = HttpStatus.BAD_REQUEST;
      message = this.handleDatabaseError(exception);
    } else if (exception instanceof Error) {
      message = exception.message;
    }

    // Log error
    this.logger.error({
      message,
      status,
      path: request.url,
      method: request.method,
      stack: exception instanceof Error ? exception.stack : undefined,
      userId: (request as any).user?.id,
    });

    // Send error response
    response.status(status).json({
      statusCode: status,
      message,
      errors,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }

  private handleDatabaseError(error: QueryFailedError): string {
    // PostgreSQL error codes
    const code = (error as any).code;

    switch (code) {
      case "23505": // unique_violation
        return "A record with this value already exists";
      case "23503": // foreign_key_violation
        return "Referenced record does not exist";
      case "23502": // not_null_violation
        return "Required field is missing";
      default:
        return "Database error occurred";
    }
  }
}
```

### Custom Exceptions

```typescript
// common/exceptions/business.exceptions.ts

export class MedicationAlreadyLoggedException extends HttpException {
  constructor(medicationId: string, scheduledTime: Date) {
    super(
      {
        message: "Medication already logged for this time",
        code: "MEDICATION_ALREADY_LOGGED",
        medicationId,
        scheduledTime: scheduledTime.toISOString(),
      },
      HttpStatus.CONFLICT
    );
  }
}

export class FamilyMemberLimitExceededException extends HttpException {
  constructor(limit: number) {
    super(
      {
        message: `Family member limit (${limit}) exceeded`,
        code: "FAMILY_MEMBER_LIMIT_EXCEEDED",
        limit,
      },
      HttpStatus.PAYMENT_REQUIRED
    );
  }
}

export class InviteExpiredException extends HttpException {
  constructor() {
    super(
      {
        message: "This invitation has expired",
        code: "INVITE_EXPIRED",
      },
      HttpStatus.GONE
    );
  }
}
```

---

## 7. Caching Strategy

### Redis Cache Implementation

```typescript
// common/cache/cache.service.ts

@Injectable()
export class CacheService {
  private readonly ttl = {
    short: 60, // 1 minute
    medium: 300, // 5 minutes
    long: 3600, // 1 hour
    veryLong: 86400, // 1 day
  };

  constructor(
    @Inject(CACHE_MANAGER) private cache: Cache,
    private readonly logger: Logger
  ) {}

  /**
   * Generate cache key with namespace
   */
  private key(namespace: string, ...parts: string[]): string {
    return `carecircle:${namespace}:${parts.join(":")}`;
  }

  /**
   * Get from cache or fetch from source
   */
  async getOrSet<T>(
    namespace: string,
    id: string,
    fetcher: () => Promise<T>,
    ttl: keyof typeof this.ttl = "medium"
  ): Promise<T> {
    const cacheKey = this.key(namespace, id);

    // Try cache first
    const cached = await this.cache.get<T>(cacheKey);
    if (cached) {
      this.logger.debug(`Cache HIT: ${cacheKey}`);
      return cached;
    }

    // Fetch from source
    this.logger.debug(`Cache MISS: ${cacheKey}`);
    const data = await fetcher();

    // Store in cache
    await this.cache.set(cacheKey, data, this.ttl[ttl]);

    return data;
  }

  /**
   * Invalidate cache entries
   */
  async invalidate(namespace: string, ...ids: string[]): Promise<void> {
    for (const id of ids) {
      const key = this.key(namespace, id);
      await this.cache.del(key);
      this.logger.debug(`Cache INVALIDATED: ${key}`);
    }
  }

  /**
   * Invalidate all entries matching pattern
   */
  async invalidatePattern(namespace: string, pattern: string): Promise<void> {
    // Note: This requires scanning, use sparingly
    const keys = await this.scanKeys(`carecircle:${namespace}:${pattern}`);
    for (const key of keys) {
      await this.cache.del(key);
    }
    this.logger.debug(`Cache INVALIDATED pattern: ${namespace}:${pattern}`);
  }
}
```

### Cache Usage Example

```typescript
// medications/medications.service.ts

@Injectable()
export class MedicationsService {
  constructor(private cacheService: CacheService) // ... other dependencies
  {}

  async findAllForRecipient(
    recipientId: string,
    familyId: string
  ): Promise<Medication[]> {
    return this.cacheService.getOrSet(
      "medications",
      `recipient:${recipientId}`,
      async () => {
        return this.medicationRepo.find({
          where: {
            careRecipientId: recipientId,
            careRecipient: { familyId },
          },
        });
      },
      "medium" // 5 minutes
    );
  }

  async logMedication(
    medicationId: string,
    dto: LogMedicationDto,
    user: User
  ): Promise<MedicationLog> {
    const log = await this.createLog(medicationId, dto, user);

    // Invalidate related caches
    const medication = await this.medicationRepo.findOne({
      where: { id: medicationId },
    });

    await this.cacheService.invalidate(
      "medications",
      `recipient:${medication.careRecipientId}`,
      `schedule:${medication.careRecipientId}:${formatDate(dto.scheduledTime)}`
    );

    return log;
  }
}
```

---

## 8. Testing Patterns

### Unit Test Example

```typescript
// medications/medications.service.spec.ts

describe("MedicationsService", () => {
  let service: MedicationsService;
  let medicationRepo: MockRepository<Medication>;
  let eventPublisher: MockEventPublisher;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        MedicationsService,
        {
          provide: getRepositoryToken(Medication),
          useValue: createMockRepository(),
        },
        {
          provide: EventPublisherService,
          useValue: createMockEventPublisher(),
        },
      ],
    }).compile();

    service = module.get<MedicationsService>(MedicationsService);
    medicationRepo = module.get(getRepositoryToken(Medication));
    eventPublisher = module.get(EventPublisherService);
  });

  describe("logMedication", () => {
    it("should log medication as GIVEN", async () => {
      // Arrange
      const medication = createMockMedication();
      const user = createMockUser();
      const dto: LogMedicationDto = {
        status: "GIVEN",
        scheduledTime: new Date(),
        notes: "Took with breakfast",
      };

      medicationRepo.findOne.mockResolvedValue(medication);

      // Act
      const result = await service.logMedication(medication.id, dto, user);

      // Assert
      expect(result.status).toBe("GIVEN");
      expect(result.loggedById).toBe(user.id);
      expect(eventPublisher.publish).toHaveBeenCalledWith(
        "medication.logged",
        expect.objectContaining({
          medicationId: medication.id,
          status: "GIVEN",
        })
      );
    });

    it("should throw NotFoundException if medication not found", async () => {
      medicationRepo.findOne.mockResolvedValue(null);

      await expect(
        service.logMedication("invalid-id", {} as any, {} as any)
      ).rejects.toThrow(NotFoundException);
    });

    it("should throw ForbiddenException if not in same family", async () => {
      const medication = createMockMedication({ familyId: "family-1" });
      const user = createMockUser({ familyId: "family-2" });

      medicationRepo.findOne.mockResolvedValue(medication);

      await expect(
        service.logMedication(medication.id, {} as any, user)
      ).rejects.toThrow(ForbiddenException);
    });
  });
});
```

### E2E Test Example

```typescript
// test/medications.e2e-spec.ts

describe("MedicationsController (e2e)", () => {
  let app: INestApplication;
  let authCookies: string[];

  beforeAll(async () => {
    app = await createTestApp();

    // Login to get auth cookies
    const loginResponse = await request(app.getHttpServer())
      .post("/api/v1/auth/login")
      .send({ email: "test@example.com", password: "TestPassword123" });

    authCookies = loginResponse.headers["set-cookie"];
  });

  afterAll(async () => {
    await app.close();
  });

  describe("POST /api/v1/medications/:id/log", () => {
    it("should log medication successfully", async () => {
      const response = await request(app.getHttpServer())
        .post("/api/v1/medications/med_test123/log")
        .set("Cookie", authCookies)
        .send({
          status: "GIVEN",
          scheduledTime: new Date().toISOString(),
          notes: "Test log",
        });

      expect(response.status).toBe(201);
      expect(response.body.data).toMatchObject({
        status: "GIVEN",
        notes: "Test log",
      });
    });

    it("should return 401 without auth", async () => {
      const response = await request(app.getHttpServer())
        .post("/api/v1/medications/med_test123/log")
        .send({ status: "GIVEN" });

      expect(response.status).toBe(401);
    });

    it("should return 400 with invalid status", async () => {
      const response = await request(app.getHttpServer())
        .post("/api/v1/medications/med_test123/log")
        .set("Cookie", authCookies)
        .send({ status: "INVALID" });

      expect(response.status).toBe(400);
      expect(response.body.message).toContain("Invalid status");
    });
  });
});
```

---

## Summary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          KEY TAKEAWAYS                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   1. MODULE STRUCTURE                                                       │
│      • Each feature is a self-contained module                              │
│      • Controllers handle HTTP, Services handle business logic              │
│      • DTOs validate input, Entities model data                             │
│                                                                             │
│   2. REQUEST LIFECYCLE                                                      │
│      Middleware → Guards → Interceptors → Pipes → Controller → Service      │
│                                                                             │
│   3. SECURITY                                                               │
│      • JwtAuthGuard validates tokens                                        │
│      • RolesGuard enforces RBAC                                             │
│      • All data scoped to user's family                                     │
│                                                                             │
│   4. DATABASE                                                               │
│      • TypeORM with PostgreSQL                                              │
│      • Transactions for data integrity                                      │
│      • Soft deletes for audit trail                                         │
│                                                                             │
│   5. CACHING                                                                │
│      • Redis for frequently accessed data                                   │
│      • Invalidation on writes                                               │
│      • Namespace-based key structure                                        │
│                                                                             │
│   6. TESTING                                                                │
│      • Unit tests for services                                              │
│      • E2E tests for API endpoints                                          │
│      • Mock repositories and dependencies                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

**Related Guides:**

- [COMPLETE_LEARNING_GUIDE.md](./COMPLETE_LEARNING_GUIDE.md) - Project overview
- [AUTH_COMPLETE_GUIDE.md](./AUTH_COMPLETE_GUIDE.md) - Authentication details
- [EVENT_DRIVEN_ARCHITECTURE.md](../EVENT_DRIVEN_ARCHITECTURE.md) - Event system

---

_CareCircle API: Enterprise-grade backend architecture. 🔧_
