# Error Handling Philosophy

> How to think about errors in CareCircle.

---

## The Mental Model

Think of errors like **medical triage**:

- **Critical** (Red) - System-wide failure, needs immediate attention
- **Serious** (Orange) - Feature broken, needs quick fix
- **Manageable** (Yellow) - Expected failure, handle gracefully
- **Informational** (Green) - Not really an error, just needs user guidance

Each type requires a different response strategy.

---

## Error Categories

### The Error Taxonomy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           ERROR CATEGORIES                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  OPERATIONAL ERRORS                   │  PROGRAMMER ERRORS                  │
│  ────────────────────                 │  ─────────────────                  │
│  Expected, can happen in production   │  Bugs that should never happen     │
│                                       │                                     │
│  Examples:                            │  Examples:                          │
│  • User not found (404)               │  • TypeError: undefined             │
│  • Invalid input (400)                │  • RangeError: array index          │
│  • Permission denied (403)            │  • Logic errors                     │
│  • Database connection lost           │  • Missing null checks              │
│  • External API timeout               │                                     │
│                                       │                                     │
│  Response: Handle gracefully          │  Response: Fix the code!            │
│  • Show user-friendly message         │  • Log with stack trace             │
│  • Retry if transient                 │  • Alert developers                 │
│  • Degrade gracefully                 │  • Never show raw error to user    │
│                                       │                                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

### HTTP Status Code Philosophy

```
CLIENT ERRORS (4xx):
"You did something wrong, here's how to fix it"

  400 Bad Request
      → Invalid input format
      → Tell user WHICH field, WHAT'S wrong

  401 Unauthorized
      → No valid credentials
      → Prompt login

  403 Forbidden
      → Valid credentials, but not allowed
      → Explain why (role, ownership)

  404 Not Found
      → Resource doesn't exist
      → Confirm what they were looking for

  409 Conflict
      → Resource already exists
      → Suggest resolution (login instead of register)

  422 Unprocessable Entity
      → Valid format, but business rule violation
      → Explain the rule


SERVER ERRORS (5xx):
"We did something wrong, we're working on it"

  500 Internal Server Error
      → Unexpected failure
      → Generic message to user, detailed log internally

  502 Bad Gateway
      → Upstream service failed
      → "Please try again"

  503 Service Unavailable
      → Overloaded or maintenance
      → "We'll be back soon"
```

---

## Error Handling Patterns

### Pattern 1: Fail Fast

```
PRINCIPLE: Detect errors early, before they cause bigger problems

❌ BAD: Let it fail deep in the stack

async function processPayment(orderId) {
  const order = await getOrder(orderId);  // Could be null!
  const items = order.items;              // TypeError if order is null
  const total = calculateTotal(items);    // Cascading failure
  // ...
}


✅ GOOD: Check early, fail with clear message

async function processPayment(orderId) {
  const order = await getOrder(orderId);
  
  if (!order) {
    throw new NotFoundException(`Order ${orderId} not found`);
  }
  
  if (order.status !== 'pending') {
    throw new BadRequestException(`Order already ${order.status}`);
  }
  
  // Now we KNOW order exists and is pending
  const total = calculateTotal(order.items);
  // ...
}
```

### Pattern 2: Error Boundaries

```
                    ┌─────────────────────────────────────┐
                    │           ERROR BOUNDARIES           │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │     Specific errors caught first    │
                    └─────────────────┬───────────────────┘

try {
  await medicationService.create(dto);
  
} catch (error) {
  // MOST SPECIFIC: Known operational errors
  if (error instanceof NotFoundException) {
    return res.status(404).json({ message: error.message });
  }
  
  if (error instanceof ValidationException) {
    return res.status(400).json({ errors: error.errors });
  }
  
  // LESS SPECIFIC: External service errors
  if (error instanceof ExternalApiError) {
    logger.warn('External API failed', { error });
    return res.status(502).json({ message: 'Service temporarily unavailable' });
  }
  
  // LEAST SPECIFIC: Unexpected errors (programmer errors)
  logger.error('Unexpected error in medication creation', {
    error,
    stack: error.stack,
    dto,
  });
  return res.status(500).json({ message: 'An unexpected error occurred' });
}
```

### Pattern 3: Result Objects (Instead of Exceptions)

```typescript
// Sometimes exceptions aren't the right choice

// For expected "failures" that aren't really errors:

type Result<T, E> = 
  | { success: true; data: T }
  | { success: false; error: E };

// Example: Drug interaction check
function checkInteractions(medications: string[]): Result<void, Interaction[]> {
  const interactions = findInteractions(medications);
  
  if (interactions.length > 0) {
    return { success: false, error: interactions };
  }
  
  return { success: true, data: undefined };
}

// Usage:
const result = checkInteractions(['aspirin', 'ibuprofen']);

if (!result.success) {
  // Show warning, but don't crash
  showInteractionWarning(result.error);
}
```

---

## Frontend Error Handling

### Error Handling Layers

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    FRONTEND ERROR HANDLING LAYERS                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  LAYER 1: REACT ERROR BOUNDARY                                              │
│  ─────────────────────────────                                              │
│  Catches: Render errors, lifecycle errors                                    │
│  Response: Show fallback UI, offer retry                                    │
│                                                                              │
│  LAYER 2: TANSTACK QUERY ERROR HANDLING                                     │
│  ──────────────────────────────────────                                     │
│  Catches: API errors, network failures                                      │
│  Response: Per-query error state, retry logic                               │
│                                                                              │
│  LAYER 3: API INTERCEPTOR                                                   │
│  ────────────────────────                                                   │
│  Catches: 401 (token expired), network errors                               │
│  Response: Refresh token, redirect to login                                 │
│                                                                              │
│  LAYER 4: FORM VALIDATION                                                   │
│  ─────────────────────────                                                  │
│  Catches: Input validation errors                                           │
│  Response: Inline field errors                                              │
│                                                                              │
│  LAYER 5: TOAST NOTIFICATIONS                                               │
│  ─────────────────────────────                                              │
│  Catches: Operation errors (delete failed, etc.)                            │
│  Response: Non-blocking notification                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### User-Facing Error Messages

```
❌ BAD: Technical errors exposed

"Error: ECONNREFUSED ::1:5432"
"TypeError: Cannot read property 'name' of undefined"
"Unhandled Rejection: AxiosError: Network Error"


✅ GOOD: User-friendly messages

"We couldn't connect to the server. Please check your internet connection."
"Something went wrong. We've been notified and are working on it."
"Unable to load medications. Please try again."


MESSAGE FORMULA:
────────────────
1. What happened (briefly)
2. What they can do about it
3. Reassurance (if appropriate)

Example:
"We couldn't save your changes. Please check your connection and try again.
Your data is safe and won't be lost."
```

---

## Backend Error Handling

### NestJS Exception Filter

```typescript
// Global exception filter catches all unhandled errors

@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  constructor(private logger: Logger) {}

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    // Known HTTP exceptions (NotFoundException, ForbiddenException, etc.)
    if (exception instanceof HttpException) {
      const status = exception.getStatus();
      const exceptionResponse = exception.getResponse();
      
      return response.status(status).json({
        success: false,
        statusCode: status,
        message: exception.message,
        errors: exceptionResponse['errors'],  // Validation errors
        timestamp: new Date().toISOString(),
      });
    }

    // Unknown/unexpected errors
    this.logger.error('Unhandled exception', {
      error: exception,
      stack: exception instanceof Error ? exception.stack : undefined,
      request: {
        method: request.method,
        url: request.url,
        body: this.sanitizeBody(request.body),  // Remove passwords!
      },
    });

    // Generic error response (never expose internals)
    return response.status(500).json({
      success: false,
      statusCode: 500,
      message: 'An unexpected error occurred',
      timestamp: new Date().toISOString(),
    });
  }
}
```

### Service-Level Error Handling

```typescript
@Injectable()
export class MedicationsService {
  async create(userId: string, dto: CreateMedicationDto) {
    // Validate ownership FIRST
    const careRecipient = await this.prisma.careRecipient.findUnique({
      where: { id: dto.careRecipientId },
      include: { family: { include: { members: true } } },
    });

    if (!careRecipient) {
      throw new NotFoundException(
        `Care recipient ${dto.careRecipientId} not found`
      );
    }

    const isMember = careRecipient.family.members.some(
      m => m.userId === userId
    );
    
    if (!isMember) {
      throw new ForbiddenException(
        'You are not a member of this family'
      );
    }

    // Now safe to create
    try {
      return await this.prisma.medication.create({
        data: {
          ...dto,
          createdById: userId,
        },
      });
    } catch (error) {
      // Handle specific database errors
      if (error.code === 'P2002') {  // Unique constraint
        throw new ConflictException(
          'A medication with this name already exists'
        );
      }
      throw error;  // Re-throw unknown errors
    }
  }
}
```

---

## Logging Errors

### What to Log

```
ALWAYS LOG:
───────────
• Error message
• Stack trace (for unexpected errors)
• Request context (URL, method, user ID)
• Relevant data (but NOT passwords, tokens)
• Timestamp

EXAMPLE LOG ENTRY:

{
  "level": "error",
  "message": "Failed to create medication",
  "error": "Unique constraint violation",
  "stack": "Error: Unique constraint violation\n    at ...",
  "context": {
    "userId": "user-123",
    "careRecipientId": "recipient-456",
    "medicationName": "Aspirin"
  },
  "request": {
    "method": "POST",
    "url": "/medications",
    "ip": "192.168.1.1"
  },
  "timestamp": "2026-01-30T10:00:00.000Z"
}
```

### What NOT to Log

```
NEVER LOG:
──────────
• Passwords (plain or hashed)
• JWT tokens
• API keys
• Credit card numbers
• Full social security numbers
• Personal health information (PHI) unless necessary

SANITIZE BEFORE LOGGING:

function sanitizeForLogging(data: any) {
  const sanitized = { ...data };
  
  const sensitiveFields = ['password', 'token', 'secret', 'apiKey'];
  sensitiveFields.forEach(field => {
    if (sanitized[field]) {
      sanitized[field] = '[REDACTED]';
    }
  });
  
  return sanitized;
}
```

---

## Error Recovery Strategies

### Retry with Backoff

```
For transient errors (network timeouts, rate limits):

EXPONENTIAL BACKOFF:
────────────────────
Attempt 1: Try immediately
Attempt 2: Wait 1 second, retry
Attempt 3: Wait 2 seconds, retry
Attempt 4: Wait 4 seconds, retry
Attempt 5: Give up, report error

CODE PATTERN:

async function retryWithBackoff<T>(
  operation: () => Promise<T>,
  maxAttempts = 5,
  baseDelay = 1000
): Promise<T> {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await operation();
    } catch (error) {
      if (attempt === maxAttempts) throw error;
      if (!isTransientError(error)) throw error;
      
      const delay = baseDelay * Math.pow(2, attempt - 1);
      await sleep(delay);
    }
  }
}
```

### Graceful Degradation

```
When a feature fails, don't break the whole app:

EXAMPLE: Chat feature down

❌ BAD:
  Chat API error → Show error page → User can't use app

✅ GOOD:
  Chat API error → Hide chat widget → Show "Chat unavailable" →
  Rest of app works normally
```

---

## Quick Reference

### Error Response Format

```json
{
  "success": false,
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    { "field": "email", "message": "must be a valid email" }
  ],
  "timestamp": "2026-01-30T10:00:00.000Z"
}
```

### Common Error Mappings

| Situation | HTTP Status | Message Pattern |
|-----------|-------------|-----------------|
| Invalid input | 400 | "Invalid {field}: {reason}" |
| Not authenticated | 401 | "Please log in to continue" |
| Not authorized | 403 | "You don't have permission to {action}" |
| Not found | 404 | "{Resource} not found" |
| Already exists | 409 | "{Resource} already exists" |
| Business rule | 422 | "Cannot {action} because {reason}" |
| Server error | 500 | "An unexpected error occurred" |

---

*Next: [Logging Best Practices](logging-best-practices.md) | [Retry Strategies](retry-strategies.md)*


