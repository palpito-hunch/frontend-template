# Error Message Standards

## Purpose

Error messages must be actionable, informative, and help developers and users understand and resolve issues quickly.

---

## Core Principles

### Every Error Message Must Include:

1. **What went wrong** (specific, not generic)
2. **Why it went wrong** (context and state)
3. **What to do about it** (if applicable)
4. **Relevant data** (IDs, values, current state)

---

## Error Structure

### Standard Error Class Pattern

```typescript
export class BusinessLogicError extends Error {
  constructor(
    message: string, // Human-readable description
    public readonly code: string, // Machine-readable error code
    public readonly context?: Record<string, any>, // Additional context
    public readonly httpStatus: number = 400
  ) {
    super(message);
    this.name = 'BusinessLogicError';
    Error.captureStackTrace(this, this.constructor);
  }
}
```

### Error Code Naming Convention

**Format:** `UPPERCASE_SNAKE_CASE`

**Pattern:** `ENTITY_ISSUE_TYPE`

**Examples:**

- `MARKET_NOT_FOUND`
- `MARKET_ALREADY_RESOLVED`
- `INSUFFICIENT_BALANCE`
- `INVALID_OUTCOME_INDEX`
- `CONCURRENT_MODIFICATION_DETECTED`
- `TRADE_AMOUNT_OUT_OF_RANGE`

**Not:**

- `ERR_001` (meaningless numbers)
- `error` (too generic)
- `marketError` (inconsistent casing)
- `market_not_found` (lowercase)

---

## Error Message Examples

### ❌ Bad Error Messages (Too Generic)

```typescript
throw new Error('Invalid input');
throw new Error('Operation failed');
throw new Error('Not found');
throw new Error('Error occurred');
throw new Error('Bad request');
```

**Problems:**

- No context about what input was invalid
- No information about what operation failed
- No clue what wasn't found
- No actionable guidance

---

### ✅ Good Error Messages (Actionable and Informative)

#### Example 1: Market Not Found

```typescript
// ❌ BAD
throw new Error('Market not found');

// ✅ GOOD
throw new BusinessLogicError(
  `Market with ID '${marketId}' does not exist`,
  'MARKET_NOT_FOUND',
  {
    marketId,
    requestedBy: userId,
    timestamp: new Date().toISOString(),
  },
  404
);
```

#### Example 2: Market Already Resolved

```typescript
// ❌ BAD
throw new Error('Cannot trade on this market');

// ✅ GOOD
throw new BusinessLogicError(
  `Cannot place trade: market '${market.title}' was resolved on ${market.resolvedAt} with outcome ${market.winningOutcome}`,
  'MARKET_ALREADY_RESOLVED',
  {
    marketId: market.id,
    marketTitle: market.title,
    resolvedAt: market.resolvedAt,
    winningOutcome: market.winningOutcome,
    attemptedBy: userId,
  },
  409
);
```

#### Example 3: Insufficient Balance

```typescript
// ❌ BAD
throw new Error('Not enough money');

// ✅ GOOD
throw new BusinessLogicError(
  `Insufficient balance: you have $${user.balance.toFixed(2)} but need $${requiredAmount.toFixed(2)} (shortfall: $${shortfall.toFixed(2)})`,
  'INSUFFICIENT_BALANCE',
  {
    userId: user.id,
    currentBalance: user.balance,
    requiredAmount: requiredAmount,
    shortfall: shortfall,
    operation: 'place_trade',
    marketId: marketId,
  },
  402
);
```

#### Example 4: Invalid Trade Amount

```typescript
// ❌ BAD
throw new Error('Invalid amount');

// ✅ GOOD
throw new ValidationError(
  `Trade amount must be between $${MIN_TRADE} and $${MAX_TRADE}, got $${amount}`,
  'TRADE_AMOUNT_OUT_OF_RANGE',
  {
    provided: amount,
    minimum: MIN_TRADE,
    maximum: MAX_TRADE,
    marketId: marketId,
    userId: userId,
  }
);
```

#### Example 5: Invalid Outcome Index

```typescript
// ❌ BAD
throw new Error('Invalid outcome');

// ✅ GOOD
throw new ValidationError(
  `Invalid outcome index ${outcomeIndex}: market '${market.title}' has ${market.outcomes.length} outcomes (valid indices: 0-${market.outcomes.length - 1})`,
  'INVALID_OUTCOME_INDEX',
  {
    provided: outcomeIndex,
    marketId: market.id,
    marketTitle: market.title,
    totalOutcomes: market.outcomes.length,
    validRange: [0, market.outcomes.length - 1],
    availableOutcomes: market.outcomes.map((o) => ({
      id: o.id,
      name: o.name,
      index: o.index,
    })),
  }
);
```

#### Example 6: Concurrent Modification

```typescript
// ❌ BAD
throw new Error('Conflict');

// ✅ GOOD
throw new BusinessLogicError(
  `Concurrent modification detected: market '${market.title}' was modified by another operation. Please retry your request.`,
  'CONCURRENT_MODIFICATION_DETECTED',
  {
    marketId: market.id,
    marketTitle: market.title,
    operation: 'resolve_market',
    attemptedBy: userId,
    retryable: true,
    suggestion: 'This is a temporary conflict. Please try again.',
  },
  409
);
```

#### Example 7: Invalid Market State

```typescript
// ❌ BAD
throw new Error('Market in wrong state');

// ✅ GOOD
throw new BusinessLogicError(
  `Cannot resolve market: market '${market.title}' is in '${market.status}' state but must be 'ACTIVE' to resolve`,
  'INVALID_MARKET_STATE',
  {
    marketId: market.id,
    marketTitle: market.title,
    currentState: market.status,
    requiredState: 'ACTIVE',
    operation: 'resolve',
    allowedTransitions: ['ACTIVE -> RESOLVED'],
  }
);
```

#### Example 8: Missing Required Field

```typescript
// ❌ BAD
throw new Error('Missing field');

// ✅ GOOD
throw new ValidationError(
  `Required field 'outcomes' is missing or empty. Markets must have at least 2 outcomes.`,
  'MISSING_REQUIRED_FIELD',
  {
    field: 'outcomes',
    requirement: 'At least 2 outcomes',
    received: data.outcomes,
    example: ['Yes', 'No'],
  }
);
```

---

## Validation Error Patterns

### Input Validation

```typescript
class ValidationError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly context?: Record<string, any>
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}

// Use in validation functions
function validateTradeAmount(amount: number): void {
  if (amount <= 0) {
    throw new ValidationError(
      `Trade amount must be positive, got ${amount}`,
      'NEGATIVE_TRADE_AMOUNT',
      { provided: amount, minimum: 0.01 }
    );
  }

  if (amount > MAX_TRADE_AMOUNT) {
    throw new ValidationError(
      `Trade amount ${amount} exceeds maximum allowed ${MAX_TRADE_AMOUNT}`,
      'TRADE_AMOUNT_TOO_LARGE',
      { provided: amount, maximum: MAX_TRADE_AMOUNT }
    );
  }

  if (!Number.isFinite(amount)) {
    throw new ValidationError(
      `Trade amount must be a finite number, got ${amount}`,
      'INVALID_TRADE_AMOUNT',
      { provided: amount, type: typeof amount }
    );
  }
}
```

### Array Validation

```typescript
function validateOutcomes(outcomes: string[]): void {
  if (!Array.isArray(outcomes)) {
    throw new ValidationError(
      `Outcomes must be an array, got ${typeof outcomes}`,
      'INVALID_OUTCOMES_TYPE',
      { provided: outcomes, expected: 'array' }
    );
  }

  if (outcomes.length < 2) {
    throw new ValidationError(
      `Market must have at least 2 outcomes, got ${outcomes.length}`,
      'INSUFFICIENT_OUTCOMES',
      {
        provided: outcomes.length,
        minimum: 2,
        receivedOutcomes: outcomes,
      }
    );
  }

  const emptyOutcomes = outcomes.filter((o) => !o || o.trim().length === 0);
  if (emptyOutcomes.length > 0) {
    throw new ValidationError(
      `All outcomes must have non-empty names. Found ${emptyOutcomes.length} empty outcomes.`,
      'EMPTY_OUTCOME_NAMES',
      {
        totalOutcomes: outcomes.length,
        emptyCount: emptyOutcomes.length,
        outcomes: outcomes.map((o, i) => ({
          index: i,
          name: o,
          isEmpty: !o || o.trim().length === 0,
        })),
      }
    );
  }
}
```

---

## Database Error Handling

### Prisma Error Translation

```typescript
import { Prisma } from '@prisma/client';

function translatePrismaError(error: unknown, context: Record<string, any>): Error {
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    switch (error.code) {
      case 'P2002': // Unique constraint violation
        const target = (error.meta?.target as string[]) || [];
        return new BusinessLogicError(
          `Duplicate entry: ${target.join(', ')} already exists`,
          'DUPLICATE_ENTRY',
          {
            ...context,
            constraintFields: target,
            originalError: error.message,
          },
          409
        );

      case 'P2025': // Record not found
        return new BusinessLogicError(
          `Record not found for operation`,
          'RECORD_NOT_FOUND',
          {
            ...context,
            originalError: error.message,
          },
          404
        );

      case 'P2034': // Transaction conflict
        return new BusinessLogicError(
          `Transaction conflict detected. Please retry your operation.`,
          'CONCURRENT_MODIFICATION_DETECTED',
          {
            ...context,
            retryable: true,
            originalError: error.message,
          },
          409
        );

      default:
        return new BusinessLogicError(
          `Database operation failed: ${error.message}`,
          'DATABASE_ERROR',
          {
            ...context,
            prismaErrorCode: error.code,
            originalError: error.message,
          },
          500
        );
    }
  }

  // Unknown error
  return new BusinessLogicError(
    'An unexpected error occurred',
    'INTERNAL_ERROR',
    {
      ...context,
      originalError: String(error),
    },
    500
  );
}

// Usage
try {
  await prisma.market.create({ data: marketData });
} catch (error) {
  throw translatePrismaError(error, {
    operation: 'create_market',
    userId: userId,
    marketData: marketData,
  });
}
```

---

## Race Condition Error Messages

```typescript
// ✅ GOOD - Specific race condition context
async function resolveMarket(id: string, outcome: number): Promise<void> {
  try {
    await prisma.$transaction(async (tx) => {
      const market = await tx.market.findUnique({ where: { id } });

      if (market.resolved) {
        throw new BusinessLogicError(
          `Cannot resolve market: already resolved by ${market.resolvedBy} at ${market.resolvedAt} with outcome ${market.winningOutcome}`,
          'MARKET_ALREADY_RESOLVED',
          {
            marketId: id,
            resolvedAt: market.resolvedAt,
            resolvedBy: market.resolvedBy,
            winningOutcome: market.winningOutcome,
            attemptedOutcome: outcome,
            suggestion: 'Another user may have resolved this market concurrently',
          }
        );
      }

      // Resolution logic...
    });
  } catch (error) {
    if (error instanceof Prisma.PrismaClientKnownRequestError && error.code === 'P2034') {
      throw new BusinessLogicError(
        `Concurrent resolution detected for market '${id}'. Another resolution is in progress. Please wait and check the market status.`,
        'CONCURRENT_RESOLUTION',
        {
          marketId: id,
          retryable: false,
          suggestion: 'Check market status before retrying',
        }
      );
    }
    throw error;
  }
}
```

---

## Error Response Format (API)

### Standard Error Response Structure

```typescript
interface ErrorResponse {
  success: false;
  error: {
    message: string; // Human-readable message
    code: string; // Machine-readable error code
    context?: Record<string, any>; // Additional context
    timestamp: string; // ISO 8601 timestamp
    requestId: string; // Unique request identifier
  };
}

// Example error handler
app.use((error: Error, req: Request, res: Response, next: NextFunction) => {
  const errorResponse: ErrorResponse = {
    success: false,
    error: {
      message: error.message,
      code: (error as any).code || 'INTERNAL_ERROR',
      context: (error as any).context,
      timestamp: new Date().toISOString(),
      requestId: req.id || generateRequestId(),
    },
  };

  const status = (error as any).httpStatus || 500;

  // Log internal errors
  if (status >= 500) {
    logger.error('Internal server error', {
      error: error,
      requestId: errorResponse.error.requestId,
      endpoint: req.path,
      method: req.method,
    });
  }

  res.status(status).json(errorResponse);
});
```

---

## Logging Error Context

### Include Relevant Context in Logs

```typescript
logger.error('Market resolution failed', {
  error: error.message,
  errorCode: error.code,
  marketId: marketId,
  attemptedOutcome: outcome,
  userId: userId,
  marketState: {
    resolved: market.resolved,
    status: market.status,
    resolvedAt: market.resolvedAt,
  },
  timestamp: new Date().toISOString(),
  stackTrace: error.stack,
});
```

---

## Summary: Error Message Checklist

When creating error messages, ensure:

- [ ] **Specific:** States exactly what went wrong, not generic "error occurred"
- [ ] **Contextual:** Includes relevant IDs, values, and current state
- [ ] **Actionable:** Tells user/developer what they can do about it
- [ ] **Consistent:** Uses standard error codes and structure
- [ ] **Machine-readable:** Includes error code for programmatic handling
- [ ] **Loggable:** Contains enough context for debugging

### Quick Reference

**Bad Error:**

```typescript
throw new Error('Invalid input');
```

**Good Error:**

```typescript
throw new ValidationError(
  `Field 'amount' must be between ${MIN} and ${MAX}, got ${value}`,
  'AMOUNT_OUT_OF_RANGE',
  { field: 'amount', provided: value, minimum: MIN, maximum: MAX }
);
```

**Remember:** Error messages are for humans who need to fix problems. Make them helpful, not cryptic.
