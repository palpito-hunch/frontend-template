# Testing Standards: Mock Usage Policy

## Purpose

This document defines when mocks are appropriate and when real implementations should be used. The goal is to maximize test reliability while keeping tests practical.

## Core Principle

**Default to real implementations. Mocks are exceptions, not the rule.**

Real tests catch real bugs. Mocks can hide integration issues, give false confidence, and create maintenance burden when implementations change.

## When to Use Real Implementations

### Always Use Real Implementations For:

#### 1. Database Operations

```typescript
// CORRECT: Use real test database
describe('MarketService', () => {
  beforeEach(async () => {
    await prisma.market.deleteMany(); // Clean slate
  });

  test('creates market with outcomes', async () => {
    const market = await marketService.createMarket(validMarketData);

    // Verify in actual database
    const saved = await prisma.market.findUnique({
      where: { id: market.id },
      include: { outcomes: true },
    });
    expect(saved.outcomes).toHaveLength(2);
  });
});

// WRONG: Mocking database
jest.mock('@prisma/client');
test('creates market', async () => {
  prisma.market.create.mockResolvedValue(fakeMarket);
  // This tests nothing - just that you called mock correctly
});
```

#### 2. Business Logic and Services

```typescript
// CORRECT: Use real service with real dependencies
const tradeService = new TradeService(realMarketRepository, realUserRepository, realLMSRCalculator);

test('executes trade with correct pricing', async () => {
  const result = await tradeService.executeTrade(tradeData);
  expect(result.cost).toBeCloseTo(expectedLMSRCost, 2);
});

// WRONG: Mocking internal services
const mockCalculator = { calculateCost: jest.fn().mockReturnValue(100) };
// You're not testing the actual calculation!
```

#### 3. Internal Service Integration

```typescript
// CORRECT: Test real service interactions
test('trade updates user balance and position atomically', async () => {
  const initialBalance = await userService.getBalance(userId);

  await tradeService.executeTrade({ userId, marketId, shares: 10 });

  const finalBalance = await userService.getBalance(userId);
  const position = await userService.getPosition(userId, marketId);

  expect(finalBalance).toBeLessThan(initialBalance);
  expect(position.shares).toBe(10);
});
```

#### 4. Property-Based Tests

```typescript
// CORRECT: Property tests MUST use real implementations
test('LMSR prices always sum to 1', () => {
  fc.assert(
    fc.property(
      fc.array(fc.float({ min: 0, max: 1000 }), { minLength: 2, maxLength: 10 }),
      fc.float({ min: 1, max: 100 }),
      (shares, liquidityParam) => {
        const calculator = new LMSRCalculator(); // Real implementation
        const prices = shares.map((_, i) => calculator.calculatePrice(shares, i, liquidityParam));
        const sum = prices.reduce((a, b) => a + b, 0);
        return Math.abs(sum - 1) < 0.0001;
      }
    ),
    { numRuns: 100 }
  );
});
```

#### 5. Validation Logic

```typescript
// CORRECT: Test real validation
test('rejects invalid market data', async () => {
  await expect(marketService.createMarket({ question: '', outcomes: [] })).rejects.toThrow(
    ValidationError
  );
});
```

## When Mocks Are Acceptable

### 1. External Third-Party APIs

```typescript
// ACCEPTABLE: Mock external payment provider
jest.mock('../services/stripe-client');

test('handles payment failure gracefully', async () => {
  stripeClient.charge.mockRejectedValue(new StripeError('Card declined'));

  await expect(paymentService.processDeposit(userId, 100)).rejects.toThrow('Payment failed');

  // Verify user balance unchanged
  const balance = await userService.getBalance(userId);
  expect(balance).toBe(originalBalance);
});
```

### 2. Non-Deterministic Behavior

```typescript
// ACCEPTABLE: Mock time for consistent tests
jest.useFakeTimers();

test('market expires after deadline', async () => {
  const market = await marketService.createMarket({
    ...validData,
    expiresAt: new Date(Date.now() + 3600000), // 1 hour
  });

  jest.advanceTimersByTime(3600001); // Advance past expiry

  const status = await marketService.getMarketStatus(market.id);
  expect(status).toBe('expired');
});

// ACCEPTABLE: Mock randomness
jest.spyOn(Math, 'random').mockReturnValue(0.5);
```

### 3. Prohibitively Slow Operations (>5 seconds)

```typescript
// ACCEPTABLE: Mock slow external service
// Only if the operation genuinely takes >5s and can't be optimized

jest.mock('../services/slow-analytics-service');

test('records trade analytics', async () => {
  slowAnalytics.recordTrade.mockResolvedValue({ success: true });

  await tradeService.executeTrade(tradeData);

  expect(slowAnalytics.recordTrade).toHaveBeenCalledWith(
    expect.objectContaining({ marketId, userId })
  );
});
```

### 4. Complex Failure Scenarios

```typescript
// ACCEPTABLE: Mock to simulate hard-to-trigger failures
test('handles database connection failure', async () => {
  // Temporarily mock to simulate connection loss
  const originalFindUnique = prisma.market.findUnique;
  prisma.market.findUnique = jest
    .fn()
    .mockRejectedValue(new PrismaClientKnownRequestError('Connection lost', { code: 'P1001' }));

  await expect(marketService.getMarket('some-id')).rejects.toThrow('Database unavailable');

  // Restore real implementation
  prisma.market.findUnique = originalFindUnique;
});
```

### 5. WebSocket/Real-time Testing (Connection Layer Only)

```typescript
// ACCEPTABLE: Mock socket connections, not business logic
const mockSocket = {
  emit: jest.fn(),
  on: jest.fn(),
  join: jest.fn(),
};

test('broadcasts market update to subscribers', async () => {
  await broadcastService.notifyMarketUpdate(marketId, updateData);

  expect(mockSocket.emit).toHaveBeenCalledWith(
    'market:updated',
    expect.objectContaining({ marketId })
  );
});
```

## Mock Decision Framework

Before adding a mock, ask:

```
1. Is this an external third-party service?
   YES -> Mock is acceptable
   NO  -> Continue to #2

2. Is behavior non-deterministic (time, random)?
   YES -> Mock is acceptable
   NO  -> Continue to #3

3. Does this operation take >5 seconds?
   YES -> Mock is acceptable (but consider if it can be optimized)
   NO  -> Continue to #4

4. Is this simulating a failure that can't be reliably triggered?
   YES -> Mock is acceptable
   NO  -> USE REAL IMPLEMENTATION
```

## Anti-Patterns to Avoid

### 1. Mocking the System Under Test

```typescript
// WRONG: You're testing nothing
jest.mock('../services/MarketService');
test('creates market', () => {
  MarketService.createMarket.mockResolvedValue(fakeMarket);
  // This is circular - you're testing your mock
});
```

### 2. Mocking Database to Avoid Setup

```typescript
// WRONG: Lazy mocking
jest.mock('@prisma/client');
test('gets user', () => {
  prisma.user.findUnique.mockResolvedValue({ id: '1', balance: 100 });
  // Write proper test setup instead!
});

// CORRECT: Proper test setup
beforeEach(async () => {
  testUser = await prisma.user.create({
    data: { email: 'test@example.com', balance: 100 },
  });
});
```

### 3. Mocking Internal Services

```typescript
// WRONG: Hiding integration issues
const mockLMSR = { calculateCost: jest.fn().mockReturnValue(50) };
const tradeService = new TradeService(mockLMSR);

// CORRECT: Use real LMSR
const realLMSR = new LMSRCalculator();
const tradeService = new TradeService(realLMSR);
```

### 4. Excessive Mock Setup

```typescript
// WRONG: If setup is this complex, you're mocking too much
beforeEach(() => {
  mockUserRepo.findById.mockResolvedValue(mockUser);
  mockMarketRepo.findById.mockResolvedValue(mockMarket);
  mockLMSR.calculateCost.mockReturnValue(50);
  mockLMSR.calculatePrice.mockReturnValue(0.5);
  mockPositionRepo.create.mockResolvedValue(mockPosition);
  // ... 10 more mocks
});

// CORRECT: Use real implementations with test database
beforeEach(async () => {
  testUser = await createTestUser();
  testMarket = await createTestMarket();
});
```

## Test Database Setup

To enable real database testing:

```typescript
// jest.setup.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

beforeAll(async () => {
  // Use test database
  process.env.DATABASE_URL = process.env.TEST_DATABASE_URL;
});

beforeEach(async () => {
  // Clean database before each test
  await prisma.$transaction([
    prisma.trade.deleteMany(),
    prisma.position.deleteMany(),
    prisma.outcome.deleteMany(),
    prisma.market.deleteMany(),
    prisma.user.deleteMany(),
  ]);
});

afterAll(async () => {
  await prisma.$disconnect();
});
```

## Summary Table

| Scenario                 | Mock?  | Reason                                      |
| ------------------------ | ------ | ------------------------------------------- |
| Database operations      | ❌ No  | Core functionality, must test real behavior |
| Business logic           | ❌ No  | Core functionality, must verify correctness |
| Internal services        | ❌ No  | Integration issues hidden by mocks          |
| Property-based tests     | ❌ No  | Must verify mathematical properties         |
| Validation               | ❌ No  | Must test real validation rules             |
| External APIs            | ✅ Yes | Can't control third-party behavior          |
| Time/randomness          | ✅ Yes | Non-deterministic, need reproducibility     |
| Slow operations (>5s)    | ✅ Yes | Practical test performance                  |
| Hard-to-trigger failures | ✅ Yes | Can't reliably reproduce                    |

---

**Remember:** Every mock is a lie you tell your test suite. Use them sparingly and only when the alternative is impractical.
