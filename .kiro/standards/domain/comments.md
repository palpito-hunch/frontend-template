# Comment and Documentation Standards

## Purpose

Good comments explain **why** code exists, not **what** it does. Code should be self-explanatory; comments should provide context that code cannot express.

---

## Core Principles

### The Golden Rule

**Code tells you HOW, comments tell you WHY**

```typescript
// ❌ BAD - States what the code does (obvious from code)
// Calculate the total price by multiplying quantity and unit price
const totalPrice = quantity * unitPrice;

// ✅ GOOD - Explains why we do it this way
// Apply early-bird discount for orders before 10 AM
const totalPrice = isBefore10AM ? quantity * unitPrice * 0.9 : quantity * unitPrice;
```

---

## When to Comment

### ✅ ALWAYS Comment:

#### 1. Complex Algorithms

Explain the approach and why it was chosen.

```typescript
/**
 * LMSR (Logarithmic Market Scoring Rule) price calculation
 *
 * Uses exponential cost function to ensure market maker never runs out
 * of liquidity. The liquidity parameter b controls market depth and
 * price sensitivity. Higher b means more liquidity but less price movement.
 *
 * Formula: P(outcome) = exp(q_i/b) / sum(exp(q_j/b))
 *
 * Reference: Hanson (2002) "Logarithmic Market Scoring Rules for
 * Modular Combinatorial Information Aggregation"
 */
function calculateLMSRPrice(shares: number[], outcome: number, liquidityParameter: number): number {
  const expShares = shares.map((s) => Math.exp(s / liquidityParameter));
  const sumExp = expShares.reduce((sum, e) => sum + e, 0);
  return expShares[outcome] / sumExp;
}
```

#### 2. Non-Obvious Business Rules

Explain requirements that aren't apparent from code.

```typescript
async function canPlaceTrade(user: User, market: Market): Promise<boolean> {
  // Users must be verified to trade on markets with >$1000 liquidity
  // to prevent market manipulation via sock puppet accounts.
  // Requirement from legal review 2024-01-15.
  if (market.liquidity > 1000 && !user.verified) {
    return false;
  }

  // Market creator cannot trade in their own market for 24 hours
  // to prevent insider trading. SEC requirement.
  if (market.creatorId === user.id) {
    const hoursSinceCreation = (Date.now() - market.createdAt.getTime()) / (1000 * 60 * 60);
    if (hoursSinceCreation < 24) {
      return false;
    }
  }

  return true;
}
```

#### 3. Race Condition Prevention Strategies

Explain why operations are structured to prevent race conditions.

```typescript
async function resolveMarket(id: string, outcome: number): Promise<void> {
  // CRITICAL: All validation must happen inside transaction to prevent
  // TOCTOU (Time-of-Check-Time-of-Use) race condition. If we check
  // market.resolved outside the transaction, another request could
  // resolve it between our check and our update, leading to double
  // resolution and incorrect payouts.
  await prisma.$transaction(async (tx) => {
    const market = await tx.market.findUnique({ where: { id } });

    if (market.resolved) {
      throw new MarketAlreadyResolvedError(id);
    }

    // Resolution logic...
  });
}
```

#### 4. Performance Optimizations

Explain why code is optimized in a specific way.

```typescript
async function getMarketDetails(ids: string[]): Promise<Market[]> {
  // Use single batch query instead of loop to avoid N+1 problem.
  // For 100 markets, this reduces queries from 100 to 1, saving ~500ms.
  // Profiled on 2024-01-20, significant improvement for market list page.
  return prisma.market.findMany({
    where: { id: { in: ids } },
    include: { outcomes: true, creator: true },
  });
}
```

#### 5. Workarounds for External Limitations

Explain why code works around a library or system limitation.

```typescript
async function createMarket(data: MarketData): Promise<Market> {
  // Prisma doesn't support setting auto-increment start value,
  // so we manually assign outcome indices to ensure they start at 0.
  // See: https://github.com/prisma/prisma/issues/XXXXX
  const outcomes = data.outcomes.map((name, index) => ({
    name,
    index, // Explicitly set index instead of relying on auto-increment
    shares: new Decimal(0),
  }));

  // ...
}
```

#### 6. Mathematical Formulas and Constants

Provide references and explain the math.

```typescript
// Calculate Kelly Criterion bet size for optimal growth
// Formula: f = (bp - q) / b
// where:
//   f = fraction of bankroll to bet
//   b = odds received on bet (decimal odds - 1)
//   p = probability of winning
//   q = probability of losing (1 - p)
// Reference: Kelly (1956) "A New Interpretation of Information Rate"
function calculateKellyBetSize(bankroll: number, odds: number, winProbability: number): number {
  const b = odds - 1;
  const p = winProbability;
  const q = 1 - p;
  const f = (b * p - q) / b;

  // Kelly suggests betting this fraction of bankroll
  return Math.max(0, f * bankroll);
}
```

#### 7. Security Considerations

Explain security decisions and requirements.

```typescript
async function processPayment(userId: string, amount: number): Promise<void> {
  // SECURITY: Always fetch fresh user data inside transaction to prevent
  // balance manipulation via concurrent requests. User could submit multiple
  // payment requests simultaneously to exploit stale balance checks.
  await prisma.$transaction(async (tx) => {
    const user = await tx.user.findUnique({ where: { id: userId } });

    if (user.balance < amount) {
      throw new InsufficientBalanceError(userId, user.balance, amount);
    }

    // Process payment...
  });
}
```

---

## When NOT to Comment

### ❌ NEVER Comment:

#### 1. What the Code Does (Make Code Self-Explanatory Instead)

```typescript
// ❌ BAD - Comment states the obvious
// Get the user by ID
const user = await getUserById(userId);

// Loop through all markets
for (const market of markets) {
  // Process the market
  processMarket(market);
}

// Calculate total
const total = items.reduce((sum, item) => sum + item.price, 0);

// ✅ GOOD - Code is self-explanatory
const user = await getUserById(userId);

for (const market of markets) {
  processMarket(market);
}

const totalPrice = items.reduce((sum, item) => sum + item.price, 0);
```

#### 2. Variable Declarations

```typescript
// ❌ BAD - Comment just repeats variable name
// The user's balance
const userBalance = user.balance;

// The market ID
const marketId = market.id;

// ✅ GOOD - Descriptive variable names need no comment
const userBalance = user.balance;
const marketId = market.id;
```

#### 3. Function Names That Explain Themselves

```typescript
// ❌ BAD - Comment duplicates function name
// Validates the trade amount
function validateTradeAmount(amount: number): void {
  // ...
}

// ✅ GOOD - Function name is self-documenting
function validateTradeAmount(amount: number): void {
  // ...
}
```

#### 4. To Explain Bad Code (Fix the Code Instead)

```typescript
// ❌ BAD - Comment explains confusing code
// Get balance by finding user and accessing their balance property or 0 if not found
const b = users.find((u) => u.id === uid)?.b || 0;

// ✅ GOOD - Clear code needs no comment
function getUserBalance(userId: string): number {
  const user = users.find((u) => u.id === userId);
  return user?.balance ?? 0;
}

const balance = getUserBalance(userId);
```

#### 5. Obvious Conditionals

```typescript
// ❌ BAD - Obvious from the condition
// Check if user is admin
if (user.role === 'ADMIN') {
  // ...
}

// ✅ GOOD - Condition is self-explanatory
if (user.role === 'ADMIN') {
  // ...
}
```

---

## JSDoc for Public APIs

### Use JSDoc for:

- Public API methods
- Complex utility functions
- Library exports
- Functions with non-obvious parameters

### JSDoc Format

````typescript
/**
 * Calculate LMSR price for specified outcome
 *
 * Uses the Logarithmic Market Scoring Rule to calculate the current
 * price (probability) of a specific outcome based on share distribution.
 *
 * @param shares - Current share distribution across all outcomes (must be non-empty)
 * @param outcome - Index of outcome to price (must be valid array index)
 * @param liquidityParameter - Market depth parameter b (must be positive)
 * @returns Price between 0 and 1 representing outcome probability
 * @throws {ValidationError} If inputs are invalid or array is empty
 * @throws {NumericalError} If calculation encounters overflow
 *
 * @example
 * ```typescript
 * const price = calculateLMSRPrice([100, 50], 0, 250);
 * // Returns ~0.67 (first outcome more likely)
 * ```
 *
 * @see {@link https://en.wikipedia.org/wiki/Logarithmic_Market_Scoring_Rule}
 */
export function calculateLMSRPrice(
  shares: number[],
  outcome: number,
  liquidityParameter: number
): number {
  // Implementation...
}
````

### JSDoc Tags Reference

```typescript
/**
 * Brief description of function
 *
 * Detailed explanation if needed
 *
 * @param paramName - Description of parameter
 * @returns Description of return value
 * @throws {ErrorType} Description of when error is thrown
 * @example Code example showing usage
 * @see Link to related documentation
 * @deprecated Explain why deprecated and what to use instead
 * @internal Mark as internal-only (not public API)
 */
```

---

## TODO Comments

### Format

```typescript
// TODO: Brief description of what needs to be done
// TODO(username): Task with assigned person
// TODO: Task description
//   - Sub-task 1
//   - Sub-task 2
```

### Examples

```typescript
// TODO: Add rate limiting to prevent API abuse
// Target: 100 requests per minute per user

// TODO(alice): Implement caching for market list query
// Currently taking 500ms on large datasets

// TODO: Refactor market resolution logic
//   - Extract payout calculation to separate service
//   - Add transaction retry logic
//   - Improve error messages
//   - Add performance monitoring
```

### When to Use TODOs

**Use for:**

- Known technical debt
- Planned improvements
- Missing features
- Performance optimizations

**Don't use for:**

- Bugs (create issues instead)
- Critical problems (fix immediately)
- Vague intentions (be specific)

---

## FIXME and HACK Comments

### FIXME

For code that works but needs improvement.

```typescript
// FIXME: This validation is incomplete
// Need to add checks for:
// - Maximum outcome name length
// - Special character restrictions
// - Duplicate outcome names
function validateOutcomes(outcomes: string[]): void {
  if (outcomes.length < 2) {
    throw new ValidationError('Need at least 2 outcomes');
  }
}
```

### HACK

For temporary solutions that should be replaced.

```typescript
// HACK: Working around Prisma limitation with manual join
// Remove when https://github.com/prisma/prisma/issues/XXXXX is fixed
const marketsWithCreators = await prisma.$queryRaw`
  SELECT m.*, u.name as creator_name
  FROM markets m
  JOIN users u ON m.creator_id = u.id
`;
```

---

## File and Module Documentation

### File Header Comments

```typescript
/**
 * Market Trading Service
 *
 * Handles all market trading operations including trade placement,
 * price calculations, and position management.
 *
 * Key responsibilities:
 * - Validate trade requests
 * - Calculate prices using LMSR
 * - Update user positions atomically
 * - Enforce trading limits and rules
 *
 * @module services/market-trading
 */
```

### Module-Level Constants

```typescript
/**
 * Maximum trade amount in USD
 *
 * Set to prevent market manipulation and limit platform risk.
 * Can be adjusted based on market conditions and liquidity.
 *
 * Last updated: 2024-01-15
 * Approved by: Risk Management Team
 */
export const MAX_TRADE_AMOUNT = 10000;

/**
 * Minimum number of outcomes required for a market
 *
 * Binary markets (2 outcomes) are simplest but we support multi-outcome.
 * No maximum limit - let market creators decide complexity.
 */
export const MIN_MARKET_OUTCOMES = 2;
```

---

## Inline Comments for Clarity

### When to Add Inline Comments

Use sparingly to clarify non-obvious code.

```typescript
async function calculatePayouts(market: Market): Promise<Payout[]> {
  const positions = await getPositions(market.id);

  // Filter to winning positions only - losers get $0
  const winningPositions = positions.filter((p) => p.outcomeId === market.winningOutcome);

  return winningPositions.map((position) => ({
    userId: position.userId,
    // Each winning share pays out $1
    amount: position.shares,
  }));
}
```

---

## Commented-Out Code

### ❌ NEVER Leave Commented-Out Code

```typescript
// ❌ BAD - Delete it, use git history
function calculatePrice(shares: number[]): number {
  // const oldImplementation = shares[0] / shares.reduce((s, sh) => s + sh, 0);
  // return oldImplementation;

  // const experimentalApproach = Math.log(shares[0]) / Math.log(shares.length);
  // return experimentalApproach;

  return newImplementation(shares);
}

// ✅ GOOD - Clean code, use git to view history
function calculatePrice(shares: number[]): number {
  return newImplementation(shares);
}
```

### Exception: Temporarily During Development

```typescript
// OK during active development (remove before commit)
function calculatePrice(shares: number[]): number {
  // Testing new approach - keep old for comparison
  // return oldImplementation(shares);
  return newImplementation(shares);
}

// But commit should only have:
function calculatePrice(shares: number[]): number {
  return newImplementation(shares);
}
```

---

## Comments for Generated Code

### Mark Generated Sections

```typescript
// ============================================================================
// AUTO-GENERATED - DO NOT EDIT
// Generated by: prisma generate
// Last updated: 2024-01-15 14:30:00
// ============================================================================

export interface Market {
  id: string;
  title: string;
  // ... generated fields
}

// ============================================================================
// END AUTO-GENERATED
// ============================================================================
```

---

## Multi-line Comments

### Block Comments for Detailed Explanations

```typescript
/**
 * Complex market resolution algorithm
 *
 * This function resolves a prediction market and distributes payouts
 * to winning positions. The process must be atomic to prevent:
 *
 * 1. Double resolution - Market gets resolved twice with different outcomes
 * 2. Partial payouts - Some users get paid, others don't (database failure)
 * 3. Balance inconsistencies - User balances don't match payout amounts
 *
 * To ensure atomicity:
 * - All operations run in a single database transaction
 * - Fresh data is fetched within transaction scope
 * - Any error rolls back all changes
 *
 * The payout formula is simple: Each winning share = $1
 *
 * Example: User holds 10 shares of winning outcome
 *          → Receives $10 payout
 *          → Their balance increases by $10
 */
async function resolveMarketAndDistributePayouts(
  marketId: string,
  winningOutcome: number
): Promise<void> {
  // Implementation...
}
```

---

## Summary: Comment Checklist

Before adding a comment, ask:

- [ ] Does this explain **WHY**, not **WHAT**?
- [ ] Is the code already self-explanatory without the comment?
- [ ] Could I make the code clearer instead of adding a comment?
- [ ] Am I explaining a complex algorithm, business rule, or non-obvious decision?
- [ ] Is this a security, performance, or race condition consideration?
- [ ] Would another developer benefit from this context?

**Good Comment:** Provides context code cannot express
**Bad Comment:** Repeats what code already says
**No Comment:** Code is self-explanatory

---

## Examples by Category

### Algorithm Explanation ✅

```typescript
// Use binary search for O(log n) lookup instead of linear O(n)
// because market list can contain 10,000+ markets
const marketIndex = binarySearch(markets, targetId);
```

### Business Rule ✅

```typescript
// Markets auto-resolve after 30 days of inactivity per platform policy
// to prevent indefinitely open markets tying up user funds
if (daysSinceLastTrade > 30) {
  await autoResolveMarket(market.id);
}
```

### Obvious Statement ❌

```typescript
// Add 1 to the counter
counter += 1;
```

### Code Explanation ❌

```typescript
// Loop through all users
for (const user of users) {
  // ...
}
```

**Remember:** Comments age poorly. Code is always up to date. Prefer self-documenting code over comments.
