# File Organization and Naming Standards

## Purpose

Consistent file organization and naming conventions improve code navigation, maintainability, and team collaboration.

---

## Directory Structure

### Standard Project Layout

```
project-root/
├── src/
│   ├── services/              # Business logic layer
│   │   ├── __tests__/        # Service tests (co-located)
│   │   │   ├── market.service.test.ts
│   │   │   ├── trade.service.test.ts
│   │   │   └── user.service.test.ts
│   │   ├── market.service.ts
│   │   ├── trade.service.ts
│   │   └── user.service.ts
│   │
│   ├── repositories/          # Data access layer
│   │   ├── __tests__/
│   │   │   └── market.repository.test.ts
│   │   ├── market.repository.ts
│   │   ├── trade.repository.ts
│   │   └── user.repository.ts
│   │
│   ├── controllers/           # API request handlers
│   │   ├── __tests__/
│   │   ├── market.controller.ts
│   │   └── trade.controller.ts
│   │
│   ├── middleware/            # Express/API middleware
│   │   ├── auth.middleware.ts
│   │   ├── error.middleware.ts
│   │   └── validation.middleware.ts
│   │
│   ├── utils/                 # Pure utility functions
│   │   ├── __tests__/
│   │   │   ├── lmsr.util.test.ts
│   │   │   └── validation.util.test.ts
│   │   ├── lmsr.util.ts
│   │   ├── validation.util.ts
│   │   └── math.util.ts
│   │
│   ├── types/                 # TypeScript type definitions
│   │   ├── market.types.ts
│   │   ├── trade.types.ts
│   │   ├── user.types.ts
│   │   └── index.ts           # Re-export all types
│   │
│   ├── errors/                # Custom error classes
│   │   ├── business-logic.error.ts
│   │   ├── validation.error.ts
│   │   ├── not-found.error.ts
│   │   └── index.ts           # Re-export all errors
│   │
│   ├── config/                # Configuration files
│   │   ├── database.config.ts
│   │   ├── server.config.ts
│   │   └── constants.ts
│   │
│   ├── migrations/            # Database migrations
│   │   └── YYYYMMDDHHMMSS_migration_name.ts
│   │
│   └── index.ts              # Application entry point
│
├── tests/                     # Integration and E2E tests
│   ├── integration/
│   │   ├── market-flow.test.ts
│   │   └── trade-flow.test.ts
│   ├── e2e/
│   │   └── user-journey.test.ts
│   └── helpers/
│       ├── test-database.ts
│       └── test-fixtures.ts
│
├── docs/                      # Documentation
│   ├── api/
│   ├── architecture/
│   └── standards/
│
├── scripts/                   # Utility scripts
│   ├── seed-database.ts
│   └── generate-types.ts
│
└── prisma/                    # Prisma schema and migrations
    ├── schema.prisma
    └── migrations/
```

---

## File Naming Conventions

### General Rules

- Use **lowercase** with **kebab-case** (hyphens) for file names
- Use descriptive names that indicate the file's purpose
- Include appropriate suffixes to indicate file type
- Be consistent across the entire codebase

### Naming Patterns

#### Services

```
market.service.ts
trade.service.ts
user.service.ts
notification.service.ts
```

#### Repositories

```
market.repository.ts
trade.repository.ts
user.repository.ts
```

#### Controllers

```
market.controller.ts
trade.controller.ts
auth.controller.ts
```

#### Utilities

```
lmsr.util.ts
validation.util.ts
date.util.ts
math.util.ts
```

#### Types

```
market.types.ts
trade.types.ts
user.types.ts
api.types.ts
```

#### Tests

```
market.service.test.ts        # Co-located with service
lmsr.util.test.ts            # Co-located with util
market-integration.test.ts    # Integration test
```

#### Errors

```
business-logic.error.ts
validation.error.ts
not-found.error.ts
insufficient-balance.error.ts
```

#### Middleware

```
auth.middleware.ts
error.middleware.ts
logging.middleware.ts
rate-limit.middleware.ts
```

#### Configuration

```
database.config.ts
server.config.ts
constants.ts
environment.ts
```

---

## Class and Interface Naming

### Classes

**Format:** PascalCase

```typescript
// Services
class MarketService {}
class TradeService {}
class UserService {}

// Repositories
class MarketRepository {}
class TradeRepository {}

// Controllers
class MarketController {}
class TradeController {}

// Errors
class BusinessLogicError extends Error {}
class ValidationError extends Error {}
class MarketNotFoundError extends BusinessLogicError {}
```

### Interfaces

**Format:** PascalCase (prefer without 'I' prefix)

```typescript
// ✅ Preferred - No 'I' prefix
interface MarketService {}
interface TradeRepository {}
interface PriceCalculator {}

// ❌ Avoid - 'I' prefix
interface IMarketService {}
interface ITradeRepository {}
```

**Exception:** Use 'I' prefix only when interface and class have same name and exist in same file:

```typescript
// Acceptable when needed for disambiguation
interface IMarketService {
  createMarket(data: MarketData): Promise<Market>;
}

class MarketService implements IMarketService {
  async createMarket(data: MarketData): Promise<Market> {
    // Implementation
  }
}
```

### Types

**Format:** PascalCase

```typescript
type MarketStatus = 'ACTIVE' | 'RESOLVED' | 'CANCELLED';
type TradeDirection = 'BUY' | 'SELL';
type UserId = string;
type Timestamp = number;

// Complex types
type MarketWithOutcomes = Market & { outcomes: Outcome[] };
type CreateMarketDto = Omit<Market, 'id' | 'createdAt'>;
```

---

## Function and Variable Naming

### Functions

**Format:** camelCase, descriptive verbs

```typescript
// ✅ Good - Clear, descriptive
function calculateMarketPrice(shares: number[], outcome: number): number {}
function validateTradeAmount(amount: number): void {}
function getUserBalance(userId: string): Promise<number> {}
function isMarketResolved(market: Market): boolean {}

// ❌ Bad - Too short, unclear
function calc(s: number[], o: number): number {}
function validate(a: number): void {}
function get(id: string): Promise<number> {}
function check(m: Market): boolean {}
```

### Variables

**Format:** camelCase, descriptive nouns

```typescript
// ✅ Good
const marketPrice = calculatePrice(shares, outcome);
const totalVolume = trades.reduce((sum, t) => sum + t.amount, 0);
const userBalance = await getUserBalance(userId);
const isResolved = market.resolved;

// ❌ Bad
const p = calculatePrice(shares, outcome);
const vol = trades.reduce((sum, t) => sum + t.amount, 0);
const bal = await getUserBalance(userId);
const res = market.resolved;
```

### Constants

**Format:** SCREAMING_SNAKE_CASE for true constants

```typescript
// ✅ Good - True constants
const MAX_TRADE_AMOUNT = 10000;
const MIN_MARKET_OUTCOMES = 2;
const DEFAULT_LIQUIDITY_PARAMETER = 250;
const API_VERSION = 'v1';

// ✅ Also acceptable - Configuration objects
const DATABASE_CONFIG = {
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT || '5432'),
  database: process.env.DB_NAME,
};

// ❌ Bad - Not constants
const max_trade = calculateMax(); // This is calculated, use camelCase
const MIN_amount = getMinimum(); // This is fetched, use camelCase
```

### Boolean Variables

**Format:** Use `is`, `has`, `can`, `should` prefixes

```typescript
// ✅ Good - Clear boolean intent
const isResolved = market.resolved;
const hasBalance = user.balance > 0;
const canTrade = !market.resolved && user.verified;
const shouldNotify = user.preferences.notifications;

// ❌ Bad - Unclear boolean intent
const resolved = market.resolved;
const balance = user.balance > 0;
const trade = !market.resolved && user.verified;
```

---

## File Size Guidelines

### Size Limits

**Soft Limits (Recommend splitting):**

- **Services:** 300 lines
- **Repositories:** 200 lines
- **Controllers:** 200 lines
- **Utilities:** 150 lines
- **Types:** 200 lines

**Hard Limits (Must split):**

- **Any file:** 500 lines

**Exceptions:**

- Complex algorithms with justification (document why)
- Generated code
- Configuration files

### When to Split Files

**Split when:**

- File exceeds 300 lines
- Class has multiple distinct responsibilities
- File contains unrelated functions
- Difficult to navigate or understand

**How to split:**

```typescript
// Before: market.service.ts (450 lines)
class MarketService {
  // Creation methods (100 lines)
  async createMarket() {}
  async createOutcomes() {}

  // Trading methods (150 lines)
  async placeTrade() {}
  async calculatePrice() {}

  // Resolution methods (100 lines)
  async resolveMarket() {}
  async distributePayouts() {}

  // Analytics methods (100 lines)
  async getMarketStats() {}
  async getVolumeHistory() {}
}

// After: Split into focused services
// market-creation.service.ts (120 lines)
class MarketCreationService {
  async createMarket() {}
  async createOutcomes() {}
}

// market-trading.service.ts (160 lines)
class MarketTradingService {
  async placeTrade() {}
  async calculatePrice() {}
}

// market-resolution.service.ts (120 lines)
class MarketResolutionService {
  async resolveMarket() {}
  async distributePayouts() {}
}

// market-analytics.service.ts (120 lines)
class MarketAnalyticsService {
  async getMarketStats() {}
  async getVolumeHistory() {}
}
```

---

## Import Organization

### Import Order

```typescript
// 1. Node.js built-in modules
import { readFileSync } from 'fs';
import { join } from 'path';

// 2. External dependencies (alphabetical)
import { Decimal } from 'decimal.js';
import { PrismaClient } from '@prisma/client';
import express from 'express';

// 3. Internal imports - types
import type { Market, Trade, User } from '@/types';
import type { MarketRepository, TradeRepository } from '@/repositories';

// 4. Internal imports - services
import { MarketService } from '@/services/market.service';
import { TradeService } from '@/services/trade.service';

// 5. Internal imports - utilities
import { calculatePrice } from '@/utils/lmsr.util';
import { validateAmount } from '@/utils/validation.util';

// 6. Internal imports - errors
import { BusinessLogicError, ValidationError } from '@/errors';

// 7. Internal imports - config/constants
import { MAX_TRADE_AMOUNT, MIN_OUTCOMES } from '@/config/constants';
```

### Import Aliases

**Configure in tsconfig.json:**

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/types": ["src/types"],
      "@/services": ["src/services"],
      "@/repositories": ["src/repositories"],
      "@/utils": ["src/utils"],
      "@/errors": ["src/errors"],
      "@/config": ["src/config"]
    }
  }
}
```

**Usage:**

```typescript
// ✅ Good - Clean, absolute imports
import { MarketService } from '@/services/market.service';
import { Market } from '@/types/market.types';
import { calculatePrice } from '@/utils/lmsr.util';

// ❌ Bad - Relative paths are harder to maintain
import { MarketService } from '../../../services/market.service';
import { Market } from '../../types/market.types';
import { calculatePrice } from '../utils/lmsr.util';
```

---

## Export Patterns

### Named Exports (Preferred)

```typescript
// market.service.ts
export class MarketService {
  // Implementation
}

export async function calculateMarketPrice(): Promise<number> {
  // Implementation
}

// Usage
import { MarketService, calculateMarketPrice } from '@/services/market.service';
```

### Default Exports (Use Sparingly)

```typescript
// Only for main class/function of file
export default class MarketService {
  // Implementation
}

// Usage
import MarketService from '@/services/market.service';
```

### Index Files for Re-exports

```typescript
// types/index.ts
export * from './market.types';
export * from './trade.types';
export * from './user.types';

// Usage
import { Market, Trade, User } from '@/types';
```

---

## Documentation in Files

### File Headers

```typescript
/**
 * Market Trading Service
 *
 * Handles all market trading operations including:
 * - Trade placement and validation
 * - Price calculations using LMSR
 * - Position management
 *
 * @module services/market-trading
 */

export class MarketTradingService {
  // Implementation
}
```

### Complex Files

```typescript
/**
 * LMSR (Logarithmic Market Scoring Rule) Implementation
 *
 * This module implements the LMSR automated market maker algorithm
 * for prediction markets. LMSR ensures the market maker never runs
 * out of liquidity while providing continuous pricing.
 *
 * Key concepts:
 * - Liquidity parameter (b): Controls market depth and price sensitivity
 * - Cost function: C(q) = b * ln(sum(exp(q_i / b)))
 * - Price: Marginal cost of increasing outcome shares
 *
 * References:
 * - Hanson (2002): "Logarithmic Market Scoring Rules for Modular
 *   Combinatorial Information Aggregation"
 * - Chen & Pennock (2007): "A Utility Framework for Bounded-Loss
 *   Market Makers"
 *
 * @module utils/lmsr
 */
```

---

## Testing File Organization

### Co-located Tests

```
src/services/
├── market.service.ts
├── __tests__/
│   └── market.service.test.ts
```

### Test File Structure

```typescript
// market.service.test.ts
import { MarketService } from '../market.service';
import { createTestDatabase } from '@/tests/helpers/test-database';

describe('MarketService', () => {
  let service: MarketService;
  let testDb: TestDatabase;

  beforeAll(async () => {
    testDb = await createTestDatabase();
    service = new MarketService(testDb.prisma);
  });

  afterAll(async () => {
    await testDb.cleanup();
  });

  describe('createMarket', () => {
    test('should create market with valid data', async () => {
      // Test implementation
    });

    test('should throw error for invalid data', async () => {
      // Test implementation
    });
  });

  describe('resolveMarket', () => {
    // More tests
  });
});
```

---

## Summary: Quick Reference

### File Names

- **Services:** `market.service.ts`
- **Repositories:** `market.repository.ts`
- **Controllers:** `market.controller.ts`
- **Utils:** `lmsr.util.ts`
- **Types:** `market.types.ts`
- **Tests:** `market.service.test.ts`
- **Errors:** `business-logic.error.ts`

### Class Names

- **PascalCase:** `MarketService`, `TradeRepository`, `ValidationError`

### Function Names

- **camelCase:** `calculatePrice`, `validateAmount`, `getUserBalance`

### Variable Names

- **camelCase:** `marketPrice`, `totalVolume`, `userBalance`
- **Booleans:** `isResolved`, `hasBalance`, `canTrade`

### Constants

- **SCREAMING_SNAKE_CASE:** `MAX_TRADE_AMOUNT`, `MIN_OUTCOMES`

### File Size

- **Soft limit:** 300 lines
- **Hard limit:** 500 lines
- **Split when exceeded**

**Remember:** Consistency is more valuable than perfection. Choose a pattern and apply it uniformly across the codebase.
