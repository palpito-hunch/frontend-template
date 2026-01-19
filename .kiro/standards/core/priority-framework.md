# AI Development Priority Framework

## Purpose

This document provides explicit guidance for AI-driven development on how to resolve conflicts between coding standards, when to apply patterns, and how to make trade-off decisions.

---

## Before Implementation: Validate the Approach

**Before writing any code, spend 2 minutes validating that the requested approach is appropriate.**

### Why This Matters

Implementing the wrong solution wastes time and creates technical debt. A quick validation can prevent:
- Using deprecated APIs or patterns
- Reinventing what already exists
- Missing better first-party solutions
- Building overly complex solutions

### Validation Checklist

Before implementing, ask:

1. **Is this approach current?**
   - Is it deprecated or superseded by something better?
   - When was it last recommended? (Patterns from 2015 may be outdated)

2. **Is there a native/first-party solution?**
   - Does the platform offer this built-in?
   - Is there an official integration or app?

3. **Is it the right level of abstraction?**
   - Am I building too low-level when a library exists?
   - Am I over-engineering when a simple solution works?

4. **What do the official docs recommend?**
   - Check the service's documentation for current best practices
   - Look for "Getting Started" or "Integration" guides

### Real-World Examples

| Request | Problem | Better Approach |
|---------|---------|-----------------|
| "Add Slack webhook for GitHub notifications" | Webhooks are legacy; requires custom code, secrets management | Use Slack's GitHub App (native, maintained, richer features) |
| "Write a JWT validation library" | Security-critical, easy to get wrong | Use established library (jose, jsonwebtoken) or framework auth |
| "Create a custom ORM" | Massive undertaking, solved problem | Use Prisma, TypeORM, Drizzle |
| "Build real-time updates with polling" | Inefficient, doesn't scale | Use WebSockets, Server-Sent Events, or push service |
| "Implement custom rate limiting" | Complex edge cases | Use framework middleware or API gateway |

### How to Validate Quickly

```bash
# 1. Search for native integration
"[Service A] [Service B] integration"  # e.g., "Slack GitHub integration"

# 2. Check if deprecated
"[technology] deprecated" OR "[technology] alternative 2024"

# 3. Find official recommendation
Go to official docs → Look for "Integrations" or "Getting Started"
```

### When to Push Back

If validation reveals a better approach, **ask before implementing**:

> "I can implement a Slack webhook, but I noticed Slack has a native GitHub App
> that provides richer notifications without custom code. Would you prefer to
> use that instead?"

This saves time and delivers better solutions.

### Exceptions

Skip deep validation when:
- The approach is explicitly specified with reasoning
- It's a prototype or proof-of-concept
- The user has domain expertise and confirms the approach
- Time-critical fix where any working solution is acceptable

---

## Priority Hierarchy

### When Requirements Conflict, Follow This Order:

#### 1. **CRITICAL - Financial Safety** (P0)

**Always prioritize over everything else**

- Race condition prevention
- Transaction atomicity
- Balance consistency
- Double-spending prevention
- Payout accuracy

**Examples:**

```typescript
// ✅ Correct: Safety first, even if more verbose
await prisma.$transaction(async (tx) => {
  const market = await tx.market.findUnique({ where: { id } });
  if (market.resolved) throw new MarketResolvedError();

  const user = await tx.user.findUnique({ where: { id: userId } });
  if (user.balance < tradeCost) throw new InsufficientBalanceError();

  // All validations inside transaction for safety
  await tx.user.update({ ... });
  await tx.position.create({ ... });
});

// ❌ Wrong: Never sacrifice safety for brevity
const market = await getMarket(id); // TOCTOU vulnerability!
if (market.resolved) throw new Error();
await updateMarket(id);
```

**Decision Rule:** If choice is between safe-but-complex vs. simple-but-risky, always choose safe.

---

#### 2. **CRITICAL - Type Safety** (P0)

**Prevent runtime errors**

- Explicit null/undefined handling
- Proper TypeScript usage
- Input validation
- Return type declarations

**Examples:**

```typescript
// ✅ Correct: Explicit handling
function getMarket(id: string): Market {
  const market = markets.find((m) => m.id === id);
  if (!market) {
    throw new MarketNotFoundError(id);
  }
  return market;
}

// ❌ Wrong: Implicit undefined
function getMarket(id: string): Market {
  return markets.find((m) => m.id === id)!; // Dangerous!
}
```

**Decision Rule:** If uncertain about nullability, add explicit check. Better verbose than crashed.

---

#### 3. **HIGH - SOLID Principles** (P1)

**Long-term maintainability**

- Single Responsibility
- Dependency Injection
- Interface segregation
- Testability

**Trade-off Guidance:**

- Apply when code will be modified/extended
- Skip for one-off scripts or simple utilities
- Defer for prototypes (but refactor before production)

**Examples:**

```typescript
// ✅ Good: DI for testability and flexibility
class MarketService {
  constructor(
    private repository: MarketRepository,
    private calculator: PriceCalculator
  ) {}
}

// ⚠️ Acceptable for simple utility:
function calculatePercentage(value: number, total: number): number {
  if (total === 0) throw new Error('Cannot divide by zero');
  return (value / total) * 100;
}
// No DI needed - pure calculation, no side effects
```

**Decision Rule:** Use SOLID unless it makes simple code complex without clear benefit.

---

#### 4. **HIGH - DRY (Don't Repeat Yourself)** (P1)

**Single source of truth**

- Extract common logic
- Centralize business rules
- Share type definitions

**When to Apply:**

- Logic is truly identical (not just similar)
- Likely to change together
- Used in 3+ places
- Doesn't reduce readability

**When NOT to Apply:**

- Code happens to be similar but represents different concepts
- Extraction creates unclear abstractions
- Only 2 occurrences unlikely to grow
- Would require complex parameters

**Examples:**

```typescript
// ✅ Good: Extract common validation
function validatePositiveNumber(value: number, fieldName: string): void {
  if (value <= 0) {
    throw new ValidationError(`${fieldName} must be positive`, 'INVALID_VALUE', {
      value,
      fieldName,
    });
  }
}

// ❌ Wrong: Over-DRY creates confusion
function validate(value: any, rules: ValidationRule[]): void {
  // Complex generic validator that's harder to understand than specific validators
}

// ✅ Better: Specific validators even if some duplication
function validateTradeAmount(amount: number): void {
  if (amount <= 0 || amount > 10000) {
    throw new ValidationError('Trade amount must be between 0 and 10000');
  }
}

function validateLiquidityParameter(param: number): void {
  if (param <= 0) {
    throw new ValidationError('Liquidity parameter must be positive');
  }
}
```

**Decision Rule:** Extract duplication only if it represents the same concept and will evolve together.

---

#### 5. **MEDIUM - Performance** (P2)

**Optimize when measured need exists**

- Profile before optimizing
- Fix obvious issues (N+1 queries)
- Measure impact of changes

**Always Fix:**

- N+1 query problems
- Database queries in loops
- Missing indexes on frequently queried columns
- O(n²) algorithms where O(n) is simple

**Profile First:**

- Operations <100ms
- Infrequently called code
- Code that would become significantly less readable

**Never Optimize:**

- Without measuring
- Before verifying it's a bottleneck
- At expense of safety or correctness

**Decision Rule:** Safety and correctness first, performance second. But fix obvious inefficiencies.

---

#### 6. **LOW - Code Brevity** (P3)

**Shorter code is not always better**

**Prioritize clarity over brevity:**

```typescript
// ✅ Clear and explicit
function isMarketResolved(market: Market): boolean {
  return market.resolved === true && market.winningOutcome !== null;
}

// ❌ Too clever, harder to understand
const isResolved = (m: Market) => m.resolved && m.winningOutcome != null;
```

**Decision Rule:** Write code for the next person reading it, not for code golf.

---

#### 7. **FUNDAMENTAL - KISS (Keep It Simple, Stupid)** (Cross-cutting)

**Simplicity is a prerequisite for reliability**

KISS applies at all levels - prefer the simplest solution that works:

- Simple functions over complex ones
- Straightforward logic over clever tricks
- Obvious solutions over "elegant" abstractions
- Direct code paths over nested conditionals

**Examples:**

```typescript
// ✅ KISS - Simple and direct
function getUserDisplayName(user: User): string {
  if (user.nickname) {
    return user.nickname;
  }
  return `${user.firstName} ${user.lastName}`;
}

// ❌ Over-engineered
function getUserDisplayName(user: User): string {
  return [
    user.nickname,
    [user.firstName, user.lastName].filter(Boolean).join(' ')
  ].find(Boolean) || 'Anonymous';
}
```

**Decision Rule:** If you can't explain it simply, you don't understand it well enough. Refactor until it's simple.

---

#### 8. **FUNDAMENTAL - Boy Scout Rule** (Cross-cutting)

**Leave the code cleaner than you found it**

Every time you touch code:

- Fix small issues you encounter (typos, unclear names, missing types)
- Remove dead code
- Improve unclear variable names
- Add missing type annotations
- Don't leave broken windows

**Boundaries:**

- Small improvements only - don't refactor entire modules
- Stay within scope of your current task
- If cleanup is large, create separate PR

**Examples:**

```typescript
// Before: You're fixing a bug in this function
function calc(u: any, m: any) {
  // bug fix here
  return u.balance - m.cost;
}

// After: Fixed bug AND cleaned up (Boy Scout Rule)
function calculateRemainingBalance(user: User, market: Market): number {
  return user.balance - market.cost;
}
```

**Decision Rule:** Make small, incremental improvements. Don't leave code worse than you found it.

---

#### 9. **FUNDAMENTAL - Law of Demeter** (Cross-cutting)

**Don't talk to strangers**

Also known as the "principle of least knowledge." An object should only communicate with its immediate neighbors, not reach through objects to access their internals.

**The Rule:** A method should only call methods on:
- `this` (itself)
- Its parameters
- Objects it creates
- Its direct component objects

**Examples:**

```typescript
// ❌ VIOLATION - Reaching through objects (train wreck)
function getCustomerCity(order: Order): string {
  return order.getCustomer().getAddress().getCity().getName();
}

// ✅ CORRECT - Ask, don't reach
function getCustomerCity(order: Order): string {
  return order.getShippingCity(); // Order delegates internally
}

// ❌ VIOLATION - Chained access
function processPayment(user: User): void {
  const card = user.getWallet().getCreditCard().getDetails();
  chargeCard(card);
}

// ✅ CORRECT - Direct delegation
function processPayment(user: User): void {
  const paymentMethod = user.getPreferredPaymentMethod();
  chargePayment(paymentMethod);
}
```

**Why it matters:**
- **Reduces coupling** - Changes to internal structures don't cascade
- **Improves encapsulation** - Objects hide their internal structure
- **Easier testing** - Fewer dependencies to mock
- **Better maintainability** - Changes are localized

**Decision Rule:** If you're chaining more than one dot (except for fluent APIs), consider whether you're violating Law of Demeter.

---

## Decision Framework for Common Scenarios

### Scenario 1: Should I extract this duplicate code?

**Ask:**

1. Is the logic truly identical, or just similar? (Different → Don't extract)
2. Does it represent the same business concept? (Different → Don't extract)
3. Will it change together? (No → Don't extract)
4. Is it used in 3+ places? (No → Consider leaving)
5. Does extraction reduce clarity? (Yes → Don't extract)

**Example Decision:**

```typescript
// Two validation functions that look similar:
function validateMarketTitle(title: string): void {
  if (!title || title.length < 5) {
    throw new ValidationError('Market title must be at least 5 characters');
  }
}

function validateOutcomeName(name: string): void {
  if (!name || name.length < 5) {
    throw new ValidationError('Outcome name must be at least 5 characters');
  }
}

// Decision: DON'T extract
// Reason: Different business concepts, may have different rules in future
// (e.g., market titles might need max length, outcomes might not)
```

---

### Scenario 2: Should I use a transaction here?

**Ask:**

1. Are you modifying data? (No → No transaction needed)
2. Are multiple related records affected? (Yes → USE TRANSACTION)
3. Is financial data involved? (Yes → USE TRANSACTION)
4. Could concurrent access cause issues? (Yes → USE TRANSACTION)
5. Is this a single atomic write? (Yes + No related changes → Transaction optional)

**Decision Examples:**

**Use Transaction:**

```typescript
// Multiple related updates - MUST use transaction
async function placeTrade(userId: string, marketId: string, amount: number) {
  await prisma.$transaction(async (tx) => {
    await tx.user.update({ where: { id: userId }, data: { balance: { decrement: amount } } });
    await tx.position.create({ data: { userId, marketId, shares: amount } });
    await tx.market.update({ where: { id: marketId }, data: { volume: { increment: amount } } });
  });
}
```

**No Transaction Needed:**

```typescript
// Read-only - no transaction needed
async function getMarketDetails(id: string): Promise<Market> {
  return prisma.market.findUnique({
    where: { id },
    include: { outcomes: true },
  });
}

// Single atomic update - transaction optional but safe to include
async function incrementViews(id: string): Promise<void> {
  await prisma.market.update({
    where: { id },
    data: { viewCount: { increment: 1 } },
  });
}
```

---

### Scenario 3: Should I add dependency injection?

**Ask:**

1. Does the code have side effects? (No → Skip DI)
2. Will it be tested? (No → Skip DI)
3. Might implementation change? (No → Skip DI)
4. Is it a service/repository? (Yes → USE DI)
5. Does DI add significant complexity? (Yes → Reconsider)

**Use DI:**

```typescript
// Service with dependencies - use DI
class MarketService {
  constructor(
    private repository: MarketRepository,
    private priceCalculator: PriceCalculator,
    private notificationService: NotificationService
  ) {}
}
```

**Skip DI:**

```typescript
// Pure calculation - no DI needed
function calculateProbability(shares: number[], outcome: number): number {
  const total = shares.reduce((sum, s) => sum + s, 0);
  return shares[outcome] / total;
}

// Simple utility - no DI needed
function formatCurrency(amount: number): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(amount);
}
```

---

### Scenario 4: Should I optimize this code?

**Ask:**

1. Have you measured it's slow? (No → Don't optimize yet)
2. Is it obviously inefficient? (N+1, query in loop → FIX IMMEDIATELY)
3. Is it called frequently? (No → Probably fine)
4. Will optimization reduce clarity? (Yes → Profile first, measure benefit)
5. Is it <100ms? (Yes → Probably fine)

**Always Fix:**

```typescript
// ❌ ALWAYS fix N+1 query
for (const market of markets) {
  const creator = await prisma.user.findUnique({ where: { id: market.creatorId } });
}

// ✅ Fixed
const markets = await prisma.market.findMany({ include: { creator: true } });
```

**Profile First:**

```typescript
// This might be fine - profile before changing
function processMarketData(data: MarketData[]): ProcessedData[] {
  return data
    .filter((d) => d.isActive)
    .map((d) => ({ ...d, calculated: expensiveCalculation(d) }))
    .sort((a, b) => b.score - a.score);
}
// Only optimize if profiling shows it's a bottleneck AND data is large
```

---

### Scenario 5: How much should I comment?

**Comment:**

- Complex algorithms (explain approach)
- Non-obvious business rules
- Race condition prevention strategies
- Mathematical formulas
- Workarounds for external limitations

**Don't Comment:**

- What the code does (code should be self-explanatory)
- Obvious operations
- Instead of fixing unclear code

**Examples:**

**Good Comments:**

```typescript
// LMSR uses logarithmic cost function to prevent market maker insolvency.
// The liquidity parameter b controls market depth and price sensitivity.
// Reference: Hanson (2002) "Logarithmic Market Scoring Rules"
const cost =
  liquidityParameter *
  Math.log(outcomes.reduce((sum, shares) => sum + Math.exp(shares / liquidityParameter), 0));

// Fetch inside transaction to prevent TOCTOU race condition.
// Market could be resolved between check and update if fetched outside.
await prisma.$transaction(async (tx) => {
  const market = await tx.market.findUnique({ where: { id } });
  if (market.resolved) throw new MarketResolvedError();
});
```

**Bad Comments:**

```typescript
// Calculate price
const price = calculatePrice(shares, outcome);

// Loop through outcomes
for (const outcome of outcomes) {
  // Process outcome
  processOutcome(outcome);
}
```

---

## Conflict Resolution Matrix

When two standards conflict, use this priority:

| Lower Priority ↓ / Higher Priority → | Financial Safety | Type Safety | SOLID | DRY   | Performance |
| ------------------------------------ | ---------------- | ----------- | ----- | ----- | ----------- |
| **Financial Safety**                 | -                | -           | -     | -     | -           |
| **Type Safety**                      | Defer            | -           | -     | -     | -           |
| **SOLID**                            | Defer            | Defer       | -     | -     | -           |
| **DRY**                              | Defer            | Defer       | Defer | -     | -           |
| **Performance**                      | Defer            | Defer       | Defer | Defer | -           |

**How to read:** If row conflicts with column, defer to column (higher priority).

**Example:** If applying DRY would reduce type safety → defer to type safety, don't extract.

---

## Summary: The "When in Doubt" Rules

1. **Safety First:** When uncertain, choose the safer option
2. **Explicit Over Implicit:** When uncertain about null, add check
3. **Clarity Over Brevity:** When uncertain, write clearer code
4. **Simple Over Complex:** When uncertain, choose simpler approach
5. **Measure Over Guess:** When uncertain about performance, profile
6. **Test It:** When uncertain about behavior, write test
7. **Ask:** When uncertain about business logic, ask for clarification

---

## AI-Specific Guidelines

### For AI Code Generation:

1. **Default to Safety:** More validations, more transactions, more type safety
2. **Don't Over-Engineer:** Start simple, add complexity only when needed
3. **Include Context:** Add comments explaining race condition prevention, business rules
4. **Fail Explicitly:** Throw descriptive errors rather than silent failures
5. **One Pattern at a Time:** Don't combine multiple patterns unnecessarily

### Red Flags for AI:

- Database write outside transaction → Add transaction
- Array operation without length check → Add validation
- `reduce()` without initial value → Add initial value
- Missing return type → Add explicit type
- Hardcoded dependency → Use dependency injection
- Copy-pasted code → Consider extracting (but check decision framework)

---

**Remember:** These priorities exist to help make decisions, not to create rigid rules. The goal is maintainable, safe, clear code. When priorities conflict, think about the consequences of each choice and default to safety and clarity.
