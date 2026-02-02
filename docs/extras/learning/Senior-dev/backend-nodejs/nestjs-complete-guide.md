# NestJS Framework - Complete Guide

> **MUST REMEMBER**: NestJS is an opinionated Node.js framework using TypeScript, decorators, and dependency injection. It's inspired by Angular and provides structure for enterprise applications. Key concepts: Modules (organization), Controllers (HTTP), Providers (services/DI), Guards (auth), Interceptors (transform), Pipes (validation), Filters (exceptions).

---

## How to Explain Like a Senior Developer

"NestJS brings structure and opinions to Node.js backend development. While Express gives you freedom, NestJS gives you architecture - modules, dependency injection, decorators, the whole nine yards. It's great for large teams because everyone writes code the same way. The DI container handles wiring dependencies, decorators define routes and validation, and the modular structure keeps things organized. Think of it as 'Angular for the backend' - same mental model, similar patterns. The learning curve is steeper than Express, but the payoff is maintainable, testable code at scale."

---

## Core Implementation

### Project Structure

```
src/
├── app.module.ts           # Root module
├── main.ts                 # Entry point
├── common/                 # Shared utilities
│   ├── decorators/
│   ├── filters/
│   ├── guards/
│   ├── interceptors/
│   └── pipes/
├── config/                 # Configuration
│   └── config.module.ts
├── users/                  # Feature module
│   ├── users.module.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   ├── dto/
│   │   ├── create-user.dto.ts
│   │   └── update-user.dto.ts
│   ├── entities/
│   │   └── user.entity.ts
│   └── users.repository.ts
├── auth/                   # Auth module
│   ├── auth.module.ts
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   ├── strategies/
│   │   ├── jwt.strategy.ts
│   │   └── local.strategy.ts
│   └── guards/
│       └── jwt-auth.guard.ts
└── prisma/                 # Database
    └── prisma.service.ts
```

### Basic Module, Controller, Service

```typescript
// users/users.module.ts
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { PrismaModule } from '../prisma/prisma.module';

@Module({
  imports: [PrismaModule],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService], // Available to other modules
})
export class UsersModule {}
```

```typescript
// users/users.controller.ts
import {
  Controller,
  Get,
  Post,
  Put,
  Delete,
  Body,
  Param,
  Query,
  HttpCode,
  HttpStatus,
  ParseUUIDPipe,
  UseGuards,
  UseInterceptors,
} from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { ListUsersDto } from './dto/list-users.dto';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { LoggingInterceptor } from '../common/interceptors/logging.interceptor';
import { ApiTags, ApiOperation, ApiResponse } from '@nestjs/swagger';

@ApiTags('users')
@Controller('users')
@UseInterceptors(LoggingInterceptor)
export class UsersController {
  constructor(private readonly usersService: UsersService) {}
  
  @Post()
  @HttpCode(HttpStatus.CREATED)
  @ApiOperation({ summary: 'Create a new user' })
  @ApiResponse({ status: 201, description: 'User created successfully' })
  @ApiResponse({ status: 400, description: 'Validation error' })
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
  
  @Get()
  @UseGuards(JwtAuthGuard)
  @ApiOperation({ summary: 'List all users' })
  findAll(@Query() query: ListUsersDto) {
    return this.usersService.findAll(query);
  }
  
  @Get(':id')
  @UseGuards(JwtAuthGuard)
  @ApiOperation({ summary: 'Get user by ID' })
  @ApiResponse({ status: 404, description: 'User not found' })
  findOne(@Param('id', ParseUUIDPipe) id: string) {
    return this.usersService.findOne(id);
  }
  
  @Put(':id')
  @UseGuards(JwtAuthGuard)
  update(
    @Param('id', ParseUUIDPipe) id: string,
    @Body() updateUserDto: UpdateUserDto,
  ) {
    return this.usersService.update(id, updateUserDto);
  }
  
  @Delete(':id')
  @UseGuards(JwtAuthGuard)
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id', ParseUUIDPipe) id: string) {
    return this.usersService.remove(id);
  }
}
```

```typescript
// users/users.service.ts
import { Injectable, NotFoundException, ConflictException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { ListUsersDto } from './dto/list-users.dto';
import * as bcrypt from 'bcrypt';

@Injectable()
export class UsersService {
  constructor(private readonly prisma: PrismaService) {}
  
  async create(createUserDto: CreateUserDto) {
    // Check if email exists
    const existing = await this.prisma.user.findUnique({
      where: { email: createUserDto.email },
    });
    
    if (existing) {
      throw new ConflictException('Email already exists');
    }
    
    // Hash password
    const hashedPassword = await bcrypt.hash(createUserDto.password, 10);
    
    return this.prisma.user.create({
      data: {
        ...createUserDto,
        password: hashedPassword,
      },
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        createdAt: true,
      },
    });
  }
  
  async findAll(query: ListUsersDto) {
    const { page = 1, limit = 20, search, role, sortBy, sortOrder } = query;
    
    const where = {
      ...(search && {
        OR: [
          { name: { contains: search, mode: 'insensitive' as const } },
          { email: { contains: search, mode: 'insensitive' as const } },
        ],
      }),
      ...(role && { role }),
    };
    
    const [users, total] = await Promise.all([
      this.prisma.user.findMany({
        where,
        select: {
          id: true,
          email: true,
          name: true,
          role: true,
          createdAt: true,
        },
        skip: (page - 1) * limit,
        take: limit,
        orderBy: sortBy ? { [sortBy]: sortOrder || 'asc' } : undefined,
      }),
      this.prisma.user.count({ where }),
    ]);
    
    return {
      data: users,
      meta: {
        total,
        page,
        limit,
        totalPages: Math.ceil(total / limit),
      },
    };
  }
  
  async findOne(id: string) {
    const user = await this.prisma.user.findUnique({
      where: { id },
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        createdAt: true,
        updatedAt: true,
      },
    });
    
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    
    return user;
  }
  
  async findByEmail(email: string) {
    return this.prisma.user.findUnique({ where: { email } });
  }
  
  async update(id: string, updateUserDto: UpdateUserDto) {
    await this.findOne(id); // Throws if not found
    
    return this.prisma.user.update({
      where: { id },
      data: updateUserDto,
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        updatedAt: true,
      },
    });
  }
  
  async remove(id: string) {
    await this.findOne(id);
    await this.prisma.user.delete({ where: { id } });
  }
}
```

### DTOs with Validation

```typescript
// users/dto/create-user.dto.ts
import { 
  IsEmail, 
  IsString, 
  MinLength, 
  MaxLength, 
  IsEnum, 
  IsOptional,
  Matches,
} from 'class-validator';
import { Transform } from 'class-transformer';
import { ApiProperty } from '@nestjs/swagger';

export enum UserRole {
  USER = 'user',
  ADMIN = 'admin',
}

export class CreateUserDto {
  @ApiProperty({ example: 'john@example.com' })
  @IsEmail()
  @Transform(({ value }) => value.toLowerCase().trim())
  email: string;
  
  @ApiProperty({ example: 'John Doe' })
  @IsString()
  @MinLength(1)
  @MaxLength(100)
  @Transform(({ value }) => value.trim())
  name: string;
  
  @ApiProperty({ example: 'Password123!' })
  @IsString()
  @MinLength(8)
  @MaxLength(100)
  @Matches(
    /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
    { message: 'Password must contain uppercase, lowercase, and number' }
  )
  password: string;
  
  @ApiProperty({ enum: UserRole, default: UserRole.USER })
  @IsEnum(UserRole)
  @IsOptional()
  role?: UserRole = UserRole.USER;
}

// users/dto/update-user.dto.ts
import { PartialType, OmitType } from '@nestjs/mapped-types';
import { CreateUserDto } from './create-user.dto';

export class UpdateUserDto extends PartialType(
  OmitType(CreateUserDto, ['password', 'email'] as const)
) {}

// users/dto/list-users.dto.ts
import { IsOptional, IsInt, Min, Max, IsEnum, IsString } from 'class-validator';
import { Type } from 'class-transformer';
import { ApiPropertyOptional } from '@nestjs/swagger';
import { UserRole } from './create-user.dto';

export class ListUsersDto {
  @ApiPropertyOptional({ default: 1 })
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;
  
  @ApiPropertyOptional({ default: 20 })
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 20;
  
  @ApiPropertyOptional()
  @IsOptional()
  @IsString()
  search?: string;
  
  @ApiPropertyOptional({ enum: UserRole })
  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole;
  
  @ApiPropertyOptional({ enum: ['name', 'email', 'createdAt'] })
  @IsOptional()
  @IsString()
  sortBy?: string;
  
  @ApiPropertyOptional({ enum: ['asc', 'desc'] })
  @IsOptional()
  @IsEnum(['asc', 'desc'])
  sortOrder?: 'asc' | 'desc';
}
```

### Authentication with JWT

```typescript
// auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { ConfigService } from '@nestjs/config';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { UsersModule } from '../users/users.module';
import { JwtStrategy } from './strategies/jwt.strategy';
import { LocalStrategy } from './strategies/local.strategy';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.registerAsync({
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        secret: config.get('JWT_SECRET'),
        signOptions: { expiresIn: '1h' },
      }),
    }),
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy, LocalStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

```typescript
// auth/auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';
import * as bcrypt from 'bcrypt';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}
  
  async validateUser(email: string, password: string) {
    const user = await this.usersService.findByEmail(email);
    
    if (user && await bcrypt.compare(password, user.password)) {
      const { password: _, ...result } = user;
      return result;
    }
    
    return null;
  }
  
  async login(user: any) {
    const payload = { email: user.email, sub: user.id, role: user.role };
    
    return {
      accessToken: this.jwtService.sign(payload),
      user: {
        id: user.id,
        email: user.email,
        name: user.name,
        role: user.role,
      },
    };
  }
  
  async register(createUserDto: any) {
    const user = await this.usersService.create(createUserDto);
    return this.login(user);
  }
}

// auth/strategies/jwt.strategy.ts
import { Injectable } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private configService: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get('JWT_SECRET'),
    });
  }
  
  async validate(payload: any) {
    return { 
      id: payload.sub, 
      email: payload.email, 
      role: payload.role 
    };
  }
}

// auth/strategies/local.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { Strategy } from 'passport-local';
import { AuthService } from '../auth.service';

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super({ usernameField: 'email' });
  }
  
  async validate(email: string, password: string) {
    const user = await this.authService.validateUser(email, password);
    
    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }
    
    return user;
  }
}

// auth/guards/jwt-auth.guard.ts
import { Injectable, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';
import { Reflector } from '@nestjs/core';
import { IS_PUBLIC_KEY } from '../../common/decorators/public.decorator';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }
  
  canActivate(context: ExecutionContext) {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (isPublic) {
      return true;
    }
    
    return super.canActivate(context);
  }
}
```

### Custom Decorators

```typescript
// common/decorators/public.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

// common/decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);

// common/decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentUser = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;
    
    return data ? user?.[data] : user;
  },
);

// Usage
@Get('profile')
getProfile(@CurrentUser() user: any) {
  return user;
}

@Get('profile/id')
getProfileId(@CurrentUser('id') userId: string) {
  return userId;
}
```

### Guards and Interceptors

```typescript
// common/guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { ROLES_KEY } from '../decorators/roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  
  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (!requiredRoles) {
      return true;
    }
    
    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.includes(user.role);
  }
}

// common/interceptors/transform.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface Response<T> {
  data: T;
  statusCode: number;
  timestamp: string;
}

@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(
    context: ExecutionContext,
    next: CallHandler,
  ): Observable<Response<T>> {
    const ctx = context.switchToHttp();
    const response = ctx.getResponse();
    
    return next.handle().pipe(
      map((data) => ({
        data,
        statusCode: response.statusCode,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}

// common/interceptors/logging.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  Logger,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(LoggingInterceptor.name);
  
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const now = Date.now();
    
    return next.handle().pipe(
      tap(() => {
        const response = context.switchToHttp().getResponse();
        this.logger.log(
          `${method} ${url} ${response.statusCode} - ${Date.now() - now}ms`,
        );
      }),
    );
  }
}
```

### Exception Filters

```typescript
// common/filters/http-exception.filter.ts
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
  Logger,
} from '@nestjs/common';
import { Request, Response } from 'express';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  private readonly logger = new Logger(AllExceptionsFilter.name);
  
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    
    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Internal server error';
    let errors: any = undefined;
    
    if (exception instanceof HttpException) {
      status = exception.getStatus();
      const exceptionResponse = exception.getResponse();
      
      if (typeof exceptionResponse === 'object') {
        const resp = exceptionResponse as any;
        message = resp.message || exception.message;
        errors = resp.errors;
      } else {
        message = exceptionResponse;
      }
    } else if (exception instanceof Error) {
      message = exception.message;
      
      // Log unexpected errors
      this.logger.error(
        `Unexpected error: ${exception.message}`,
        exception.stack,
      );
    }
    
    response.status(status).json({
      statusCode: status,
      message,
      errors,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}

// common/filters/validation-exception.filter.ts
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  BadRequestException,
} from '@nestjs/common';
import { Response } from 'express';

@Catch(BadRequestException)
export class ValidationExceptionFilter implements ExceptionFilter {
  catch(exception: BadRequestException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const status = exception.getStatus();
    const exceptionResponse = exception.getResponse() as any;
    
    response.status(status).json({
      statusCode: status,
      error: 'Validation Error',
      message: 'Request validation failed',
      details: Array.isArray(exceptionResponse.message)
        ? exceptionResponse.message.map((msg: string) => ({
            message: msg,
          }))
        : [{ message: exceptionResponse.message }],
      timestamp: new Date().toISOString(),
    });
  }
}
```

### Testing

```typescript
// users/users.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UsersService } from './users.service';
import { PrismaService } from '../prisma/prisma.service';
import { ConflictException, NotFoundException } from '@nestjs/common';

describe('UsersService', () => {
  let service: UsersService;
  let prisma: PrismaService;
  
  const mockPrismaService = {
    user: {
      findUnique: jest.fn(),
      findMany: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
      count: jest.fn(),
    },
  };
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: PrismaService,
          useValue: mockPrismaService,
        },
      ],
    }).compile();
    
    service = module.get<UsersService>(UsersService);
    prisma = module.get<PrismaService>(PrismaService);
    
    jest.clearAllMocks();
  });
  
  describe('create', () => {
    const createUserDto = {
      email: 'test@example.com',
      name: 'Test User',
      password: 'Password123!',
    };
    
    it('should create a user', async () => {
      mockPrismaService.user.findUnique.mockResolvedValue(null);
      mockPrismaService.user.create.mockResolvedValue({
        id: '1',
        email: createUserDto.email,
        name: createUserDto.name,
        role: 'user',
        createdAt: new Date(),
      });
      
      const result = await service.create(createUserDto);
      
      expect(result.email).toBe(createUserDto.email);
      expect(mockPrismaService.user.create).toHaveBeenCalled();
    });
    
    it('should throw ConflictException if email exists', async () => {
      mockPrismaService.user.findUnique.mockResolvedValue({ id: '1' });
      
      await expect(service.create(createUserDto)).rejects.toThrow(
        ConflictException,
      );
    });
  });
  
  describe('findOne', () => {
    it('should return a user', async () => {
      const user = { id: '1', email: 'test@example.com', name: 'Test' };
      mockPrismaService.user.findUnique.mockResolvedValue(user);
      
      const result = await service.findOne('1');
      
      expect(result).toEqual(user);
    });
    
    it('should throw NotFoundException if user not found', async () => {
      mockPrismaService.user.findUnique.mockResolvedValue(null);
      
      await expect(service.findOne('1')).rejects.toThrow(NotFoundException);
    });
  });
});

// users/users.controller.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

describe('UsersController', () => {
  let controller: UsersController;
  let service: UsersService;
  
  const mockUsersService = {
    create: jest.fn(),
    findAll: jest.fn(),
    findOne: jest.fn(),
    update: jest.fn(),
    remove: jest.fn(),
  };
  
  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        {
          provide: UsersService,
          useValue: mockUsersService,
        },
      ],
    }).compile();
    
    controller = module.get<UsersController>(UsersController);
    service = module.get<UsersService>(UsersService);
  });
  
  describe('create', () => {
    it('should create a user', async () => {
      const dto = { email: 'test@example.com', name: 'Test', password: 'Pass123!' };
      const expectedResult = { id: '1', ...dto };
      
      mockUsersService.create.mockResolvedValue(expectedResult);
      
      const result = await controller.create(dto);
      
      expect(result).toEqual(expectedResult);
      expect(service.create).toHaveBeenCalledWith(dto);
    });
  });
});
```

---

## Real-World Scenarios

### Scenario 1: Rate Limiting with Guards

```typescript
// common/guards/throttle.guard.ts
import { Injectable, ExecutionContext } from '@nestjs/common';
import { ThrottlerGuard, ThrottlerException } from '@nestjs/throttler';

@Injectable()
export class CustomThrottlerGuard extends ThrottlerGuard {
  protected getTracker(req: Record<string, any>): string {
    // Use user ID if authenticated, IP otherwise
    return req.user?.id || req.ip;
  }
  
  protected throwThrottlingException(): void {
    throw new ThrottlerException('Rate limit exceeded. Try again later.');
  }
}

// Apply to specific routes
@UseGuards(CustomThrottlerGuard)
@Throttle(10, 60) // 10 requests per 60 seconds
@Post('expensive-operation')
async expensiveOperation() {
  // ...
}

import { Throttle } from '@nestjs/throttler';
import { UseGuards, Post } from '@nestjs/common';
```

### Scenario 2: Event-Driven Architecture

```typescript
// events/user-created.event.ts
export class UserCreatedEvent {
  constructor(
    public readonly userId: string,
    public readonly email: string,
    public readonly name: string,
  ) {}
}

// users/users.service.ts
import { Injectable } from '@nestjs/common';
import { EventEmitter2 } from '@nestjs/event-emitter';
import { UserCreatedEvent } from '../events/user-created.event';

@Injectable()
export class UsersService {
  constructor(
    private prisma: any,
    private eventEmitter: EventEmitter2,
  ) {}
  
  async create(createUserDto: any) {
    const user = await this.prisma.user.create({ data: createUserDto });
    
    // Emit event
    this.eventEmitter.emit(
      'user.created',
      new UserCreatedEvent(user.id, user.email, user.name),
    );
    
    return user;
  }
}

// listeners/user-created.listener.ts
import { Injectable } from '@nestjs/common';
import { OnEvent } from '@nestjs/event-emitter';
import { UserCreatedEvent } from '../events/user-created.event';

@Injectable()
export class UserCreatedListener {
  @OnEvent('user.created')
  async handleUserCreated(event: UserCreatedEvent) {
    // Send welcome email
    console.log(`Sending welcome email to ${event.email}`);
  }
  
  @OnEvent('user.created', { async: true })
  async handleUserAnalytics(event: UserCreatedEvent) {
    // Track in analytics
    console.log(`Tracking user creation: ${event.userId}`);
  }
}
```

---

## Common Pitfalls

### 1. Circular Dependencies

```typescript
// ❌ BAD: Circular dependency
// users.module.ts imports auth.module.ts
// auth.module.ts imports users.module.ts

// ✅ GOOD: Use forwardRef
@Module({
  imports: [forwardRef(() => AuthModule)],
})
export class UsersModule {}

@Module({
  imports: [forwardRef(() => UsersModule)],
})
export class AuthModule {}

import { forwardRef, Module } from '@nestjs/common';
```

### 2. Not Using DTOs

```typescript
// ❌ BAD: Raw request body
@Post()
create(@Body() body: any) {
  // No validation, no type safety
}

// ✅ GOOD: Validated DTO
@Post()
create(@Body() createUserDto: CreateUserDto) {
  // Validated and typed
}
```

### 3. Business Logic in Controllers

```typescript
// ❌ BAD: Logic in controller
@Post()
async create(@Body() dto: CreateUserDto) {
  const hashedPassword = await bcrypt.hash(dto.password, 10);
  return this.prisma.user.create({
    data: { ...dto, password: hashedPassword },
  });
}

// ✅ GOOD: Logic in service
@Post()
async create(@Body() dto: CreateUserDto) {
  return this.usersService.create(dto);
}
```

---

## Interview Questions

### Q1: What are the main building blocks of NestJS?

**A:** 
1. **Modules**: Organize related code (controllers, services)
2. **Controllers**: Handle HTTP requests, define routes
3. **Providers/Services**: Business logic, injectable via DI
4. **Guards**: Authorization logic
5. **Interceptors**: Transform requests/responses
6. **Pipes**: Validation and transformation
7. **Filters**: Exception handling

### Q2: How does NestJS dependency injection work?

**A:** NestJS uses an IoC (Inversion of Control) container. Mark classes as `@Injectable()`, register them as providers in modules, and request them via constructor injection. The container resolves dependencies automatically. Supports different scopes: singleton (default), request, transient.

### Q3: What's the difference between Guards and Middleware?

**A:** Middleware runs before route handlers, has access to request/response, but no execution context. Guards run after middleware and before interceptors, have access to ExecutionContext, and return boolean to allow/deny access. Use middleware for general transformations, guards for authorization.

### Q4: How do you handle validation in NestJS?

**A:** Use class-validator decorators in DTOs, enable ValidationPipe globally or per-route. The pipe automatically validates incoming data against the DTO schema and throws BadRequestException with details on validation failure.

---

## Quick Reference Checklist

### Project Setup
- [ ] Use proper module structure
- [ ] Configure global pipes (ValidationPipe)
- [ ] Set up global filters for exceptions
- [ ] Configure Swagger documentation

### Best Practices
- [ ] Keep controllers thin
- [ ] Use DTOs for validation
- [ ] Implement proper error handling
- [ ] Write unit and integration tests
- [ ] Use environment configuration

### Security
- [ ] Implement authentication (Passport + JWT)
- [ ] Add authorization guards
- [ ] Use rate limiting
- [ ] Validate all input

### Performance
- [ ] Use caching where appropriate
- [ ] Implement proper database queries
- [ ] Add logging and monitoring
- [ ] Handle graceful shutdown

---

*Last updated: February 2026*

